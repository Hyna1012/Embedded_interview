# 多普勒模拟器项目面试问答

> 说明：以下问答基于当前代码仓库整理。能从代码直接确认的地方按实现来写；涉及设计动机的部分，如果代码没有明确注释，我会用“从当前实现看”来表述，避免虚构。

## 核心代码速览

### 1. 系统主入口

文件：`C_armgcc_SGR2301dopplerDetection/app/src/main.c`

```c
int main(void)
{
    extend_sram();
    BSP_Init();

    Sensor_Start();
    tlv_init();
    while (1)
    {
        Sensor_Process();
    }
}
```

你讲这段时要抓住 3 个点：
- 先做 SRAM 和板级准备，再启动传感器
- 初始化通信协议
- 主循环核心是持续处理雷达事件消息

### 2. 板级初始化与相位补偿准备

文件：`C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`

```c
void BSP_Init(void)
{
    nvic_priority_group_config(NVIC_PRIORITY_GROUP_4);
    Gpio_All_Ana_Cfg();
    CLK_Init_240MHz();
    FL_DelayInit();
    BSP_CfgInit();
    RawData_Init(921600);

    adc_if_init(&g_adc_cfg);
    adc_aux_init();
    CommuProtocolInit();
    g_sensor_status.afe_status = afe_init(&g_sensor_cfg_default.rfConfig);

    for (uint16_t i = 0; i < virtualChannels; i++)
    {
        compensateReal = arm_cos_f32(deltaPhiList[i] / 180.f * PI);
        compensateImag = arm_sin_f32(deltaPhiList[i] / 180.f * PI);
        arm_fill_f32(compensateReal, (float32_t *)compensateArray[i], ADCSamples * 2);
        for (uint16_t j = 0; j < ADCSamples; j++)
        {
            compensateArray[i][j * 2 + 1] = compensateImag;
        }
    }

    arm_hamming_f32(winHammingf32, RangeFFTSize);
    WDT_Init(3000);
}
```

你讲这段时要抓住 4 个点：
- MCU、UART、ADC、AFE 都在这里拉起
- 串口波特率是 `921600`
- 6 个虚拟通道的相位补偿向量在初始化阶段预生成
- Hamming 窗和看门狗也在启动阶段准备好

### 3. 单个 chirp 的数据处理流程

文件：`C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`

```c
for (uint16_t i = 0; i < ADCSamples; i++)
{
    chirp_buffer[0][i] = (float32_t) * (adcBuffer1 + i);
    chirp_buffer[1][i] = (float32_t) * (adcBuffer2 + i);
    chirp_buffer[2][i] = (float32_t) * (adcBuffer3 + i);
}

for (uint16_t i = 0; i < NUM_RX_ANT_MAX; i++)
{
    arm_mean_f32(chirp_buffer[i], ADCSamples, &sum_mean);
    arm_offset_f32(chirp_buffer[i], -sum_mean, chirp_buffer[i], ADCSamples);
    arm_mult_f32(winHammingf32, chirp_buffer[i], chirp_buffer[i], ADCSamples);

    for (uint16_t j = ADCSamples - 1; j > 0; j--) {
        chirp_buffer[i][j*2] = chirp_buffer[i][j];
        chirp_buffer[i][j*2+1] = 0.f;
    }
    chirp_buffer[i][1] = 0.f;

    arm_cfft_f32(&arm_cfft_sR_f32_len256, chirp_buffer[i], 0, 1);
}
```

```c
arm_cmplx_mult_cmplx_f32((float32_t *)chirp_buffer[i],
                         (float32_t *)compensateArray[i],
                         (float32_t *)chirp_buffer[i],
                         RangeFFTSize);
arm_copy_f32(chirp_buffer[i], radarCube[chirp_tdm_idx][i], RangeFFTSize);
```

你讲这段时要抓住 4 个点：
- 三路 ADC 先转成浮点
- 做 DC 去除、窗函数、复数化和 Range FFT
- 对不同虚拟通道做固定相位补偿
- 最终结果按 chirp 和通道写入 `radarCube`

### 4. 帧级处理与距离-多普勒热图生成

文件：`C_armgcc_SGR2301dopplerDetection/gesture/process.c`

```c
for (i = 0; i < virtualChannels; i++)
{
    arm_scale_f32((float32_t *)staticRadarCube[i], 1.f / chirpLoops, (float32_t *)staticRadarCube[i], RangeFFTSize);
    for (j = 0; j < chirpLoops; j++)
    {
        arm_sub_f32((float32_t *)radarCube[j][i],
                    (float32_t *)staticRadarCube[i],
                    (float32_t *)radarCube[j][i],
                    ADCSamples);
    }
}
```

```c
for (i = rangeBinStart; i < rangeBinEnd; i++)
{
    for (j = 0; j < chirpLoops; j++)
    {
        dopplerFFTBuffer[j * 2] = radarCube[j][1][i * 2];
        dopplerFFTBuffer[j * 2 + 1] = radarCube[j][1][i * 2 + 1];
    }
    arm_cfft_f32(&arm_cfft_sR_f32_len32, dopplerFFTBuffer, 0, 1);
    arm_cmplx_mag_f32(dopplerFFTBuffer, dopplerFFTBuffer, chirpLoops);
    for (j = 0; j < chirpLoops; j++)
    {
        rdMap[j][i] = dopplerFFTBuffer[j];
    }
}
```

你讲这段时要抓住 4 个点：
- 先做静态杂波抑制
- 再按距离 bin 取慢时间序列
- 对慢时间序列做 Doppler FFT
- 最终把结果写成 `Range-Doppler Map`

### 5. TLV 协议与模式发送

文件：`C_armgcc_SGR2301dopplerDetection/gesture/tlv.c`

```c
static void tlv_mode_raw_data() {
    tlv_mask_reset();
    tlv_mask[tlvIdxRawData] = true;
    header.tlv_nums = 1;
}

static void tlv_mode_range_doppler_map() {
    tlv_mask_reset();
    tlv_mask[tlvIdxRangeDopplerMap] = true;
    header.tlv_nums = 1;
}
```

```c
void data_transmit() {
    work_mode_identify();
    TransmitData_header();
    for (uint16_t i = 0; i < TLV_TOTAL_NUMS; i++) {
        if (tlv_mask[i]) {
            func_arr[i]();
        }
    }
}
```

你讲这段时要抓住 3 个点：
- 支持 `RawData` 和 `RangeDopplerMap` 两种工作模式
- 帧头后按 TLV 类型和长度发数据
- 当前模式切换是按工作模式动态决定发送内容

### 6. 上位机帧同步与解包

文件：`python310_SGR2301DopplerDetection/gui/capThread.py`

```python
class CapThread(QThread):
    cpltSig = QtCore.Signal(bool)
    start_magic_word = b'\\x02\\x01\\x04\\x03\\x06\\x05\\x08\\x07'
```

```python
pos = self.find_start(pointer - len(self.start_magic_word) + 1,
                      pointer + len(receive) - len(self.start_magic_word) + 1, cache)
if pos != -1:
    self.main_thread.bytes_data = cache[:pos]
    self.cpltSig.emit(True)
```

文件：`python310_SGR2301DopplerDetection/gui/unpack.py`

```python
header_data = data[:HEADER_SIZE]
info = struct.unpack("<HHHH", header_data[8:HEADER_SIZE])
frame_data.header.frame = info[0]
frame_data.header.delay = info[1]
frame_data.header.tlv_nums = info[2]
frame_data.header.data_len = info[3]
```

```python
elif t == DataTypeEnum.RangeDopplerMap:
    unpack_res = struct.unpack("<" + "f" * frame_data.chirp_loops * frame_data.range_bin_len, data[:l])
```

你讲这段时要抓住 3 个点：
- 先靠 magic word 做帧同步
- 再按 header + TLV 逐段解包
- 解包后恢复成原始数据或热图矩阵

### 7. Python 仿真脚本

文件：`python312_dopplerSimulator/simu/simuDoppler.py`

```python
simulatorComplexSignal = np.exp(
    1j * 2 * np.pi * (fd_simu + fr_simu) * (np.arange(0, T_ADC, T_ADC / ADC_SAMPLES) + i * T_CHIRP)
)
radarCube[i] *= simulatorComplexSignal
```

```python
radarCube = np.fft.fft(radarCube, axis=-1)
radarCube = np.fft.fft(radarCube, axis=0)
```

你讲这段时要抓住 3 个点：
- 仿真脚本会同时叠加距离频率和多普勒频率
- 再用两次 FFT 验证距离维和速度维结果
- 它的价值是帮助理解热图和速度/距离映射，而不是替代板端实现

## 1. 你这个多普勒项目整体是怎么工作的？

**参考回答：**

这个项目可以分成三层。第一层是嵌入式采集和处理层，基于 `AT32F435 / RKB2301` 完成 `24GHz FMCW` 雷达采样、Range FFT、相位补偿和 Doppler FFT；第二层是通信和可视化层，通过 `TLV + UART` 把原始数据或距离-多普勒热图发到上位机并实时显示；第三层是 Python 仿真层，用脚本去验证距离频率、多普勒频率以及热图生成逻辑。整体上是“板端处理 - 协议传输 - 上位机显示 - 仿真验证”的闭环。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/app/src/main.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/process.c`
- `/D:/project/postgraduate/sgr/doppler/python310_SGR2301DopplerDetection/gui/gui.py`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/simu/simuDoppler.py`

## 2. 你在这个项目里主要负责什么？

**参考回答：**

我主要负责两部分。第一部分是板端链路开发，包括采样参数配置、ADC 数据处理、二维 FFT 处理、热图生成和串口传输；第二部分是 Python 配套工具开发，包括上位机解包显示和多普勒仿真验证。换句话说，我做的不只是单点算法，而是把板端处理链路和验证工具一起打通。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/tlv.c`
- `/D:/project/postgraduate/sgr/doppler/python310_SGR2301DopplerDetection/gui/unpack.py`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/dopplergui/main.py`

## 3. 这个项目为什么既有检测工程，又有模拟器脚本和 GUI？

**参考回答：**

因为这个项目不只是做板端处理，还要验证“距离-速度映射关系”和“热图是否符合预期”。检测工程负责真实雷达数据采集和处理，Python 上位机负责接收和显示，模拟器脚本负责从原理上生成带距离频率和多普勒频率的信号，帮助验证二维 FFT 的结果。所以这三部分组合起来，能同时覆盖真实链路和理论验证。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/python310_SGR2301DopplerDetection/gui/gui.py`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/simu/simuDoppler.py`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/simu/simuRangeDoppler.py`

## 4. MCU 端的数据处理流程具体是怎样的？

**参考回答：**

MCU 端主循环不断调用 `Sensor_Process()`。当一个 chirp 的 ADC 采样结束后，会先把三路 ADC 数据转成浮点，再做 DC 去除、Hamming 窗和 Range FFT。因为这里是 `2Tx3Rx` 的 TDM-MIMO 结构，所以每个 chirp 还会根据当前发射通道做虚拟通道相位补偿，然后存进 `radarCube`。等一帧结束后，再做静态杂波抑制，对每个距离 bin 提取慢时间序列做 Doppler FFT，最后得到距离-多普勒热图并通过 TLV 发到上位机。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/process.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/tlv.c`

## 5. 这个项目的关键雷达参数是什么？

**参考回答：**

从默认配置看，项目的起始频率是 `23600MHz`，带宽是 `1000MHz`，每帧 chirp 数是 `chirpLoops * NUM_TX_ANT_MAX`，也就是 `32 * 2 = 64` 个 chirp，单个 chirp 的采样点数是 `256`。这些参数决定了距离分辨率、速度分辨率以及整帧计算量，也说明这个项目是按 `24GHz FMCW` 方案在做距离-多普勒验证。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp_config.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/defines.h`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/Config/sensor_cfg.h`

## 6. 2Tx3Rx 和 6 个虚拟通道在这里是什么意思？

**参考回答：**

这里实际接收通道是 `3` 路，发射通道是 `2` 路。通过 TDM 的方式让两个发射天线轮流工作，就能把两次发射对应的 3 路接收结果组合成 `6` 个虚拟通道。代码里也是按这个逻辑组织数据的，偶数 chirp 对应一组发射天线，奇数 chirp 对应另一组，最终在 `radarCube` 里形成 `chirp × virtualChannel × range` 的数据结构。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/defines.h`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/Config/sensor_cfg.h`

## 7. 你们为什么要做相位补偿？怎么实现的？

**参考回答：**

相位补偿是为了减小不同虚拟通道之间的固定相位偏差，避免后续通道数据不一致。代码里提前准备了一组 `deltaPhiList`，然后在初始化时把每个通道对应的旋转向量预先算出来，真正处理 chirp 时直接做复数乘法补偿。这么做的好处是把补偿成本提前到初始化阶段，运行时只做一次复数乘法就可以。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`

## 8. `deltaPhiList` 这些补偿角度是怎么来的？

**参考回答：**

从当前实现看，这些角度是离线标定后固化到代码里的固定补偿值，因为代码里直接给了 6 个通道的角度列表，并在初始化时生成补偿向量。也就是说，这更像是一次标定结果在运行时的应用，而不是每次上电都重新估计。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`

## 9. 为什么要做 DC 去除、窗函数和 Range FFT？

**参考回答：**

这是 FMCW 雷达的基础处理链路。DC 去除是为了减小直流偏置和静态耦合对频谱的影响；窗函数是为了抑制频谱泄漏；Range FFT 是把单个 chirp 的时域采样映射到距离维。只有先把距离维算出来，后面才有可能进一步做速度维处理。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`

## 10. 静态杂波抑制在这里怎么做的？

**参考回答：**

当前代码用了静态杂波抑制，而不是更复杂的自适应方法。做法是先在 chirp 处理阶段把每个通道的结果累加到 `staticRadarCube`，帧结束后再除以 chirp 数得到平均静态背景，然后从每个 chirp 的结果里减掉。这样可以把场景里相对静止的强反射先压下去，让后面的 Doppler 结果更聚焦在动态目标上。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/process.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/defines.h`

## 11. 距离-多普勒热图是怎么生成的？

**参考回答：**

一帧数据经过 Range FFT 和静态杂波抑制后，代码会对每个距离 bin 抽取一条慢时间序列，也就是同一距离 bin 在多个 chirp 上的复数值变化。然后对这条慢时间序列做 Doppler FFT，再取幅值，最终把所有距离 bin 的结果拼起来形成二维热图。横轴可以理解为速度维，纵轴是距离维。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/process.c`
- `/D:/project/postgraduate/sgr/doppler/python310_SGR2301DopplerDetection/gui/gui.py`

## 12. 为什么这里只处理 `rangeBinStart=0` 到 `rangeBinEnd=10`？

**参考回答：**

从当前实现看，这是一个验证型工程，所以距离 bin 只取了前 `10` 个做 Doppler 图。这样做可以先把计算量和上位机显示范围控制住，同时聚焦近距离目标的验证。后续如果要扩更远距离或者更完整的热图范围，可以把 bin 范围放大。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/defines.h`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/process.c`

## 13. 为什么 `process.c` 里当前只取了 `radarCube[j][1]` 这个通道来做热图？

**参考回答：**

从当前实现看，热图生成阶段确实只用了一个虚拟通道的数据，也就是 `radarCube[j][1]`。这说明当前代码更偏向先验证单通道链路和热图结果，而不是做多通道融合、波束形成或者角度估计。这样工程复杂度更低，也更适合先把距离-速度维度跑通。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/process.c`

## 14. 这个项目为什么设计了 `RawData` 和 `RangeDopplerMap` 两种模式？

**参考回答：**

因为不同阶段关注的数据不同。调底层链路时更适合直接看原始采样数据；验证热图算法时更关心 Range-Doppler Map。用工作模式切换能把“采样验证”和“结果展示”分开，不需要每次都传所有数据，联调起来也更清晰。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/tlv.h`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/tlv.c`
- `/D:/project/postgraduate/sgr/doppler/python310_SGR2301DopplerDetection/gui/capThread.py`

## 15. 你们的 TLV 协议是怎么设计的？

**参考回答：**

协议结构是“帧头 + 若干 TLV 数据段”。帧头里有 magic word、frame、delay、tlv_nums 和 data_len，用来做帧同步和长度校验；每个 TLV 再带 type 和 length，接收端根据 type 判断当前是原始数据还是热图数据。这样设计的好处是扩展性比较好，后面想加新的中间结果也比较方便。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/tlv.h`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/tlv.c`
- `/D:/project/postgraduate/sgr/doppler/python310_SGR2301DopplerDetection/gui/unpack.py`

## 16. 为什么串口要开到 `921600`？

**参考回答：**

因为这个项目不仅要发控制信息，还要发浮点原始数据或整幅热图矩阵，数据量比普通状态打印大得多。`921600` 可以在实现成本不高的前提下提供相对更高的吞吐，比较适合实验阶段做上位机联调。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/driver/src/drv_rawdata.c`

## 17. 上位机怎么保证接收到的是完整一帧？

**参考回答：**

上位机不是简单地按固定长度读串口，而是先在字节流里找 magic word，找到一帧起点后继续缓存数据，直到遇到下一帧的起点，再把前一帧完整截出来处理。这样做能适应串口流式接收的特点，也能减少半帧数据直接参与解包带来的错误。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/python310_SGR2301DopplerDetection/gui/capThread.py`

## 18. 上位机里热图的坐标轴是怎么来的？

**参考回答：**

GUI 里先根据雷达参数算出距离分辨率和速度分辨率，再把热图的像素坐标映射成真实物理坐标。显示前还会先对 Doppler 维做 `fftshift`，这样负速度和正速度会分布在中心两侧，更符合我们对速度轴的直观理解。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/python310_SGR2301DopplerDetection/gui/gui.py`

## 19. 这个项目里的 Python 模拟器具体做了什么？

**参考回答：**

模拟器主要做两件事。第一是从原理上生成带距离频率和多普勒频率的复数信号，再通过两次 FFT 看热图是否落在预期的位置；第二是做参数和公式验证，比如距离、速度、IF 频率之间的换算关系。它的作用是帮助确认理论和工程实现是一致的。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/simu/simuDoppler.py`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/simu/simuRangeDoppler.py`

## 20. `simuDoppler.py` 和 `simuRangeDoppler.py` 有什么区别？

**参考回答：**

`simuDoppler.py` 更偏向直接验证“叠加距离频率和多普勒频率后，两次 FFT 能不能在热图上得到预期结果”；`simuRangeDoppler.py` 更像从 FMCW 发射、回波、混频和 ADC 采样过程往下走，模拟链路更完整一些。前者更适合理解热图位置，后者更适合理解 FMCW 采样和混频机理。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/simu/simuDoppler.py`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/simu/simuRangeDoppler.py`

## 21. 串口控制界面里的 `v=` 命令和速度有什么关系？

**参考回答：**

从 GUI 代码看，界面会先把用户输入的 Doppler 频率换算成速度，同时还会根据当前 chirp 斜率和目标距离算出距离对应的 IF 频率，最后把两者叠加成一个总频率发给设备。所以 `v=` 实际上不是单纯在发“速度”，而是在发和当前模拟目标对应的综合频率参数。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/dopplergui/main.py`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/testgui/main.py`

## 22. 这个项目里还有 CW 模式，它是做什么的？

**参考回答：**

从板端和 GUI 的实现看，除了 FMCW 模式，系统还支持切到 CW 模式。CW 更适合做纯多普勒或频率相关的单项验证，比如测试频率设置、状态切换或者某些调试场景。它说明这个项目不只是一个固定算法 demo，而是做了多种工作状态的工程化验证。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/dopplergui/main.py`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/testgui/main.py`

## 23. 为什么这个项目会选择 `AT32F435 / RKB2301`？

**参考回答：**

因为这个项目在 MCU 端不仅要做 ADC 采样和外设控制，还要做 Range FFT、Doppler FFT、相位补偿和热图输出，对算力、内存和外设带宽都有要求。代码里一开始就做了 `extend_sram()`，说明内存资源也比较敏感；另外项目本身就是基于 `RKB2301` 平台开发的，所以 `AT32F435` 既满足需求，也符合现有硬件平台条件。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/app/src/main.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/projects/prj_config.h`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/Config/sensor_cfg.h`

## 24. 为什么这里用了 `CMSIS-DSP`？

**参考回答：**

因为项目里大量用到了 FFT、均值、偏移、窗函数、复数乘法和幅值计算，这些都是 `CMSIS-DSP` 很擅长的场景。对于 `Cortex-M` 平台来说，用 ARM 官方的 DSP 库比自己手写基础算子更稳，也更容易兼顾性能和可维护性。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/process.c`

## 25. 这个项目里最有代表性的工程问题是什么？

**参考回答：**

我觉得最有代表性的问题不是单个 FFT，而是“怎么把理论上的距离-速度关系稳定地变成板端和上位机都能对上的热图结果”。这里面包括采样参数设置、TDM-MIMO 数据组织、相位补偿、静态杂波抑制、协议设计、串口同步和仿真验证，任何一个环节不稳，最后热图位置都可能不对。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/process.c`
- `/D:/project/postgraduate/sgr/doppler/python310_SGR2301DopplerDetection/gui/capThread.py`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/simu/simuDoppler.py`

## 26. 这个项目里你做了哪些和实时性、性能相关的考虑？

**参考回答：**

从当前实现看，主要有几类。第一是处理流程尽量依赖 `CMSIS-DSP`；第二是工作模式可切换，不必每次都传全部数据；第三是热图只先取前 `10` 个距离 bin 做验证，控制计算和传输开销；第四是把相位补偿向量提前生成，减少运行时重复计算。整体思路是先把链路跑通，再逐步往更复杂的算法扩展。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/defines.h`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/tlv.c`

## 27. 你是怎么做调试和排障的？

**参考回答：**

我会分三层排查。第一层看板端链路是否正常，比如 AFE 是否锁定、ADC 是否完成采样、帧中断和 chirp 中断有没有按预期走；第二层看中间结果，比如原始数据、Range FFT 或热图矩阵是不是合理；第三层再用 Python 仿真去对照距离和速度应该落在哪个位置。这样可以把“底层问题”和“算法理解问题”分开。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/tlv.c`
- `/D:/project/postgraduate/sgr/doppler/python310_SGR2301DopplerDetection/gui/unpack.py`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/simu/simuRangeDoppler.py`

## 28. 这个项目当前还有哪些局限？

**参考回答：**

从当前实现看，它更像一个热图验证和模拟工程，而不是完整的目标检测系统。比如 Doppler 图生成阶段只用了一个通道，距离 bin 范围也比较小，还没有看到更完整的目标检测、CFAR、角度估计或多目标跟踪逻辑。所以它的核心价值更偏向“把 FMCW 多普勒链路和验证工具跑通”。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/process.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/defines.h`

## 29. 如果继续做这个项目，你会优先优化什么？

**参考回答：**

我会优先做三件事。第一，把单通道热图扩到多通道融合，为后续角度估计或更稳的目标检测打基础；第二，引入更完整的目标检测逻辑，比如 CFAR 或峰值筛选，而不只是直接显示热图；第三，把仿真参数、板端参数和上位机显示参数统一成一套更清晰的配置体系，减少联调时的歧义。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/process.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp_config.c`
- `/D:/project/postgraduate/sgr/doppler/python310_SGR2301DopplerDetection/gui/gui.py`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/simu/simuDoppler.py`

## 30. 如果面试官让你一句话总结这个项目的价值，你怎么说？

**参考回答：**

我会说，这个项目的价值不只是画出一张距离-多普勒热图，而是把 `24GHz FMCW` 雷达板端采集、二维 FFT 处理、协议传输、上位机可视化和 Python 多普勒仿真验证整体打通，形成了一套可联调、可解释、可扩展的验证链路。

**复习代码：**

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection`
- `/D:/project/postgraduate/sgr/doppler/python310_SGR2301DopplerDetection`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator`

## 复习顺序建议

1. 先讲清系统链路：`main.c -> bsp.c -> process.c -> tlv.c -> capThread.py -> unpack.py`
2. 再复习板端信号处理：DC 去除、Hamming 窗、Range FFT、相位补偿、静态杂波抑制、Doppler FFT
3. 再复习协议和显示：magic word、TLV、串口帧同步、热图坐标映射
4. 最后复习 Python 仿真：`simuDoppler.py` 和 `simuRangeDoppler.py`

## 你下一步最该背熟的 3 个点

- 板端从 ADC 到距离-多普勒热图的完整流程
- 2Tx3Rx TDM-MIMO、虚拟通道和相位补偿的作用
- Python 仿真脚本为什么能帮助验证热图和距离/速度映射

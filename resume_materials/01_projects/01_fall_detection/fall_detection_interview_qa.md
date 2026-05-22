# 跌倒检测项目面试问答

> 说明：以下问答基于当前代码仓库整理。回答尽量贴合已有实现；没有明确实验指标的地方，统一用原理性表达，不虚构数据。

## 核心代码速览

### 1. 系统主入口

文件：`C_armgcc_SGR2301FallDetection/app/src/main.c`

```c
int main(void)
{
    extend_sram();
    BSP_Init();
    Sensor_Start();
    frame_process_init();
    tlv_init();
#ifdef NN_INFERENCE_TIME
    Timer_Init_Us(4, reload_mode, 10, timer_callback);
#endif
    while (1)
    {
        Sensor_Process();
    }
}
```

你讲这段时要抓住 3 个点：
- 先做板级和内存准备，再启动传感器
- 初始化推理和通信模块
- 主循环核心就是持续处理传感器消息

### 2. 板级初始化与采样链路启动

文件：`C_armgcc_SGR2301FallDetection/BSP/src/bsp.c`

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
    g_sensor_status.mcuTemp = adc_aux_get_mcu_temp();

    CommuProtocolInit();
    g_sensor_status.afe_status = afe_init(&g_sensor_cfg_default.rfConfig);

    arm_hamming_f32(hamming, ADCSamples);
    WDT_Init(3000);
}
```

你讲这段时要抓住 4 个点：
- 时钟、GPIO、串口、ADC、AFE 都在这里拉起
- 串口波特率是 `921600`
- Hamming 窗在初始化阶段提前生成
- 最后加了看门狗保证运行稳定

### 3. 单个 Chirp 的信号处理

文件：`C_armgcc_SGR2301FallDetection/BSP/src/bsp.c`

```c
for (uint16_t i = 0; i < ADCSamples; i++)
{
    chirpBuffer[i] = (float32_t) * (adcBuffer1 + i);
}

arm_mean_f32(chirpBuffer, ADCSamples, &mean);
arm_offset_f32(chirpBuffer, -mean, chirpBuffer, ADCSamples);
arm_mult_f32(chirpBuffer, hamming, chirpBuffer, ADCSamples);

for (uint16_t i = ADCSamples - 1; i > 0; i--)
{
    chirpBuffer[i * 2] = chirpBuffer[i];
    chirpBuffer[i * 2 + 1] = 0;
}
chirpBuffer[1] = 0.f;

arm_cfft_f32(matchFFTInstance(rangeFFTSize), chirpBuffer, 0, 1);
arm_copy_f32(&(chirpBuffer[rangeBinStart * 2]), radarCube[chirp_idx], rangeBinLen * 2);
arm_add_f32(staticRadarCube, (float32_t *)radarCube[chirp_idx], staticRadarCube, rangeBinLen * 2);
```

你讲这段时要抓住 4 个点：
- ADC 原始值先转成浮点
- 做 DC 去除和窗函数
- 组织成复数输入后做 Range FFT
- 每个 chirp 的结果写入 `radarCube`，同时累积静态杂波参考

### 4. 帧级处理、存在性检测与特征构造

文件：`C_armgcc_SGR2301FallDetection/fallen/Src/process.c`

```c
arm_scale_f32(staticRadarCube, 1.f / chirpLoops, staticRadarCube, rangeBinLen * 2);
for (i = 0; i < chirpLoops; i++)
{
    arm_sub_f32(radarCube[i], staticRadarCube, radarCube[i], rangeBinLen * 2);
}

arm_cmplx_mag_f32(radarCube[0], &(rangeProfileChirp0[rangeBinStart]), rangeBinLen);
arm_max_f32(rangeProfileChirp0, rangeBinEnd, &maxValue, &maxIdx);

if (maxValue > NN_EXISTANCE_THRESHOLD)
{
    ring_array_update_u8(&nnExistHandler, 1);
}
else
{
    ring_array_update_u8(&nnExistHandler, 0);
}
```

```c
for (i = 0; i < rangeBinLen; i++)
{
    arm_fill_f32(0.f, dopplerBuffer, dopplerFFTSize * 2);
    for (j = 0; j < chirpLoops; j++)
    {
        dopplerBuffer[j * 2] = radarCube[j][i * 2];
        dopplerBuffer[j * 2 + 1] = radarCube[j][i * 2 + 1];
    }
    arm_cfft_f32(matchFFTInstance(dopplerFFTSize), dopplerBuffer, 0, 1);
    arm_cmplx_mag_f32(dopplerBuffer, dopplerBuffer, dopplerFFTSize);
    arm_max_f32(dopplerBuffer, dopplerFFTSize, &maxValue, &maxIdx);
}
```

你讲这段时要抓住 4 个点：
- 先做静态杂波抑制
- 再做存在性检测，避免无人场景无效推理
- 然后对每个距离 bin 做 Doppler FFT
- 最终提取最大幅值形成距离向量给模型

### 5. 模型输入与分类调用

文件：`C_armgcc_SGR2301FallDetection/fallen/Src/process.c`

```c
#ifdef NN_USING_CNN
nn.input[CNN_WIN_LWN - 1][i] = maxValue * RANGE_VECTOR_SCALE;
if (nn.input[CNN_WIN_LWN - 1][i] > 1.f)
{
    nn.input[CNN_WIN_LWN - 1][i] = 1.f;
}
#endif

#ifdef NN_USING_GRU
nn.gru1.x[i] = maxValue * RANGE_VECTOR_SCALE;
if (nn.gru1.x[i] > 1.f)
{
    nn.gru1.x[i] = 1.f;
}
#endif

#ifdef NN_USING_LSTM
nn.lstm1.x[i] = maxValue * RANGE_VECTOR_SCALE;
if (nn.lstm1.x[i] > 1.f)
{
    nn.lstm1.x[i] = 1.f;
}
#endif

classification(&nn);
```

你讲这段时要抓住 3 个点：
- 不同模型共用同一套特征提取流程
- 输入会先做缩放和裁剪
- 最终统一通过 `classification(&nn)` 调用推理

### 6. TLV 协议发送

文件：`C_armgcc_SGR2301FallDetection/fallen/Src/tlv.c`

```c
static void tlv_mode_classification(void)
{
    tlv_mask_reset();
    tlv_mask.v.rangeVector = true;
    tlv_mask.v.Prob = true;
    tlv_mask.v.MTI = true;
    header.numTlvs = 3;
}

void data_transmit(void)
{
    work_mode_identify();
    SendData_header();
    for (uint16_t i = 0; i < TLV_TOTAL_NUMS; i++)
    {
        if (tlv_mask.array[i])
        {
            func_arr[i]();
        }
    }
}
```

你讲这段时要抓住 3 个点：
- 不同工作模式发送不同数据内容
- 每个数据块都带类型和长度
- 分类模式下重点发送 `MTI + rangeVector + Prob`

### 7. 上位机解包与模型导出

文件：`python312_SGR2301FallDetection/unpack.py`

```python
header_data = data[:const.cfg.HEADER_SIZE]
frame_data.header.start_magic_word = header_data[:8]
info = struct.unpack("<HHHHH", header_data[8:const.cfg.HEADER_SIZE])
frame_data.header.frame = info[0]
frame_data.header.delay = info[1]
frame_data.header.tlv_nums = info[2]
frame_data.header.data_len = info[3]
frame_data.header.nnExist = info[4]
```

```python
[t, l] = struct.unpack("<HH", data[0:4])
if t == DataTypeEnum.RangeVector:
    unpack_res = struct.unpack("<" + "f" * const.cfg.RANGE_BIN_LEN, data[:l])
elif t == DataTypeEnum.Prob:
    unpack_res = struct.unpack("<ff", data[:l])
```

文件：`python312_SGR2301FallDetection/nn/cnn/export2c.py`

```python
cFile = open("./C/cnnData.h", "w")
for ker in range(len(nameList)):
    s = "float " + nameList[ker] + shapeList[ker] + " = "
    cFile.write(s)
    kernel_data = dataList[ker]
    if "Conv" in nameList[ker]:
        s = supportFunc.print_in_c_array(kernel_data, nameList[ker], "float", False, False)
        cFile.write(s)
```

你讲这段时要抓住 3 个点：
- 上位机按帧头和 TLV 逐段解包
- 不同类型恢复成不同结构化数据
- 训练好的模型参数会转成 C 数组头文件，供 MCU 直接编译使用

## 1. 你这个跌倒检测项目整体是怎么工作的？

**参考回答：**

这个项目可以分成三层。第一层是嵌入式采集层，基于 `AT32F435 / RKB2301` 完成雷达前端初始化、ADC 采样和主循环调度；第二层是端侧信号处理和推理层，对每帧雷达数据做预处理、FFT、杂波抑制、距离向量提取，再送入轻量化神经网络做跌倒分类；第三层是上位机和训练工具链，负责数据采集、TLV 解包、可视化、模型训练和参数导出。整体上是“采集 - 特征提取 - 端侧推理 - 上位机联调”的闭环。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/app/src/main.c`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/gui/inference.py`

## 2. 你在这个项目里主要负责什么？

**参考回答：**

我主要负责两部分。第一部分是嵌入式端开发，包括板级初始化、数据采集链路、串口数据输出以及信号处理和模型推理流程的集成。第二部分是算法部署配套工作，包括轻量化模型训练、参数导出、上位机解包与可视化联调。换句话说，我做的是把模型和雷达数据流真正落到板子上跑起来，而不是只停留在离线训练。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/tlv.c`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/nn/cnn/export2c.py`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/unpack.py`

## 3. MCU 端的数据处理流程具体是怎样的？

**参考回答：**

MCU 端主循环在 `main` 里不断调用 `Sensor_Process()`。当 ADC 完成一个 chirp 采样后，会先把原始 ADC 数据转成浮点格式，然后做直流分量去除、窗函数处理，再组织成 IQ 形式进行 Range FFT。每个 chirp 的结果会写入 `radarCube`。一帧 chirp 全部结束后，再做静态杂波抑制或 MTI，然后对每个距离 bin 做 Doppler FFT，提取最大幅值形成距离向量，最后把这个向量送入 CNN/GRU/LSTM 做分类，并通过 TLV 协议发给上位机。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/BSP/src/bsp.c#L166`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/BSP/src/bsp.c#L173`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c#L93`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Inc/defines.h#L12`

## 4. 你们为什么要做 DC 去除、窗函数、FFT 和静态杂波抑制？

**参考回答：**

这些步骤本质上是在把原始 ADC 采样数据变成更稳定、更有区分度的特征。DC 去除是为了减小直流偏置对频谱的影响；窗函数是为了抑制频谱泄漏；Range FFT 是把时域采样映射到距离维；静态杂波抑制或 MTI 是为了削弱环境中的静止反射，让真正和人体运动相关的成分更突出。这样后面的距离向量和模型输入会更干净，误检率也会更低。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/BSP/src/bsp.c#L209`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/BSP/src/bsp.c#L213`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/BSP/src/bsp.c#L224`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c#L97`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c#L117`

## 5. 你们的存在性检测是怎么做的，为什么要加这一层？

**参考回答：**

存在性检测是为了避免“无人场景也强行做分类”。代码里会先对一个参考 chirp 做幅值计算，找到最大值，再和阈值比较，把结果放进一个环形数组做短时平滑。如果连续多帧都判断没有人，就不继续做后面的分类，而是只做必要的状态更新。这样做的好处是可以减少无效推理，降低空场景噪声对结果的干扰，也能避免 RNN 类模型在无目标状态下隐藏态被错误激活。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c#L124`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c#L140`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c#L155`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/func.c#L15`

## 6. 你们为什么同时做了 CNN、GRU、LSTM 这些模型？

**参考回答：**

因为这个任务既有空间结构，也有时间连续性。CNN 更适合处理滑窗后的局部时空特征，结构简单、端侧部署也相对直接；GRU 和 LSTM 更适合捕捉时间序列上的动态变化，比如跌倒前后的连续演化过程。项目里把几种模型都做了，一方面是为了对比效果，另一方面也是为了在精度、计算复杂度和端侧资源之间找到更合适的平衡。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/gui/inference.py#L21`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/const.py#L31`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/const.py#L35`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/const.py#L60`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/nn/cnn/train.py#L16`

## 7. 你们的 CNN 输入是什么，网络结构大概长什么样？

**参考回答：**

从代码实现看，CNN 的输入不是原始 ADC 数据，而是滑窗后的距离向量序列，窗口长度是 `30`，单帧特征长度是 `38`。网络结构上是多层卷积加池化，再接全连接层输出二分类结果。这个设计的优点是输入已经经过信号处理压缩，维度相对可控，比较适合端侧部署。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/const.py#L31`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/nn/cnn/train.py#L16`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c#L229`

## 8. 模型是怎么从 Python 侧部署到 MCU 端的？

**参考回答：**

模型训练在 Python 侧完成，训练好之后会先导出成 `npz` 或类似中间参数格式，再通过脚本把卷积层、全连接层、BatchNorm 等权重转换成 C 数组，生成对应的头文件。MCU 端再把这些头文件编进工程，通过手写的推理实现完成前向计算。这个过程的关键点不是简单“保存模型”，而是把 Python 训练结果转换成嵌入式可直接编译、可直接访问的参数格式。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/nn/cnn/train.py#L118`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/nn/cnn/export2c.py#L77`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/nn/cnn/export2c.py#L89`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/cnn.c`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Inc/cnn/cnnData.h`

## 9. 你们为什么设计了 TLV 协议？这个协议解决了什么问题？

**参考回答：**

我设计 TLV 协议主要是为了兼顾可扩展性和调试便利性。因为项目里有多种数据类型，比如 IF 信号、MTI、距离向量和分类概率，不同模式下需要发的数据并不一样。TLV 的好处是每段数据都带类型和长度，接收端可以按需解析，也方便后续扩展新的数据类型。对联调来说，这种格式比裸数据流更稳定，也更容易排查丢帧和解析错误。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/tlv.c#L24`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/tlv.c#L48`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/tlv.c#L108`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/unpack.py#L72`

## 10. 串口输出这一块你是怎么做的？为什么选这个方案？

**参考回答：**

代码里是基于 `UART5` 做原始数据输出，初始化波特率设置为 `921600`。发送层做得比较直接，支持单字节、缓冲区和格式化打印三种方式。这个方案实现简单、调试方便，适合和上位机快速联调。对当前项目来说，重点不是做高吞吐协议栈，而是确保中间结果能稳定发出去，方便验证端侧算法和上位机解包逻辑。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/BSP/src/bsp.c#L115`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/driver/src/drv_rawdata.c#L33`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/driver/src/drv_rawdata.c#L75`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/driver/src/drv_rawdata.c#L140`

## 11. 上位机主要做了什么，不只是显示界面吧？

**参考回答：**

对，这里的上位机不只是简单显示。它一方面负责串口接收和 TLV 解包，把板端送上来的 IF、MTI、距离向量和概率恢复出来；另一方面也承担了离线验证和模型对比的作用，可以切换不同模型、做滑窗推理、看预测结果和标签对应关系。实际上它是算法验证和嵌入式联调之间的桥梁。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/unpack.py#L72`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/gui/inference.py#L21`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/gui/mti.py`

## 12. 这个项目里你觉得最有代表性的工程问题是什么？

**参考回答：**

我觉得最有代表性的工程问题不是单一算法，而是“怎么把离线可用的算法稳定地搬到 MCU 上”。这里面涉及到采样参数和特征维度的统一、空场景下的存在性判断、模型输入窗口和时序延迟的设计、Python 参数导出和 C 侧推理的一致性，以及串口协议和上位机联调。真正有价值的是把这些环节串起来，让系统能够持续稳定运行。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/const.py`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Inc/defines.h`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/nn/cnn/export2c.py`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c`

## 13. 你们的关键参数是怎么定的，比如 128 个采样点、32 个 chirp、256 点 FFT？

**参考回答：**

这些参数本质上是在分辨率、计算量和端侧资源之间做平衡。当前配置里 ADC 采样点是 `128`，Range FFT 是 `256` 点，chirp 数是 `32`。这样做一方面能保证距离维和速度维有基本可用的分辨率，另一方面不会让 MCU 端计算量和内存占用太高。因为项目是端侧实时推理，不是离线高精度处理，所以参数选择更偏工程折中。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Inc/defines.h#L12`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/const.py#L7`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/const.py#L15`

## 14. 你们为什么用了 Hamming 窗，而不是直接做 FFT？

**参考回答：**

直接做 FFT 会有明显频谱泄漏，尤其是在目标回波不严格落在整点频率上时，旁瓣会比较高。Hamming 窗的作用是降低旁瓣，让目标峰值更稳定，后面做距离向量提取时抗噪能力也更强。对这个项目来说，特征稳定性比极致分辨率更重要，所以用了 Hamming 窗。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/BSP/src/bsp.c#L154`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/BSP/src/bsp.c#L213`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/const.py#L61`

## 15. 静态杂波抑制和 MTI 有什么区别？你们最终用了哪个？

**参考回答：**

静态杂波抑制本质上是用当前帧内多个 chirp 的平均结果估计静态背景，再从每个 chirp 中减掉；MTI 更偏向时域高通，通过相邻 chirp 差分突出动态目标。当前工程里 `defines.h` 默认打开的是 `DYNAMIC_SCR`，也就是静态杂波抑制，而不是 `DYNAMIC_MTI`。这么做说明当前实现更偏向用静态背景消除来构造稳定输入特征。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Inc/defines.h#L8`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/BSP/src/bsp.c#L186`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c#L97`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/tlv.c#L69`

## 16. 为什么特征不是直接用整帧频谱，而是提取距离向量？

**参考回答：**

因为项目目标是端侧部署，输入维度必须受控。整帧频谱信息虽然更全，但维度大、计算重、对内存也不友好。这里做法是对每个距离 bin 经过 Doppler FFT 后取最大幅值，形成一个长度固定的距离向量，相当于把二维信息压缩成更紧凑的特征表示。这样既保留了和人体活动相关的主要信息，又更适合轻量化模型部署。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c#L207`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c#L226`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Inc/defines.h#L22`

## 17. CNN、GRU、LSTM 在输入形式上有什么区别？

**参考回答：**

CNN 用的是滑窗输入，也就是连续多帧距离向量组成一个窗口后一起送进去；GRU 和 LSTM 更像逐时刻喂入，它们保留隐藏状态，适合处理连续时间序列。代码里 CNN 的窗口长度是 `30`，而 GRU/LSTM 会结合观测延迟和左右偏置去做输出对齐。所以这几种模型虽然任务相同，但输入组织方式和时序建模逻辑是不一样的。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/gui/inference.py#L58`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/gui/inference.py#L69`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/const.py#L31`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/const.py#L35`

## 18. 你们是怎么处理 GRU/LSTM 隐藏状态的？

**参考回答：**

GRU 和 LSTM 的关键是隐藏状态管理。Python 训练阶段会显式在每个 batch 开始前重置隐藏状态，保证不同样本之间不串扰。MCU 端也考虑了隐藏状态清理问题，在无人场景或必要更新时会把隐藏状态置为干净状态，避免前一段序列对后一段序列造成错误影响。这一块如果不处理好，时序模型很容易出现“拖尾”问题。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/nn/gru/train.py`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/nn/lstm/train.py`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c#L74`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c#L83`

## 19. 你们为什么要做滑窗和左右偏置？

**参考回答：**

因为跌倒不是单帧事件，而是一个有持续时间的过程。滑窗可以把跌倒前后连续变化整合起来，提升模型判断稳定性；左右偏置则是为了让标签和模型输出更好对齐，因为真实跌倒发生点和模型窗口中心不一定重合。如果不做这一步，训练标签会和输入语义错位，模型效果通常会受影响。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/const.py#L31`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/const.py#L35`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/gui/inference.py#L15`

## 20. 训练数据是怎么组织的？类别不平衡怎么处理？

**参考回答：**

从 CNN 训练代码看，训练时会先根据标签把跌倒样本和非跌倒样本拆开，然后每个 epoch 都随机抽取数量相同的非跌倒样本和跌倒样本拼接，构造平衡数据集。这样做的目的是减轻类别不平衡对训练结果的影响，否则模型可能会更偏向输出非跌倒。这个处理虽然简单，但在工程上很有效。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/nn/cnn/train.py#L46`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/nn/cnn/train.py#L71`

## 21. 你们为什么做了工作模式切换？

**参考回答：**

这个系统不是只有一个“分类模式”。从 TLV 配置可以看出，至少包含 IF 信号模式、数据采集模式和分类模式。这样做的好处是方便分阶段调试：算法开发时可以先看原始 IF 或中间结果，采集数据时只发特征，正式运行时再发分类结果。也就是说，工作模式切换是为了把调试、采集和部署运行解耦。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/tlv.c#L24`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/tlv.c#L32`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/tlv.c#L39`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/tlv.c#L129`

## 22. TLV 帧头里为什么要带 `frame`、`delay`、`tlv_nums`、`data_len`、`nnExist`？

**参考回答：**

这些字段本质上是为了解析和联调服务。`frame` 用来识别帧序号和定位丢帧；`delay` 可以辅助看链路时延；`tlv_nums` 和 `data_len` 是为了让接收端知道后面有多少段数据、总长是多少；`nnExist` 则把存在性检测结果也一并送到上位机，方便判断当前分类结果是不是在“有人场景”下得出的。这样上位机拿到的不只是分类概率，而是完整的上下文信息。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/tlv.c#L48`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/unpack.py#L72`

## 23. 上位机怎么判断丢帧或解析错误？

**参考回答：**

上位机解包时会先读帧头里的 `data_len`，然后拿接收到的实际数据长度和它比对。如果长度不一致，就直接报“帧长度不匹配”，这通常意味着丢帧或者串口数据截断。另外如果 TLV 类型不在已知枚举范围内，也会报未知 TLV 错误。也就是说，上位机在协议层已经做了基本完整性校验。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/unpack.py#L81`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/unpack.py#L90`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/unpack.py#L115`

## 24. 你怎么验证 MCU 端推理和 Python 端结果是一致的？

**参考回答：**

这个项目的思路是先在 Python 端完成模型训练和离线前向验证，再把参数导出成 C 数组，在 MCU 端用同一份参数做前向计算。同时还会生成测试数据头文件，把同一组输入喂给 C 端模型。这样就能逐层或逐输入验证导出参数和推理结果是否一致，保证部署不是“黑盒搬运”。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/nn/cnn/export2c.py#L113`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/nn/cnn/C`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/nn/cnn/DSP_C`

## 25. 为什么项目里要单独做上位机推理线程，而不是都在主线程里跑？

**参考回答：**

因为上位机除了推理，还有串口接收、界面刷新、数据显示等工作。如果把推理也全放在主线程里，界面容易卡顿，串口接收也可能受影响。代码里用 `QThread` 单独封装了不同模型的推理线程，这样能把 UI 响应和推理解耦，联调体验会更稳定。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/gui/inference.py#L21`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/gui/inference.py#L47`

## 26. 这个项目里你做过哪些和实时性、性能相关的考虑？

**参考回答：**

主要有几类。第一是特征层面没有直接保留整帧高维数据，而是提取距离向量，降低模型输入规模；第二是加了存在性检测，在无目标场景减少无效推理；第三是使用轻量化模型而不是复杂网络；第四是整个信号处理流程依赖 `CMSIS-DSP`，尽量使用成熟的 ARM 优化库来完成 FFT 和基础向量运算。这些都是为了让系统更接近端侧实时运行。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/nn/cnn/train.py`

## 27. 这个项目里你怎么做调试和排障？

**参考回答：**

我一般分三层排查。第一层看底层链路，比如板级初始化、ADC 采样、串口输出是否正常；第二层看中间结果，比如 IF 信号、MTI、距离向量是否符合预期；第三层看模型输入输出和 Python 侧离线结果是不是一致。这个项目里 TLV 协议和上位机可视化工具的价值就在这里，它让排查不需要只靠串口日志，而是能看结构化数据。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/driver/src/drv_rawdata.c`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/tlv.c`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/unpack.py`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/gui`

## 28. 看门狗在这个项目里起什么作用？

**参考回答：**

看门狗的意义是防止系统在长时间运行中因为异常状态卡死。这个项目是持续采集和处理的在线系统，不是一次性任务，所以稳定性很重要。初始化里把看门狗拉起来，说明设计上考虑了端侧长期运行的风险控制。面试时可以把它理解成系统级可靠性措施，而不是单纯的外设初始化。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/BSP/src/bsp.c#L157`

## 29. 如果让你继续优化这个项目，你会优先做什么？

**参考回答：**

我会优先做三件事。第一，补齐更系统的精度和时延评估，把模型、特征和参数选择做成可量化对比；第二，进一步优化端侧推理和串口发送策略，降低链路开销；第三，完善异常场景处理，比如多人、边界动作、遮挡以及更复杂室内环境下的鲁棒性。如果资源允许，也会尝试更统一的量化部署方案，进一步提升端侧效率。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/const.py`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/tlv.c`

## 30. 如果面试官让你用一句话总结这个项目的价值，你怎么说？

**参考回答：**

我会说，这个项目的价值不只是做了一个跌倒检测模型，而是把毫米波雷达采集、信号处理、轻量化模型训练、参数导出、MCU 端部署和上位机联调整体打通，真正实现了一个可在端侧运行的完整跌倒检测系统。

**复习代码：**

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/app/src/main.c`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/tlv.c`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection`

## 复习顺序建议

1. 先讲清系统链路：`main.c -> bsp.c -> process.c -> tlv.c -> unpack.py`
2. 再复习信号处理：DC 去除、窗函数、Range FFT、静态杂波抑制/MTI、Doppler FFT
3. 再复习模型输入输出：距离向量怎么形成、CNN/GRU/LSTM 的输入差异
4. 最后复习部署和联调：`export2c.py`、`drv_rawdata.c`、`unpack.py`

## 你下一步最该背熟的 3 个点

- 端侧数据流从 ADC 到分类结果的完整过程
- 为什么做存在性检测和杂波抑制
- Python 训练结果怎么导出并部署到 C 端

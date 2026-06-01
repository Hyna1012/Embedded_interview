# 多普勒项目记忆索引

## 项目名称

毫米波雷达底层驱动与多普勒数据采集系统

## 一句话定位

这是一个以下位机底层驱动和采集链路为主、Python 处理验证为辅的毫米波多普勒项目：板端负责雷达 `AFE / ADC` 配置、`frame/chirp` 采样调度、`rawRadarCube` 数据组织和原始帧输出，Python 侧负责解包、二维 FFT、热图显示和距离-速度映射校验。

## 默认背景

- 目标平台：`AT32F435 / RKB2301`
- 雷达体制：`24GHz FMCW`
- 下位机入口：`main.c` 中完成 `extend_sram()`、`BSP_Init()`、`SoftSPI_Init()`、`init_tlv()`、`Sensor_Start()`，主循环调用 `Sensor_Process()`
- 采样参数：`ADC_SAMPLES = 512`，`CHIRPS = 32`，`REFRESH_RATE_MS = 50`
- 数据缓存：`rawRadarCube[CHIRPS][ADC_SAMPLES]`
- 采样流程：单 chirp 采样完成后拷贝 ADC buffer，整帧采集完成后封装并输出原始帧
- 输出链路：带 magic word、总长度、类型和长度字段的数据帧，通过 GPIO 模拟 SPI 时序输出完整原始雷达数据
- Python 验证：原始帧解包、均值去除、距离维 FFT、多普勒维 FFT、静态杂波抑制、`Range-Doppler Map` 显示
- 配套仿真：`simuDoppler.py`、`simuRangeDoppler.py` 用于验证距离、速度、IF 频率、多普勒频率与热图峰值的关系

## 你后续问我这个项目时，我默认从这些角度回答

1. 下位机启动流程
2. BSP、AFE 和 ADC 配置
3. `frame/chirp` 时序调度
4. ADC 采样完成消息处理
5. `rawRadarCube` 原始数据组织
6. GPIO 模拟 SPI 时序原始帧输出与帧格式
7. Python 原始帧解包与二维 FFT 验证
8. 距离-速度映射关系和仿真校验
9. 简历表述、面试追问和项目边界说明

## 核心材料入口

### 1. 项目经历材料

- `/D:/project/postgraduate/work/Embedded_interview/resume_materials/01_projects/02_doppler_simulator/doppler_simulator_project.md`

### 2. 面试问答材料

- `/D:/project/postgraduate/work/Embedded_interview/resume_materials/01_projects/02_doppler_simulator/doppler_simulator_interview_qa.md`

### 3. 关键源码入口

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/app/src/main.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp_config.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/doppler/Inc/defines.h`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/doppler/Src/tlv.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/driver/src/drv_soft_spi.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/driver/src/drv_rawdata.c`
- `/D:/project/postgraduate/sgr/doppler/python312_SGR2301DopplerDetection/const.py`
- `/D:/project/postgraduate/sgr/doppler/python312_SGR2301DopplerDetection/gui/frameData.py`
- `/D:/project/postgraduate/sgr/doppler/python312_SGR2301DopplerDetection/gui/widget.py`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/simu/simuDoppler.py`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/simu/simuRangeDoppler.py`

## 默认复习顺序

1. `main.c`
2. `bsp_config.c`
3. `bsp.c`
4. `tlv.c`
5. `drv_soft_spi.c`
6. `frameData.py`
7. `widget.py`
8. `simuDoppler.py`
9. `simuRangeDoppler.py`

## 默认关键词

- `24GHz FMCW`
- 多普勒数据采集
- `AT32F435`
- `RKB2301`
- BSP
- AFE
- ADC
- `frame/chirp`
- 采样完成事件
- `rawRadarCube`
- GPIO 模拟 SPI 时序
- `TLV`
- 原始雷达帧
- `Range FFT`
- `Doppler FFT`
- `Range-Doppler Map`
- 距离-速度映射验证

## 表述边界

- 优先强调“下位机底层驱动、采样时序、原始数据组织、帧封装和数据输出”，不要把项目写成以上位机界面为主。
- 简历中不要直接写 `SoftSPI`，建议写成“基于 GPIO 模拟 SPI 时序的数据输出调试”；面试展开时再解释它是软件模拟 SPI。
- 当前主线不要写“板端生成热图”“板端二维 FFT”“`UART5` 输出热图”“`2Tx3Rx TDM-MIMO` 虚拟通道补偿”，除非后续确认存在对应代码或分支。
- 上位机只作为验证侧表述：解包、二维 FFT、热图显示和仿真对照。

## 后续协作约定

- 你后面只要说“多普勒项目”或“多普勒数据采集项目”，我默认指这个项目。
- 如果你要我写简历、准备面试、优化回答、做项目介绍，我会优先使用这份更新后的下位机主线。
- 如果你补充了真实实验结论、性能指标、个人分工，我会继续在这个目录里迭代，不另起一套内容。

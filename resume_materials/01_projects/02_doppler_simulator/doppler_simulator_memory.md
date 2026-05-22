# 多普勒项目记忆索引

## 项目名称

24GHz FMCW 毫米波多普勒模拟与距离-多普勒热图验证系统

## 一句话定位

这是一个把 `24GHz FMCW` 雷达采集、板端二维 FFT 处理、`TLV` 数据传输、上位机可视化和 Python 多普勒模拟验证打通的毫米波多普勒项目。

## 默认背景

- 目标平台：`AT32F435 / RKB2301`
- 雷达体制：`24GHz FMCW`
- 天线配置：`2Tx3Rx TDM-MIMO`，形成 `6` 个虚拟通道
- 采样参数：`ADCSamples = 256`，`chirpLoops = 32`
- 工作模式：`RawData` / `RangeDopplerMap`
- 处理链路：DC 去除 + Hamming 窗 + Range FFT + 相位补偿 + 静态杂波抑制 + Doppler FFT
- 通信方式：`UART5@921600` + 自定义 `TLV`
- 上位机与工具链：`Python` `PySide6` `pyqtgraph`
- 配套脚本：`simuDoppler.py` `simuRangeDoppler.py`

## 你后续问我这个项目时，我默认从这些角度回答

1. 系统链路
2. MCU 端采样和 frame/chirp 调度
3. TDM-MIMO 和虚拟通道相位补偿
4. Range FFT / Doppler FFT / 热图生成
5. 静态杂波抑制与中间结果观察
6. TLV 协议与高速串口联调
7. 上位机解包、帧同步和热图显示
8. Python 模拟器对距离/速度映射关系的验证
9. 面试问题、口语化表达和追问应对

## 核心材料入口

### 1. 项目经历材料

- `/D:/project/postgraduate/work/Embedded_interview/resume_materials/01_projects/02_doppler_simulator/doppler_simulator_project.md`

### 2. 面试问答材料

- `/D:/project/postgraduate/work/Embedded_interview/resume_materials/01_projects/02_doppler_simulator/doppler_simulator_interview_qa.md`

### 3. 关键源码入口

- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/app/src/main.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/BSP/src/bsp_config.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/defines.h`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/process.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/tlv.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/gesture/tlv.h`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/driver/src/drv_rawdata.c`
- `/D:/project/postgraduate/sgr/doppler/C_armgcc_SGR2301dopplerDetection/Config/sensor_cfg.h`
- `/D:/project/postgraduate/sgr/doppler/python310_SGR2301DopplerDetection/gui/unpack.py`
- `/D:/project/postgraduate/sgr/doppler/python310_SGR2301DopplerDetection/gui/capThread.py`
- `/D:/project/postgraduate/sgr/doppler/python310_SGR2301DopplerDetection/gui/gui.py`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/simu/simuDoppler.py`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/simu/simuRangeDoppler.py`
- `/D:/project/postgraduate/sgr/doppler/python312_dopplerSimulator/dopplergui/main.py`

## 默认复习顺序

1. `main.c`
2. `bsp.c`
3. `process.c`
4. `tlv.c`
5. `unpack.py`
6. `capThread.py`
7. `gui.py`
8. `simuDoppler.py`
9. `simuRangeDoppler.py`

## 默认关键词

- `24GHz FMCW`
- 多普勒模拟器
- 距离-多普勒热图
- `AT32F435`
- `RKB2301`
- `TDM-MIMO`
- 虚拟通道
- 相位补偿
- `CMSIS-DSP`
- `Range FFT`
- `Doppler FFT`
- 静态杂波抑制
- `TLV`
- `UART5`
- `PySide6`
- `pyqtgraph`

## 后续协作约定

- 你后面只要说“多普勒项目”或“多普勒模拟器项目”，我默认指这个项目。
- 如果你要我写简历、准备面试、优化回答、做项目介绍，我会优先读取这份索引和对应问答材料。
- 如果你补充了真实实验结论、性能指标、个人分工，我会继续在这个目录里迭代，不另起一套内容。

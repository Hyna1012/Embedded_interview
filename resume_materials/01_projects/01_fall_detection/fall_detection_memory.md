# 跌倒项目记忆索引

## 项目名称

基于毫米波雷达与轻量化神经网络的跌倒检测系统

## 一句话定位

这是一个把毫米波雷达数据采集、信号处理、轻量化神经网络训练、参数导出、MCU 端部署和上位机联调整体打通的端侧跌倒检测项目。

## 默认背景

- 目标平台：`AT32F435 / RKB2301`
- 项目方向：嵌入式端开发 + 信号处理 + 轻量化模型部署 + 上位机联调
- 信号处理库：`CMSIS-DSP`
- 通信方式：`UART5` + 自定义 `TLV` 协议
- 上位机与工具链：`Python` `PyTorch` `PySide6`
- 模型类型：`CNN` `GRU` `LSTM` `CNN-LSTM`

## 你后续问我这个项目时，我默认从这些角度回答

1. 系统链路
2. MCU 端数据处理流程
3. 杂波抑制 / MTI / FFT 原理
4. 存在性检测设计
5. 模型输入构造与部署
6. TLV 协议与串口联调
7. 上位机解包、可视化与离线验证
8. 面试问题、口语化表达和追问应对

## 核心材料入口

### 1. 项目经历材料

- `/D:/project/postgraduate/work/Embedded_interview/resume_materials/01_projects/01_fall_detection/fall_detection_project.md`

### 2. 面试问答材料

- `/D:/project/postgraduate/work/Embedded_interview/resume_materials/01_projects/01_fall_detection/fall_detection_interview_qa.md`

### 3. 关键源码入口

- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/app/src/main.c`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/BSP/src/bsp.c`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/process.c`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Src/tlv.c`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/driver/src/drv_rawdata.c`
- `/D:/project/postgraduate/sgr/fall/C_armgcc_SGR2301FallDetection/fallen/Inc/defines.h`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/const.py`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/unpack.py`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/gui/inference.py`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/nn/cnn/train.py`
- `/D:/project/postgraduate/sgr/fall/python312_SGR2301FallDetection/nn/cnn/export2c.py`

## 默认复习顺序

1. `main.c`
2. `bsp.c`
3. `process.c`
4. `tlv.c`
5. `unpack.py`
6. `train.py`
7. `export2c.py`

## 默认关键词

- 毫米波雷达
- 跌倒检测
- 端侧推理
- CMSIS-DSP
- FFT
- 静态杂波抑制
- MTI
- 存在性检测
- 距离向量
- CNN
- GRU
- LSTM
- TLV
- UART
- 上位机联调

## 后续协作约定

- 你后面只要说“跌倒项目”，我默认指这个项目。
- 如果你要我写简历、准备面试、优化回答、做项目介绍，我会优先读取这份索引和上面的两份材料。
- 如果你补充了新的实验结果、性能指标、个人真实分工，我会继续在这个目录里迭代，不重新起一套内容。

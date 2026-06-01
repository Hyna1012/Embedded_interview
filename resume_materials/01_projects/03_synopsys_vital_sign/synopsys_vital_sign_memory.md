# 新思科技企业赛项目记忆索引

## 项目名称

基于 `60GHz` 毫米波雷达的多人生命体征探测

## 一句话定位

这是一个基于加特兰 `CAL60S244-IB` 毫米波雷达平台，实现多人目标跟踪、静止目标维持、呼吸/心跳非接触式检测和生命体征结果输出的项目。

## 默认背景

- 赛事背景：第十九届中国研究生电子设计竞赛，新思科技企业命题赛道
- 作品名称：基于 `60GHz` 毫米波雷达的多人生命体征探测
- 硬件平台：加特兰 `CAL60S244-IB` / `RDP-60S244-IB-A-AIP` 雷达开发平台
- 工具链：`MetaWare`，代码目录中包含 `ARC` 下位机工程
- 任务链路：`ARC + FreeRTOS`，帧完成队列处理、用户任务组织、多模块数据流衔接
- 下位机/算法：雷达基础处理、群组跟踪、长时间 `MTI`、目标维持、`EDACM` 相位解调、`VMD` 频谱分离
- 验证侧：目标状态、生命体征结果和波形数据解包显示
- 主要指标：最多跟踪 `5` 个目标，目标显示位置误差 `15cm` 以内；静止目标呼吸测量准确率 `95%+`，心跳测量准确率 `90%+`

## 重要口径提醒

当前简历口径：你作为项目负责人，表述为“负责系统联调和嵌入式算法实现”。回答时优先讲 `ARC + FreeRTOS` 任务链路、多人目标跟踪、静止目标维护、相位微动处理和生命体征结果输出，不要把重点放在界面展示。

另外，报告中出现“`RISC-V` 处理器”表述，而代码目录和工具链显示为 `ARC` 下位机 / `MetaWare`。正式面试前建议统一为当前简历中的 `ARC + FreeRTOS` 口径，避免被追问时前后不一致。

## 你后续问我这个项目时，我默认从这些角度回答

1. 系统整体链路
2. `60GHz FMCW` 雷达基础信号处理
3. `4D` 点云、`CFAR`、`DBF`
4. 基于扩展卡尔曼滤波的群组跟踪
5. 长时间 `MTI` 与静止目标维持
6. 生命体征微动相位提取
7. `EDACM` 相位解调与一阶线性补偿
8. `VMD` 呼吸/心跳频谱分离
9. 下位机资源受限实现
10. 目标状态、生命体征结果和波形数据输出
11. 面试问题、口语化表达和追问应对

## 核心材料入口

### 1. 项目经历材料

- `/D:/project/postgraduate/work/Embedded_interview/resume_materials/01_projects/03_synopsys_vital_sign/synopsys_vital_sign_project.md`

### 2. 面试问答材料

- `/D:/project/postgraduate/work/Embedded_interview/resume_materials/01_projects/03_synopsys_vital_sign/synopsys_vital_sign_interview_qa.md`

### 3. 原始资料入口

- `/D:/project/postgraduate/competition/企业赛 新思科技/决赛/本子/第十九届中国研究生电子设计竞赛报告.docx`
- `/D:/project/postgraduate/competition/企业赛 新思科技/省赛/网页表单.txt`
- `/D:/project/postgraduate/competition/企业赛 新思科技/决赛/PPT/答辩PPT.pptx`
- `/D:/project/postgraduate/competition/企业赛 新思科技/省赛/PPT/答辩PPT.pptx`

### 4. 关键源码入口

- `/D:/project/postgraduate/competition/企业赛 新思科技/20240703研电赛代码/ARC下位机/calterah/common/user_app/user_task.c`
- `/D:/project/postgraduate/competition/企业赛 新思科技/20240703研电赛代码/ARC下位机/calterah/common/user_app/user_task.h`
- `/D:/project/postgraduate/competition/企业赛 新思科技/20240703研电赛代码/ARC下位机/calterah/common/user_app/u_gtrack.c`
- `/D:/project/postgraduate/competition/企业赛 新思科技/20240703研电赛代码/ARC下位机/calterah/common/user_app/MTI.c`
- `/D:/project/postgraduate/competition/企业赛 新思科技/20240703研电赛代码/ARC下位机/calterah/common/user_app/vmd.c`
- `/D:/project/postgraduate/competition/企业赛 新思科技/20240703研电赛代码/ARC下位机/calterah/common/user_app/fft.c`
- `/D:/project/postgraduate/competition/企业赛 新思科技/20240703研电赛代码/ARC下位机/calterah/common/radarapp/post_bb_proc_task.c`
- `/D:/project/postgraduate/competition/企业赛 新思科技/20240703研电赛代码/python上位机/host.py`
- `/D:/project/postgraduate/competition/企业赛 新思科技/20240703研电赛代码/python上位机/unpack.py`
- `/D:/project/postgraduate/competition/企业赛 新思科技/20240703研电赛代码/python上位机/param.py`
- `/D:/project/postgraduate/competition/企业赛 新思科技/20240703研电赛代码/python上位机/capThread.py`
- `/D:/project/postgraduate/competition/企业赛 新思科技/20240703研电赛代码/python上位机/c/vmd/VMD.c`
- `/D:/project/postgraduate/competition/企业赛 新思科技/20240703研电赛代码/python上位机/util/simpleVMD.py`

## 默认复习顺序

1. `synopsys_vital_sign_project.md`
2. `synopsys_vital_sign_interview_qa.md`
3. 原始报告摘要、创新点、软件框架、测试结果
4. `user_task.c`
5. `u_gtrack.c`
6. `MTI.c`
7. `vmd.c`
8. `host.py`
9. `unpack.py`
10. `param.py`

## 默认关键词

- `60GHz` 毫米波雷达
- 多人生命体征探测
- 非接触式检测
- `FMCW`
- `CAL60S244-IB`
- `MetaWare`
- `ARC`
- `CFAR`
- `DBF`
- `4D` 点云
- 群组跟踪
- 扩展卡尔曼滤波
- 长时间 `MTI`
- 静止目标维持
- `EDACM`
- 相位解调
- 一阶线性补偿
- `VMD`
- 呼吸检测
- 心跳检测
- Python 上位机

## 后续协作约定

- 你后面只要说“新思项目”“企业赛项目”“60G 生命体征项目”，我默认指这个项目。
- 如果你确认个人分工，我会把项目材料改成与你真实贡献一致的简历版和面试版。
- 如果你要把它正式加入简历，我会优先做“与跌倒项目/多普勒项目不重复”的差异化表达。

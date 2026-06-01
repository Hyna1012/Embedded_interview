# 嵌入式秋招材料复习与优化路线

> 用途：把当前材料库变成可执行的复习计划。简历暂时不动，所有优化先沉淀在题库、项目问答和模拟面试材料里。

## 1. 当前材料分层

### 第一层：快速总览

- `resume_materials/04_mock_interviews/resume_mock_interview_100qa.md`
- `resume_materials/04_mock_interviews/project_deep_dive_optimized_qa.md`

用途：面试前快速过一遍，形成个人表达主线。

### 第二层：项目深挖

- 跌倒检测：`resume_materials/01_projects/01_fall_detection/fall_detection_interview_qa.md`
- 多普勒采集：`resume_materials/01_projects/02_doppler_simulator/doppler_simulator_interview_qa.md`
- 生命体征：`resume_materials/01_projects/03_synopsys_vital_sign/synopsys_vital_sign_interview_qa.md`
- 美的实习：`resume_materials/02_internship/01_midea/midea_internship_interview_qa.md`

用途：准备面试官围绕简历逐条追问。

### 第三层：八股基础

- `c_language_memory_qa.md`
- `c_advanced_interview_qa.md`
- `mcu_startup_exception_qa.md`
- `mcu_peripherals_deep_qa.md`
- `communication_bus_deep_qa.md`
- `driver_communication_debug_qa.md`
- `freertos_core_interview_qa.md`
- `rtos_deep_interview_qa.md`
- `cache_knowledge_and_interview_qa.md`
- `mpu_knowledge_and_interview_qa.md`
- `bootloader_flash_upgrade_qa.md`

用途：补通用基础，防止项目讲得好但八股断层。

### 第四层：大厂真题风格

- `big_company_embedded_interview_questions.md`

用途：按大厂常问组合训练，尤其是“C + RTOS + 外设 + 调试 + 项目”的连续追问。

## 2. 7 天复习路线

### Day 1：C 语言与内存

- 指针、数组、`sizeof/strlen`
- `volatile/const/static/extern`
- 结构体对齐、大小端
- 栈、堆、全局区、Flash/RAM

重点文件：

- `c_language_memory_qa.md`
- `c_advanced_interview_qa.md`

### Day 2：MCU 启动、中断、异常

- 上电到 `main()`
- 向量表、栈指针、Reset_Handler
- NVIC、中断优先级
- HardFault 排查
- SysTick、PendSV、SVC

重点文件：

- `mcu_startup_exception_qa.md`
- `mcu_peripherals_deep_qa.md`

### Day 3：FreeRTOS / RTOS

- 任务状态、调度、优先级
- 队列、信号量、互斥锁
- FromISR、任务通知
- 任务栈、临界区、优先级反转
- 结合生命体征项目讲帧完成队列

重点文件：

- `freertos_core_interview_qa.md`
- `rtos_deep_interview_qa.md`
- `rtos_interrupt_concurrency_qa.md`

### Day 4：通信和驱动调试

- UART / SPI / IIC / CAN 对比
- IIC 无 ACK 排查
- SPI 模式和时序排查
- UART 乱码、丢包、帧解析错位
- GPIO 模拟 SPI 与硬件 SPI 区别

重点文件：

- `communication_bus_deep_qa.md`
- `driver_communication_debug_qa.md`
- `communication_protocols_knowledge_and_interview_qa.md`

### Day 5：DMA / Cache / MPU / ADC

- DMA 价值和典型场景
- Cache clean/invalidate
- MPU 区域属性
- ADC 采样链路
- 雷达采样 buffer 和任务栈区别

重点文件：

- `adc_knowledge_and_interview_qa.md`
- `cache_knowledge_and_interview_qa.md`
- `mpu_knowledge_and_interview_qa.md`

### Day 6：项目深挖

- 跌倒项目：端侧识别、`CMSIS-DSP`、`TLV`、模型部署一致性
- 生命体征项目：`ARC + FreeRTOS`、队列、目标跟踪、静止目标维护、`VMD`
- 多普勒项目：`AFE / ADC`、`frame/chirp`、`rawRadarCube`、GPIO 模拟 SPI
- 美的实习：状态机、安全联锁、继电器控制、样机联调

重点文件：

- `project_deep_dive_optimized_qa.md`
- `resume_mock_interview_100qa.md`

### Day 7：大厂模拟

- 按 `big_company_embedded_interview_questions.md` 随机抽题
- 每题回答控制在 1-2 分钟
- 每个项目准备：
  - 1 个链路介绍
  - 1 个难点
  - 1 个排查案例
  - 1 个工程取舍

## 3. 回答原则

### 先结论，再链路

不要上来堆背景。先说“我负责的是哪条链路”，再说输入、处理、输出、验证。

### 讲自己做过的动作

少说“项目实现了”，多说：

- 我负责
- 我调试
- 我通过什么方式验证
- 我遇到问题会怎么排查

### 每个知识点都桥接项目

例如：

- FreeRTOS 队列 → 生命体征帧完成队列
- SPI → 多普勒 GPIO 模拟 SPI 时序
- TLV → 跌倒项目中间结果输出
- 状态机 → 美的微波炉运行/暂停/开门/故障

### 不要过度夸大

多普勒项目不要主动说板端生成热图；美的实习不要说自己独立负责整机软件；生命体征项目要围绕当前简历口径讲系统联调和嵌入式算法实现。

## 4. 面试前最后 30 分钟

1. 读一遍自我介绍。
2. 读三个项目的一句话定位。
3. 各背一个排查案例。
4. 复习 FreeRTOS：任务状态、队列、FromISR、互斥锁。
5. 复习 C：`volatile`、数组指针、结构体对齐。
6. 复习通信：IIC 无 ACK、SPI 错位、UART 乱码。

## 5. 后续待补

- 每个项目补 1 个真实 bug 案例。
- 针对目标公司 JD 定制题单。
- 把 `resume_mock_interview_100qa.md` 中通用八股继续替换成更贴简历的问题。

# 嵌入式面试复习路线 V2

> 目标：把当前材料库压成一条可执行路线。核心原则是：先建立简历主线，再补八股，再用项目把八股串起来，最后做大厂风格模拟。

## 0. 总体策略

### 你的面试主线

你不是单纯算法岗，也不是纯硬件调试岗，主线应当是：

**嵌入式软件 + 雷达感知 + 底层调试 + 工程落地**

四段经历的定位：

- **跌倒检测**：端侧识别和模型部署，讲 `CMSIS-DSP`、`TLV`、模型输入一致性。
- **生命体征**：RTOS 任务链路和嵌入式算法，讲 `ARC + FreeRTOS`、帧完成队列、目标跟踪、`VMD`。
- **多普勒采集**：底层驱动和原始数据链路，讲 `AFE / ADC`、`frame/chirp`、`rawRadarCube`、GPIO 模拟 SPI。
- **美的实习**：产品电控和工程闭环，讲状态机、安全联锁、继电器、样机联调、缺陷闭环。

### 优先级

1. **必须会**：简历项目、C 基础、FreeRTOS、UART/IIC/SPI、ADC/DMA、HardFault。
2. **应该会**：Cache/MPU、Bootloader、编译链接、CAN、Linux 入门。
3. **加分项**：项目排查案例、工程取舍、性能/资源分析。

## 1. 三天急救路线

适合突然来面试，时间很紧。

### Day 1：简历项目主线

上午：

- 背 `project_deep_dive_optimized_qa.md`
- 每个项目准备 1 分钟介绍

下午：

- 跌倒项目：`CMSIS-DSP`、`TLV`、模型部署一致性
- 生命体征：`ARC + FreeRTOS`、帧完成队列、静止目标维护

晚上：

- 多普勒：`rawRadarCube`、GPIO 模拟 SPI、帧解析错位
- 美的：状态机、门联锁、继电器控制、缺陷闭环

每日产出：

- 每个项目写出一句话定位
- 每个项目写出一个排查问题
- 每个项目写出一个“我具体做了什么”

### Day 2：高频八股

上午：C 语言

- `volatile`
- 数组和指针
- 结构体对齐
- 栈/堆/全局区
- `malloc` 风险

下午：FreeRTOS

- 任务状态
- 调度和优先级
- 队列、信号量、互斥锁
- FromISR
- 任务栈

晚上：通信和调试

- UART 乱码
- IIC 无 ACK
- SPI 错位
- DMA/Cache 一致性
- HardFault 排查

每日产出：

- 每个知识点用 30 秒讲清楚
- 每个知识点找一个项目桥接点

### Day 3：模拟面试

上午：

- 随机抽 `big_company_embedded_interview_questions.md`
- 每题 1 分钟作答

下午：

- 读 `resume_mock_interview_100qa.md`
- 重点看自我介绍、项目区别、项目难点

晚上：

- 进行一次完整模拟：
  1. 自我介绍
  2. 深挖一个项目
  3. 问 5 个八股
  4. 问一个实习问题
  5. 总结为什么录用你

## 2. 七天稳态路线

### Day 1：C 语言和内存

文件：

- `c_language_memory_qa.md`
- `c_advanced_interview_qa.md`
- `big_company_embedded_interview_questions.md`

必须掌握：

- `volatile` 不等于线程安全
- 数组名不是指针
- 结构体对齐和协议解析
- 栈溢出和大数组问题
- `malloc` 在嵌入式的风险

项目桥接：

- `rawRadarCube` 不放任务栈
- `TLV` 字段解析要注意长度、端序、对齐

### Day 2：MCU、中断、异常

文件：

- `mcu_startup_exception_qa.md`
- `mcu_peripherals_deep_qa.md`

必须掌握：

- 上电到 `main()`
- 向量表
- NVIC 优先级
- SysTick / PendSV
- HardFault 排查

项目桥接：

- FreeRTOS 任务切换和 PendSV
- ISR 里只通知任务，不做复杂处理

### Day 3：FreeRTOS

文件：

- `freertos_core_interview_qa.md`
- `rtos_deep_interview_qa.md`
- `rtos_interrupt_concurrency_qa.md`

必须掌握：

- 任务状态
- 抢占式调度
- 队列、信号量、互斥锁
- 优先级反转
- FromISR
- 任务栈

项目桥接：

- 生命体征帧完成队列
- 用户任务组织点云、跟踪、生命体征计算

### Day 4：通信和驱动

文件：

- `communication_bus_deep_qa.md`
- `driver_communication_debug_qa.md`
- `communication_protocols_knowledge_and_interview_qa.md`

必须掌握：

- UART / IIC / SPI / CAN 对比
- IIC 无 ACK 排查
- SPI CPOL/CPHA 和片选时序
- UART 不定长接收
- 帧头、长度、校验、TLV

项目桥接：

- 多普勒 GPIO 模拟 SPI
- 跌倒项目 `TLV`
- 美的输入检测和联锁信号

### Day 5：ADC / DMA / Cache / MPU

文件：

- `adc_knowledge_and_interview_qa.md`
- `cache_knowledge_and_interview_qa.md`
- `mpu_knowledge_and_interview_qa.md`

必须掌握：

- ADC 采样链路
- DMA 价值
- 双缓冲
- Cache clean/invalidate
- MPU 区域属性

项目桥接：

- 雷达 ADC 采样和 buffer
- DMA/Cache 导致旧数据
- 大 buffer 静态分配

### Day 6：项目深挖

文件：

- `project_deep_dive_optimized_qa.md`
- 三个项目各自的 `*_interview_qa.md`
- `midea_internship_interview_qa.md`

必须准备：

- 每个项目 1 分钟介绍
- 每个项目 1 个难点
- 每个项目 1 个排查案例
- 每个项目 1 个工程取舍

### Day 7：大厂模拟

文件：

- `big_company_embedded_interview_questions.md`
- `resume_mock_interview_100qa.md`

训练方式：

- C 语言抽 5 题
- RTOS 抽 5 题
- 通信抽 5 题
- 项目深挖抽 5 题
- 每题先 30 秒回答，再扩展到 2 分钟

## 3. 十四天强化路线

### 第 1 周：基础 + 项目

- Day 1：C 语言
- Day 2：MCU / 中断
- Day 3：FreeRTOS
- Day 4：通信
- Day 5：ADC / DMA / Cache
- Day 6：四段经历深挖
- Day 7：第一次全真模拟

### 第 2 周：补短板 + 大厂风格

- Day 8：复盘模拟面试卡住的问题
- Day 9：HardFault / 调试专题
- Day 10：Bootloader / Flash / 编译链接
- Day 11：项目排查案例专项
- Day 12：大厂综合题随机抽问
- Day 13：二面表达：工程取舍、稳定性、复盘
- Day 14：最终模拟 + 面试前 30 分钟速背

## 4. 每日固定动作

每天至少完成：

1. 10 分钟自我介绍和项目主线。
2. 30 分钟八股。
3. 30 分钟项目深挖。
4. 20 分钟随机抽问。
5. 10 分钟记录卡住的问题。

## 5. 面试前 30 分钟速背

### 自我介绍一句主线

我主要围绕嵌入式软件和毫米波雷达应用做项目，既做过板端采集、信号处理和模型部署，也做过 `FreeRTOS` 任务链路、底层采集驱动和真实产品电控联调。

### 四段经历一句话

- 跌倒：端侧跌倒识别，重点是 `CMSIS-DSP + 模型部署 + TLV`。
- 生命体征：多人跟踪和呼吸心跳检测，重点是 `ARC + FreeRTOS + VMD`。
- 多普勒：底层驱动和采集链路，重点是 `AFE / ADC + rawRadarCube + GPIO 模拟 SPI`。
- 美的：产品电控实习，重点是状态机、安全联锁、继电器和缺陷闭环。

### 八股最后看

- `volatile`
- 数组和指针
- 结构体对齐
- FreeRTOS 任务状态
- 队列 / 信号量 / 互斥锁
- FromISR
- IIC 无 ACK
- SPI 错位
- UART 乱码
- HardFault

## 6. 卡住时的回答模板

### 不确定细节时

“这个细节我不硬编。我当时主要负责的是……如果让我排查，我会从……几个层次去看。”

### 被问到没做过的点时

“这个点我没有完整独立做过，但我能结合项目理解它的使用场景。比如……”

### 被追问项目真实性时

“我可以按输入、处理、输出、验证来讲我负责的链路。输入是……处理是……输出是……验证方式是……”

## 7. 当前最优复习入口

1. `resume_materials/04_mock_interviews/project_deep_dive_optimized_qa.md`
2. `interview_tracking/02_question_banks/02_bagu/freertos_core_interview_qa.md`
3. `interview_tracking/02_question_banks/02_bagu/big_company_embedded_interview_questions.md`
4. `resume_materials/04_mock_interviews/resume_mock_interview_100qa.md`

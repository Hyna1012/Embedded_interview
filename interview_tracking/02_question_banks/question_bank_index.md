# Question Bank 索引

这份索引用来说明 `question_bank` 目录下的结构和使用方式。

## 目录说明

### 1. 面试细节

路径：`01_interview_details/`

- `embedded_interview_details_question_bank.md`
  面试细节题库
- `embedded_interview_details_detailed_qa.md`
  面试细节详细回答

这里主要放：
- 项目深挖
- 实习细节
- 算法落地细节
- 行为面

### 2. 八股

路径：`02_bagu/`

- `embedded_bagu_question_bank.md`
  八股题库
- `embedded_bagu_detailed_qa.md`
  八股详细回答
- `adc_knowledge_and_interview_qa.md`
  ADC 专题整理与面试问答
- `mpu_knowledge_and_interview_qa.md`
  MPU 专题整理与面试问答
- `cache_knowledge_and_interview_qa.md`
  Cache 专题整理与面试问答
- `communication_protocols_knowledge_and_interview_qa.md`
  通信协议专题整理与面试问答
- `c_language_memory_qa.md`
  C 语言与内存高频问答
- `mcu_startup_exception_qa.md`
  MCU 启动、异常与基础外设问答
- `rtos_interrupt_concurrency_qa.md`
  RTOS、中断与并发高频问答
- `driver_communication_debug_qa.md`
  驱动、通信与调试高频问答
- `bootloader_flash_upgrade_qa.md`
  Flash、Bootloader 与升级高频问答
- `embedded_linux_intro_qa.md`
  嵌入式 Linux 入门高频问答
- `bagu_project_bridge_answers.md`
  八股到项目表达桥接模板
- `compiler_linker_build_qa.md`
  编译、链接与工程构建高频问答
- `mcu_peripherals_deep_qa.md`
  MCU 外设与板级调试深挖问答
- `rtos_deep_interview_qa.md`
  RTOS 深挖面试问答
- `freertos_core_interview_qa.md`
  FreeRTOS 核心知识点与项目化面试问答，重点覆盖任务状态、调度、队列、信号量、互斥锁、FromISR、中断优先级、任务栈和项目表达。
- `communication_bus_deep_qa.md`
  通信总线深挖面试问答
- `c_advanced_interview_qa.md`
  C 语言进阶与嵌入式代码质量问答
- `big_company_embedded_interview_questions.md`
  大厂嵌入式软件真题方向整理，按 C 语言、RTOS、MCU、中断、通信、DMA/Cache、调试和项目深挖分类。
- `embedded_review_route_v2.md`
  嵌入式面试 3/7/14 天复习路线，按简历主线、八股基础、项目深挖和大厂模拟组织。

这里主要放：
- RTOS / RT-Thread
- 中断 / DMA / Cache / MPU
- SPI / IIC / UART / 驱动 bring-up
- C 语言基础
- 网络基础
- ADC 专题
- MPU 专题
- Cache 专题
- 通信协议专题
- MCU 启动 / 异常 / Bootloader
- 嵌入式 Linux 入门
- 八股到项目表达桥接
- 编译 / 链接 / 构建 / map 文件
- GPIO / Timer / PWM / ADC / DMA / 板级调试
- UART / IIC / SPI / CAN / RS-485 深挖
- C 语言代码质量与安全边界

## 推荐使用方式

1. 要准备项目和实习追问，优先看 `01_interview_details/`
2. 要集中背八股，优先看 `02_bagu/`
3. 要专门补 ADC，直接看 `adc_knowledge_and_interview_qa.md`
4. 要专门补 MPU，直接看 `mpu_knowledge_and_interview_qa.md`
5. 要专门补 Cache，直接看 `cache_knowledge_and_interview_qa.md`
6. 要专门补通信协议，直接看 `communication_protocols_knowledge_and_interview_qa.md`
7. 要补 C 语言、指针和内存，直接看 `c_language_memory_qa.md`
8. 要补 MCU 启动和异常，直接看 `mcu_startup_exception_qa.md`
9. 要补 RTOS 和并发，直接看 `rtos_interrupt_concurrency_qa.md`
10. 要补驱动调试和串口协议，直接看 `driver_communication_debug_qa.md`
11. 要补 Bootloader 和升级，直接看 `bootloader_flash_upgrade_qa.md`
12. 要补 Linux 入门，直接看 `embedded_linux_intro_qa.md`
13. 要把八股说到项目上，直接看 `bagu_project_bridge_answers.md`
14. 要补编译、链接、构建，直接看 `compiler_linker_build_qa.md`
15. 要补 MCU 外设和板级调试，直接看 `mcu_peripherals_deep_qa.md`
16. 要深挖 RTOS 面试，直接看 `rtos_deep_interview_qa.md`
17. 要深挖通信总线，直接看 `communication_bus_deep_qa.md`
18. 要补 C 语言进阶和代码质量，直接看 `c_advanced_interview_qa.md`
19. 要专门补 FreeRTOS，直接看 `freertos_core_interview_qa.md`
20. 要按大厂真题风格刷题，直接看 `big_company_embedded_interview_questions.md`
21. 要按 3/7/14 天路线复习，直接看 `embedded_review_route_v2.md`

## 建议复习路径

### 路线 A：嵌入式一面高频

1. `c_language_memory_qa.md`
2. `c_advanced_interview_qa.md`
3. `driver_communication_debug_qa.md`
4. `communication_bus_deep_qa.md`
5. `mcu_startup_exception_qa.md`

### 路线 B：项目深挖前置基础

1. `mcu_peripherals_deep_qa.md`
2. `adc_knowledge_and_interview_qa.md`
3. `communication_protocols_knowledge_and_interview_qa.md`
4. `driver_communication_debug_qa.md`
5. `bagu_project_bridge_answers.md`

### 路线 C：二面/主管面加分

1. `rtos_interrupt_concurrency_qa.md`
2. `rtos_deep_interview_qa.md`
3. `compiler_linker_build_qa.md`
4. `bootloader_flash_upgrade_qa.md`
5. `embedded_linux_intro_qa.md`

### 路线 D：FreeRTOS + 大厂综合突击

1. `freertos_core_interview_qa.md`
2. `big_company_embedded_interview_questions.md`
3. `driver_communication_debug_qa.md`
4. `project_deep_dive_optimized_qa.md`

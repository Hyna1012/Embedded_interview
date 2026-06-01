# 嵌入式八股题库

> 这份题库只保留通用八股问题，方便你单独背概念，不和项目细节混在一起。

## 1. RTOS / RT-Thread

1. 为什么项目里需要 RTOS，而不是裸机？
2. 为什么选择 `RT-Thread`，而不是 `FreeRTOS`？
3. RT-Thread 调度器的核心机制是什么？
4. 线程间通信常见方式有哪些？
5. 为什么线程间通信不推荐直接共享大量全局变量？
6. 信号量、互斥锁、事件、邮箱、消息队列分别适合什么场景？
7. 什么是优先级反转？怎么缓解？
8. `PendSV` 在任务切换里起什么作用？
9. `MSP` 和 `PSP` 分别做什么？
10. 什么叫实时操作系统的“实时性”？

## 2. 中断 / DMA / Cache / MPU

1. 中断触发的完整过程是什么？
2. 中断服务函数里为什么不建议做太重的工作？
3. DMA 的根本价值是什么？
4. 为什么很多串口接收方案会用“DMA + 空闲中断”？
5. DMA 搬完数据后，CPU 为什么可能还读到旧数据？
6. 什么是双缓冲或 ping-pong buffer？
7. `Clean` 和 `Invalidate` 分别什么时候用？
8. `MPU` 是什么？
9. `I-Cache` 和 `D-Cache` 的区别是什么？
10. 为什么 DMA 场景里一定要考虑 Cache 一致性？

## 3. 驱动 / 接口

1. 如果给你一块新板子，你怎么 bring-up？
2. 如果读回来的寄存器值不对，你怎么排查？
3. `SPI` 和 `IIC` 的区别是什么？
4. `IIC` 的起始、停止、`ACK/NACK` 分别是什么？
5. 硬件 `IIC` 和软件模拟 `IIC` 的区别是什么？
6. `UART` 波特率 `115200` 发 1 字节为什么不能只按 8 位算？
7. 串口空闲中断为什么适合做不定长接收？
8. `CLI` 控制台是什么？是不是类似 Shell？
9. `UART / SPI / IIC / CAN / RS-485` 你分别会怎么选？
10. `TCP / UDP / MQTT` 分别在什么层，各自适合什么场景？

## 4. C 语言

1. `sizeof` 和 `strlen` 的区别是什么？
2. 数组名和指针的区别是什么？
3. 为什么说数组名会退化成指针，但数组本身不是指针？
4. `volatile` 的作用是什么？
5. `malloc` 为什么可能带来内存碎片？
6. `static` 在 C 里常见的两层含义是什么？
7. 为什么嵌入式里要谨慎使用动态内存？

## 5. 网络

1. `TCP` 和 `UDP` 的区别是什么？
2. 为什么说 TCP 更可靠，但不一定更适合所有场景？
3. TCP 为什么要三次握手？
4. TCP 为什么要四次挥手？
5. 打开一个网页大概经历什么网络流程？
6. 阻塞和非阻塞 I/O 的区别是什么？

## 6. 推荐优先级

如果时间有限，优先准备这些八股：

1. 为什么选择 `RT-Thread`
2. RT-Thread 调度器核心机制
3. DMA 的根本价值
4. 串口空闲中断 + DMA
5. Cache 一致性问题
6. `SPI / IIC` 区别
7. `volatile`
8. `sizeof / strlen`
9. `TCP / UDP`

## 7. 专题补充入口

### A. C 语言

1. `static / const / volatile`
2. 数组和指针
3. 栈、堆、全局区
4. 内存对齐和结构体填充
5. 大小端
6. 宏函数和 `inline`
7. 函数指针和回调
8. 缓冲区溢出和长度检查

对应文件：
- `c_language_memory_qa.md`
- `c_advanced_interview_qa.md`

### B. MCU 与板级基础

1. MCU 上电启动流程
2. 向量表和 `Reset_Handler`
3. `HardFault` 定位
4. 时钟树
5. GPIO / EXTI / Timer / PWM
6. ADC / DMA / 双缓冲
7. 看门狗和复位原因
8. 板子无反应或偶发跑飞排查

对应文件：
- `mcu_startup_exception_qa.md`
- `mcu_peripherals_deep_qa.md`

### C. RTOS 与并发

1. 任务状态
2. 抢占式调度
3. 上下文切换
4. `SysTick / PendSV`
5. 信号量、互斥锁、事件、队列
6. 临界区和原子性
7. 优先级反转
8. 任务栈和实时性

对应文件：
- `rtos_interrupt_concurrency_qa.md`
- `rtos_deep_interview_qa.md`
- `freertos_core_interview_qa.md`

### D. 通信与驱动

1. 外设驱动编写步骤
2. 读不到外设 ID 排查
3. `UART / IIC / SPI / CAN / RS-485`
4. 串口 DMA、空闲中断、环形缓冲
5. 帧头、长度、校验、`TLV`
6. 协议状态机
7. 示波器和逻辑分析仪调试

对应文件：
- `driver_communication_debug_qa.md`
- `communication_bus_deep_qa.md`
- `communication_protocols_knowledge_and_interview_qa.md`

### E. 工程化与系统

1. 编译、链接、启动文件
2. 链接脚本和 `map` 文件
3. Flash 写入和参数保存
4. Bootloader / IAP / 固件升级
5. 嵌入式 Linux 基础
6. 八股到项目表达桥接

对应文件：
- `compiler_linker_build_qa.md`
- `bootloader_flash_upgrade_qa.md`
- `embedded_linux_intro_qa.md`
- `bagu_project_bridge_answers.md`

## 8. FreeRTOS 高频新增题

1. FreeRTOS 任务有哪些状态？`Blocked` 和 `Suspended` 有什么区别？
2. FreeRTOS 的调度机制是什么？高优先级任务为什么不能一直运行？
3. `vTaskDelay()` 和 `vTaskDelayUntil()` 有什么区别？
4. 队列、信号量、互斥锁分别适合什么场景？
5. 什么是优先级反转？互斥锁为什么能缓解它？
6. 中断里为什么要用 `FromISR` API？
7. `configMAX_SYSCALL_INTERRUPT_PRIORITY` 有什么作用？
8. 任务栈溢出怎么排查？为什么大数组不适合放任务栈？
9. 软件定时器回调里为什么不适合做耗时操作？
10. 项目里为什么要用帧完成队列？

对应文件：
- `freertos_core_interview_qa.md`

## 9. 大厂嵌入式综合新增题

1. `volatile` 的作用是什么？为什么它不等于线程安全？
2. 数组名和指针有什么区别？
3. 结构体对齐会影响什么？协议解析为什么要注意对齐和端序？
4. MCU 上电到 `main()` 前发生了什么？
5. HardFault 怎么排查？
6. IIC 无 ACK 怎么查？
7. SPI 数据错位怎么查？
8. UART 乱码有哪些原因？
9. DMA 和 Cache 为什么会有一致性问题？
10. 如果系统偶现卡死，你怎么定位是真死机还是某个任务饿死？

对应文件：
- `big_company_embedded_interview_questions.md`

## 10. 复习路线入口

如果不知道先看哪份材料，优先按路线执行：

- `embedded_review_route_v2.md`

推荐顺序：

1. 先看 3 天急救路线，建立主线。
2. 再看 7 天稳态路线，补齐八股。
3. 最后看 14 天强化路线，做大厂风格模拟。

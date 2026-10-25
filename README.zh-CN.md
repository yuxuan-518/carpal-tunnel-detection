# 腕管综合征相关姿态提醒探索——双传感器反馈原型

[English](README.md) | **简体中文**

这是 **Shaoxuan Xiong（熊少轩）与 Alex Wang** 在 Cooper Union Summer STEM 期间合作开发的嵌入式项目。我们使用 Arduino Uno 和两颗 MPU6050 传感器，探索通过比较传感器数据，为姿态变化提供即时反馈的方法。

本项目是用于姿态提醒的教学原型。仓库名称体现了最初的研究动机；当前实现不具备诊断腕管综合征的能力，也未建立经过医学验证的姿态阈值。

## 当前实现的功能

- 通过同一条 I²C 总线与两颗 MPU6050 通信，分别使用 `0x68` 和 `0x69` 地址。
- 大约每 250 毫秒读取两颗传感器的三轴加速度，并将原始读数转换为以 g 为单位的数据。
- 构造两个向量，计算它们的点积，作为比较两组传感器数据的简单指标。
- 当点积低于设定阈值 `0.65` 时，触发 LED 交替闪烁；否则关闭 LED 提示。
- 基于 Arduino Uno V3 的 ATmega328P，使用 AVR C 和底层外设、寄存器控制实现。

当前[主程序](driver/main.c)使用的是**未经归一化的加速度向量点积**，并未调用向量库中的完整夹角计算函数。这里的阈值是原型参数，不是角度值；运动和加速度大小也会影响计算结果。

## 项目开发过程

**双 MPU6050 的设计由熊少轩提出，并由他完成初期代码编写。** 在 2025 年 8 月 7 日最早的源代码提交中，他添加了 `src/mpu6050_dual.cpp`，包含针对两颗传感器的初始化与初期陀螺仪读取框架，以及轮询式 I²C 通信和串口输出代码。当时第二颗传感器的读取段仍处于注释状态。[查看早期实现](https://github.com/yuxuan-518/carpal-tunnel-detection/commit/0cbff6d4e49c6801180622b6def0209ec3a31d34)。

后续开发探索了数值积分、向量计算、FIFO 采样、串口调试以及 LED／蜂鸣器反馈。Alex 随后开发了中断式通信方案，完善传感器采样与配置，并完成最终主程序的整合：直接读取两颗传感器的加速度数据，通过 LED 提供反馈。

仓库同时保留了这些探索代码与最终主程序。数值积分、完整夹角计算和独立蜂鸣器模块属于**项目开发过程中实现并保留的模块**，并非全部在当前主循环中启用。

## 团队分工

### Shaoxuan Xiong（熊少轩）— [yuxuan-518](https://github.com/yuxuan-518)

Git 历史中也使用 `uxuan` 作为提交者名称。

- 提出双传感器架构，编写初期双 MPU6050 初始化与数据读取框架，以及早期轮询式 I²C 通信代码。
- 开发向量计算功能，包括点积、叉积、归一化和夹角、角度转换。最终主程序使用了其中的向量构造与点积函数。[向量模块提交](https://github.com/yuxuan-518/carpal-tunnel-detection/commit/a0e96cf233082244ba05169ead734a1e09fcf422)。
- 探索数值积分与数据缓存，并实现基于累计值、采样次数和经过时间的替代处理方案，以避免存储动态数组。[替代处理方案](https://github.com/yuxuan-518/carpal-tunnel-detection/commit/f88e261ad1d1768baa6e8215beae1dc71274587b)。
- 编写早期 USART 串口输出代码，开发 LED 与蜂鸣器控制模块。[反馈模块提交](https://github.com/yuxuan-518/carpal-tunnel-detection/commit/482a4290facfca65b753df9c18612367aa7e124b)。
- 参与系统整合，在中间版本的主程序中接入串口输出、向量计算与反馈模块。[整合提交](https://github.com/yuxuan-518/carpal-tunnel-detection/commit/b8c17474353295f06978cb4d959a71e2fabf1761)。

### Alex Wang — [alex-wanger](https://github.com/alex-wanger)

- 开发最终主程序使用的中断式 TWI／I²C 驱动，包括事务队列与中断处理。
- 完善 MPU6050 配置、FIFO 采样与复位、双传感器通信及计时工具。
- 整理代码目录，合并双方修改并解决整合冲突。
- 整合最终主程序，实现直接读取加速度、点积阈值判断和 LED 交替闪烁反馈。[最终整合提交](https://github.com/yuxuan-518/carpal-tunnel-detection/commit/79a594083f1b6401bd64179db7f6b8f098c2615f)。

双方在开发过程中存在交叉修改与协作。本仓库保留了来自 [Alex 原始仓库](https://github.com/alex-wanger/carpal-tunnel-detection)的共同提交历史，包括两位成员的工作。

## 代码导航

| 文件或目录 | 用途 |
| --- | --- |
| [`driver/main.c`](driver/main.c) | 最终主循环：双传感器读取、向量比较与 LED 反馈 |
| [`driver/twi.c`](driver/twi.c)、[`include/twi.h`](include/twi.h) | 中断式 I²C／TWI 通信 |
| [`include/config.h`](include/config.h) | 传感器地址、比例常量及配置事务 |
| [`driver/vector.c`](driver/vector.c)、[`include/vector.h`](include/vector.h) | 向量计算 |
| [`driver/timer.c`](driver/timer.c) | 计时支持 |
| [`driver/usart.c`](driver/usart.c) | 串口输出工具 |
| `driver/FIFO.c`、`driver/cal.c`、`driver/calcal.c`、`driver/simpson.c` | 采样与数据处理探索 |
| `driver/led.c`、`driver/buzzer.c` | 开发过程中的独立反馈模块 |

## 开源许可

采用 [GNU 通用公共许可证 v3.0](LICENSE)。

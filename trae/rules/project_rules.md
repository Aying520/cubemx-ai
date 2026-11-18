# Trae 项目规则（STM32 / C）

放到项目根的 `.trae/rules/project_rules.md`。

## 环境

- MCU：STM32F103 / STM32F407，HAL 库 + CubeMX 生成
- 工具链：Keil MDK 或 CMake + arm-none-eabi-gcc
- `Core/Src`、`Core/Inc` 会被 CubeMX 重新生成，**不要手改**
- 自定义代码写进 `USER CODE BEGIN/END` 段，或单独建 `App/` 目录

## 配置改动

- 改引脚 / 时钟 / 外设，**一律走 CubeMX CLI 或 MCP 工具**，不要直接改 `.ioc` 文本
- 每次写之前先 `dryRun: true`，把结果贴给我确认
- 写之前确认 CubeMX GUI 没开着
- 一次只改一类东西（引脚、时钟、外设分开提），方便回滚

## 代码风格

- 缩进 4 空格，大括号换行
- 变量：模块内 `static`，全局加模块前缀（`uart_rx_buf`）
- 类型：固定宽度 `uint8_t` / `uint16_t` / `uint32_t`，不用 `int`
- 宏和常量全大写，`#define` 后面写单位（`TIMEOUT_MS`、`PRESCALER_DIV`）
- 每个 `.c` 配一个同名 `.h`，头文件里只放声明，不定义变量
- 所有 `extern` 声明集中在头文件，不在 `.c` 里散落

## 中断与实时性

- **中断里禁止** `HAL_Delay`、`printf`、浮点运算、动态内存、长循环
- 中断只置标志位 / 塞队列，业务放主循环处理
- 共享变量加 `volatile`，跨中断和主循环读写时用临界区（`__disable_irq()` / `__enable_irq()`）
- 中断优先级必须显式配，别全用默认值

## HAL 使用约定

- 函数返回值必须检查，不要裸调 `HAL_UART_Transmit`
- 超时用宏，不写魔法数字
- 串口接收优先 DMA + IDLE 中断，不用轮询死等
- `HAL_Init()` 后必须调 `SystemClock_Config()`
- 外设初始化顺序：`MX_GPIO_Init()` → 通信 → 定时器

## 你交付代码时

- 只给完整函数，不给 `...略`
- 改哪个文件、改哪一段，先说清楚
- 涉及寄存器 / 时钟计算，把公式和数值都写出来
- 不确定的 HAL 版本行为，直接说不确定，不要编

## 编译

- 改完先编译，报错自己先读一遍再改
- 警告不能留着，尤其是 `unused variable` 和类型转换

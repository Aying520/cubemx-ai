# stm32-cubemx 技能说明

当任务是「改 STM32 工程配置 / 新外设 / 改时钟 / 生成 HAL 工程」时，**优先调工具，不要凭记忆猜寄存器值或 `.ioc` 文本**。

## 固定顺序

1. `stm32_cubemx_inspect(iocPath)` —— 先看清楚 MCU 型号、当前引脚、可选功能
2. `stm32_cubemx_intent(iocPath, dryRun: true, intents: [...])` —— 把需求写成意图行，干跑
3. 把 dryRun 结果里的改动差异展示给用户，**等用户点头**
4. 去掉 `dryRun` 真写（自动留 `.bak-<时间戳>`）
5. `stm32_cubemx_generate(iocPath)` 生成工程
6. 失败就用 `stm32_cubemx_rollback` 退回，不要手工修补 `.ioc`

## 意图写法

```text
USART2 115200 TX=PA2 RX=PA3
TIM3 PWM CH1=PA6 1kHz duty=25
ADC1 IN0=PA0
CAN1 TX=PA12 RX=PA11
CLOCK max
NVIC TIM3_IRQn priority=2
PB12 输入 上拉 label=KEY1
```

一次可以把多条意图放一个数组里，但**同类改动不要一次束太多**，留回滚粒度。

## 禁止

- 引导用户开 CubeMX GUI 改同一个工程（会覆盖 CLI 改动，工具也会拒写）
- 不改 ADC 采样时间（CLI 落盘为 `null`，让用户去 GUI 改）
- 不碰 `.ioc2`（CubeMX2 不支持）
- 不猜引脚可用性，用 `inspect` 查
- 不在没备份的情况下批量写

## 配完之后的活

生成完成后切回 Trae 写业务逻辑，遵守 `trae/rules/project_rules.md`：

- 回调里只置标志位
- 共享变量加 `volatile`
- 超时用宏
- 不手改 `Core/` 下 CubeMX 生成的文件

# cubemx-ai

STM32CubeMX + Trae + DeepSeek Harness 的 AI 编码工作流配置库。

目标只有一个：**让 AI 直接改 `.ioc`、生成 HAL 工程，然后在 Trae 里写业务代码**，整个过程不打开 CubeMX GUI。

## 目录

| 路径 | 内容 |
|---|---|
| `mcp/mcp-config.json` | Trae / Cursor / Cline 等 MCP 客户端的接入模板 |
| `mcp/README.md` | 三种接入方式、环境变量、排错清单 |
| `trae/rules/project_rules.md` | Trae 项目规则（STM32 HAL / C 语言专用） |
| `trae/skills/stm32-cubemx.md` | 给智能体的技能说明，让它优先调工具而不是猜配置 |
| `.gitignore` | 忽略生成的工程与 `.bak` 备份 |

## 引擎仓库（已 fork 到本账号）

本仓库只放配置，真正的引擎在 fork 过来的仓库里：

- 本账号 fork：**[Aying520/stm32-cubemx-agent](https://github.com/Aying520/stm32-cubemx-agent)**
- 上游原仓：[zhihuishuaiqi66/stm32-cubemx-agent](https://github.com/zhihuishuaiqi66/stm32-cubemx-agent)（MIT）

同步上游更新：

```bash
git remote add upstream https://github.com/zhihuishuaiqi66/stm32-cubemx-agent.git
git fetch upstream
git merge upstream/main
```

## 工作流

```
需求 → AI 一句话配外设 → CubeMX CLI 改 .ioc → 回读校验 → 生成 HAL 工程
     → Trae 里写回调 / 业务逻辑 → 编译下载
```

1. **查现状**：`stm32_cubemx_inspect` 看 MCU、引脚、可选功能
2. **先试跑**：`stm32_cubemx_intent` 带 `dryRun: true`，确认会改成什么
3. **真写**：去掉 `dryRun` 执行，自动留 `.bak-<时间戳>`
4. **生成**：`stm32_cubemx_generate` 出 HAL 工程（可选 CMake）
5. **出问题**：`stm32_cubemx_rollback` 一条命令退回

## 能说的人话

```text
USART2 115200 TX=PA2 RX=PA3
I2C1 SCL=PB6 SDA=PB7
SPI1 SCK=PA5 MISO=PA6 MOSI=PA7
TIM3 PWM CH1=PA6 1kHz duty=25
ADC1 IN0=PA0
CAN1 TX=PA12 RX=PA11
DAC OUT1=PA4 OUT2=PA5
USB 设备
DMA USART1_TX direction=P2M priority=HIGH
FSMC NAND 8bit
CLOCK max
NVIC TIM3_IRQn priority=2
PB12 输入 上拉 label=KEY1
```

引脚可以省略，会自动从封装里挑并标注 `auto-selected`；中文也认（`串口2`、`定时器3`、`上拉`）。

## 环境要求

- STM32CubeMX **6.x**（自带 JRE，不用另装 JDK）
- 对应系列的**固件包**（CubeMX 里下载，如 F1 需 `STM32Cube_FW_F1_V1.8.7`）
- **Node.js ≥ 18**
- Windows 为主要验证平台（CubeMX 命令行只有 Windows / Linux）

## 必读注意

1. **CubeMX GUI 别同时开着** —— 它会覆盖 CLI 改动，工具检测到窗口会直接拒写
2. **写操作一律先 dryRun** —— `.ioc` 写错很难肉眼发现
3. **ADC 采样时间写不进去**（CLI 落盘为 `null`），请在 GUI 改或用默认值
4. **F4 的 ADC 预分频** CLI 不支持，启用 ADC 要另行处理
5. **只实测 F1 全量、F4 部分** —— 其他系列先装固件包再试
6. 工具调用单次耗时 **10–60s**，客户端超时建议 **≥ 120s**
7. `.ioc2`（CubeMX2）不支持

## 其他参考

- 同作者的 MCP/CLI 精简包：[zhihuishuaiqi66/stm32-cubemx](https://github.com/zhihuishuaiqi66/stm32-cubemx)
- 不装 CubeMX 也能跑的备选：[hyl64/mxmcp](https://github.com/hyl64/mxmcp)
- Trae 规则参考：[MorningStar0709/trae-agent-enhancements](https://github.com/MorningStar0709/trae-agent-enhancements)、[Cooper-X-Oak/trae-agent-rules](https://github.com/Cooper-X-Oak/trae-agent-rules)

> 本仓库**只放配置、规则和笔记**，不收录第三方源码。CubeMX、HAL、固件包版权归 STMicroelectronics，使用需遵守 ST 的许可协议。

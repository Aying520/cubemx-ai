# 接入方式

三种，同一套 10 个工具，行为一致：

| 方式 | 适合 | 入口 |
|---|---|---|
| DeepSeek Harness 插件 | DSH 桌面 | `plugins/dsh-stm32-cubemx/` |
| MCP stdio server | Trae / Cursor / Cline / Claude Code | `plugins/stm32-cubemx-mcp/server.mjs` |
| CLI | 脚本、不想配 MCP | `plugins/stm32-cubemx-mcp/cli.mjs` |

## 1. Trae（MCP）

把 `mcp-config.json` 里的 `mcpServers` 段合并进 Trae 的 MCP 配置，改两个地方：

- `args[0]` → 你自己 clone 下来的 `server.mjs` 绝对路径
- `CUBEMX_HOME` → 你的 CubeMX 安装目录

然后重启 Trae，确认 MCP 面板里 `stm32-cubemx` 是已加载状态。

## 2. DeepSeek Harness（插件）

```text
plugin_manager(action="install_bundle", target="<仓库>/plugins/dsh-stm32-cubemx")
```

再改 `plugins/dsh-stm32-cubemx/cordis.patch.yml`：

```yaml
mxHome: 'C:\Program Files\STMicroelectronics\STM32Cube\STM32CubeMX'
```

## 3. CLI（最稳）

```bash
node plugins/stm32-cubemx-mcp/cli.mjs inspect   --ioc "工程.ioc"
node plugins/stm32-cubemx-mcp/cli.mjs intent    --ioc "工程.ioc" --dry-run --intent "USART2 115200 TX=PA2 RX=PA3"
node plugins/stm32-cubemx-mcp/cli.mjs generate  --ioc "工程.ioc" --out "生成目录"
node plugins/stm32-cubemx-mcp/cli.mjs rollback  --ioc "工程.ioc" --action restore
```

退出码：`0` 成功 / `1` 工具失败 / `2` 用法错误。路径带空格要加引号。

## 10 个工具

| 工具 | 作用 |
|---|---|
| `stm32_cubemx_intent` | 一句话配外设（推荐入口） |
| `stm32_cubemx_inspect` | 只读：MCU、引脚、可选功能 |
| `stm32_cubemx_configure` | 改引脚 signal / label / pull，改外设 mode |
| `stm32_cubemx_clock` | 时钟树，`preset:"max"` 或显式参数 |
| `stm32_cubemx_new_project` | 新建工程（CMake / Advanced 结构 / 最大 HCLK） |
| `stm32_cubemx_nvic` | 中断开关与优先级 |
| `stm32_cubemx_dma` | 查 DMA 请求与流 |
| `stm32_cubemx_generate` | 生成 HAL 工程 |
| `stm32_cubemx_rollback` | 恢复 `.bak-<时间戳>` |
| `stm32_cubemx_script` | 逃生通道：直接跑 CubeMX CLI 命令 |

## 环境变量

| 变量 | 说明 |
|---|---|
| `CUBEMX_HOME` | CubeMX 安装目录 |
| `CUBEMX_TIMEOUT_MS` | 单次调用超时，建议 120000 |
| `CUBEMX_ALLOW_ANY_PATH` | 默认 true；共享给他人时改 false |
| `CUBEMX_ALLOWED_ROOTS` | 配合上一条的可写目录白名单 |

## 排错顺序

1. **工具没加载** → 先单独跑冒烟测试：`echo {"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{}}} | node server.mjs`
2. **报 `CUBEMX_HOME` 找不到** → 环境变量写错，或路径里有空格没加引号
3. **写到一半失败** → 检查是不是 CubeMX GUI 开着（有窗口时默认拒写）
4. **报 IP not ready** → 工程里有未就绪外设，日志会点名，回去补配
5. **超时** → 客户端超时调到 120s 以上，CubeMX 首次冷启动本身就要几十秒
6. **生成不出来** → 先 `inspect` 确认 `.ioc` 的 `MxCube.Version` 与本机固件包版本一致

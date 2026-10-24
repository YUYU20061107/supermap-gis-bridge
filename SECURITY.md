# 安全模型与漏洞报告（SECURITY）

本仓（supermap-gis-bridge，SuperMap iDesktop / iDesktopX 的 AI Agent 桥）
的安全设计写在这里供任何人审查；与实现不一致时以代码为准并欢迎提
Issue 指出。本仓是 gis-agent-bridges 合集仓中 supermap 包的单仓发行版，
安全口径与合集仓保持一致。

## 安全模型

- **只在本机**：应用内桥只监听 `127.0.0.1:47615`，不监听任何对外网卡；
  MCP 服务端走 stdio，只与本机客户端通信。
- **随机 token**：应用内桥每次启动生成随机 token，写入
  `~/.supermap-live-bridge.token`（可用 `SM_BRIDGE_TOKEN_FILE` 环境变量
  改位置）；请求不带对 token 一律拒绝（`BAD_TOKEN`）。
- **confirm 闸**：内容级写工具（add_dataset_to_map /
  make_label_thematic / save_workspace）缺 `confirm=true` 一律拒绝执行；
  confirm 只能在用户明确同意后由 Agent 填入，Agent 不许自作主张绕过。
- **原件只读**：无头盘点永不直接打开原件；自动启动桥时只在工作空间
  **副本**上拉起；备份生成时间戳副本并回报 sha256。无头桥另有
  PathGuard-lite 校验：拒绝 UNC 路径、含控制字符的路径，以及写目标
  与源路径解析后相同的调用。
- **无任意代码执行**：不提供 eval/exec 外部输入、任意脚本或任意处理
  工具调用的通道；Agent 能做的事被限定在可枚举、可审计的 13 个工具内。

## 威胁模型（在范围内）

- 同一台机器上其他进程试图直连桥端口冒用 Agent 身份（token 防线）；
- Agent 被诱导在未获用户同意时执行写操作（confirm 防线）；
- Agent 或用户误传路径导致覆盖原件（备份＋副本＋PathGuard 防线）。

## 不在范围内的事项（如实声明）

- 本项目**不是沙箱**：拿到本机 token 文件读取权限的进程即可调用桥，
  token 文件的权限管理取决于使用者的操作系统账户与目录权限。
- confirm 是「经用户同意」的协议约定，不是密码学强制：被完全劫持的
  客户端可以自行填 confirm。SuperMap 软件自身的权限与许可不在本
  项目控制范围内；本项目不分发任何厂商组件。

## 校准状态

文件回退模式已在 iDesktop 10i(2020) 上实机校准（13/13，见
`docs/CALIBRATION-RESULTS-20261024.md`）；iDesktopX 实时模式仍未实机
校准。「协议自检通过」不等于「已在真实软件上校准」，细节见
`docs/CALIBRATION.md`。

## 漏洞报告

发现安全问题请开 GitHub Issue（标题以 `[security]` 开头），写明复现
步骤与影响范围；涉及可用利用细节的，请先在 Issue 中只报概要，维护者
确认后会与你约定披露细节的方式。请勿在公开 Issue 中粘贴任何真实
token、凭据或私人工程路径。

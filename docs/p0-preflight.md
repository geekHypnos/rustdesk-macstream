# P0 实现前核查

检查日期：2026-11-14。状态：进行中，已建立双机 Direct IP 连接，尚未完成交互与稳定性验收。

## 固定上游候选

- 原版基线：RustDesk 1.5.3。
- 标签对应提交：`95cca3f661f635b3cc85c04935fe8d96d8f432ef`。
- [官方发布页](https://github.com/rustdesk/rustdesk/releases/tag/1.5.3)。尚未安装或运行；固定版本不等于已验证兼容。
- Apple Silicon 安装包：`rustdesk-1.5.3-aarch64.dmg`，31,464,911 字节。
- 官方 API 提供的 SHA256：`e6d30fa1633d772c48935907c208ca4e555a856fb2038af3aaa87a327c29360b`。下载后必须实际校验，不能将此值视为已完成校验。

## 本机检查（设备 A）

| 项目 | 结果 |
| --- | --- |
| 系统与架构 | macOS 15.3.1 / arm64 |
| Rust / Cargo | 1.96.0，尚未验证与上游构建兼容 |
| CMake / Ninja | 4.3.2 / 1.13.2 |
| Flutter / CocoaPods | PATH 未发现 |
| Xcode | 当前选择 CommandLineTools；标准应用目录未发现完整 Xcode |
| RustDesk | 系统及用户 Applications 标准位置未发现 |
| 可用存储 | 检查时约 4.1 GiB；暂不进行完整源码构建和大型依赖安装 |
| Tailscale | 官方应用 1.104.4；CLI 包装器调用应用内可执行文件 |

## 双机连接阶段结果

- 两台 Mac 均运行 Tailscale，检测到唯一在线的 macOS 对端；未记录设备名、地址或账号。
- 被控端已由用户手动开启 RustDesk Direct IP、配置认证并授予所需系统权限。
- 控制端安装经官方 SHA256、Developer ID 签名及 Apple 公证验证的 RustDesk 1.5.3 Apple Silicon 版。
- 对端默认 Direct IP 端口可达，用户确认已通过 Tailscale 地址建立 RustDesk 连接。
- 固定密码由用户在 RustDesk 界面设置和输入，本项目未读取、保存或上传密码。
- 键鼠控制、文字输入、滚动、窗口拖动、剪贴板、重连和持续稳定性仍待验证，因此 P0 尚未通过。

Tailscale 状态读取在沙箱内退出码为 0，但输出不是 JSON。经授权在正常权限下重试同一只读命令，得到合法 JSON，`BackendState=Running`。因此不能仅凭退出码判断在线，也不能把受限环境的读取异常判断成客户端未连接。原始输出未打印、未保存，设备列表、地址、账号均未写入本报告。

`Running` 只说明本机后端状态，不代表对端可达、RustDesk 已运行、Direct IP 已启用或认证已通过。其他状态及异常分支尚未实际验证。

## 官方构建要求核查

依据固定提交的 [flutter-build.yml](https://github.com/rustdesk/rustdesk/blob/95cca3f661f635b3cc85c04935fe8d96d8f432ef/.github/workflows/flutter-build.yml)：

- macOS arm64 使用 Flutter 3.44.9；Intel 使用 Flutter 3.24.5，不能统一套用一个版本。
- macOS Rust 工具链变量为 1.81；当前本机版本不同，尚未进行构建对照。
- Apple Silicon 构建启用 ScreenCaptureKit，不应将其作为本项目从零新增的能力。
- 上游依赖递归子模块、vcpkg、Flutter 和 CocoaPods 路径；arm64 流程明确禁用 Flutter 默认的 Swift Package Manager 迁移以继续使用 CocoaPods。
- Podfile 声明 macOS 10.14，但不能把单个 Podfile 声明当成所有架构、引擎及依赖的运行兼容性保证。

当前只下载少量公开构建文本进行核查，没有克隆完整源码、安装开发工具或更改系统配置。

## 下一步与门槛

1. 补录设备 B 架构、系统及安装包版本；不得公开设备身份或地址。
2. 测试文字编辑、滚动、拖动、剪贴板、断开重连及持续连接，记录脱敏结果。未完成前 P0 不得标为通过。
4. 在进入 P1b 前解决构建磁盘空间和完整工具链缺口，验证固定提交可构建；不擅自删除文件或升级系统。
5. 完成上述门槛后再实现最小模块；不以独立模拟 UI、文档或解析器替代真实 RustDesk 接入验收。

本轮未改 HA、VPN、路由、代理设置或权限；网络请求仅使用用户指定的现有本机代理。未发起远程控制连接。

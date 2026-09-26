# Radeon Cloud Robotwin Team Guide

这个公开仓库用于共享 AMD Radeon Cloud / Robotwin 的通用操作经验、连接检查 Notebook 和队内实验记录建议。它共享的是文档与代码，不会共享账号、credits、私钥或云实例访问权。

## 安全边界

- 不要提交私钥、账号密码、令牌、个人 SSH 配置或实例连接命令。
- `instance` 和 `local_settings.json` 是每位成员自己的本地配置，已列入 `.gitignore`。
- 不要把带有运行输出的 Notebook 提交到公开仓库。输出可能包含 IP、用户名、主机指纹或实验数据；提交前清除所有输出。
- 队员使用自己的 Radeon Cloud 账号和 SSH 密钥。不要共用平台账号或转让 credits。
- 官方算力规则以赛事文档为准：同一队伍最多同时运行一个实例；队伍 credits 由队伍按赛事流程使用。

## 每位队员的本地设置

1. 克隆仓库，并在 AMD Cloud 网页启动或获授权使用队伍实例。
2. 复制 `instance.example` 为本地文件 `instance`，把内容替换成平台实例页给出的 SSH 命令，例如 `ssh <user>@<host> -p <port>`。不要提交这个本地文件。
3. 复制 `local_settings.example.json` 为 `local_settings.json`。默认密钥路径为 `~/.ssh/id_ed25519`；如果密钥文件名不同，只在本地配置中修改 `identity_file`。不要在配置中填写私钥内容。
4. 首次连接时，在本地 PowerShell 交互式运行 `instance` 文件中的 SSH 命令，核对平台提供的服务器指纹后接受 host key。之后再运行 Notebook。
5. 在 VS Code 中打开 `check_amd_cloud_ssh_template.ipynb`，选择本地 Python 内核，从上到下运行单元。

Notebook 使用 Python 标准库检查地址解析、TCP 端口和 SSH 公钥认证。HTTP 测试仅用于另有 Web 服务的情况。若希望测试本地代理，在 `local_settings.json` 中设置 `run_proxy_check` 为 `true`；本机须有 HTTP/Mixed 代理监听 `7890`。反向转发测试是一次性 SSH 连接；实际使用代理时，需要保持带 `-R` 的 SSH 会话运行。

## 队员共享同一实例

队员不需要把私钥交给队长。每个人用自己的电脑打开 [`get_ssh_public_key.ipynb`](get_ssh_public_key.ipynb) 并运行单元。Notebook 会列出本机 `.pub` 公钥、显示指纹，并将选定的完整公钥行复制到剪贴板。队员把剪贴板内容发给 **Alex**，由 Alex 添加到共享实例。若本机还没有公钥，Notebook 中提供了 Windows、macOS 和 Linux 的密钥生成命令。

只发送 `.pub` 公钥；不要发送对应的无后缀私钥、口令或账号密码。Notebook 不会自动上传密钥，也不会把密钥写入仓库。

按官方 FAQ 操作：新实例按平台 SSH key 设置流程登记队员公钥；如果实例已经运行，由有权限的队员登录该实例，把每位队员的公钥作为单独一行追加到对应 Linux 用户的 `~/.ssh/authorized_keys`，不需要销毁实例。随后每个人用自己的私钥和实例 SSH 地址连接。只分享公钥，绝不要上传或发送私钥、账号密码。

这种授权通常让成员以同一个 Linux 用户进入实例，意味着他们共享该用户的文件和权限。只添加可信队员的公钥，并在队员退出项目时按需移除对应公钥。

## 实验同步建议

- 代码、配置模板和通用命令通过 Git 提交与同步。
- 训练日志和 checkpoint 保存在实例持久化的 `/workspace`；仓库只放小型、可公开的说明或索引，不要把大模型权重推入 Git。
- 记录每次实验的代码版本、数据版本、关键参数、运行时间和结果摘要，避免提交私有数据或受限材料。
- 实例与 credits 生命周期按赛事规则管理；任务结束后及时停止/释放实例。

## 官方参考

- [Radeon Cloud 用户指南](https://github.com/AMD-DEV-CONTEST/Embodied-AI-Challenge-AMD-Platform-2026-09/blob/main/Radeon-Cloud-User-Guide/README.md)
- [AMD VLA Contest 算力申请与使用规则](https://github.com/AMD-DEV-CONTEST/Embodied-AI-Challenge-AMD-Platform-2026-09/blob/main/Radeon-Cloud-User-Guide/AMD_VLA_Contest_Compute_Rules.md)
- [Robotwin Radeon Cloud 训练示例](https://github.com/ZiguanWang/Robotwin-radeon-cloud)

## License

本仓库使用 MIT License，详见 [LICENSE](LICENSE)。

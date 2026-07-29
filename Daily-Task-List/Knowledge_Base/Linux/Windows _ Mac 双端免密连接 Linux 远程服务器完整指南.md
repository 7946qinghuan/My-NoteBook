# Windows / Mac 双端免密连接 Linux 远程服务器完整指南

本文档为**双平台通用 SSH 免密登录手册**，涵盖 Windows、Mac 两套完整操作流程，统一采用更安全的 ED25519 密钥算法，实现终端、VS Code Remote\-SSH 永久免密快捷连接，附带完整排错方案。

## 一、免密登录核心原理

SSH 免密基于**非对称公钥加密**验证身份，无需账号密码登录：

- **私钥（Private Key）**：存放于本地电脑（Windows/Mac），等同于个人钥匙，严禁外泄、外传、上传公网。

- **公钥（Public Key）**：上传至 Linux 服务器，写入 `~/.ssh/authorized_keys`，服务器持有“锁”，本地私钥匹配即可免密登录。

核心成功条件：**密钥配对正确 \+ 服务器文件权限合规 \+ SSH 公钥认证开启**。

---

# 第一部分：Windows 端免密配置（PowerShell）

## 1\.1 生成 ED25519 密钥对

打开 Windows PowerShell / CMD，执行专属密钥生成命令，自定义密钥文件避免冲突：

```bash
ssh-keygen -t ed25519 -f "$HOME\.ssh\id_ed25519_remote"
```

参数说明：

- `-t ed25519`：新一代加密算法，安全性、速度全面优于 RSA。

- `-f`：指定密钥保存路径与文件名，区分多台服务器密钥。

交互提示：**全部直接回车**，不设置 Passphrase，实现真正全程免密。

## 1\.2 公钥一键上传至 Linux 服务器（推荐）

PowerShell 直接执行以下命令，自动创建目录、写入公钥、配置权限，无需手动操作：

```powershell
$pubKey = Get-Content "$HOME\.ssh\id_ed25519_remote.pub"
ssh user@remote_host "mkdir -p ~/.ssh && echo '$pubKey' >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

替换说明：将 `user@remote_host` 改为你的服务器账号和 IP，例如 `root@192.168.1.100`。

## 1\.3 手动上传方案（备用）

1. 记事本打开 `C:\Users\用户名\.ssh\id_ed25519_remote.pub`，复制全部内容。

2. 密码登录服务器，执行编辑命令：`nano ~/.ssh/authorized_keys`

3. 文件末尾粘贴公钥，保存退出（nano：Ctrl\+O 回车、Ctrl\+X）。

4. 执行权限修复：
`chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys`

## 1\.4 本地 Config 快捷登录配置

在路径 `C:\Users\用户名\.ssh\config` 新建/修改配置文件，实现别名一键连接：

```plaintext
Host my_server
    HostName 1.2.3.4
    User root
    IdentityFile ~/.ssh/id_ed25519_remote
    ServerAliveInterval 30
    ServerAliveCountMax 4
```

新增保活参数，解决长时间挂机断开、VS Code 掉线问题。

---

# 第二部分：Mac 端免密配置（终端通用）

## 2\.1 生成 ED25519 密钥对

打开 Mac 终端，执行同规范密钥生成命令，保持双端统一：

```bash
ssh-keygen -t ed25519 -f "$HOME/.ssh/id_ed25519_remote"
```

全程回车，不设置密钥密码，实现完全免密。

## 2\.2 一键推送公钥至服务器（最简方案）

Mac 自带 `ssh-copy-id` 工具，一行命令自动完成配置：

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_remote.pub user@remote_host
```

首次输入服务器登录密码，自动写入公钥并配置权限。

## 2\.3 Mac 手动上传备用方案

```bash
# 查看并复制公钥
cat ~/.ssh/id_ed25519_remote.pub

# 登录服务器后执行
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "粘贴公钥内容" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

## 2\.4 Mac 本地 Config 快捷配置

编辑 Mac 本地 SSH 配置文件：

```bash
touch ~/.ssh/config
open ~/.ssh/config
```

写入与 Windows 完全一致的配置（跨平台通用）：

```plaintext
Host my_server
    HostName 1.2.3.4
    User root
    IdentityFile ~/.ssh/id_ed25519_remote
    ServerAliveInterval 30
    ServerAliveCountMax 4
```

配置权限（Mac 必需）：

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/config
```

---

# 第三部分：双端通用连接方式

Windows / Mac 配置完成后，终端统一命令一键秒连：

```bash
ssh my_server
```

VS Code Remote\-SSH 自动读取 `config` 文件，直接点击连接，无需重复配置。

---

# 第四部分：通用故障排查（双端通用）

## 4\.1 权限错误（90% 免密失败原因）

Linux SSH 对目录/文件权限极严格，权限过松直接拒绝免密登录，服务器执行修复：

```bash
chmod 700 ~
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

## 4\.2 服务器未开启公钥认证

编辑 SSH 服务配置：

```bash
sudo vim /etc/ssh/sshd_config
```

确保开启以下两项：

```plaintext
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```

重启 SSH 服务：

```bash
# CentOS / Rocky
systemctl restart sshd

# Ubuntu / Debian
systemctl restart ssh
```

## 4\.3 密钥带 Passphrase 导致仍需输密码

重新生成无密码密钥，或清空旧密钥密码：

```bash
ssh-keygen -p -f 密钥路径
```

## 4\.4 频繁掉线断开

确认本地 config 中已添加 `ServerAliveInterval` 保活配置，适配长时间开发、挂机场景。

---

# 第五部分：多服务器拓展规范

如需连接多台 Linux 服务器，只需：

1. 生成多组独立密钥（命名区分，如`id_ed25519_server1`、`id_ed25519_server2`）

2. 在 config 中新增独立 Host 段落，分别绑定 IP、用户名、私钥路径

3. 互不冲突，自由切换免密连接

> （注：部分内容可能由 AI 生成）

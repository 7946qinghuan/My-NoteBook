---
title: M‑Series Mac（Apple Silicon）Homebrew 完整安装&卸载教程
create_at: 2026-08-14
update_at: 2026-08-14
tags:
  - Mac
  - "#Homebrew"
---
M‑Series Mac（Apple Silicon）Homebrew 完整安装&卸载教程

**适用范围**：M1/M2/M3/M4 等 Apple Silicon 芯片 MacBook Pro/Air，macOS 系统（终端默认 zsh）

**环境区分**：Intel(x86‑64) Mac 安装路径为 `/usr/local`，本文档专属 **ARM64(M芯片)** 设备

# 一、安装 Homebrew

## 1. 执行官方安装脚本

通过 `/bin/bash` 执行脚本，规避 zsh 语法兼容问题，设备交互 shell 仍保留为 zsh。

```zsh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

安装操作步骤：

1. 脚本请求 **sudo 权限**，输入 Mac 开机密码（输入无屏幕回显为正常现象）
    
2. 出现提示 `Press RETURN/ENTER to continue or any other key to abort:`，直接按**回车键**确认
    
3. 等待自动下载、解压安装，输出 `==> Installation successful!` 即代表 Homebrew 本体安装完成

## 2. 注入 zsh 环境变量（核心关键）

未执行该步骤会出现 `brew: command not found` 报错，直接复制以下命令逐条在终端执行：
> 💡注意：Homebrew新版本输出推荐写入 `.zprofile`（登录shell加载）；网上旧教程很多写入`.zshrc`，两种都可以，**不要两处同时写，避免PATH重复污染**。




## 3. 验证安装结果

执行以下命令，输出版本号即代表安装成功：

```zsh
brew --version
```

## 4. 基础使用示例（安装 GitHub CLI）

```zsh
# 安装 GitHub CLI 工具
brew install gh

# 验证安装是否成功
gh --version

# 执行 GitHub 账号登录授权
gh auth login
```

## 5. 国内网络备选方案

若官方脚本下载长时间卡住、网络超时，可替换**国内镜像安装脚本**完成安装。

# 二、Homebrew 常用命令速查

```zsh
brew update                  # 更新brew自身包索引
brew install xxx             # 安装指定软件包
brew uninstall xxx           # 卸载指定软件包
brew list                    # 列出所有已安装的软件包
brew info xxx                # 查看指定软件包的详细信息
brew doctor                  # 检测brew环境异常，排查问题首选命令
brew upgrade                 # 更新全部已安装的软件包及依赖
```

# 三、完全卸载 Homebrew（完整干净清理）

执行该流程会**删除所有通过brew安装的程序、依赖文件**，操作前请确认无需要保留的软件，谨慎操作！

## 步骤1：执行官方卸载脚本

```zsh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/uninstall.sh)"
```

按终端提示输入 sudo 密码，确认卸载操作即可。

## 步骤2：手动清理M芯片根目录（必做）

官方卸载脚本无法自动删除核心根目录，需手动执行命令彻底清理：

```zsh
sudo rm -rf /opt/homebrew
```

## 步骤3：删除环境PATH配置

1. 删除系统路径配置文件

```zsh
sudo rm /etc/paths.d/homebrew
```

2. 移除 zsh 配置中的 brew 环境变量（二选一即可）

方式一：手动打开 `~/.zprofile`，删除以下内容

```zsh
eval "$(/opt/homebrew/bin/brew shellenv zsh)"
```

方式二：终端一键清空对应配置

```zsh
sed -i '' '/brew shellenv/d' ~/.zprofile
```

## 步骤4：重载Shell环境

```zsh
source ~/.zprofile
```

## 步骤5：校验卸载完成

```zsh
brew --version
```

终端输出 `command not found: brew`，代表彻底卸载清理完毕。

# 四、核心知识点&踩坑避坑清单

1. **安装脚本为何用bash而非zsh？** Homebrew官方安装脚本基于bash语法开发，独立bash子进程运行可规避zsh语法差异导致的隐蔽报错，不影响日常zsh交互使用。
    
2. **路径区分**：M系列ARM芯片 Mac 路径为 `/opt/homebrew`，Intel芯片 Mac 路径为`/usr/local`，切勿混淆。
    
3. **必做操作**：安装完成后必须注入zsh环境变量，否则新开终端无法识别brew命令。
    
4. **网络问题**：官方下载卡顿/超时，直接切换国内镜像源。
    
5. **故障排查**：brew运行异常优先执行 `brew doctor`，工具会自动检测问题并给出修复方案。
    
6. **卸载注意**：官方卸载脚本不会删除 `/opt/homebrew`根目录，必须手动执行rm命令清理。
    

# 五、补充说明（GitHub CLI 相关）

1. **gh 与 git 区别**：`gh(GitHub CLI)` ≠ git。git 负责本地代码版本控制；gh 用于调用GitHub API，处理远端PR、issue、代码评审等远程操作。
    
2. **本地使用无影响**：本地仓库代码管理、git基础操作**不需要gh工具**，未安装gh仅无法操作远端GitHub相关功能，本地文件、git流程完全不受影响。
    

**快速上手提示**：安装完成后，直接执行 `brew install gh` 安装GitHub CLI，再通过 `gh auth login` 完成账号授权即可正常使用。如需详细的授权步骤指引，可随时查看配套操作指南。
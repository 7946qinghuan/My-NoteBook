Mac Terminal 配置 Clash Verge 代理完整笔记

## 一、核心结论

**Mac 默认终端（Terminal/iTerm2）不会自动跟随系统代理**。

macOS 图形界面（浏览器、桌面软件）可读取系统网络代理配置，但命令行工具（Terminal、Git、Homebrew、Curl 等）不识别系统代理，需要手动配置 `Proxy 环境变量` 才能走 Clash Verge 代理。

---

## 二、前置准备

1. 打开 **Clash Verge** → 进入设置
    
2. 查看并记录 **Mixed 混合端口**（默认：`7890`，部分版本为 `7897`）
    
3. 确保 Clash Verge 正常启动、代理模式开启
    

---

## 三、三种终端代理配置方案

macOS 新版系统默认 Shell 为 **zsh**，以下配置均适配 zsh 环境。

### 方案1：临时生效（仅当前终端窗口）

适合临时使用，关闭终端后配置自动失效，直接复制执行（替换为自己的端口）：

```bash
# 配置 HTTP/HTTPS/SOCKS 代理
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=http://127.0.0.1:7890
export ALL_PROXY=socks5://127.0.0.1:7890

# 兼容小写变量（适配部分小众命令行工具）
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export all_proxy=socks5://127.0.0.1:7890
```

#### 代理有效性测试：
```bash
curl cip.cc
```
`curl cip.cc` 走的是「直连（Direct）」
- `cip.cc` 是国内的 IP 查询服务。
    
- 分流规则识别到这是国内域名，自动绕过代理，直接使用你本地的网络请求。
    
- 例如**结果**：返回的是你真实的本地宽带 IP（**中国XXXX电信** `183.12.46.4`）。

```shell
curl ip.sb
```
`curl ip.sb` 走的是「代理（Proxy）」
- `ip.sb` 是海外的 IP 查询服务。
    
- 分流规则识别到这是海外域名，自动将请求转发给了你的代理节点。
    
- 例如**结果**：返回的是代理服务器的出口 IP（**美国 AWS 节点** `35.92.96.54`）。
```

```
返回代理 IP 地址即配置成功。

### 方案2：永久生效（推荐｜带开关指令）

配置全局终端代理，支持一键开启/关闭，永久复用。

#### 1. 编辑环境配置文件

```bash
nano ~/.zshrc
```


#### 2. 创建独立代理脚本（精简.zshrc核心方案）

为避免 `~/.zshrc` 文件臃肿、配置杂乱，将代理功能代码独立为专属脚本文件，**.zshrc 仅保留一行调用命令**，极致简洁、方便后续维护修改。

第一步：创建专属代理脚本文件，存放自定义代理配置

```bash
# 统一标准自定义脚本文件夹（推荐 zsh_scripts，更通用）
mkdir -p ~/.zsh_scripts
# 创建代理专属脚本
vim ~/.zsh_scripts/proxy_clash.zsh
```

第二步：在 `proxy_clash.zsh` 文件中粘贴完整代理功能代码（端口可自行修改）

```bash
# Clash Verge 终端代理独立配置脚本
# 统一端口变量，后续修改端口仅需改动此处
CLASH_MIXED_PORT="7890"
CLASH_PROXY_HTTP="http://127.0.0.1:${CLASH_MIXED_PORT}"
CLASH_PROXY_SOCKS="socks5://127.0.0.1:${CLASH_MIXED_PORT}"

# 一键开启终端代理
function proxy_on {
    export HTTP_PROXY=$CLASH_PROXY_HTTP
    export HTTPS_PROXY=$CLASH_PROXY_HTTP
    export ALL_PROXY=$CLASH_PROXY_SOCKS
    export http_proxy=$CLASH_PROXY_HTTP
    export https_proxy=$CLASH_PROXY_HTTP
    export all_proxy=$CLASH_PROXY_SOCKS
    echo "✅ Clash 终端代理已开启（端口：${CLASH_MIXED_PORT}）"
}

# 一键关闭终端代理
function proxy_off {
    unset HTTP_PROXY HTTPS_PROXY ALL_PROXY http_proxy https_proxy all_proxy
    echo "❌ Clash 终端代理已关闭"
}

# 可选：默认打开终端自动开启代理（不需要可注释/删除此行）
proxy_on
```

保存退出：`Ctrl+O` 回车 → `Ctrl+X`

第三步：极简配置 `~/.zshrc`，仅添加一行调用代码

```bash
nano ~/.zshrc
```

在文件末尾仅添加 **一行注释 + 一行调用指令**，全程无冗余代码：

```bash
# 加载Clash终端代理独立配置（解耦zshrc，保持配置简洁）
# 路径必须与自己创建的脚本文件夹一致！
[ -f ~/.zsh_scripts/proxy_clash.zsh ] && source ~/.zsh_scripts/proxy_clash.zsh
```

#### 3. 保存并生效

- 保存退出：`Ctrl+O` 回车 → `Ctrl+X`
    
- 重载配置：

```bash
source ~/.zshrc
```

#### 日常使用指令

- 开启终端代理：`proxy_on`
    
- 关闭终端代理：`proxy_off`

### 方案3：单独工具代理（不全局生效）

仅让 Git、Homebrew 等单个工具走代理，不影响整个终端环境。

#### Git 代理配置

```bash
# 开启 Git 全局代理
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

# 取消 Git 代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

#### Homebrew 单次代理

```bash
ALL_PROXY=socks5://127.0.0.1:7890 brew install 软件名
```

---

## 四、常见踩坑解决方案

### 1. 本地服务（127.0.0.1）被代理拦截、502 报错

Clash 默认会代理本地地址，导致本地项目、本地接口无法访问，需添加直连规则：

Clash Verge → 订阅配置 → 合并配置，添加如下规则：

```yaml
prepend-rules:
  - MATCH,DIRECT
  - IP-CIDR,127.0.0.0/8,DIRECT
  - IP-CIDR,::1/128,DIRECT
```

保存重启 Clash Verge 即可，本地流量直连，外网流量走代理。

### 2. 忘记 Clash 代理端口

终端执行命令查询当前占用端口：

```bash
lsof -iTCP -sTCP:LISTEN | grep clash
```

### 3. VSCode 内置终端、iTerm2 不生效

所有第三方终端均依赖系统 Shell 环境变量，配置 `.zshrc` 后，**重启终端**即可正常生效。

---

## 七、高频报错修复：zsh: command not found: proxy_on

### 报错根因

1、文件夹路径不匹配：教程默认 `.zsh_scripts`，你实际创建的是 `.zsh_custom`，`.zshrc` 读取不到脚本； 2、未重载配置 / 脚本未成功加载，代理函数未注册到终端； 3、使用 vim 编辑脚本时，存在空字符、格式错乱、代码粘贴不完整问题。

### 一键修复步骤

1、确认并修正 `~/.zshrc` 最终引入路径（必须和你的文件夹一致）

```bash
# 清空原有代理引入代码，粘贴以下唯一正确配置
[ -f ~/.zsh_scripts/proxy_clash.zsh ] && source ~/.zsh_scripts/proxy_clash.zsh
```

2、强制重载配置（必执行）

```bash
source ~/.zshrc
```

3、测试指令（正常无报错即可使用）

```bash
proxy_on
proxy_off
```

### 关键注意事项

✅ 路径大小写、文件夹名称**必须完全一致**，zsh 严格区分大小写； ✅ 若仍报错，打开脚本检查代码是否完整，无乱码、无粘贴截断； ✅ 不建议混用 `.zsh_custom` / `.zsh_scripts`，全程统一一个文件夹； ✅ `curl ip.sb` 能拿到代理IP但命令不存在，**仅为脚本函数未加载**，代理本身是通的。

## 五、原理说明

- macOS **系统网络代理**：仅对 GUI 图形桌面程序生效
    
- 终端 CLI 工具：遵循 Unix 标准，仅识别 `HTTP_PROXY/HTTPS_PROXY/ALL_PROXY` 环境变量
    
- 二者相互独立，因此终端必须手动配置环境变量才能走代理

---

## 六、老旧设备兼容（Bash 环境）

**Mac 所有版本都会内置并兼容多种 Shell**（bash、zsh、sh 等），不是新版就只能用 zsh、旧版只能用 bash，核心规则如下：

1.**系统默认规则**：macOS Catalina（10.15）及以上出厂**默认预设 Shell 为 zsh**；10.15 以下版本出厂默认是 bash。

2. **完全自由切换**：无论新旧系统，均可手动切换 Shell，你当前使用 zsh，大概率是自己手动配置/切换的，属于自定义选择，不是系统强制固定。

3. **配置文件一一对应**：不同 Shell 独立加载专属配置文件，互不干扰：

- **zsh** → 专属配置文件：`~/.zshrc`（本文全程使用）
    
- **bash** → 专属配置文件：`~/.bash_profile`

---

#### ✅ 常用 Shell 环境说明（Mac 全系内置）

- `/bin/zsh`：现代 macOS 出厂默认，功能强、支持插件/主题，终端体验更好
    
- `/bin/bash`：旧版 macOS 出厂默认，全系系统保留兼容，稳定无兼容问题
    
- `/bin/sh`：基础通用标准 Shell，仅用于基础命令执行，极少自定义配置
    

---

#### ✅ 实操命令：查看 & 切换 Shell 环境

**1. 查看当前正在使用的 Shell**

```bash
echo $SHELL
```

**2. 查看系统所有支持的 Shell**

```bash
cat /etc/shells
```

**3. 手动切换默认 Shell（输入开机密码确认）**

```bash
# 切换为 zsh
chsh -s /bin/zsh

# 切换为 bash
chsh -s /bin/bash
```

**切换生效条件**：执行命令后，**彻底关闭终端重新打开**，即可完成环境切换。

---

#### ✅ 环境适配说明

本文的独立脚本代理方案，**zsh / bash 完全通用**：只需根据自己当前 Shell，选择对应的配置文件引入脚本即可，`proxy_on`、`proxy_off`、端口变量配置逻辑无需任何修改。

旧版 Bash 环境完整适配操作：创建、引入代理脚本的步骤、端口变量、proxy_on/proxy_off 开关指令完全不变，仅需将读取配置的文件替换为 bash 专属文件，操作命令如下：

```bash
nano ~/.bash_profile
source ~/.bash_profile
```

12
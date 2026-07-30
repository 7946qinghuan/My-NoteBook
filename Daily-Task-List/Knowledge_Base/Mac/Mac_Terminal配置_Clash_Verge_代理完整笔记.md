# Mac Terminal 配置 Clash Verge 代理完整笔记

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

#### 代理有效性测试

```bash
curl cip.cc
```

返回代理 IP 地址即配置成功。

### 方案2：永久生效（推荐｜带开关指令）

配置全局终端代理，支持一键开启/关闭，永久复用。

#### 1\. 编辑环境配置文件

```bash
nano ~/.zshrc
```

#### 2\. 文件末尾追加以下代码

```bash
# Clash Verge 终端代理开关
function proxy_on {
    export HTTP_PROXY=http://127.0.0.1:7890
    export HTTPS_PROXY=http://127.0.0.1:7890
    export ALL_PROXY=socks5://127.0.0.1:7890
    export http_proxy=http://127.0.0.1:7890
    export https_proxy=http://127.0.0.1:7890
    export all_proxy=socks5://127.0.0.1:7890
    echo "✅ Clash 终端代理已开启"
}

function proxy_off {
    unset HTTP_PROXY HTTPS_PROXY ALL_PROXY http_proxy https_proxy all_proxy
    echo "❌ Clash 终端代理已关闭"
}

# 可选：打开终端自动开启代理（不需要可删除此行）
proxy_on
```

#### 3\. 保存并生效

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

### 1\. 本地服务（127\.0\.0\.1）被代理拦截、502 报错

Clash 默认会代理本地地址，导致本地项目、本地接口无法访问，需添加直连规则：

Clash Verge → 订阅配置 → 合并配置，添加如下规则：

```yaml
prepend-rules:
  - MATCH,DIRECT
  - IP-CIDR,127.0.0.0/8,DIRECT
  - IP-CIDR,::1/128,DIRECT
```

保存重启 Clash Verge 即可，本地流量直连，外网流量走代理。

### 2\. 忘记 Clash 代理端口

终端执行命令查询当前占用端口：

```bash
lsof -iTCP -sTCP:LISTEN | grep clash
```

### 3\. VSCode 内置终端、iTerm2 不生效

所有第三方终端均依赖系统 Shell 环境变量，配置 `.zshrc` 后，**重启终端**即可正常生效。

---

## 五、原理说明

- macOS **系统网络代理**：仅对 GUI 图形桌面程序生效

- 终端 CLI 工具：遵循 Unix 标准，仅识别 `HTTP_PROXY/HTTPS_PROXY/ALL_PROXY` 环境变量

- 二者相互独立，因此终端必须手动配置环境变量才能走代理

---

## 六、老旧设备兼容（Bash 环境）

若设备为旧版 macOS，默认 Shell 为 bash，配置文件更换为 `~/.bash_profile`，操作一致：

```bash
nano ~/.bash_profile
source ~/.bash_profile
```

> （注：部分内容可能由 AI 生成）

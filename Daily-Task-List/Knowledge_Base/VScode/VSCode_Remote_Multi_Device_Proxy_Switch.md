# VS Code Remote-SSH 多设备环境隔离与服务端动态代理自动切换指南

## 1. 背景与核心需求

在拥有多台客户端设备（如 **MacBook Pro** 与 **Windows Desktop**）并借助 Tailscale 等内网穿透/组网工具远程连接同一台 Linux 开发服务器的场景下，我们通常希望服务器上的 Shell 能根据**当前发起的客户端设备**自动切换对应的代理配置（如 HTTP/SOCKS5 代理）。

- **Mac 设备**（Tailscale IP: `100.64.0.40`，代理端口 `7897`）
    
- **Windows 设备**（Tailscale IP: `100.64.0.4`，代理端口 `7890`）
    
- **Linux 远程服务器**（Tailscale IP: `100.64.0.15`）
    

## 2. 排查过程与踩坑思维链 (Chain of Thought)

整个问题的解决经历了从**网络层行为分析**到**进程生命周期与机制突破**的推演过程：

### 阶段一：Terminal 正常，但 VS Code 流量走向异常

- **现象**：Mac 原生 Terminal 连服务器，`echo $SSH_CLIENT` 显示 Mac IP (`100.64.0.40`)；而 Mac VS Code 连接时，显示的却是 Windows IP (`100.64.0.4`)。
    
- **原因排查**：VS Code 作为 GUI 应用，默认会读取系统代理或设置中的 `Http: Proxy Support`。当本地存在代理软件转发时，VS Code 的 SSH 建连流量被本地代理拦截中转，导致服务端感知到的源 IP 变成了代理出口 IP。
    
- **第一阶段修复**：在 VS Code 中关闭 `Http: Proxy Support`，并在 Clash/Surge 等本地代理工具中将 Tailscale 网段（`100.64.0.0/10`）划入 Direct 直连规则。
    

### 阶段二：跨设备连接时环境变量“冻结”与端口一致性悬案

- **现象**：Mac VS Code 恢复正常后，换用 Windows VS Code 连接服务器，新建终端执行 `echo $SSH_CLIENT`，依然显示 Mac 的 IP，且**源端口号完全相同**（如 `100.64.0.40 57268 22`）；但 Windows CMD 原生连接却显示 `100.64.0.4`。
    
- **关键突破**：TCP 协议中，不同物理设备建立的新连接**绝对不可能分配到完全相同的源端口号**。这表明：Windows VS Code 根本没有发起新的 Shell 主进程，而是挂载了 Mac 之前建立的远程服务端进程。
    

## 3. 底层机制与原理拆解

要彻底解决该问题，必须理解 VS Code Remote-SSH 的工作原理与 Linux 进程继承关系。

### 3.1 VS Code Remote-SSH 进程架构

当通过 VS Code 连接远程服务器时，系统并非单纯建立一个 SSH 交互 Shell，而是形成了如下的树状进程模型：

Plaintext

```
[本地 VS Code 客户端] 
        │ (SSH Tunnel)
        ▼
[远程 Linux Server]
        └── vscode-server (后台常驻守护进程，Node.js)
                 ├── extension-host (插件宿主进程)
                 └── terminal-server (终端管理进程)
                          └── zsh (集成终端 Shell 子进程)
```

### 3.2 环境变量继承与进程生命周期固化

1. **`vscode-server` 常驻机制**：当第一个客户端（如 Mac）连入服务器时，`sshd` 派生并启动了 `vscode-server` 进程。此时，`sshd` 将 Mac 的 SSH 环境变量（包括 `SSH_CLIENT=100.64.0.40 ...`）写入到了 `vscode-server` 主进程中。
    
2. **连接复用与复活 (Session Revive)**：当第二个客户端（如 Windows）连接同一台服务器时，为了提升连接速度，VS Code 默认会复用服务器上已有的 `vscode-server` 进程，而不会杀死重新启动。
    
3. **环境变量冻结**：在 Linux 中，**子进程只能继承父进程被创建时的环境变量**。因此，无论你在 Windows VS Code 里新建多少个终端标签页，它们都是由同一个 `vscode-server` 派生出来的，继承的依然是最初 Mac 建立连接时的 `$SSH_CLIENT`。
    

### 3.3 结论：为什么不能依赖 `$SSH_CLIENT`？

在原生 Terminal / CMD 中，每次登录都会触发一次全新的 `sshd` 鉴权与 Shell 启动，`$SSH_CLIENT` 准确无误；而在 VS Code Remote 场景下，由于 `vscode-server` 进程的常驻性，**依靠网络层的 `$SSH_CLIENT` 来区分客户端设备是天然不可靠的**。

## 4. 终极解决方案：端侧变量注入 + 动态 Shell 路由

既然依靠“服务端推断”不可靠，思路转变为：**由客户端 VS Code 主动向服务端终端注入当前设备的身份标识**。

### 步骤 1：客户端配置（VS Code 本地用户设置）

利用 VS Code 的 `terminal.integrated.env.linux` 配置，在打开远程 Linux 终端时自动注入自定义环境变量 `VSCODE_CLIENT_DEV`。

- **Mac 端**（打开 Mac VS Code 的 `settings.json`）：
    

JSON

```
{
  "terminal.integrated.env.linux": {
    "VSCODE_CLIENT_DEV": "mac"
  }
}
```

- **Windows 端**（打开 Windows VS Code 的 `settings.json`）：
    

JSON

```
{
  "terminal.integrated.env.linux": {
    "VSCODE_CLIENT_DEV": "win"
  }
}
```

### 步骤 2：服务端脚本改写（`~/.auto-proxy.zsh`）

在服务器上修改自动代理脚本，**优先校验客户端主动注入的变量**；如果非 VS Code 环境（如终端直连），再回退到依据 `$SSH_CLIENT` 识别。

Bash

```
# ==========================================
# 自动化代理切换脚本 (~/.auto-proxy.zsh)
# ==========================================

# --- 代理网络配置区 ---
WIN_TS_IP="100.64.0.4"
MAC_TS_IP="100.64.0.40"
WIN_PROXY_PORT=7890
MAC_PROXY_PORT=7897
# ----------------------

auto_proxy_switch() {
    local client_ip
    client_ip=$(echo "$SSH_CLIENT" | awk '{print $1}')

    # 设置内网绕过代理名单
    export no_proxy=127.0.0.1,localhost,192.168.0.0/16,100.64.0.0/10,.steins.net
    export NO_PROXY="${no_proxy}"

    # 清空历史代理变量
    unset all_proxy ALL_PROXY http_proxy https_proxy HTTP_PROXY HTTPS_PROXY socks5_proxy

    # ----------------------------------------------------
    # 策略 1：优先根据 VS Code 客户端本地注入的环境变量判断
    # ----------------------------------------------------
    if [[ "${VSCODE_CLIENT_DEV}" == "win" ]]; then
        export all_proxy=socks5h://${WIN_TS_IP}:${WIN_PROXY_PORT}
        export ALL_PROXY=$all_proxy
        export http_proxy=http://${WIN_TS_IP}:${WIN_PROXY_PORT}
        export https_proxy=$http_proxy
        export socks5_proxy=socks5://${WIN_TS_IP}:${WIN_PROXY_PORT}
        return
    elif [[ "${VSCODE_CLIENT_DEV}" == "mac" ]]; then
        export all_proxy=socks5h://${MAC_TS_IP}:${MAC_PROXY_PORT}
        export ALL_PROXY=$all_proxy
        export http_proxy=http://${MAC_TS_IP}:${MAC_PROXY_PORT}
        export https_proxy=$http_proxy
        export socks5_proxy=socks5://${MAC_TS_IP}:${MAC_PROXY_PORT}
        return
    fi

    # ----------------------------------------------------
    # 策略 2：非 VS Code 环境（如 Mac Terminal / Win CMD），回退至 SSH 来源 IP 判断
    # ----------------------------------------------------
    if [[ "${client_ip}" =~ ^192\.168\.3\. ]] || [[ "${client_ip}" == "${WIN_TS_IP}" ]]; then
        export all_proxy=socks5h://${WIN_TS_IP}:${WIN_PROXY_PORT}
        export ALL_PROXY=$all_proxy
        export http_proxy=http://${WIN_TS_IP}:${WIN_PROXY_PORT}
        export https_proxy=$http_proxy
        export socks5_proxy=socks5://${WIN_TS_IP}:${WIN_PROXY_PORT}
    elif [[ "${client_ip}" == "${MAC_TS_IP}" ]]; then
        export all_proxy=socks5h://${MAC_TS_IP}:${MAC_PROXY_PORT}
        export ALL_PROXY=$all_proxy
        export http_proxy=http://${MAC_TS_IP}:${MAC_PROXY_PORT}
        export https_proxy=$http_proxy
        export socks5_proxy=socks5://${MAC_TS_IP}:${MAC_PROXY_PORT}
    fi
}

# 挂载时立即执行
auto_proxy_switch
```

## 5. 效果验证与总结

完成上述配置后，运行架构达到了完美解耦的状态：

|**客户端类型**|**连接方式**|**识别机制**|**生效代理 IP & 端口**|
|---|---|---|---|
|**MacBook**|Mac Terminal (SSH)|`$SSH_CLIENT` (`100.64.0.40`)|`100.64.0.40:7897`|
|**MacBook**|Mac VS Code|`VSCODE_CLIENT_DEV=mac`|`100.64.0.40:7897`|
|**Windows**|Windows CMD (SSH)|`$SSH_CLIENT` (`100.64.0.4`)|`100.64.0.4:7890`|
|**Windows**|Windows VS Code|`VSCODE_CLIENT_DEV=win`|`100.64.0.4:7890`|

### 最佳实践复盘

1. **网络层与应用层分离**：网络层的 IP 地址容易受代理中转、会话复用等因素干扰，在复杂的远程开发工具链中，通过**应用层显式传递环境变量（Terminal Env Injection）** 是处理多端差异最稳妥的方式。
    
2. **理解守护进程生命周期**：Remote-SSH 工具（如 VS Code、Cursor、JetBrains Remote）普遍存在服务端守护进程常驻机制，排查这类问题时，重点关注父子进程的环境变量继承，而非单纯关注网络连接本身。
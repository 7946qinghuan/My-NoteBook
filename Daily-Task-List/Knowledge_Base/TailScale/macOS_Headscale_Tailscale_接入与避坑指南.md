# macOS 接入自建 Headscale (Tailscale) 组网与踩坑指南

本指南记录了在 macOS 系统下，使用官方 Tailscale CLI/GUI 客户端接入自建 Headscale 控制平面（`https://223.4.6.21:26001`）的完整操作流程、踩坑避坑经验及退出组网的操作说明。

---

## 一、 快速接入步骤

### 1. 导入自签名 HTTPS 证书（必做）
由于 Headscale 使用了自签名证书，Mac 默认不信任该证书，必须先将服务端的 `headscale.crt` 导入 macOS **“系统”钥匙串** 并设置为“始终信任”：

在终端进入证书存放目录，运行：
```bash
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain headscale.crt
```


### 2. 清理后台进程缓存

macOS 的 Tailscale 后台守护进程（`tailscaled` / `IPNExtension`）会缓存证书信任状态，必须强杀进程以重载系统证书：

```bash
sudo killall -9 IPNExtension Tailscale tailscaled 2>/dev/null
```

### 3. 发起连接与登录认证

运行 `tailscale up` 命令绑定登录服务器，并带上 `--reset` 参数清空旧状态：

Bash

```bash
sudo /Applications/Tailscale.app/Contents/MacOS/Tailscale up --login-server=[https://223.4.6.21:26001](https://223.4.6.21:26001) --reset
```

终端将输出形如 `https://223.4.6.21:26001/register/xxxxx` 的链接，将其发送给 Headscale 管理员在服务端运行 `headscale nodes register` 进行授权。

## 二、 踩坑记录与解决方案

### 坑点 1：`x509: certificate signed by unknown authority` 报错

- **问题现象**：连接时终端提示 TLS 证书未被信任，握手失败。
    
- **主要原因**：
    1. 未导入 Headscale 的公钥证书。
    2. **证书位置放错**：误将证书导入到了“钥匙串访问”的 **“登录”（Login）** 栏下。Tailscale 后台服务以 `root`/系统级权限运行，无法读取用户级的“登录”钥匙串。
- **解决办法**：
    - 确保证书必须位于 **“系统”（System）** 钥匙串下，且信任设置中的 **“使用此证书时”** 已改为 **“始终信任”**。

### 坑点 2：导入证书并信任后，连接依然报证书错误

- **问题现象**：已经在“钥匙串访问”中把证书拖到了系统下并设为信任，但执行 `tailscale up` 依然报 `x509` 错误。
    
- **主要原因**：Tailscale 的系统拓展进程（`IPNExtension`）或后台守护进程（`tailscaled`）在后台持续运行，缓存了旧的“不可信”证书状态，没有实时读取更新后的系统钥匙串。
    
- **解决办法**：强制杀死所有 Tailscale 进程以强制刷新证书缓存：
```bash
sudo killall -9 IPNExtension Tailscale tailscaled 2>/dev/null
```

### 坑点 3：旧登录状态残留导致认证超时或失败

- **问题现象**：之前连接过官方 Tailscale 或其他 Headscale 节点，导致再次连接时无法弹出新的 Register URL，或提示 `already logged in`。
    
- **主要原因**：本地客户端保存了旧的 session key 和节点状态。
    
- **解决办法**：
    - 在 `tailscale up` 时加上 `--reset` 参数清空本地缓存状态重新握手：
```bash
sudo /Applications/Tailscale.app/Contents/MacOS/Tailscale up --login-server=[https://223.4.6.21:26001](https://223.4.6.21:26001) --reset
``` 

### 坑点 4：提示 `Some peers are advertising routes but --accept-routes is false`

- **问题现象**：连接成功（显示 `Success.`），但健康检查提示路由未接受。
    
- **主要原因**：组网内有其他节点（如子网路由器）广播了机房/内网网段（如 `192.168.x.x` 或 `10.x.x.x`），但本客户端默认关闭了“接收子网路由”功能。
    
- **解决办法**：
    
    - **仅节点点对点通信**：忽略该警告即可，不影响通过 `100.64.x.x` IP 互相访问。
        
    - **需要访问服务端所在的私有内网**：追加 `--accept-routes` 重新启动：
```bash
sudo /Applications/Tailscale.app/Contents/MacOS/Tailscale up --login-server=[https://223.4.6.21:26001](https://223.4.6.21:26001) --accept-routes
```

## 三、 常用客户端运维命令

|**需求**|**命令**|
|---|---|
|**查看当前分配的 IP**|`sudo /Applications/Tailscale.app/Contents/MacOS/Tailscale ip`|
|**查看网络内所有节点状态**|`sudo /Applications/Tailscale.app/Contents/MacOS/Tailscale status`|
|**修改当前设备节点名称**|`sudo /Applications/Tailscale.app/Contents/MacOS/Tailscale up --login-server=https://223.4.6.21:26001 --hostname=新设备名`|

## 四、 如何退出组网（操作步骤与注意事项）

如果未来需要暂停使用或彻底从 Headscale 组网中移除本设备，请参考以下步骤：

### 1. 临时断开连接（不清除登录信息）

如果只是临时不需要使用组网，希望恢复本地正常网络，可以暂时下线虚拟网卡：

```bash
sudo /Applications/Tailscale.app/Contents/MacOS/Tailscale down
```

- **效果**：关闭虚拟网卡，断开内网连接。后续只需运行 `sudo ... Tailscale up` 即可重新上线，无需再次审核。

### 2. 彻底退出并注销客户端（Logout）

如果需要切换账号、退出当前 Headscale 实例或彻底注销：

```bash
sudo /Applications/Tailscale.app/Contents/MacOS/Tailscale logout
```

- **效果**：清除本地的登录密钥（Auth Key）、状态配置和节点缓存。重新连接时必须重新访问链接让管理员授权。
    

### 3. 服务端节点清理（必须联系管理员）

客户端退出后，Headscale 服务端可能仍保留该节点的废弃注册信息，需要联系管理员从控制端清理：
```bash
# 管理员在 Headscale 服务器上执行：
headscale nodes list                   # 查看节点 ID
headscale nodes delete -i <节点ID>     # 删除该节点
```

### 4. 退出后的注意事项

- **移除信任证书（可选）**：如果不再连接该 Headscale 服务器，可打开 macOS “钥匙串访问”，在左侧选择 **“系统”**，搜索 `223.4.6.21` 证书并将其右键删除，以保持系统证书库简洁。
- **检查 DNS 与路由残留**：执行 `tailscale logout` 后，Tailscale 会自动清理创建的虚拟网卡（`utun`）及全局路由规则。若发现局部网络异常，重启一次系统或重启网络网卡即可完全复位。


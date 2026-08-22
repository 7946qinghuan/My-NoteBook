## 下载镜像
南京大学 e‑Science 镜像
专门做海外容器仓库**域名代理**：直接替换域名即可

|原始地址|南大代理|
|---|---|
|`ghcr.io`|`ghcr.nju.edu.cn`|
|`gcr.io`|`gcr.nju.edu.cn`|
|`quay.io`|`quay.nju.edu.cn`|
|`nvcr.io`|`ngc.nju.edu.cn`|
拉拉取OpenCode的镜像时，直接把域名`ghcr.io`替换成 `ghcr.nju.edu.cn`，不需要改 Docker 全局配置
原命令：

```bash
docker pull ghcr.io/anomalyco/opencode:1.18.21
```

改成：

```bash
docker pull ghcr.nju.edu.cn/anomalyco/opencode:1.18.21
```
或者：

```
docker pull ghcr.m.daocloud.io/anomalyco/opencode:1.18.21
```


> ⚠️ 下载后的本地镜像名字会携带ghcr 代理域名，如：`ghcr.nju.edu.cn/anomalyco/opencode:1.18.21`而不是原来的`ghcr.io/anomalyco/opencode:1.18.21`

现在本地镜像名字带南大代理域名，先改名还原官方名称，后续命令统一：
```bash
docker tag ghcr.nju.edu.cn/anomalyco/opencode:1.18.21 ghcr.io/anomalyco/opencode:1.18.21
```


## Open‑Code 两种部署模式

### 模式 A：后台 API 服务（端口`20001`）

`opencode serve` 启动 http 接口服务，可以被你的 `stream_events.py`客户端调用

完整可直接复制运行命令 (Mac)

bash

```
docker run -d \
  --name opencode-server \
  -p 20001:20001 \
  -v $(pwd):/app \
  -v ~/.config/opencode:/root/.config/opencode \
  ghcr.io/anomalyco/opencode:1.18.21 \
  opencode serve --port 20001 --hostname 0.0.0.0
```

#### 参数拆解

表格

| 参数                                               | 作用                                                |
| ------------------------------------------------ | ------------------------------------------------- |
| `-d`                                             | 后台守护运行                                            |
| `-p 20001:20001`                                 | 端口映射，宿主机 20001 →容器 20001                          |
| `-v $(pwd):/app`                                 | 挂载当前项目目录到容器内 /app                                 |
| `-v ~/.config/opencode:/root/.config/opencode`   | 挂载宿主机 opencode 配置文件，provider、模型配置持久化，容器删掉配置不会丢    |
| `opencode serve --port 20001 --hostname 0.0.0.0` | **容器内执行启动 API 服务**，`0.0.0.0`必须写，否则只能容器内部访问，宿主机连不上 |

### 模式 B：交互式 TUI（终端界面）

```bash

docker run -it --rm -v $(pwd):/app ghcr.io/anomalyco/opencode:1.18.21 opencode tui
```

## 启动完成后校验


```bash
#查看容器状态
docker ps
#查看日志（排查报错）
docker logs opencode-server
#测试接口是否通
curl http://127.0.0.1:20001/health
```

成功后你就可以直接运行你的 python 脚本

```bash
python examples/stream_events.py --url http://127.0.0.1:20001 --provider steins-middleware-vllm --model Qwen3.8-27B
```

## Mac 专属坑点

1. **容器访问本机 vLLM 服务**：如果你模型服务跑在 Mac 宿主机，容器内部不能直接用`127.0.0.1`，需要用特殊域名 `host.docker.internal`
    
    provider 里面 baseURL 要改成`http://host.docker.internal:8000/v1`
    
    2. 配置挂载：`~/.config/opencode`存放你的`opencode.json`，里面写好`steins-middleware-vllm`模型配置，容器直接复用
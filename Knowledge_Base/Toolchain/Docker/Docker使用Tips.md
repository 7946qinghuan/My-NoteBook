南京大学 e‑Science 镜像
专门做海外容器仓库**域名代理**：直接替换域名即可

|原始地址|南大代理|
|---|---|
|`ghcr.io`|`ghcr.nju.edu.cn`|
|`gcr.io`|`gcr.nju.edu.cn`|
|`quay.io`|`quay.nju.edu.cn`|
|`nvcr.io`|`ngc.nju.edu.cn`|
例如拉拉取OpenCode的镜像：
直接把域名`ghcr.io`替换成 `ghcr.nju.edu.cn`，不需要改 Docker 全局配置
原命令

```bash
docker pull ghcr.io/anomalyco/opencode:1.18.21
```

改成

```bash
docker pull ghcr.nju.edu.cn/anomalyco/opencode:1.18.21
```

> ⚠️ 下载后的本地镜像名字是`ghcr.nju.edu.cn/anomalyco/opencode:1.18.21`而不是`ghcr.io/anomalyco/opencode:1.18.21`


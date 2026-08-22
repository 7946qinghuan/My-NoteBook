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



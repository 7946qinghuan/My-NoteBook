
## 1. 安装`OpenCode`：
```bash
brew install anomalyco/tap/opencode
```
安装后校验命令：
```bash
# 查看版本
opencode --version

# 查看brew安装信息
brew info opencode

# 二进制软链接位置
which opencode
```
### 常用管理命令

#### 1、卸载

```bash
brew uninstall opencode
```

> 只会删掉 opencode，默认**不会自动删掉 pcre2 /ripgrep**（因为 brew 会保留有可能被别的程序用到的依赖）

如果你想顺带把本次安装引入、现在没用的依赖也清理掉：

```bash
brew autoremove
```

#### 2、升级 opencode

```bash
brew update
brew upgrade opencode
```

`brew update` 会同步 `anomalyco/tap` 这个第三方源最新版本

#### 3、查看信息

```bash
brew info opencode
```

可以看到版本、源地址、依赖、安装路径

#### 4、重装

```bash
brew reinstall opencode
```

#### 5、查看是否已经安装

```bash
brew list | grep opencode
```

### 需要注意 2 个坑

1. **不要手动删 `/opt/homebrew/Cellar/opencode`**
    手动删除会破坏 brew 的数据库，brew 会状态错乱，必须统一用 `brew uninstall`
    
2. 第三方 tap 本身可以删掉（不影响已装软件）


## 2. 安装`Claude Code`：
```bash
brew install --cask claude-code
```

### 异常问题：
如果出现一下图片的弹窗：
![Apple M系列 Brew安装Claude Code CLI无法使用](https://cdn.statically.io/gh/comet-7x/My-NoteBook/main/images/20260818210022568.png)

问题原因：

macOS 安全隔离（Gatekeeper）拦截了刚通过 brew 安装的`claude-code`二进制文件，报错 `Apple无法验证“claude”是否包含...恶意软件`、进程直接被系统 kill 掉。
>虽然是 Homebrew 安装，claude‑code 的预编译二进制没有苹果公证签名，Arm Mac 会直接阻止运行。

#### 方案 1：终端一键解除隔离（推荐，最简单）

直接执行这条命令

```bash
xattr -cr /opt/homebrew/bin/claude
```

参数解释：
- `xattr`：mac 扩展属性工具
- `-cr`：递归清除隔离标记 (com.apple.quarantine)

执行完立刻测试

```bash
claude --version
```

如果上面命令无效，找到真实本体

`/opt/homebrew/bin/claude`只是软链接，需要清除**源文件**的隔离标记

```bash
#获取真实路径
realpath /opt/homebrew/bin/claude
```

输出类似

`/opt/homebrew/Caskroom/claude-code/版本号/claude`

然后对真实文件执行

```bash
xattr -cr /opt/homebrew/Caskroom/claude-code/*/claude
```

再次运行

```bash
claude --version
```

#### 备选方案（图形界面方式）

1. 弹出弹窗时点**完成**，不要点移到废纸篓

2. 打开「系统设置 → 隐私与安全性」，往下滑

3. 你会看到一行提示：`“claude”被阻止使用`，点【仍然允许】

4. 回到终端重新运行`claude --version`

#### 补充排查
1. 确认 PATH 没问题

```bash
which claude
#正确输出 /opt/homebrew/bin/claude
```

2. 如果持续 killed，可以看系统日志确认是安全拦截

```bash
log show --predicate 'process == "syspolicyd"' --last 5m
```

> 小提示：curl 脚本安装版本一般自带签名跳过逻辑，很少触发这个报错；brew cask 下载的二进制包更容易触发 Gatekeeper 隔离。


安装后校验命令：
```bash
# 查看版本
claude --version

# 查看brew安装信息
brew info claude

# 二进制软链接位置
which claude
```

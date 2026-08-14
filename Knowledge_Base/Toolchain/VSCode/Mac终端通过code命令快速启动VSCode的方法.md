---
title: Mac终端通过code命令快速启动VSCode的方法
create_at: 2026-08-13
update_at: 2026-08-13
tags:
  - VSCode
---
在Mac开发过程中，通过终端快速启动VSCode可以显著提升效率。本文介绍两种核心方法：通过VSCode内置功能安装`code`命令和手动配置别名

---

## 通过VSCode内置功能安装`code`命令

### 操作步骤

1. **打开VSCode**，使用快捷键 **`Shift + Command + P`** 打开命令面板。
2. 在命令面板中输入 **`Shell Command: Install 'code' command in PATH`**
3. 选择并执行该命令，完成后关闭终端并重新打开。
4. 验证是否成功：在终端输入以下命令，若返回版本号则表示配置成功：
```bash
code --version
```

### 适用场景

- 适用于大多数标准安装路径的VSCode
- 无需手动修改系统配置文件
- 推荐作为首选方案

## 手动配置别名（备用方案）

### 操作步骤

1. 打开终端配置文件（如`~/.zshrc`或`~/.bash_profile`）：

```bash
vi ~/.zshrc  # 若使用zsh
```

2. 在文件末尾添加以下行（路径需根据实际安装位置调整）：

```bash
alias code='/Applications/Visual\ Studio\ Code.app/Contents/Resources/app/bin/code'
```
3. 保存文件并退出编辑器（按`Esc`后输入`:wq`），然后执行：
```bash
source ~/.zshrc  # 使配置立即生效
```
### 注意事项

- 路径中的空格需使用反斜杠转义
- 若VSCode安装在非默认路径，需修改路径为实际安装位置
- 适用于Shell类型为zsh或bash的情况

## 常用`code`命令示例

|命令|功能描述|
|---|---|
|`code .`|打开当前目录|
|`code /path/to/file.txt`|打开指定文件|
|`code -n /path/to/folder`|新建窗口打开目录|
|`code --help`|显示帮助信息|

## 常见问题解决

### 路径问题

若VSCode安装在非默认路径（如`/Applications/VSCode.app`），需在别名中替换为实际路径

### Shell类型问题

Mac默认使用`zsh`，若使用其他Shell（如`bash`），需修改对应的配置文件（如`~/.bash_profile`）

### 权限问题

若遇到权限错误，可尝试使用`sudo`执行链接命令（方法一更推荐）
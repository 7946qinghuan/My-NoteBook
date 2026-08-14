---
title: Python contextlib 上下文管理器类型注解零基础速查笔记
create_at: 2026-08-14
update_at: 2026-08-14
tags:
  - "#Python"
  - "#contextlib"
---
Python contextlib 上下文管理器类型注解零基础速查笔记

## 一、零基础前置必懂核心概念

### 1. 什么是上下文管理器？

就是 Python 中 `with xxx:` 语法专用的工具，核心作用：**自动执行前置准备、后置收尾，哪怕代码报错也不会漏执行收尾逻辑**。

日常场景：打开文件、数据库连接、加锁、开关状态、资源释放等。

### 2. 两个核心装饰器（全程只用这两个）

- **同步场景**：`@contextmanager`（普通函数、无 async/await）
    
- **异步场景**：`@asynccontextmanager`（async 异步函数、搭配 await）
    

### 3. 关键类型区别（解决90%报错）

- **绝对废弃**：`Iterator`（新版 Python 静态检查直接抛弃用警告，禁止用于上下文管理器）
    
- **同步专用**：`Generator[Yield, Send, Return]`（生成器类型，唯一合法同步注解）
    
- **异步专用**：`AsyncGenerator[Yield, Send]`（异步生成器，唯一合法异步注解）
    

### 4. Generator 三个参数零基础释义

格式：`Generator[产出值, 接收值, 返回值]`

1. **产出值**：`yield` 后面跟着的值（`with xxx as 变量` 拿到的就是它）
    
2. **接收值**：高级用法 `.send()` 传值，普通业务场景统一填 `None`
    
3. **返回值**：函数最后 `return` 的值，上下文管理器几乎不用，统一填 `None`
    

✅ 绝大多数常规场景通用模板：`Generator[None, None, None]`

### 5. AsyncGenerator 两个参数释义

格式：`AsyncGenerator[产出值, 接收值]`

异步场景无返回值参数，常规业务统一：`AsyncGenerator[None, None]`

---

## 二、同步上下文管理器（@contextmanager）速查

### 场景1：无 yield 值（最常用，仅做状态/资源开关）

特点：只有前置、后置逻辑，`yield` 后无内容，`with` 语句不需要接收变量

```python
from contextlib import contextmanager
from typing import Generator

@contextmanager
def sync_no_value() -> Generator[None, None, None]:
    # 前置逻辑：准备工作
    print("开始执行前置操作")
    try:
        # 无任何返回，仅占位切分前后逻辑
        yield
    finally:
        # 后置逻辑：收尾、释放资源（必执行）
        print("执行后置收尾操作")

# 使用方式
with sync_no_value():
    print("执行业务代码")
```

### 场景2：有 yield 值（with xxx as 变量接收数据）

特点：`yield` 返回指定类型数据，外部可以通过变量接收使用

```python
from contextlib import contextmanager
from typing import Generator

# 产出字符串，参数对应修改第一个泛型
@contextmanager
def sync_has_value() -> Generator[str, None, None]:
    print("前置：初始化资源")
    try:
        # 产出字符串，外部可接收
        yield "我是上下文产出的字符串数据"
    finally:
        print("后置：释放资源")

# 使用方式
with sync_has_value() as res:
    print(f"接收数据：{res}")
```

---

## 三、异步上下文管理器（@asynccontextmanager）速查

核心区别：函数必须加 `async`，使用时必须搭配 `async with`

### 场景1：无 yield 值

```python
from contextlib import asynccontextmanager
from typing import AsyncGenerator

@asynccontextmanager
async def async_no_value() -> AsyncGenerator[None, None]:
    print("异步前置操作")
    try:
        yield
    finally:
        print("异步后置收尾")

# 必须在异步函数中使用
async def main():
    async with async_no_value():
        print("异步业务代码")
```

### 场景2：有 yield 值

```python
from contextlib import asynccontextmanager
from typing import AsyncGenerator

@asynccontextmanager
async def async_has_value() -> AsyncGenerator[int, None]:
    print("异步初始化")
    try:
        # 产出数字类型数据
        yield 200
    finally:
        print("异步资源释放")

# 使用
async def main():
    async with async_has_value() as num:
        print(f"接收异步数据：{num}")
```

---

## 四、零基础必记避坑规则（根治所有报错）

### 1. 绝对禁止写法（100% 抛弃用警告）

不要用 `Iterator` 注解上下文管理器，新旧版本不兼容：

```python
# ❌ 错误、已弃用
from typing import Iterator
@contextmanager
def func() -> Iterator[None]:
    yield
```

### 2. 参数数量绝对不能错

- 同步 `Generator`：**必须写满3个参数**，不能少写
    
- 异步 `AsyncGenerator`：**必须写2个参数**
    
- ❌ 错误：`Generator[None]` / `AsyncGenerator[None]`
    
- ✅ 正确：`Generator[None, None, None]` / `AsyncGenerator[None, None]`
    

### 3. 场景匹配原则

- 普通函数 + with → 同步 `contextmanager + Generator`
    
- async 函数 + async with → 异步 `asynccontextmanager + AsyncGenerator`
    

---

## 五、极简万能模板（直接复制即用）

### 1. 同步无返回（90%业务场景）

```python
from contextlib import contextmanager
from typing import Generator

@contextmanager
def your_func() -> Generator[None, None, None]:
    # 前置逻辑
    try:
        yield
    finally:
        # 后置逻辑
```

### 2. 同步有返回

```python
from contextlib import contextmanager
from typing import Generator

@contextmanager
def your_func() -> Generator[你要的类型, None, None]:
    # 前置逻辑
    try:
        yield 返回值
    finally:
        # 后置逻辑
```

### 3. 异步无返回

```python
from contextlib import asynccontextmanager
from typing import AsyncGenerator

@asynccontextmanager
async def your_func() -> AsyncGenerator[None, None]:
    # 前置逻辑
    try:
        yield
    finally:
        # 后置逻辑
```

### 4. 异步有返回

```python
from contextlib import asynccontextmanager
from typing import AsyncGenerator

@asynccontextmanager
async def your_func() -> AsyncGenerator[你要的类型, None]:
    # 前置逻辑
    try:
        yield 返回值
    finally:
        # 后置逻辑
```

---

## 六、Python版本导入补充

- Python 3.8 及以下：必须从 `typing` 导入
    
- Python 3.9+：可从 `collections.abc` 导入，兼容性更好
    

```python
# 3.9+ 新版导入方式
from collections.abc import Generator, AsyncGenerator
```
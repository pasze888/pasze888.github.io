---
layout: post
title: Fedora运行Minecraft
description: Fedora使用HMCL和NVIDIA专有驱动运行Minecraft
author: pasze
date: 2026-02-24 09:56:05   +0800
image:
category:
tags:#
- linux 
- fedora 
- hmcl 
- minecraft 
- nvidia
---
## HMCL 添加环境变量

在 `设置->全局游戏设置->编辑高级游戏设置->环境变量` 添加

> 需要注意的是，HMCL 只会添加一个 `export`，如果有多个环境变量要添加，除第一个环境变量外，其他的均需要手动添加 `export`
{: .prompt-warning }

如需要添加多个环境变量，例如

```shell
var1=1
var2=1
```

则需要在环境变量栏填写

```shell
var1=1
export var2=1
```

如果实例使用了 `实例特定游戏设置`，请在 `实例管理` 中选择复制全局设置，或者手动在 `实例管理->编辑高级设置->环境变量` 手动添加环境变量

## Zink

```shell
export DRI_PRIME=1
```

在 hmcl `高级设置->环境变量` 添加 `DRI_PRIME=1`，我的 Fedora 43 上默认会用 zink 运行
如果不是 zink 可以添加

```shell
export MESA_LOADER_DRIVER_OVERRIDE=zink
```

> 需要注意的是：这样开启的 zink 会使用垂直同步

## Nouveau

```shell
export DRI_PRIME=1
export MESA_LOADER_DRIVER_OVERRIDE=nouveau
```

在 hmcl `高级设置->环境变量` 添加

```shell
DRI_PRIME=1
export MESA_LOADER_DRIVER_OVERRIDE=nouveau
```

## Nvidia 专有驱动

```shell
export __NV_PRIME_RENDER_OFFLOAD=1 
export __GLX_VENDOR_LIBRARY_NAME=nvidia
```

在 hmcl `高级设置->环境变量` 添加

```shell
__NV_PRIME_RENDER_OFFLOAD=1 
export __GLX_VENDOR_LIBRARY_NAME=nvidia
```

---
layout: post
title: 解决 SSH 报错 Permission denied (publickey)
description:
author:
date: 2026-02-17 14:54 +0800
image:
category:
tags:
- ssh
- github
- linux
---
最近想在 Ubuntu 服务器上开发，第一步就是给服务器配置 GitHub 的 SSH 访问权限，结果踩了个经典的坑——明明生成了密钥、添加到 GitHub，却始终认证失败。记录一下完整的排查和解决过程，帮大家少走弯路。

## 一、背景：服务器配置GitHub SSH的初衷

本地开发环境切换到云服务器后，需要频繁拉取/推送 GitHub 代码，为了避免每次输入密码，优先选择 SSH 认证方式。

## 二、操作步骤（踩坑版）

1. **生成 SSH 密钥对**：
   因为服务器可能要配置多个 Git 平台的密钥，所以没有用默认的 `id_rsa`，而是自定义了名称：

   ```bash
   ssh-keygen -t ed25519 -C "你的邮箱@xxx.com" -f ~/.ssh/id_git
   ```

   > `-f` 指定生成的私钥文件名是 `id_git`，公钥会自动生成 `id_git.pub`

2. **添加公钥到 GitHub**：
   复制 `~/.ssh/id_git.pub` 的全部内容，粘贴到 GitHub 账号的「Settings → SSH and GPG keys → New SSH key」中。

3. **测试认证（首次失败）**：
   执行验证命令，直接报错：

   ```bash
   ssh -T git@github.com
   # 报错：git@github.com: Permission denied (publickey).
   ```

4. **尝试常规修复（无效）**：
   - 修改 `.ssh` 目录和密钥权限（`chmod 700 ~/.ssh`、`chmod 600 ~/.ssh/id_git`）；
   - 尝试修改 SSH 端口（GitHub 22 端口本身没问题）；
   以上操作均未解决问题。

## 三、排查：用 ssh -vT 找到核心原因

参考 GitHub 官方文档[^footnote]，用 `ssh -vT` 查看详细的认证日志，定位问题：

```bash
ssh -vT git@github.com
```

### 关键日志片段

```bash
# 省略前序日志...
debug1: Authentications that can continue: publickey
debug1: Next authentication method: publickey
debug1: Will attempt key: /home/username/.ssh/id_rsa 
debug1: Will attempt key: /home/username/.ssh/id_ecdsa 
debug1: Will attempt key: /home/username/.ssh/id_ecdsa_sk 
debug1: Will attempt key: /home/username/.ssh/id_ed25519 
debug1: Will attempt key: /home/username/.ssh/id_ed25519_sk 
debug1: Will attempt key: /home/username/.ssh/id_xmss 
debug1: Will attempt key: /home/username/.ssh/id_dsa 
debug1: Trying private key: /home/username/.ssh/id_rsa
debug1: Trying private key: /home/username/.ssh/id_ecdsa
# ... 省略其他默认密钥尝试过程 ...
debug1: No more authentication methods to try.
git@github.com: Permission denied (publickey).
```

### 日志解读

SSH 客户端只会**按内置规则尝试默认命名的私钥**（id_rsa、id_ed25519 等），完全没尝试我自定义的 `id_git`，这就是认证失败的原因——SSH 没有使用我的密钥。

## 四、临时解决：-i 指定私钥（验证密钥有效性）

先通过 `-i` 参数手动指定私钥路径，验证密钥本身是有效的：

```bash
ssh -i ~/.ssh/id_git -T git@github.com
```

输出以下内容，说明密钥和 GitHub 配置都没问题：

```bash
Hi 你的GitHub用户名! You've successfully authenticated, but GitHub does not provide shell access.
```

但每次都手动加 `-i` 太麻烦，需要一个一劳永逸的方案。

## 五、配置 ~/.ssh/config 自动指定私钥

通过 SSH 的配置文件，让访问 GitHub 时自动使用自定义的 `id_git` 密钥：

### 1. 创建/编辑 config 文件

```bash
nano ~/.ssh/config
```

### 2. 写入配置内容

```config
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_git  # 指向自定义的私钥文件
  IdentitiesOnly yes          # 强制只使用指定的密钥，不尝试其他
```

> 这样配置后，服务器访问 GitHub就不需要每次手动指定私钥了
{: .prompt-tip }

### 3. 设置 config 文件权限（可选）

创建的文件权限一般为644，这里修改为600：

```bash
chmod 600 ~/.ssh/config
```

## 六、最终验证

无需手动指定私钥，直接执行验证命令：

```bash
ssh -T git@github.com
```

成功输出：

```bash
Hi 你的GitHub用户名! You've successfully authenticated, but GitHub does not provide shell access.
```

## 七、避坑总结

1. **自定义密钥名建议配置 config**：如果不用默认的 `id_rsa`/`id_ed25519`，一定要通过 `config` 指定 `IdentityFile`；
2. **使用日志排查问题**：`ssh -vT git@github.com` 能清晰看到 SSH 尝试了哪些密钥，是定位问题的核心命令；

## 参考

[^footnote]: <https://docs.github.com/zh/authentication/troubleshooting-ssh/error-permission-denied-publickey>
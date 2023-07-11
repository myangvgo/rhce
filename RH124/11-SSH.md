# Chapter 11 - SSH

[返回](../README.md)

[TOC]

## 1. SSH 基本使用

### 什么是 SSH

SSH 协议使系统能够通过不安全的网络通过加密和安全的方式进行通信。

```sh
ssh remotehost

ssh user@remotehost

# 在 remotehost 上以 user 的身份运行 hostname，不使用远程交互式 shell
ssh root@remotehost hostname
```

## 2. SSH 打开远端图形化界面

默认情况下，SSH 隧道只能传输字符。通过指定 `-X` 参数，可以传输 XClient

打开远端服务器的 XClient 图形化客户端，需要满足

* SSH 连接的时候，指定了 `-X` 参数
* 客户端主机运行了 `XServer`
* 云端服务器安装了 `xorg-x11-xauth`



## 3. 无密码登录 SSH 

SSH 使用公钥加密的方式保证通信安全。

* 当某一 SSH 客户端连接到 SSH 服务器的时候，服务器会向其发送公钥副本。
* 当用户使用 ssh 命令连接到服务器时，会检查本地是否有公钥副本。用户的家目录中可能会包含一个含有公钥的 `~/.ssh/known_hosts` 文件
* 如果客户端有公钥副本，ssh 就会对比 `~/.ssh/known_hosts` 文件中的公钥和收到的公钥；如果没有，需要先确认，然后保存公钥副本到 `~/.ssh/known_hosts` 文件

密钥认证

* 客户端生成密钥对，将公钥发送给root@server，保存在 root@server 的家目录下的 `.ssh/authorzied_keys`

```sh
# 生成密钥对，私钥不加密(没有指定密语)
ssh-keygen -N ""

# 把公钥拷贝到 server 上用户的家目录下 `.ssh/authorized_keys`
ssh-copy-id root@server
```

### `ssh-agent` 进行非交互式身份验证

如果 SSH 私钥受密语保护，通常需要输入密语才能使用密钥进行身份验证。

可以使用 `ssh-agent` 临时将密语保存到内存中，由 `ssh-agent` 自动提供密语

```sh
# 启动 ssh-agent
eval $(ssh-agent)

# 添加私钥
ssh-add
```

## 4. SSH 安全设置

OpenSSH 由 `sshd` 守护进程提供，配置文件为 `/etc/ssh/sshd_config`

### 禁止 root 用户使用 ssh 登录

```sh
PermitRootLogin no
```

### 禁用密码登录

```sh
PasswordAuthentication no
```

### 禁用密钥登录

```sh
# PubkeyAuthentication yes
```

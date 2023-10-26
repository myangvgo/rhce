# Chapter 05 SELinux

[返回](../)

- [Chapter 05 SELinux](#chapter-05-selinux)
  - [1. 什么是 SELinux](#1-什么是-selinux)
  - [2. SELinux 上下文](#2-selinux-上下文)
    - [查看上下文](#查看上下文)
    - [SELinux 标签](#selinux-标签)
    - [初始 SELinux 上下文](#初始-selinux-上下文)
    - [更改文件的 SELinux 上下文](#更改文件的-selinux-上下文)
  - [3. SELinux 端口上下文](#3-selinux-端口上下文)
  - [4. SELinux 模式](#4-selinux-模式)
  - [5. SELinux 布尔值](#5-selinux-布尔值)

---

## 1. 什么是 SELinux 

SELinux全称是Security-Enhanced Linux，目的是提高系统的安全性。当我们执行某个操作的时候如果selinux认为此操作有危险，则会拒绝进一步的访问。

SELinux 由应用开发人员定义的若干组策略组成。策略声明了各个**程序**、**文件**和**网络端口**上防止的预定义标签。

SELinux 是一个额外的系统安全层，用于确定哪个进程可以访问哪些文件、目录和端口的一组规则。

## 2. SELinux 上下文

在开启了selinux的情况下，selinux会为每个文件、进程、目录和端口都分配一个安全标签，这个标签我们称之为 **SELinux 上下文**(context)，后续说标签和上下文是同一个概念。

上下文是一个名称，SELinux 策略使用它来确定某个进程能否访问特定上下文的文件、目录或端口。

### 查看上下文

需要加上`Z`选项

```sh
# 查看上下文 `-Z`
[root@server130 opt]# ls -ld logs/
drwxr-xr-x. 2 root root 25 10月 19 21:34 logs/
[root@server130 opt]# ls -ldZ logs/
drwxr-xr-x. 2 root root unconfined_u:object_r:usr_t:s0 25 10月 19 21:34 logs/

# grep 进程过滤加上 Z
[root@server130 opt]# ps auxZ | grep -v grep | grep httpd
system_u:system_r:httpd_t:s0    root       32525  0.0  0.6 282900 11824 ?        Ss   21:59   0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0    apache     32526  0.0  0.4 296780  8592 ?        S    21:59   0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0    apache     32527  0.0  0.6 1813320 12344 ?       Sl   21:59   0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0    apache     32528  0.0  0.7 1944448 14392 ?       Sl   21:59   0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0    apache     32529  0.0  0.8 1813320 16424 ?       Sl   21:59   0:00 /usr/sbin/httpd -DFOREGROUND
```

### SELinux 标签

SELinux 标签有多种上下文：用户、角色、类型和敏感度。

类型上下文通常以 `_t` 结尾

```sh
unconfined_u:object_r:usr_t:s0 logs/
```

* `unconfined_u` SELinux User
* `object_r` Role
* `usr_t` Type
* `s0` Level
* `logs/` File

### 初始 SELinux 上下文

新文件通常从父目录继承其 SELinux 上下文，从而确保它们具有合适的上下文。

一般通过 `mv` 移动后的文件保留了原始的上下文标签，通过 `cp` 命令复制的命令则继承了新的目录的标签

### 更改文件的 SELinux 上下文

```sh
# chcon 更改 SELinux 上下文
chcon -R -t httpd_sys_content_t /www/

# 查看目录的上下文
ls -ldZ /www

# 还原默认值
restorecon -R /www
ls -ldZ /www

# 修改文件的默认标签
semanage fcontext -m -t httpd_sys_content_t "/www(/.*)?"

# 取消默认值
semanage fcontext -d -t httpd_sys_content_t "/www(/.*)?"

# 添加默认值
semanage fcontext -a -t httpd_sys_content_t "/www(/.*)?"
```

## 3. SELinux 端口上下文

```sh
semanage port -l | grep '\b80\b'

[root@server130 opt]# semanage port -l | grep '\b80\b'
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
[root@server130 opt]# semanage port -a -t http_port_t -p tcp 8888
[root@server130 opt]# semanage port -l | grep '\b8888\b'
http_port_t                    tcp      8888, 80, 81, 443, 488, 8008, 8009, 8443, 9000
```

## 4. SELinux 模式

```sh
# 查看当前 seinux 的模式
[root@server130 opt]# getenforce
Enforcing

# 修改当前 selinux 模式   
[root@server130 opt]# setenforce
usage:  setenforce [ Enforcing | Permissive | 1 | 0 ]
[root@server130 opt]# setenforce 0
[root@server130 opt]# getenforce
Permissive

# 通过修改配置文件，重启系统使配置永久生效（重启之后）
[root@server130 opt]# ls /etc/selinux/config
/etc/selinux/config
[root@server130 opt]# ls -l /etc/sysconfig/selinux
lrwxrwxrwx. 1 root root 17 10月 18 2022 /etc/sysconfig/selinux -> ../selinux/config
```

Enforcing：如果不满足selinux条件，则 selinux 会阻止访问服务，并提供警报

Permissive：如果不满足selinux条件，则 selinux 不会阻止访问服务，但会提供警报

Disabled：禁用 SELinux，不推荐

## 5. SELinux 布尔值

SELinux 布尔值是可更改 SELinux 策略行为的参数，是可以启用或者禁用的规则。

* 列出布尔值及其状态 `getsebool`
* 修改布尔值 `setsebool`
* 修改布尔值并写入策略持久保留 `setsebool -P`
* 列出布尔值是否为持久值 `semanage boolean -l`

```sh
[root@server130 opt]# getsebool -a | grep ftp
ftpd_anon_write --> off
ftpd_connect_all_unreserved --> off
ftpd_connect_db --> off
ftpd_full_access --> off
ftpd_use_cifs --> off
ftpd_use_fusefs --> off
ftpd_use_nfs --> off
ftpd_use_passive_mode --> off
httpd_can_connect_ftp --> off
httpd_enable_ftp_server --> off
tftp_anon_write --> off
tftp_home_dir --> off

# 修改当前 ftpd_full_access 布尔值
# 要使配置永久生效，则使用 -P 参数
setsebool ftpd_full_access on
setsebool -P ftpd_full_access 1

setsebool ftpd_full_access off
setsebool -P ftpd_full_access 0
```

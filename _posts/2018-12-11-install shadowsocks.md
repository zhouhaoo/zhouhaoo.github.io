---
layout:     post
title:      "install shadowsocks"
date:       2018-12-11 12:24:31
subtitle:   " \"Hello Shadowsocks\""
author:     "zhouhaoh"
header-img: "img/banner/the_lopez_family.png"
catalog: true
tags:
    - shadowsocks
---

 shadowsocks shell note 
 
#### 获取root权限

```shell
sudo su
```

#### 安装ssr

1. 获取脚本

```shell
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
```

2. 可执行权限

```shell
chmod +x shadowsocks.sh
```

3. 安装

```shell
./shadowsocks.sh 2>&1 | tee shadowsocks.log
```

#### 安装BBR加速

```shell
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
```

#### 脚本

> 不同版本：[shadowsocks_install](https://github.com/teddysun/shadowsocks_install)

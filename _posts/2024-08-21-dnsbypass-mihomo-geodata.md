---
title: 搭载 mihomo 内核进行 DNS 分流教程-geodata 方案
description: 此方案适用于 Clash，搭载 mihomo 内核并使用其特性进行 DNS 分流。包括 ShellCrash 和 Clash Verge 配置方法
date: 2024-08-21 08:48:20 +0800
categories: [DNS 配置, DNS 分流]
tags: [Clash, mihomo, 进阶, DNS, DNS 分流]
---

注：
- 1. 使用 [ShellCrash](https://github.com/juewuy/ShellCrash) 搭配 [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) 并将 AdGuard Home 作为上游时不要使用该方法
- 2. DNS 分流简单来说就是**指定国内域名走国内 DNS 解析，其它域名包括国外域名走 `fake-ip`**

# 一、 导入路由规则文件
geosite.dat 文件须包含 `fakeip-filter` 和 `cn`，推荐导入我定制的[路由规则文件](https://github.com/DustinWin/ruleset_geodata?tab=readme-ov-file#%E4%B8%80-geodata-%E8%A7%84%E5%88%99%E9%9B%86%E6%96%87%E4%BB%B6%E8%AF%B4%E6%98%8E)

# 二、 ShellCrash 设置
1. 进入主菜单 -> 2 内核功能设置 -> 2 切换 DNS 运行模式 -> 4 DNS 进阶设置，将“当前基础 DNS”和“PROXY-DNS”都设置为“null”  
<img src="/assets/img/dns/dns-null.png" alt="ShellCrash 设置" width="60%" />

2. 连接 SSH 后执行命令 `vi $CRASHDIR/yamls/user.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：

```yaml
dns:
  enable: true
  prefer-h3: true
  ipv6: true
  listen: 0.0.0.0:1053
  fake-ip-range: 198.18.0.1/16
  enhanced-mode: fake-ip
  fake-ip-filter: ['geosite:fakeip-filter,cn']
  nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

# 三、 [Clash Verge](https://github.com/clash-verge-rev/clash-verge-rev) 设置
进入 Clash Verge -> 订阅，右击“全局扩展配置”，选择“编辑文件”，将 `dns` 部分修改为如下内容并“保存”：

```yaml
dns:
  enable: true
  prefer-h3: true
  ipv6: true
  listen: 0.0.0.0:1053
  fake-ip-range: 198.18.0.1/16
  enhanced-mode: fake-ip
  fake-ip-filter: ['geosite:fakeip-filter,cn']
  nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
```
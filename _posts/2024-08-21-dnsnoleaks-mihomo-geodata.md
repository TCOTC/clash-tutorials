---
title: 搭载 mihomo 内核配置 DNS 不泄露教程-geodata 方案
description: 此方案适用于 Clash，搭载 mihomo 内核并使用其特性防止 DNS 泄露。包含 ShellCrash 和 Clash Verge 配置方法
date: 2024-08-21 08:18:30 +0800
categories: [DNS 配置, DNS 防泄漏]
tags: [Clash, mihomo, 进阶, DNS, DNS 泄露]
---

注：
- 1. 此方案彻底防止了 DNS 泄露（针对不在规则集内的域名和 IP 全部走 `fake-ip` 或国外 DNS 解析），配置简单粗暴，兼容性无法保证，请慎用
- 2. 可进入 <https://ipleak.net> 测试 DNS 是否泄露，“DNS Addresses” 栏目下没有中国国旗（因 `ipleak.net` 域名默认走代理），即代表 DNS 没有发生泄露

# 一、 导入路由规则文件
geosite.dat 文件须包含 `fakeip-filter`，推荐导入我定制的[路由规则文件](https://github.com/DustinWin/ruleset_geodata?tab=readme-ov-file#%E4%B8%80-geodata-%E8%A7%84%E5%88%99%E9%9B%86%E6%96%87%E4%BB%B6%E8%AF%B4%E6%98%8E)

# 二、 额外编辑配置文件
在《[生成带有自定义策略组和规则的 Clash 配置文件直链-geodata 方案/添加模板](https://proxy-tutorials.dustinwin.top/posts/link-clash-geodata/#%E4%BA%8C-%E6%B7%BB%E5%8A%A0%E6%A8%A1%E6%9D%BF)》编辑 .yaml 配置文件时，将 `rules` 里的所有 `GEOIP` 规则末尾加上 `no-resolve`，即修改为：

```yaml
  - GEOIP,telegram,📲 电报消息,no-resolve
  - GEOIP,private,🔒 私有网络,no-resolve
  - GEOIP,cn,🇨🇳 直连 IP,no-resolve
```

# 三、 [ShellCrash](https://github.com/juewuy/ShellCrash) 设置
1. 进入主菜单 -> 2 内核功能设置 -> 2 切换 DNS 运行模式 -> 4 DNS 进阶设置，将“当前基础 DNS”和“PROXY-DNS”都设置为 `null`  
<img src="/assets/img/dns/dns-null.png" alt="ShellCrash 设置" width="60%"  />

2. user.yaml 配置

- ① DNS 模式为 `fake-ip`  
连接 SSH 后执行 `vi $CRASHDIR/yamls/user.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：

  ```yaml
  dns:
    enable: true
    prefer-h3: true
    ipv6: true
    listen: 0.0.0.0:1053
    fake-ip-range: 198.18.0.1/16
    enhanced-mode: fake-ip
    fake-ip-filter: ['geosite:fakeip-filter']
    nameserver:
      - https://doh.pub/dns-query
      - https://dns.alidns.com/dns-query
  ```

  按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
- ② DNS 模式为 `redir-host`  
连接 SSH 后执行 `vi $CRASHDIR/yamls/user.yaml`，按一下 Ins 键（Insert 键），粘贴如下内容：

  ```yaml
  dns:
    enable: true
    prefer-h3: true
    ipv6: true
    listen: 0.0.0.0:1053
    fake-ip-range: 198.18.0.1/16
    enhanced-mode: fake-ip
    fake-ip-filter: ['+.*']
    nameserver:
      - https://dns.google/dns-query
      - https://cloudflare-dns.com/dns-query
    proxy-server-nameserver:
      - https://doh.pub/dns-query
      - https://dns.alidns.com/dns-query
    nameserver-policy:
      'geosite:ads': rcode://success
      'geosite:fakeip-filter,microsoft-cn,apple-cn,google-cn,games-cn,private,cn': [https://doh.pub/dns-query, https://dns.alidns.com/dns-query]
  ```

  按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

# 四、 [Clash Verge](https://github.com/clash-verge-rev/clash-verge-rev) 设置
1. DNS 模式为 `fake-ip`
进入 Clash Verge -> 订阅，右击“全局扩展配置”，选择“编辑文件”，将 `dns` 部分修改为如下内容并“保存”：

```yaml
dns:
  enable: true
  prefer-h3: true
  ipv6: true
  listen: 0.0.0.0:1053
  fake-ip-range: 198.18.0.1/16
  enhanced-mode: fake-ip
  fake-ip-filter: ['geosite:fakeip-filter']
  nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
```

2. DNS 模式为 `redir-host`
进入 Clash Verge -> 订阅，右击“全局扩展配置”，选择“编辑文件”，将 `dns` 部分修改为如下内容并“保存”：

```yaml
dns:
  enable: true
  prefer-h3: true
  ipv6: true
  listen: 0.0.0.0:1053
  fake-ip-range: 198.18.0.1/16
  enhanced-mode: fake-ip
  fake-ip-filter: ['+.*']
  nameserver:
    - https://dns.google/dns-query
    - https://cloudflare-dns.com/dns-query
  proxy-server-nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
  nameserver-policy:
    'geosite:ads': rcode://success
    'geosite:fakeip-filter,microsoft-cn,apple-cn,google-cn,games-cn,private,cn': [https://doh.pub/dns-query, https://dns.alidns.com/dns-query]
```
---
title: 搭载 sing-boxp 内核进行 DNS 分流教程-geodata 方案
description: 此方案适用于 sing-box，搭载 sing-boxp 内核，采用 `geosite` 和 `geoip` 规则搭配 geosite.db 和 geoip.db 路由规则文件
date: 2024-08-22 18:15:49 +0800
categories: [DNS 配置, DNS 分流]
tags: [sing-box, sing-boxp, geodata, geosite, 进阶, DNS, DNS 分流]
---

注：
- 1. [ShellCrash](https://github.com/juewuy/ShellCrash) 搭配 [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) 并将 AdGuard Home 作为上游时不要使用该方法
- 2. DNS 分流简单来说就是**指定国内域名走国内 DNS 解析，其它域名包括国外域名走 `fake-ip`**
- 3. 所有步骤完成后，请连接 SSH 执行命令 `$CRASHDIR/start.sh restart` 后生效

# 一、 导入路由规则文件
geosite.db 文件须包含 `fakeip-filter` 和 `cn`，推荐导入我定制的[路由规则文件](https://github.com/DustinWin/ruleset_geodata?tab=readme-ov-file#%E4%B8%80-geodata-%E8%A7%84%E5%88%99%E9%9B%86%E6%96%87%E4%BB%B6%E8%AF%B4%E6%98%8E)

# 二、 DNS 分流配置（以 ShellCrash 为例）
1. 进入主菜单 -> 2 内核功能设置 -> 2 切换 DNS 运行模式 -> 4 DNS 进阶设置，将“当前基础 DNS”和“PROXY-DNS”都设置为“null”  
<img src="/assets/img/dns/dns-null.png" alt="ShellCrash 设置" width="60%" />

2. 连接 SSH 后执行命令 `vi $CRASHDIR/jsons/dns.json`，按一下 Ins 键（Insert 键），粘贴如下内容：

```json
{
  "dns": {
    "servers": [
      { "tag": "dns_direct", "address": [ "h3://223.5.5.5/dns-query", "https://1.12.12.12/dns-query" ], "detour": "DIRECT" },
      { "tag": "dns_proxy", "address": [ "h3://8.8.8.8/dns-query", "h3://1.1.1.1/dns-query" ] },
      { "tag": "dns_fakeip", "address": "fakeip" }
    ],
    "rules": [
      { "outbound": "any", "server": "dns_direct" },
      { "clash_mode": "Direct", "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "clash_mode": "Global", "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
      { "geosite": [ "cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" }
    ],
    "final": "dns_direct",
    "strategy": "prefer_ipv4",
    "independent_cache": true,
    "lazy_cache": true,
    "reverse_mapping": true,
    "mapping_override": true,
    "fakeip": {
      "enabled": true,
      "inet4_range": "198.18.0.0/15",
      "inet6_range": "fc00::/18",
      "exclude_rule": {
        "geosite": [ "fakeip-filter" ]
      }
    }
  }
}
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
---
title: 搭载 sing-boxp 内核进行 DNS 分流教程-ruleset方案
description: 此方案适用于 sing-box，搭载 sing-boxp 内核，采用 `rule_set` 规则搭配 .srs 和 .json 规则集文件
date: 2024-08-22 18:24:06 +0800
categories: [DNS 配置, DNS 分流]
tags: [sing-box, sing-boxp, ShellCrash, ruleset, rule_set, 进阶, DNS, DNS 分流]
---

注：
- 1. [ShellCrash](https://github.com/juewuy/ShellCrash) 搭配 [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) 并将 AdGuard Home 作为上游时不要使用该方法
- 2. DNS 分流简单来说就是**指定国内域名走国内 DNS 解析，其它域名包括国外域名走 `fake-ip`**

# 一、 导入规则集合文件
`route.rule_set` 须添加 `fakeip-filter` 和 `cn`，如下：

```json
{
  "route": {
    "rule_set": [
      {
        "tag": "fakeip-filter",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/fakeip-filter.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/fakeip-filter.srs"
      },
      {
        "tag": "cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/cn.srs"
      }
    ]
  }
}
```

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
      { "rule_set": [ "cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
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
        "rule_set": [ "fakeip-filter" ]
      }
    }
  }
}
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
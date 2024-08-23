---
title: Clash Verge 搭载 mihomo 内核的配置-ruleset 方案
description: 此方案适用于 Clash，搭载 mihomo 内核，采用 `RULE-SET` 规则搭配 .yaml、.text 和 .mrs 规则集合文件
date: 2024-08-21 06:27:20 +0800
categories: [工具配置, Clash Verge 配置]
tags: [Clash, mihomo, Clash Verge, ruleset, rule-set, 基础, Windows]
---

---
# 一、 导入配置
## 1. 导入配置文件
- ① 进入 [Clash Verge](https://github.com/clash-verge-rev/clash-verge-rev) -> 订阅，在“订阅文件链接”处粘贴《[生成带有自定义策略组和规则的 Clash 配置文件直链-ruleset 方案](https://proxy-tutorials.dustinwin.top/posts/link-clash-ruleset)》中生成的 .yaml 配置文件直链，然后点击“导入”
- ② 配置文件下载完成后，右击导入的配置文件，选择“编辑信息”，“更新间隔”设置为“1440”，然后“保存”
- ③ 再次右击导入的配置文件，点击“使用”

## 2. 新建自定义配置
进入 Clash Verge -> 订阅，右击“全局扩展配置”，选择“编辑文件”，将原配置全部删除后粘贴如下内容并“保存”：

```yaml
mode: rule
log-level: error
ipv6: true
allow-lan: true
unified-delay: false
tcp-concurrent: true
external-controller-tls: 127.0.0.1:9090
find-process-mode: strict
global-client-fingerprint: chrome

sniffer:
  enable: true
  parse-pure-ip: true
  sniff: {HTTP: {ports: [80, 8080-8880]}, TLS: {ports: [443, 8443]}, QUIC: {ports: [443, 8443]}}
  skip-domain: ['Mijia Cloud']

dns:
  enable: true
  prefer-h3: true
  ipv6: true
  listen: 0.0.0.0:1053
  fake-ip-range: 198.18.0.1/16
  enhanced-mode: fake-ip
  fake-ip-filter: [rule-set:fakeip-filter]
  nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
```

# 二、 启动 Clash
1. 进入 Clash Verge -> 设置 -> 系统设置 -> 服务模式，点击右边的盾牌图标，点击“安装”，完成后启用“服务模式”
2. 进入设置 -> 系统设置 -> Tun 模式，点击右边的螺帽图标，“Tun 堆栈模式”选择“Mixed”（若无法选择，可重启一下软件），然后点击“保存”
3. 进入设置 -> Clash 设置，启用“局域网连接”和“IPv6”
4. 进入设置 -> Clash 设置 -> UWP 工具，点击“Exempt All”后再点击“Save Changes”
5. 进入设置 -> Clash 设置 -> 外部控制，将“外部控制器监听地址”修改为 `127.0.0.1:9090`，然后点击“保存”
6. 进入设置 -> Verge 设置 -> 杂项设置，将“默认测试链接”修改为 `https://www.gstatic.com/generate_204`，然后点击“保存”
7. 进入设置 -> 系统设置，启用“Tun 模式”

# 三、 在线 Dashboard 面板
推荐使用在线 Dashboard 面板 [metacubexd](https://github.com/metacubex/metacubexd)，访问地址：<https://metacubex.github.io/metacubexd>  
首次进入需要添加“后端地址”，输入 `http://127.0.0.1:9090` 并点击“添加”即可访问 Dashboard 面板  
<img src="/assets/img/tools/127-9090-dashboard.png" alt="在线 Dashboard 面板" width="60%" />

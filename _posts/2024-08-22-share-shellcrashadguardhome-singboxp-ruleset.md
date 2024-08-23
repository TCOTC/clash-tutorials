---
title: 分享 ShellCrash 搭载 sing-boxp 内核搭配 AdGuard Home 采用 ruleset 方案的一套配置
description: 此方案适用于 sing-box，搭载 sing-boxp 内核，采用 `rule_set` 规则搭配 .srs 和 .json 规则集文件
date: 2024-08-22 19:51:07 +0800
categories: [分享配置, Router]
tags: [sing-box, sing-boxp, ShellCrash, AdGuard Home, ruleset, rule_set, 分享, Router]
---

# 声明：
1. 此方案采用 [ShellCrash](https://github.com/juewuy/ShellCrash) 作为上游，[AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) 作为下游的模式
2. 请根据自身情况进行修改，**适合自己的方案才是最好的方案**，如无特殊需求，可以照搬
3. 此方案中 ShellCrash 采用了**绕过 CN_IP** 的模式（仍与 AdGuard Home 配合完美）
4. 此方案适用于 ShellCrash（以 arm64 架构为例，且安装路径为 */data/ShellCrash*）
5. 此方案适用于 AdGuard Home（以 arm64 架构为例，且安装路径为 */data/AdGuardHome*）

# 一、 生成配置文件 .json 文件直链
具体方法此处不再赘述，请看《[生成带有自定义出站和规则的 sing-box 配置文件直链-ruleset 方案](https://proxy-tutorials.dustinwin.top/posts/link-singbox-ruleset)》，贴一下我使用的配置：

```json
{
  "outbounds": [
    { "tag": "🚀 节点选择", "type": "selector", "outbounds": [ "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇰🇷 韩国节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点", "🆓 免费节点" ] },
    { "tag": "🐟 漏网之鱼", "type": "selector", "outbounds": [ "🚀 节点选择", "🎯 全球直连" ] },
    { "tag": "📈 网络测试", "type": "selector", "outbounds": [ "🎯 全球直连", "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇰🇷 韩国节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点", "🆓 免费节点" ] },
    { "tag": "🤖 人工智能", "type": "selector", "outbounds": [ "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇰🇷 韩国节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点" ] },
    { "tag": "🎮 游戏服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🪟 微软服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🇬 谷歌服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🍎 苹果服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🇨🇳 直连域名", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🇨🇳 直连 IP", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🪜 代理域名", "type": "selector", "outbounds": [ "🚀 节点选择", "🎯 全球直连" ] },
    { "tag": "📲 电报消息", "type": "selector", "outbounds": [ "🚀 节点选择" ] },
    { "tag": "🔒 私有网络", "type": "selector", "outbounds": [ "🎯 全球直连" ] },
    { "tag": "🎯 全球直连", "type": "selector", "outbounds": [ "DIRECT" ] },
    { "tag": "DIRECT", "type": "direct" },
    { "tag": "GLOBAL", "type": "selector", "outbounds": [ "DIRECT", "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇰🇷 韩国节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点", "🆓 免费节点" ] },
    { "tag": "dns-out", "type": "dns" },
    // 若没有单个出站节点，须删除所有 `🆓 免费节点` 相关内容
    {
      "tag": "🆓 免费节点",
      "type": "vless",
      "server": "example.com",
      "server_port": 443,
      "uuid": "{uuid}",
      "network": "tcp",
      "tls": { "enabled": true, "server_name": "example.com", "insecure": false },
      "transport": { "type": "ws", "path": "/?ed=2048", "headers": { "Host": "example.com" } }
    },
    { "tag": "🇭🇰 香港节点", "type": "urltest", "tolerance": 50, "use_all_providers": true, "includes": [ "🇭🇰" ] },
    { "tag": "🇹🇼 台湾节点", "type": "urltest", "tolerance": 50, "use_all_providers": true, "includes": [ "🇹🇼" ] },
    { "tag": "🇯🇵 日本节点", "type": "urltest", "tolerance": 50, "use_all_providers": true, "includes": [ "🇯🇵" ] },
    { "tag": "🇰🇷 韩国节点", "type": "urltest", "tolerance": 50, "use_all_providers": true, "includes": [ "🇰🇷" ] },
    { "tag": "🇸🇬 新加坡节点", "type": "urltest", "tolerance": 50, "use_all_providers": true, "includes": [ "🇸🇬" ] },
    { "tag": "🇺🇸 美国节点", "type": "urltest", "tolerance": 50, "use_all_providers": true, "includes": [ "🇺🇸" ] }
  ],
  "outbound_providers": [
    {
      "tag": "🛫 我的机场",
      "type": "remote",
      // 修改为你的 Clash 订阅链接
      "download_url": "https://example.com/xxx/xxx&flag=clash",
      "path": "./providers/airport.yaml",
      "download_interval": "24h",
      "download_ua": "clash.meta",
      "includes": [ "🇭🇰|🇹🇼|🇯🇵|🇰🇷|🇸🇬|🇺🇸" ],
      "healthcheck_url": "https://www.gstatic.com/generate_204",
      "healthcheck_interval": "10m"
    }
  ],
  "route": {
    "rules": [
      { "protocol": [ "dns" ], "outbound": "dns-out" },
      { "clash_mode": "Direct", "outbound": "DIRECT" },
      { "clash_mode": "Global", "outbound": "GLOBAL" },
      { "rule_set": [ "private" ], "outbound": "🔒 私有网络" },
      { "rule_set": [ "microsoft-cn" ], "outbound": "🪟 微软服务" },
      { "rule_set": [ "apple-cn" ], "outbound": "🍎 苹果服务" },
      { "rule_set": [ "google-cn" ], "outbound": "🇬 谷歌服务" },
      { "rule_set": [ "games-cn" ], "outbound": "🎮 游戏服务" },
      { "rule_set": [ "ai" ], "outbound": "🤖 人工智能" },
      { "rule_set": [ "networktest" ], "outbound": "📈 网络测试" },
      { "rule_set": [ "proxy" ], "outbound": "🪜 代理域名" },
      { "rule_set": [ "cn" ], "outbound": "🇨🇳 直连域名" },
      { "rule_set": [ "telegramip" ], "outbound": "📲 电报消息", "skip_resolve": true },
      { "rule_set": [ "privateip" ], "outbound": "🔒 私有网络", "skip_resolve": true },
      { "rule_set": [ "cnip" ], "outbound": "🇨🇳 直连 IP" }
    ],
    "rule_set": [
      {
        "tag": "fakeip-filter",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/fakeip-filter.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/fakeip-filter.srs"
      },
      {
        "tag": "private",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/private.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/private.srs"
      },
      {
        "tag": "microsoft-cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/microsoft-cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/microsoft-cn.srs"
      },
      {
        "tag": "apple-cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/apple-cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/apple-cn.srs"
      },
      {
        "tag": "google-cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/google-cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/google-cn.srs"
      },
      {
        "tag": "games-cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/games-cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/games-cn.srs"
      },
      {
        "tag": "ai",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/ai.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/ai.srs"
      },
      {
        "tag": "networktest",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/networktest.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/networktest.srs"
      },
      {
        "tag": "proxy",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/proxy.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/proxy.srs"
      },
      {
        "tag": "cn",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/cn.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/cn.srs"
      },
      {
        "tag": "telegramip",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/telegramip.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/telegramip.srs"
      },
      {
        "tag": "privateip",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/privateip.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/privateip.srs"
      },
      {
        "tag": "cnip",
        "type": "remote",
        "format": "binary",
        "path": "./ruleset/cnip.srs",
        "url": "https://github.com/DustinWin/ruleset_geodata/releases/download/sing-box-ruleset/cnip.srs"
      }
    ],
    "final": "🐟 漏网之鱼",
    "auto_detect_interface": true,
    "concurrent_dial": true
  }
}
```

# 二、 导入 [sing-box PuerNya 版内核](https://github.com/PuerNya/sing-box/tree/building)和 CN_IP 文件
连接 SSH 后执行如下命令：

```shell
curl -L https://cdn.jsdelivr.net/gh/DustinWin/clash_singbox-tools@sing-box/sing-box-puernya-linux-armv8.tar.gz | tar -zx -C /tmp/
curl -o $CRASHDIR/cn_ip.txt -L https://cdn.jsdelivr.net/gh/DustinWin/geoip@ips/cn_ipv4.txt
curl -o $CRASHDIR/cn_ipv6.txt -L https://cdn.jsdelivr.net/gh/DustinWin/geoip@ips/cn_ipv6.txt
crash
```

此时脚本会自动“发现可用的内核文件”，选择 1 加载，后选择 5 Sing-Box-Puer 内核

# 三、 编辑 dns.json 文件
连接 SSH 后执行 `vi $CRASHDIR/jsons/dns.json`，按一下 Ins 键（Insert 键），粘贴如下内容：  
注：
- 1. 由于 ShellCrash 采用的 DNS 模式为 `mix`，**ShellCrash 传给 AdGuard Home 的国外域名对应 IP 为假 IP**，会导致 AdGuard Home 检查更新和下载更新 DNS 黑名单时失败
- 2. `dns.fakeip.exclude_rule` 中添加了 AdGuard Home 常用域名，包括：`adguardteam.github.io`（AdGuard Home 自带 DNS 黑名单下载域名）、`adrules.top`（常用广告拦截下载域名）、`anti-ad.net`（常用广告拦截下载域名）和 `static.adtidy.org`（AdGuard Home 检查更新域名），使这些域名不走 `fakeip`
- 3. 不推荐使用自带更新去更新，推荐第《四》步通过定时任务去自动更新（AdGuard Home 程序已被压缩，节省空间）  
<img src="/assets/img/share/update-adguardhome.png" alt="编辑 dns.json 文件" width="60%" />

```json
{
  "dns": {
    "hosts": {
      "miwifi.com": [ "192.168.31.1" ],
      "doh.pub": [ "1.12.12.12", "120.53.53.53", "2402:4e00::" ],
      "dns.alidns.com": [ "223.5.5.5", "223.6.6.6", "2400:3200::1", "2400:3200:baba::1" ],
      "dns.google": [ "8.8.8.8", "8.8.4.4", "2001:4860:4860::8888", "2001:4860:4860::8844" ],
      "cloudflare-dns.com": [ "1.1.1.1", "1.0.0.1", "2606:4700:4700::1111", "2606:4700:4700::1001" ]
    },
    "servers": [
      { "tag": "dns_direct", "address": [ "https://doh.pub/dns-query", "https://dns.alidns.com/dns-query" ], "detour": "DIRECT" },
      { "tag": "dns_proxy", "address": [ "https://dns.google/dns-query", "https://cloudflare-dns.com/dns-query" ] },
      { "tag": "dns_fakeip", "address": "fakeip" }
    ],
    "rules": [
      { "outbound": "any", "server": "dns_direct" },
      { "clash_mode": "Direct", "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "clash_mode": "Global", "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
      { "rule_set": [ "proxy" ], "query_type": [ "A", "AAAA" ], "server": "dns_fakeip", "rewrite_ttl": 1 },
      { "rule_set": [ "cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "fallback_rules": [ { "rule_set": [ "cnip" ], "server": "dns_direct" }, { "match_all": true, "server": "dns_fakeip", "rewrite_ttl": 1 } ], "server": "dns_proxy" }
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
      "exclude_rule": { "rule_set": [ "fakeip-filter", "private" ] } }
  }
}
```

按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车

# 四、 添加定时任务
1. 连接 SSH 后执行 `vi $CRASHDIR/task/task.user`，按一下 Ins 键（Insert 键），粘贴如下内容：

```shell
201#curl -o /data/ShellCrash/CrashCore.tar.gz -L https://mirror.ghproxy.com/https://github.com/DustinWin/clash_singbox-tools/releases/download/sing-box/sing-box-puernya-linux-armv8.tar.gz && /data/ShellCrash/start.sh restart >/dev/null 2>&1#更新sing-box_PuerNya版内核
202#curl -o /data/ShellCrash/cn_ip.txt -L https://cdn.jsdelivr.net/gh/DustinWin/geoip@ips/cn_ipv4.txt && curl -o /data/ShellCrash/cn_ipv6.txt -L https://cdn.jsdelivr.net/gh/DustinWin/geoip@ips/cn_ipv6.txt >/dev/null 2>&1#更新CN_IP文件
203#curl -o /data/AdGuardHome/AdGuardHome -L https://mirror.ghproxy.com/https://github.com/DustinWin/clash_singbox-tools/releases/download/AdGuardHome/AdGuardHome_prerelease_linux_armv8 && /data/AdGuardHome/AdGuardHome -s restart >/dev/null 2>&1#更新AdGuardHome
```

2. 按一下 Esc 键（退出键），输入英文冒号 `:`，继续输入 `wq` 并回车
3. 执行 `crash`，进入 ShellCrash -> 5 配置自动任务 -> 1 添加自动任务，可以看到末尾就有添加的定时任务，输入对应的数字并回车后可设置执行条件  
<img src="/assets/img/share/task-singbox-adguardhome.png" alt="添加定时任务" width="60%" />

# 五、 新建文件夹
连接 SSH 后执行命令 `mkdir -p $CRASHDIR/providers/ $CRASHDIR/ruleset/`  
注：
- 1. 因 `outbound_providers` 代理集合配置的 `path` 路径中含有文件夹“*providers*”，须手动新建此文件夹才能使 .yaml 订阅文件保存到本地，否则将保存到内存中（每次启动服务都要重新下载）
- 2. 因 `route.rule_set` 代理集合配置的 `path` 路径中含有文件夹“*ruleset*”，须手动新建此文件夹才能使 .srs 规则集文件保存到本地，否则将保存到内存中（每次启动服务都要重新下载）

# 六、 设置部分
## 1. ShellCrash 设置
- ① 设置可参考《[ShellCrash 搭载 sing-boxp 内核的配置-ruleset 方案](https://proxy-tutorials.dustinwin.top/posts/toolsettings-shellcrash-singboxp-ruleset)》，此处只列举配置的不同之处
- ② 进入主菜单 -> 2 内核功能设置，设置如下：  
<img src="/assets/img/share/tproxy-mix-cnip.png" alt="设置部分 1" width="60%" />

- ③ 进入主菜单 -> 2 内核功能设置 -> 1 切换防火墙运行模式 -> 9 ipv6 设置，开启 3 CNV6 绕过内核
- ④ 进入主菜单 -> 2 内核功能设置 -> 2 切换 DNS 运行模式 -> 4 DNS 进阶设置，设置如下：  
<img src="/assets/img/share/close-dns-null.png" alt="设置部分 2" width="60%" />

- ⑤ 进入主菜单 -> 7 内核进阶设置 -> 5 自定义端口及秘钥，设置为 `9090`
- ⑥ 进入主菜单 -> 6 导入配置文件 -> 2 在线获取完整配置文件，粘贴第《一》步中生成的配置文件 .json 文件直链，启动服务即可

## 2. AdGuard Home 设置
设置可参考《[全网最详细的解锁 SSH ShellCrash 搭载 sing-boxp 内核搭配 AdGuard Home 安装和配置教程/AdGuard Home 配置](https://proxy-tutorials.dustinwin.top/posts/pin-shellcrashadguardhome-singbox/2-adguard-home-%E9%85%8D%E7%BD%AE)》（可跳过“添加 DNS 重写”的步骤）

# 七、 在线 Dashboard 面板
推荐使用在线 Dashboard 面板 [metacubexd](https://github.com/metacubex/metacubexd)，访问地址：<https://metacubex.github.io/metacubexd>
1. 需要设置该网站“允许不安全内容”，以 Chrome 浏览器为例，进入设置 -> 隐私和安全 -> 网站设置 -> 更多内容设置 -> 不安全内容（或者直接打开 `chrome://settings/content/insecureContent` 进行设置），在“允许显示不安全内容”内添加 `metacubex.github.io`  
<img src="/assets/img/share/chrome-setting-dashboard.png" alt="在线 Dashboard 面板 1" width="60%" />

2. 首次进入 <https://metacubex.github.io/metacubexd> 需要添加“后端地址”，输入 `http://192.168.31.1:9090` 并点击“添加”即可访问 Dashboard 面板  
<img src="/assets/img/share/192-9090-dashboard.png" alt="在线 Dashboard 面板 2" width="60%" />

# 生成带有自定义策略组和规则的 sing-box 配置文件直链-geodata 方案
- 注：此方案适用于 [sing-box](https://github.com/SagerNet/sing-box)，采用 `GEOSITE` 和 `GEOIP` 规则搭配 geosite.dat 和 geoip.dat（或 Country.mmdb） [路由规则文件](https://github.com/MetaCubeX/meta-rules-dat)
# 前言：
1. 本教程可以生成扩展名为 .json 文件的直链，可以**一键导入使用了 sing-box 内核的客户端**  
如：[ShellCrash](https://github.com/juewuy/ShellCrash) 和 [sing-box for Android and Apple platforms](https://github.com/SagerNet/sing-box/releases) 等，详见[支持 sing-box 的工具](https://sing-box.sagernet.org/zh/clients)
2. 生成的订阅链接地址不会改变，支持更新订阅，**支持国内访问，支持同步机场节点**
3. 生成的订阅链接**自带规则集**，规则集来源 [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat)
4. **在 .json 文件中不支持以 `#` 开头的注释，本教程为方便初学者理解，特意增加了“注释”，在导入 sing-box 前必须删除干净**
5. 目前 sing-box 内核不支持类似 [Clash.Meta 内核](https://github.com/MetaCubeX/mihomo)的 `proxy-providers` 功能，需要手动导入节点：
① 前往[肥羊在线订阅转换工具](https://suburl.v1.mk)粘贴订阅链接，“生成类型”选择“Sing-Box”，其它参数保持默认即可，点击“生成订阅链接”
② 使用浏览器打开复制的订阅链接，复制 `"outbounds"` 内的所有节点配置，范围参考：
```
# 开始部分
{"type":"trojan","tag":"🇭🇰 香港 xxx 节点 1"

# 结束部分
"network":"tcp","tcp_fast_open":false}
```
③ 粘贴到 [VS Code](https://code.visualstudio.com/Download) 等专业文本编辑器中，右击输入区域，点击“格式化文档”  
④ 将格式化后的节点配置粘贴到 .json 文件 `"outbounds"` 的 `此处粘贴节点配置` 中  
⑤ 根据粘贴的节点配置中的国家或地区节点，配置 `url-test` 类型的出站，以 `🇭🇰 香港节点` 为例：
```
{
  "outbounds": [
    { "tag": "🚀 节点选择", "type": "selector", "outbounds": [ "🇭🇰 香港节点" ] },
    { "tag": "🇭🇰 香港节点", "type": "url-test", "outbounds": [ "🇭🇰 香港 xxx 节点 1", "🇭🇰 香港 xxx 节点 2" ], "url": "https://www.gstatic.com/generate_204", "interval": "10m", "tolerance": 100 },
    {
      "type": "trojan",
      "tag": "🇭🇰 香港 xxx 节点 1",
      "server": "example.com",
      "server_port": 12345,
      "password": "{password}",
      "tls": {
        "enabled": true,
        "server_name": "example.com",
        "insecure": true
      }
    },
    {
      "type": "trojan",
      "tag": "🇭🇰 香港 xxx 节点 2",
      "server": "example.com",
      "server_port": 54321,
      "password": "{password}",
      "tls": {
        "enabled": true,
        "server_name": "example.com",
        "insecure": true
      }
    }
  ]
}
```
---
# 一、 注册 [Gist](https://gist.github.com)
进入 https://gist.github.com 网站并注册
# 二、 准备编辑 .json 文件
## 1. 打开编辑页面
登录并打开 Gist 可以直接编辑文件，或者鼠标点击页面右上角头像左边的“+”图标新建文件
## 2. 输入描述和完整文件名
“Gist description...”输入描述，随意填写；“Filename including extension...”输入完整文件名**包括扩展名**，如 singboxlink.json  
<img src="https://github.com/DustinWin/clash_singbox-tutorials/assets/45238096/13346166-85cf-474c-9da7-55182e095758" width="60%"/>
# 三、 编辑 .json 文件
## 1. 选择规则集模式
① 白名单模式（没有命中规则的网络流量统统使用代理，适用于服务器线路网络质量稳定、快速，不缺服务器流量的用户）
```
{
  # 日志
  "log": { "level": "error", "timestamp": true },
  # DNS
  "dns": {
    # DNS 服务器
    "servers": [
      # 广告 DNS
      { "tag": "dns_block", "address": "rcode://success" },
      # 国外 DNS
      { "tag": "dns_proxy", "address": "https://dns.google/dns-query", "address_resolver": "dns_ip" },
      # 国内 DNS
      { "tag": "dns_direct", "address": "h3://dns.alidns.com/dns-query", "address_resolver": "dns_ip", "detour": "DIRECT" },
      # FakeIP
      { "tag": "dns_fakeip", "address": "fakeip" },
      # IP 格式的 DNS
      { "tag": "dns_ip", "address": "https://223.5.5.5/dns-query", "detour": "DIRECT" }
    ],
    # DNS 规则
    "rules": [
      { "outbound": "any", "server": "dns_ip" },
      { "clash_mode": "global", "server": "dns_fakeip" },
      { "clash_mode": "direct", "server": "dns_direct" },
      # geosite.db 规则集文件内必须包含 `category-ads-all` 规则
      { "geosite": "category-ads-all", "server": "dns_block" },
      # geosite.db 规则集文件内必须包含 `geolocation-!cn` 规则
      { "geosite": "geolocation-!cn", "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" },
      # geosite.db 规则集文件内必须包含 `cn` 规则
      { "geosite": "cn", "query_type": [ "A", "AAAA" ], "server": "dns_direct" }
    ],
    # 默认 DNS 服务器，即上述 DNS 规则外的域名使用该 DNS 解析
    "final": "dns_direct",
    "strategy": "prefer_ipv4",
    "independent_cache": true,
    "reverse_mapping": true,
    "fakeip": { "enabled": true, "inet4_range": "198.18.0.0/15", "inet6_range": "fc00::/18" }
  },
  "ntp": { "enabled": true, "server": "time.apple.com", "server_port": 123, "interval": "30m" },
  "inbounds": [
    { "tag": "mixed-in", "type": "mixed", "listen": "::", "listen_port": 7890, "sniff": false },
    { "tag": "dns_in", "type": "direct", "listen": "::", "listen_port": 1053, "sniff": true, "sniff_override_destination": false },
    { "tag": "redirect-in", "type": "redirect", "listen": "::", "listen_port": 7892, "sniff": true, "sniff_override_destination": true },
    { "tag": "tproxy-in", "type": "tproxy", "listen": "::", "listen_port": 7893, "sniff": true, "sniff_override_destination": true },
    { "tag": "tun-in", "type": "tun", "inet4_address": "172.19.0.1/30", "inet6_address": "fdfe:dcba:9876::1/126", "mtu": 9000, "auto_route": true, "strict_route": true, "stack": "mixed", "sniff": true, "sniff_override_destination": true }
  ],
  "outbounds": [
    # 手动选择国家或地区节点；根据“国家或地区出站”的名称对 `outbounds` 值进行增删改，须一一对应
    { "tag": "🚀 节点选择", "type": "selector", "outbounds": [ "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇰🇷 韩国节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点" ] },
    # Speedtest 测速网站：选择`🎯 全球直连` 为测试本地网络速度（运营商网络速度），可选择其它节点用于测试机场节点速度
    { "tag": "📈 网络测速", "type": "selector", "outbounds": [ "🎯 全球直连", "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇰🇷 韩国节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点" ] },
    { "tag": "🔗 直连域名", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🪜 代理域名", "type": "selector", "outbounds": [ "🚀 节点选择", "🎯 全球直连" ] },
    { "tag": "🎮 游戏平台", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "Ⓜ️ 微软服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "📢 谷歌服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🍎 苹果服务", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "🇨🇳 国内 IP", "type": "selector", "outbounds": [ "🎯 全球直连", "🚀 节点选择" ] },
    { "tag": "📲 电报消息", "type": "selector", "outbounds": [ "🚀 节点选择" ] },
    { "tag": "🔒 私有网络", "type": "selector", "outbounds": [ "🎯 全球直连" ] },
    { "tag": "🛑 广告拦截", "type": "selector", "outbounds": [ "REJECT" ] },
    { "tag": "🎯 全球直连", "type": "selector", "outbounds": [ "DIRECT" ] },
    { "tag": "REJECT", "type": "block" },
    # 本地网络有 IPv6 时可以配置 `"domain_strategy": "prefer_ipv6"`
    { "tag": "DIRECT", "type": "direct", "domain_strategy": "prefer_ipv6" },
    { "tag": "dns-out", "type": "dns" },

    # -------------------- 国家或地区出站 --------------------
    # 根据粘贴的节点配置中的国家或地区节点，配置 `url-test` 类型的出站；自动选择节点，即按照 url 测试结果使用延迟最低的节点；测试后容差大于 100ms 才会切换到延迟低的那个节点
    { "tag": "🇭🇰 香港节点", "type": "url-test", "outbounds": [ "🇭🇰 香港 xxx 节点 1", "🇭🇰 香港 xxx 节点 2" ], "url": "https://www.gstatic.com/generate_204", "interval": "10m", "tolerance": 100 },
    { "tag": "🇹🇼 台湾节点", "type": "url-test", "outbounds": [ "🇹🇼 台湾 xxx 节点 1", "🇹🇼 台湾 xxx 节点 2" ], "url": "https://www.gstatic.com/generate_204", "interval": "10m", "tolerance": 100 },
    { "tag": "🇯🇵 日本节点", "type": "url-test", "outbounds": [ "🇯🇵 日本 xxx 节点 1", "🇯🇵 日本 xxx 节点 2" ], "url": "https://www.gstatic.com/generate_204", "interval": "10m", "tolerance": 100 },
    { "tag": "🇰🇷 韩国节点", "type": "url-test", "outbounds": [ "🇰🇷 韩国 xxx 节点 1", "🇰🇷 韩国 xxx 节点 2" ], "url": "https://www.gstatic.com/generate_204", "interval": "10m", "tolerance": 100 },
    { "tag": "🇸🇬 新加坡节点", "type": "url-test", "outbounds": [ "🇸🇬 新加坡 xxx 节点 1", "🇸🇬 新加坡 xxx 节点 2" ], "url": "https://www.gstatic.com/generate_204", "interval": "10m", "tolerance": 100 },
    { "tag": "🇺🇸 美国节点", "type": "url-test", "outbounds": [ "🇺🇸 美国 xxx 节点 1", "🇺🇸 美国 xxx 节点 2" ], "url": "https://www.gstatic.com/generate_204", "interval": "10m", "tolerance": 100 },

    # 此处粘贴节点配置
  ],
  "route": {
    "rules": [
      { "protocol": "dns", "outbound": "dns-out" },
      { "clash_mode": "global", "outbound": "🚀 节点选择" },
      { "clash_mode": "direct", "outbound": "DIRECT" },
      # 自定义规则优先放前面
      { "geosite": "category-ads-all", "outbound": "🛑 广告拦截" },
      # 为过滤 P2P 流量（BT 下载），可添加一条 `port_range` 规则
      { "port_range": 6881-6889, "🎯 全球直连"},
      { "geosite": "private", "outbound": "🔒 私有网络" },
      { "geosite": "microsoft@cn", "outbound": "Ⓜ️ 微软服务" },
      { "geosite": "apple-cn", "outbound": "🍎 苹果服务" },
      { "geosite": "google-cn", "outbound": "📢 谷歌服务" },
      { "geosite": "category-games@cn", "outbound": "🎮 游戏平台" },
      { "geosite": "speedtest", "outbound": "📈 网络测速" },
      { "geosite": "geolocation-!cn", "outbound": "🪜 代理域名" },
      { "geosite": "cn", "outbound": "🔗 直连域名" },
      { "geoip": "telegram", "outbound": "📲 电报消息" },
      { "geoip": "private",  "outbound": "🔒 私有网络" },
      { "geoip": "cn", "outbound": "🇨🇳 国内 IP" }
    ],
    "geosite": {
      "download_url": "https://fastly.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@release/geosite.db",
      "download_detour": "DIRECT"
    },
    "geoip": {
      "download_url": "https://fastly.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@release/geoip.db",
      "download_detour": "DIRECT"
    },
    # 默认出站，相当于 Clash 的 `漏网之鱼`，即没有命中规则的域名或 IP 走该规则
    "final": "🚀 节点选择",
    "auto_detect_interface": true
  },
  "experimental": {
    "cache_file": { "enabled": true, "cache_id": "", "store_fakeip": true },
    # 面板配置
    "clash_api": {
      "external_controller": "127.0.0.1:9090",
      "external_ui": "ui",
      "external_ui_download_url": "https://gh.dustinwin.top/https://github.com/MetaCubeX/metacubexd/archive/refs/heads/gh-pages.zip",
      "external_ui_download_detour": "DIRECT",
      "secret": "",
      "default_mode": "Rule"
    }
  }
}
```
将模板内容复制到自己 Gist 新建的 .json 文件中  
**贴一张面板效果图（举个例子：我手动选择 `🇹🇼 台湾节点` 策略组，而该策略组是将机场内所有台湾节点按照 url 测试结果自动选择延迟最低的台湾节点）：**  
<img src="https://github.com/DustinWin/clash_singbox-tutorials/assets/45238096/1c1a1866-1fc1-4277-92b7-d138e36a4a4b" width="60%"/>  
② 黑名单模式（只有命中规则的网络流量才使用代理，适用于服务器线路网络质量不稳定或不够快，或服务器流量紧缺的用户。通常也是软路由用户、家庭网关用户的常用模式）
```
{
  # 日志
  "log": { "level": "error", "timestamp": true },
  # DNS
  "dns": {
    # DNS 服务器
    "servers": [
      # 广告 DNS
      { "tag": "dns_block", "address": "rcode://success" },
      # 国外 DNS
      { "tag": "dns_proxy", "address": "https://dns.google/dns-query", "address_resolver": "dns_ip" },
      # 国内 DNS
      { "tag": "dns_direct", "address": "h3://dns.alidns.com/dns-query", "address_resolver": "dns_ip", "detour": "DIRECT" },
      # FakeIP
      { "tag": "dns_fakeip", "address": "fakeip" },
      # IP 格式的 DNS
      { "tag": "dns_ip", "address": "https://223.5.5.5/dns-query", "detour": "DIRECT" }
    ],
    # DNS 规则
    "rules": [
      { "outbound": "any", "server": "dns_ip" },
      { "clash_mode": "global", "server": "dns_fakeip" },
      { "clash_mode": "direct", "server": "dns_direct" },
      # geosite.db 规则集文件内必须包含 `category-ads-all` 规则
      { "geosite": "category-ads-all", "server": "dns_block" },
      # geosite.db 规则集文件内必须包含 `gfw` 规则
      { "geosite": "gfw", "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" }
    ],
    # 默认 DNS 服务器，即上述 DNS 规则外的域名使用该 DNS 解析
    "final": "dns_direct",
    "strategy": "prefer_ipv4",
    "independent_cache": true,
    "reverse_mapping": true,
    "fakeip": { "enabled": true, "inet4_range": "198.18.0.0/15", "inet6_range": "fc00::/18" }
  },
  "ntp": { "enabled": true, "server": "time.apple.com", "server_port": 123, "interval": "30m" },
  "inbounds": [
    { "tag": "mixed-in", "type": "mixed", "listen": "::", "listen_port": 7890, "sniff": false },
    { "tag": "dns_in", "type": "direct", "listen": "::", "listen_port": 1053, "sniff": true, "sniff_override_destination": false },
    { "tag": "redirect-in", "type": "redirect", "listen": "::", "listen_port": 7892, "sniff": true, "sniff_override_destination": true },
    { "tag": "tproxy-in", "type": "tproxy", "listen": "::", "listen_port": 7893, "sniff": true, "sniff_override_destination": true },
    { "tag": "tun-in", "type": "tun", "inet4_address": "172.19.0.1/30", "inet6_address": "fdfe:dcba:9876::1/126", "mtu": 9000, "auto_route": true, "strict_route": true, "stack": "mixed", "sniff": true, "sniff_override_destination": true }
  ],
  "outbounds": [
    # 手动选择国家或地区节点；根据“国家或地区出站”的名称对 `outbounds` 值进行增删改，须一一对应
    { "tag": "🚀 节点选择", "type": "selector", "outbounds": [ "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇰🇷 韩国节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点" ] },
    # Speedtest 测速网站：选择`🎯 全球直连` 为测试本地网络速度（运营商网络速度），可选择其它节点用于测试机场节点速度
    { "tag": "📈 网络测速", "type": "selector", "outbounds": [ "🎯 全球直连", "🇭🇰 香港节点", "🇹🇼 台湾节点", "🇯🇵 日本节点", "🇰🇷 韩国节点", "🇸🇬 新加坡节点", "🇺🇸 美国节点" ] },
    { "tag": "🧱 GFWList 域名", "type": "selector", "outbounds": [ "🚀 节点选择", "🎯 全球直连" ] },
    { "tag": "📲 电报消息", "type": "selector", "outbounds": [ "🚀 节点选择" ] },
    { "tag": "🛑 广告拦截", "type": "selector", "outbounds": [ "REJECT" ] },
    { "tag": "🎯 全球直连", "type": "selector", "outbounds": [ "DIRECT" ] },
    { "tag": "REJECT", "type": "block" },
    # 本地网络有 IPv6 时可以配置 `"domain_strategy": "prefer_ipv6"`
    { "tag": "DIRECT", "type": "direct", "domain_strategy": "prefer_ipv6" },
    { "tag": "dns-out", "type": "dns" },

    # -------------------- 国家或地区出站 --------------------
    # 根据粘贴的节点配置中的国家或地区节点，配置 `url-test` 类型的出站；自动选择节点，即按照 url 测试结果使用延迟最低的节点；测试后容差大于 100ms 才会切换到延迟低的那个节点
    { "tag": "🇭🇰 香港节点", "type": "url-test", "outbounds": [ "🇭🇰 香港 xxx 节点 1", "🇭🇰 香港 xxx 节点 2" ], "url": "https://www.gstatic.com/generate_204", "interval": "10m", "tolerance": 100 },
    { "tag": "🇹🇼 台湾节点", "type": "url-test", "outbounds": [ "🇹🇼 台湾 xxx 节点 1", "🇹🇼 台湾 xxx 节点 2" ], "url": "https://www.gstatic.com/generate_204", "interval": "10m", "tolerance": 100 },
    { "tag": "🇯🇵 日本节点", "type": "url-test", "outbounds": [ "🇯🇵 日本 xxx 节点 1", "🇯🇵 日本 xxx 节点 2" ], "url": "https://www.gstatic.com/generate_204", "interval": "10m", "tolerance": 100 },
    { "tag": "🇰🇷 韩国节点", "type": "url-test", "outbounds": [ "🇰🇷 韩国 xxx 节点 1", "🇰🇷 韩国 xxx 节点 2" ], "url": "https://www.gstatic.com/generate_204", "interval": "10m", "tolerance": 100 },
    { "tag": "🇸🇬 新加坡节点", "type": "url-test", "outbounds": [ "🇸🇬 新加坡 xxx 节点 1", "🇸🇬 新加坡 xxx 节点 2" ], "url": "https://www.gstatic.com/generate_204", "interval": "10m", "tolerance": 100 },
    { "tag": "🇺🇸 美国节点", "type": "url-test", "outbounds": [ "🇺🇸 美国 xxx 节点 1", "🇺🇸 美国 xxx 节点 2" ], "url": "https://www.gstatic.com/generate_204", "interval": "10m", "tolerance": 100 },

    # 此处粘贴节点配置
  ],
  "route": {
    "rules": [
      { "protocol": "dns", "outbound": "dns-out" },
      { "clash_mode": "global", "outbound": "🚀 节点选择" },
      { "clash_mode": "direct", "outbound": "DIRECT" },
      # 自定义规则优先放前面
      { "geosite": "category-ads-all", "outbound": "🛑 广告拦截" },
      { "geosite": "speedtest", "outbound": "📈 网络测速" },
      { "geosite": "gfw", "outbound": "🧱 GFWList 域名" },
      { "geoip": "telegram", "outbound": "📲 电报消息" },
    ],
    "geosite": {
      "download_url": "https://fastly.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@release/geosite.db",
      "download_detour": "DIRECT"
    },
    "geoip": {
      "download_url": "https://fastly.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@release/geoip.db",
      "download_detour": "DIRECT"
    },
    # 默认出站，相当于 Clash 的 `漏网之鱼`，即没有命中规则的域名或 IP 走该规则
    "final": "🎯 全球直连",
    "auto_detect_interface": true
  },
  "experimental": {
    "cache_file": { "enabled": true, "cache_id": "", "store_fakeip": true },
    # 面板配置
    "clash_api": {
      "external_controller": "127.0.0.1:9090",
      "external_ui": "ui",
      "external_ui_download_url": "https://gh.dustinwin.top/https://github.com/MetaCubeX/metacubexd/archive/refs/heads/gh-pages.zip",
      "external_ui_download_detour": "DIRECT",
      "secret": "",
      "default_mode": "Rule"
    }
  }
}
```
将模板内容复制到自己 Gist 新建的 .json 文件中
## 2. 修改模板
① 首先确定自己机场中有哪些国家或地区的节点，然后对模板文件中“**国家或地区出站**”和 `🚀 节点选择` 出站下的 `outbounds` 里面的国家或地区进行增删改
- 注：两者中的国家或地区必须一一对应，新增就全部新增，删除就全部删除，修改就全部修改（重要）

② 在出站里的 `此处粘贴节点配置` 粘贴自己机场的节点配置（详见《前言：5》）  
③ 在 `🚀 节点选择` 出站下的 `outbounds` 里，可以将最稳定的节点放在最前面，配置完成后会自动选择最稳定的节点  
⑤ 在“国家或地区出站”里，`type` 为 `url-test` 就是自动选择延迟最低的节点，将 `url-test` 改成 `selector` 就是手动选择节点  
举个例子：我的机场有 2 个节点，分别是香港节点和日本节点，我想让[哔哩哔哩](https://www.bilibili.com)（B 站）自动选择延迟最低的香港节点，[AcFun](https://www.acfun.cn)（A 站）可以手动选择日本节点，这个需求怎么写？  
我们可以进入 [v2fly/domain-list-community/data](https://github.com/v2fly/domain-list-community/tree/master/data) 后按 Ctrl+F 组合键搜索“bilibili”和“acfun”，显然可以**精确搜索到结果**，那么就可以这样编写：
- 注：以下只是节选，请酌情套用

```
{
  "outbounds": [
    # 默认选择香港节点
    { "tag": "📺 哔哩哔哩", "type": "selector", "outbounds": [ "🇭🇰 香港节点", "🎯 全球直连" ] },

    # 默认选择日本节点
    { "tag": "📽️ AcFun", "type": "selector", "outbounds": [ "🇯🇵 日本节点", "🎯 全球直连" ] },

    # 自动选择延迟最低的香港节点；容差大于 100ms 才会切换到延迟低的那个节点；未选择到当前策略组时不会进行延迟测试
    { "tag": "🇭🇰 香港节点", "type": "urltest", "outbounds": [ "🇭🇰 香港 xxx 节点 1", "🇭🇰 香港 xxx 节点 2" ], "url": "https://www.gstatic.com/generate_204", "interval": "10m", "tolerance": 100 },

    # 手动选择日本节点
    { "tag": "🇯🇵 日本节点", "type": "selector", "outbounds": [ "🇯🇵 日本 xxx 节点 1", "🇯🇵 日本 xxx 节点 2", "🎯 全球直连" ] },

    { "tag": "🎯 全球直连", "type": "selector", "outbounds": [ "DIRECT" ] },
    { "tag": "DIRECT", "type": "direct", "domain_strategy": "prefer_ipv6" },

    # 此处粘贴香港和日本节点配置
  ],
  "route": {
    "rules": [
      # 自定义规则优先放前面
      { "geosite": "bilibili", "outbound": "📺 哔哩哔哩" },
      { "geosite": "acfun", "outbound": "📽️ AcFun" }
    ]
  }
}
```
# 四、 生成 .json 文件链接
编辑完成后，点击右下角的“Create secret gist”按钮，然后点击右上角的“Raw”按钮  
<img src="https://github.com/DustinWin/clash_singbox-tutorials/assets/45238096/de55458c-b403-438c-adf9-2503f1aad71f" width="60%"/>  
取出地址栏中的网址，删除后面的一串随机码，**完成后该 .json 文件直链才是最终生成的订阅链接**，该订阅链接地址不会改变，在不更改文件名的情况下即使编辑该 .json 文件并提交了 n 次也不会改变  
举例，这是原地址：  
`https://gist.githubusercontent.com/DustinWin/40c0611fda5d6fcd0795ee5a15de7c73/raw/b3c63051134510ae9825068bbcf5219817761f57/singboxlink.json`  
删除后面的一串随机码（当前编辑该文件生成的随机码“b3c63051134510ae9825068bbcf5219817761f57”）  
<img src="https://github.com/DustinWin/clash_singbox-tutorials/assets/45238096/a409ace5-8f03-4776-b9e3-dce3ba804844" width="60%"/>  
删除后变成：  
`https://gist.githubusercontent.com/DustinWin/40c0611fda5d6fcd0795ee5a15de7c73/raw/singboxlink.json`
# 五、 导入订阅链接
## 1. 在 ShellCrash 中导入订阅链接
进入 ShellCrash 配置脚本->6->2，粘贴最终生成的订阅链接即可，具体设置请参考《[ShellCrash 配置-geodata 方案](https://github.com/DustinWin/clash_singbox-tutorials/blob/main/%E6%95%99%E7%A8%8B%E5%90%88%E9%9B%86/%E5%9F%BA%E7%A1%80%E7%AF%87/ShellCrash%20%E9%85%8D%E7%BD%AE-geodata%20%E6%96%B9%E6%A1%88.md)》
## 2. 在 Clash Verge（Windows 端）中导入订阅链接
进入 Clash Verge->订阅，在“订阅文件链接”处粘贴最终生成的订阅链接，直接“导入”即可，具体设置请参考《[Clash Verge 配置-geodata 方案](https://github.com/DustinWin/clash_singbox-tutorials/blob/main/%E6%95%99%E7%A8%8B%E5%90%88%E9%9B%86/%E5%9F%BA%E7%A1%80%E7%AF%87/Clash%20Verge%20%E9%85%8D%E7%BD%AE-geodata%20%E6%96%B9%E6%A1%88.md)》
# 六、 私人定制
到了这里，相信你对里面的机制已经有了一定的认识，那么我们可以对自己的需求进行定制了  
最常见的有：我购买的机场支持[奈飞](https://www.netflix.com)和[亚马逊](https://www.primevideo.com)，但仅新加坡这一个节点支持亚马逊，日本和韩国节点支持奈飞，这个规则怎么写？
1. 进入 [v2fly/domain-list-community/data](https://github.com/v2fly/domain-list-community/tree/master/data) 后按 Ctrl+F 组合键分别搜索“netflix”和“primevideo”
2. 进入 [Loyalsoldier/geoip/text](https://github.com/Loyalsoldier/geoip/tree/release/text) 后按 Ctrl+F 组合键分别搜索“netflix”和“primevideo”
3. 在 v2fly/domain-list-community/data 中可以**精确搜索**到“netflix”和“primevideo”，但在 Loyalsoldier/geoip/text 中只能**精确搜索**到“netflix”，那么我们可以这样编写：
- 注：以下只是节选，请酌情套用

```
{
  "outbounds": [
    # 打开奈飞后手动选择日本或韩国节点
    { "tag": "🎥 奈飞视频", "type": "selector", "outbounds": [ "🇯🇵 日本节点", "🇰🇷 韩国节点" ] },

  # 打开亚马逊后自动选择延迟最低的新加坡节点；容差大于 100ms 才会切换到延迟低的那个节点；未选择到当前策略组时不会进行延迟测试
    { "tag": "🎬 Prime Video", "type": "selector", "outbounds": [ "🇸🇬 新加坡节点" ] },

    { "tag": "🇯🇵 日本节点", "type": "selector", "outbounds": [ "🇯🇵 日本 xxx 节点 1", "🇯🇵 日本 xxx 节点 2" ] },
    { "tag": "🇰🇷 韩国节点", "type": "selector", "outbounds": [ "🇰🇷 韩国 xxx 节点 1", "🇰🇷 韩国 xxx 节点 2" ] },
    { "tag": "🇸🇬 新加坡节点", "type": "urltest", "outbounds": [ "🇸🇬 新加坡 xxx 节点 1", "🇸🇬 新加坡 xxx 节点 2" ], "url": "https://www.gstatic.com/generate_204", "interval": "10m", "tolerance": 100 },

    # 此处粘贴日本、韩国和新加坡节点配置
  ],
  "route": {
    "rules": [
      # 自定义规则优先放前面
      { "geosite": "netflix", "outbound": "🎥 奈飞视频" },
      { "geoip": "netflix", "outbound": "🎥 奈飞视频" }
      { "geosite": "primevideo", "outbound": "🎬 Prime Video" }
    ]
  }
}
```

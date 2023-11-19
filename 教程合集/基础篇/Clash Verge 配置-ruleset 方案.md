# [Clash Verge](https://github.com/zzzgydi/clash-verge)（Windows 端）配置-ruleset 方案
- 注：此方案采用 `RULE-SET` 规则搭配 `rule-providers` 配置项
---
# 一、 设置部分
1. 进入 Clash Verge->设置->Verge 设置->语言设置，可切换到“中文”
2. 进入设置->Clash 设置->Clash 内核，点击“螺帽图标”并切换至“[Clash Meta 内核](https://github.com/MetaCubeX/Clash.Meta)”
3. 进入设置->系统设置->服务模式，点击“盾牌图标”，“INSTALL”即可
# 二、 导入或更新 Clash Meta 内核
以管理员身份运行 CMD，执行如下命令：
```
taskkill /f /t /im "Clash Verge*"
taskkill /f /t /im clash-meta*
curl -o %PROGRAMFILES%\Clash Verge\clash-meta.exe -L https://ghproxy.com/https://raw.githubusercontent.com/DustinWin/clash-tools/main/Clash.Meta-release/clash.meta-windows-amd64.exe
```
# 三、 导入配置
## 1. 导入配置文件
① 进入 Clash Verge->配置，在“配置文件链接”处粘贴《[生成带有自定义策略组和规则的 yaml 配置文件直链-ruleset 方案](https://github.com/DustinWin/clash-tutorials/blob/main/%E6%95%99%E7%A8%8B%E5%90%88%E9%9B%86/%E5%9F%BA%E7%A1%80%E7%AF%87/%E7%94%9F%E6%88%90%E5%B8%A6%E6%9C%89%E8%87%AA%E5%AE%9A%E4%B9%89%E7%AD%96%E7%95%A5%E7%BB%84%E5%92%8C%E8%A7%84%E5%88%99%E7%9A%84%20yaml%20%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E7%9B%B4%E9%93%BE-ruleset%20%E6%96%B9%E6%A1%88.md)》中生成的配置文件 .yaml 文件直链  
② 右击导入的配置文件，选择“编辑信息”，“更新间隔”设置为“1440”，然后“保存”
## 2. 新建自定义配置
① 进入 Clash Verge->配置，点击“新建”，类型选择“Merge”，完成后点击“保存”，右击新建的 Merge 文件，选择“启用”  
② 再次右击新建的 Merge 文件，选择“编辑文件”，粘贴如下内容并“保存”：
```
dns:
  enable: true
  prefer-h3: true
  ipv6: true
  listen: 0.0.0.0:1053
  use-hosts: true
  fake-ip-range: 198.18.0.1/16
  enhanced-mode: fake-ip
  fake-ip-filter: ['+.*']
  default-nameserver:
    - https://1.12.12.12/dns-query
    - https://223.5.5.5/dns-query
  nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
  fallback:
    - https://dns.google.com/dns-query
    - https://dns.cloudflare.com/dns-query
    - https://doh.opendns.com/dns-query
```
# 四、 启动 Clash
1. 进入 Clash Verge->设置->Clash 设置->Clash 字段，勾选带有感叹号的字段，“保存”即可
2. 进入设置->系统设置->Tun 模式，启用
# 五、 在线 Dashboard 面板
推荐使用在线面板 [Yacd-meta](https://github.com/MetaCubeX/Yacd-meta)，访问地址：https://yacd.metacubex.one  
1. 需要设置该网站“允许不安全内容”，以 [Chrome 浏览器](https://www.google.com/chrome)为例，进入设置-->隐私和安全-->网站设置-->更多内容设置-->不安全内容（或者直接打开 `chrome://settings/content/insecureContent` 进行设置），在“允许显示不安全内容”内添加 `https://yacd.metacubex.one`  
<img src="https://user-images.githubusercontent.com/45238096/235448980-52331db5-6b9f-4b0c-a876-1509d34db51a.png" width="60%"/>

2. 首次进入 https://yacd.metacubex.one 需要添加“后端地址”，输入 `http://192.168.31.1:9090` 并点击“添加”即可访问 Dashboard 面板  
<img src="https://github.com/DustinWin/clash-tutorials/assets/45238096/67d5caee-e6a8-42d9-94d2-3f707dc20e14" width="60%"/>

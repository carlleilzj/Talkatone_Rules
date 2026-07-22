# Talkatone 全面分流规则

适用于 **Quantumult X / Surge / Clash(Mihomo) / Loon**。  
聚合社区常用域名与通话 IP，并附带广告拦截列表。

## 链接

| 资源 | URL |
|------|-----|
| 仓库 | https://github.com/carlleilzj/Talkatone_Rules |
| **QX 完整片段** | https://raw.githubusercontent.com/carlleilzj/Talkatone_Rules/main/QuantumultX/Talkatone.conf |
| QX 分流 list | https://raw.githubusercontent.com/carlleilzj/Talkatone_Rules/main/QuantumultX/Talkatone.list |
| QX 广告 list | https://raw.githubusercontent.com/carlleilzj/Talkatone_Rules/main/QuantumultX/Talkatone_Ads.list |
| Surge 分流 | https://raw.githubusercontent.com/carlleilzj/Talkatone_Rules/main/Surge/Talkatone.list |
| Surge 广告 | https://raw.githubusercontent.com/carlleilzj/Talkatone_Rules/main/Surge/Talkatone_Ads.list |
| Surge 模块 | https://raw.githubusercontent.com/carlleilzj/Talkatone_Rules/main/Surge/Talkatone.sgmodule |
| Clash 分流 | https://raw.githubusercontent.com/carlleilzj/Talkatone_Rules/main/Clash/Talkatone.yaml |
| Clash 广告 | https://raw.githubusercontent.com/carlleilzj/Talkatone_Rules/main/Clash/Talkatone_Ads.yaml |
| 通用 DOMAIN 列表 | https://raw.githubusercontent.com/carlleilzj/Talkatone_Rules/main/Talkatone.list |

## Quantumult X（推荐）

### 1. 策略组

在配置 `[policy]` 增加（节点名改成你自己的）：

```ini
static=Talkatone, 美国节点1, 美国节点2, proxy, direct
```

或按地区自动测速：

```ini
url-latency-benchmark=Talkatone, server-tag-regex=(?i)(美|US|United|Los Angeles|San Jose|Chicago|Seattle|Dallas), check-interval=600, tolerance=50
```

> 注册 / 登录 / 收发短信 / 通话：优先 **美国住宅或原生 IP** 节点。部分机房 IP 会被风控。

### 2. 远程规则

```ini
[filter_remote]
https://raw.githubusercontent.com/carlleilzj/Talkatone_Rules/main/QuantumultX/Talkatone_Ads.list, tag=Talkatone广告, update-interval=86400, opt-parser=true, enabled=true
https://raw.githubusercontent.com/carlleilzj/Talkatone_Rules/main/QuantumultX/Talkatone.list, tag=Talkatone分流, update-interval=86400, opt-parser=true, enabled=true
```

顺序建议：**广告 list 在分流 list 上面**；二者都在 `GEOIP,CN` / `FINAL` 之前。

### 3. 手动粘贴

见 [QuantumultX/Talkatone.conf](QuantumultX/Talkatone.conf) 的 `[filter_local]` 段。

## Surge

```ini
[Rule]
RULE-SET,https://raw.githubusercontent.com/carlleilzj/Talkatone_Rules/main/Surge/Talkatone_Ads.list,REJECT
RULE-SET,https://raw.githubusercontent.com/carlleilzj/Talkatone_Rules/main/Surge/Talkatone.list,Talkatone
```

或安装模块：`Surge/Talkatone.sgmodule`

## Clash / Mihomo

```yaml
rule-providers:
  Talkatone:
    type: http
    behavior: classical
    url: "https://raw.githubusercontent.com/carlleilzj/Talkatone_Rules/main/Clash/Talkatone.yaml"
    path: ./ruleset/Talkatone.yaml
    interval: 86400
  Talkatone_Ads:
    type: http
    behavior: classical
    url: "https://raw.githubusercontent.com/carlleilzj/Talkatone_Rules/main/Clash/Talkatone_Ads.yaml"
    path: ./ruleset/Talkatone_Ads.yaml
    interval: 86400

proxy-groups:
  - name: Talkatone
    type: select
    proxies:
      - 美国节点
      - DIRECT

rules:
  - RULE-SET,Talkatone_Ads,REJECT
  - RULE-SET,Talkatone,Talkatone
```

## 规则包含什么

**走 Talkatone 策略（建议美国代理）**

- `talkatone.com` / `talkamobile.com`
- `tktn.at` / `tktn.be`（含 `cnt.tktn.be`）
- `vm.talkatone.com` 语音留言
- `tenor.com` 贴图
- 通话/短信相关 IP-CIDR（约 16 段）

**REJECT 广告**

- Google Ads / DoubleClick  
- Amazon / InMobi / Smaato / Criteo / PubMatic  
- MobileFuse / Fyber / Liftoff / Appier / Tappx 等  

**刻意不强制代理**

- `crashlytics` / `firebase` 等通用 SDK（避免误伤其他 App）  
- 系统推送 `apple.com`（保持直连）

## 使用建议

1. **注册失败**：换干净美国 IP；或临时把 `tktn.at` 改为 `direct` 试一次后再改回  
2. **能登不能打**：确认 IP-CIDR 规则生效，且节点支持 UDP（部分通话走 UDP）  
3. **有广告漏网**：在 QX 最近请求里找广告域名，提 issue 补规则  
4. **规则位置**：Talkatone 规则应在 `FINAL` 之前；不要被过于宽泛的 `GEOIP,CN,direct` 提前截走 IP（已用 no-resolve 的 IP-CIDR 更稳）


## 还是有广告？

请按顺序检查：

1. **同时启用**（缺一不可）  
   - `Talkatone_Ads.list`（分流拦截）  
   - `Talkatone_Ads_Rewrite.conf`（重写拦截素材/SDK）  
2. 广告规则在 **所有规则最上面**（先于 Talkatone 分流、先于代理规则）  
3. 远程资源点 **更新**，或删了重建  
4. 开启 **HTTPS 解密**，并信任证书（重写需要）  
5. **彻底杀掉** Talkatone 再开（清后台）  
6. 打开 QX「最近请求」，刷出广告时看域名：  
   - 若是 `smaato` / `inmobi` / `applovin` 等却没显示 reject → 规则没生效  
   - 若是 `talkatone.com` 自身接口返回的广告位 → 域名拦截去不掉（服务端下发），只能尽量拦第三方 SDK  

**去广告不需要节点**，策略必须是 `reject`。

重写配置：  
https://raw.githubusercontent.com/carlleilzj/Talkatone_Rules/main/QuantumultX/Talkatone_Ads_Rewrite.conf

## 参考来源

- [czy13724/Surge Talkatone.list](https://github.com/czy13724/Surge)  
- [eslco Talkatone_Opt](https://github.com/eslco/base)  
- 社区 Shadowrocket / Surge 公开模块  

## License

MIT

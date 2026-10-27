# 自定义代理与分流配置

本目录存放个人定制的订阅转换配置、分流规则，以及 Shadowrocket、Stash 的客户端配置。虽然位于 `Clash/` 下，但并非所有文件都能直接导入 Clash。

## 文件总览

| 文件 | 类型 | 用途 | 当前状态 |
| --- | --- | --- | --- |
| [ACL4SSR_Online_Full.ini](ACL4SSR_Online_Full.ini) | subconverter 外部配置 | 绑定规则集、生成策略组及分流规则 | 主配置 |
| [AiChat.list](AiChat.list) | 规则列表 | 补充 AI 服务域名 | 绑定美国节点组 |
| [Direct.list](Direct.list) | 规则列表 | 自定义直连域名 | 绑定全球直连组 |
| [HK.list](HK.list) | 规则列表 | Tailscale 地址段及指定自建服务 | 绑定香港节点组，部分域名被前置规则覆盖 |
| [JP.list](JP.list) | 规则列表 | 指定使用日本出口的域名 | 绑定日本节点组 |
| [LoadBalance.list](LoadBalance.list) | 规则列表 | 翻译服务等负载均衡候选规则 | 主配置中的引用和策略组均未启用 |
| [Oracle.list](Oracle.list) | 规则列表 | Oracle 甲骨文各产品线与历史品牌域名 | 绑定全球直连组，排在 ProxyGFWlist 之前 |
| [Proxy.list](Proxy.list) | 规则列表 | 指定使用代理的域名 | 绑定节点选择组 |
| [Shadowrocket_Personal.conf](Shadowrocket_Personal.conf) | Shadowrocket 配置 | 独立的基础设置、策略组和完整分流规则 | 不由主 `.ini` 引用 |
| [StashOverride.yml](StashOverride.yml) | Stash 覆写配置 | 将 DNS 服务器设置为系统 DNS | 独立使用 |

## 使用方式

### 订阅转换方案

将节点订阅与 `ACL4SSR_Online_Full.ini` 配合使用，由 subconverter 等兼容工具生成客户端配置：

```text
节点订阅 + ACL4SSR_Online_Full.ini
                 │
                 ├─ 自定义 .list 规则
                 ├─ 上游 ACL4SSR 规则
                 └─ 策略组定义
                 │
                 ▼
          转换后的客户端配置
```

当前主配置的远程地址为：

```text
https://raw.githubusercontent.com/chingwl/ACL4SSR/master/Clash/Custom/ACL4SSR_Online_Full.ini
```

在订阅转换工具的“外部配置”或相应配置项中指定该地址，并提供自己的节点订阅。主配置本身不包含节点服务器地址、密码等连接信息，也不是可直接导入 Clash 的 YAML 文件。

**主配置通过 GitHub 远程 URL 获取规则，而不是读取本地相邻文件。** 修改本地 `.list` 后，需要更新对应远程仓库，或修改主配置中的引用地址；转换结果才会使用修改后的内容。规则获取还取决于转换服务的网络和缓存状态。

### 独立客户端配置

- **Shadowrocket**：导入 `Shadowrocket_Personal.conf`，并配合已有节点或订阅使用；地区组按节点名称筛选。
- **Stash**：将 `StashOverride.yml` 作为覆写配置应用到现有配置。它不包含节点、策略组或分流规则，不能单独作为完整配置使用。

`Shadowrocket_Personal.conf`、`StashOverride.yml` 不会被 `ACL4SSR_Online_Full.ini` 自动合并。

## 规则语法

`.list` 文件仅定义匹配条件，不指定最终使用的策略。策略由主配置中的 `ruleset=策略组,规则地址` 绑定。

| 语法 | 含义 | 示例 |
| --- | --- | --- |
| `DOMAIN,域名` | 精确匹配完整主机名 | `DOMAIN,searxng.881889.xyz` |
| `DOMAIN-SUFFIX,域名` | 匹配域名本身及其子域名 | `DOMAIN-SUFFIX,infini.money` |
| `IP-CIDR,网段,no-resolve` | 匹配目标 IP 网段，不为该规则主动解析域名 | `IP-CIDR,100.88.88.0/24,no-resolve` |
| `# 注释` | `.list` 中的注释，不参与匹配 | `# tailscale` |

主 `.ini` 使用 `;` 注释。以 `;` 开头的规则、策略组和配置项均未启用。

规则按顺序匹配，先命中的规则优先。策略组名称只是标签，实际动作取决于组内当前选择。

## 各文件说明

### 1. ACL4SSR_Online_Full.ini

这是订阅转换主配置，包含规则绑定、策略组定义和规则生成开关。

#### 主要分流

| 规则来源或服务 | 策略组 |
| --- | --- |
| `Proxy.list` | 🚀 节点选择 |
| `Direct.list` | 🎯 全球直连 |
| `JP.list` | 🇯🇵 日本节点 |
| `HK.list` | 🇭🇰 香港节点 |
| `AiChat.list`、OpenAI、Claude、Gemini、Bing、GitHub | 🇺🇲 美国节点 |
| YouTube | 📹 油管视频 |
| Telegram | 📲 电报消息 |
| 局域网、国内域名、国内公司 IP、下载规则、`GEOIP,CN` | 🎯 全球直连 |
| 广告、应用广告 | 🛑 广告拦截 |
| OneDrive、微软服务 | Ⓜ️ 微软服务 |
| Google FCM、Google 中国相关服务 | 🎯 全球直连 |
| 苹果、网易音乐、Epic、Origin、Sony、Steam、Nintendo | 🎯 全球直连 |
| Netflix、巴哈姆特、国外媒体、GFW 列表 | 🚀 节点选择 |
| Bilibili、国内媒体 | 🎯 全球直连 |
| 未命中前面规则的流量（`FINAL`） | 🐟 漏网之鱼 |

自定义规则排在上述上游规则之前，因此可以优先处理指定域名。具体匹配范围由引用的规则集决定，不代表覆盖相应服务的全部流量。

#### 策略组

| 策略组 | 类型 | 可选项或筛选方式 |
| --- | --- | --- |
| 🚀 节点选择 | `select` | 美国、香港、日本、台湾、自建、机场、链式代理组 |
| Ⓜ️ 微软服务 | `select` | 节点选择、`DIRECT` |
| 📹 油管视频 | `select` | 名称匹配 `ZGO` 的节点、美国组、香港组、节点选择 |
| 📲 电报消息 | `select` | 美国组、香港组、节点选择 |
| 🎯 全球直连 | `select` | `DIRECT`、节点选择 |
| 🛑 广告拦截 | `select` | `REJECT`、`DIRECT`、节点选择 |
| 🐟 漏网之鱼 | `select` | 节点选择、`DIRECT` |
| 🇺🇲 美国节点 | `select` | 名称匹配“美”、美国城市名、`US`、`United States` 等 |
| 🇭🇰 香港节点 | `select` | 名称匹配“港”、`HK`、`hk`、`Hong Kong` 等 |
| 🇯🇵 日本节点 | `select` | 名称匹配“日本”、东京、大阪、`JP`、`Japan` 等 |
| 🇨🇳 台湾节点 | `select` | 名称匹配“台”、新北、彰化、`TW`、`Taiwan` 等 |
| ME 自建节点 | `select` | 名称包含“自建” |
| 🐱 机场节点 | `select` | 名称包含 `SakuraCat` |
| 🔗 链式代理 | `relay` | 依次使用机场节点组、自建节点组 |

- `select` 是手动选择组，不会自动测速并选择最快节点。
- 地区组按节点名称的正则表达式筛选，不验证实际出口位置；订阅节点名称需与筛选条件匹配。
- `[]组名` 引用其他策略组；`DIRECT` 表示直连，`REJECT` 表示拒绝连接。
- 链式代理的预期路径为“本机 → 机场节点 → 自建节点 → 目标网站”，实际可用性取决于客户端内核及节点协议支持。
- “全球直连”和“广告拦截”均可手动切换到其他选项，组名不代表固定动作。

当前启用了：

```ini
enable_rule_generator=true
overwrite_original_rules=true
```

即启用规则生成，并覆盖原配置中的规则。

韩国、新加坡等自动测速组、家宽组、负载均衡组，以及末尾的 DNS、TUN 模板配置均被注释，不会生效。

### 2. AiChat.list

补充以下 AI 服务域名及其子域名，交给“🇺🇲 美国节点”组：

- `openrouter.ai`
- `perplexity.ai`
- `coze.com`
- `cursor.com`
- `cursor.sh`
- `grok.com`
- `x.ai`

OpenAI、Claude、Gemini 等规则由主配置另外引用的上游规则集提供，不在此文件中。

### 3. Direct.list

将以下域名及其子域名交给“🎯 全球直连”组：

- `jingwl.cn`
- `quickconnect.to`
- `quickconnect.cn`
- `juchats.com`
- `qq.com`

Things 3 的 `culturedcode.com` 规则被注释，当前未启用。

**顺序注意：** 主配置先加载 `Direct.list`，后加载 `HK.list`。因此这里的 `DOMAIN-SUFFIX,jingwl.cn` 会覆盖 `HK.list` 中的四个 `jingwl.cn` 子域名，实际先进入“全球直连”组，而不是香港组。

若希望这些特定主机名优先走香港，需要将 `HK.list` 的绑定放到 `Direct.list` 之前，或缩小此处的域名后缀规则。当前文件尚未做此调整。

### 4. HK.list

将以下目标交给“🇭🇰 香港节点”组：

```text
IP-CIDR,100.88.88.0/24,no-resolve
DOMAIN,portainer.jingwl.cn
DOMAIN,bw.jingwl.cn
DOMAIN,freshrss.jingwl.cn
DOMAIN,open-webui.jingwl.cn
```

- IP 规则用于文件注释所指的 Tailscale 地址段，只匹配该 `/24` 网段，不是全部 Tailscale 地址。
- 域名规则精确匹配四个自建服务主机名；当前被前置的 `Direct.list` 覆盖，详见上一节。
- `no-resolve` 表示不为匹配这条 IP 规则主动解析域名，不代表整个请求过程不需要 DNS。
- 代理到该地址段的前提是所选代理服务器能够路由到目标地址；分流规则不会自动建立 Tailscale 网络连接。

### 5. JP.list

将以下目标交给“🇯🇵 日本节点”组：

```text
DOMAIN,searxng.881889.xyz
DOMAIN-SUFFIX,infini.money
```

第一条只匹配指定主机名，第二条匹配 `infini.money` 及其全部子域名。该文件是需要日本出口的目标规则，不是日本代理节点列表。

### 6. LoadBalance.list

预留以下负载均衡候选规则：

```text
DOMAIN-SUFFIX,deepl.com
DOMAIN,translate.google.com
DOMAIN,translate.googleapis.com
DOMAIN,get-client-ip.jingwl.cn
```

主配置中的对应 `ruleset` 和“🔮 负载均衡”组均被注释，因此该文件当前不参与主配置分流。仅修改此文件不会启用负载均衡。

### 7. Oracle.list

将 197 个 Oracle 甲骨文域名及其子域名交给“🎯 全球直连”组，按产品线分为四组：

| 分组 | 条数 | 说明 |
| --- | --- | --- |
| 核心品牌与官网 | 36 | `oracle.com`、`oraclecloud.com`、各国站点 `oracle.<cc>`、`oracle.cn`、`oracle.com.cn` 等 |
| 应用与数据库产品 | 86 | `netsuite.com`、`taleo.net`、`peoplesoft.com`、`siebel.com`、`micros.com`、`dyn.com`、`texturacorp.com` 等 Oracle 已收购产品域名 |
| 开发者与开源项目 | 22 | `java.com`、`mysql.com`、`sun.com`、`virtualbox.org`、`netbeans.org`、`lustre.org` 等 |
| 历史业务与客户门户 | 53 | 证书日志中仍解析、但已不发新证书的历史品牌与客户门户，如 `think.com`、`compendium.com`、`mvtresearch.com`、`toadirect.com` |

- 域名来源为 v2fly `domain-list-community`、MetaCubeX `meta-rules-dat`、blackmatrix7 `ios_rule_script` 等公开规则集，叠加 crt.sh 证书透明度日志枚举，并经 Oracle 权威 NS（`ns*.dns.oraclecloud.net`、`akam.net`）与 A 记录存活校验。
- 证书日志中 229 个根域里有 32 个未收录：已停售、转入第三方或不再解析，如 `cerner.com`、`netbeans.org`、`openoffice.org`、`lustre.org`、`oracle.com.cn`、`upshot.com`、`rsys*.net`。
- 营销与广告技术类域名（`eloqua.com`、`responsys.net`、`addthis.com`、`bluekai.com`、`moatads.com`、`maxymiser.*`、`elqsandbox.com` 等 32 条）**未纳入**：仓库 `BanAD.list` 已对其中部分域名下发 `REJECT`，纳入直连会使其降级为直连。
- **顺序注意：** 主配置中该规则集排在 `ProxyGFWlist.list` 之前，这一点是必需的：`ProxyGFWlist.list` 含 `DOMAIN-SUFFIX,uk`、`jp`、`hk`、`sg`、`eu`、`us`、`ca` 等国家级后缀规则，会抢先匹配 `netsuite.co.uk`、`netsuite.jp`、`netsuite.com.hk`、`netsuite.com.sg`、`netsuite.eu`、`netsuite.ca`、`oracle-is.jp`、`otn-shop.jp`、`livelook.us`、`microsdc.us`、`rfer.us`、`simphony.eu` 等 12 个域名。
- 同时 `ProxyGFWlist.list` 原有的 `# Oracle Cloud` 段（`cloud.oracle.com`、`oraclecloud.com`）已移除，两处调整缺一不可。
- Oracle 官方 IP 段（`public_ip_ranges.json`，1107 条 CIDR，含 `OCI`/`OSN`/`OBJECT_STORAGE` 标签）未纳入：OCI 计算与对象存储 IP 由多租户共享，按 IP 分流可能命中非 Oracle 流量。

### 8. Proxy.list

将以下域名及其子域名交给“🚀 节点选择”组：

```text
DOMAIN-SUFFIX,adspower.net
DOMAIN-SUFFIX,bitbrowser.net
```

主配置最先加载该规则集，使这些域名优先进入节点选择组，不被后续国内域名或国内 IP 规则判定为直连。

### 9. Shadowrocket_Personal.conf

独立的 Shadowrocket 配置，不是 Clash 配置，也不依赖本目录的 `.list` 文件。

| 配置段 | 内容 |
| --- | --- |
| `[General]` | 系统绕过、DNS、IPv6、TUN 排除路由和 UDP 行为等 |
| `[Proxy Group]` | `US`、`TW` 按节点名称筛选；`Bybit` 默认台湾组；`Google`、`TG`、`Talkatone` 默认美国组；六个组均为手动选择组 |
| `[Rule]` | 自定义地区分流、广告拦截、国内外服务、局域网及兜底规则 |
| `[Host]` | `localhost = 127.0.0.1` |
| `[URL Rewrite]` | 将匹配的 `g.cn`、`google.cn` 请求以 HTTP 302 重定向到 Google |

关键设置与分流：

- DNS 和备用 DNS 均使用 `system`；启用 IPv6，但不优先使用 IPv6。
- 配置了部分私有、保留地址的 TUN 排除路由；`100.64.0.0/10` 不在排除列表中，以便由规则处理 Tailscale 特例。策略不支持 UDP 转发时拒绝该流量。
- 开启直连域名解析失败时使用代理规则的回退行为。
- `infini.money` 使用 `US`；整个 `881889.xyz` 后缀使用 `US`。该后缀下的子域名并不同源：`n.881889.xyz`、`one-api.881889.xyz` 公网解析到 Cloudflare CDN（`104.21.63.9`、`172.67.142.12`），属公网服务；`www`、`searxng` 公网无记录。因此不能把该后缀当作自建 tailnet 服务统一改写。
- `100.88.88.0/24` 和五个自建服务域名使用 `US`（含 `yy-dm.jingwl.cn`）。网段特例先于后面的 `100.64.0.0/10,DIRECT`，其余 CGNAT 地址默认直连。`*.jingwl.cn` 为泛解析，顶域与未单独指定的子域名均指向公网 `39.105.83.184`；只有 `portainer`、`bw`、`yy-dm` 指向 `100.88.88.3`、`freshrss` 指向 `100.88.88.8`，即仅这四个主机名的目标地址在 tailnet 内（2026-10-27 公网 DNS 查询结果）。
- AI 规则统一绑定 `US`，覆盖 ChatGPT、Sora、Claude、Gemini、NotebookLM、Copilot、Cursor、OpenRouter、Perplexity、Coze、Grok 等已列出的服务及相关接口。国内的 `qwen.ai` 单独配置为 `DIRECT`。
- AI 规则先于 Microsoft、Google 和通用代理规则，避免 Gemini、Copilot 等先命中其他策略。ChatGPT 原有依赖域名及 IP 规则也改为 `US`；其中包含 `auth0.com`、`sentry.io`、`stripe.com` 等共享服务，其他应用访问这些匹配目标时也会使用美国组，并非仅影响 AI 请求。
- `live.com`、`microsoft.com` 使用 `PROXY`，其中先命中 AI 规则的 Copilot 子域名例外使用 `US`；`office365.com`、`outlook.com` 等仍沿用已有直连策略。
- Talkatone 业务域名、通话地址及 `tenor.com` 使用 `Talkatone` 组；列出的广告域名使用 `REJECT`。HTTP/3、QUIC 拦截规则被注释，未启用。
- 国内直连规则补充了阿里云、百度网盘、Bilibili 视频、抖音及字节资源、腾讯邮箱、微信、开发与知识网站等常用域名，以及京东、快手、爱奇艺、大众点评、腾讯云、七牛、国内公共资源和验证码服务的相关域名，主要依据仓库的 `ChinaDomain.list`，并非全量合并。爱奇艺域名修正为 `71.am`；搜狐相关的 `v-56.com` 保持原有正确配置。
- Oracle 甲骨文规则与 `Custom/Oracle.list` 内容一致，共 197 条，接入国内直连块末尾；原单条 `oracle.com` 直连规则已并入该段，`oraclecloud.com`（原由 `ProxyGFWlist.list` 提供代理）改为直连，不再受 `FINAL,PROXY` 影响。
- Bilibili 视频 CDN 精确域名 `upos-hz-mirrorakam.akamaized.net` 使用 `DIRECT`，来源为 `Ruleset/Bilibili.list`；该规则先于后置的 `DOMAIN-KEYWORD,aka,PROXY`，避免视频资源被关键词规则分流到代理。
- 运营商一键登录补充 `cmpassport.com`、`id6.me`、`mob.com` 后缀及 `enrichgw.10010.com`、`nishub1.10010.com` 精确域名直连，来源为 `Ruleset/ChinaOneKeyLogin.list`；不保证登录一定成功，实际仍取决于蜂窝网络和应用实现。
- `adspower.net`、`bitbrowser.net` 后缀使用 `PROXY`，同步自 `Custom/Proxy.list`，先于国内域名和 `GEOIP,CN` 直连规则，避免解析到国内 IP 时被判为直连。
- `quickconnect.to`、`juchats.com` 使用 `DIRECT`；`quickconnect.cn` 已由 `.cn` 后缀直连规则覆盖。群晖的 `synology.com`、`synology.me` 后缀使用 `DIRECT`，分别依据 `ChinaDomain.list`、`UnBan.list`；若使用海外 NAS，应将 `synology.me` 规则缩小为需要直连的具体主机名。
- GitHub 明确覆盖 `github.com`、`github.io`、`githubapp.com`、`githubassets.com`、`githubusercontent.com` 并使用 `PROXY`；Copilot 的前置精确规则例外使用 `US`。Docker 的 `docker.com`、`docker.io`、`dockerhub.com`、`compose-spec.io` 使用 `PROXY`。
- 非 AI 的 Google、YouTube 使用 `Google` 组，Telegram 使用 `TG` 组，两组当前均只列出 `US` 一项。`googleapis.cn`、`gstatic.cn` 仍优先命中 `.cn` 后缀直连规则，未改为代理。
- 苹果资源、本地域名、IPv4 局域网与保留地址、IPv6 的 `::1/128`、`fc00::/7`、`fe80::/10` 以及 `GEOIP,CN` 使用 `DIRECT`，IP 规则使用 `no-resolve`。其他已列出的社交平台等仍使用 `PROXY`。
- 最终规则为 `FINAL,PROXY`，未命中前面规则的流量使用代理。

**Tailscale 使用前提（实机确认的架构）：** 自建服务采用**节点回程**：`US` 组里选中的 `self建|US-BWH-中转HK` 本身在 tailnet 内，由它去连 `100.88.88.0/24`；本机不需要 Tailscale 连接（且它与 Shadowrocket 互为唯一活动 VPN，两者不要同时开启）。代理类型必须是 **TUN**：配置已用 `compatibility-mode = 3` 强制 TUN Only（优先级高于 **设置 → 代理类型**）。原因：`HTTP` 系统代理模式下，浏览器对 **IP 字面量**的连接不走系统代理（实测访问 `100.88.88.8:23386` 时代理日志无记录），导致 `IP-CIDR` 对裸 IP 访问完全无效；域形式则不受影响。访问方式归纳：`100.88.88.3:8443`（`bw`、`portainer`）、`100.88.88.8:23386`（open-webui，部署在 vm-debian-12）、`100.88.88.11:*` 等裸 IP 均命中 `IP-CIDR,100.88.88.0/24,US`；`https://open-webui.jingwl.cn/`的公网泛解析指向 `39.105.83.184`，该机器 443 上没有这个 vhost（实测 TLS 握手超时），因此已用 `[Host] open-webui.jingwl.cn = 100.88.88.8` 覆写解析，但 **端口无法由配置改写**，实际需访问 `http://open-webui.jingwl.cn:23386/`；若要求 `https://open-webui.jingwl.cn/`（443）可用，必须在服务端补反代或改 DNS 记录，不是本配置能解决的。家里 NAS 的 `192.168.31.22:5001` 新增了 `IP-CIDR,192.168.31.0/24,US,no-resolve`（位于 `192.168.0.0/16,DIRECT` 之前），**前提是 tailnet 内存在到 `192.168.31.0/24` 的子网路由**（家里 NAS/软路由已 advertise subnet）；若没有，改用 NAS 自己的 tailnet 地址访问。QuickConnect 中继地址（`*.direct.quickconnect.cn`）需 DSM 客户端建隧道，浏览器直连并非有效访问方式，建议弃用。待实机验证：TUN 模式下裸 IP 是否进入代理日志并命中 `US`、`192.168.31.0/24` 能否经节点回程、TUN 全接管后打印机/AirPlay 等本地服务是否受影响（可在 **设置 → 隧道** 调整）。

文件不包含具体节点定义，需要配合 Shadowrocket 中已有的节点或订阅。`US`、`TW` 分别按节点名称中的 `US`、`TW` 筛选，`US` 默认选择 `自建|US-DMIT-直连`；筛选条件不验证实际出口地区，AI 使用美国出口的前提是 `US` 组中选中了实际美国节点。此配置与 `.ini + .list` 方案存在差异，不应视为两份完全等价的分流配置。

### 10. StashOverride.yml

Stash 的系统 DNS 覆写配置：

```yaml
name: 添加系统 DNS 服务器
desc: 添加系统 DNS 服务器

dns:
  default-nameserver:
    - system
  nameserver:
    - system
```

- `default-nameserver`：通常用于解析 DNS 服务器自身的域名。
- `nameserver`：用于常规域名解析。
- 两者均指定 `system`，使用系统 DNS；最终与原配置的合并结果以 Stash 的覆写机制为准。

该文件不配置 DNS 请求的代理路径，不能单凭此覆写保证没有 DNS 泄漏。

## 维护注意事项

1. 新增域名时，选择对应 `.list` 文件，并检查是否被前置的域名后缀或其他规则覆盖。
2. 新增规则文件时，需在主 `.ini` 中添加 `ruleset` 绑定；仅创建文件不会使其生效。
3. 修改策略组筛选条件时，以实际订阅中的节点名称为依据。
4. 启用负载均衡时，需要同时启用规则引用及对应策略组，并确认客户端支持。
5. 修改远程规则后，重新执行订阅转换并检查生成配置；本地文件变更不会直接影响远程转换服务。
6. 上游规则集会独立更新，最终分流结果以转换时获取的规则和客户端实际配置为准。
7. Shadowrocket 配置独立维护，不会自动同步 `.list` 文件；修改时需同步本说明，并检查前置 AI 规则、Tailscale 网段特例及 TUN 排除项。

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
| 🚀 节点选择 | `select` | 美国、香港、日本、自建、机场、链式代理组 |
| Ⓜ️ 微软服务 | `select` | 节点选择、`DIRECT` |
| 📹 油管视频 | `select` | 名称匹配 `ZGO` 的节点、美国组、香港组、节点选择 |
| 📲 电报消息 | `select` | 美国组、香港组、节点选择 |
| 🎯 全球直连 | `select` | `DIRECT`、节点选择 |
| 🛑 广告拦截 | `select` | `REJECT`、`DIRECT`、节点选择 |
| 🐟 漏网之鱼 | `select` | 节点选择、`DIRECT` |
| 🇺🇲 美国节点 | `select` | 名称匹配“美”、美国城市名、`US`、`United States` 等 |
| 🇭🇰 香港节点 | `select` | 名称匹配“港”、`HK`、`hk`、`Hong Kong` 等 |
| 🇯🇵 日本节点 | `select` | 名称匹配“日本”、东京、大阪、`JP`、`Japan` 等 |
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

台湾、韩国、新加坡等自动测速组、家宽组、负载均衡组，以及末尾的 DNS、TUN 模板配置均被注释，不会生效。

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

### 7. Proxy.list

将以下域名及其子域名交给“🚀 节点选择”组：

```text
DOMAIN-SUFFIX,adspower.net
DOMAIN-SUFFIX,bitbrowser.net
```

主配置最先加载该规则集，使这些域名优先进入节点选择组，不被后续国内域名或国内 IP 规则判定为直连。

### 8. Shadowrocket_Personal.conf

独立的 Shadowrocket 配置，不是 Clash 配置，也不依赖本目录的 `.list` 文件。

| 配置段 | 内容 |
| --- | --- |
| `[General]` | 系统绕过、DNS、IPv6、TUN 排除路由和 UDP 行为等 |
| `[Proxy Group]` | `HK`、`US` 按节点名称筛选；`Google`、`TG` 可选香港或美国；`Talkatone` 只使用美国组；五个组均为手动选择组 |
| `[Rule]` | 自定义地区分流、广告拦截、国内外服务、局域网及兜底规则 |
| `[Host]` | `localhost = 127.0.0.1` |
| `[URL Rewrite]` | 将匹配的 `g.cn`、`google.cn` 请求以 HTTP 302 重定向到 Google |

关键设置与分流：

- DNS 和备用 DNS 均使用 `system`；启用 IPv6，但不优先使用 IPv6。
- 配置了部分私有、保留地址的 TUN 排除路由；`100.64.0.0/10` 不在排除列表中，以便由规则处理 Tailscale 特例。策略不支持 UDP 转发时拒绝该流量。
- 开启直连域名解析失败时使用代理规则的回退行为。
- `infini.money` 使用 `HK`；整个 `881889.xyz` 域名后缀使用 `US`，其中包括 `searxng.881889.xyz`。当前没有 `JP` 策略组。
- `100.88.88.0/24` 和指定自建服务使用 `HK`，其中额外包含 `yy-dm.jingwl.cn`。香港网段特例先于后面的 `100.64.0.0/10,DIRECT`，其余该网段地址默认直连。
- AI 规则统一绑定 `US`，覆盖 ChatGPT、Sora、Claude、Gemini、NotebookLM、Copilot、Cursor、OpenRouter、Perplexity、Coze、Grok 等已列出的服务及相关接口。国内的 `qwen.ai` 单独配置为 `DIRECT`。
- AI 规则先于 Microsoft、Google 和通用代理规则，避免 Gemini、Copilot 等先命中其他策略。ChatGPT 原有依赖域名及 IP 规则也改为 `US`；其中包含 `auth0.com`、`sentry.io`、`stripe.com` 等共享服务，其他应用访问这些匹配目标时也会使用美国组，并非仅影响 AI 请求。
- `live.com`、`microsoft.com` 使用 `PROXY`，其中先命中 AI 规则的 Copilot 子域名例外使用 `US`；`office365.com`、`outlook.com` 等仍沿用已有直连策略。
- Talkatone 业务域名、通话地址及 `tenor.com` 使用 `Talkatone` 组；列出的广告域名使用 `REJECT`。HTTP/3、QUIC 拦截规则被注释，未启用。
- 国内直连规则补充了阿里云、百度网盘、Bilibili 视频、抖音及字节资源、腾讯邮箱、微信、开发与知识网站等常用域名，主要依据仓库的 `ChinaDomain.list`，并非全量合并。爱奇艺域名修正为 `71.am`；搜狐相关的 `v-56.com` 保持原有正确配置。
- `quickconnect.to`、`juchats.com` 使用 `DIRECT`；`quickconnect.cn` 已由 `.cn` 后缀直连规则覆盖。
- GitHub 明确覆盖 `github.com`、`github.io`、`githubapp.com`、`githubassets.com`、`githubusercontent.com` 并使用 `PROXY`；Copilot 的前置精确规则例外使用 `US`。Docker 的 `docker.com`、`docker.io`、`dockerhub.com`、`compose-spec.io` 使用 `PROXY`。
- 非 AI 的 Google、YouTube 使用 `Google` 组，Telegram 使用 `TG` 组，两组均可手动选择 `HK` 或 `US`。`googleapis.cn`、`gstatic.cn` 仍优先命中 `.cn` 后缀直连规则，未改为代理。
- 苹果资源、本地域名、IPv4 局域网与保留地址、IPv6 的 `::1/128`、`fc00::/7`、`fe80::/10` 以及 `GEOIP,CN` 使用 `DIRECT`，IP 规则使用 `no-resolve`。其他已列出的社交平台等仍使用 `PROXY`。
- 最终规则为 `FINAL,PROXY`，未命中前面规则的流量使用代理。

**Tailscale 使用前提：** `HK` 中选中的代理服务器必须能够路由到 `100.88.88.0/24`；本机配置不会自动建立 Tailscale 网络连接。已移除该网段被整个 `100.64.0.0/10` TUN 排除项绕过的配置冲突，但系统路由、其他 VPN 和客户端实际接管行为仍需实机验证。导入后建议分别测试 `100.88.88.x` 使用 `HK`、该 `/10` 内其他地址使用 `DIRECT`，并检查自建服务、AI 服务和 IPv6 局域网的匹配日志。

文件不包含具体节点定义，需要配合 Shadowrocket 中已有的节点或订阅。`HK`、`US` 分别按节点名称中的 `HK`、`US` 筛选，默认选择名称分别为 `自建|HK-直连`、`自建|US-DMIT-直连`；筛选条件不验证实际出口地区，AI 使用美国出口的前提是 `US` 组中选中了实际美国节点。此配置与 `.ini + .list` 方案存在差异，不应视为两份完全等价的分流配置。

### 9. StashOverride.yml

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

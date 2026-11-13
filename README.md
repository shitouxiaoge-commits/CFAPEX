作者频道：https://www.youtube.com/@SHITOUGE
- TG 粉丝交流群：https://t.me/ROTOGE
---
# CFAPEX 1.4 更新说明

## 更新内容

- 修复 VLESS/XHTTP 的传输实现：由 `stream-one` 改为 `packet-up`，使用 `CfapexXhttpSession` Durable Object 保存跨请求会话。未绑定 `XHTTP_SESSION` 或设置 `XHTTP=0` 时不下发 XHTTP 节点，避免订阅导入不可用配置。
- 新增 Workers 自动部署脚本。通过 `tools/deploy-cfapex-mainline.mjs` 部署时，会创建或复用 XHTTP Durable Object 并绑定 `XHTTP_SESSION`；提供 Wrangler 模板与操作说明。
- 概览订阅区支持自定义下发节点数量，最多 1000 条；数量限制作用于最终订阅节点条目。
- 调整 Clash 规则订阅：`.cn` 域名及中国 IP 直连，其他流量按代理规则处理。
- 保留 VLESS/WS、Trojan/WS 与现有自定义优选 IP、Proxy IP、出站 IP 功能。

## 部署与兼容性

- Workers：使用 `CFAPEX-worker.1.4.js`。若需要 XHTTP，按项目中的自动部署说明运行脚本，或用 Wrangler 配置部署；单独在控制台粘贴 JS 不会自动创建 Durable Object 绑定。
- Pages：使用 `CFAPEX-Pages-1.4.zip`。仅上传 ZIP 无法创建 XHTTP Durable Object，XHTTP 需要额外部署并绑定对应 Worker；VLESS/WS 与 Trojan/WS 不依赖该绑定。
- XHTTP 节点需要支持 XHTTP 的 Xray 客户端核心。v2rayN 使用 Xray 核心；sing-box 核心不支持 XHTTP，可使用 VLESS/WS 或 Trojan/WS。
- 更新部署后，请在客户端重新更新订阅并删除旧的 `stream-one` 节点。线上是否可连接还取决于域名的 Cloudflare 安全设置及客户端网络，不能仅凭订阅下发判断。

## 交付文件

- Workers：`dist/CFAPEX-worker.1.4.js`
- Pages：`dist/CFAPEX-Pages-1.4.zip`
- 自动部署说明：`docs/CFAPEX-Workers-XHTTP-自动部署.md`

CFAPEX 1.3 更新日志

更新范围：自定义优选 IP 支持一次保存 1000 条、GitHub Raw 列表拉取超时修复之后的改动。 

## 功能与兼容性

- 新增 **Trojan/WS** 和 **VLESS/XHTTP（stream-one）**，与原有 VLESS/WS 共存；Trojan 和 XHTTP 只在 TLS 端口下发，均支持 TCP。Trojan 可单独设置密码，两个新协议均可通过环境变量关闭。
- 概览保留四个订阅入口：VLESS/WS YAML、Trojan/WS YAML、通用订阅、VLESS/XHTTP YAML。通用入口可切换 URI 与 sing-box JSON；显式 JSON 地址避免 sing-box 桌面客户端因请求头未被识别而收到 URI。XHTTP 查询转为 sing-box JSON 时提供兼容的 VLESS/WS 节点。
- 参考 CFNext V3.0.2 的地区选择，将内置节点池从 234 条更新为 555 条去重地址；用户保存的自定义优选列表仍优先。修正“CF 移动优选”备注被误识别为国家代码的问题。
- Clash YAML 修复 `MATCH` 规则的代理组名称；默认选择组包含自动测速，最多抽样 24 条测速，全部节点仍可手动选择。增加 UDP/443 拒绝规则，让视频流量的 QUIC 请求回退 TCP。

## 转发与资源控制

- 自定义 Proxy IP 仅接管 Cloudflare IPv4 目标的 TLS 流量；候选需完成 TLS 首响应验证，失败时尝试同地区下一条，最终恢复原有出站。非 Cloudflare 目标沿用原有出站。
- 错误 VLESS UUID 与 Trojan 密码立即拒绝；未认证 WebSocket 首包上限降至 64 KiB。默认限制目标端口；WS/XHTTP 单连接最长 15 分钟，上行最多 64 MiB、下行最多 128 MiB。
- 修复无关请求也读取 KV 的问题：无关路径和无效订阅地址在读取前返回；同一 Worker 实例 10 秒内复用并合并 KV 读取，保存或还原后立即失效。面板停止每 15 秒自动轮询，重新显示时刷新状态。
- Worker 字符串及面板 HTML 使用运行时解码表交付，Workers 与 Pages 保持相同的单文件入口。代码混淆不会改变网络行为，也不能消除 Cloudflare 的滥用通报。

## 验证与待确认事项

- 本地 KV 计数：40 个无关请求从 40 次读取降为 **0 次**；同一时窗内 30 次订阅请求共读取 **3 次**。协议、Proxy IP、真实出站及实际 workerd 回归通过；Workers、Pages 和 ZIP 内入口逐字节一致。
- 内置节点抽样 21 条进行 TLS/WebSocket 握手，12 条返回 101。该结果不能证明移动网络下全部节点可用，也不能保证 4K 视频速度。sing-box 桌面程序实际导入及 Cloudflare 线上 KV 告警变化仍需部署后验证。


# CFAPEX
项目是可部署在 Cloudflare 上的代理面板，支持 VLESS/WS、Trojan/WS 和 VLESS/XHTTP。用户可在面板中管理优选 IP、按地区设置 Proxy IP、自定义真实出站和节点 UUID，并获取 Clash YAML、通用 URI 或 sing-box JSON 订阅。项目提供 KV 设置存储、订阅访问控制及短时读取缓存。
# CFAPEX  详细使用教程

CFAPEX 可部署为 Cloudflare Worker 或 Pages Advanced Mode。它提供网页面板、节点订阅和 VLESS/WS、Trojan/WS、VLESS/XHTTP 节点。以下步骤针对当前面板版主线；轻量版文件 `src/worker.js` 不适用。

## 一、准备文件与变量

部署时二选一：

| 平台 | 使用文件 |
|---|---|
| Cloudflare Workers | `dist/CFAPEX-worker.1.3.js`，完整复制为一个模块 Worker |
| Cloudflare Pages Advanced Mode | `dist/CFAPEX-Pages-1.3.zip`，其中只有根目录的 `_worker.js`；也可上传 `dist/pages-panel` 目录 |

必须设置环境变量 **`A`**：变量名就是大写字母 `A`，值由你设定，作为面板访问密码。建议使用较长的随机值，不必填写 UUID。代码会从 `A` 派生默认节点 UUID；之后也可以在面板“设置”页单独修改节点 UUID。

建议先在 Cloudflare 控制台的 **KV / KV Namespace** 页面创建一个专用命名空间（例如 `CFAPEX_CONFIG`），再以绑定名 **`K`** 绑定到该 Worker 或 Pages 项目。面板设置会存入此 KV；如果不绑定 `K`，设置只能保存在边缘缓存，缓存被回收后可能恢复默认。`K` 是绑定名称，不能只把 `K` 填成普通文本环境变量。请为 CFAPEX 使用专用 KV，因为“还原默认”会清空整个绑定命名空间。

其他可选变量：

| 名称 | 作用 |
|---|---|
| `TROJAN_PASSWORD` | 为 Trojan 设置独立密码；不设置时使用节点 UUID |
| `TROJAN=0` | 关闭 Trojan 下发及入站 |
| `XHTTP=0` | 关闭 VLESS/XHTTP 下发及入站 |
| `OUTBOUND_PORTS` | 用逗号列出允许访问的目标端口，最多 32 个；设置后取代默认端口列表 |
| `PROXYIP` | 旧部署方式的全局 Proxy IP，未设置面板地区 Proxy IP 时仍可沿用 |
| `HEALTH_PROBE=1` | 启用订阅触发的后台健康探测；默认关闭 |
| `MASK_HTML` / `MASK_URL` | 设置非面板路径的伪装页面内容或地址 |

默认允许的目标端口为 `80,443,2052,2053,2082,2083,2086,2087,2095,2096,8080,8443,8880`。自定义出站代理服务器自身的端口不受此列表限制。

## 二、部署到 Cloudflare

### 方式 A：Workers

1. 在 Cloudflare 控制台创建 Worker，打开代码编辑器。
2. 用 `dist/CFAPEX-worker（版本号）js` 的**全部内容**替换编辑器内容并部署。不要把 Pages ZIP 粘贴到 Worker 编辑器。
3. 在该 Worker 的“变量和机密”中新增 `A`，值填你的面板密码。可选变量按上表添加。
4. 如需持久保存设置，在“绑定 / Bindings”中添加 KV Namespace 绑定，名称填 `K`，选择为本项目准备的命名空间。
5. 保存设置并使新部署生效，然后记下 Worker 的访问域名。

### 方式 B：Pages Advanced Mode

1. 创建 Pages 项目并选择直接上传，将 `dist/CFAPEX-Pages（版本号）.zip` 作为项目文件上传；ZIP 根目录只有 `_worker.js`。如果使用目录上传，则选 `dist/pages-panel`。
2. 在 Pages 项目设置中新增环境变量 `A`。需要 KV 时，在 Pages 的绑定设置中添加名称为 `K` 的 KV Namespace 绑定。
3. 确认变量和绑定应用到实际使用的环境；设置变更后按 Cloudflare 控制台提示重新部署，再记下 Pages 域名。

升级现有部署时直接替换代码或重新上传 1.3 ZIP。沿用原 `A` 和原 `K` 可保留面板设置；若更换 `A`，面板路径、订阅令牌及默认派生 UUID 都会变化，客户端必须重新导入订阅。旧版 1.2 文件保留作历史版本，升级时请选择文件名含 **1.3** 的交付文件。

## 三、登录面板

假设部署域名为 `https://example.workers.dev`，且设置了 `A=你的密码`，在浏览器地址栏访问：

```text
https://example.workers.dev/你的密码
```

密码区分大小写。密码包含空格、中文、`/`、`?`、`#` 或 `%` 时，路径部分需要 URL 编码；最省事的做法是使用仅含字母、数字及安全符号的长随机值。不要把带密码的面板地址分享给他人。旧变量名 `U` 不再作为面板入口。

导航顺序为：**概览 / 节点列表 / 自定义优选 IP / 自定义 Proxy IP / 自定义出站 IP / 设置 / 关于**。页面右下角的“保存并生效”是所有设置共用的保存按钮。修改设置后，保存成功还需在客户端更新订阅并重连；已经建立的连接不会立即切换配置。

## 四、获取并导入订阅

进入“概览”，复制面板提供的独立 `/s/<令牌>` 订阅地址。它与 `/<A的值>` 面板地址不同；不要把面板地址直接当作客户端订阅。概览显示四个入口：

| 入口 | 下发内容 | 常见用途 |
|---|---|---|
| VLESS/WS | Clash YAML | 支持 VLESS/WS 的 Clash/Mihomo 客户端 |
| Trojan/WS | Clash YAML | 支持 Trojan/WS 的 Clash/Mihomo 客户端 |
| 通用 | 可切换“通用 URI”或“sing-box JSON” | v2rayN 可尝试 URI；sing-box 桌面版请选择 JSON |
| VLESS/XHTTP | Clash YAML | 客户端核心支持 XHTTP 时使用 |

sing-box 远程配置请选择通用入口中的 **“sing-box JSON”**，再复制随之更新的链接。该链接显式带 `?format=singbox`，不依赖客户端请求头识别。sing-box JSON 主要提供 VLESS/WS；XHTTP 专用查询转换为 JSON 时提供兼容的 VLESS/WS 节点。不要把 Clash YAML 直接当作 sing-box 原生 JSON 导入。

v2rayN 可先试“通用 URI”；如果客户端无法按订阅导入逐行 URI，可点击“复制全部节点链接”，从剪贴板导入。Clash/Mihomo 使用对应 YAML 地址。VLESS/XHTTP 是否可用取决于客户端核心对该传输方式的支持。

导入后执行一次**更新订阅**，选择 CFAPEX 节点并重连。检查节点时，SNI 与 WS Host 应为你的部署域名，不能是优选 IP；WS 路径由订阅自动生成。不要手工把面板密码路径填到 WS Path。

## 五、设置自定义优选 IP

“自定义优选 IP”控制**客户端连接的节点入口地址**，会写进订阅节点的服务器地址。可手填 IP、域名或 HTTP(S) 文本列表地址。例如：

```text
1.1.1.1:443#US
1.1.1.2,8443#JP
https://bestcf.pages.dev/random-region/CA/50.txt
```

示例只说明格式，不保证示例地址在你的网络中可用。未写端口默认 443；也支持中文逗号、CSV 风格、IPv6 方括号及多个列表地址混排。保存时服务端会取回列表、规范化节点并按“地址 + 端口”去重。单次最多保存 **1000 条有效且不重复的节点**，输入及合并文本最多 **262144 字符**；重复或无效行不计入有效节点数。远程来源任一失败时旧列表保留。

保存自定义列表后，订阅只使用该列表；需要重新使用内置节点池时，在此页选择恢复内置列表。1.3 内置快照有 555 条候选，地址可用性会随地区、运营商和时间变化，请在自己的网络上测试。移动网络可用性不能由本机 TLS 握手结果保证。

## 六、区分 Proxy IP 与真实出站 IP

这两项设置解决的问题不同：

| 设置 | 控制位置 | 是否直接成为订阅节点服务器地址 |
|---|---|---|
| 自定义优选 IP | 客户端到 Cloudflare 的入口 | 是 |
| 自定义 Proxy IP | Worker 转发 Cloudflare IPv4 TLS 目标时尝试的中继 | 否；地区信息体现在节点路径 |
| 自定义出站 IP | Worker 到目标的真实 TCP 出口代理 | 否 |

### 自定义 Proxy IP

在“自定义 Proxy IP”页按地区填写，多个地址用 `/` 或换行分隔：

```text
109.168.1.1,443 #JP/109.168.2.2,8443 #US
```

这些地址仅是格式示例。单项可省略地区标签；省略后作为通用候选。最多 100 条、16000 字符。保存后要更新订阅并重连，使节点使用带地区的 WS 路径。Worker 只对解析为 Cloudflare IPv4 的 TLS 目标尝试这些候选；握手失败会尝试下一条并回到原有出站。**在 `ip.sb` 查到的出口 IP 不保证等于填写的 Proxy IP**，不能仅以两者不同判断该功能失效。

### 自定义出站 IP

“自定义出站 IP”控制 Worker 的真实 TCP 出口，支持：

```text
socks5://user:pass@host:1080
http://user:pass@host:8080
https://user:pass@host:443
ss://...
host:port
```

裸 `host:port` 按 SOCKS5 处理；未填端口时默认 1080。SS 仅支持 `aes-128-gcm`、`aes-256-gcm`、`chacha20-ietf-poly1305` 三种 AEAD 方法，不支持插件或 SS2022。代理地址及凭据只用于 Worker 出站设置，不写入订阅节点字段。

“出站方式”默认是 `auto`：出站代理连接或握手失败时可回退直连。要核对真实出口，在测试期间选择 **“仅走出口（only）”**；该模式失败即报错，不回退直连。保存、更新订阅并重连后，用已确认走该节点的浏览器访问 IP 检测网站。验证完毕可按需要调整模式。

## 七、设置 UUID 与还原默认

“设置”页允许填写标准 UUID，作为节点协议认证 UUID；它独立于面板密码 `A`。留空并保存会恢复由 `A` 派生的默认 UUID。更改 UUID 后必须更新订阅并断开旧连接再重连。Trojan 未设置独立 `TROJAN_PASSWORD` 时也会使用节点 UUID 作为密码。

右下角“还原默认”会**直接清空当前绑定 `K` 中的全部数据**，不限于 CFAPEX 已知的键；面板会分批删除并显示进度。使用前确认 `K` 是 CFAPEX 专用命名空间。未绑定 K 时，它清理当前主机的相关边缘缓存。环境变量（包括 `A`）不会被这个按钮删除。

## 八、常见问题

| 现象 | 检查方法 |
|---|---|
| 打不开面板 | 确认环境变量名为 `A`、部署已生效、访问的是 `https://域名/<A的值>`；特殊字符要 URL 编码。 |
| 订阅导入失败 | 从概览重新复制独立订阅地址；sing-box 选择 JSON，Clash/Mihomo 选 YAML；确认客户端已更新订阅。 |
| 保存后节点未变化 | 检查是否绑定了 `K`，保存是否显示成功；更新订阅、断开旧连接并重连。不同 Cloudflare 接入点的 KV 更新可能有传播延迟，单实例读取还会复用最多 10 秒。 |
| 自定义优选 IP 保存失败 | 检查列表 URL 能否访问、是否超时，以及 1000 条、262144 字符和单来源 256 KiB 限制。远程来源任一失败时旧列表不会被覆盖。 |
| 节点延迟为 -1 | 检查优选入口 IP、端口、SNI、WS Host、WS Path 与客户端日志；在同一网络测试部署域名入口。若只在移动网络失败，需要该网络下的实际日志定位。 |
| Proxy IP 已保存但查 IP 未变 | 它不是必然的最终出口；确认节点地区、订阅和重连，再检查是否发生候选失败回退。若要固定测试真实出口，使用“自定义出站 IP”的 only 模式。 |
| KV 读取额度仍高 | 确认已部署 1.3 文件；查看是否有旧订阅客户端持续刷新、面板仍开在多个设备或大量有效订阅请求。无关路径在当前版本不读取 KV，面板也不再每 15 秒轮询。 |

当前代码通过本地语法、协议、设置、KV 读量及 workerd 回归；公网线路、移动网络、sing-box 桌面客户端的实际连接和 Cloudflare 线上额度，仍应以部署后的客户端及后台数据为准。

## 一、轻量版简介

一个跑在 Cloudflare 边缘上的 Worker，主要做这几件事：

- 生成接入信息：访问 `/<UUID>` 就能拿到可以直接导入客户端的订阅（明文 / Clash 配置 / base64 三种格式）
- 跑 WebSocket 隧道：客户端握手升级之后按 VLESS 协议转发流量，支持早期数据
- 转发 DNS：客户端的 DNS 请求走 TCP 53 出去
- 优选地址：你可以自己填优选 IP，也可以挂在线列表，客户端连 IP、握手用你的域名
- 出站竞速：直连和中继一起试，能直连就尽量不绕路

---

## 二、快速开始

1. **建 Worker**：Cloudflare 面板 → Workers  → 新建一个 Worker，把 `src/worker.js` 全文粘进去（或者用 wrangler 部署）。
2. **填必填变量**：Settings → Variables 里加一个 `UUID`，值随便生成一个合法 UUID 就行。
3. **拿节点**：浏览器打开 `https://你的域名/<UUID>`。

`UUID` 是唯一的必填项。不填的话这个 Worker 对外就是个普通网站，不吐任何节点、不建隧道，所有请求都落到伪装页。根路径 `/` 也已经被锁了，别拿它去取节点，只会看到伪装页。

---

## 三、部署

### Workers（推荐）

- 面板：新建 Worker → 粘代码 → Deploy → 绑自定义域
- 脚本：`node tools/deploy-worker.mjs deploy <文件> <脚本名> UUID=<值>`
- Wrangler：`npx wrangler deploy`（仓库没带 wrangler.toml，得自己 init 一个）

自定义域在 Worker 的 Settings → Domains & Routes 里绑。绑了自有域之后 `*.workers.dev` 那个子域可以关掉，而且自定义域才是推荐的形态。

---

## 四、配置

### 部署变量

- `UUID`（必填）：接入令牌，决定了 `/<UUID>` 能不能访问。没配的话全站只剩伪装页。
- `PROXYIP`（选填）：自定义中继出口，填 `host` 或 `host:port`。全功能模式下生效，TLS 流量优先走它。
- `MASK_HTML`（选填）：伪装页的内联 HTML；填 `off` / `0` 就关掉伪装（回到 404）。
- `MASK_URL`（选填）：伪装页站点地址，按原路径把内容取回来；取失败就落回内置默认页。


---

## 五、路由和订阅格式

- `GET /<UUID>`：明文接入条目（默认格式）
- `GET /<UUID>?format=clash` 或 `?format=yaml`：Clash / Mihomo 配置
- `GET /<UUID>?format=base64` 或 `?format=v2ray`：base64 订阅
- `GET /<UUID>/clash`：也是 YAML 配置（路径写法）
- 客户端 UA 里带 `clash` / `mihomo` / `stash` / `verge` / `quantumult` / `surge` / `karing` 这些字样的，自动返回 YAML
- 带 `Upgrade: websocket` 且配了 `UUID` 的：进 WebSocket 隧道
- `GET /` 或其它路径：伪装页（根路径已锁）
- 没配 `UUID` 时：所有请求一律伪装页

订阅每行一个节点，长这样（端口决定走 TLS 还是明文）：

```
# 标准 TLS（绝大多数端口）
vless://<UUID>@<IP>:443?encryption=none&security=tls&sni=<你的域名>&fp=chrome&type=ws&host=<你的域名>&path=%2F%3Fed%3D2048#备注名

# 明文（端口是 80/8080 等，见 6.3）
vless://<UUID>@<IP>:80?encryption=none&security=none&type=ws&host=<你的域名>&path=%2F%3Fed%3D2048#备注名
```

自带的 Clash 配置里有两个策略组：一个手动选（节点选择），一个自动测速（url-test，每 5 分钟探一次），默认规则走手动组。

---

## 六、优选地址

这是整个项目最关键的一项配置，决定了下发给客户端哪些连接地址。

### 写法

- 直接写地址：`149.104.28.62:443#香港-01,149.104.28.62:80#香港-01明文`
- 在线列表 URL：`https://example.com/list.txt`（内容按行解析成 `host:port#备注`）

注意几点：

- 分隔可以用中英文逗号，也可以换行
- 每项写 `host:port#备注`，`#` 后面是备注名，会原样变成客户端里的节点名
- 多个在线列表 URL 之间一定要用逗号隔开。要是用 `#` 连两个 URL，会被当成网页锚点整个丢掉，表现就是「只取到第一个列表、还毫无报错」。代码现在对这种情况有容错，但别依赖它，老老实实写逗号
- 条数不限：填多少发多少，不截断
- 去重按 `host:port` 算，一样的只留一条。想加一条新节点而不是改个名，IP 或端口至少得有一个不一样
- 你自填的列表只要非空，就完全不会去拉区域池（不会混进没验证过的 IP）；只有列表为空时才用实时区域池兜底
- 当前内置列表
- 共 300 条 原生 TLS 实测最快节点，命名为 优选1 ~ 优选300（香港 / 日本 / 美国 / 新加坡 各约 56 条，台湾 22 条，韩国 55 条），全部本机实测可达
- 无固定品牌节点（v1.2.1 曾置顶「熊猫cloud三网机场」，v1.3.0 起移除，节点完全由实测优选池决定）
区域池兜底源是 bestcf.pages.dev 上按地区分的几个 txt。刷新/重建列表用 node build/fetch-fast-nodes.mjs 60 6 build/avail/fastlist300.txt（6 区均衡凑 300，注：build/ 为本地临时目录不进 git），校验用 node tools/verify-preferred.mjs。
提醒：区域池原始可用率只有六到八成，一定要先用 build-nodelist.mjs 预筛过再写进 PREFERRED_LIST，别直接把池子内容贴进来。
节点传输方式：明文还是 TLS（看端口）

### 节点传输方式：明文还是 TLS（看端口）

接入条目走明文还是 TLS，由优选地址的端口自动决定，不用额外配：

- 端口是 `80` `8080` `8880` `2052` `2082` `2086` `2095` → 明文 ws，不校验证书
- 其它端口（`443` `8443` `2053` `2083` `2087` `2096` 等）→ 标准 TLS 握手

前提：明文端口走的是 CF 的 HTTP 入口，得把域名在 **SSL/TLS → Edge Certificates** 里把 **Always Use HTTPS** 关掉，不然 CF 会把明文请求 301 跳走，而 ws 升级不跟跳转，就连不上。

例外：`*.workers.dev` 那个子域是平台强制 HTTPS 的，所以这类域名一律不生明文节点，全走 TLS。

同一个 IP 可以同时写两种端口，生成两条（明文 + TLS），客户端自己挑。

---

## 七、客户端接入

在客户端里填订阅地址就行，同一个地址会根据你的 UA 自动适配格式：

```
https://你的域名/<UUID>              # 浏览器/curl 看到明文条目，Clash 系 UA 看到 YAML
https://你的域名/<UUID>?format=clash  # 强制 YAML
https://你的域名/<UUID>?format=base64 # 强制 base64
```

各客户端建议：

- v2rayN / v2rayNG / NekoBox / Shadowrocket：直接填 `/<UUID>`（明文条目，兼容老客户端）
- Clash Verge / Mihomo / Stash / Surge / Karing：直接填 `/<UUID>` 就行（UA 自动识别返回 YAML），或者显式加 `?format=clash`
- 任何支持 base64 订阅的：`/<UUID>?format=base64`



---

## 八、伪装页

没匹配到任何路由的请求都会返回伪装页。代码里自带一份中性的默认页，开箱即用不用配；你也可以用部署变量换掉（运行期读，改配置不用重新部署）。优先级从高到低：

1. `MASK_HTML`：内联 HTML 原样返回；填 `off` / `0` 就关伪装（回到 404）
2. `MASK_URL`：把这个站点内容取回来当伪装页。填站点根就透传原路径，填具体页面就固定取那个地址；取失败落回内置默认页，不会暴露上游错误
3. 都没配 → 内置默认页（一个 Welcome 静态页，返回 200）

已经授权的令牌路径（`/<UUID>`、`/<UUID>/clash`）和 WebSocket 通道完全不受伪装配置影响，伪装页只兜底那些没匹配到路由的请求。

注意：`MASK_URL` 会让这个 Worker 变成那个站点的反向代理，只填你自己拥有或明确授权的站点。

---

## 九、出站和合规模式

出站逻辑大致是：合规模式开 → 纯直连，不经过任何第三方中继；合规模式关（当前默认）→ 先试自定义中继，再试 SOCKS5，然后直连和中继一起竞速，最后还有备用地址和中继兜底。

细节上直连有个优先窗口，能直连就尽量不绕路；碰到某些目标直连会被回环保护拦掉，这时候窗口就失效。非 TLS 的首包（比如 MTProto 或明文 HTTP）不走 SNI 型中继，否则会连上了但一个字节都收不到。

---

## 十、安全须知

1. `UUID` 就是唯一门禁。泄露等于服务被白嫖，别写进公开仓库、别截图、别发聊天记录。怀疑泄露就换个值，面板改一下就行，不用重新部署。
2. 根路径 `/` 已锁，没配 `UUID` 时全站只剩伪装页，对外看就是个普通网站。
3. `PROXYIP` 和 `MASK_*` 这些只在运行期读取，源码里不出现这些字符串明文。
4. 部署脚本名铁律（Error 1101）：每次线上验证必须用全新脚本名 + 全新子域；脚本名避开 `edgetunnel` `edtunnel` `bpb` `vless` `trojan` 这类黑名单词。已经污染过的名字（`edge-gateway`、`probe-hard`）永远别复用——一旦同名脚本报过 1101，之后同名部署任何内容都会 1101。改完记得重跑加固和校验（见第十二节）。
5. `src/worker.js.pre-harden` 是没门禁的旧基线，千万别部署——它硬编码了 UUID 还公开根路径。部署前确认源文件是 `src/worker.js` 或它重建出来的 `dist/*`。

---

## 十一、故障排查

| 现象 | 原因 | 处理 |
|---|---|---|
| 访问返回 **522** | 绑定的域没路由到 Worker | 去 Workers → Settings → Domains  看，确认是「自定义域」而不是只建了 Zone Route 
| 明文节点连不上 | 域名的 Always Use HTTPS 没关，明文被 301 了 | 去 SSL/TLS → Edge Certificates 关掉，或者改用 TLS 端口 |
| 订阅只下发了一部分节点 | 两个在线列表 URL 之间误用了 `#` 连接，锚点被丢掉 | 改用逗号分隔（代码现在会自动拆，但还是规范写） |
| 节点数比填的少 | `host:port` 去重了，同 IP 同端口只留一条 | 让重复项端口不同，或删掉重复项 |
| Worker 报 500 | 出站全失败 | 看响应体，检查目标可达性 / 优选 IP 是否已失效 |
| 订阅里中文节点名乱码 | 客户端把明文订阅当成 base64 解了 | 改用 `?format=base64`，或者客户端本身 UTF-8 支持不好，换明文订阅 |

---

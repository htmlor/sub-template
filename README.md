# 🚀 Subscription Templates / 订阅模板

本项目保存了 **Sing-box** 和 **Clash Meta (Mihomo)** 的通用配置模板，旨在提供一致且高效的代理体验。适用于订阅转换工具（如 Subconverter）作为远程配置（Remote Config）使用，或作为手动配置的基础。

## 📂 文件说明

| 文件名 | 适用核心 | 说明 |
| :--- | :--- | :--- |
| **[`singbox.json`](singbox.json)** | Sing-box | 包含 Tun/TProxy/Mixed 入站，使用 SRS 二进制规则集 |
| **[`clashmeta.yaml`](clashmeta.yaml)** | Clash Meta | 包含 Tun/TProxy/Mixed 入站，使用 Meta 格式 Geodata |

## ✨ 核心特性

- **统一策略组**: 两份配置采用了完全一致的分组逻辑，降低切换成本。
- **DNS 优化**: 均配置为 **Fake-IP** 模式，减少本地 DNS 解析耗时，防止 DNS 污染。
- **规则集**:
  - **Sing-box**: 引用 `testingcf.jsdelivr.net` 加速的 SagerNet `.srs` 规则集。
  - **Clash Meta**: 引用 `MetaCubeX` 维护的 `geoip.dat` 和 `geosite.dat`。
- **Tun 模式**: 均预设了 Tun 模式配置（Clash Meta 需配合启用），适合接管系统流量。

## 🧠 DNS 深度解析

本项目采用了 **DNS 分流** 与 **Fake-IP 混合模式**，在保证国外访问速度的同时，最大限度兼容国内网络环境。

### 1. 混合模式 (Fake-IP & Real-IP)

- **默认行为**: 启用 `enhanced-mode: fake-ip`。对于国外域名，直接返回 `198.18.x.x` 的虚假 IP，**立即响应**，并将域名解析请求封装在代理隧道中发送给远端节点。这不仅极大地加快了响应速度，还从根源上杜绝了本地 DNS 污染。
- **国内优化**: 通过 `fake-ip-filter` 过滤了 `geosite:cn` (国内域名)。
  - **效果**: 国内域名不返回 Fake-IP，而是进行真实的本地 DNS 解析。
  - **目的**: 确保国内流量能解析到最近的 CDN 节点 IP，避免因 Fake-IP 导致的国内服务访问异常或速度变慢。

### 2. DNS 分流策略 (Nameserver Policy)

配置了精细的 DNS 分流规则，确保“国人走国路，洋人走洋路”：

- **`nameserver-policy`**:
  - `geosite:cn` 强制使用国内 DNS (阿里 `223.5.5.5` / 腾讯 `119.29.29.29`)。
  - 保证国内域名解析结果纯净且通过 UDP/TCP 直连查询，速度最快。
- **`nameserver` & `fallback`**:
  - 国外域名优先尝试 `nameserver` (8.8.8.8 等)，同时并发查询 `fallback` (Google/Cloudflare DoH)。
  - 利用 Clash 的并发机制，取最快响应结果（通常是被污染的），但在 `fallback-filter` 介入前不会直接使用。

### 3. 防污染兜底 (Fallback Filter) - *仅 Clash Meta*

这是解决 DNS 污染的最后一道防线。当 `nameserver` 返回结果时，Clash 会检查：

- **`geoip: true`**: 如果返回的 IP 地址是 `CN` (中国)，则认为该结果有效（通常是国内 CDN），直接采用。
- **`ipcidr`**: 如果返回的 IP 是保留地址（如 `127.0.0.1`, `0.0.0.0` 等常见的污染结果），直接丢弃，强制等待 `fallback` (DoH) 的可信结果。
- **`domain`**: 对于 `google.com`, `twitter.com` 等必然被污染的域名，直接忽略普通 DNS 结果，强制使用加密 DNS (DoH) 解析。

### 4. 特殊兼容性 (Fake-IP Filter)

除了国内域名，以下服务也被强制排除在 Fake-IP 之外（返回真实 IP），以解决兼容性问题：

- **局域网/NTP**: `+.lan`, `+.local`, `ntp.*` 等，防止时间同步或内网发现失效。
- **STUN/P2P**: `stun.*`, `*.xboxlive.com` 等。游戏机和 P2P 软件通常需要真实 IP 来进行 NAT 穿透，使用 Fake-IP 会导致联机失败 (NAT 类型严格)。
- **微软/验证服务**: `msftconnecttest` 等连接性检查域名。

## 🛠️ 端口定义

| 服务类型 | 端口 | 说明 |
| :--- | :--- | :--- |
| **Mixed Port** | `7893` | 同时支持 HTTP 和 SOCKS5 |
| **Redir Port** | `7892` | 透明代理 (Redirect) |
| **TProxy Port** | `7895` | 透明代理 (TProxy) |
| **DNS Listen** | `7874` | 仅 Clash Meta |
| **Clash API** | `9090` | 外部控制端口 |

## 🧩 策略分组概览

主要策略组逻辑如下，覆盖了主流使用场景：

- **🚀 节点选择**: 手动切换的主入口，默认包含自动选择和手动节点。
- **⚡ 自动选择**: 基于 url-test (Google / Generate_204) 自动选择低延迟节点。
- **🤖 AI 服务**: 包含 OpenAI, Anthropic (Claude), Gemini 等 AI 服务的规则。
- **🎬 全球视频**: YouTube, Netflix, Disney+ 等流媒体服务。
- **📲 电报消息**: Telegram 专用分组，支持 IP 和 域名规则。
- **🐙 GitHub**: 单独的 GitHub 流量分组。
- **🛑 广告拦截**: 默认拦截常见的广告域名。
- **🐟 漏网之鱼**: Final 规则，未匹配到的流量默认走此代理组。

## 📝 使用建议

1. **订阅转换**: 将订阅转换工具的 `base` 或 `config` 参数指向本仓库文件的 Raw 链接。
2. **手动修改**: 下载文件后，请务必修改 `proxies` 部分，填入你自己的节点信息。

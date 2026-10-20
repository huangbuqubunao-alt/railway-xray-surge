# Railway Xray + Surge Proxy Service

基于 Railway 托管的轻量代理服务（Xray 核心 + Node.js 前门），专为 Surge（原生支持 VMess）及其他客户端（兼容 VLESS）设计。

## 架构说明
- **前门程序 (`server.js`)**: 监听公网端口，负责路由 WebSocket 流量并返回托管配置：
  - `/tk-rail` (WebSocket) -> 转发给本机 Xray 10000 端口（VMess，Surge 专用，稳定）
  - `/tk-vless` (WebSocket) -> 转发给本机 Xray 10001 端口（VLESS，其他客户端如 Shadowrocket / Stash 备用）
  - `/sub-<SUB_TOKEN>` -> 返回 Surge 托管配置文件（Managed Config）
  - `/vless` -> 返回单节点 `vless://` 链接
  - 其它所有未经鉴权的请求 -> 返回 `400 Bad Request`（伪装成普通失效网站）
- **流量出口**: Railway 机房 IP（不在本地或 AI 主机，不泄露本地网络）

## 环境变量要求（Railway Variables）
必须在 Railway 的环境变量中配置以下两项：
- `UUID`: Xray 客户端身份验证密钥（UUID 格式）
- `SUB_TOKEN`: 订阅路径防护口令（防止未授权人员探测订阅配置）

## 验证与验收
- **伪装验证**: 访问 `https://<你的Railway域名>/`，应返回纯文本 `400 Bad Request`
- **Surge 托管订阅**: 访问 `https://<你的Railway域名>/sub-<SUB_TOKEN>`
- **备用单节点链接**: 访问 `https://<你的Railway域名>/vless`

---
name: sbp-im-embed
description: 为任意 IM（飞书/钉钉/企业微信/Telegram/Slack）一键生成"能在 IM 内置浏览器里直接接管电脑会话"的懒加载网页，并给出该 IM 的机器人配置步骤。当用户说"把这个 agent 接进飞书/钉钉""做一个 IM 里能控制电脑会话的网页""生成 SBP 嵌入页""接入 sbp-sdk"时使用。
---

# sbp-im-embed

把一个 `sbp-server` 上的设备会话，变成 **一个可以塞进任何 IM 的网页**。
产物永远是三件套：`embed.html` + `im.config.json` + 一段可直接贴进机器人后台的说明。

## 何时用

- 用户有 `sbp-server`（或本仓库参考实现）在跑，想把会话接进某个 IM；
- 用户要"手机上看电脑上 claude code 在干嘛，并且能打字"；
- 用户想新增一个 IM 平台适配（已有飞书/钉钉/企微/Telegram 模板，新平台按同样套路加）。

## 输入（缺什么就用 AskUserQuestion 问，最多问 2 个）

| 必填 | 说明 |
|---|---|
| `im` | `feishu` / `dingtalk` / `wecom` / `telegram` / `web` |
| `server` | 例如 `https://sbp.example.com`（**必须 HTTPS**，IM webview 不收 http） |
| `token` | client_token，或说明"由机器人服务端换取"（推荐后者，token 不进 URL 明文） |
| `device_id` | `dev_xxx` |
| `sid` | 可选，默认落到会话列表 |

## 执行步骤

1. **确认能力位**：连一次 `GET {server}/api/health` 看设备是否在线；能力位决定页面上出现哪些按钮，**不要自己假设**。
2. **生成 `embed.html`**：只引一个脚本，其余交给 SDK。
   ```html
   <!doctype html>
   <html lang="zh-CN">
   <head>
     <meta charset="utf-8">
     <meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover,user-scalable=no">
     <meta name="referrer" content="no-referrer">
     <title>会话</title>
     <style>html,body,#app{margin:0;height:100%;background:#f6f7f9}</style>
   </head>
   <body>
     <div id="app"></div>
     <script src="__SERVER__/sbp-sdk.js"></script>
     <script>
       // token 优先级：机器人下发 -> URL fragment -> sessionStorage
       SBP.mount('#app', {
         server: '__SERVER__',
         token: new URLSearchParams(location.hash.slice(1)).get('t') || '',
         deviceId: '__DEVICE_ID__'
       });
     </script>
   </body>
   </html>
   ```
3. **生成 `im.config.json`**：把该 IM 的应用配置、入口、鉴权方式写清楚（见下表模板）。
4. **给出机器人侧代码片段**（三选一，按 im 字段）：

   **飞书**：卡片 `url` 打开网页；进页前用 `authen/v1/access_token` 换 `jsapi_ticket`，
   由服务端在 HTML 里注入 token，**不要**把 `app_secret` 放前端。
   ```json
   {"im":"feishu","entry":"bot_card_url","auth":"server_inject_token",
    "card":{"config":{"wide_screen_mode":true},
            "elements":[{"tag":"action","actions":[
              {"tag":"button","text":{"tag":"plain_text","content":"打开会话"},
               "url":"https://sbp.example.com/embed.html#t={{token}}","type":"primary"}]}]}}
   ```
   **钉钉**：工作台/消息卡片 `singleURL`；用 `dd.runtime.permission.requestAuthCode` 拿 authCode 换 token。
   **企业微信**：`snsapi_privateinfo` OAuth2；**localStorage 不可靠，token 必须走 URL fragment**。
   **Telegram**：Inline Keyboard `web_app` 按钮；服务端校验 `initData` 的 HMAC 后再下发 token。

5. **自检清单**（逐条打勾再交付）：
   - [ ] 首屏 HTML < 16KB，SDK < 32KB gzip
   - [ ] 页面里没有任何 `alert/confirm`
   - [ ] token 不出现在 `<script>` 明文里（除 fragment）
   - [ ] `#app` 高度用 `100dvh`，有 `env(safe-area-inset-bottom)` 兜底
   - [ ] 无 `input.choice` 能力时不渲染选项
   - [ ] 断网时显示"重连中"，不白屏

## 输出

- `embed.html`（可直接部署）
- `im.config.json`
- 一段 3 步以内的接入说明（贴给用户）

## 反模式

- ❌ 在网页里直连被控电脑（必须经 server（SBP 是 CS 模式，电脑只出站））
- ❌ 首屏就拉全量历史（必须 `session.history` 分页，20 条/页）
- ❌ 自己写 WebSocket 协议（用 `sbp-sdk`，协议升级由 SDK 兜住）
- ❌ 把 `device_secret` 放进网页

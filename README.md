# sbp-im-embed · SBP IM 内嵌技能

一个 WorkBuddy / Claude / Codex 类智能体的**技能（Skill）**：让智能体在 IM（飞书、
钉钉、Slack 等）对话里**懒加载一个 SBP 网页**，从而接管用户电脑上正在跑的 Agent
会话（claude-code / codex / dsh / hermes / openclaw / workbuddy / qwen-office）。

## 用法

把本仓库作为技能安装到你的智能体：

- WorkBuddy：`skills/` 目录或用户级 `~/.workbuddy/skills/` 下放本目录即可被识别。
- 其它支持 SKILL.md 的智能体：按各自的技能加载机制引入 `SKILL.md`。

技能触发后，智能体会：
1. 向 `sbp-server` 申请一个会话绑定 / 设备码；
2. 生成一条「在 IM 内打开 SBP 面板」的卡片（卡片里的网页来自 `sbp-server` 提供的
   `sbp-sdk.js`，即 `sbp-sdk` 仓库）；
3. 用户在 IM 里直接看屏、点选项、发指令。

## 依赖

- 协议规范：`sbp-spec`
- 公网中继：`sbp-server`（负责把 `sbp-sdk` 作为静态网页吐给卡片）
- 本机守护：`sbp-agent`

本仓库仅含 `SKILL.md`（技能定义），不含运行时代码。

# 🎬 Tutorial Generator

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![AI Skill](https://img.shields.io/badge/AI%20Skill-Compatible-green.svg)](#支持的工具)
[![PRs Welcome](https://img.shields.io/badge/PRs-Welcome-brightgreen.svg)](#贡献)

> **输入一个 URL，自动生成图文教程。** 支持 Markdown、HTML、PDF、视频（含字幕和 TTS 旁白）多种输出格式。

一个**工具无关**的 AI Skill，让任何支持浏览器自动化的 AI Agent 都能录制操作流程、截图、生成教程。

---

## ✨ 特性

- 🔧 **工具无关** — 使用抽象能力标识符，适配 Claude Code / Hermes / Gemini CLI / Codex 等
- 🔐 **登录处理** — 自动检测登录态，支持账密、验证码、OAuth 三种登录方式
- 📸 **丰富截图** — 每步操作前后各截图，弹窗/下拉/滚动额外截图
- 🎥 **多格式输出** — Markdown / HTML / PDF / 视频（含字幕 + TTS 旁白）
- 🛡️ **安全边界** — 删除操作只截图到确认弹窗，支付页面只截图不点击

## 🖼️ 输出示例

```
example.com/
├── example.com-tutorial.md       # Markdown 教程
├── example.com-tutorial.html     # HTML 教程（截图内嵌 base64）
├── example.com-tutorial.pdf      # PDF 教程
├── example.com-tutorial.mp4      # 视频教程（含字幕/旁白）
├── example.com-tutorial.srt      # 字幕文件
└── screenshots/
    ├── shot_00_home.png
    ├── shot_01_login.png
    ├── shot_02_step_before.png
    ├── shot_02_step_after.png
    └── ...
```

## 🚀 快速开始

### 安装

```bash
# 克隆到通用目录（推荐，所有工具可用）
git clone https://github.com/dashenbibi/tutorial-generator ~/.skills/tutorial-generator
```

<details>
<summary>📦 其他安装方式</summary>

**仅下载 skill 文件：**
```bash
mkdir -p ~/.skills/tutorial-generator
curl -o ~/.skills/tutorial-generator/SKILL.md \
  https://raw.githubusercontent.com/dashenbibi/tutorial-generator/main/SKILL.md
```

**Claude Code 专用路径（自动加载）：**
```bash
mkdir -p ~/.claude/skills/tutorial-generator
cp ~/.skills/tutorial-generator/SKILL.md ~/.claude/skills/tutorial-generator/SKILL.md
```
</details>

### 使用

向 AI Agent 发送：

```
生成 https://example.com 的使用教程
```

Agent 会自动执行 6 个阶段：

1. **Phase 0** — 询问目标读者、功能范围、登录信息、输出格式
2. **Phase 1** — 侦察网站结构，展示发现的功能模块
3. **Phase 2** — 检测登录状态，按需处理认证
4. **Phase 3** — 逐模块探索，每步操作前后截图
5. **Phase 4** — 汇总截图和步骤，生成教程
6. **Phase 5** — 输出文件，展示预览

### 输出格式

```bash
# 仅 Markdown（默认）
生成 https://example.com 的使用教程

# Markdown + HTML
生成 https://example.com 的教程，输出格式：markdown html

# 视频 + 字幕 + TTS 旁白
生成 https://example.com 的教程，输出格式：video+sub+audio

# 全套输出
生成 https://example.com 的教程，输出格式：markdown html video+sub+audio
```

## 🛠️ 支持的工具

| 工具 | 浏览器 | 登录态 | 视频录制 |
|------|--------|--------|----------|
| **Claude Code** | ✅ | 复用真实 Chrome | 需配合 screencapture |
| **Hermes** (NousResearch) | ✅ | CDP 附加 / 持久化 Session | ✅ 原生支持 |
| **Gemini CLI** | ✅ | 复用真实 Chrome | 需配合 screencapture |
| **OpenHands** | ✅ | ❌ 沙箱 | 需配合 recordmydesktop |
| **Codex** (OpenAI) | ✅ | ❌ 沙箱 | Computer Use |
| 任意支持 Playwright MCP 的工具 | ✅ | 取决于配置 | Playwright 内置 |

## 📋 依赖

| 功能 | 依赖 | 安装 |
|------|------|------|
| 视频合成 | ffmpeg | `brew install ffmpeg` |
| TTS 旁白（推荐） | edge-tts | `pip install edge-tts` |
| TTS 旁白（备选） | gtts | `pip install gtts` |
| PDF 输出 | pandoc | `brew install pandoc` |

> 无依赖时自动降级：视频 → 截图序列，TTS → 纯字幕，PDF → HTML。

## 🔧 能力映射

Skill 使用抽象能力标识符，适配任何工具：

| 能力 | 说明 |
|------|------|
| `NAVIGATE` | 打开/跳转 URL |
| `CAPTURE` | 截图并保存文件 |
| `READ_PAGE` | 读取页面结构 |
| `CLICK` | 点击元素 |
| `TYPE` | 向输入框填写文字 |
| `PRESS_KEY` | 键盘按键 |
| `RUN_JS` | 执行 JS 表达式 |
| `VISUAL_ANALYZE` | 截图 + AI 视觉分析 |
| `SCREEN_RECORD` | 录制屏幕 |

## 📖 版本历史

| 版本 | 主要变化 |
|------|---------|
| v2.0.0 | 全面重构：抽象能力标识符替代硬编码工具名 |
| v1.9.0 | CAPTURE 与 VISUAL_ANALYZE 解耦 |
| v1.8.0 | 视频选项支持 +sub / +audio 可选组合 |
| v1.7.0 | 录屏方案五档优先级检测 |
| v1.6.0 | 新增视频输出格式 |
| v1.5.0 | Phase 1 硬停止重构 |
| v1.4.0 | 支持 Markdown / HTML / PDF |
| v1.3.0 | 操作类型分类表 |
| v1.2.0 | 强制截图规则 |
| v1.1.0 | 登录处理按浏览器模式分流 |
| v1.0.0 | 初始版本 |

## 🤝 贡献

欢迎提 Issue 或 PR：

- 补充新工具的能力映射示例
- 改进登录处理逻辑
- 新增输出格式支持
- 修复在特定工具上的执行问题

## 📄 License

MIT

---

<p align="center">
  <i>如果这个项目对你有帮助，欢迎 ⭐ Star 支持！</i>
</p>

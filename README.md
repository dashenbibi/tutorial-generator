# tutorial-generator

一个工具无关的 AI Skill，输入网站 URL，自动探索页面、截图、记录操作步骤，生成图文教程。支持 Markdown、HTML、PDF、视频（含字幕和 TTS 旁白）多种输出格式。

## 特性

- **工具无关** — 使用抽象能力标识符，适配任何支持浏览器自动化的 AI Agent
- **登录处理** — 自动检测登录态，支持账密、验证码、OAuth 三种登录方式
- **丰富截图** — 每步操作前后各截图，弹窗/下拉/滚动额外截图，保底每模块 3 张
- **多格式输出** — Markdown / HTML（截图内嵌 base64）/ PDF / 视频
- **视频支持** — 录屏 + 可选 SRT 字幕烧录 + 可选 TTS 旁白合成
- **安全边界** — 删除操作只截图到确认弹窗，支付页面只截图不点击

## 支持的工具

| 工具 | 浏览器能力 | 登录态 | 视频录制 |
|------|-----------|--------|---------|
| Claude Code (Claude in Chrome) | ✅ | 复用真实 Chrome | 需配合 screencapture |
| Hermes (NousResearch) | ✅ | CDP 附加 / 持久化 Session | ✅ 原生支持 |
| Gemini CLI | ✅ | 复用真实 Chrome | 需配合 screencapture |
| OpenHands | ✅ | ❌ 沙箱 | 需配合 recordmydesktop |
| Codex (OpenAI) | ✅ App 内 | ❌ 沙箱 | Computer Use |
| 任意支持 Playwright MCP 的工具 | ✅ | 取决于配置 | Playwright 内置 |

## 安装

**方式一：克隆到通用目录（推荐，所有工具可用）**

```bash
git clone https://github.com/YOUR_USERNAME/tutorial-generator ~/.skills/tutorial-generator
```

**方式二：仅下载 skill 文件**

```bash
mkdir -p ~/.skills/tutorial-generator
curl -o ~/.skills/tutorial-generator/SKILL.md \
  https://raw.githubusercontent.com/YOUR_USERNAME/tutorial-generator/main/SKILL.md
```

**Claude Code 专用路径（自动加载）：**

```bash
mkdir -p ~/.claude/skills/tutorial-generator
cp ~/.skills/tutorial-generator/SKILL.md ~/.claude/skills/tutorial-generator/SKILL.md
```

**其他工具（Hermes / Gemini CLI / Codex 等）：**

在 system prompt 或会话开始时引用：

```
请先读取 ~/.skills/tutorial-generator/SKILL.md，然后按其中的流程执行。
```

## 使用方法

向 AI Agent 发送以下格式的请求：

```
生成 https://example.com 的使用教程
```

Agent 会按以下流程执行：

1. **Phase 0** — 询问目标读者、要记录的功能、登录信息、输出格式
2. **Phase 1** — 侦察网站结构，展示发现的功能模块，**等你选择范围**
3. **Phase 2** — 检测登录状态，按需处理认证
4. **Phase 3** — 逐模块探索，每步操作前后截图
5. **Phase 4** — 汇总截图和步骤，生成教程
6. **Phase 5** — 输出文件，展示预览

### 输出格式示例

```
# 仅 Markdown（默认）
生成 https://example.com 的使用教程

# Markdown + HTML
生成 https://example.com 的教程，输出格式：markdown html

# 视频 + 字幕 + TTS 旁白
生成 https://example.com 的教程，输出格式：video+sub+audio

# 全套输出
生成 https://example.com 的教程，输出格式：markdown html video+sub+audio
```

### 视频格式依赖

| 功能 | 依赖 | 安装 |
|------|------|------|
| 视频合成 | ffmpeg | `brew install ffmpeg` |
| TTS 旁白（推荐） | edge-tts | `pip install edge-tts` |
| TTS 旁白（备选） | gtts | `pip install gtts` |
| PDF 输出 | pandoc | `brew install pandoc` |

无依赖时自动降级：视频降为截图序列，TTS 降为纯字幕，PDF 降为 HTML。

## 输出文件结构

```
{网站域名}/
├── {网站域名}-tutorial.md       # Markdown 教程
├── {网站域名}-tutorial.html     # HTML 教程（截图内嵌）
├── {网站域名}-tutorial.pdf      # PDF 教程
├── {网站域名}-tutorial.mp4      # 视频教程（含字幕/旁白）
├── {网站域名}-tutorial.srt      # 字幕文件
└── screenshots/
    ├── shot_00_home.png
    ├── shot_01_login_overview.png
    ├── shot_02_step1_before.png
    ├── shot_02_step1_after.png
    └── ...
```

## 能力映射

Skill 使用抽象能力标识符，执行前需确认你的工具对应关系：

| 能力 | 说明 |
|------|------|
| `NAVIGATE` | 打开/跳转 URL |
| `CAPTURE` | 截图并保存文件 |
| `READ_PAGE` | 读取页面结构（compact/full） |
| `CLICK` | 点击元素 |
| `TYPE` | 向输入框填写文字 |
| `PRESS_KEY` | 键盘按键（可选） |
| `RUN_JS` | 执行 JS 表达式（可选） |
| `VISUAL_ANALYZE` | 截图 + AI 视觉分析（可选增强） |
| `SCREEN_RECORD` | 录制屏幕（video 格式需要） |

> 如果你的工具将截图和视觉分析合并为一个工具（如 Hermes 的 `browser_vision`），
> 直接将 CAPTURE 和 VISUAL_ANALYZE 都映射到它即可。

## 版本历史

| 版本 | 主要变化 |
|------|---------|
| v2.0.0 | 全面重构：用抽象能力标识符替代硬编码工具名，新增能力映射自检表 |
| v1.9.0 | CAPTURE 与 VISUAL_ANALYZE 解耦，修复 vision 失败导致截图丢失的问题 |
| v1.8.0 | 视频选项支持 +sub / +audio 可选组合，TTS 五档降级方案 |
| v1.7.0 | 录屏方案五档优先级检测（原生录制 / Playwright / screencapture / recordmydesktop / 截图序列） |
| v1.6.0 | 新增视频输出格式，支持 ffmpeg 合成 MP4 |
| v1.5.0 | Phase 1 硬停止重构，browser_vision 失败处理 |
| v1.4.0 | 支持 Markdown / HTML / PDF 三种输出格式 |
| v1.3.0 | 新增操作类型分类表，编辑/删除操作专项处理 |
| v1.2.0 | 强制截图规则，每步操作前后各一张，最低保底数量 |
| v1.1.0 | 登录处理按浏览器模式分流，沙箱/真实浏览器差异化处理 |
| v1.0.0 | 初始版本 |

## 贡献

欢迎提 Issue 或 PR：
- 补充新工具的能力映射示例
- 改进登录处理逻辑
- 新增输出格式支持
- 修复在特定工具上的执行问题

## License

MIT

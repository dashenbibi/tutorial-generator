**中文** | [English](./README.md)

# tutorial-generator

> 当前工作流以 [SKILL.md](./SKILL.md) 为准：用户在浏览器自行登录，无需在聊天中提供密码或验证码；支持的格式、语言和权限取决于实际 Agent 与系统环境。

一个用于生成网站图文教程的 AI Skill。告诉 Agent 网址和使用目标，它可以利用当前环境提供的工具探索可访问的页面、截图，并记录经过验证的操作步骤。

## 特性

- **适配多种 Agent** — 优先使用当前 Agent 可用的浏览器和文档工具；具体能力取决于环境
- **登录处理** — 用户在浏览器自行完成登录；Agent 验证是否能访问目标内容
- **截图核验** — 按步骤需要记录页面和操作结果，检查空白图及错误页面
- **多格式输出** — Markdown / HTML / PDF / 视频；实际能力取决于当前环境
- **视频支持** — 根据可用工具使用真实录屏或截图幻灯片；旁白与字幕按需生成和校验
- **谨慎操作** — 保护敏感信息，可能产生实际影响的操作需要相应授权

## 真实产物预览

查看仓库自带的端到端生成示例：[examples/xroute.ai](./examples/xroute.ai)
- **目标网站**：[XRoute.ai](https://xroute.ai)（AI 模型聚合路由平台）
- **涵盖流程**：账号注册、获取 API Key、额度充值、浏览模型、在线对话
- **生成产物**：[完整 Markdown 图文教程](./examples/xroute.ai/XRoute-tutorial.md)（含 20 张高清步骤截图）、[MP4 视频教程](./examples/xroute.ai/xroute-tutorial.mp4)、SRT 字幕与 TTS 语音旁白音频

## 适用场景与支持的网站类型

本技能**不仅限于教程或文档网站**，它的核心价值在于将**任何交互式网站或 Web 产品**自动转化为易懂的新手操作指南：

| 适用场景 | 典型网站示例 | 说明与目标 |
|---------|-------------|-----------|
| **SaaS 与 Web 应用**（最核心） | GitHub, Notion, Linear, Stripe, Canva | 自动化生成产品功能使用指南、配置流程（如“如何创建项目并邀请成员”） |
| **企业内部系统与管理后台** | CRM, ERP, 数据大屏, 运维平台 | 用户安全登录后，由 Agent 探索各模块功能，生成员工内部培训 SOP |
| **业务与电商平台** | Shopify, 电商商城, 在线工具 | 记录注册流程、商品搜索、选购下单、个人中心设置等全流程步骤 |
| **文档与知识库** | 官方开发者文档, 开源技术站点 | 自动梳理复杂文档结构，提取核心路径，生成精简的“快速入门指引” |

## 支持的工具

可与支持 skill 或能读取 `SKILL.md` 的 Agent 配合使用。浏览器、登录态、录屏、语音和文档导出能力取决于具体 Agent、工具和系统配置；运行前会按请求检测，缺少权限或能力时说明并提供可用的替代方式。尚未对所有 Agent 和系统完成端到端验收。

## 安装

### 方式一：一键安装（推荐）

通过 [skills.sh](https://skills.sh) 官方 CLI，一条命令即可自动识别并安装到你的 Agent（Claude Code、Cursor、Antigravity 等）：

```bash
# 全局安装（推荐，所有项目均可使用）
npx skills add dashenbibi/tutorial-generator -g

# 或仅安装到当前项目
npx skills add dashenbibi/tutorial-generator
```

---

### 方式二：手动安装（无需 Node.js）

**Claude Code：**

单行命令直接下载到技能目录（自动创建目录）：

```bash
curl -fsSL https://raw.githubusercontent.com/dashenbibi/tutorial-generator/main/SKILL.md -o ~/.claude/skills/tutorial-generator/SKILL.md --create-dirs
```

或直接克隆仓库：

```bash
git clone https://github.com/dashenbibi/tutorial-generator ~/.claude/skills/tutorial-generator
```

**通用目录 / 其他 Agent（Antigravity、Cursor、Codex、Hermes、Gemini CLI 等）：**

克隆到通用技能目录：

```bash
git clone https://github.com/dashenbibi/tutorial-generator ~/.skills/tutorial-generator
```

并在提示词或会话开始时引用：

```
请先读取 ~/.skills/tutorial-generator/SKILL.md，然后按其中的流程执行。
```

## 使用方法

向 AI Agent 发送自然语言请求即可开始。建议提供目标**网址**以及想要重点记录的**功能目标**：

```
# 示例 1：基础全站探索（自动梳理核心功能）
生成 https://example.com 的使用教程

# 示例 2：指定具体的业务目标（推荐，更聚焦）
生成 https://xroute.ai 的使用教程：包含注册账号、获取 API Key 和在线体验模型对话

# 示例 3：指定多格式产物与语言
生成 https://github.com 的教程：如何创建仓库并提交 Pull Request，格式：markdown html，语言：中文

# 示例 4：生成带字幕与语音旁白的视频教程
生成 https://example.com 的上手教程，格式：video+sub+audio
```

Agent 会根据请求检查浏览器、登录、截图和输出工具，探索指定功能并核实操作结果，再生成请求的教程格式。只有范围不明确或确需用户登录、授权时才需要进一步询问。

### 教程语言

可以在请求中指定教程语言；语音和 PDF 的实际效果取决于可用的语言、字体和语音工具：

```
生成 https://example.com 的教程，语言：中文
生成 https://example.com 的教程，语言：日本語
生成 https://example.com 的教程，语言：English
```

可以请求的语言示例：

| 语言 | 写法 |
|------|------|
| English | `English` |
| 简体中文 | `中文` |
| 日本語 | `日本語` |
| 한국어 | `한국어` |
| Español | `Español` |
| Français | `Français` |
| Deutsch | `Deutsch` |
| Português | `Português` |
| العربية | `العربية` |

默认使用用户的语言；网站界面标签可以保留原文并附翻译。字体或语音不支持时应说明。

### 输出格式示例

```
# 仅 Markdown（默认）
生成 https://example.com 的使用教程

# Markdown + HTML
生成 https://example.com 的教程，格式：markdown html

# 视频 + 字幕 + TTS 旁白
生成 https://example.com 的教程，格式：video+sub+audio

# 多种格式
生成 https://example.com 的教程，格式：markdown html video+sub+audio
```

### 视频和 PDF 的环境要求

优先使用 Agent 已有的录屏、文档导出和语音工具。需要额外工具或系统权限时，Agent 会说明具体缺口；不能录屏时可提供明确标注的截图幻灯片。PDF 和视频输出并非在所有环境中都可用。

## 输出文件结构

以下是示例结构；实际文件取决于请求的格式和可用工具：

```
{域名}/
├── {域名}-tutorial.md
├── {域名}-tutorial.html
├── {域名}-tutorial.pdf
├── {域名}-tutorial.mp4      （含字幕/旁白，如有选择）
├── {域名}-tutorial.srt
└── screenshots/
    ├── shot_00_home.png
    ├── shot_01_module_overview.png
    ├── shot_02_step1_before.png
    ├── shot_02_step1_after.png
    └── ...
```

## 能力映射

Agent 会根据任务检查所需能力并对应到当前可用的工具。下面是能力示例，并非强制的工具名称：

| 标识符 | 说明 |
|--------|------|
| `NAVIGATE` | 打开/跳转 URL |
| `CAPTURE` | 截图并保存文件 |
| `READ_PAGE` | 读取页面结构（compact / full） |
| `CLICK` | 点击元素 |
| `TYPE` | 向输入框填写文字 |
| `PRESS_KEY` | 键盘按键（可选） |
| `RUN_JS` | 执行 JavaScript（可选） |
| `VISUAL_ANALYZE` | 截图 + AI 视觉分析（可选增强） |
| `SCREEN_RECORD` | 开始/停止录屏（视频格式需要） |

> 如果你的工具将截图和视觉分析合并为一个（如 Hermes 的 `browser_vision`），
> 将 `CAPTURE` 和 `VISUAL_ANALYZE` 都映射到它即可。

## 常见问题 (FAQ)

**Q：它只能给已有的教程网站或文档网站做总结吗？**  
**A**：不是！它是为**任意网站或 Web 产品**制作操作教程的。无论是 SaaS 工具、企业内部后台、电商网站还是文档库，只要能在浏览器中打开，Agent 都能帮你一步步探索、截图并记录使用指南。

**Q：需要登录的网站怎么处理？我的账号密码安全吗？**  
**A**：绝对安全。本工作流遵循安全优先原则，**严禁在 AI 聊天中索取或填写明文密码与验证码**。当遇到登录墙时，Agent 会提示你在真实浏览器中自行登录，登录完成后 Agent 仅核验当前可见的页面状态并继续探索。

**Q：如果当前 Agent 环境没有录屏或 TTS 工具，还能用吗？**  
**A**：完全可以。默认输出为最轻量、通用的 Markdown 图文教程。如果请求了视频但缺少录屏工具，Agent 会自动降级为“截图幻灯片 + 步骤切换”，缺少 TTS 则仅生成字幕，优雅降级，不会报错中断。

**Q：它是一个命令行脚本（CLI）还是 AI Agent 技能？**  
**A**：它是一个标准的 AI Agent Skill（遵循 `SKILL.md` 规范）。安装后，由支持该规范的 AI Agent（如 Claude Code、Cursor、Antigravity、Codex 等）读取并在会话中自主驱动浏览器工具执行。

## 版本历史

| 版本 | 主要变化 |
|------|---------|
| v4.0.0 | 自包含、适配不同 Agent 的工作流；截图核验、登录交接、多格式与能力检查 |
| v3.2.0 | SKILL.md 改回纯英文；README 拆分为中英双语独立文件 |
| v3.1.0 | SKILL.md 内联双语（英文主体 + 中文注释） |
| v3.0.0 | 全面英文重写；支持多语言教程输出 |
| v2.0.0 | 用抽象能力标识符替代硬编码工具名 |
| v1.9.0 | CAPTURE 与 VISUAL_ANALYZE 解耦，修复截图丢失问题 |
| v1.8.0 | 视频附加选项 +sub / +audio 可自由组合；TTS 五档降级 |
| v1.7.0 | 录屏方案五档优先级检测 |
| v1.6.0 | 新增视频输出格式，ffmpeg 合成 MP4 |
| v1.5.0 | Phase 1 硬停止重构；browser_vision 失败处理 |
| v1.4.0 | 支持 Markdown / HTML / PDF 三种输出格式 |
| v1.3.0 | 操作类型分类；编辑/删除专项处理 |
| v1.2.0 | 强制截图规则；最低截图数量保底 |
| v1.1.0 | 登录处理按浏览器模式分流 |
| v1.0.0 | 初始版本 |

## 贡献

欢迎提出建议、报告问题或提交 PR！如果教程不完整、截图空白、登录或视频生成失败，或者你有希望改进的功能，请[提交 Issue](https://github.com/dashenbibi/tutorial-generator/issues/new)。

报告问题时，欢迎附上使用的 Agent、操作系统、输出格式、预期结果、实际情况和复现步骤。网址和截图请在方便分享时提供，并先移除密码、令牌等隐私信息。功能建议和成功使用的案例也非常欢迎。

## License

MIT

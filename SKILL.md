---
name: tutorial-generator
version: 3.1.0
description: "Automatically generate illustrated tutorials for any website. Given a URL, explore pages, take screenshots, record steps, and produce tutorials in Markdown, HTML, PDF, or Video formats. · 自动生成网站图文教程。输入 URL，探索页面、截图、记录步骤，输出 Markdown、HTML、PDF 或视频教程。"
metadata:
  requires:
    capabilities:
      - NAVIGATE       # Navigate to a URL · 导航到 URL
      - CAPTURE        # Take a screenshot and save to file · 截图并保存文件
      - READ_PAGE      # Read page structure/content · 读取页面结构/内容
      - CLICK          # Click a page element · 点击页面元素
      - TYPE           # Type text into an input field · 向输入框填写文字
  optional:
    - PRESS_KEY        # Keyboard keys (Enter / Esc / Tab etc.) · 键盘按键
    - RUN_JS           # Execute a JavaScript expression · 执行 JS 表达式
    - VISUAL_ANALYZE   # Screenshot + AI visual analysis (optional, failure is non-fatal) · 截图+视觉分析（可选）
    - SCREEN_RECORD    # Record screen (video format only) · 录屏（仅视频格式需要）
---

# Tutorial Generator · 教程生成器

Explore a website and automatically generate an illustrated step-by-step tutorial.
根据 URL 自动探索网站并生成图文教程。

---

## Capability Mapping · 能力映射（执行前自检）

This skill uses abstract capability identifiers. **Before starting, confirm which tool maps to each capability.**
本 skill 使用抽象能力标识符。**开始前，先确认当前工具环境中各能力对应的具体工具名。**

| Identifier · 标识符 | Description · 说明 | Your tool · 你的工具 |
|--------------------|--------------------|---------------------|
| `NAVIGATE` | Open / navigate to a URL · 打开/跳转 URL | fill in · 填入 |
| `CAPTURE` | Screenshot and save to file · 截图并保存文件 | fill in · 填入 |
| `READ_PAGE` | Read page structure (compact = interactive only, full = all content) · 读取页面结构 | fill in · 填入 |
| `CLICK` | Click an element · 点击元素 | fill in · 填入 |
| `TYPE` | Type into a field (clears first) · 向输入框填写文字（自动清空） | fill in · 填入 |
| `PRESS_KEY` | Press a key: Enter / Esc / Tab · 键盘按键（可选） | fill in · 填入 |
| `RUN_JS` | Execute JS, read DOM · 执行 JS（可选） | fill in · 填入 |
| `VISUAL_ANALYZE` | Screenshot + AI visual understanding · 截图+视觉理解（可选，失败不阻塞） | fill in · 填入 |
| `SCREEN_RECORD` | Start/stop screen recording · 开始/停止录屏（视频格式需要） | fill in · 填入 |

> **Important · 重要：** `CAPTURE` must save a clean screenshot file — do NOT use `VISUAL_ANALYZE` as a substitute.
> `CAPTURE` 必须保存干净截图文件，不可用 `VISUAL_ANALYZE` 替代。
> If your platform combines both into one tool (e.g. Hermes `browser_vision`), map both identifiers to it.
> 如果你的工具将截图和视觉分析合并为一个（如 Hermes 的 `browser_vision`），两个标识符都映射到它即可。

**Browser mode · 浏览器模式：**
- **Real browser** — connects to user's running browser, inherits existing cookies · **真实浏览器** — 连接用户正在运行的浏览器，自动继承已登录 Cookie
- **Sandbox** — fresh browser, no login state · **沙箱** — 全新浏览器，无任何登录态

---

## Full Workflow · 完整流程

### Phase 0: Initialization · 初始化

Before doing anything, **ask the user:**
开始任何操作前，**必须先问用户：**

```
1. Target audience? (Beginner / Advanced, default: Beginner)
   目标读者？（新手 / 进阶，默认新手）

2. Which features to document? (Leave blank = scout first, user picks from results)
   记录哪些功能？（留空 = 先侦察，再由用户从结果中选择，不自动决定）

3. Login required? If yes, provide credentials or log in beforehand.
   需要登录吗？如需要，请提前登录或提供账号信息。

4. Output language? (default: English — any language supported)
   教程语言？（默认 English，支持任意语言，如 中文 / Español / 日本語 / 한국어 / Français / Deutsch / Português / العربية）

5. Output format?
   输出格式？
   - markdown   (default) — plain text, relative screenshot paths, easy to edit · 纯文本，截图相对路径，便于二次编辑
   - html       — single file, screenshots embedded as base64, shareable · 单文件，截图内嵌，可直接分享
   - pdf        — requires pandoc or wkhtmltopdf · 需要 pandoc 或 wkhtmltopdf
   - video      — screen recording, requires ffmpeg · 录屏视频，需要 ffmpeg
                  Add-ons · 附加选项（combinable · 可组合，all off by default · 默认全不开启）：
                    +sub        — burn subtitles into video · 字幕烧录
                    +audio      — TTS narration · TTS 旁白
                    +slide=N    — slideshow: N seconds per image (default 3) · 截图序列，每张 N 秒（默认 3）
                  Examples · 示例：
                    "video+sub+audio+slide=4" = slideshow, 4s/image, subtitles + narration
                                               截图序列，每张 4 秒，带字幕和旁白
   - Combine · 多选："markdown video+sub" = doc + subtitled video · 图文教程 + 带字幕视频
```

> **Note · 注意：** If question 2 is blank, complete Phase 1, present results, and **wait for user to select modules** — do not skip this step or generate tutorial automatically.
> 问题 2 留空时，必须完成 Phase 1 侦察并展示结果，**等用户选择模块后才能继续**，不得跳过直接生成教程。

---

### Phase 1: Scouting · 侦察

```
1. NAVIGATE(url)
2. CAPTURE → shot_00_home.png
3. READ_PAGE(full) → read complete page structure · 读取完整页面结构
4. Extract: page title, all nav links, core feature areas · 提取：页面标题、所有导航链接、核心功能区域
   (VISUAL_ANALYZE optional for layout understanding · 可选，辅助理解布局)
```

After scouting, output the following and **end the response — do not proceed further:**
侦察完成后，输出以下内容并**结束本轮回复，不得继续执行任何步骤：**

---
## Scouting Results · 侦察结果 — {Website Name · 网站名称}

Modules found · 发现以下功能模块：
1. {Module A · 模块A} (path: {url})
2. {Module B} (path: {url})
3. {Module C} (path: {url})

💡 Recommended for beginners · 建议优先记录（适合新手）: {list 2–4 · 列出 2-4 个}

**Reply with module numbers (e.g. "1 3") or say "use recommendation". · 请回复要记录的编号（如 "1 3"），或回复"按建议来"。**
**I will not start until I receive your reply. · 收到你的回复后才会开始探索和截图。**

---

> Phase 2+ must not start until user has selected modules. · Phase 2 及后续步骤在收到用户选择前不得启动。

---

### Phase 2: Login Detection & Handling · 登录墙检测与处理

#### Step 1: Check login state · 检查当前登录状态

After navigating, **check if already logged in — do not assume otherwise.**
导航到目标页面后，**先检查是否已登录，不要假设未登录。**

```
CAPTURE → shot_login_check.png
READ_PAGE(compact)

Logged-in signals (any one → proceed to Phase 3) · 已登录信号（满足任意一项 → 直接进入 Phase 3）：
  ✅ User avatar / username / email visible · 显示用户头像、用户名、邮箱
  ✅ URL is /dashboard, /home, /console, /app etc. · URL 是后台路径
  ✅ "Sign Out" / "Log Out" button present · 存在退出登录按钮
  ✅ Personal account menu in top nav · 顶部导航含账户菜单

Logged-out signals (any one → go to Step 2) · 未登录信号（满足任意一项 → 进入步骤二）：
  ❌ URL contains /login, /signin, /auth · URL 含登录路径
  ❌ Password input with no user info · 存在密码输入框且无用户信息
  ❌ "Please log in" / "Sign in to continue" prompt · 出现登录提示
  ❌ Page body empty or covered by login overlay · 页面主体为空或被遮罩覆盖
```

#### Step 2: Identify available login methods · 识别可用登录方式

**Read the login page first — do not ask the user yet.**
**先读取登录页结构，不要直接问用户。**

```
READ_PAGE(full) — identify · 识别：
  - HAS_EMAIL_PASSWORD = email input + password input present · 邮箱+密码输入框
  - HAS_GOOGLE         = "Continue with Google" button · Google 登录按钮
  - HAS_GITHUB         = "Continue with GitHub" button · GitHub 登录按钮
  - HAS_OTHER_OAUTH    = other third-party login buttons · 其他第三方登录按钮
  - HAS_MAGIC_LINK     = "Send magic link" option · 发送登录链接选项
  - HAS_SMS_CODE       = phone number input · 手机号输入框
```

Ask the user, **listing only actually detected methods · 只列出实际检测到的方式询问用户：**

```
IF HAS_EMAIL_PASSWORD AND (HAS_GOOGLE OR other OAuth):
  → List all detected options · 列出所有检测到的方式

ELSE IF email/password only:
  → "Email/password login required. Please provide your credentials."
    "需要邮箱密码登录，请提供账号信息。"

ELSE IF OAuth only:
  → "Only {OAuth name} login available. Please authorize in your browser."
    "仅支持 {OAuth名称} 登录，请在浏览器中完成授权。"
```

#### Step 3: Execute login · 执行登录

**Strategy A: Email / password · 邮箱密码登录**
```
CLICK(email field), TYPE(email), PRESS_KEY(Tab) or CLICK(password field)
TYPE(password), CLICK(login button), wait up to 10s
```

**Strategy B: Verification code · 验证码登录**
```
IF code input appears after submit · 提交后出现验证码输入框:
  → Pause: "Code sent to {email/phone}. Please share the code."
    暂停："已向 {邮箱/手机} 发送验证码，请告诉我验证码。"
  TYPE(code field, user-provided code), CLICK(confirm)
```

**Strategy C: OAuth**
```
CLICK(OAuth button)
→ Pause: "Please complete authorization in your browser, then let me know."
  暂停："请在浏览器中完成授权，完成后告诉我继续。"
```

**Strategy D: Skip · 跳过登录**
```
Record skipped modules · 记录跳过的模块
Mark in tutorial: "⚠️ This feature requires login. · 此功能需要登录后使用。"
```

#### Step 4: Verify login · 验证登录成功

```
CAPTURE → shot_login_verify.png + READ_PAGE(compact)
IF logged-in signals present · 通过 → proceed to Phase 3 · 进入 Phase 3
IF failed · 失败 → prompt user to retry, max 2 attempts · 提示重试，最多 2 次
```

---

### Phase 3: Module Exploration · 逐模块探索

**Video init · 视频录制初始化（video format only · 仅视频格式需要）：**

```
Detect recording method in priority order · 按优先级检测录制方案：
  A — SCREEN_RECORD available           → RECORD_MODE = "native"
  B — Playwright-driven browser         → RECORD_MODE = "playwright"
  C — macOS (screencapture)             → RECORD_MODE = "screencapture"
  D — Linux (recordmydesktop)           → RECORD_MODE = "recordmydesktop"
  E — ffmpeg only                       → RECORD_MODE = "slideshow"
  F — nothing available                 → RECORD_MODE = "none", inform user

Start recording, record T0 (ms), initialize click_events = [], subtitles = []
```

**Screenshot rules · 截图规则（mandatory · 强制执行，no exceptions · 不得省略）：**
- 1 CAPTURE on entering each module (overview) · 每个模块进入时截 1 张首屏
- 1 CAPTURE before + 1 after each action step · 每步操作前后各 1 张
- Extra CAPTURE on modal / dropdown / toast · 弹窗/下拉/提示额外 1 张
- Minimum: total ≥ modules × 3 · 最低保底：总截图数 ≥ 模块数 × 3

⚠️ **ANNOTATE mode is strictly banned in Phase 3. · Phase 3 期间严禁使用 ANNOTATE（带红框标注模式）。**
Use `READ_PAGE(compact)` for navigation refs — text list, no visual markers.
用 `READ_PAGE(compact)` 获取元素 ref，返回文字列表，不产生任何截图。

```
Correct order per step · 每步正确顺序：
  1. CAPTURE → shot_xxx_before.png         ← clean screenshot · 干净截图，先保存
  2. READ_PAGE(compact)                    ← get element refs · 获取 ref，不截图
  3. IF video: wait(1.5s), record (x,y)   ← focus pause · 停顿让观众聚焦
  4. Execute action (CLICK / TYPE)         · 执行操作
  5. Wait for response (up to 3s)         · 等待页面响应
  6. CAPTURE → shot_xxx_after.png          ← clean screenshot · 干净截图，后保存
  7. VISUAL_ANALYZE (optional, skip on failure · 可选，失败直接跳过)
  8. IF +sub/+audio: append to subtitles  · 追加字幕条目
  9. Record step description              · 记录步骤描述

Extra CAPTUREs · 额外截图：
  Modal / overlay   → shot_{nn}_step{s}_modal.png
  Dropdown/tooltip  → shot_{nn}_step{s}_dropdown.png
  Scroll needed     → scroll, then shot_{nn}_step{s}_scroll.png
```

⛔ **Prohibited · 禁止：**
- Skipping CAPTURE because "page didn't change much" · 因"页面变化不大"跳过截图
- Reusing a previous screenshot · 复用已有截图
- Saving VISUAL_ANALYZE / ANNOTATE output as a tutorial screenshot · 将标注图存入教程

---

**Action type classification · 操作类型分类：**

| Action · 操作 | Execute? · 执行？ | Screenshots · 截图策略 |
|--------------|-----------------|----------------------|
| Browse / view · 浏览/查看 | ✅ Full | Entry + key content areas |
| Create / add · 创建/新增 | ✅ Full, sample data | Empty form + filling + success |
| Edit / modify · 编辑/修改 | ✅ Sample data, revert if needed | Original + edit form + saved |
| Delete · 删除 | ⚠️ Cancel at confirm dialog · 截图到确认弹窗后取消 | Trigger + dialog + after cancel |
| Payment · 支付/充值 | ❌ Screenshot only, do not proceed · 截图到支付页即止 | Entry + payment page |
| Settings / permissions · 设置/权限 | ⚠️ Screenshot only, no changes · 仅截图，不修改 | Settings overview + key areas |

---

### Phase 4: Compile Tutorial · 汇总与生成教程

````markdown
# {Website} — Tutorial · 使用教程

> Audience · 目标读者: {audience}
> Language · 语言: {output language}
> Updated · 更新时间: {date}

## Table of Contents · 目录
{auto-generated · 自动生成}

---

## Prerequisites · 准备工作
{login / registration notes · 登录/注册说明（如有）}

---

## {Module A} Guide · 使用指南

### Step 1 · 第 1 步: {action}

![{action}]({screenshot_path})

{1–2 sentences: WHY + caveats · 说明目的和注意事项}

...
````

**Post-processing annotation · 截图标注（事后处理，非实时）：**
If ANNOTATE is available as a post-processing tool, annotate saved clean screenshots after Phase 3 is complete.
如果 ANNOTATE 作为事后处理工具可用，在 Phase 3 完成后对干净截图进行标注。
Do NOT use ANNOTATE during live exploration. · 不得在探索过程中实时使用 ANNOTATE。

---

### Phase 5: Output & Delivery · 输出与交付

**Base output · 基础输出（all formats · 所有格式共用）：**
```
Screenshots → {domain}/screenshots/shot_{nn}_{module}_{step}_{before|after}.png
```

**Markdown:** `{domain}-tutorial.md` — relative screenshot paths · 相对路径引用截图

**HTML:** `{domain}-tutorial.html` — base64 screenshots embedded, single shareable file · 截图内嵌，单文件可分享

**PDF:** pandoc → wkhtmltopdf → fallback to HTML + suggest browser print · 降级输出 HTML，建议浏览器打印

**Video · 视频：**
```
1. Stop recording (per RECORD_MODE) · 停止录制

2. IF +sub OR +audio: generate SRT
   Timing strategy · 时间轴策略（two modes, do NOT mix · 两种模式严禁混用）：

   [Real recording · 真实录屏]
   Use Phase 3 timestamps: start = T_start, end = T_end + 1500ms (min 1500ms)
   使用 Phase 3 时间戳，最短 1500ms

   [Slideshow · 截图序列]
   ⚠️ Ignore T_start/T_end — calculate from image sequence:
   不使用 T_start/T_end，按图片序号计算：
     subtitle N: start = N × SLIDE_DURATION, end = (N+1) × SLIDE_DURATION - 0.2s
     TTS audio must fit within SLIDE_DURATION (speed up via atempo if needed, max 1.5x)
     TTS 音频必须在 SLIDE_DURATION 内（atempo 加速，最大 1.5x）

3. IF +audio: TTS narration · TTS 旁白
   Priority · 优先级: edge-tts → OpenAI TTS → macOS say → gtts → subtitles only · 降级为纯字幕

4. Compose MP4 with ffmpeg
   IF click_events not empty → apply zoompan focus effect at each click coordinate
   如有点击坐标 → 对每个点击位置应用 zoompan 推进效果（1.0→1.3→1.0）
   IF no coordinates → fallback to center zoom · 降级为居中缩放

5. Clean up temp files · 清理临时文件
```

**Delivery · 交付确认：**
```
List all generated files · 列出所有文件 → show 30-line preview · 展示前 30 行预览
Ask: "Add modules, re-explore any step, or export to another format?"
询问："是否需要补充模块、重新探索步骤，或转换为其他格式？"
```

---

## Error Handling · 错误处理

| Situation · 情况 | Action · 处理方式 |
|-----------------|-----------------|
| Page load timeout · 页面加载超时 | Wait 5s, retry once, skip and log if still failing · 等待 5s 重试，失败则跳过记录 |
| Element not found · 元素找不到 | READ_PAGE to refresh refs, retry once; CAPTURE full page and continue · 重新获取 ref 重试，失败截整页继续 |
| Modal blocking · 弹窗/遮罩阻挡 | PRESS_KEY(Esc) or click overlay; log if persists · Esc 关闭，失败则记录 |
| VISUAL_ANALYZE failure · 视觉分析失败 | Skip immediately, CAPTURE is unaffected · 直接跳过，截图不受影响 |
| Too few screenshots · 截图数量不足 | Re-capture missing steps before Phase 4 · Phase 4 前补截缺失步骤 |
| No browser tools · 无浏览器工具 | Fallback to fetch + HTML, text-only tutorial · 降级为纯文字教程 |
| Slow dynamic content · 动态内容加载慢 | Wait 2s before CAPTURE · 截图前等待 2s |
| TTS unavailable · TTS 不可用 | Fallback to subtitles only · 降级为纯字幕 |
| ffmpeg unavailable · ffmpeg 不可用 | Skip video, keep screenshots, suggest install · 跳过视频，保留截图 |

---

## Key Principles · 关键原则

1. **Confirm scope before acting · 先确认范围再动手** — never explore before user confirms module list · 用户未确认前不探索任何链接
2. **CAPTURE is always independent · CAPTURE 必须独立调用** — saved separately from any analysis tool · 截图保存与视觉分析工具解耦
3. **Before + after each step · 操作前后各一张图** — show "what to click" and "what happens next" · 让读者看到点什么和结果
4. **ANNOTATE banned in Phase 3 · Phase 3 禁用 ANNOTATE** — use READ_PAGE for refs; never save annotated screenshots · 用 READ_PAGE 获取 ref，标注图不入教程
5. **VISUAL_ANALYZE is optional · VISUAL_ANALYZE 是增强** — skip on failure, never save as screenshot · 失败跳过，输出不保存为截图
6. **Always pause at login walls · 遇到登录墙必须暂停** — never guess or bypass auth · 不猜测不绕过认证
7. **Imperative voice for steps · 步骤用祈使句** — "Click Login" not "User should click" · "点击登录"而非"用户需要点击"
8. **Match output language · 教程语言匹配用户选择** — all tutorial text in the language selected in Phase 0 · 所有教程文字按 Phase 0 选择的语言生成

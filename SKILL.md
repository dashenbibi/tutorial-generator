---
name: tutorial-generator
version: 2.0.0
description: "自动生成网站图文教程。输入 URL，自动探索页面、截图、记录操作步骤，生成可分享的 Markdown 图文教程。当用户需要为某个网站或工具生成使用教程时使用。"
metadata:
  requires:
    capabilities:
      - NAVIGATE       # 导航到 URL
      - CAPTURE        # 截图并保存到文件
      - READ_PAGE      # 读取页面结构/内容
      - CLICK          # 点击页面元素
      - TYPE           # 向输入框填写文字
  optional:
    - PRESS_KEY        # 键盘按键（Enter/Esc/Tab 等）
    - RUN_JS           # 执行 JavaScript 表达式
    - VISUAL_ANALYZE   # 截图 + AI 视觉分析（可选增强，失败不影响主流程）
    - ANNOTATE         # 在截图上标注交互元素
    - SCREEN_RECORD    # 录制屏幕（video 格式需要）
---

# Tutorial Generator

根据 URL 自动探索网站并生成图文教程。

---

## 能力映射（执行前自检）

本 skill 使用抽象能力标识符。**在开始前，先确认当前工具环境提供了哪些对应的具体工具**，填入下表：

| 能力标识符 | 说明 | 你的工具对应 |
|-----------|------|------------|
| `NAVIGATE` | 打开/跳转到指定 URL | 填入你的工具名 |
| `CAPTURE` | 截图并保存为文件 | 填入你的工具名 |
| `READ_PAGE` | 读取页面结构（compact=仅交互元素，full=完整内容） | 填入你的工具名 |
| `CLICK` | 点击元素（by ref / selector / coordinates） | 填入你的工具名 |
| `TYPE` | 向输入框填写文字（自动清空再输入） | 填入你的工具名 |
| `PRESS_KEY` | 键盘按键：Enter / Esc / Tab 等 | 填入你的工具名（可选） |
| `RUN_JS` | 执行 JS 表达式，读取 DOM 信息 | 填入你的工具名（可选） |
| `VISUAL_ANALYZE` | 截图 + AI 视觉理解，辅助判断页面状态 | 填入你的工具名（可选，失败不影响主流程） |
| `ANNOTATE` | 在截图上标注交互元素（如叠加编号/红框） | 填入你的工具名（可选） |
| `SCREEN_RECORD` | 开始/停止录屏，输出视频文件 | 填入你的工具名（video 格式需要） |

> **重要：** `CAPTURE` 必须保存截图文件，不可只做视觉分析而不留文件。
> 如果你的平台截图和视觉分析是同一个工具（如 Hermes 的 browser_vision），直接用它即可——它内部会保存文件，CAPTURE 和 VISUAL_ANALYZE 都映射到它。
> 如果你的工具只做视觉分析、不保存文件，则需要在调用前先用其他方式单独存图。

**浏览器模式（影响登录处理）：**
- **真实浏览器模式** — 连接用户正在运行的浏览器，自动继承已登录的 Cookie 和 Session
- **沙箱模式** — 每次启动全新空白浏览器，无任何登录态，需额外处理认证

如不确定当前模式，询问用户或尝试访问目标网站并检查是否已登录。

---

## 完整流程

### Phase 0：初始化

在开始任何操作前，**必须先问用户**：

```
1. 教程目标读者是谁？（新手 / 进阶用户，默认新手）
2. 想记录哪些功能？（留空 = 先侦察，再由用户从侦察结果中选择，不自动决定）
3. 需要登录吗？如需要，请提前登录或提供账号信息。
4. 教程输出格式？
   - markdown （默认）— 纯文本，截图用相对路径引用，便于二次编辑
   - html       — 独立 HTML 文件，截图内嵌为 base64，可直接在浏览器打开分享
   - pdf        — 需要本地有 pandoc 或 wkhtmltopdf，先生成 HTML 再转换
   - video      — 录制操作视频，需要 ffmpeg；可与其他格式同时选
                  视频附加选项（可组合，默认全不开启）：
                    +sub    — 生成 SRT 字幕并烧录进视频
                    +audio  — 用 TTS 生成旁白音频并混入视频
                  示例："video+sub+audio" = 视频 + 字幕 + 旁白（最完整）
   - 多选示例："markdown video+sub" = 同时输出图文教程和带字幕视频
```

**注意：** 问题 2 留空时，必须完成 Phase 1 侦察并展示结果，**等用户选择模块后才能继续**，不得跳过选择步骤直接生成教程。

如果用户已明确列出要记录的功能（如"记录注册和支付流程"），可跳过 Phase 0 和 Phase 1 侦察，直接进入 Phase 2。

---

### Phase 1：侦察

```
1. NAVIGATE(url)
2. CAPTURE → shot_00_home.png
3. READ_PAGE(full) → 读取完整页面结构
4. 提取：页面标题、所有主导航链接、核心功能区域
   （如有 VISUAL_ANALYZE，可额外调用辅助理解布局，但非必须）
```

侦察完成后，输出以下格式并结束本轮回复。**本轮回复必须以这段内容结尾，不得在后面继续执行任何步骤：**

---
## 侦察结果 — {网站名称}

发现以下功能模块：
1. {模块A}（路径：{url}）
2. {模块B}（路径：{url}）
3. {模块C}（路径：{url}）
4. {模块D}（路径：{url}）

💡 建议优先记录（适合新手）：{列出 2-4 个}

**请选择要记录的模块编号（如 "1 3"），或回复"按建议来"。**
**收到你的回复后，我才会开始探索和截图。**

---

> Phase 2 和后续步骤在收到用户模块选择前不得启动。
> 当前状态：等待用户输入。

---

### Phase 2：登录墙检测与处理

#### 步骤一：先判断当前登录状态

导航到目标页面后，**先检查是否已登录**，不要假设未登录：

```
CAPTURE → shot_login_check.png
READ_PAGE(compact)

检查已登录信号（满足任意一项 → 已登录，直接进入 Phase 3）：
  ✅ 页面显示用户头像、用户名、邮箱地址
  ✅ URL 是 /dashboard、/home、/console、/app 等后台路径
  ✅ 页面存在"退出登录"/"Sign Out"/"Log Out"按钮
  ✅ 顶部导航含个人账户菜单

检查未登录信号（满足任意一项 → 未登录，进入步骤二）：
  ❌ URL 包含 /login、/signin、/auth
  ❌ 页面存在密码输入框且无用户信息
  ❌ 出现"请登录"/"Sign in to continue"提示
  ❌ 页面主体为空或被登录遮罩覆盖
```

---

#### 步骤二：分析登录页，识别可用方式

**不要直接问用户，先读取登录页结构：**

```
READ_PAGE(full) 分析登录表单，识别以下元素：
  - HAS_EMAIL_PASSWORD = 页面存在邮箱输入框 + 密码输入框
  - HAS_GOOGLE         = 页面含 "Continue with Google" / Google 图标按钮
  - HAS_GITHUB         = 页面含 "Continue with GitHub" / GitHub 图标按钮
  - HAS_OTHER_OAUTH    = 页面含其他第三方登录按钮（记录名称）
  - HAS_MAGIC_LINK     = 页面含 "发送登录链接"/"Send magic link" 选项
  - HAS_SMS_CODE       = 页面含手机号输入框
```

根据识别结果，**只列出实际存在的选项**询问用户：

```
IF HAS_EMAIL_PASSWORD AND (HAS_GOOGLE OR HAS_OTHER_OAUTH):
  → 提示列出所有检测到的方式，让用户选择

ELSE IF HAS_EMAIL_PASSWORD ONLY:
  → 提示："该页面需要邮箱密码登录，请提供邮箱和密码"

ELSE IF ONLY OAuth（无密码表单）:
  → 提示："该页面只支持 {OAuth名称} 登录，需要你在浏览器中手动完成授权"
```

---

#### 步骤三：执行登录

**策略 A：邮箱/密码登录**

```
CLICK(邮箱输入框)
TYPE(邮箱输入框, 用户提供的邮箱)
CLICK(密码输入框) 或 PRESS_KEY(Tab)
TYPE(密码输入框, 用户提供的密码)
CLICK(登录按钮)
等待页面响应（最多 10s）
```

**策略 B：验证码（提交后触发）**

```
IF 提交后出现验证码输入框:
  → 暂停，提示用户："已向 {邮箱/手机号} 发送验证码，请告诉我收到的验证码"
  → 等待用户输入
  TYPE(验证码输入框, 用户提供的验证码)
  CLICK(确认按钮)
```

**策略 C：OAuth 第三方登录**

```
CLICK(OAuth 按钮)
→ 暂停自动化，提示用户在浏览器中完成授权，完成后告知继续
```

**策略 D：跳过登录**

```
记录：以下模块因需要登录已跳过：{模块列表}
在教程中标注"⚠️ 此功能需要登录后使用"
直接进入 Phase 3（跳过需登录的模块）
```

---

#### 步骤四：登录成功验证

```
CAPTURE → shot_login_verify.png
READ_PAGE(compact)

检查已登录信号（同步骤一）：
  IF 通过 → 进入 Phase 3
  IF 失败 → 提示用户确认账密，返回步骤三重试（最多 2 次）
```

---

### Phase 3：逐模块探索

**视频录制初始化（仅 video 格式需要）：**

```
IF 输出格式包含 video:

  按优先级检测可用录制方案，选第一个可用的：
    方案 A — SCREEN_RECORD 可用          → RECORD_MODE = "native"
    方案 B — 浏览器由 Playwright 驱动    → RECORD_MODE = "playwright"（启动时配置录制目录）
    方案 C — macOS 系统（screencapture） → RECORD_MODE = "screencapture"
    方案 D — Linux 系统（recordmydesktop）→ RECORD_MODE = "recordmydesktop"
    方案 E — 仅有 ffmpeg                 → RECORD_MODE = "slideshow"（截图序列合成）
    方案 F — 无任何工具                  → RECORD_MODE = "none"，告知用户并跳过视频

  启动录制（按 RECORD_MODE 使用对应命令）
  记录开始时间 T0（毫秒）
  初始化 subtitles = []
```

**截图规则（强制，不得省略）：**
- 每个模块进入时：CAPTURE 1 张首屏
- 每个操作步骤：操作前 CAPTURE 1 张 + 操作后 CAPTURE 1 张
- 弹窗/模态框/下拉菜单出现时：额外 CAPTURE 1 张
- 最低保底：总截图数 ≥ 模块数 × 3

⚠️ **截图用途严格区分，不得混用：**

| 用途 | 调用方式 | 说明 |
|------|---------|------|
| 教程截图（存文件） | `CAPTURE`，**不带标注参数** | 干净截图，读者看的，不能有红框/编号 |
| 页面导航分析（内部） | `VISUAL_ANALYZE` 或 `ANNOTATE` | Agent 自己看的，找元素用，**结果不保存为教程截图** |

操作顺序必须是：
1. `CAPTURE → shot_xxx.png`（先保存干净截图）
2. `VISUAL_ANALYZE / ANNOTATE`（可选，帮助找元素，结果仅供内部使用）
3. 执行操作（CLICK / TYPE）
4. `CAPTURE → shot_xxx_after.png`（保存操作后干净截图）

对每个确认的模块：

```
FOR each module in confirmed_scope:

  【模块首屏】
  1. NAVIGATE(module_url)
  2. 等待页面加载完成
  3. CAPTURE → shot_{nn}_{module}_overview.png        ← 先保存
  4. READ_PAGE(compact)                               ← 理解页面结构
  5. VISUAL_ANALYZE（可选，失败跳过）
  6. 记录：URL、页面标题

  【逐步操作】
  FOR each step:
    a. CAPTURE → shot_{nn}_step{s}_before.png          ← 先保存
    b. IF +sub OR +audio: 记录 T_start = now() - T0
    c. 执行操作（CLICK / TYPE / PRESS_KEY）
    d. 等待页面响应（最多 3s）
    e. CAPTURE → shot_{nn}_step{s}_after.png           ← 先保存
    f. VISUAL_ANALYZE（可选，失败跳过）
    g. IF +sub OR +audio:
         subtitles.append({ start: T_start, end: now()-T0+1500, text: "{步骤描述}" })
    h. 记录步骤：{操作描述 + 预期结果}

  【额外截图时机】
  出现弹窗/模态框  → CAPTURE → shot_{nn}_step{s}_modal.png
  出现下拉菜单    → CAPTURE → shot_{nn}_step{s}_dropdown.png
  需要滚动查看    → 滚动后 CAPTURE → shot_{nn}_step{s}_scroll.png
```

⛔ **禁止：**
- 因"页面变化不大"跳过截图
- 因"已有类似截图"复用截图
- 用 VISUAL_ANALYZE 替代 CAPTURE

---

**操作类型分类处理：**

| 操作类型 | 是否执行 | 截图策略 |
|---------|---------|---------|
| 浏览 / 查看 | ✅ 完整执行 | 进入前 + 关键内容区 |
| 创建 / 新增 | ✅ 完整执行，用示例数据填写 | 空表单 + 填写中 + 创建成功后 |
| 编辑 / 修改 | ✅ 用示例数据执行，保存后可还原 | 原始内容 + 编辑表单 + 保存结果 |
| 删除 | ⚠️ 截图到确认弹窗后点取消，不实际删除 | 触发按钮 + 确认弹窗 + 取消后 |
| 支付 / 充值 | ❌ 不执行，截图到支付页即止 | 入口按钮 + 支付页全览 |
| 权限 / 账户设置 | ⚠️ 仅截图展示，不修改任何配置 | 设置页全览 + 关键选项区 |

**编辑操作步骤：**
```
1. CAPTURE → 记录原始状态
2. CLICK(编辑入口)
3. CAPTURE → 记录表单打开状态
4. TYPE(各字段, 示例数据)
5. CAPTURE → 记录填写完成状态
6. CLICK(保存按钮)
7. CAPTURE → 记录保存结果
```

**删除操作步骤：**
```
1. CAPTURE → 记录目标项存在状态
2. CLICK(删除入口)
3. CAPTURE → 记录触发方式
4. IF 出现确认弹窗:
     CAPTURE → 记录确认弹窗
     CLICK(取消按钮)        ← 不点确认，不实际删除
     CAPTURE → 记录取消后恢复
5. 教程中说明："⚠️ 点击确认后永久删除，无法恢复"
```

---

### Phase 4：汇总与生成教程

**教程结构模板：**

````markdown
# {网站名称} 使用教程

> 本教程面向：{目标读者}
> 更新时间：{date}

## 目录
{自动生成}

---

## 准备工作
{登录/注册说明（如有）}

---

## {模块A} 使用指南

### 第 1 步：{操作描述}

![{操作描述}]({screenshot_path})

{1-2 句说明 WHY 和注意事项}

### 第 2 步：{操作描述}
...

---

## 常见问题
{探索过程中遇到的异常/提示，整理为 Q&A}
````

**截图标注（可选后处理，非实时标注）：**
- 如需标注，在教程生成阶段对已保存的干净截图进行后处理
- 标注内容：用红框圈出操作区域、箭头指向关键按钮
- **不使用 ANNOTATE 的实时标注模式**——那会把 Agent 的导航编号（@e1 @e2）暴露给读者

---

### Phase 5：输出与交付

**基础输出（所有格式共用）：**
```
截图统一保存到：{网站域名}/screenshots/
命名规范：shot_{nn}_{module}_{step}_{before|after}.png
```

**Markdown：**
```
输出：{网站域名}-tutorial.md
截图引用：![描述](screenshots/shot_xx_xxx.png)  ← 相对路径
```

**HTML：**
```
输出：{网站域名}-tutorial.html
截图内嵌为 base64，添加基础 CSS，单文件可直接分享
```

**PDF：**
```
依赖检测（优先级）：pandoc → wkhtmltopdf → 降级输出 HTML + 提示用浏览器打印
```

**视频合成（video 格式）：**
```
1. 停止录制（按 RECORD_MODE 对应方式）
2. IF +sub OR +audio: 生成 SRT 字幕文件
3. IF +audio: 用 TTS 生成旁白（优先级：edge-tts → OpenAI TTS → say → gtts）
4. ffmpeg 合成 MP4（按 RECORD_MODE 选输入源，按选项叠加字幕/音频）
5. 清理临时文件

TTS 优先级：
  edge-tts（免费，质量好）→ OpenAI TTS（需 API Key）→ macOS say（内置）→ gtts → 降级为纯字幕

视频选项组合：
  "video"           = 纯录屏
  "video+sub"       = 录屏 + 字幕烧录
  "video+audio"     = 录屏 + TTS 旁白
  "video+sub+audio" = 录屏 + 字幕 + 旁白
```

**交付确认：**
```
列出所有生成的文件 → 展示教程前 30 行预览 → 询问是否补充或转换格式
```

---

## 错误处理

| 情况 | 处理方式 |
|------|---------|
| 页面加载超时 | 等待 5s 重试一次，失败则跳过并记录 |
| 元素找不到 | READ_PAGE 重新获取最新元素引用，再重试一次；仍失败则 CAPTURE 整页继续，不中断 |
| 弹窗/遮罩阻挡 | PRESS_KEY(Esc) 或点击遮罩关闭；失败则记录为已知问题 |
| VISUAL_ANALYZE 失败 | 直接跳过，不重试；截图由独立 CAPTURE 保证 |
| 截图数量不足 | 探索结束后清点，不足则补截再进入 Phase 4 |
| 无浏览器工具 | 降级为 fetch + HTML 解析，生成纯文字步骤（无截图） |
| 动态内容加载慢 | CAPTURE 前等待 2s，或等待特定元素出现 |
| TTS 全部不可用 | 降级为纯字幕（+sub），告知用户 |
| ffmpeg 不可用 | 跳过视频合成，保留截图序列，提示用户安装 |

---

## 关键原则

1. **先确认范围，再动手** — 不要在用户未确认前探索所有链接
2. **CAPTURE 独立调用** — 截图必须先保存文件，不依赖视觉分析工具
3. **操作前后各一张图** — 让读者看到"点什么"和"变成什么"
4. **VISUAL_ANALYZE 是增强，不是依赖** — 失败直接跳过，不影响主流程
5. **遇到登录墙必须暂停** — 不尝试猜测或绕过认证
6. **步骤描述用祈使句** — "点击「登录」按钮"而非"用户需要点击"

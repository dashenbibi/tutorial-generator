---
name: tutorial-generator
version: 3.0.0
description: "Automatically generate illustrated tutorials for any website. Given a URL, explore pages, take screenshots, record steps, and produce tutorials in Markdown, HTML, PDF, or Video formats."
metadata:
  requires:
    capabilities:
      - NAVIGATE       # Navigate to a URL
      - CAPTURE        # Take a screenshot and save to file
      - READ_PAGE      # Read page structure/content
      - CLICK          # Click a page element
      - TYPE           # Type text into an input field
  optional:
    - PRESS_KEY        # Keyboard keys (Enter / Esc / Tab etc.)
    - RUN_JS           # Execute a JavaScript expression
    - VISUAL_ANALYZE   # Screenshot + AI visual analysis (optional enhancement, failure does not block main flow)
    - SCREEN_RECORD    # Record screen (required for video format)
---

# Tutorial Generator

Explore a website and automatically generate an illustrated step-by-step tutorial.

---

## Capability Mapping (Self-check before starting)

This skill uses abstract capability identifiers. **Before starting, confirm which tool in your environment maps to each capability:**

| Identifier | Description | Your tool |
|-----------|-------------|-----------|
| `NAVIGATE` | Open / navigate to a URL | fill in |
| `CAPTURE` | Take a screenshot and save to file | fill in |
| `READ_PAGE` | Read page structure (compact = interactive elements only, full = all content) | fill in |
| `CLICK` | Click an element (by ref / selector / coordinates) | fill in |
| `TYPE` | Type text into an input field (clears existing text first) | fill in |
| `PRESS_KEY` | Press a keyboard key: Enter / Esc / Tab etc. | fill in (optional) |
| `RUN_JS` | Execute a JS expression, read DOM info | fill in (optional) |
| `VISUAL_ANALYZE` | Screenshot + AI visual understanding, helps identify page state | fill in (optional, failure is non-fatal) |
| `SCREEN_RECORD` | Start / stop screen recording, output video file | fill in (video format only) |

> **Important:** `CAPTURE` must save a clean screenshot file. Do NOT use `VISUAL_ANALYZE` as a substitute.
> If your platform combines screenshot and visual analysis into one tool (e.g. Hermes `browser_vision`),
> map both `CAPTURE` and `VISUAL_ANALYZE` to it — it saves the file internally.
> If your tool only analyzes visually without saving a file, find another way to save screenshots first.

**Browser mode (affects login handling):**
- **Real browser mode** — connects to the user's running browser, inherits existing cookies and sessions
- **Sandbox mode** — fresh browser each run, no login state, requires explicit authentication

---

## Full Workflow

### Phase 0: Initialization

Before doing anything, **ask the user:**

```
1. Who is the target audience? (Beginner / Advanced, default: Beginner)
2. Which features to document? (Leave blank = scout first, then user picks from results)
3. Login required? If yes, provide credentials or log in beforehand.
4. Output language? (default: English — any language supported, e.g. 中文, Español, 日本語, 한국어, Français, Deutsch, Português, العربية...)
5. Output format?
   - markdown     (default) — plain text, screenshots referenced by relative path, easy to edit
   - html         — single HTML file, screenshots embedded as base64, shareable directly
   - pdf          — requires pandoc or wkhtmltopdf locally; falls back to HTML
   - video        — screen recording + optional subtitles/narration, requires ffmpeg
                    Video add-ons (combinable, all off by default):
                      +sub        — burn SRT subtitles into video
                      +audio      — generate TTS narration and mix into video
                      +slide=N    — slideshow mode: display each screenshot for N seconds (default 3, range 2–8)
                    Examples:
                      "video"               = plain recording, no subtitles, no audio
                      "video+sub+audio"     = recording + subtitles + narration (full)
                      "video+sub+audio+slide=4" = slideshow, 4s per image, subtitles + narration
   - Combine formats: "markdown video+sub" = tutorial doc + subtitled video
```

**Note:** If question 2 is left blank, complete Phase 1 scouting, present results, and **wait for the user to select modules** before proceeding. Do not skip the selection step.

If the user has already named the features to document (e.g. "document signup and payment"), skip Phase 0 and Phase 1 and go straight to Phase 2.

---

### Phase 1: Scouting

```
1. NAVIGATE(url)
2. CAPTURE → shot_00_home.png
3. READ_PAGE(full) → read full page structure
4. Extract: page title, all main navigation links, core feature areas
   (VISUAL_ANALYZE optional — may help understand layout, not required)
```

After scouting, output the following and **end the response here. Do not proceed to any next step:**

---
## Scouting Results — {Website Name}

Modules found:
1. {Module A} (path: {url})
2. {Module B} (path: {url})
3. {Module C} (path: {url})
4. {Module D} (path: {url})

💡 Recommended for beginners: {list 2–4 core modules}

**Reply with the module numbers you want (e.g. "1 3"), or say "use recommendation".**
**I will not start exploring or taking screenshots until I receive your reply.**

---

> Phase 2 and beyond must not start until the user has selected modules.
> Current status: waiting for user input.

---

### Phase 2: Login Wall Detection & Handling

#### Step 1: Check current login state

After navigating to the target page, **check if already logged in — do not assume otherwise:**

```
CAPTURE → shot_login_check.png
READ_PAGE(compact)

Logged-in signals (any one → already logged in, proceed to Phase 3):
  ✅ Page shows user avatar, username, or email address
  ✅ URL is /dashboard, /home, /console, /app or similar
  ✅ Page has a "Sign Out" / "Log Out" button
  ✅ Top nav contains a personal account menu

Logged-out signals (any one → not logged in, go to Step 2):
  ❌ URL contains /login, /signin, /auth
  ❌ Page has a password input field and no user info
  ❌ Page shows "Please log in" / "Sign in to continue"
  ❌ Page body is empty or covered by a login overlay
```

---

#### Step 2: Analyze login page to identify available methods

**Do not ask the user yet — first read the login page structure:**

```
READ_PAGE(full) — identify:
  - HAS_EMAIL_PASSWORD = email input + password input present
  - HAS_GOOGLE         = "Continue with Google" / Google icon button present
  - HAS_GITHUB         = "Continue with GitHub" / GitHub icon button present
  - HAS_OTHER_OAUTH    = other third-party login buttons (note the name)
  - HAS_MAGIC_LINK     = "Send magic link" / "Email me a link" option present
  - HAS_SMS_CODE       = phone number input present
```

Then ask the user, **listing only the methods actually detected:**

```
IF HAS_EMAIL_PASSWORD AND (HAS_GOOGLE OR HAS_OTHER_OAUTH):
  → List all detected options and let the user choose

ELSE IF HAS_EMAIL_PASSWORD only:
  → "This page requires email/password login. Please provide your email and password."

ELSE IF OAuth only (no password form):
  → "This page only supports {OAuth name} login. Please complete authorization in your browser."
```

---

#### Step 3: Execute login

**Strategy A: Email / password**
```
CLICK(email field)
TYPE(email field, user-provided email)
PRESS_KEY(Tab) or CLICK(password field)
TYPE(password field, user-provided password)
CLICK(login button)
Wait up to 10s for response
```

**Strategy B: Verification code (triggered after submit)**
```
IF a verification code input appears after submitting:
  → Pause: "A code has been sent to {email/phone}. Please share the code."
  → Wait for user input
  TYPE(code field, code provided by user)
  CLICK(confirm button)
```

**Strategy C: OAuth**
```
CLICK(OAuth button)
→ Pause automation: "The {Google/GitHub} authorization page is open. Please complete login in your browser, then let me know."
→ Wait for user confirmation
```

**Strategy D: Skip login**
```
Record: the following modules were skipped (login required): {module list}
Mark skipped sections in tutorial as "⚠️ This feature requires login."
Proceed to Phase 3 with public-only modules.
```

---

#### Step 4: Verify login success

```
CAPTURE → shot_login_verify.png
READ_PAGE(compact)

Check logged-in signals (same as Step 1):
  IF passed → proceed to Phase 3
  IF failed → "Login did not succeed. Please check your credentials or choose another method."
             → Return to Step 3, retry up to 2 times
```

---

### Phase 3: Module Exploration

**Video recording initialization (video format only):**

```
IF output format includes video:

  Detect available recording method in priority order, use the first available:
    Option A — SCREEN_RECORD available           → RECORD_MODE = "native"
    Option B — Browser driven by Playwright      → RECORD_MODE = "playwright" (set record_video_dir at launch)
    Option C — macOS system (screencapture)      → RECORD_MODE = "screencapture"
    Option D — Linux system (recordmydesktop)    → RECORD_MODE = "recordmydesktop"
    Option E — Only ffmpeg available             → RECORD_MODE = "slideshow" (compose from screenshots)
    Option F — No tools available                → RECORD_MODE = "none", inform user, skip video

  Start recording (use the appropriate command for RECORD_MODE)
  Record start time T0 (milliseconds)
  Initialize click_events = [], subtitles = []
```

**Screenshot rules (mandatory, no exceptions):**
- On entering each module: 1 CAPTURE (overview)
- Each action step: 1 CAPTURE before + 1 CAPTURE after
- On modal / dropdown / toast appearing: 1 extra CAPTURE
- Minimum guarantee: total screenshots ≥ modules × 3

⚠️ **During Phase 3, ANNOTATE mode is strictly forbidden.**
Use `READ_PAGE(compact)` to get element refs for navigation — it returns a text list (e1, e2…) with no visual markers:

```
READ_PAGE(compact) → returns element list with refs
CLICK(e3)          → click directly using ref, no annotated screenshot needed
```

Screenshots use only two calls, strictly in order:

```
Step 1: CAPTURE → shot_xxx.png          ← save clean screenshot (no annotation flags)
Step 2: READ_PAGE(compact)              ← get element refs for navigation (no screenshot)
Step 3: execute action (CLICK / TYPE)
Step 4: CAPTURE → shot_xxx_after.png   ← save clean screenshot
```

VISUAL_ANALYZE (non-annotated) may optionally be called to understand page content, but **its output is not saved as a tutorial screenshot and does not replace CAPTURE.**

For each confirmed module:

```
FOR each module in confirmed_scope:

  [Module overview]
  1. NAVIGATE(module_url)
  2. Wait for page to finish loading
  3. CAPTURE → shot_{nn}_{module}_overview.png
  4. READ_PAGE(compact)
  5. VISUAL_ANALYZE (optional, skip on failure)
  6. Record: URL, page title

  [Step-by-step actions]
  FOR each step:
    a. CAPTURE → shot_{nn}_step{s}_before.png
    b. IF +sub OR +audio: record T_start = now() - T0
    c. IF video format:
         get action target screen coordinates (x, y) from element ref or bounding box
         wait(1.5s)                              ← pause so viewer can find visual focus
         click_events.append({ t: now()-T0, x, y })
    d. Execute action (CLICK / TYPE / PRESS_KEY)
    e. Wait for page response (up to 3s)
    f. CAPTURE → shot_{nn}_step{s}_after.png
    g. VISUAL_ANALYZE (optional, skip on failure — does not affect saved screenshots)
    h. IF +sub OR +audio:
         subtitles.append({ start: T_start, end: now()-T0+1500, text: "{step description}" })
    i. Record step: {action description + expected result}

  [Extra screenshot triggers]
  Modal / overlay appears    → CAPTURE → shot_{nn}_step{s}_modal.png
  Dropdown / tooltip appears → CAPTURE → shot_{nn}_step{s}_dropdown.png
  Page needs scrolling       → scroll, then CAPTURE → shot_{nn}_step{s}_scroll.png
```

⛔ **Prohibited:**
- Skipping a screenshot because "the page didn't change much"
- Reusing a previous screenshot for a different step
- Using VISUAL_ANALYZE output as a tutorial screenshot
- Using ANNOTATE / annotated screenshots anywhere in Phase 3

---

**Action type classification:**

| Action type | Execute? | Screenshot strategy |
|------------|---------|-------------------|
| Browse / view | ✅ Full execution | On entry + key content areas |
| Create / add | ✅ Full execution, use sample data | Empty form + filling + success state |
| Edit / modify | ✅ Use sample data, revert after if needed | Original + edit form + saved result |
| Delete | ⚠️ Screenshot to confirmation dialog, then cancel | Trigger + confirm dialog + after cancel |
| Payment / top-up | ❌ Screenshot to payment page only, do not proceed | Entry button + payment page overview |
| Permissions / account settings | ⚠️ Screenshot only, do not change anything | Settings overview + key option areas |

**Edit step-by-step:**
```
1. CAPTURE → record original state
2. CLICK(edit entry point)
3. CAPTURE → record edit form open
4. TYPE(each field, sample data)
5. CAPTURE → record filled-in state
6. CLICK(save button)
7. CAPTURE → record saved result
Note in tutorial: "Replace sample data with your own content."
```

**Delete step-by-step:**
```
1. CAPTURE → record target item exists
2. CLICK(delete entry: right-click menu / delete button / kebab menu)
3. CAPTURE → record how delete was triggered
4. IF confirmation dialog appears:
     CAPTURE → record the dialog (including warning text)
     CLICK(cancel button)     ← do NOT click confirm
     CAPTURE → record page returned to normal
5. Add to tutorial: "⚠️ Clicking Confirm permanently deletes this item and cannot be undone."
```

---

### Phase 4: Compile & Generate Tutorial

**Tutorial template:**

````markdown
# {Website Name} — Tutorial

> Audience: {target audience}
> Language: {output language}
> Updated: {date}

## Table of Contents
{auto-generated}

---

## Prerequisites
{login / registration notes if applicable}

---

## {Module A} Guide

### Step 1: {action description}

![{action description}]({screenshot_path})

{1–2 sentences explaining WHY and any caveats}

### Step 2: {action description}
...

---

## {Module B} Guide
...

---

## FAQ
{exceptions / prompts encountered during exploration, formatted as Q&A}
````

**Screenshot annotation (if ANNOTATE capability is available as post-processing):**
- Annotate saved clean screenshots after exploration is complete
- Mark click/input areas with a red box or arrow
- Do NOT use live ANNOTATE mode during Phase 3

---

### Phase 5: Output & Delivery

**Base output (all formats):**
```
Screenshots saved to: {domain}/screenshots/
Naming: shot_{nn}_{module}_{step}_{before|after}.png
```

**Markdown:**
```
Output: {domain}-tutorial.md
Screenshot refs: ![desc](screenshots/shot_xx_xxx.png)  ← relative paths
```

**HTML:**
```
Output: {domain}-tutorial.html
Screenshots embedded as base64 — single self-contained file, shareable directly
Include basic CSS (font, spacing, step numbering, max-width images)
```

**PDF:**
```
Dependency check (in order): pandoc → wkhtmltopdf → fallback to HTML + suggest browser print-to-PDF
```

**Video (video format):**

```
1. Stop recording (method depends on RECORD_MODE)

2. IF +sub OR +audio: generate SRT subtitle file
   Timing calculation — two separate strategies, do NOT mix:

   [Real recording mode — native / playwright / screencapture / recordmydesktop]
   Use actual timestamps from Phase 3:
     subtitle N: start = T_start_N, end = T_end_N + 1500ms
     If end - start < 1500ms, pad to 1500ms minimum

   [Slideshow mode]
   ⚠️ Do NOT use T_start / T_end. Calculate from image sequence:
     SLIDE_DURATION = value from +slide=N (default 3.0s)
     subtitle N: start = N × SLIDE_DURATION
     subtitle N: end   = (N+1) × SLIDE_DURATION - 0.2s
     TTS audio must be trimmed / sped up to fit within SLIDE_DURATION

3. IF +audio: generate TTS narration
   Priority: edge-tts → OpenAI TTS → macOS say → gtts → fallback to subtitles only

   Audio duration alignment:
   [Real recording] natural duration; if longer than subtitle window, speed up (atempo=1.2 max)
   [Slideshow] must be ≤ SLIDE_DURATION; speed up first (atempo max 1.5x), truncate if still over

4. Compose final MP4 with ffmpeg
   Input source depends on RECORD_MODE
   Overlay subtitles if +sub
   Mix narration audio if +audio
   
   IF click_events is not empty (video format):
     Apply zoompan focus effect at each click:
       zoom in  t-0.3s → t+0.5s: scale 1.0 → 1.3
       zoom out t+0.5s → t+1.2s: scale 1.3 → 1.0
       center on (x, y) coordinates of the click
     IF coordinates unavailable: fallback to center zoom (1.0 → 1.2 → 1.0)

   Slideshow compose command:
     ffmpeg -framerate 1/SLIDE_DURATION \
       -pattern_type glob -i 'screenshots/shot_*.png' \
       -i narration.mp3 \
       -vf "subtitles=tutorial.srt:force_style='FontSize=22,...'" \
       -c:v libx264 -pix_fmt yuv420p -c:a aac -shortest \
       tutorial.mp4

5. Clean up temp files (keep final MP4 and SRT)
```

**Delivery confirmation:**
```
List all generated files:
  ✅ {domain}-tutorial.md       (if applicable)
  ✅ {domain}-tutorial.html     (if applicable)
  ✅ {domain}-tutorial.pdf      (if applicable)
  ✅ {domain}-tutorial.mp4      (if applicable, with subtitles/narration)
  ✅ {domain}-tutorial.srt      (if applicable)
  📁 screenshots/ — {n} screenshots

Show tutorial preview (first 30 lines)

Ask: "Do you want to add any modules, re-explore any step, or export to another format?"
```

---

## Error Handling

| Situation | Action |
|-----------|--------|
| Page load timeout | Wait 5s, retry once; skip and log if still failing |
| Element not found | READ_PAGE to refresh refs, retry once; if still failing, CAPTURE full page and continue |
| Modal / overlay blocking | PRESS_KEY(Esc) or click overlay to dismiss; log as known issue if it persists |
| VISUAL_ANALYZE failure | Skip immediately, no retry; CAPTURE is independent and unaffected |
| Screenshot count too low (< modules × 3) | After exploration, identify missing screenshots and re-capture before Phase 4 |
| No browser tools available | Fallback to fetch + HTML parsing; generate text-only tutorial (no screenshots) |
| Slow dynamic content | Wait 2s before CAPTURE, or wait for a specific element to appear |
| TTS unavailable | Fallback to subtitles only (+sub), inform user |
| ffmpeg unavailable | Skip video composition; keep screenshot sequence; suggest installing ffmpeg |

---

## Key Principles

1. **Confirm scope before acting** — never explore links before the user has confirmed the module list
2. **CAPTURE is always independent** — screenshots must be saved separately from any visual analysis tool
3. **One screenshot before, one after** — show the viewer "what to click" and "what it looks like after"
4. **ANNOTATE is banned in Phase 3** — use READ_PAGE for element refs; never save annotated screenshots to the tutorial
5. **VISUAL_ANALYZE is an enhancement, not a dependency** — skip on failure; its output is never saved as a screenshot
6. **Always pause at login walls** — never guess or bypass authentication
7. **Use imperative voice for steps** — "Click the Login button" not "The user should click"
8. **Match output language to user's choice** — generate all tutorial text (headings, descriptions, captions, narration) in the language selected in Phase 0

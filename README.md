[中文](./README.zh-CN.md) | **English**

# tutorial-generator

An AI skill that automatically generates illustrated tutorials for any website. Input a URL, and the agent will explore pages, take screenshots, record action steps, and produce a polished tutorial — without writing a single line manually.

## Features

- **Tool-agnostic** — uses abstract capability identifiers, works with any AI agent that supports browser automation
- **Login handling** — detects login state automatically; supports email/password, verification code, and OAuth flows
- **Rich screenshots** — before + after each step, plus extras for modals, dropdowns, and scroll areas; minimum 3 per module guaranteed
- **Multiple output formats** — Markdown / HTML (base64 screenshots embedded) / PDF / Video
- **Video support** — screen recording + optional SRT subtitle burn-in + optional TTS narration
- **Safe by default** — delete actions stop at the confirm dialog; payment pages are screenshot-only

## Supported Tools

| Tool | Browser | Login state | Video recording |
|------|---------|-------------|----------------|
| Claude Code (Claude in Chrome) | ✅ | Reuses real Chrome session | Via screencapture |
| Hermes (NousResearch) | ✅ | CDP attach / persistent session | ✅ Native |
| Gemini CLI | ✅ | Reuses real Chrome session | Via screencapture |
| OpenHands | ✅ | ❌ Sandbox | Via recordmydesktop |
| Codex (OpenAI) | ✅ In-app | ❌ Sandbox | Computer Use |
| Any Playwright MCP tool | ✅ | Depends on config | Playwright built-in |

## Installation

**Option 1 — Clone to universal skills directory (recommended, works with all tools)**

```bash
git clone https://github.com/dashenbibi/tutorial-generator ~/.skills/tutorial-generator
```

**Option 2 — Download skill file only**

```bash
mkdir -p ~/.skills/tutorial-generator
curl -o ~/.skills/tutorial-generator/SKILL.md \
  https://raw.githubusercontent.com/dashenbibi/tutorial-generator/main/SKILL.md
```

**Claude Code (auto-loaded):**

```bash
mkdir -p ~/.claude/skills/tutorial-generator
cp ~/.skills/tutorial-generator/SKILL.md ~/.claude/skills/tutorial-generator/SKILL.md
```

**Other tools (Hermes / Gemini CLI / Codex etc.):**

Add to system prompt or at the start of a session:

```
Please read ~/.skills/tutorial-generator/SKILL.md before starting.
```

## Usage

Send a request to your AI agent:

```
Generate a tutorial for https://example.com
```

The agent will follow this workflow:

1. **Phase 0** — Ask about target audience, features to cover, login info, output language, and format
2. **Phase 1** — Scout the site structure, list discovered modules, **wait for you to pick scope**
3. **Phase 2** — Check login state; handle authentication if needed
4. **Phase 3** — Explore each module step-by-step with screenshots
5. **Phase 4** — Compile all steps and screenshots into a tutorial
6. **Phase 5** — Output files, show preview, ask if anything needs to be added

### Output format examples

```
# Markdown only (default)
Generate a tutorial for https://example.com

# Markdown + HTML
Generate a tutorial for https://example.com  format: markdown html

# Video with subtitles and narration
Generate a tutorial for https://example.com  format: video+sub+audio

# Full output
Generate a tutorial for https://example.com  format: markdown html video+sub+audio
```

### Video format dependencies

| Feature | Dependency | Install |
|---------|-----------|---------|
| Video composition | ffmpeg | `brew install ffmpeg` |
| TTS narration (recommended) | edge-tts | `pip install edge-tts` |
| TTS narration (fallback) | gtts | `pip install gtts` |
| PDF output | pandoc | `brew install pandoc` |

All dependencies have automatic fallbacks — missing tools degrade gracefully rather than failing.

## Output structure

```
{domain}/
├── {domain}-tutorial.md
├── {domain}-tutorial.html
├── {domain}-tutorial.pdf
├── {domain}-tutorial.mp4      (with subtitles / narration if requested)
├── {domain}-tutorial.srt
└── screenshots/
    ├── shot_00_home.png
    ├── shot_01_module_overview.png
    ├── shot_02_step1_before.png
    ├── shot_02_step1_after.png
    └── ...
```

## Capability mapping

The skill uses abstract identifiers. Map them to your tool before running:

| Identifier | Description |
|-----------|-------------|
| `NAVIGATE` | Open / navigate to a URL |
| `CAPTURE` | Take a screenshot and save to file |
| `READ_PAGE` | Read page structure (compact / full) |
| `CLICK` | Click an element |
| `TYPE` | Type text into a field |
| `PRESS_KEY` | Press keyboard keys (optional) |
| `RUN_JS` | Execute JavaScript (optional) |
| `VISUAL_ANALYZE` | Screenshot + AI visual analysis (optional enhancement) |
| `SCREEN_RECORD` | Start/stop screen recording (video format only) |

> If your tool combines screenshot and visual analysis (e.g. Hermes `browser_vision`),
> map both `CAPTURE` and `VISUAL_ANALYZE` to it.

## Changelog

| Version | Changes |
|---------|---------|
| v3.2.0 | Pure English SKILL.md; separate bilingual README files |
| v3.1.0 | Bilingual SKILL.md (English + Chinese inline) |
| v3.0.0 | Full English rewrite; multi-language output support |
| v2.0.0 | Abstract capability identifiers replace hard-coded tool names |
| v1.9.0 | Decouple CAPTURE from VISUAL_ANALYZE |
| v1.8.0 | Video add-ons: +sub / +audio combinable; TTS 5-tier fallback |
| v1.7.0 | 5-tier screen recording detection |
| v1.6.0 | Video output format with ffmpeg MP4 |
| v1.5.0 | Phase 1 hard stop; browser_vision failure handling |
| v1.4.0 | Markdown / HTML / PDF output formats |
| v1.3.0 | Action type classification; edit/delete specialized handling |
| v1.2.0 | Mandatory screenshot rules; minimum count guarantee |
| v1.1.0 | Login handling by browser mode |
| v1.0.0 | Initial release |

## Contributing

Issues and PRs welcome:
- Add capability mapping examples for new tools
- Improve login handling logic
- Add new output format support
- Fix execution issues on specific platforms

## License

MIT

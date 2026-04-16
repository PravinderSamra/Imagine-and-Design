# Image Generation Setup

## Overview

This project uses a hybrid pipeline for AI image generation:
- **Banana Claude skill** — acts as Creative Director, engineers high-quality prompts
- **Pollinations.ai FLUX** — free image generation backend, no API key or billing required

---

## Banana Claude Skill

### What it is
An open-source Claude Code skill by [@AgriciDaniel](https://github.com/AgriciDaniel/banana-claude) that acts as a Creative Director for AI image generation. Rather than passing raw user text to an API, it interprets intent, selects a domain mode, and constructs an optimised prompt using Google's official 5-component formula.

### Installation
```bash
git clone https://github.com/AgriciDaniel/claude-banana.git /tmp/claude-banana
cd /tmp/claude-banana
./install.sh --with-mcp YOUR_GOOGLE_AI_KEY
```

### Location
```
~/.claude/skills/banana/
```

### Commands
| Command | What it does |
|---------|-------------|
| `/banana generate <idea>` | Generate image with full prompt engineering |
| `/banana edit <path> <instructions>` | Edit existing image |
| `/banana chat` | Multi-turn creative session |
| `/banana batch <idea> [N]` | Generate N variations |
| `/banana inspire [category]` | Browse prompt ideas |
| `/banana preset` | Manage brand/style presets |
| `/banana cost` | View usage and cost estimates |
| `/banana setup` | Reconfigure MCP/API key |

### 5-Component Prompt Formula
Every generation follows: **Subject → Action → Location/Context → Composition → Style**

Key rules:
- Name real cameras: `Sony A7R IV`, `Canon EOS R5`
- Use prestigious anchors: `National Geographic cover`, `Vanity Fair editorial`
- Never use banned terms: `8K`, `masterpiece`, `ultra-realistic`

---

## MCP Server (nanobanana-mcp)

### What it is
An MCP (Model Context Protocol) server that connects Claude Code to Google's Gemini image generation models. Configured automatically by the banana install script.

### Configuration
Stored in `~/.claude/settings.json`:
```json
{
  "mcpServers": {
    "nanobanana-mcp": {
      "command": "npx",
      "args": ["-y", "@ycse/nanobanana-mcp"],
      "env": {
        "GOOGLE_AI_API_KEY": "<your-google-ai-key>",
        "NANOBANANA_MODEL": "gemini-3.1-flash-image-preview"
      }
    }
  }
}
```

### MCP Tools
| Tool | Purpose |
|------|---------|
| `gemini_generate_image` | Generate new image from prompt |
| `gemini_edit_image` | Edit an existing image |
| `gemini_chat` | Multi-turn iterative generation |
| `set_aspect_ratio` | Set ratio before generating |
| `set_model` | Switch between Gemini models |
| `get_image_history` | Review session history |

### Gemini Models
| Model | Notes |
|-------|-------|
| `gemini-3.1-flash-image-preview` | Default, best quality, requires billing |
| `gemini-2.5-flash-image` | Budget option, also requires billing |

> **Note:** As of December 2025, Google's free tier for image generation has a limit of 0 RPD. Billing must be enabled on your Google AI Studio account to use Gemini image generation.

---

## Free Generation Backend (Pollinations.ai)

### What it is
A completely free image generation service powered by FLUX. No API key, no billing, no account required.

### Script location
```
~/.claude/skills/banana/scripts/hf_generate.py
```

### Usage
```bash
python3 ~/.claude/skills/banana/scripts/hf_generate.py \
  --prompt "your engineered prompt here" \
  --aspect-ratio "1:1"
```

### Supported aspect ratios
`1:1` `16:9` `9:16` `4:3` `3:4` `3:2` `2:3` `4:5` `5:4` `21:9`

### How it works
- Calls `https://image.pollinations.ai/prompt/{encoded_prompt}`
- Uses FLUX model at 1024×1024 (or ratio-matched resolution)
- Saves output to `~/Documents/nanobanana_generated/`
- No rate limits documented; free forever for basic use

### When to use
Use this whenever Gemini quota is exhausted or billing is not enabled. The banana skill handles prompt engineering; this script handles generation.

---

## Full Pipeline

```
User idea
    ↓
/banana generate "idea"
    ↓
Banana Creative Director
  - Reads gemini-models.md + prompt-engineering.md
  - Analyses intent and selects domain mode
  - Constructs 5-component prompt
    ↓
hf_generate.py
  - Sends engineered prompt to Pollinations.ai
  - FLUX model generates image
  - Saves PNG to ~/Documents/nanobanana_generated/
    ↓
Image displayed in Claude Code chat
```

---

## API Keys in settings.json

| Key | Purpose | Where to get |
|-----|---------|-------------|
| `GOOGLE_AI_API_KEY` | Gemini image generation (requires billing) | https://aistudio.google.com/apikey |
| `HF_TOKEN` | HuggingFace (reserved, not currently used) | https://huggingface.co/settings/tokens |

> Keys are stored in `~/.claude/settings.json` under the `env` and `mcpServers` sections.

---

## Generated Images

All generated images are saved to:
```
~/Documents/nanobanana_generated/
```
Filename format: `banana_flux_YYYYMMDD_HHMMSS_ffffff.png`

Images committed to this repo are in the `/images` folder.

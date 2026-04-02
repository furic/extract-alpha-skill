# Extract Alpha &middot; [![Claude Code Plugin](https://img.shields.io/badge/Claude_Code-Plugin-blue)](https://docs.anthropic.com/en/docs/claude-code) [![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE) [![Python](https://img.shields.io/badge/Python-3.x-yellow?logo=python&logoColor=white)](https://www.python.org/) [![Pillow](https://img.shields.io/badge/Pillow-required-orange)](https://python-pillow.org/)

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that removes backgrounds from AI-generated images using **two-pass difference matting**.

## The Problem

AI image generators (Gemini, DALL-E, Midjourney, etc.) often don't support transparent backgrounds. You get a PNG with a solid white or black background baked in.

## The Solution

Generate the **same image twice** — once on white, once on black. The difference between the two reveals the exact transparency of every pixel:

- **Fully opaque** pixels look identical on both backgrounds
- **Fully transparent** pixels show pure white vs pure black
- **Semi-transparent** pixels show proportional differences

Math: `alpha = 1 - colorDistance(whiteImg, blackImg) / maxDistance`

This produces pixel-perfect alpha extraction — no AI guessing, no edge artifacts.

## Quick Install

```
/install furic/extract-alpha
```

That's it — the plugin installs both skills automatically.

### Manual Install

If you prefer to install manually:

```bash
git clone https://github.com/furic/extract-alpha.git
mkdir -p ~/.claude/skills/extract-alpha
cp extract-alpha/skills/extract/SKILL.md ~/.claude/skills/extract-alpha/SKILL.md
```

### Requirements

- Python 3 with [Pillow](https://python-pillow.org/) installed (`pip install Pillow`)

## Skills

This plugin provides two skills:

### `/extract-alpha:extract` — Extract transparency

Processes a white/black background image pair and outputs a transparent PNG.

```
/extract-alpha:extract path/to/image_white.png
```

Accepts any of these file path forms:
- `{name}_white.png` — automatically finds the matching `{name}_black.png`
- `{name}_black.png` — automatically finds the matching `{name}_white.png`
- `{name}.png` — looks for both `{name}_white.png` and `{name}_black.png`

Output: `{name}.png` with full transparency, saved alongside the input files.

### `/extract-alpha:prompt` — Generate image prompts

Generates a pair of AI image prompts (white + black background) ready for alpha extraction.

```
/extract-alpha:prompt a dragon logo for my fantasy game
```

This outputs two ready-to-use prompts:
1. **Prompt 1** — A detailed image generation prompt with a white background
2. **Prompt 2** — A follow-up prompt to swap to black background

## Workflow

### Option A: Use the prompt skill (recommended)

1. Run `/extract-alpha:prompt <description>` to generate the prompt pair
2. Send Prompt 1 to your AI image generator (Gemini, ChatGPT, etc.)
3. Save the result as `{name}_white.png`
4. In the **same conversation**, send Prompt 2
5. Save the result as `{name}_black.png`
6. Run `/extract-alpha:extract {name}_white.png`

### Option B: Write prompts manually

Add this line at the end of your image prompt:

```
- On a pure solid white #FFFFFF background
```

Then send a follow-up in the same conversation:

```
Change the white background to a solid pure black #000000. Keep everything else exactly unchanged.
```

Save the results as `{name}_white.png` and `{name}_black.png`, then run `/extract-alpha:extract`.

> **Why the same conversation?** The AI retains context about the exact image it just created, so only the background changes. Generating two separate images from scratch produces inconsistent results.

## How It Works

1. Load both white-background and black-background images
2. For each pixel, calculate the color distance between the two versions
3. Derive alpha: `alpha = 1 - distance / maxDistance`
4. Recover original RGB: `rgb = blackImg_rgb / alpha`
5. Output RGBA image with extracted transparency

The script handles minor size differences between images (common with AI generators) by cropping to the smaller dimensions.

## License

MIT

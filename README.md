# Extract Alpha &middot; [![Claude Code Plugin](https://img.shields.io/badge/Claude_Code-Plugin-blue)](https://docs.anthropic.com/en/docs/claude-code) [![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE) [![Python](https://img.shields.io/badge/Python-3.x-yellow?logo=python&logoColor=white)](https://www.python.org/) [![Pillow](https://img.shields.io/badge/Pillow-required-orange)](https://python-pillow.org/)

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that removes backgrounds from AI-generated images using **two-pass difference matting**.

## The Problem

AI image generators (Gemini, DALL-E, Midjourney, etc.) often don't support transparent backgrounds. You get a PNG with a solid white or black background baked in.

## The Solution

Generate the **same image twice** — once on white, once on black. The difference between the two reveals the exact transparency of every pixel:

- **Fully opaque** pixels look identical on both backgrounds
- **Fully transparent** pixels show pure white vs pure black
- **Semi-transparent** pixels show proportional differences

Math: `alpha = 1 - colorDistance(whiteImg, blackImg) / maxDistance`

This produces pixel-perfect alpha extraction — no AI guessing, no edge artifacts.

## Quick Install (Claude Code Plugin)

```bash
/install furic/extract-alpha
```

That's it — the plugin installs the skill automatically.

### Manual Install

If you prefer to install manually:

```bash
# Clone this repo
git clone https://github.com/furic/extract-alpha.git

# Copy to your Claude Code skills directory
mkdir -p ~/.claude/skills/extract-alpha
cp extract-alpha-skill/skills/extract-alpha/SKILL.md ~/.claude/skills/extract-alpha/SKILL.md
```

### Requirements

- Python 3 with [Pillow](https://python-pillow.org/) installed (`pip install Pillow`)

## Usage

In Claude Code, run:

```
/extract-alpha path/to/image_white.png
```

The skill accepts any of these file path forms:
- `{name}_white.png` — automatically finds the matching `{name}_black.png`
- `{name}_black.png` — automatically finds the matching `{name}_white.png`
- `{name}.png` — looks for both `{name}_white.png` and `{name}_black.png`

Output: `{name}.png` with full transparency, saved alongside the input files.

## Generating Image Pairs

Generate the white version first, then use a follow-up prompt to get the black version.

### Step 1: Generate the white background image

Write your image prompt as normal, and add this line at the end:

```
- On a pure solid white #FFFFFF background
```

**Full example:**
```
Game UI text graphic for a space adventure game. The words "STAR QUEST" rendered as a stylized 3D game title with:
- Bold blocky font with metallic chrome finish
- Blue-to-purple gradient across letters
- Thin white outline for contrast
- Bright specular highlights on top edges
- Soft drop shadow beneath
- Small star sparkle decorations around text
- Energetic, adventurous mood
- Cartoon style, NOT realistic
- Clean and legible at small sizes
- Centered composition, landscape orientation
- Resolution: 1024 x 512 pixels
- On a pure solid white #FFFFFF background
```

Save the result as `{name}_white.png`.

### Step 2: Generate the black background image

In the **same conversation**, send a follow-up prompt:

```
Change the white background to a solid pure black #000000. Keep everything else exactly unchanged.
```

Save the result as `{name}_black.png`.

> **Why this approach?** Using a follow-up prompt in the same conversation produces more consistent results than generating two separate images from scratch. The AI retains context about the exact image it just created, so only the background changes.

## How It Works

1. Load both white-background and black-background images
2. For each pixel, calculate the color distance between the two versions
3. Derive alpha: `alpha = 1 - distance / maxDistance`
4. Recover original RGB: `rgb = blackImg_rgb / alpha`
5. Output RGBA image with extracted transparency

The script handles minor size differences between images (common with AI generators) by cropping to the smaller dimensions.

## License

MIT
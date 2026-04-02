# extract-alpha

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

## Installation

Copy the skill into your Claude Code skills directory:

```bash
# Clone this repo
git clone https://github.com/furic/extract-alpha-skill.git

# Copy to your Claude Code skills directory
cp -r extract-alpha-skill/SKILL.md ~/.claude/skills/extract-alpha/SKILL.md
```

Or add as a git submodule in your project:

```bash
mkdir -p .claude/skills
git submodule add https://github.com/furic/extract-alpha-skill.git .claude/skills/extract-alpha
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

When prompting an AI image generator, create the same image twice with only the background line changed:

**White background version:**
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
- Solid white background, flat uniform color, no patterns
- Centered composition, landscape orientation
- Resolution: 1024 x 512 pixels
```

**Black background version:** Same prompt but change the last background line to:
```
- Solid black background, flat uniform color, no patterns
```

## How It Works

1. Load both white-background and black-background images
2. For each pixel, calculate the color distance between the two versions
3. Derive alpha: `alpha = 1 - distance / maxDistance`
4. Recover original RGB: `rgb = blackImg_rgb / alpha`
5. Output RGBA image with extracted transparency

The script handles minor size differences between images (common with AI generators) by cropping to the smaller dimensions.

## License

MIT
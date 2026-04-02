---
name: extract-alpha
description: Remove background from AI-generated (e.g. Gemini) images using two-pass difference matting. Use when the user has white/black background image pairs that need transparent backgrounds.
argument-hint: "<file_path>"
---

# Two-Pass Difference Matting

Extract transparent alpha from AI-generated images that lack transparency support.

## Argument parsing

The user provides a file path that can be any of these forms:
- `{name}_white.png` — look for `{name}_black.png` in the same directory
- `{name}_black.png` — look for `{name}_white.png` in the same directory
- `{name}.png` — look for `{name}_white.png` and `{name}_black.png` in the same directory

Derive:
- **`name`**: strip `_white` or `_black` suffix if present, and strip the extension
- **`src_dir`**: the directory containing the input file
- **`out_dir`**: same as `src_dir` (output alongside inputs)

If the file path has no directory component, search the current working directory.

## How it works

For any pixel, the difference between its appearance on white vs black backgrounds reveals its transparency. Fully opaque pixels look identical on both; fully transparent pixels show pure white vs pure black.

Math: `alpha = 1 - colorDistance(whiteImg, blackImg) / maxDistance`, then `rgb = blackImg_rgb / alpha`.

## Steps

### Step 1: Resolve files

1. Parse the provided file path to extract `name` and `src_dir`
2. Verify both `{name}_white.png` and `{name}_black.png` exist in `src_dir`
3. If either is missing, tell the user which file is missing and stop

### Step 2: Find Python with Pillow

Check in order:
1. `~/.local/pipx/venvs/rembg/bin/python3` (has Pillow from rembg)
2. `python3` with Pillow installed
3. If neither works, tell the user to install Pillow: `pip install Pillow`

### Step 3: Run the extraction script

Write a temporary Python script and execute it:

```python
from PIL import Image
import math, sys, os

def extract_alpha(name, src_dir, out_dir):
    white = Image.open(f'{src_dir}/{name}_white.png').convert('RGB')
    black = Image.open(f'{src_dir}/{name}_black.png').convert('RGB')
    w = min(white.width, black.width)
    h = min(white.height, black.height)
    white = white.crop((0, 0, w, h))
    black = black.crop((0, 0, w, h))
    pw, pb = white.load(), black.load()
    out = Image.new('RGBA', (w, h))
    po = out.load()
    bg_dist = math.sqrt(3.0 * 255.0 * 255.0)
    for y in range(h):
        for x in range(w):
            rW, gW, bW = pw[x, y]
            rB, gB, bB = pb[x, y]
            dist = math.sqrt((rW - rB) ** 2 + (gW - gB) ** 2 + (bW - bB) ** 2)
            alpha = max(0.0, min(1.0, 1.0 - dist / bg_dist))
            if alpha > 0.01:
                rO = min(255, round(rB / alpha))
                gO = min(255, round(gB / alpha))
                bO = min(255, round(bB / alpha))
            else:
                rO = gO = bO = 0
            po[x, y] = (rO, gO, bO, round(alpha * 255))
    os.makedirs(out_dir, exist_ok=True)
    out.save(f'{out_dir}/{name}.png')
    print(f'Saved {out_dir}/{name}.png ({w}x{h})')

extract_alpha(sys.argv[1], sys.argv[2], sys.argv[3])
```

Run: `<python> /tmp/extract_alpha.py <name> <src_dir> <out_dir>`

### Step 4: Report result

- Show the output file path and dimensions
- Use the Read tool to display the output image to the user

## Generating image pairs

Generate the white version first, then use a follow-up prompt to get the black version:

**First prompt** — include at the end of your image description:
```
- On a pure solid white #FFFFFF background
```

**Second prompt** — tell the AI to swap the background:
```
Change the white background to a solid pure black #000000. Keep everything else exactly unchanged.
```

> **Tip:** See the project README for a full example prompt template.

## Notes

- Works with JPG or PNG inputs (output is always PNG with alpha)
- AI generators may produce 1px size differences between runs — handled automatically
- Requires Pillow (`pip install Pillow`) — check rembg venv first
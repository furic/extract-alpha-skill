---
name: prompt
description: Generate a pair of AI image prompts (white + black background) ready for alpha extraction. Use when the user wants to create an image with transparency using the extract-alpha workflow.
argument-hint: "<image description>"
---

# Generate Alpha-Ready Image Prompts

Generate two prompts for AI image generators that produce a white/black background pair, ready for alpha extraction with `/extract-alpha:extract`.

## Argument parsing

`$ARGUMENTS` is the user's description of the image they want. It can be brief (e.g. "a dragon logo") or detailed.

## Steps

### Step 1: Build the full image prompt

Take the user's description and enhance it into a detailed AI image generation prompt. Add specifics for style, detail, and composition if not already provided. Always append these lines at the end:

```
- Centered composition, landscape orientation
- Resolution: 1024 x 512 pixels
- On a pure solid white #FFFFFF background
```

### Step 2: Output both prompts

Present two clearly labeled prompts the user can copy:

**Prompt 1 (White background):**
The full prompt from Step 1.

**Prompt 2 (Black background):**
```
Change the white background to a solid pure black #000000. Keep everything else exactly unchanged.
```

### Step 3: Remind the workflow

After showing the prompts, add a brief reminder:

> **Next steps:**
> 1. Send Prompt 1 to your AI image generator (Gemini, ChatGPT, etc.)
> 2. Save the result as `{name}_white.png`
> 3. In the **same conversation**, send Prompt 2
> 4. Save the result as `{name}_black.png`
> 5. Run `/extract-alpha:extract {name}_white.png` to extract the transparent PNG

## Notes

- Always use the bulleted list format for prompts — it's more reliable with AI generators
- Keep "Cartoon style, NOT realistic" if the user wants game/UI graphics, but omit for photorealistic requests
- The second prompt must always be sent in the same conversation for consistency

---
name: mov-to-gif
description: Convert MOV screen recordings or short video clips to optimized GIFs using ffmpeg with palette generation for better quality
---

Convert `.mov` files to optimized `.gif` files using `ffmpeg`.

Use this skill when the user asks to:
- convert a `.mov` file to `.gif`
- make a GIF from a screen recording
- optimize a short video clip for sharing in issues, pull requests, documentation, or chat

## Requirements

Use `ffmpeg`. Before converting, verify that:
- the input file exists
- the input file is a video file, usually `.mov`
- `ffmpeg` is available, or clearly state that it is missing

Do not overwrite an existing output file unless the user explicitly asks. If the default output path already exists, choose a safe variant such as `<name>-converted.gif` or ask before overwriting.

## Default Command

Use this command shape by default:

```bash
ffmpeg -i <input.mov> -vf "fps=10,scale=840:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" -c:v gif <output.gif>
```

This balances size and quality for UI screen recordings:
- `fps=10` keeps the GIF reasonably small
- `scale=840:-1` sets width to 840px and preserves aspect ratio
- `flags=lanczos` keeps scaling quality high
- `palettegen` and `paletteuse` improve GIF color quality

## Workflow

1. Identify the input file
   - Use the file the user names.
   - If no file is named, look for likely `.mov` files in the current directory.
   - If multiple likely files exist, ask which one to convert.

2. Choose the output file
   - Default to the same basename with `.gif` extension.
   - Example: `node_toolbar_issue.mov` → `node_toolbar_issue.gif`.
   - Avoid overwriting existing files without explicit permission.

3. Run conversion
   - Use the default command unless the user asks for different dimensions, frame rate, quality, or output path.
   - Quote filenames that contain spaces.

4. Verify output
   - Confirm the GIF file exists.
   - If practical, report the file size.

## Tuning Options

Adjust the command only when useful or requested:
- Smaller GIF: reduce `fps` to `6` or `8`, or reduce width below `840`
- Smoother GIF: increase `fps` to `12` or `15`
- Larger display size: increase width above `840`
- Preserve original width: remove `scale=840:-1:flags=lanczos`

Prefer the smallest GIF that still clearly shows the behavior.

## Output

Use `output-template.md` for response formatting.

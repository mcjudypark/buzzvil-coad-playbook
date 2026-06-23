# Slide extraction spec — follow EXACTLY

You convert ONE Figma slide into a self-contained HTML fragment for a 1920×1080 slide deck. You are given a slide number N (and its zero-padded form NN) and a Figma nodeId.

## Steps

1. **Load the Figma tool**: ToolSearch query `select:mcp__f27e0f2a-89b3-48d4-8011-4c830f4a8f86__get_design_context`

2. **Get the design**: call `mcp__f27e0f2a-89b3-48d4-8011-4c830f4a8f86__get_design_context` with:
   - fileKey: `AIMxKf1iUStlbLejaOxNgB`
   - nodeId: (the given nodeId)
   - clientFrameworks: `unknown`
   - clientLanguages: `html,css,javascript`
   Keep the screenshot — you will use it to verify visual fidelity. The slide is 1920×1080.

3. **Download every asset locally** (Figma URLs expire in 7 days):
   - `mkdir -p /Users/judy.park/cc/coad_v2/assets/sNN`
   - For each `const imgXXX = "https://www.figma.com/api/mcp/asset/..."` in the returned code: `curl -sL "URL" -o /Users/judy.park/cc/coad_v2/assets/sNN/<short-name>.png` (use a clear short filename; `.png` for everything unless it is clearly an SVG vector, then `.svg`).
   - Verify each file is non-empty with `ls -la /Users/judy.park/cc/coad_v2/assets/sNN`.

4. **Convert to a plain HTML fragment** (NO React, NO Tailwind, NO class names, NO external CSS):
   - Single root: `<div class="slide-canvas" data-slide="N" style="position:relative;width:1920px;height:1080px;overflow:hidden;font-family:'Pretendard',sans-serif; <ROOT-BACKGROUND-IF-ANY> ">`
   - Recreate EVERY visible element absolutely positioned using INLINE styles only, so fragments compose without collisions.
   - Resolve every Tailwind `calc(NN%+Xpx)` to an absolute pixel value on the 1920×1080 canvas: 33.33% = 640, 50% = 960, 66.67% = 1280, 83.33% = 1600, 100% = 1920 (horizontal); 50% = 540 etc (vertical). Preserve rotations with `transform:rotate(...)`.
   - Text: real `<p>`/`<div>` with the EXACT Korean text. `font-family:'Pretendard'`. Map font weight: ExtraBold→800, Bold→700, SemiBold→600, Medium→500, Black→900, Regular→400, Light→300. Use exact `font-size`, `color`, `line-height`, `letter-spacing` from the code. Keep `white-space:nowrap` where present. If the source uses Noto Sans KR or any other font, still use Pretendard (the deck is Pretendard-only).
   - Rounded rectangles / shapes → `<div>` with the right `border-radius`, `background`, `border`, `box-shadow`.
   - Images → `<img src="assets/sNN/<name>.png" style="position:absolute;left:..px;top:..px;width:..px;height:..px;object-fit:..">`. Paths are RELATIVE to `coad_v2/index.html` (which is one level above `assets/`), so always `assets/sNN/<name>.png`.
   - Root background: if the root has a gradient like `bg-gradient-to-b from-[#fafafa] to-[#82dc28]`, apply `background:linear-gradient(180deg,#fafafa,#82dc28);` matching the stops/percentages as closely as the visible canvas shows.
   - **Decorative background blobs / swooshes** (large rotated ellipses usually named `Ellipse 6xx`, faint circles, vector swooshes like `Vector 71`): these export as tiny/blurry or BROKEN PNGs (often < 2 KB) and show up as broken-image icons — DO NOT use the downloaded image for them. Recreate them with CSS instead: a large positioned `<div>` with `border-radius:50%`, a soft green fill (e.g. `background:radial-gradient(...)` or a translucent `#b6e89a`/`#82dc28` tone) matching the screenshot, plus `filter:blur(40px)` or similar if soft-edged. Keep them BEHIND the content. Only use real downloaded image assets for photos, logos, icons, phone mockups, and detailed illustrations.
   - **Validate downloaded assets**: after `curl`, check sizes with `ls -la`. Any asset under ~2 KB is likely a broken/empty export — if it is a real photo/logo it may have failed (retry once); if it is decorative, recreate with CSS per the rule above instead of referencing the broken file.
   - Faithfully match the screenshot in layout, color, size, and position.

5. **Write the file**: write ONLY the `<div class="slide-canvas">…</div>` fragment to `/Users/judy.park/cc/coad_v2/slides/sNN.html`. No `<html>`/`<head>`/`<body>`, no markdown code fences, nothing else.

## Return (final message — keep it short)
- The slide's main heading text (one line, for the table of contents)
- Number of assets downloaded
- Any fidelity caveat (one line)
Do NOT paste the HTML back.

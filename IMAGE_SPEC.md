# Render a Figma slide to a high-res image and replace its fragment with an <img>

You are given a list of slides, each as (slideNumber N, zero-padded NN, nodeId). For EACH slide do:

1. Load the tool once: ToolSearch query `select:mcp__f27e0f2a-89b3-48d4-8011-4c830f4a8f86__get_screenshot`
2. `mkdir -p /Users/judy.park/cc/coad_v2/pages`
3. Call `mcp__f27e0f2a-89b3-48d4-8011-4c830f4a8f86__get_screenshot` with:
   - fileKey: `AIMxKf1iUStlbLejaOxNgB`
   - nodeId: (the given nodeId)
   - maxDimension: 3840
   The response is JSON containing an `image_url` field (a short-lived figma asset URL).
4. Download it: `curl -sL "<image_url>" -o /Users/judy.park/cc/coad_v2/pages/sNN.png`
   Then verify: `file /Users/judy.park/cc/coad_v2/pages/sNN.png` must report a PNG image and `ls -la` should show > 50 KB. If it is HTML/empty/tiny, re-call get_screenshot and curl again (URLs expire fast).
5. Overwrite `/Users/judy.park/cc/coad_v2/slides/sNN.html` with EXACTLY this fragment (substitute the real N and NN, no markdown fences, nothing else):

<div class="slide-canvas" data-slide="N" style="position:relative;width:1920px;height:1080px;overflow:hidden;background:#fff;">
  <img src="pages/sNN.png" style="position:absolute;inset:0;width:100%;height:100%;object-fit:fill;display:block;" alt="slide N">
</div>

Process every slide in your assigned list. Return a short summary: which slides succeeded (with file sizes) and any that failed.

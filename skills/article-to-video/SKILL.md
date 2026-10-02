---
name: article-to-video
description: Turn a blog post or article (plus its cover image) into a short, silent, autoplay-friendly animated video (MP4) built around 3 key messages, asking the user for approval at each step.
---

# Article → animated video

Make a short animated social video from an article, entirely with Claude: HTML/CSS/JS scenes driven by one deterministic timeline, captured frame by frame in headless Chromium (Playwright), and encoded with ffmpeg. No outside video or design tools.

**Get the user's approval at every gate (🛑) before moving on.** Ask with your question tool (e.g. AskUserQuestion) or in chat, then wait. Don't skip ahead.

## Requirements
- Python 3 with `playwright` (plus its Chromium: `playwright install chromium`) and `Pillow`.
- `ffmpeg` on the PATH.
- If any are missing, install them yourself where you can (e.g. in a sandbox). If you can't, tell the user what to install before Step 2.
- Work in a new folder of its own (e.g. `article-video/<slug>/`) and run every command from inside it.

## Core principle: 3 messages, no context required
The viewer is scrolling LinkedIn or X, has never read the post, and is watching muted. A cut with many scenes, product names, and stats is too much for someone without context. So:
- **Exactly 3 key messages**, shaped as **Problem → Answer → Takeaway**.
  - *Problem* = the hook. A tension anyone in the audience instantly recognizes.
  - *Answer* = the article's thesis, usually its title. Show it with one simple visual (e.g., a 3-step staircase).
  - *Takeaway* = the single most quotable, actionable line in the post.
- **Cut anything that needs context:** product names, vendor comparisons, detailed stats, and sub-arguments. They belong in the post, not the video.
- **~25–30 seconds total.** One idea per scene, with each scene held long enough to read (~2.5 words/sec, plus about 1s once the kicker lands).
- Then a CTA with the cover image, the URL, and the byline.
- Only offer a longer "full cut" if the user asks for one (e.g., to embed in the post itself).

## Step 0: Gather inputs
- Fetch the article with WebFetch and ask for the full text: headings, key lines, quotes. If curl is blocked by a proxy, WebFetch often still works.
- The cover image: download it if you can. If the host blocks it (Substack CDNs and X often are in sandboxed environments), ask the user to attach the cover in chat or give its file path. In Claude Code, an image pasted into chat isn't saved as a file, so ask for the path.
- Ask for the byline and, optionally, a short brand tag (e.g., the newsletter or publication name) for the top-left corner.
- Format: default to **1:1 square, 1080×1080** (works in LinkedIn and X feeds). Offer 9:16 (1080×1920) for Reels/Shorts, or 16:9 (1920×1080) to embed in the post.

## Step 1: Propose the 3 messages 🛑
Propose the Problem / Answer / Takeaway lines, each with the exact on-screen text and a one-sentence description of the visual. Give 1–2 alternates for the Takeaway, and list what you're deliberately leaving out. Let the user pick.

## Step 2: Storyboard 🛑
1. Sample the palette from the cover with PIL (background, accent/highlighter, dark UI, brand colour, plus red and green for bad and good). Define them as CSS variables.
2. If the cover has hand lettering, extract it as a transparent PNG and use it in the Answer scene with a clip-path wipe. To extract it: threshold dark pixels to alpha, keep the accent color, and mask out any intruding subject.
3. Build `index.html` at the chosen size (e.g., 1080×1080) with:
   - each scene as an absolutely positioned `.scene` div;
   - `window.W` and `window.H` set to the chosen size (render.py reads them, so every render matches the format);
   - a `SCENES=[[id,start,end],...]` table, a `DURATION`, and `window.KEYS` (one storyboard time per scene);
   - `window.renderAt(t)`, which sets every element's style as a *pure function of t*. Helpers: `P(t,a,b)` for progress; easeOut, easeInOut and back easings; `pop()` for fade/slide/scale-in; and `hl()` to animate a highlighter swipe behind key words via background-size.
   - Scenes crossfade over ~0.4s. Use no CSS transitions, no requestAnimationFrame, and no unseeded randomness.
4. Fonts: use a font installed locally (`fc-list`, or on a Mac without it, `ls /Library/Fonts ~/Library/Fonts /System/Library/Fonts`) so rendering never waits on the network; Poppins Bold works well. If the environment has no web access, local fonts are the only option. Use inline SVG for icons.
5. Save `render.py` (below) next to `index.html`. Render one still per scene and assemble a labeled contact sheet (scene name + time range) at `storyboard.png` in the outputs folder (in Cowork and claude.ai, `/mnt/user-data/outputs/` if it exists; otherwise, the working folder).
6. Look at every still yourself before sending. Check for text wrapping inside inputs or cards, elements overlapping headers, text near the edges, and dead space.
7. When sending, flag for the user: any stat to verify, anything invented as an illustrative example (quotes, form inputs), and the runtime.

## Step 3: Render 🛑 (after storyboard approval)
`python3 render.py video 30` → MP4 (H.264, yuv420p, crf 18, faststart). A 30s clip takes about a minute to render.

## Step 4: QA and refine 🛑
- Make a 1 fps contact sheet with `ffmpeg -i out.mp4 -vf "fps=1,scale=270:-2,tile=10x3" -frames:v 1 contact.png`. Read it and zoom into suspicious frames with `render.py stills <t>`.
- Watch for elements animating through headings, counters overshooting, text shown too briefly, and letters clipped at the edges.
- Copy the file to `<slug>.mp4` in the outputs folder, share it with the user (with SendUserFile where that tool exists; otherwise, give the file path), and ask for notes. Apply them, re-render, and re-check the changed scenes.

## Posting notes
- **LinkedIn:** upload the MP4 natively; don't share a YouTube or Substack link. Native video autoplays muted in the feed, while links show up as a preview card and don't autoplay.
- The video has no audio, so every message must work as on-screen text, and the first frame should already show readable text.

## render.py
```python
import sys, os, subprocess, pathlib
from playwright.sync_api import sync_playwright
HERE = pathlib.Path(__file__).parent.resolve()
PAGE = os.environ.get("PAGE","index.html"); OUT = os.environ.get("OUT","out.mp4")
URL = (HERE / PAGE).as_uri()
mode = sys.argv[1]
with sync_playwright() as p:
    b = p.chromium.launch(); pg = b.new_page()
    pg.goto(URL); W, H = pg.evaluate("[window.W||1080, window.H||1080]")
    pg.set_viewport_size({"width":W,"height":H})
    pg.wait_for_load_state("networkidle"); pg.evaluate("document.fonts.ready")
    if mode == "stills":
        times = [float(x) for x in sys.argv[2:]] or pg.evaluate("window.KEYS")
        (HERE/"frames").mkdir(exist_ok=True)
        for i,t in enumerate(times):
            pg.evaluate(f"renderAt({t})"); pg.screenshot(path=str(HERE/"frames"/f"still_{i:02d}.png"))
    else:
        fps = int(sys.argv[2]) if len(sys.argv)>2 else 30; n = int(round(pg.evaluate("window.DURATION")*fps))
        ff = subprocess.Popen(["ffmpeg","-y","-loglevel","error","-f","image2pipe","-framerate",str(fps),"-i","-",
            "-c:v","libx264","-pix_fmt","yuv420p","-crf","18","-movflags","+faststart",str(HERE/OUT)], stdin=subprocess.PIPE)
        for f in range(n):
            pg.evaluate(f"renderAt({f/fps})"); ff.stdin.write(pg.screenshot(type="jpeg", quality=95))
        ff.stdin.close(); ff.wait()
    b.close()
```

## Style defaults
Use these unless the user has their own brand style. Take colours from the cover palette in Step 2.
- A light background (the cover's lightest tone, or cream `#F6F1E7`), near-black ink (`#1A1A1A`), and a highlighter swipe in the cover's accent colour on the key word of each kicker.
- If the user gave a brand tag, show it small, uppercase and letterspaced in the top-left corner, and hide it on the CTA.
- Headlines 48–66px bold, kickers 60–76px, body text ≥26px (it must read on a phone). Keep 64px of margin from every edge.
- Hook pattern that works: show the familiar "everything looks fine" state (e.g., green ✓ status pills), then reveal the problem (e.g., the result gets stamped red).
- End on the cover image (slow zoom-out), "READ THE FULL POST", the URL with highlighter, and the byline.

## Lessons from past runs
- Too many messages loses cold viewers. Default to 3 and cut the jargon.
- Long typed text inside a form input wraps; keep strings under ~35 characters at 30px.
- Animated chips overlapped a panel heading; start moving elements below header blocks.
- Stats taken from a summarized fetch may be off; have the user verify any numbers.

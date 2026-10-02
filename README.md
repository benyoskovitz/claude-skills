# claude-skills

Skills for Claude that I use and think others will find useful. Each skill is a folder under [`skills/`](skills/). Read it, copy it, or install it.

| Skill | What it does |
|---|---|
| [article-to-video](skills/article-to-video/SKILL.md) | Turns a blog post and its cover image into a 25 to 30 second silent animated video (MP4) for LinkedIn or X, built around 3 key messages. Claude asks for your approval at each step. |

## Install in Claude Code

### Option A: plugin marketplace (easiest to keep updated)

Inside Claude Code, run:

```
/plugin marketplace add benyoskovitz/claude-skills
/plugin install claude-skills@benyoskovitz
```

Every skill in this repo comes along. They show up with a `claude-skills:` prefix, for example `/claude-skills:article-to-video`. To get new versions later, run `/plugin marketplace update benyoskovitz`.

### Option B: copy one skill

```bash
git clone https://github.com/benyoskovitz/claude-skills.git
```

```bash
mkdir -p ~/.claude/skills && cp -R claude-skills/skills/article-to-video ~/.claude/skills/
```

Run both in the same terminal. Then start a new Claude Code session. The skill is now `/article-to-video`. To install it for one project only, copy the folder into that project's `.claude/skills/` instead.

## Install in claude.ai or Cowork

1. Get the files. Either clone the repo as above, or use **Code → Download ZIP** on GitHub and unzip it. A clone makes a folder called `claude-skills`. The ZIP makes one called `claude-skills-main`.
2. Zip just the `article-to-video` folder, which is inside that folder's `skills` folder. On a Mac, right-click it in Finder and choose **Compress**. Or use a terminal.

   If you downloaded the ZIP to your Downloads folder:

   ```bash
   cd ~/Downloads/claude-skills-main/skills && zip -r article-to-video.zip article-to-video
   ```

   If you cloned, in the same terminal you cloned from:

   ```bash
   cd claude-skills/skills && zip -r article-to-video.zip article-to-video
   ```

3. In claude.ai, open **Settings → Capabilities**, make sure code execution is on, then upload the zip under **Skills**.
4. Start a new chat and ask Claude to turn an article into a video. Skills you upload are private to your account.

## Requirements for article-to-video

The skill renders video on the machine Claude runs on, so it needs:

- Python 3 with `playwright` and `Pillow`, plus Playwright's browser: `python3 -m pip install playwright pillow && python3 -m playwright install chromium` (if pip refuses with "externally-managed-environment", run it inside a virtual environment: `python3 -m venv .venv && source .venv/bin/activate` first)
- `ffmpeg` (on a Mac: `brew install ffmpeg`)

In claude.ai and Cowork, Claude can usually install these in its own sandbox. In Claude Code, install them yourself first.

## License

MIT. See [LICENSE](LICENSE).

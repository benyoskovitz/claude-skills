# claude-skills

Skills for Claude that I use and think others will find useful. Each skill is a folder under [`skills/`](skills/). Read it, copy it, or install it.

| Skill | What it does |
|---|---|
| [article-to-video](skills/article-to-video/SKILL.md) | Turns a blog post and its cover image into a 25 to 30 second silent animated video (MP4) for LinkedIn or X, built around 3 key messages. Claude asks for your approval at each step. |
| [pitch-deck-reviewer](skills/pitch-deck-reviewer/SKILL.md) | Gives clear, candid feedback on a startup or corporate venture pitch deck, one slide, or the overall storyline, using the patterns and checklists I use when I review decks. It can also rewrite weak slides, propose a new slide order, compare two versions of a deck, and list the questions investors are likely to ask. A [ChatGPT plugin version](https://benyoskovitz.github.io/pitch-deck-reviewer/) is awaiting OpenAI's approval. |

## Install in Claude Code

### Option A: plugin marketplace (easiest to keep updated)

In a terminal, run the two commands below. They need the `claude` command. If your terminal says `claude` isn't found or isn't recognized, install it first:

1. On Mac or Linux, run `curl -fsSL https://claude.ai/install.sh | bash`. On Windows, follow [Anthropic's setup guide](https://code.claude.com/docs/en/setup).
2. Open a **new** terminal window and run `claude --version`. It should print a version number.
3. If it still isn't found, follow [Fix your PATH](https://code.claude.com/docs/en/troubleshoot-install#command-not-found-claude-after-installation), then try again in a new window.

```bash
claude plugin marketplace add benyoskovitz/claude-skills
```

```bash
claude plugin install claude-skills@benyoskovitz
```

If you use Claude Code in a terminal, you can type the same thing inside a session instead: `/plugin marketplace add benyoskovitz/claude-skills`, then `/plugin install claude-skills@benyoskovitz`. If `/plugin` opens a plugins screen instead of running the command (this can happen in the Claude desktop app), use the terminal commands above.

Every skill in this repo comes along. Start a new session and they show up with a `claude-skills:` prefix, for example `/claude-skills:article-to-video` or `/claude-skills:pitch-deck-reviewer`. To get new versions later, run `claude plugin marketplace update benyoskovitz`, then `claude plugin update claude-skills@benyoskovitz`, and start a new session.

### Option B: copy one skill

```bash
git clone https://github.com/benyoskovitz/claude-skills.git
```

```bash
mkdir -p ~/.claude/skills && cp -R claude-skills/skills/article-to-video ~/.claude/skills/
```

Run both in the same terminal. Then start a new Claude Code session. The skill is now `/article-to-video`. For a different skill, put its folder name in place of `article-to-video`, for example `pitch-deck-reviewer`. To install it for one project only, copy the folder into that project's `.claude/skills/` instead.

## Install in claude.ai or Cowork

1. Get the files. Either clone the repo as above, or use **Code → Download ZIP** on GitHub and unzip it. A clone makes a folder called `claude-skills`. The ZIP makes one called `claude-skills-main`.
2. Zip just the folder of the skill you want, which is inside that folder's `skills` folder. The steps below use `article-to-video`. For another skill, put its folder name in place of `article-to-video`. On a Mac, right-click it in Finder and choose **Compress**. Or use a terminal.

   If you downloaded the ZIP to your Downloads folder:

   ```bash
   cd ~/Downloads/claude-skills-main/skills && zip -r article-to-video.zip article-to-video
   ```

   If you cloned, in the same terminal you cloned from:

   ```bash
   cd claude-skills/skills && zip -r article-to-video.zip article-to-video
   ```

3. In claude.ai, open **Settings → Capabilities**, make sure code execution is on, then upload the zip under **Skills**.
4. Start a new chat and ask Claude to use the skill, for example to turn an article into a video or to review your pitch deck. Skills you upload are private to your account.

## Requirements for article-to-video

The skill renders video on the machine Claude runs on, so it needs:

- Python 3 with `playwright` and `Pillow`, plus Playwright's browser: `python3 -m pip install playwright pillow && python3 -m playwright install chromium` (if pip refuses with "externally-managed-environment", run it inside a virtual environment: `python3 -m venv .venv && source .venv/bin/activate` first)
- `ffmpeg` (on a Mac: `brew install ffmpeg`)

In claude.ai and Cowork, Claude can usually install these in its own sandbox. In Claude Code, install them yourself first.

## Using pitch-deck-reviewer

It needs no installs. Give Claude your deck as a PDF so it can see the slides as well as read them. If you only have PowerPoint, Keynote, or Google Slides, export a PDF first, or share screenshots of the slides. If you can't do either, paste the slide text instead. Claude then reviews the text only and says so.

After a review, ask for any of the extra modes in your own words, for example "rewrite my three weakest slides" or "what will investors ask me?"

Reviews are AI generated. They are not a personal review by me and do not guarantee funding.

## License

MIT. See [LICENSE](LICENSE).

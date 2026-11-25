# RyanBot

An autonomous AI agent with its own Windows computer. You give it everyday tasks on Discord, in plain
language, and it does the whole job itself, without you copying, pasting or clicking:

> "Research the top 5 budget mechanical keyboards, compare them in a spreadsheet and tell me which to buy."
> "Go through my inbox and tell me what's missing from the invoices."
> "Turn this bug report into GitHub tickets."
> "Rename all the photos in Downloads by date."

It works with any OpenAI-compatible model API. The default is Grok.

> Not affiliated with xAI. "Grok" refers to the default model backend.

## What it can use

For each step it picks the most direct way in:

| Module | What it does | Why |
|---|---|---|
| **Desktop** | screenshots, mouse, keyboard, opens apps | works with *anything* on the PC |
| **Browser** | drives Chrome with its own logged-in profile; reads pages as numbered buttons and fields | far faster and more accurate than clicking pixels on websites |
| **GitHub** | create, list, edit and comment on issues through the API | no copy-paste into GitHub |
| **Files & docs** | read/write/edit files, make Word docs (tailored resumes, cover letters), run commands | |
| **Job log** | every application goes into `jobs.xlsx` | |
| **`me/` folder** | your details, master resume and standing instructions | it never has to ask you the same thing twice, and never makes facts up |

Other things: autonomous by default, `!stop` from Discord, a mouse-to-corner kill switch, a step cap per task, and it posts what it's doing as it goes.

## Give it its own desktop (recommended)

RyanBot works best on a computer it has to itself, so you and it aren't fighting over the mouse.

**Option A: Windows VM on your PC (best).** Turn on Hyper-V (Windows Pro), create a Windows 11 VM with
Quick Create, and install RyanBot inside it. The bot gets its own screen, browser and accounts, and you
keep using your PC. If it messes something up, roll back to a checkpoint.

**Option B: old laptop or mini PC** left on at home, running only RyanBot.

**Option C: your own PC.** Works, but it takes over the mouse while it's doing a task.

Whichever you pick: set sleep to *Never*, stay signed in (it can't click past a lock screen), and give it
its own accounts where you can.

## How desktop mode works

1. The bot takes a screenshot, shrinks it to 1280px wide and draws a labelled grid on it.
2. The model reads the screenshot and calls a tool like `click(x=640, y=360)`, using the grid to aim.
3. RyanBot scales the coordinates back up to your real screen, does the action, and takes a new screenshot.
4. Only the two newest screenshots are kept in context to keep API costs down.

## Quick start (Windows)

Desktop control needs **normal Windows Python**. It won't work from WSL, because WSL can't reach the Windows screen.

```powershell
git clone https://github.com/rhinoswag/ryanbot.git
cd ryanbot
py -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
copy .env.example .env    # then fill it in
python -m ryanbot
```

First run creates `me/` in the workspace: fill in `about-me.md` and `resume.md`. Then open the browser once
(ask the bot to "open seek.com.au") and log into your sites in that window. The bot keeps those logins.

Install the browser driver once: `playwright install chromium`. It's only a fallback if Chrome or Edge isn't found.

For GitHub: create a fine-grained token with Issues read & write and put it in `GITHUB_TOKEN`.

To start it on login: press Win+R, type `shell:startup`, and put a shortcut to `deploy\start-ryanbot.bat` there.
It restarts itself if it crashes.

For a PC left on at home: set Power & sleep to *Never* sleep, and keep the user signed in.
A locked screen can't be clicked.

### Discord setup
1. https://discord.com/developers/applications → **New Application** → **Bot**.
2. Turn on **Message Content Intent**. Copy the token into `DISCORD_TOKEN`.
3. OAuth2 → URL Generator → scope `bot`; permissions: Send Messages, Read Message History, Add Reactions. Open the URL to invite it.
4. Turn on Developer Mode in Discord, right-click your name → **Copy User ID** → `OWNER_ID`.

### Model setup

| Provider | `LLM_BASE_URL` | `LLM_MODEL` |
|---|---|---|
| xAI (Grok) | `https://api.x.ai/v1` | a current vision-capable Grok model from xAI's docs |
| OpenAI | `https://api.openai.com/v1` | a GPT model with vision |
| OpenRouter | `https://openrouter.ai/api/v1` | any vision + tools model |
| Ollama (local) | `http://localhost:11434/v1` | a local vision model with tool calling |

## Commands

| Discord message | What it does |
|---|---|
| anything | starts a task |
| `!stop` | cancels the task in progress |
| `!reset` | clears the bot's memory of this channel |

## Adding a tool

```python
from ryanbot.tools import tool

@tool({"type": "object", "properties": {"city": {"type": "string"}}, "required": ["city"]})
async def weather(ctx, city: str) -> str:
    """Get the weather for a city."""
    ...
```

Return a `ToolResult(text, image_b64)` if the model should see an image.

## Roadmap

- [ ] UI Automation tree (pywinauto) so it can click buttons by name, not just pixels
- [ ] Email tools (Gmail API) so inbox tasks don't need the browser
- [ ] Scheduled tasks ("every morning at 8…")
- [ ] Long-term memory notes
- [ ] Multi-monitor support

## License

MIT. See [LICENSE](LICENSE). Read [SECURITY.md](SECURITY.md) before running it.

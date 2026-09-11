# This is V.1, uses open source rules, and is a work in progress. Lmk if you have any feedback

This is a plain, step-by-step walkthrough for getting Rosetta running
locally, written for someone doing it for the first time. If you just want
the short version, see the **Quickstart** in `README.md`; this document is
the long version, including exactly what gets downloaded and why.

Rosetta is a local web app: it runs a small Python server on your own
computer and you use it in your browser at `http://127.0.0.1:8765`. Nothing
is hosted anywhere — no account, no cloud service, no data leaves your
machine unless you explicitly turn on the optional AI-assist or push-to-SIEM
features and give it a key to use.

## 1. Install Python

Rosetta needs **Python 3.9 or later** (3.10+ recommended). Check what you
already have:

```bash
python3 --version
```

If that prints `Python 3.9.x` or higher, skip to step 2. Otherwise:

- **Windows**: download the installer from
  [python.org/downloads](https://www.python.org/downloads/) and run it.
  On the first install screen, check **"Add python.exe to PATH"** before
  clicking Install — this is the single most common thing people miss.
- **macOS**: download from the same link, or `brew install python3` if you
  use Homebrew.
- **Linux**: install via your distro's package manager, e.g.
  `sudo apt install python3 python3-pip` (Debian/Ubuntu).

This step downloads the Python installer itself from python.org — the one
unavoidable download if you don't already have Python.

## 2. Get the Rosetta code onto your computer

If you're reading this from a folder that already has `app.py` in it, you
already have the code — skip to step 3.

Otherwise, from the project's GitHub page:

- Click the green **Code** button → **Download ZIP**, then unzip it
  somewhere on your computer, **or**
- If you have `git` installed: `git clone <the repo's URL>`

Either way, this downloads the project's own files (this is separate from
the rule-data downloads in step 5 below).

## 3. Install the one required dependency: PyYAML

The app itself (the web server and UI) needs nothing beyond what ships
with Python. Only the rule-loading step needs one extra package, PyYAML,
which reads the `.yml`/`.yaml` rule files:

```bash
pip install pyyaml
```

**This is a download** — `pip` fetches the PyYAML package from
[pypi.org](https://pypi.org) (Python's package index) over an HTTPS GET
request and installs it into your Python environment. It's a small package
(a few hundred KB) and this only needs to happen once per computer.

If `pip` isn't recognized, try `python3 -m pip install pyyaml` instead.

On some Linux setups you may see an "externally managed environment"
error — in that case use `pip install pyyaml --break-system-packages`, or
create a virtual environment first (`python3 -m venv venv`, then activate
it and re-run the plain `pip install pyyaml`).

## 4. Run the app

From inside the project folder:

```bash
python3 app.py
```

You should see it print something like `Serving on http://127.0.0.1:8765`.
Open that address in your browser. Leave the terminal window open — closing
it stops the server. To stop it yourself, press `Ctrl+C` in that terminal.

On first run, Rosetta creates `data/rules.db` and automatically loads the
small bundled sample set in `seed_data/` (12 real rules across all five
sources), so there's something to look at immediately — no extra step
needed for that part.

Want a different port, or an existing database file?

```bash
python3 app.py --port 9000
python3 app.py --db path/to/rules.db
```

## 5. (Optional, recommended) Download the full rule set

The 12 seed rules are just a taste. To pull the **complete, current** rule
corpus — thousands of real detection rules from all five projects — either:

- Click **Update rules** in the app's top-right corner, then **Sync all
  sources**, or
- Run it from the command line:

```bash
python3 -m ingest.live --source all
```

**What this downloads, specifically:** this step needs `git` installed and
on your `PATH` (check with `git --version`; if missing, get it from
[git-scm.com/downloads](https://git-scm.com/downloads)). It clones (a
shallow, sparse checkout — just the rules folders, not full repo history)
these five public GitHub repositories:

| Source | Repository |
|---|---|
| Sigma | `github.com/SigmaHQ/sigma` |
| Elastic | `github.com/elastic/detection-rules` |
| Splunk | `github.com/splunk/security_content` |
| Microsoft Sentinel | `github.com/Azure/Azure-Sentinel` |
| Falco | `github.com/falcosecurity/rules` |

Each is downloaded (via `git`, over HTTPS) into a local cache folder outside
the project directory — `%LOCALAPPDATA%\rosetta-rules\repo_cache` on
Windows, or `~/.cache/rosetta-rules/repo_cache` on macOS/Linux — and parsed
into `data/rules.db`. This is deliberately kept out of the project folder
itself so that if the project folder is synced by OneDrive/Dropbox/Google
Drive, the sync client doesn't try to watch or lock the downloaded repo
files.

You can re-run this any time to pick up upstream rule updates — it updates
existing rules in place rather than duplicating them. Useful variations:

```bash
python3 -m ingest.live --source sigma --limit 200   # quick smoke test, one source, capped
python3 -m ingest.live --source elastic --no-sparse # if the sparse checkout fails
```

If this step fails, it's almost always one of: `git` not installed, or no
outbound internet access to github.com (a network/firewall restriction).
The app still works fine with just the seed data if this step doesn't work
for you.

## 6. (Optional) Connect an AI API for a second-pass review

Entirely optional, and off by default. If you want a language model to
double-check or refine a translation, click **Connect an AI API** on any
rule, or **Settings** in the top bar, and paste in an API key from
Anthropic, OpenAI, or any OpenAI-compatible provider (Groq, Mistral,
Together, OpenRouter, a local Ollama server, etc.) — no extra packages
needed. Nothing is sent anywhere until you click **Improve with AI** or
**Request changes** on a specific rule. Your key is saved locally in
`data/ai_config.json` and never leaves your machine except in the direct
call you trigger.

## 7. (Optional) Push rules to a live SIEM

Also optional and off by default. A rule's drawer has a **Push to your
environment** panel for sending a converted rule to a real Kibana, Splunk,
or Microsoft Sentinel instance you connect. See the **Pushing rules to your
live environment** section of `README.md` for exactly what each platform
needs. Every pushed rule always lands **disabled** — there's no setting
that changes this — so nothing goes live without you reviewing and
enabling it yourself in that environment.

## Summary: every network download involved

| What | Command | Source |
|---|---|---|
| Python itself (if you don't have it) | installer from python.org | python.org |
| The Rosetta project files | ZIP download or `git clone` | GitHub (this repo) |
| PyYAML (required) | `pip install pyyaml` | PyPI |
| `git` itself (only needed for step 5) | installer from git-scm.com | git-scm.com |
| Full rule corpus (optional) | `python3 -m ingest.live --source all` | 5 GitHub repos listed above |
| AI-assist calls (optional, opt-in per click) | happens only when you click "Improve with AI" / "Request changes" | whichever AI provider you configured |
| Push-to-SIEM calls (optional, opt-in per click) | happens only when you click "Push" | whichever Kibana/Splunk/Sentinel host you configured |

Everything above steps 1–4 is required to run the app at all; steps 5–7 are
optional and each only downloads/connects when you explicitly ask it to.

# Setup

About 30 minutes, most of it filling in your profile.

> **Keep your filled-in copy private.** GitHub's "Use this template" and forks of a
> public repository are public by default, and this workspace fills with your
> name, address, answers, resume, saved job postings and application history.
> Create your copy as a **private** repository (or clone without pushing). The
> pre-commit hook installed in step 2 refuses to commit those files unless it can
> confirm `origin` is private. `profile/credentials.md` and `profile/llm.json` are
> gitignored — keep them that way.

## 1. Prerequisites

| Need | Why |
|---|---|
| **Node.js ≥ 22** | every tool and test (built-in `WebSocket`/`fetch`) |
| **Python ≥ 3.9** | the Python scripts — standard library only. Docs write `python`; on macOS/Linux type `python3` |
| **Google Chrome** | the browser the agent drives, with the autofill extension loaded |
| **An AI agent app** | the operating agent — any app that reads files and runs shell commands here: Claude Code (CLI, or the Claude desktop app's Code tab), OpenAI Codex (CLI, IDE, or the ChatGPT / Codex desktop app), Cursor, GitHub Copilot in VS Code, Gemini CLI, Windsurf. It reads `AGENTS.md` and the skills in `.agents/skills/` |
| *(optional)* read-only email access in that app | e.g. the Gmail connector in Claude, or a Gmail/IMAP MCP server — lets the agent read verification codes and links; without it you paste them in |
| *(optional)* [GitHub CLI](https://cli.github.com) (`gh`), signed in | lets the privacy hook confirm your remote is private before it allows a personal-data commit |
| *(optional)* a local model server — [Ollama](https://ollama.com), LM Studio, llama.cpp or vLLM | only if you choose a local model in step 2. Any model works (gpt-oss, Qwen, Llama, Mistral…); setup asks for the server URL and model name, and says whether your hardware runs the default gpt-oss:20b well |

Windows is the primary platform; macOS and Linux are supported, less tested.

## 2. Run setup

Not a git repository yet (you downloaded a copy rather than cloning)? Run
`git init` first — setup installs the privacy hook only inside one.

```bash
node tools/setup.mjs
```

It checks Node, Python and Chrome, probes your RAM and GPU, asks who answers what
the extension can't (default: your AI agent) and writes that to
`profile/llm.json`, wires the workspace into
your AI app (it guesses the app you run it from; `--app codex` etc. names it),
asks which legal consents the agent may accept for you, and turns on the
pre-commit privacy hook
(`git config core.hooksPath tools/hooks`). `--check` reports without changing
anything. `--yes` accepts every default — your AI agent as the model, nothing
downloaded — but never answers the consent questions; `--consent backgroundCheck=y,creditCheck=y,arbitration=y,drugScreen=n`
does (an agent running setup for you asks you these in chat and passes them).

**Legal consents.** Many applications end in a background-check authorization,
a credit-check authorization, a binding arbitration agreement or a drug-screen
consent. Setup asks, one by one, whether the agent may accept each kind for you;
answer **n** and every one of that kind is handed to you instead. What each
covers, the suggested answers and how to change your mind:
[`profile/README.md`](profile/README.md#legal-consents-profileconsents).

**Your AI agent is the model, by default.** The extension fills everything it
can answer deterministically — your saved answers, your profile, what it
verified on a site before — and hands every other field (essays, cover letters,
odd dropdowns) to the AI agent you are chatting with, which writes them from
your dossier and the job description and checks them before saving. No API key,
nothing to install. Setup never changes that on its own. If you want the
extension to call a model itself, you can pick one: a local model (any OpenAI-compatible
server — Ollama, LM Studio, llama.cpp, vLLM — running any model you like), or an API key for OpenAI (or any
OpenAI-compatible service — Gemini, OpenRouter, Groq) or Anthropic, pasted into
the extension's Settings and never into the repo. That adds speed on long forms,
not accuracy, and the agent still verifies everything it writes.

## 3. Fill in your profile

Replace every `<PLACEHOLDER>` — [`profile/README.md`](profile/README.md) lists
each file:

1. [`profile/applicant-dossier.md`](profile/applicant-dossier.md) and
   [`profile/autofill-profile.json`](profile/autofill-profile.json) — the same
   facts in two shapes; keep them in sync. Review every default that isn't a
   placeholder (standard answers, `additionalContext`).
2. Your resume as `profile/resume.pdf` (and `profile/resume.docx`, the fallback
   when a site refuses the PDF). No resume, or one in columns and tables that
   applicant systems misread? `node tools/build-resume.mjs --init`, fill in
   `profile/resume.json`, then `node tools/build-resume.mjs` — a one-column,
   parser-safe PDF and .docx, printed by Chrome, no extra installs.
3. Copy [`profile/search-profile-example.txt`](profile/search-profile-example.txt)
   to `profile/search-profile-<lane>.txt` — one file per kind of job you're
   after (e.g. `search-profile-it-support.txt`) — and fill it in: target roles,
   exclusions, experience, geography.
4. `profile/credentials.md` was created by setup from `credentials.example.md`
   (gitignored); the agent records careers-site logins there as it creates them.

## 4. Start the browser and load your profile

```bash
node tools/chrome.mjs              # debug Chrome on port 9222 with the extension loaded
node tools/verify-agent-bridge.mjs # confirms browser → extension → agent bridge
node tools/load-profile.mjs        # copies your profile, resume and provider choice into the extension
```

On a brand-new Chrome profile the extension may load **disabled**: open
`chrome://extensions`, switch **Developer mode** on, then switch the extension on.
Re-run `load-profile.mjs` whenever you edit the profile. In later sessions
`node tools/startup.mjs` does all three (plus the local model, if you chose it)
in one go. Details: [`agent/reference/browser-setup.md`](agent/reference/browser-setup.md).

## 5. Verify

```bash
npm test                   # every browserless suite; each should report 0 failed
npm run test:fixtures      # optional: the browser fixtures, in a throwaway Chrome
```

No `npm install` is needed — the workspace has no npm dependencies.

## 6. First run — just say what you want

Open your AI app in this folder — the Code tab of the Claude desktop app, Codex
in the ChatGPT desktop app, Cursor, VS Code, or a terminal agent started here.
Setup wired it; to add or switch apps later run `node tools/agent-setup.mjs <app>`
(`--list` shows every app it knows). The first time, approve the two browser
servers (`playwright`, `playwright-apply`) if it asks — their first use downloads
Playwright's MCP server through `npx` — and, in Codex, trust the folder. An app
not on the list works too if it can read files and run commands: tell it to read
`AGENTS.md`. A chat-only window (plain Claude or ChatGPT chat) cannot — it has no
access to this folder.

Then talk to it. Some first sentences, one per mode:

| Mode | Say |
|---|---|
| 1 · Job search | "Find me entry-level IT support roles near Springfield." |
| 2 · Board recheck | "Recheck the job boards that are due." |
| 3 · Apply | "Apply to the next listing." · "I got a rejection from Globex — log it." |
| 4 · Auto-apply | "Keep applying on your own until you run out of usage." |
| 5 · Queue & clear | "Queue up applications while I'm away" — later: "Clear the queue." |
| 6 · Extension | "Why didn't the extension fill the degree field on that form?" |
| 7 · Training | "Do a recon session on a job site we haven't seen yet." |

The agent submits an application only after a field-by-field audit, never enters
your SSN, date of birth, government ID or bank details, and never solves a
CAPTCHA — those come back to you (or, in unattended modes, into a queue you clear
later). The full rules are in [`agent/policy.md`](agent/policy.md); the routing
from sentence to mode is [`AGENTS.md`](AGENTS.md).

## Terms

- **ATS** — applicant tracking system: the vendor behind a careers site's
  application form (Workday, Greenhouse, iCIMS…). `training/ats-notes/` has one
  note per vendor.
- **CDP** — Chrome DevTools Protocol: how the tools drive the debug Chrome
  (port 9222 by default).
- **MCP** — Model Context Protocol: how agent apps reach extra tools such as the
  browser. Optional here — every browser action also has a plain `node tools/…` command.
- **Lane** — one kind of job you're targeting; each has its own search profile.

## What starts empty

Your tracker (`applications/`), the hunt CSVs (`company-hunt/`), saved job
descriptions (only `EXAMPLE.md`), playbooks and employer notes (`training/`),
and your AI app's own memory for this project. The extension, drivers, hunt scripts,
vendor notes and tests ship ready.

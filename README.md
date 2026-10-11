# iPersist

iPersist is an AI-agent workspace that finds jobs and applies to them for you. You talk to
the AI coding agent you already use (Claude Code, OpenAI Codex / ChatGPT, Cursor,
GitHub Copilot, Gemini CLI, Windsurf…) in plain sentences. It searches
employers' own career sites, keeps a tracker of what fits, and fills in real
application forms in Chrome from a profile you write once.

## How it works

- **Your AI agent runs the show.** It reads the rules in [`agent/`](agent/README.md),
  picks the right mode for what you asked, and does the work. No API key is
  needed: the agent you're chatting with writes anything that takes judgment
  (essays, cover letters, unusual questions).
- **A Chrome extension does the typing.** It fills every field it can answer for
  certain from your profile and saved answers, and gets better with each form it
  sees. It knows more than 40 applicant systems, including Workday, Greenhouse,
  iCIMS, Lever, SmartRecruiters and Taleo.
- **Your profile is the only source of facts.** The agent never invents
  anything about you. It audits every field before it clicks Submit, and it
  logs an application only once the submission is confirmed.
- **You keep the sensitive parts.** The agent never enters your SSN, date of
  birth, government ID or bank details, and never solves a CAPTCHA. Those come
  back to you. Legal agreements (background checks, arbitration and so on) are
  accepted only if you said yes to that kind during setup.

The application logic is built for **US job applications**: EEO, veteran,
disability and work-authorization questions, US addresses and phone numbers.

## Modes

Say what you want and the agent picks the mode.

| Mode | What it does | Try saying |
|---|---|---|
| **1 · Job search** | Works through employers' own career sites in your area, reads each posting's real requirements, and saves the ones that fit your search profile | "Find me entry-level IT support roles near Springfield." |
| **2 · Board recheck** | Re-scans every tracked employer's job board on a 10-day cycle to catch new postings and drop closed ones | "Recheck the job boards that are due." |
| **3 · Apply** | Fills one application with you available to answer questions, audits it, submits, and logs it | "Apply to the next listing." |
| **4 · Auto-apply** | Keeps applying unattended until your usage runs out, skipping anything that needs you and finding more listings when the list runs dry | "Keep applying on your own until you run out of usage." |
| **5 · Queue & clear** | Fills applications in a batch while you're away and queues every CAPTCHA, login code and identity check. Later you clear just those and the agent submits | "Queue up applications while I'm away." Later: "Clear the queue." |
| **6 · Extension** | Fixes or improves the autofill extension itself | "Why didn't the extension fill the degree field on that form?" |
| **7 · Training** | Opens unfamiliar application systems to learn how their forms work. Submits nothing | "Do a recon session on a job site we haven't seen yet." |

Modes 3, 4 and 5 submit real applications, and they improve the extension on
every form they fill.

## Before you use it

- **It applies in your name.** Every application it submits is yours, and you're
  responsible for what it says. Check the tracker in `applications/` regularly.
- **Some sites forbid automation.** A careers site's terms may ban automated
  applications, and breaking them can get your account there suspended.
- **Recruiters may spot AI-written answers.** Read what it writes for essays and
  cover letters, at least until you trust it.

## Setup

About 30 minutes, most of it filling in your profile. The full walkthrough is
[`SETUP.md`](SETUP.md).

1. **Install** Node.js 22 or newer, Python 3.9 or newer, Google Chrome, and an
   AI agent app that can read files and run commands in a folder.
2. **Make a private copy.** This workspace fills up with your personal data, so
   never keep it in a public repository.
3. **Run setup** from the folder:

   ```bash
   node tools/setup.mjs
   ```

   It checks your machine, connects the workspace to your AI app, asks which
   legal agreements the agent may accept for you, and turns on a git hook that
   keeps personal data off public repositories.
4. **Fill in your profile.** Replace every `<PLACEHOLDER>` in
   `profile/applicant-dossier.md` and `profile/autofill-profile.json`, add your
   resume (or build one with `node tools/build-resume.mjs`), and write a search
   profile from `profile/search-profile-example.txt`.
5. **Start Chrome with the extension:**

   ```bash
   node tools/startup.mjs
   ```

6. **Open your AI app in this folder** and say what you want, for example "Find
   me jobs."

No `npm install` is needed: the workspace has no npm dependencies. Optionally,
the extension can call a model of its own: a local one (Ollama, LM Studio,
llama.cpp or vLLM running any model) or an OpenAI-compatible or Anthropic API
key. That only adds speed on long forms. It is never the default.

## Folders

| | |
|---|---|
| [`agent/`](agent/README.md) | The directives: hard rules, the seven mode docs, reference |
| [`profile/`](profile/README.md) | **You**: dossier, machine-readable profile, resume, search profiles |
| [`company-hunt/`](company-hunt/README.md) | The listing pipeline: qualified-listing CSVs, HTML views, scripts |
| [`applications/`](applications/STATE.md) | Tracker, saved job descriptions, learning log, current state |
| [`autofill-extension/`](autofill-extension/README.md) | The Chrome extension: source, docs, tests |
| [`training/`](training/recon-plan.md) | Per-vendor form notes, harvests, per-employer playbooks and one-off employer facts |
| [`tools/`](tools/) | Setup, Chrome launcher, browser drivers, resume builder, tests |
| `archive/` | Retired files, nothing live (created on first use) |

Agents start at [`AGENTS.md`](AGENTS.md) (Claude Code through `CLAUDE.md`), which
routes each request to one mode. `node tools/agent-setup.mjs` connects your app to it.

## Platforms

Windows is the primary platform. macOS and Linux are supported but less tested.

## License

MIT. See [`LICENSE`](LICENSE).

# iPersist

**New candidate?** Read [`SETUP.md`](SETUP.md) first.

Seven modes. **`agent/` is the only source of truth for behaviour** — this file
routes, it does not rule. It is the one instruction file for every agent app
(Claude Code reaches it through `CLAUDE.md`; the others read it directly or are
pointed at it by `tools/agent-setup.mjs`).

**Every session: read [`agent/policy.md`](agent/policy.md) (its top table says how
much, per mode), then the one mode doc below.** Not all seven. Read
[`agent/delegation.md`](agent/delegation.md) before launching a subagent; a
subagent does not read it — its prompt carries its limits.

## Routing

| The user says something like | Mode | Read |
|---|---|---|
| find roles · scan / re-scout `<company>` · what's in the hunt · hunt status · is this job a fit · qualify or cut a listing | **1 · Job search** | [`agent/modes/1-job-search.md`](agent/modes/1-job-search.md) |
| recheck the job boards · what's due for a recheck · check the companies we haven't looked at in 10 days · work the recheck queue | **2 · Board recheck** | [`agent/modes/2-board-recheck.md`](agent/modes/2-board-recheck.md) |
| apply to X · fill this application · next listing · I got a rejection / interview · log this application · update the tracker | **3 · Apply** | [`agent/modes/3-apply.md`](agent/modes/3-apply.md) |
| auto-apply · just keep applying · apply to everything on the list · run on your own until you're out of usage · don't ask me anything | **4 · Auto-apply** | [`agent/modes/4-auto-apply.md`](agent/modes/4-auto-apply.md) |
| queue up applications while I'm away · batch/unattended fill · work the queue · clear the queue · knock out the captchas/confirmations | **5 · Queue & clear** | [`agent/modes/5-queue-and-clear.md`](agent/modes/5-queue-and-clear.md) |
| fix the matcher · add an adapter · why did `scan` miss X · run the tests · the extension isn't filling Y · model provider settings | **6 · Extension** | [`agent/modes/6-extension.md`](agent/modes/6-extension.md) |
| recon · learn a new ATS · harvest a form · work the recon queue | **7 · Training** | [`agent/modes/7-extension-training.md`](agent/modes/7-extension-training.md) |

**Disambiguation**

- **"Find something and apply to it"** → mode 1, *then* mode 3, as separate
  passes. They have different stopping rules; do not interleave. **Mode 4 is the
  one exception** — it interleaves on purpose, and only to refill an empty list.
- **1 vs 2** — mode 1 *discovers* employers not in the registry yet. Mode 2
  re-scans employers already in it, on the 10-day clock
  (`company-hunt/scripts/recheck_due.py` is the queue). Neither is policy §9,
  the single-company freshness check that rides along with applying.
- **3 vs 4 vs 5** — all submit real applications. 3: one, attended, asks. 4:
  unattended until usage runs out, never asks, refills the list itself. 5: a
  bounded unattended batch, then an attended pass clearing what it queued.
  "Keep going on your own" is 4; "while I'm out, I'll clear the captchas later" is 5.
- **3/4/5 vs 7** — 7 never submits (ATS coverage, companies never applied to).
- **Modes 3, 4 and 5 always contain mode 6** — filling a form without improving
  the extension is half a job (policy §4), unattended or not.
- Modes 3, 4, 5, and 7 also read [`applications/learning-log.md`](applications/learning-log.md)
  *In force now* — corrections newer than anything in `agent/`.

**Not a mode**

| Task | Where to go |
|---|---|
| "I passed <cert>" · new cert, address, phone, LinkedIn | Edit **both** `profile/applicant-dossier.md` and `profile/autofill-profile.json` (a cert also goes on the search profile's `Certifications held:` line), then `node tools/load-profile.mjs`. Check `company-hunt/README.md` §4 for listings the change unblocks |
| "Stop accepting background checks for me" · change a legal consent | `node tools/setup.mjs --consent <key>=y\|n` after asking the user — keys and meaning in [`profile/README.md`](profile/README.md#legal-consents-profileconsents), rule in policy §3 |
| Build or update the resume | `profile/resume.json` → `node tools/build-resume.mjs` (header of the script); content from the dossier only (policy §1) |
| Use a different AI app · browser MCP tools missing · mode skills not listed | `node tools/agent-setup.mjs <app>` (`--list` names the apps, `--check` says what is wired) — header of the script |
| Start the browser · CDP won't connect · which port | [`agent/reference/browser-setup.md`](agent/reference/browser-setup.md) |
| How do I read `<vendor>`'s job board | [`agent/reference/ats-playbook.md`](agent/reference/ats-playbook.md) |
| Which profile key matches this form label | [`agent/reference/field-mapping.md`](agent/reference/field-mapping.md) |
| Create careers accounts · passwordless / OTP login · locked out | [`agent/reference/account-creation.md`](agent/reference/account-creation.md); secrets only in `profile/credentials.md` |
| Are the *saved listings* still up · re-verify the whole listings file | Run **mode 2** across every company (`recheck_due.py --all`) — re-sweeping a board re-verifies its saved listings as a side effect |
| Where did we leave off | [`applications/STATE.md`](applications/STATE.md) |
| What do we know about `<company>` · what did we answer there last time | `node tools/employer.mjs show "<Company>"` — [`training/employers/README.md`](training/employers/README.md) |
| Applying at a company we've already applied to · what's replayable here | `node tools/playbook.mjs plan --appId <appId>` — [`training/playbooks/README.md`](training/playbooks/README.md), rule in policy §11 |
| Writing a STATE entry · a subagent prompt | [`agent/templates/`](agent/templates); a new ats-note starts from [`training/ats-notes/_template.md`](training/ats-notes/_template.md) |
| Is the tracker right? · duplicate rows · a date looks wrong | [`applications/data-quality-review.md`](applications/data-quality-review.md) |
| A plain question about a file, no application impact | Just answer — skip all of the above |

## Non-negotiable

The hard rules are [`agent/policy.md`](agent/policy.md): §1 never invent a fact,
§2 audit then Submit, §3 identity gates and agreements, §5 displayed ≠ saved,
§7 tracker = verified submits only, §11 replay only what is byte-identical to a
verified submit, §12 what to do when a form won't cooperate.

## Delegate by default

One application / company / vendor = one subagent; the orchestrating session
keeps the plan, the tracker writes, and every user-facing stop —
[`agent/delegation.md`](agent/delegation.md).

## Working here

- **Before adding a fact to any doc** read [`agent/reference/writing-rules.md`](agent/reference/writing-rules.md)
  (one fact, one home · the fact, not the story · removing means archiving).
  Folder map: [`README.md`](README.md) § Folders.
- **Say what you changed. Commit as you go.** Retired files go to `archive/`, not
  `rm` (rule 7) — it keeps removed context greppable.
- **Env:** Windows primary, macOS/Linux supported; Node ≥ 22 and Python ≥ 3.9 on
  PATH. `package.json` holds scripts only (`npm test`, `npm run setup`) — no npm
  dependencies. Commands in docs are cross-platform unless marked PowerShell.
- **Never hardcode a drive letter or absolute repo path** — the scripts and configs
  derive their own root; keep it that way. The per-app wiring files that must hold
  one are written by `tools/agent-setup.mjs` and gitignored.
- **Skills live in `.agents/skills/`** — the folder every agent app reads.
  `.claude/skills/` is a generated copy for Claude Code; never edit it.

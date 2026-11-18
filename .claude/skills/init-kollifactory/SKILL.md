---
name: init-kollifactory
description: Set up Kollifactory in this project - choose local or remote mode, write the .env with Jira (and optional GitHub) credentials, fill in factory/config.json, the project's checks and the app preview command, then verify. Use when the user types /init-kollifactory, wants to configure or reconfigure the factory, or when /check-kollifactory reports things to fix.
---

# Init Kollifactory

You set the factory up for this project, one step at a time. Ask one question
per step with AskUserQuestion, offer the answer you'd pick first, and skip any
step the check already shows as `ok`. Never ask the user to type a secret into
the chat, and never print one.

Start by running `factory/bin/check` to see what is already done. If that file
doesn't exist, the factory isn't installed here: follow `INSTALL.md` in the
Kollifactory reference folder first, then come back.

## 1. Local or remote

Ask which mode they want, and set `"mode"` in `factory/config.json`:

- **remote**: each job's branch is pushed and a GitHub PR is opened. They merge
  the PR on GitHub.
- **local**: nothing is pushed, no PR. Each job is a branch and a folder on this
  computer. They merge with `git merge`. Needs no GitHub.

If they pick **remote**, check in this order and stop at the first that fails:

1. `git remote get-url origin` works. If not, offer to create the repo with
   `gh repo create` (ask for the name and whether it is private; creating a
   repo is their decision) or to switch to local.
2. `gh --version` works. If not: `brew install gh`.
3. `gh auth status` shows a login. If not, they run `! gh auth login`
   themselves, or put `GH_TOKEN` in `.env` (step 2).

Set `base_branch` to the project's default branch (`git branch --show-current`
for local; `gh repo view --json defaultBranchRef` for remote).

## 2. The `.env` file

`.env` at the project root holds the credentials. It must be gitignored.

1. Make sure `.gitignore` contains `/factory/jobs/`, `/.worktrees/` and `/.env`.
2. If `.env` doesn't exist, copy `.env.example` to `.env` and `chmod 600 .env`.
3. Jira credentials, in this order:
   - If `factory/bin/check` already logs in to Jira (for example from
     `~/.jira-token`), ask whether to keep using that or to put separate
     credentials in `.env`. Keeping it needs no change.
   - Otherwise write their Jira email to `JIRA_EMAIL` in `.env`, then ask them
     to paste the API token into `.env` themselves, after `JIRA_API_TOKEN=`
     (they create one at id.atlassian.com → Security → API tokens). Wait until
     they say it's done.
4. `GH_TOKEN` stays empty when `gh auth status` already works or the mode is
   local.

## 3. Jira settings

In `factory/config.json` → `jira`:

- `base_url`: their Jira site, `https://<site>.atlassian.net`.
- `projects`: run `factory/bin/check`, which now logs in; list the projects the
  account can see and ask which ones the factory may take tickets from. Don't
  leave it empty unless they ask for every project.
- `in_progress_status`: the check says whether each chosen project has a status
  with this name, and lists the real names when it doesn't. Use one of those.

## 4. The project's checks and app

- `factory/checks.sh`: when it is still the placeholder, replace the last two
  lines with the project's real commands: install what a fresh checkout needs,
  then lint, typecheck, tests and build, whichever exist. Find them in
  `AGENTS.md`, `CLAUDE.md`, `package.json`, `Makefile`, `pyproject.toml` or CI
  config. No watch modes, no prompts, nothing that rewrites files. Show the
  user the commands before writing them.
- `preview` in `factory/config.json`: `command` is how to start the app on a
  given port, with `{port}` where the port goes (for example
  `npm run dev -- --port {port}`); `setup` is what a fresh folder needs first
  (for example `npm ci`). Leave `command` empty if the project has no app to
  run, and tell them a ready job then won't open in the browser.

## 5. Verify and hand over

Run `factory/bin/check --run-checks`. Fix what it reports and run it again
until it prints `Ready`. If something can only be fixed by the user (a token, a
login, a Jira permission), say exactly what and stop.

In remote mode the factory's files must be on `origin/<base_branch>` before the
first run. Show the user what would be committed and ask before committing or
pushing.

Finish with three lines: the mode, where tickets come from (site and
projects), and the next step: add the label `kollifactory` to a small Jira
ticket and run `/kollifactory run <KEY>`.

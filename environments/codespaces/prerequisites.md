# Before the Workshop — Participant Prerequisites (Codespaces option, 5 minutes, zero installs)

Everything in this workshop runs **in your browser**, inside a GitHub Codespace (a full VS Code + Linux dev machine in the cloud). You will not install anything on your computer.

> **Lab environment option B — GitHub Codespaces.** Same Labs 1 → 2 → 3 as the Killercoda option. If you chose **Killercoda** instead, stop and use `../killercoda/prerequisites.md` (see also `../../prerequisites-chooser.md`).

## Create two free accounts (do this before the session)

| # | Account | Where | Used for | Cost |
|---|---|---|---|---|
| 1 | **GitHub** | https://github.com/signup | Hosts your code, runs the CI/CD pipeline, **and provides your lab environment** (Codespaces — included free with every personal account) | Free |
| 2 | **Docker Hub** | https://hub.docker.com/signup | The registry your container image ships to — **write your username down**, you'll type it several times | Free |

That's one account fewer than the Killercoda track — GitHub Codespaces *is* the lab environment, and it logs you into git automatically (no GitHub token to create).

## One thing to prepare in advance (2 minutes)

Create a **Docker Hub access token** (used by `docker login` in Lab 1 and by the pipeline in Lab 3, safer than your password):

1. Sign in at hub.docker.com → click your avatar → **Account Settings → Security → New Access Token**
2. Name: `workshop` · Permissions: **Read & Write** → **Generate**
3. **Copy it somewhere safe now** — Docker Hub shows it only once.

## Good to know about Codespaces (nothing to do, just read)

- Every personal GitHub account includes **120 core-hours/month free** — this workshop uses roughly 4–8 of them.
- Your codespace **auto-stops after 30 minutes idle** but **nothing is lost** — files, git history, everything persists. You reopen it and continue where you left off.
- A stopped codespace is kept for ~30 days; your pushed code lives on GitHub forever regardless.

## Quick self-check (1 minute)

- [ ] I can sign in to GitHub and see https://github.com/codespaces (an empty list is fine)
- [ ] I know my Docker Hub **username** and have my **access token** saved

## What you'll build

One microservice, end to end across **three labs**: **Lab 1** build & containerize → **Lab 2** run replicated on a multi-node Kubernetes cluster (kind) with visible load balancing → **Lab 3** deliver version 2.0 through a CI/CD pipeline into that same cluster, live. All in the browser, all free. (~70–80 minutes hands-on.)

*Browser requirements: any modern browser (Chrome, Edge, Firefox, Safari). A laptop/desktop is strongly recommended over a tablet — you'll be copy-pasting commands.*

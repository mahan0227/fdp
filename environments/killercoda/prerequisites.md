# Before the Workshop — Participant Prerequisites (Killercoda option, 5 minutes, zero installs)

Everything in this workshop runs **in your browser**. You will not install anything on your computer.

> **Lab environment option A — Killercoda.** Same Labs 1 → 2 → 3 as the Codespaces option. If you chose **GitHub Codespaces** instead, stop and use `../codespaces/prerequisites.md` (see also `../../prerequisites-chooser.md`).

## Create three free accounts (do this before the session)

| # | Account | Where | Used for | Cost |
|---|---|---|---|---|
| 1 | **Killercoda** | https://killercoda.com — sign in with email or GitHub | Your lab environment: a real 2-node Kubernetes cluster with Docker, in the browser | Free |
| 2 | **Docker Hub** | https://hub.docker.com/signup | The registry your container image ships to — **write your username down**, you'll type it several times | Free |
| 3 | **GitHub** | https://github.com/signup | Hosts your code and runs the CI/CD pipeline | Free |

## Two things to prepare in advance (3 minutes)

### 1. Docker Hub access token
(Used by `docker login` in Lab 1 and by the pipeline in Lab 3 — safer than your password.)

1. Sign in at hub.docker.com → click your avatar → **Account Settings → Security → New Access Token**
2. Name: `workshop` · Permissions: **Read & Write** → **Generate**
3. **Copy it somewhere safe now** — Docker Hub shows it only once.

### 2. GitHub personal access token
(Used in Lab 0 so Killercoda can `git push` to your repo — Codespaces does **not** need this.)

1. Sign in at github.com → **Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token**
2. Note: `workshop` · Expiration: 7 days (or longer) · Scope: check **`repo`** → **Generate token**
3. **Copy it somewhere safe now** — GitHub shows it only once. In the lab guide this is `<GHTOKEN>`.

## Quick self-check (1 minute)

- [ ] I can open https://killercoda.com/playgrounds/scenario/kubernetes and see a terminal boot in my browser
- [ ] I know my Docker Hub **username** and have my **Docker Hub access token** saved
- [ ] I have a GitHub **personal access token** with `repo` scope saved
- [ ] I can sign in to GitHub

## What you'll build

One microservice, end to end across **three labs**: **Lab 1** build & containerize → **Lab 2** run replicated on a 2-node Kubernetes cluster with visible load balancing → **Lab 3** deliver version 2.0 through a CI/CD pipeline into that same cluster, live. All in the browser, all free. (~70–80 minutes hands-on.)

*Browser requirements: any modern browser (Chrome, Edge, Firefox, Safari). A laptop/desktop is strongly recommended over a tablet — you'll be copy-pasting commands.*

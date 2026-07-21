# Lab Environment — Killercoda Track

Self-contained package for the Cloud Native Workshop labs on **Killercoda**.

Participants may choose this **or** the Codespaces track (`../codespaces/`). Same Labs **1 → 2 → 3**.

## What's in this folder

| File | Use |
|---|---|
| `prerequisites.md` / `.html` | Email to participants who chose Killercoda |
| `lab-guide.md` / `.html` | Hands-on guide: Lab 0 boot → Lab 1 → Lab 2 → Lab 3 (~70–80 min) |
| `demo-video-scripts.md` / `.html` | Shot-by-shot scripts for re-recording |
| `videos/lab1-build-and-ship.mp4` | Demo: build, isolate, push |
| `videos/lab2-kubernetes-hosting.mp4` | Demo: deploy, load-balance, heal, scale |
| `videos/lab3-cicd-end-to-end.mp4` | Demo: pipeline → rollout → rollback |

## How to run this track

1. Start from the workshop [`prerequisites-chooser`](../../prerequisites-chooser.md) so everyone picks a track.
2. Email this folder's `prerequisites.html`.
3. Share `lab-guide.html` when labs begin.
4. Optionally play each `videos/*.mp4` before that lab.
5. Playground: https://killercoda.com/playgrounds/scenario/kubernetes

## Environment notes

- 60-minute session cap — push after every lab (recovery box in Lab 0).
- NodePorts on `localhost:$PORT`.
- Needs GitHub PAT with `repo` scope for `git push` from the playground.

## Twin track

GitHub Codespaces: [`../codespaces/`](../codespaces/).

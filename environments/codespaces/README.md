# Lab Environment — GitHub Codespaces Track

Self-contained package for the Cloud Native Workshop labs on **GitHub Codespaces**.

Participants may choose this **or** the Killercoda track (`../killercoda/`). Same Labs **1 → 2 → 3**.

## What's in this folder

| File | Use |
|---|---|
| `prerequisites.md` / `.html` | Email to participants who chose Codespaces |
| `lab-guide.md` / `.html` | Hands-on guide: Lab 0 (kind) → Lab 1 → Lab 2 → Lab 3 (~70–80 min) |
| `demo-video-scripts.md` / `.html` | Shot-by-shot scripts for re-recording |
| `videos/lab1-build-and-ship.mp4` | Demo: boot codespace/kind, build, push |
| `videos/lab2-kubernetes-hosting.mp4` | Demo: deploy, `$URL` NodePort, load-balance, heal, scale |
| `videos/lab3-cicd-end-to-end.mp4` | Demo: pipeline → rollout → rollback |
| `videos/*-narration.json` | Scene timings for voice replacement |

## How to run this track

1. Start from the workshop [`prerequisites-chooser`](../../prerequisites-chooser.md).
2. Email this folder's `prerequisites.html`.
3. Share `lab-guide.html` when labs begin.
4. Optionally play each `videos/*.mp4` before that lab.
5. Participants: create repo `hello-cncf` → **Code → Codespaces → Create codespace on main**.

## Environment notes

- No hourly wipe — codespace auto-stops when idle; files persist. Kind cluster may need rebuild after a stop (Lab 0 recovery box).
- Reach the app via `$URL` (kind node IP + NodePort), **not** `localhost`.
- Git push works automatically — no GitHub PAT.
- Uses free Codespaces core-hours (~4–8 of 120/month).

## Twin track

Killercoda: [`../killercoda/`](../killercoda/).

## Regenerate walkthrough videos

```bash
python3 tools/generate_codespaces_videos.py
```

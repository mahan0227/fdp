# Cloud Native Workshop — Complete Facilitator Package

**Format:** Half-Day Workshop (3.5 hours) · **Audience:** Faculty Development · **Style:** Clean & Corporate

## Lab environments — participants choose one

Same **three labs** (1 → 2 → 3). Two browser environments. Email the chooser first:

| | **Killercoda** | **GitHub Codespaces** |
|---|---|---|
| Chooser | [`prerequisites-chooser.md`](prerequisites-chooser.md) / [`.html`](prerequisites-chooser.html) | ← start here |
| Package | [`environments/killercoda/`](environments/killercoda/) | [`environments/codespaces/`](environments/codespaces/) |
| Prep | `prerequisites.md` / `.html` | `prerequisites-codespaces.md` / `.html` |
| Guide | `lab-guide.md` / `.html` | `lab-guide-codespaces.md` / `.html` |
| Videos | `videos/lab1`, `lab2`, `lab3` (+ narration JSON) | `videos/lab1`, `lab2`, `lab3` (+ narration JSON) |

Each package is self-contained: prerequisites + lab guide + demo scripts + MP4s + HTML copies.

**App under test:** `hello-cncf` — web portal at `/`, JSON at `/api`, probes at `/healthz`. Lab videos show the guide steps **including how the portal opens in the browser**.

**Hands-on duration:** ~70–80 minutes (Lab 0 boot + Labs 1, 2, and 3 only — shortened session).

## What's in the full package

| File | What it is | How to use it |
|---|---|---|
| `cloud-native-workshop.pptx` | 20-slide deck with embedded animated GIFs and speaker notes | Present in slideshow mode; notes in Presenter View |
| `speaker-script.md` / `.html` / `.docx` | Slide-by-slide narration with ASK/DEMO/TRANSITION cues | Print or rehearse from |
| `teleprompter.html` | Auto-scrolling teleprompter | Browser · Space = pause · ↑/↓ = speed · R = reset |
| `prerequisites-chooser.md` / `.html` | **Participant environment chooser** (Killercoda vs Codespaces) | Email first, with both prep sheets |
| `environments/killercoda/` | Complete Killercoda kit (guide + 3 videos) | Share when participants pick Option A |
| `environments/codespaces/` | Complete Codespaces kit (guide + 3 videos) | Share when participants pick Option B |
| `prerequisites*.md` / `.html` · `lab-guide*.md` / `.html` | Root shortcuts to the same materials | Backward compatible |
| `demo-video-scripts.md` / `.html` | Recording scripts index (per-track scripts live in each environment folder) | Re-record in your own voice |
| `lab1/2/3-*.mp4` (repo root) | Killercoda walkthroughs (same as `environments/killercoda/videos/`) | Play before each lab block |
| `cicd-pipeline-diagram.*` · `kubernetes-architecture-diagram.*` | Readable SVG/PNG/HTML companions for slides 11 / 9 | Handouts or projector |
| `dashboard.html` | Interactive topic explorer | Pre-reading or post-workshop reference |
| `quiz.html` | 10-question interactive quiz | End of Part 2 or self-check |
| `feedback-form-google.gs` / `feedback-form.html` | Digital or paper feedback | Collect after the session |
| `certificate.html` | Print-ready certificate | Landscape print |
| `posters.pdf` | 4 printable posters | Room walls |
| `voice-replacement-kit.md` · `mux_voice.sh` | Swap narration on walkthrough videos | Optional |
| `tools/generate_lab_videos.py` | Regenerate detailed Lab 1–3 MP4s for both tracks | `python3 tools/generate_lab_videos.py` |
| [`archive/`](archive/) | Older Lab 2B + pre-portal videos (reference only) | Do not use in the live session |

## Suggested run of show (210 min)

1. **Foundations** (25 min) — slides 1–6
2. **Architecture I** (27 min) — slides 7–10 · *Break 10 min*
3. **Architecture II** (22 min) — slides 11–14
4. **Labs** (70–80 min) — slide 15 + chosen environment guide · *Break 10 min*
5. **Teaching toolkit + close** (36–46 min) — slides 16–20, quiz, feedback

## Track comparison (faculty quick reference)

| | **Killercoda** | **GitHub Codespaces** |
|---|---|---|
| Accounts | Killercoda + Docker Hub + GitHub | GitHub + Docker Hub |
| Tokens | Docker Hub + GitHub `repo` PAT | Docker Hub only |
| Cluster | Pre-built 2-node playground | Create 2-node **kind** cluster in Lab 0 |
| Reach the app (curl) | `localhost:$PORT/api` | `$URL/api` (kind node IP + NodePort) |
| Reach the portal (browser) | `http://localhost:$PORT` | `kubectl port-forward` → PORTS → Open in Browser |
| Persistence | Expires after 60 min (recover via Git push) | Files survive stops; kind may need rebuild |
| Videos | Detailed walkthroughs (portal + `/api` steps) | Detailed walkthroughs (portal + `/api` steps) |

## Before the session — facilitator checklist

- [ ] Email `prerequisites-chooser.html` **and** both prerequisites sheets
- [ ] Confirm venue network reaches Killercoda, GitHub Codespaces, Docker Hub, and GitHub
- [ ] Cue three videos per track under `environments/*/videos/`
- [ ] Open `teleprompter.html` on a second screen if desired
- [ ] Print `posters.pdf` and feedback forms
- [ ] Run the deck once in slideshow mode (GIFs on slides 9 and 11)
- [ ] Walk Labs 0→1→2→3 yourself on **both** tracks once before delivery

## Faculty dry-run (end-to-end)

1. Open `environments/killercoda/lab-guide.html` → complete Labs 0–3 (or follow videos).
2. Open `environments/codespaces/lab-guide.html` → complete Labs 0–3 in a fresh codespace.
3. Confirm checkpoints: Docker Hub tags `1.0` + `2.0`, green Actions run, curl `/api` loop shows cycling `served_by` then `version: 2.0`, and the **browser portal** shows the same version.

## Reuse rights

All materials were generated for this workshop and are yours to adapt. Data points cite the CNCF Annual Cloud Native Survey (2025) and CNCF/SlashData State of Cloud Native Development reports.

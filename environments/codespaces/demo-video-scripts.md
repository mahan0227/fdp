# Demo Video Recording Scripts — Cloud Native Workshop Labs
## GitHub Codespaces Edition

Shot-by-shot scripts matching the Codespaces lab guide and MP4s in `environments/codespaces/videos/`.

**Target output:** 3 videos · Lab 1 ≈ 3–4 min · Lab 2 ≈ 3–4 min · Lab 3 ≈ 3–4 min  
**Environment:** ONE GitHub Codespace on repo `hello-cncf`. Lab 0 creates a 2-node **kind** cluster.

> Killercoda twin: `environments/killercoda/demo-video-scripts.md`.

---

# VIDEO — Lab 1 · Build & Ship the Microservice

**Start state:** GitHub signed in · Docker Hub token ready · no codespace yet

### 🎬 SHOT 1 — Cold open
| 🗣️ SAY | "Welcome to the Cloud Native workshop labs on GitHub Codespaces. Three labs in one codespace: build and ship, Kubernetes hosting with kind, then CI/CD end to end. You install nothing on your computer." |

### 🎬 SHOT 2 — Lab 0 boot
| 🖱️ | New repo `hello-cncf` → Code → Codespaces → Create codespace on main |
| ⌨️ TYPE | Lab 0 verify + install + `kind create cluster` from the lab guide |
| 🗣️ SAY | "Repo already cloned, Docker ready, two node kind cluster Ready. Untaint the control plane so pods can schedule on both nodes." |

### 🎬 SHOT 3 — Create app + Dockerfile
| ⌨️/🖱️ | Paste `app.py` + `Dockerfile` (or use the VS Code editor) |
| 🗣️ SAY | "Served by exposes load balancing, version exposes rolling updates, health z is the probe hook." |

### 🎬 SHOT 4 — Build, isolate, push, save
| ⌨️ TYPE | `docker build` · run 8080/8081 · login · tag · push · `git push` |
| 🗣️ SAY | "Build, prove isolation, ship to Docker Hub. Codespaces authenticates git automatically." |
| 🎬 | Stop recording |

---

# VIDEO — Lab 2 · Host & Manage on Kubernetes

**Start state:** Same codespace · image pushed · kind cluster Ready

### 🎬 SHOT 1 — Cold open
| 🗣️ SAY | "Lab 2, in the same Codespace. Replicated, load balanced, self healing, health probed." |

### 🎬 SHOT 2 — Deploy
| ⌨️ TYPE | `kubectl apply -f app.yaml` · `kubectl get pods -o wide` |
| 🗣️ SAY | "Three pods spread across both kind nodes." |

### 🎬 SHOT 3 — Service + `$URL` (Codespaces-specific)
| ⌨️ TYPE | expose NodePort · capture `PORT` + `NODE_IP` into `$URL` · `curl -s $URL` |
| 🗣️ SAY | "On Codespaces the NodePort lives on the kind node container I P, not localhost." |

### 🎬 SHOT 4 — Load balance, heal, scale ⭐
| ⌨️ TYPE | curl loop on `$URL` · delete a pod · scale 5 → 3 |
| 🗣️ SAY | "Served by changes every request. Kill a pod — replacement appears. Keep this codespace open for Lab 3." |
| 🎬 | Stop recording |

---

# VIDEO — Lab 3 · CI/CD End to End

**Start state:** Same codespace · secrets `DOCKERHUB_USERNAME` + `DOCKERHUB_TOKEN` set

### 🎬 SHOT 1 — Cold open
| 🗣️ SAY | "Lab 3: edit, commit, and push from the codespace. Actions builds in the cloud. You roll out on the same kind cluster." |

### 🎬 SHOT 2 — v2.0 + tests + workflow
| 🖱️/⌨️ | Edit to 2.0 · add `test_app.py` · add `.github/workflows/cicd.yml` · commit · push |
| 🗣️ SAY | "That push triggers the pipeline. Watch Actions go green." |

### 🎬 SHOT 3 — Rollout + proof + undo ⭐
| ⌨️ TYPE | `kubectl set image ...:2.0` · curl `$URL` · `kubectl rollout undo` |
| 🗣️ SAY | "Every response says version two point zero, still load balanced. Undo once — version one point zero is back." |
| 🎬 | Stop recording |

---

## After Recording / Packaging

- [ ] Export 1080p H.264 into `environments/codespaces/videos/`
- [ ] Names: `lab1-build-and-ship.mp4`, `lab2-kubernetes-hosting.mp4`, `lab3-cicd-end-to-end.mp4`
- [ ] Or regenerate walkthroughs: `python3 tools/generate_codespaces_videos.py`

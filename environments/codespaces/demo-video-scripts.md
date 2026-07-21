# Demo Video Recording Scripts — Cloud Native Workshop Labs
## GitHub Codespaces Edition

Shot-by-shot scripts matching the Codespaces lab guide and MP4s in `environments/codespaces/videos/`.

**App faces:** browser portal at `/` · JSON at `/api` · probes at `/healthz`  
**Target output:** 3 detailed videos · Lab 1 · Lab 2 · Lab 3  
**Environment:** ONE GitHub Codespace on repo `hello-cncf`. Lab 0 creates a 2-node **kind** cluster.

> Killercoda twin: `environments/killercoda/demo-video-scripts.md`.  
> Regenerate: `python3 tools/generate_lab_videos.py`

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
| ⌨️/🖱️ | Paste `app.py` + `Dockerfile` (portal at `/`, JSON at `/api`) |
| 🗣️ SAY | "Slash serves the web portal. Slash api returns JSON for curl. Health z is the probe hook." |

### 🎬 SHOT 4 — Build, curl, Open in Browser, push
| ⌨️ TYPE | `docker build` · run 8080/8081 · `curl localhost:8080/api` · **PORTS → 8080 → Open in Browser** · prove isolation · login · tag · push · `git push` |
| 🗣️ SAY | "Curl slash api for JSON. Open the Ports tab, select eighty eighty, Open in Browser to launch the portal. Prove isolation, then ship." |
| 🎬 | Stop recording |

---

# VIDEO — Lab 2 · Host & Manage on Kubernetes

**Start state:** Same codespace · image pushed · kind cluster Ready

### 🎬 SHOT 1 — Cold open
| 🗣️ SAY | "Lab 2, in the same Codespace. Replicated, load balanced, self healing, health probed." |

### 🎬 SHOT 2 — Deploy
| ⌨️ TYPE | `kubectl apply -f app.yaml` · `kubectl get pods -o wide` |
| 🗣️ SAY | "Three pods spread across both kind nodes." |

### 🎬 SHOT 3 — Service + `$URL` + port-forward portal
| ⌨️ TYPE | expose NodePort · `$URL` · `curl -s $URL/api` · `kubectl port-forward svc/hello-cncf 8080:80` · **PORTS → Open in Browser** |
| 🗣️ SAY | "Curl slash api via the kind node URL. For the browser, port forward to localhost eighty eighty, then Open in Browser. Kind NodePorts are not visible to your laptop." |

### 🎬 SHOT 4 — Load balance, heal, scale ⭐
| ⌨️ TYPE | curl loop on `$URL/api` · refresh portal · delete a pod · scale 5 → 3 · git push `app.yaml` |
| 🗣️ SAY | "Served by changes every request on curl and on the portal. Kill a pod — replacement appears. Keep this codespace open for Lab 3." |
| 🎬 | Stop recording |

---

# VIDEO — Lab 3 · CI/CD End to End

**Start state:** Same codespace · secrets `DOCKERHUB_USERNAME` + `DOCKERHUB_TOKEN` set

### 🎬 SHOT 1 — Cold open
| 🗣️ SAY | "Lab 3: edit, commit, and push from the codespace. Actions builds in the cloud. You roll out on the same kind cluster." |

### 🎬 SHOT 2 — v2.0 + tests + workflow
| 🖱️/⌨️ | Edit to 2.0 · add `test_app.py` (asserts `/api`) · add `.github/workflows/cicd.yml` · commit · push |
| 🗣️ SAY | "That push triggers the pipeline. Watch Actions go green." |

### 🎬 SHOT 3 — Rollout + curl + portal + undo ⭐
| ⌨️ TYPE | `kubectl set image ...:2.0` · `curl -s $URL/api` · **refresh port-forward portal → 2.0** · `kubectl rollout undo` |
| 🗣️ SAY | "Curl and the portal both show version two point zero. Undo once — version one point zero is back." |
| 🎬 | Stop recording |

---

## After Recording / Packaging

- [ ] Export 1080p H.264 into `environments/codespaces/videos/`
- [ ] Names: `lab1-build-and-ship.mp4`, `lab2-kubernetes-hosting.mp4`, `lab3-cicd-end-to-end.mp4`
- [ ] Or regenerate: `python3 tools/generate_lab_videos.py`

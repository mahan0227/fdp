# Demo Video Recording Scripts — Cloud Native Workshop Labs
## Killercoda Edition

Shot-by-shot scripts matching the Killercoda lab guide and MP4s in `environments/killercoda/videos/`.

**Target output:** 3 videos · Lab 1 ≈ 3–4 min · Lab 2 ≈ 3–4 min · Lab 3 ≈ 3–4 min  
**Environment:** ONE Killercoda Kubernetes playground for Labs 1–2; GitHub in a second tab for Lab 3.

> Codespaces twin: `environments/codespaces/demo-video-scripts.md`.

---

# VIDEO — Lab 1 · Build & Ship the Microservice

**Start state:** Killercoda Kubernetes playground, fresh session · GitHub repo `hello-cncf` created · tokens from prerequisites ready

### 🎬 SHOT 1 — Cold open
| 🗣️ SAY | "Welcome to the Cloud Native workshop labs. The entire hands on session runs in one free Killercoda browser session: a real two node Kubernetes cluster with Docker on board. You will install nothing on your computer. Three labs: build and ship, Kubernetes hosting, then CI/CD end to end." |

### 🎬 SHOT 2 — Lab 0 verify + clone
| ⌨️ TYPE | `kubectl get nodes` · docker guard · `git clone` into `~/hello-cncf` · wire push URL with `<GHTOKEN>` |
| 🗣️ SAY | "Lab zero: two Ready nodes, Docker present, repo cloned, push credential wired once." |

### 🎬 SHOT 3 — Create app + Dockerfile
| ⌨️ TYPE | `cat > app.py` and `Dockerfile` blocks from the lab guide |
| 🗣️ SAY | "Served by exposes load balancing, version exposes rolling updates, health z is the probe hook." |

### 🎬 SHOT 4 — Build, isolate, push, save
| ⌨️ TYPE | `docker build` · run 8080/8081 · login · tag · push · `git commit` · `git push` |
| 🗣️ SAY | "Build, prove isolation, ship to Docker Hub, and push — your code survives the sixty minute session cap." |
| 🎬 | Stop recording |

---

# VIDEO — Lab 2 · Host & Manage on Kubernetes

**Start state:** SAME Killercoda terminal · image pushed

### 🎬 SHOT 1 — Cold open
| 🗣️ SAY | "Lab 2, in the same Killercoda terminal. The image from Lab 1 will run replicated, load balanced, self healing, and health probed." |

### 🎬 SHOT 2 — Deploy + Service
| ⌨️ TYPE | `kubectl apply -f app.yaml` · `get pods -o wide` · `kubectl expose ... NodePort` · capture `$PORT` |
| 🗣️ SAY | "Three pods across both nodes. The Service is one stable address." |

### 🎬 SHOT 3 — Load balance, heal, scale ⭐
| ⌨️ TYPE | curl loop on `localhost:$PORT` · delete a pod · scale 5 → 3 |
| 🗣️ SAY | "Served by changes every request. Kill a pod — replacement appears. Scale is one number. Keep this session open for Lab 3." |
| 🎬 | Stop recording |

---

# VIDEO — Lab 3 · CI/CD End to End

**Start state:** GitHub repo holds Labs 1–2 files · secrets set · Killercoda tab still open

### 🎬 SHOT 1 — Cold open
| 🗣️ SAY | "Lab 3: a code change travels through a pipeline into the registry and rolls out on the Killercoda cluster from Lab 2, with zero downtime." |

### 🎬 SHOT 2 — Bump to v2.0 + tests + workflow
| 🖱️/⌨️ | Edit existing `app.py` + `Dockerfile` · add `test_app.py` · add `.github/workflows/cicd.yml` · commit |
| 🗣️ SAY | "That commit triggers Actions. Watch it go green and Docker Hub show tag two point zero." |

### 🎬 SHOT 3 — Rollout + proof + undo ⭐
| ⌨️ TYPE | `kubectl set image ...:2.0` · curl `localhost:$PORT` · `kubectl rollout undo` |
| 🗣️ SAY | "Every response says version two point zero. Undo once — version one point zero is back." |
| 🎬 | Stop recording |

---

## After Recording

- [ ] Export 1080p H.264 MP4 into `environments/killercoda/videos/`
- [ ] Names: `lab1-build-and-ship.mp4`, `lab2-kubernetes-hosting.mp4`, `lab3-cicd-end-to-end.mp4`

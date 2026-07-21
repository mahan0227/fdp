# Lab Guide: Cloud Native Workshop — One App, One Environment, End to End

**The session:** you will build ONE microservice and take it through the entire cloud native journey — containerize it, run it replicated on a real 2-node Kubernetes cluster, and deliver an update through a CI/CD pipeline into that same running cluster.

**The environment:** the whole hands-on session lives in **one Killercoda browser session** — and because Killercoda expires after 60 minutes, **your GitHub repo is the durable home for your work**: you clone it in Lab 0, push after every lab, and clone it back in ~2 minutes if the session ever dies (a free, real 2-node Kubernetes cluster with Docker on board). Lab 3 adds GitHub — also browser-only. **You install nothing on your computer.**

**Duration:** 70–80 minutes · **Cost:** $0

> **Choose your lab environment** (same Labs 1 → 2 → 3 either way):
> - **This guide = Killercoda** — prep sheet: `prerequisites.md` (or `environments/killercoda/`)
> - **Codespaces option** — use `../codespaces/lab-guide.md` + `prerequisites-codespaces.md` (or `../codespaces/`)
>
> Complete the matching prerequisites before the session. Same app, same checkpoints; only Lab 0 and how you reach the Service differ.

## The application: `hello-cncf`

One tiny Python microservice, engineered so the platform's invisible behavior becomes visible:

| Design choice | What it makes visible |
|---|---|
| `served_by: <hostname>` in every response | **Load balancing** — repeated requests show different pod names |
| `version` in every response | **Rolling updates** — watch v1.0 become v2.0 live |
| `/healthz` endpoint | **Probes & self-healing** — Kubernetes' hook into the app |
| Stateless, env-configured | **Scalability** — any replica serves any request |
| Browser portal at `/` + JSON at `/api` | **Same app, two faces** — humans use the portal; curl demos use `/api` |

---

## Lab 0: Boot Your Environment (5 minutes — do once, keep it open all session)

1. Open **https://killercoda.com/playgrounds/scenario/kubernetes** and sign in.
2. A 2-node cluster boots (~30 s). Verify:
```bash
kubectl get nodes
```
   Expected:
```
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   ...   v1.3x.x
node01         Ready    <none>          ...   v1.3x.x
```
3. Guarantee Docker is present (one line — installs into the *browser VM* if missing, never onto your machine):
```bash
which docker || (apt-get update -qq && apt-get install -y -qq docker.io)
docker version --format 'Docker {{.Server.Version}} ready'
```
   Expected: `Docker 2x.x.x ready`

4. **Create your GitHub repo and clone it — this is where your work lives.**

On **github.com**: **+ → New repository** → name it `hello-cncf` → **Public** → check **Add a README** → **Create repository**. (Lab 3's pipeline will run from this same repo.)

Back in the Killercoda terminal — set your identity and clone (replace both placeholders):

```bash
git config --global user.name "<GHUSER>"
git config --global user.email "<GHUSER>@users.noreply.github.com"
git clone https://github.com/<GHUSER>/hello-cncf.git
cd hello-cncf
```

Expected: `Cloning into 'hello-cncf'... done.` — your prompt is now inside `hello-cncf/`.

5. **Wire the push credential once** (uses the GitHub token from your prerequisites sheet):

```bash
git remote set-url origin https://<GHUSER>:<GHTOKEN>@github.com/<GHUSER>/hello-cncf.git
git push origin main && echo "PUSH OK"
```

Expected: `Everything up-to-date` (or a small push), then `PUSH OK`.
**🔧 If it fails** with `Invalid username or password`: the token is wrong or lacks scope → regenerate it with **`repo`** scope and re-run the `set-url` line. *(The token lives only in this throwaway VM, destroyed when the session ends.)*

> ### 💾 THE GOLDEN RULE OF THIS WORKSHOP
> **Do all your work inside `hello-cncf/`, and push after every lab.**
> Killercoda destroys your VM after 60 minutes — but everything you pushed is safe on GitHub, and your image is safe on Docker Hub. A **💾 Save your work** box ends each lab; the **🔄 Session expired?** box below brings you back in ~2 minutes.

> ### 🔄 SESSION EXPIRED? Recover in 2 minutes (bookmark this box)
> Open a fresh playground → re-run Lab 0 steps 2–3 (cluster + docker check) → then paste:
>
>     git config --global user.name "<GHUSER>"
>     git config --global user.email "<GHUSER>@users.noreply.github.com"
>     git clone https://<GHUSER>:<GHTOKEN>@github.com/<GHUSER>/hello-cncf.git
>     cd hello-cncf
>     kubectl apply -f app.yaml 2>/dev/null || echo "(no app.yaml yet — you were still in Lab 1)"
>     kubectl get pods
>
> Your code is back, and if you'd reached Lab 2 your app is already redeploying — Kubernetes pulls the image straight from Docker Hub, so **nothing is rebuilt**. Need the Service again? Re-run Lab 2 step 3 (`kubectl expose ...` + capture `PORT`).

> **Session note:** Killercoda sessions last 60 minutes. If yours expires mid-workshop, open a fresh playground and re-run Lab 0 + the "Fast rebuild" box at the end of Lab 2 — under 3 minutes to be fully back.

---

## Lab 1: Build & Ship the Microservice (20 minutes)
*Same terminal. Objective: understand the app, containerize it, prove isolation, push to the registry.*

### Steps

> ### ⚠️ READ THIS ONCE — how the file-creation blocks work
> Blocks that start with `cat > somefile << 'EOF'` and end with a line containing only `EOF` are **shell commands**. The first and last lines create the file — **they must never appear inside it**.
> - ✅ Paste the **whole block** at the terminal **`$` prompt** and press Enter.
> - ❌ Never paste these blocks inside an editor (nano/vim) — that's how `cat > app.py` ends up *inside* your file.
> - Every file step is followed by a **✔ Verify** check that catches mistakes instantly, with a one-line fix.
> - Prefer an editor? Use **Plan B** at the end of Lab 1 — file contents shown *without* the cat/EOF lines.

1. **Create the application** — you're already inside `hello-cncf/` from Lab 0. Paste the whole block at the `$` prompt:
```bash
cd ~/hello-cncf
cat > app.py << 'EOF'
import os, socket
from flask import Flask, jsonify, render_template_string

app = Flask(__name__)
VERSION = os.getenv("APP_VERSION", "1.0")

PORTAL = """
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1"/>
  <title>hello-cncf</title>
  <style>
    :root { --bg:#0F172A; --card:#1E293B; --accent:#38BDF8; --ok:#34D399; --text:#E2E8F0; --muted:#94A3B8; }
    * { box-sizing: border-box; }
    body { margin:0; min-height:100vh; font-family: Segoe UI, system-ui, sans-serif;
           background: radial-gradient(1200px 600px at 10% -10%, #1E3A5F 0%, var(--bg) 55%);
           color: var(--text); display:flex; align-items:center; justify-content:center; padding:24px; }
    .card { width:min(520px,100%); background:var(--card); border:1px solid #334155; border-radius:18px;
            padding:28px 28px 22px; box-shadow:0 20px 50px rgba(0,0,0,.35); }
    .eyebrow { color:var(--accent); font-size:13px; font-weight:700; letter-spacing:.08em; text-transform:uppercase; }
    h1 { margin:8px 0 6px; font-size:28px; }
    .msg { color:var(--muted); margin:0 0 22px; line-height:1.5; }
    .grid { display:grid; gap:12px; }
    .row { background:#0F172A; border-radius:12px; padding:14px 16px; display:flex; justify-content:space-between; gap:12px; }
    .k { color:var(--muted); font-size:13px; }
    .v { font-weight:700; font-family: ui-monospace, SFMono-Regular, Menlo, monospace; word-break:break-all; text-align:right; }
    .v.ok { color:var(--ok); }
    .hint { margin-top:18px; font-size:13px; color:var(--muted); }
    button { margin-top:14px; width:100%; border:0; border-radius:10px; padding:12px 14px; font-weight:700;
             background:var(--accent); color:#0F172A; cursor:pointer; }
  </style>
</head>
<body>
  <div class="card">
    <div class="eyebrow">Cloud Native Workshop</div>
    <h1>hello-cncf</h1>
    <p class="msg">{{ message }}</p>
    <div class="grid">
      <div class="row"><span class="k">version</span><span class="v ok">{{ version }}</span></div>
      <div class="row"><span class="k">served_by</span><span class="v">{{ served_by }}</span></div>
      <div class="row"><span class="k">service</span><span class="v">{{ service }}</span></div>
    </div>
    <button onclick="location.reload()">Refresh — watch served_by change</button>
    <p class="hint">JSON API for curl demos: <code>/api</code> · health: <code>/healthz</code></p>
  </div>
</body>
</html>
"""

def payload():
    return dict(
        service="hello-cncf",
        version=VERSION,
        served_by=socket.gethostname(),
        message="Hello from Cloud Native!",
    )

@app.get("/")
def portal():
    return render_template_string(PORTAL, **payload())

@app.get("/api")
def api():
    return jsonify(payload())

@app.get("/healthz")
def healthz():
    return "ok", 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
EOF
ls
```
   Expected: `app.py`

   **✔ Verify** (catches the classic paste mistake in 2 seconds):

```bash
head -1 app.py && grep -c "cat > \|^EOF" app.py
```

   Expected: `import os, socket` then `0`. **🔧 If wrong** (you see `cat > app.py` or a count above 0): `rm app.py`, re-paste the block at the `$` prompt — or use Plan B below.

2. **Dependencies + Dockerfile** — the build recipe:
```bash
echo "flask==3.0.3" > requirements.txt
cat > Dockerfile << 'EOF'
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
ENV APP_VERSION=1.0
EXPOSE 8080
CMD ["python", "app.py"]
EOF
```

   **✔ Verify:**

```bash
head -1 Dockerfile && grep -c "^EOF" Dockerfile
```

   Expected: `FROM python:3.12-slim` then `0`. **🔧 If wrong:** `rm Dockerfile`, re-paste at the `$` prompt.

3. **Build the image:**
```bash
docker build -t hello-cncf:1.0 .
```
   Expected (last line): `Successfully tagged hello-cncf:1.0`

4. **Run & test it (API + web portal):**
```bash
docker run -d -p 8080:8080 --name svc1 hello-cncf:1.0
curl -s localhost:8080/api
curl -s localhost:8080/healthz
```
   Expected: JSON with `"version":"1.0"` and `"served_by"`, then `ok`. **Remember `served_by`** — it becomes the star of Lab 2.

   **🌐 Open the web portal:** in a new browser tab open **http://localhost:8080**
   You should see the hello-cncf portal (version + served_by). Click **Refresh**. *(Killercoda exposes localhost ports to your laptop browser.)*

5. **Prove isolation** — second copy, same host, zero conflict:
```bash
docker run -d -p 8081:8080 --name svc2 hello-cncf:1.0
curl -s localhost:8080/api | grep -o '"served_by":"[^"]*"'
curl -s localhost:8081/api | grep -o '"served_by":"[^"]*"'
```
   Expected: **two different hostnames**. Optional: open **http://localhost:8081** for a second portal instance.

6. **Ship it to the registry** (the bridge to Labs 2 and 3):
```bash
docker login          # Docker Hub username + password (or token)
docker tag hello-cncf:1.0 <DOCKERUSER>/hello-cncf:1.0
docker push <DOCKERUSER>/hello-cncf:1.0
```
   Expected: ends with a `sha256:` digest. Browser-verify: `hub.docker.com/r/<DOCKERUSER>/hello-cncf` shows tag `1.0`.

### 💾 Save your work (30 seconds — do not skip)

```bash
cd ~/hello-cncf
git add app.py requirements.txt Dockerfile
git commit -m "Lab 1: hello-cncf microservice + Dockerfile"
git push origin main
```

Expected: the output ends with `main -> main`. Refresh your repo on github.com — the three files are there. **Your code now survives any session timeout** (your image is already safe on Docker Hub).

### 🅱️ Plan B — create files with an editor instead
If pasting blocks at the prompt keeps going wrong: `nano app.py`, paste **only the file contents** (everything *between* the `cat` line and the `EOF` line — never those two lines), then **Ctrl+O, Enter, Ctrl+X**. Same for `Dockerfile`. Re-run the ✔ Verify checks afterwards.

### ✅ Checkpoint 1
You can explain: image vs container, layers, isolation, the registry hand-off, and opening the app as a web portal. Clean up the test containers: `docker rm -f svc1 svc2`

---

## Lab 2: Host & Manage It on Kubernetes (28 minutes)
*Same terminal, same session. Objective: replicate the app across 2 nodes and watch Kubernetes load-balance, probe, heal, and scale it.*

### Steps

1. **Declare the Deployment** — 3 replicas, health-probed:
```bash
cd ~/hello-cncf && cat > app.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-cncf
spec:
  replicas: 3
  selector:
    matchLabels: { app: hello-cncf }
  template:
    metadata:
      labels: { app: hello-cncf }
    spec:
      containers:
      - name: web
        image: <DOCKERUSER>/hello-cncf:1.0
        ports: [{ containerPort: 8080 }]
        readinessProbe:
          httpGet: { path: /healthz, port: 8080 }
          initialDelaySeconds: 2
          periodSeconds: 5
EOF
sed -i "s/<DOCKERUSER>/YOUR_ACTUAL_USERNAME/" app.yaml    # <-- edit!
kubectl apply -f app.yaml
```
   **✔ Verify first:** `head -2 app.yaml` must show `apiVersion: apps/v1` — not a `cat >` line. **🔧 If wrong:** `rm app.yaml`, re-paste at the `$` prompt.

   Expected: `deployment.apps/hello-cncf created`
   > **Foolproof fallback:** if your Lab 1 push failed, substitute image `stefanprodan/podinfo` with ports/probe on `9898` — every step below works identically.

2. **Pods spread across nodes automatically:**
```bash
kubectl get pods -o wide
```
   Expected: 3 pods `1/1 Running`, placed across `controlplane` and `node01`.

3. **Service = built-in load balancer:**
```bash
kubectl expose deployment hello-cncf --port=80 --target-port=8080 --type=NodePort
PORT=$(kubectl get svc hello-cncf -o jsonpath='{.spec.ports[0].nodePort}')
echo "Service on port $PORT"
curl -s localhost:$PORT/api
```

   **🌐 Open the web portal on Kubernetes:** in your laptop browser open **http://localhost:$PORT**
   (paste the number `echo` printed). Refresh a few times — **served_by** should change as the Service load-balances across pods.

4. **⭐ SEE load balancing** — what `served_by` was built for:
```bash
for i in $(seq 1 8); do curl -s localhost:$PORT/api | grep -o '"served_by":"[^"]*"'; done
```
   Expected: the hostname **changes between requests**, cycling across all 3 pods. One address, distributed traffic — that's a Service.

5. **⭐ SEE self-healing** — kill a pod on purpose:
```bash
kubectl get pods                       # note a pod name
kubectl delete pod <paste-a-pod-name>
kubectl get pods
```
   Expected: deleted pod `Terminating`, replacement already `ContainerCreating`. Re-run the curl loop — traffic never stopped (readiness probes steered around the dying pod).

6. **Scale declaratively:**
```bash
kubectl scale deployment hello-cncf --replicas=5
for i in $(seq 1 8); do curl -s localhost:$PORT/api | grep -o '"served_by":"[^"]*"'; done
kubectl scale deployment hello-cncf --replicas=3
```
   Expected: the loop cycles across **five** hostnames, then back to three.

7. **The declaration Kubernetes defends:**
```bash
kubectl get deployment hello-cncf -o yaml | grep " replicas:"
```
   Expected: `  replicas: 3` — the thermostat setting.

> **Fast rebuild box** (if your session ever expires): Lab 0 → Lab 2 steps 1 & 3 → you're back in ~3 minutes; your image is safe on Docker Hub.

### ✅ Checkpoint 2
You can explain — with evidence you produced — Deployments, Services & load balancing, readiness probes, self-healing, and declarative scaling.

### 💾 Save your work

```bash
cd ~/hello-cncf
git add app.yaml
git commit -m "Lab 2: Kubernetes Deployment manifest with readiness probe"
git push origin main
```

Expected: ends with `main -> main`. Now your cluster manifest is recoverable too — the 🔄 Session expired? box re-applies it automatically.

**Keep this session open — Lab 3 continues right here.** (If it expires, use the 🔄 recovery box: ~2 minutes.)

---


## Lab 3: CI/CD End to End — Commit to Cluster (30 minutes)
*GitHub in one browser tab, your Killercoda terminal in another. Objective: a code change travels commit → pipeline → registry → rolling update → visible to users.*

### One-time setup (2 min — your repo already exists from Lab 0)
1. Your `hello-cncf` repo on GitHub already holds `app.py`, `Dockerfile`, and `app.yaml` — everything you pushed during Labs 1–2. **The pipeline will build from exactly that repo.**
2. Repo **Settings → Secrets and variables → Actions → New repository secret** — add both:
   - `DOCKERHUB_USERNAME` = your Docker Hub username
   - `DOCKERHUB_TOKEN` = the access token from your prerequisites sheet

### Steps

1. **Bump the app to v2.0 — editing the files you already pushed.** On github.com, open `app.py` → pencil ✏️ → change two things: `VERSION = os.getenv("APP_VERSION", "2.0")` and inside `payload()` set `message="Now delivered by a pipeline!"` → **Commit to main**. Then open `Dockerfile` → pencil ✏️ → change `ENV APP_VERSION=1.0` to `ENV APP_VERSION=2.0` → **Commit to main**.

   *(Prefer the terminal? `cd ~/hello-cncf`, edit with `nano`, then `git add -A && git commit -m "v2.0" && git push origin main` — same result. Just remember to `git pull` in the terminal afterwards if you also edit on the web.)*

2. **Add the quality gate** — `test_app.py`:
```python
from app import app

def test_root():
    r = app.test_client().get("/api")
    data = r.get_json()
    assert r.status_code == 200
    assert data["service"] == "hello-cncf"
    assert data["version"] == "2.0"

def test_healthz():
    assert app.test_client().get("/healthz").status_code == 200
```
   Commit.

3. **Declare the pipeline** — `.github/workflows/cicd.yml` (type the full path; folders auto-create):
```yaml
name: CICD
on: [push]

jobs:
  test-build-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Run tests
        run: |
          pip install flask==3.0.3 pytest
          pytest -v

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push v2.0
        run: |
          docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/hello-cncf:2.0 .
          docker push ${{ secrets.DOCKERHUB_USERNAME }}/hello-cncf:2.0
```
   Commit — **this commit triggers the pipeline.**

4. **Watch it go green.** **Actions** tab → all steps ✓. Docker Hub now shows tag `2.0` beside `1.0` — the pipeline just did automatically everything you did by hand in Lab 1.

   **💾 Your work is already saved** — the pipeline runs *from* your repo, so the code is on GitHub by definition. If your Killercoda session expired while the pipeline ran, use the 🔄 recovery box (Lab 0), then continue below.

5. **⭐ Roll it out** — back in your Killercoda tab:
```bash
kubectl set image deployment/hello-cncf web=<DOCKERUSER>/hello-cncf:2.0
kubectl rollout status deployment/hello-cncf
```
   Expected: `deployment "hello-cncf" successfully rolled out` — pods replaced one by one, zero downtime, gated by your readiness probe.

6. **⭐ Prove it to users:**
```bash
for i in $(seq 1 6); do curl -s localhost:$PORT/api | grep -oE '"(version|served_by)":"[^"]*"' | paste -d' ' - -; done
```
   Expected: every answer `"version":"2.0"`, still load-balanced across fresh pods.

   **🌐 Portal check:** refresh **http://localhost:$PORT** — the page should now show **version 2.0** and the new message.

7. **Instant rollback:**
```bash
kubectl rollout undo deployment/hello-cncf
curl -s localhost:$PORT/api | grep -o '"version":"[^"]*"'
```
   Expected: `"version":"1.0"`. Change is cheap in both directions.

8. **Correlate out loud:** commit (GitHub) → tested & built (Actions) → immutable versioned image (Docker Hub) → declarative rolling update (Kubernetes) → user-visible change (`version`) → one-command rollback. **One app, one environment, the entire loop — and you operated every stage.**

### 💾 Final save — sync the terminal copy

Your repo already has everything (the pipeline built from it). Pull the web-edited changes back into the VM so both copies match:

```bash
cd ~/hello-cncf && git pull origin main && ls
```

Expected: `app.py  app.yaml  Dockerfile  requirements.txt  test_app.py  README.md` — your complete workshop artifact, permanently on GitHub. **Everything you built today is reproducible from this repo: clone it anywhere, apply `app.yaml`, and the app runs.**

### ✅ Checkpoint 3
You can trace one change through every stage of the delivery loop and name the guarantee each stage provides.

---

## Troubleshooting (foolproof table)

| Symptom | Likely cause | Fix |
|---|---|---|
| `git push`: `Invalid username or password` | GitHub token wrong, expired, or missing `repo` scope | Regenerate a classic token with **repo** scope, re-run the Lab 0 `git remote set-url` line |
| `git push`: `rejected — non-fast-forward` | You edited files on github.com AND in the terminal | `git pull --rebase origin main`, then push again |
| Lost everything — session expired | Killercoda 60-min cap | Use the **🔄 Session expired?** box in Lab 0 — clone + apply, back in ~2 minutes; nothing is rebuilt |
| `git clone` asks for a password | Cloned the plain HTTPS URL | Clone with the token URL from the recovery box, or re-run `git remote set-url` |
| A file contains a `cat > ...` or `EOF` line | Block pasted inside an editor, or the closing EOF was indented | `rm <file>`, re-paste the whole block at the `$` prompt (see the ⚠️ callout in Lab 1) — or Plan B with nano |
| Lab 0: `docker: command not found` after install line | apt lock still held on fresh VM | Wait 20 s, re-run the Lab 0 docker line |
| `denied: requested access to the resource is denied` on push | Tag missing your username, or not logged in | `docker login`, re-tag `<DOCKERUSER>/hello-cncf:1.0`, push again |
| Pods `ImagePullBackOff` | Username typo in app.yaml, or push skipped | `kubectl describe pod <name>` shows the pull error; fix the image ref, or use fallback `stefanprodan/podinfo` (port 9898) |
| Pods `Pending` | Over-scaled the small cluster | `kubectl scale deployment hello-cncf --replicas=3` |
| curl loop shows one hostname only | Other pods not Ready yet | `kubectl get pods` — wait for all `1/1`, retry |
| Browser shows raw JSON instead of portal | You opened `/api` | Open `/` (no path) for the portal; `/api` is for curl |
| Portal page won't load | Wrong port or Service not ready | `echo $PORT` and open `http://localhost:$PORT`; confirm `kubectl get svc,pods` |
| Portal still shows version 1.0 after Lab 3 | Browser cache or old pods | Hard-refresh; confirm rollout finished; check `curl -s localhost:$PORT/api` |
| Killercoda session expired | 60-min cap | New playground → Lab 0 → Fast-rebuild box (~3 min); image is safe on Docker Hub |
| Actions: `Username and password required` | Secrets missing/misnamed | Exactly `DOCKERHUB_USERNAME` / `DOCKERHUB_TOKEN` under Settings → Secrets → Actions |
| Actions: `insufficient_scope` on push | Token is read-only | New token with **Read & Write** |
| Tests fail `ModuleNotFoundError: flask` | pip line missed | `Run tests` step must `pip install flask==3.0.3 pytest` |
| `kubectl set image`: container not found | Name mismatch | It's `web=` (container name from app.yaml) |
| Rollout stuck | v2.0 failing readiness | `kubectl describe pod <new-pod>`; `kubectl rollout undo` recovers instantly |

## Facilitator & reuse notes

- Recordings in `videos/` (`lab1/2/3-*.mp4`) — and the same files at repo root — mirror this guide step-for-step, including **opening the web portal in the browser**. Play a lab's video first, then release participants to reproduce it, or share videos for self-paced follow-along.
- **Codespaces twin package:** `../codespaces/` (guides + Codespaces walkthrough videos with Ports / port-forward portal steps).
- The `served_by` + `version` + `/healthz` trio (portal at `/`, JSON at `/api`) is the reusable teaching pattern — steal it for any demo app.
- Assessment: fork the repo; grade = green pipeline + own Docker Hub tags 1.0 & 2.0 + screenshot of the curl `/api` loop cycling ≥3 hostnames on v2.0 **and** the portal showing version 2.0.

# Lab Guide: Cloud Native Workshop — One App, One Environment, End to End (GitHub Codespaces Edition)

**The session:** you will build ONE microservice and take it through the entire cloud native journey — containerize it, run it replicated on a real 2-node Kubernetes cluster, and deliver an update through a CI/CD pipeline into that same running cluster.

**The environment:** the whole hands-on session lives in **one GitHub Codespace** — a full Linux dev machine with VS Code, a terminal, and Docker preinstalled, running in your browser. In Lab 0 you add a real 2-node Kubernetes cluster to it with **kind** (Kubernetes-in-Docker). Because the codespace is created *from your own GitHub repo*, your code is already cloned and `git push` just works — no tokens to paste. **You install nothing on your computer.**

**Duration:** 70–80 minutes · **Cost:** $0 (well inside the free 120 core-hours/month every GitHub account includes)

> **Choose your lab environment** (same Labs 1 → 2 → 3 either way):
> - **This guide = GitHub Codespaces** — prep sheet: `prerequisites.md` (or `environments/codespaces/`)
> - **Killercoda option** — use `../killercoda/lab-guide.md` + `prerequisites.md` (or `../killercoda/`)
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

## Lab 0: Boot Your Environment (8 minutes — do once, keep it open all session)

1. **Create your GitHub repo — this is where your work lives AND where your lab machine boots from.**

   On **github.com**: **+ → New repository** → name it `hello-cncf` → **Public** → check **Add a README** → **Create repository**. (Lab 3's pipeline will run from this same repo.)

2. **Launch the codespace.** On your new repo's page: green **Code** button → **Codespaces** tab → **Create codespace on main**. In ~1 minute VS Code opens in your browser with a terminal at the bottom (if you don't see it: menu ☰ → **Terminal → New Terminal**). The default 2-core machine is plenty.

   Your repo is **already cloned and you're already inside it** — the prompt sits in `/workspaces/hello-cncf`. Git identity and push credentials are wired automatically by Codespaces. Verify both:

```bash
pwd && git remote -v
```

   Expected: `/workspaces/hello-cncf` and an `origin` pointing at your repo.

3. **Docker is preinstalled** — verify:

```bash
docker version --format 'Docker {{.Server.Version}} ready'
```

   Expected: `Docker 2x.x.x ready`

4. **Install the two Kubernetes tools** into the codespace (never onto your machine) — `kubectl` (the CLI) and `kind` (runs cluster nodes as Docker containers):

```bash
which kubectl || (curl -sLo /tmp/kubectl "https://dl.k8s.io/release/$(curl -sL https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && sudo install /tmp/kubectl /usr/local/bin/kubectl)
which kind || (curl -sLo /tmp/kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64 && sudo install /tmp/kind /usr/local/bin/kind)
kubectl version --client | head -1 && kind version
```

   Expected: a kubectl client version and a `kind v0.x.x` line.

5. **Create your 2-node cluster** (~1–2 minutes — one control-plane, one worker, exactly like the Killercoda playground):

```bash
cat > kind-config.yaml << 'EOF'
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
EOF
kind create cluster --name cncf --config kind-config.yaml
kubectl taint nodes cncf-control-plane node-role.kubernetes.io/control-plane- 2>/dev/null || true
kubectl get nodes
```

   Expected:

```
NAME                 STATUS   ROLES           AGE   VERSION
cncf-control-plane   Ready    control-plane   ...   v1.3x.x
cncf-worker          Ready    <none>          ...   v1.3x.x
```

   *(The `taint` line lets app pods schedule on both nodes — same behavior as the Killercoda playground, and it's what makes the "pods spread across nodes" moment in Lab 2 visible.)*

> ### 💾 THE GOLDEN RULE OF THIS WORKSHOP
> **Do all your work inside `/workspaces/hello-cncf`, and push after every lab.**
> Unlike Killercoda, your codespace **does not expire in 60 minutes** — it auto-stops after ~30 idle minutes but keeps every file. Pushing is still your safety net (and Lab 3's pipeline builds from the repo), so a **💾 Save your work** box ends each lab anyway.

> ### 🔄 CODESPACE STOPPED OR CLUSTER GONE? Recover in 3 minutes (bookmark this box)
> Reopen the codespace from https://github.com/codespaces (or the repo's **Code → Codespaces** tab). **All your files are still there** — nothing to re-clone. The kind cluster, however, may not survive a stop/restart (its node containers get new IPs). If `kubectl get nodes` errors or nodes are `NotReady`, rebuild it:
>
>     cd /workspaces/hello-cncf
>     kind delete cluster --name cncf
>     kind create cluster --name cncf --config kind-config.yaml
>     kubectl taint nodes cncf-control-plane node-role.kubernetes.io/control-plane- 2>/dev/null || true
>     kubectl apply -f app.yaml 2>/dev/null || echo "(no app.yaml yet — you were still in Lab 1)"
>     kubectl get pods
>
> If you'd reached Lab 2, your app is already redeploying — Kubernetes pulls the image straight from Docker Hub, so **nothing is rebuilt**. Need the Service again? Re-run Lab 2 step 3 (`kubectl expose ...` + capture `$URL`).

> ### 🖥️ NEW TERMINAL? Restore your variables (you'll need this in Labs 2–3)
> Shell variables like `$URL` live per-terminal. In any new terminal tab, paste:
>
>     cd /workspaces/hello-cncf
>     PORT=$(kubectl get svc hello-cncf -o jsonpath='{.spec.ports[0].nodePort}' 2>/dev/null)
>     NODE_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' cncf-control-plane)
>     URL="$NODE_IP:$PORT"; echo "App URL: $URL"
>     # Web portal (browser): kubectl port-forward svc/hello-cncf 8080:80  → PORTS → Open in Browser
---

## Lab 1: Build & Ship the Microservice (20 minutes)
*Same terminal. Objective: understand the app, containerize it, prove isolation, push to the registry.*

### Steps

> ### ⚠️ READ THIS ONCE — two ways to create files, both fine
> - **Paste-at-prompt:** blocks starting `cat > somefile << 'EOF'` and ending with a line containing only `EOF` are **shell commands** — paste the **whole block** at the terminal **`$` prompt** and press Enter. The first and last lines create the file and **must never appear inside it**.
> - **VS Code editor (you have a real one here!):** right-click the Explorer sidebar → **New File** → type the name → paste **only the contents between** the `cat` line and the `EOF` line → **Ctrl+S / Cmd+S**.
> - Either way, every file step is followed by a **✔ Verify** check that catches mistakes instantly, with a one-line fix.

1. **Create the application** — you're already inside `hello-cncf` from Lab 0. Paste the whole block at the `$` prompt (or use the editor):

```bash
cd /workspaces/hello-cncf
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

   Expected: `app.py` (plus `README.md` and `kind-config.yaml`)

   **✔ Verify** (catches the classic paste mistake in 2 seconds):

```bash
head -1 app.py && grep -c "cat > \|^EOF" app.py
```

   Expected: `import os, socket` then `0`. **🔧 If wrong** (you see `cat > app.py` or a count above 0): `rm app.py`, re-paste the block at the `$` prompt — or create it in the editor.

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

   Expected (last line): `Successfully tagged hello-cncf:1.0` (or `naming to docker.io/library/hello-cncf:1.0` with BuildKit)

4. **Run & test it (API + web portal):**

```bash
docker run -d -p 8080:8080 --name svc1 hello-cncf:1.0
curl -s localhost:8080/api
curl -s localhost:8080/healthz
```

   Expected: JSON with `"version":"1.0"` and `"served_by"`, then `ok`. **Remember `served_by`** — it becomes the star of Lab 2.

   **🌐 Open the web portal (Codespaces port forwarding):**
   1. Codespaces pops a toast **“Port 8080 is available”** — click **Open in Browser**, **or**
   2. Open the **PORTS** tab (next to TERMINAL) → find **8080** → right-click → **Open in Browser**.
   You should see the hello-cncf portal. Click **Refresh**.

5. **Prove isolation** — second copy, same host, zero conflict:

```bash
docker run -d -p 8081:8080 --name svc2 hello-cncf:1.0
curl -s localhost:8080/api | grep -o '"served_by":"[^"]*"'
curl -s localhost:8081/api | grep -o '"served_by":"[^"]*"'
```

   Expected: **two different hostnames**. Optional: in PORTS, open **8081** the same way for a second portal.

6. **Ship it to the registry** (the bridge to Labs 2 and 3):

```bash
docker login          # Docker Hub username + the access token from your prerequisites sheet
docker tag hello-cncf:1.0 <DOCKERUSER>/hello-cncf:1.0
docker push <DOCKERUSER>/hello-cncf:1.0
```

   Expected: ends with a `sha256:` digest. Browser-verify: `hub.docker.com/r/<DOCKERUSER>/hello-cncf` shows tag `1.0`.

### 💾 Save your work (30 seconds — do not skip)

```bash
cd /workspaces/hello-cncf
git add app.py requirements.txt Dockerfile kind-config.yaml
git commit -m "Lab 1: hello-cncf microservice + Dockerfile"
git push origin main
```

Expected: the output ends with `main -> main` — no password prompt, Codespaces authenticated you automatically. Refresh your repo on github.com — the files are there. **Your code now survives anything** (your image is already safe on Docker Hub).

### ✅ Checkpoint 1
You can explain: image vs container, layers, isolation, the registry hand-off, and opening the app as a web portal. Clean up the test containers: `docker rm -f svc1 svc2`

---

## Lab 2: Host & Manage It on Kubernetes (28 minutes)
*Same terminal, same codespace. Objective: replicate the app across 2 nodes and watch Kubernetes load-balance, probe, heal, and scale it.*

### Steps

1. **Declare the Deployment** — 3 replicas, health-probed:

```bash
cd /workspaces/hello-cncf && cat > app.yaml << 'EOF'
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

   Expected: 3 pods `1/1 Running`, placed across `cncf-control-plane` and `cncf-worker`. *(All on one node? You skipped the `taint` line in Lab 0 step 5 — run it now and `kubectl rollout restart deployment hello-cncf`.)*

3. **Service = built-in load balancer.** One Codespaces-specific wrinkle: your cluster's nodes are Docker containers, so the NodePort lives on the **node's container IP**, not on `localhost`. Capture both into one `$URL`:

```bash
kubectl expose deployment hello-cncf --port=80 --target-port=8080 --type=NodePort
PORT=$(kubectl get svc hello-cncf -o jsonpath='{.spec.ports[0].nodePort}')
NODE_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' cncf-control-plane)
URL="$NODE_IP:$PORT"
echo "Service reachable at $URL"
curl -s $URL/api
```

   Expected: JSON from inside the cluster. *(Open a new terminal later? The 🖥️ box in Lab 0 restores `$URL`.)*

   **🌐 Open the web portal via port-forward** (kind NodePorts are not visible to your laptop browser — Codespaces forwards *localhost* ports instead):

```bash
# leave this running in a split terminal (or background with &)
kubectl port-forward svc/hello-cncf 8080:80
```

   Then: **PORTS** tab → **8080** → **Open in Browser**. Refresh a few times — **served_by** changes as traffic hits different pods.
   *(Keep the port-forward terminal open for the rest of Lab 2 and Lab 3, or re-run that one line if it stops.)*

4. **⭐ SEE load balancing** — what `served_by` was built for:

```bash
for i in $(seq 1 8); do curl -s $URL/api | grep -o '"served_by":"[^"]*"'; done
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
for i in $(seq 1 8); do curl -s $URL/api | grep -o '"served_by":"[^"]*"'; done
kubectl scale deployment hello-cncf --replicas=3
```

   Expected: the loop cycles across **five** hostnames, then back to three.

7. **The declaration Kubernetes defends:**

```bash
kubectl get deployment hello-cncf -o yaml | grep " replicas:"
```

   Expected: `  replicas: 3` — the thermostat setting.

> **Fast rebuild box** (if the cluster ever breaks): the 🔄 box in Lab 0 rebuilds cluster + app in ~3 minutes; your image is safe on Docker Hub and your files never left the codespace.

### ✅ Checkpoint 2
You can explain — with evidence you produced — Deployments, Services & load balancing, readiness probes, self-healing, and declarative scaling.

### 💾 Save your work

```bash
cd /workspaces/hello-cncf
git add app.yaml
git commit -m "Lab 2: Kubernetes Deployment manifest with readiness probe"
git push origin main
```

Expected: ends with `main -> main`. Now your cluster manifest is recoverable anywhere — the 🔄 box re-applies it automatically.

**Keep this codespace open — Lab 3 continues right here.**

---


## Lab 3: CI/CD End to End — Commit to Cluster (30 minutes)
*Everything in one place this time: you edit, commit, and push from the codespace, GitHub Actions builds in the cloud, and you roll out in the same terminal. Objective: a code change travels commit → pipeline → registry → rolling update → visible to users.*

### One-time setup (2 min — your repo already exists and holds Labs 1–2)

Repo **Settings → Secrets and variables → Actions → New repository secret** — add both:

- `DOCKERHUB_USERNAME` = your Docker Hub username
- `DOCKERHUB_TOKEN` = the access token from your prerequisites sheet

### Steps

1. **Bump the app to v2.0 — right in your editor.** In the codespace Explorer, open `app.py` and change two things: `VERSION = os.getenv("APP_VERSION", "2.0")` and inside `payload()` set `message="Now delivered by a pipeline!"`. Then open `Dockerfile` and change `ENV APP_VERSION=1.0` to `ENV APP_VERSION=2.0`. **Ctrl+S / Cmd+S** on both. *(This is the everyday developer loop — no web pencil icon needed, though editing on github.com works too; just `git pull` afterwards.)*

2. **Add the quality gate** — create `test_app.py` (editor or paste-at-prompt):

```bash
cat > test_app.py << 'EOF'
from app import app

def test_root():
    r = app.test_client().get("/api")
    data = r.get_json()
    assert r.status_code == 200
    assert data["service"] == "hello-cncf"
    assert data["version"] == "2.0"

def test_healthz():
    assert app.test_client().get("/healthz").status_code == 200
EOF
```

3. **Declare the pipeline** — create the folder and file `.github/workflows/cicd.yml`:

```bash
mkdir -p .github/workflows
cat > .github/workflows/cicd.yml << 'EOF'
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
EOF
```

4. **Commit and push — this push triggers the pipeline:**

```bash
git add -A
git commit -m "v2.0: message change + test + CI/CD pipeline"
git push origin main
```

   Expected: ends with `main -> main`. **🔧 If the push is refused** with a message about `workflow` scope: create `.github/workflows/cicd.yml` via the github.com web editor instead (repo → **Add file → Create new file**, type the full path, paste the YAML, commit), then `git pull` in the codespace.

5. **Watch it go green.** Repo **Actions** tab → all steps ✓. Docker Hub now shows tag `2.0` beside `1.0` — the pipeline just did automatically everything you did by hand in Lab 1.

   **💾 Your work is already saved** — the pipeline runs *from* your repo, so the code is on GitHub by definition.

6. **⭐ Roll it out** — back in your codespace terminal:

```bash
kubectl set image deployment/hello-cncf web=<DOCKERUSER>/hello-cncf:2.0
kubectl rollout status deployment/hello-cncf
```

   Expected: `deployment "hello-cncf" successfully rolled out` — pods replaced one by one, zero downtime, gated by your readiness probe.

7. **⭐ Prove it to users:**

```bash
for i in $(seq 1 6); do curl -s $URL/api | grep -oE '"(version|served_by)":"[^"]*"' | paste -d' ' - -; done
```

   Expected: every answer `"version":"2.0"`, still load-balanced across fresh pods. *(`$URL` empty? Paste the 🖥️ box from Lab 0.)*

   **🌐 Portal check:** with `kubectl port-forward svc/hello-cncf 8080:80` running, refresh the forwarded **8080** browser tab — it should show **version 2.0**.

8. **Instant rollback:**

```bash
kubectl rollout undo deployment/hello-cncf
curl -s $URL/api | grep -o '"version":"[^"]*"'
```

   Expected: `"version":"1.0"`. Change is cheap in both directions.

9. **Correlate out loud:** commit (GitHub) → tested & built (Actions) → immutable versioned image (Docker Hub) → declarative rolling update (Kubernetes) → user-visible change (`version`) → one-command rollback. **One app, one environment, the entire loop — and you operated every stage.**

### 💾 Final state check

```bash
cd /workspaces/hello-cncf && git status && ls
```

Expected: `working tree clean` and `app.py  app.yaml  Dockerfile  kind-config.yaml  requirements.txt  test_app.py  README.md` (+ `.github/`) — your complete workshop artifact, permanently on GitHub. **Everything you built today is reproducible from this repo: open a fresh codespace on it, run Lab 0 steps 4–5, apply `app.yaml`, and the app runs.**

When you're done for the day, stop the codespace to conserve free hours: https://github.com/codespaces → **⋯ → Stop codespace** (deleting it is also safe — everything is pushed).

### ✅ Checkpoint 3
You can trace one change through every stage of the delivery loop and name the guarantee each stage provides.

---

## Troubleshooting (foolproof table)

| Symptom | Likely cause | Fix |
|---|---|---|
| Codespace won't start / quota message | Free core-hours exhausted (rare in one workshop) | github.com/codespaces → delete old codespaces; or Settings → Billing to check usage |
| `kubectl get nodes`: connection refused after reopening codespace | kind node containers got new IPs on restart | Use the **🔄 recovery box** in Lab 0 — delete + recreate cluster, re-apply `app.yaml` (~3 min) |
| `kind create cluster` fails / hangs | Docker daemon still warming up on a fresh codespace | Wait 20 s, `docker ps` to confirm Docker answers, re-run the kind line |
| All 3 pods on `cncf-worker` only | Control-plane taint not removed | Run the `kubectl taint nodes cncf-control-plane ...` line from Lab 0 step 5, then `kubectl rollout restart deployment hello-cncf` |
| `curl $URL` fails / `URL` is empty | New terminal — shell variables don't carry over | Paste the **🖥️ restore-variables box** from Lab 0 |
| `curl localhost:<nodeport>` gives connection refused | NodePorts live on the kind node's container IP, not localhost | Use `$URL` (node IP + port) as shown in Lab 2 step 3 |
| A file contains a `cat > ...` or `EOF` line | Block pasted inside an open editor tab instead of the terminal | `rm <file>`, re-paste the whole block at the `$` prompt — or create the file in the VS Code editor with only the contents |
| `git push`: `refusing to allow ... workflow scope` | Codespace token can't create workflow files in some org setups | Create `.github/workflows/cicd.yml` via the github.com web editor, then `git pull` |
| `git push`: `rejected — non-fast-forward` | You edited files on github.com AND in the codespace | `git pull --rebase origin main`, then push again |
| `denied: requested access to the resource is denied` on `docker push` | Tag missing your username, or not logged in | `docker login`, re-tag `<DOCKERUSER>/hello-cncf:1.0`, push again |
| Pods `ImagePullBackOff` | Username typo in app.yaml, or push skipped | `kubectl describe pod <name>` shows the pull error; fix the image ref, or use fallback `stefanprodan/podinfo` (port 9898) |
| Pods `Pending` | Over-scaled the small cluster | `kubectl scale deployment hello-cncf --replicas=3` |
| curl loop shows one hostname only | Other pods not Ready yet | `kubectl get pods` — wait for all `1/1`, retry |
| Browser shows raw JSON instead of portal | You opened `/api` | Open `/` (no path) for the portal; `/api` is for curl |
| PORTS tab has no 8080 / Open in Browser fails | Port-forward not running or wrong port | Re-run `kubectl port-forward svc/hello-cncf 8080:80`; wait for “Forwarding from …” |
| `port-forward` exits immediately | Service/pods not Ready | `kubectl get pods,svc`; re-run when pods are `1/1` |
| Portal still shows version 1.0 after Lab 3 | Browser cache or old pods | Hard-refresh; confirm `kubectl set image` finished; check `/api` with curl |
| Actions: `Username and password required` | Secrets missing/misnamed | Exactly `DOCKERHUB_USERNAME` / `DOCKERHUB_TOKEN` under Settings → Secrets → Actions |
| Actions: `insufficient_scope` on push | Docker Hub token is read-only | New token with **Read & Write** |
| Tests fail `ModuleNotFoundError: flask` | pip line missed | `Run tests` step must `pip install flask==3.0.3 pytest` |
| `kubectl set image`: container not found | Name mismatch | It's `web=` (container name from app.yaml) |
| Rollout stuck | v2.0 failing readiness | `kubectl describe pod <new-pod>`; `kubectl rollout undo` recovers instantly |

## Facilitator & reuse notes

- Recordings in `videos/` are **Codespaces-native walkthroughs** matching this guide (Lab 0 kind boot, `$URL/api`, **port-forward + Open in Browser** for the portal). Play a lab's video first, then release participants to reproduce it.
- **Killercoda twin package:** `../killercoda/` (guides + Killercoda walkthrough videos with `localhost` portal steps).
- **Choosing a track:** Codespaces gives persistence (no 60-minute cap, files survive stops), a real editor, and one fewer account — but consumes participant core-hours and needs the taint/`$URL`/port-forward wrinkles above. Killercoda gives an instant pre-built cluster with `localhost` NodePorts — but evaporates hourly. Both run the identical app, labs, and checkpoints, so you can even mix tracks in one room.
- The `served_by` + `version` + `/healthz` trio (portal at `/`, JSON at `/api`) is the reusable teaching pattern — steal it for any demo app.
- Assessment: fork the repo; grade = green pipeline + own Docker Hub tags 1.0 & 2.0 + screenshot of the curl `/api` loop cycling ≥3 hostnames on v2.0 **and** the portal showing version 2.0.
- **Optional pre-baked environment:** to skip Lab 0 steps 4–5 entirely, add a `.devcontainer/devcontainer.json` to the repo participants will open — Codespaces then pre-installs the tools on boot:

```json
{
  "image": "mcr.microsoft.com/devcontainers/universal:2",
  "features": {
    "ghcr.io/devcontainers/features/kubectl-helm-minikube:1": { "minikube": "none" },
    "ghcr.io/devcontainers-extra/features/kind:1": {}
  }
}
```

  Participants would still run the `kind create cluster` step (the cluster itself can't be pre-baked into the image).

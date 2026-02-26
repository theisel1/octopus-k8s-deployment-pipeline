# Octopus K8s Deployment Pipeline

This repo is a simple demo of CI/CD for `randomquotes-k8s`:
- GitHub stores source code
- GitHub Actions builds and pushes the container to GHCR
- Octopus Deploy creates/releases and deploys to Kubernetes
- Kubernetes runs locally (Docker Desktop Kubernetes or kind)

## Files in this repo

```text
.
├── .github/workflows/ci-cd.yml
└── k8s
    ├── namespace.yaml
    ├── deployment.yaml
    └── service.yaml
```

## Prerequisites

Install:
- Docker Desktop (or Docker + kind)
- `kubectl`
- `helm`
- GitHub repo with Actions enabled
- Octopus Cloud account

Quick checks:

```bash
docker --version
kubectl version --client
helm version
```

## Local Kubernetes

Use one option.

Option A: Docker Desktop Kubernetes
1. Open Docker Desktop.
2. Enable Kubernetes in Settings.
3. Wait for cluster health to show ready.

Option B: kind

```bash
kind create cluster --name randomquotes
kubectl cluster-info
```

## Octopus setup

### 1) Create account and API key
1. Sign up at https://octopus.com/start/cloud
2. Create or choose a space
3. Go to `Profile` -> `My API Keys` -> `New API Key`
4. Keep these values for GitHub secrets:
- `OCTOPUS_URL`
- `OCTOPUS_API_KEY`
- `OCTOPUS_SPACE`

### 2) Create environments
Create:
- `Dev`
- `Test`
- `Prod`

### 3) Add Kubernetes Agent target
1. In Octopus, open `Infrastructure` -> `Deployment Targets`.
2. Add a `Kubernetes Agent` target.
3. Run the Helm commands Octopus provides to register the agent in your local cluster.

### 4) Create project and process
1. Create project `randomquotes-k8s`.
2. Add step `Deploy Kubernetes YAML`.
3. Point it at:
- `k8s/namespace.yaml`
- `k8s/deployment.yaml`
- `k8s/service.yaml`
4. Select the Kubernetes Agent target.

### 5) Add project variables
Create:
- `RandomQuotes.Replicas`
- Scope by environment (example: `Dev=1`, `Test=2`, `Prod=3`)
- `RandomQuotes.Image.Tag`
- Mark as prompted

The workflow passes `GitHub.Owner` and `GitHub.Repo` at deploy time.

## GitHub setup

Add repository secrets:
- `OCTOPUS_URL`
- `OCTOPUS_API_KEY`
- `OCTOPUS_SPACE`

No separate GHCR secret is needed in this demo. The workflow uses `GITHUB_TOKEN`.

## Pipeline flow

On push to `main` (or manual run):
1. Build short SHA tag from commit (for example `a1b2c3d`).
2. Build container from `./src` using `./src/RandomQuotes.Web/Dockerfile`.
3. Push image to `ghcr.io/<owner>/<repo>:<short_sha>`.
4. Create Octopus release with the same short SHA.
5. Deploy that release to `Dev`.
6. Pass deployment variables:
- `RandomQuotes.Image.Tag`
- `GitHub.Owner`
- `GitHub.Repo`

This keeps commit, image tag, and release number aligned.

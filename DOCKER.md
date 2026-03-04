# Docker: build, tag, and push to Docker Hub

Quick reference for building the OpenClaw image and publishing it to Docker Hub. Run all commands from the **repository root**.

---

## Prerequisites

- Docker installed and running
- Docker Hub account ([hub.docker.com](https://hub.docker.com))
- Logged in: `docker login` (use your Docker Hub username and password or access token)

---

## 1. Build

Replace `YOUR_DOCKERHUB_USER` with your Docker Hub username (e.g. `tianclaassens`).

### Default image (gateway + CLI only)

```bash
docker build -t YOUR_DOCKERHUB_USER/openclaw:latest -f Dockerfile .
```

### Image with Homebrew, openai-whisper, himalaya, and ClawHub CLI

Adds Linuxbrew, `whisper`, `himalaya`, and `clawhub` for skills that need them:

```bash
docker build --build-arg OPENCLAW_INSTALL_BREW_CLI=1 -t YOUR_DOCKERHUB_USER/openclaw:latest -f Dockerfile .
```

Optional: use a descriptive tag instead of `latest` (e.g. for the brew variant):

```bash
docker build --build-arg OPENCLAW_INSTALL_BREW_CLI=1 -t YOUR_DOCKERHUB_USER/openclaw:brew-whisper-clawhub-himalaya -f Dockerfile .
```

---

## 2. Tag (optional)

Tag the same image with a version or alias so you can push multiple tags.

```bash
# Example: tag as version 2026.3.4
docker tag YOUR_DOCKERHUB_USER/openclaw:latest YOUR_DOCKERHUB_USER/openclaw:2026.3.4

# Example: tag as "brew" variant
docker tag YOUR_DOCKERHUB_USER/openclaw:brew-whisper-clawhub-himalaya YOUR_DOCKERHUB_USER/openclaw:brew
```

---

## 3. Push

Push the tags you want to publish.

```bash
docker push YOUR_DOCKERHUB_USER/openclaw:latest
```

If you created extra tags:

```bash
docker push YOUR_DOCKERHUB_USER/openclaw:2026.3.4
docker push YOUR_DOCKERHUB_USER/openclaw:brew
```

---

## One-liner (build + push)

Default image:

```bash
docker build -t YOUR_DOCKERHUB_USER/openclaw:latest -f Dockerfile . && docker push YOUR_DOCKERHUB_USER/openclaw:latest
```

With brew/whisper/himalaya/clawhub:

```bash
docker build --build-arg OPENCLAW_INSTALL_BREW_CLI=1 -t YOUR_DOCKERHUB_USER/openclaw:latest -f Dockerfile . && docker push YOUR_DOCKERHUB_USER/openclaw:latest
```

---

## Other build options

- **Docker CLI in image** (for sandbox):  
  `docker build --build-arg OPENCLAW_INSTALL_DOCKER_CLI=1 -t ...`
- **Chromium + Xvfb** (for browser automation):  
  `docker build --build-arg OPENCLAW_INSTALL_BROWSER=1 -t ...`
- **Extra apt packages**:  
  `docker build --build-arg OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg jq" -t ...`

Combine as needed, e.g.:

```bash
docker build \
  --build-arg OPENCLAW_INSTALL_BREW_CLI=1 \
  --build-arg OPENCLAW_INSTALL_DOCKER_CLI=1 \
  -t YOUR_DOCKERHUB_USER/openclaw:full \
  -f Dockerfile .
```

---

For running the image locally (Compose, env vars, bind mounts), see [docs/install/docker.md](docs/install/docker.md).

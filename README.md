# cyclonedx-python-test

Small Flask app used to try out **CycloneDX** SBOM generation for Python and a **GitHub Actions** pipeline that builds an SBOM, runs **Trivy**, and posts results on pull requests.

## What’s in the repo

- **`app.py`** — minimal Flask API (`GET /` returns `{"status": "ok"}`).
- **`requirements.txt`** — runtime dependencies (`flask`, `requests`, `numpy`).
- **`requirements-dev.txt`** — tooling (`cyclonedx-bom` for `cyclonedx-py`).
- **`.github/workflows/sbom.yml`** — installs deps, generates CycloneDX JSON/XML, uploads artifacts, runs Trivy (table + SARIF), comments on PRs, fails on CRITICAL/HIGH findings, attaches SBOMs to releases.

## Local setup

```bash
python3 -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
pip install -r requirements-dev.txt
```

## Run the app

```bash
python app.py
```

Then open [http://127.0.0.1:5000/](http://127.0.0.1:5000/).

## Generate an SBOM locally

With the dev dependencies installed and your venv active:

```bash
cyclonedx-py environment -o sbom.json
cyclonedx-py environment -o sbom.xml
```

This inventories the **current Python environment** (the packages installed in that venv), not only `requirements.txt`.

## CI behavior (summary)

On push, PR, and release, the workflow installs `requirements.txt` and `requirements-dev.txt`, runs `cyclonedx-py environment` to produce `sbom.json` and `sbom.xml`, uploads them as artifacts, scans the repo with Trivy, uploads SARIF to GitHub Security, and on PRs adds a comment with SBOM component count and Trivy output. The job fails if Trivy reports CRITICAL or HIGH issues in its table output.

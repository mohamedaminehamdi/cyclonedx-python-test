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
cyclonedx-py requirements requirements.txt -o sbom.json
cyclonedx-py requirements requirements.txt -o sbom.xml --of XML
```

That builds the SBOM from **declared runtime dependencies** in `requirements.txt` (and their resolved transitive deps), not from every package in the venv—so build tools like `cyclonedx-bom` stay out of the BOM.

To inventory *everything* installed in the active environment instead, use `cyclonedx-py environment` (not used in CI for this repo).

## CI behavior (summary)

On push, PR, and release, the workflow installs `requirements-dev.txt`, generates `sbom.json` / `sbom.xml` with `cyclonedx-py requirements requirements.txt`, then installs `requirements.txt` so Trivy can scan installed packages. It uploads SBOM artifacts, runs Trivy (table + SARIF), uploads SARIF to GitHub Security, and on PRs comments with the SBOM component count and Trivy output. The job fails if Trivy reports CRITICAL or HIGH issues in its table output.

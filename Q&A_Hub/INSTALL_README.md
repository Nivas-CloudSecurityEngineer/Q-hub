# 🛠️ Install & Deployment Guide — Q&A Hub MkDocs Site

This guide walks you through running the **AWS & DevOps Interview Q&A Hub** as a searchable documentation website using **MkDocs (Material theme)**, previewing it locally with **Docker**, and publishing it for free on **GitHub Pages**.

---

## 📁 What Was Added

| File/Folder | Purpose |
|---|---|
| `docs/` | All 28 topic folders + `index.md` (site content, moved here from repo root) |
| `mkdocs.yml` | Site config — theme, nav menu, plugins |
| `requirements.txt` | Python packages needed to build the site |
| `Dockerfile` | Builds a container image that runs MkDocs |
| `docker-compose.yml` | One command to preview the site locally at `http://localhost:8000` |
| `.github/workflows/deploy.yml` | GitHub Actions workflow — auto-builds & publishes the site to the `gh-pages` branch on every push to `main` |
| `.gitignore` | Ignores the generated `site/` build output, venvs, etc. |

---

## ✅ Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed (for the local Docker workflow)
- A [GitHub](https://github.com) account and a repository to push this project to
- Git installed locally

---

## 1️⃣ Option A — Preview Locally with Docker (recommended)

From the project root (where `docker-compose.yml` lives):

```bash
docker compose up --build
```

- The site will be available at **http://localhost:8000**
- Editing any `.md` file under `docs/` auto-reloads the browser (live reload via bind mount)
- Stop it with `Ctrl+C`, or run in the background with `docker compose up -d --build`

To build the **static site** (the `site/` folder that gets deployed) using Docker instead of just serving it:

```bash
docker compose run --rm mkdocs mkdocs build
```

To stop and remove the container:

```bash
docker compose down
```

---

## 2️⃣ Option B — Preview Locally without Docker (Python)

```bash
python3 -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
mkdocs serve
```

Open **http://localhost:8000** in your browser.

---

## 3️⃣ Push the Project to GitHub

If this folder isn't a Git repository yet:

```bash
git init
git add .
git commit -m "Add MkDocs site with Docker setup"
git branch -M main
git remote add origin https://github.com/<your-username>/<your-repo>.git
git push -u origin main
```

> Replace `<your-username>/<your-repo>` with your actual GitHub username and repository name.

---

## 4️⃣ Update the Site URL in `mkdocs.yml`

Before deploying, edit these three lines at the top of [`mkdocs.yml`](mkdocs.yml) with your real GitHub username/repo:

```yaml
site_url: https://<your-github-username>.github.io/<your-repo-name>/
repo_url: https://github.com/<your-github-username>/<your-repo-name>
repo_name: <your-github-username>/<your-repo-name>
```

Commit and push this change.

---

## 5️⃣ Deploy to GitHub Pages (Automatic via GitHub Actions)

A workflow at [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml) is already set up to:

1. Trigger on every `push` to `main`
2. Install MkDocs + Material theme
3. Run `mkdocs gh-deploy --force`, which builds the site and pushes it to a `gh-pages` branch

**One-time step — enable Pages for that branch:**

1. Go to your repo on GitHub → **Settings** → **Pages**
2. Under **Build and deployment** → **Source**, choose **Deploy from a branch**
3. Under **Branch**, select `gh-pages` / `(root)` → **Save**
4. Push any commit to `main` (or re-run the workflow from the **Actions** tab)
5. After the workflow finishes, your site will be live at:
   `https://<your-github-username>.github.io/<your-repo-name>/`

You can watch progress under the **Actions** tab of your GitHub repository.

---

## 6️⃣ Manual Deployment (Alternative, No CI)

If you prefer to deploy directly from your machine instead of using GitHub Actions:

```bash
pip install -r requirements.txt
mkdocs gh-deploy --force
```

This builds the site and force-pushes it to the `gh-pages` branch of the remote configured in your local Git repo. Then enable GitHub Pages for the `gh-pages` branch as described in step 5.

---

## 🔍 Troubleshooting

| Issue | Fix |
|---|---|
| Port `8000` already in use | Change the port mapping in `docker-compose.yml`, e.g. `"8080:8000"` |
| Mermaid diagrams not rendering | Make sure you're using `mkdocs-material` (already pinned in `requirements.txt`) — Mermaid support is built into the `pymdownx.superfences` config in `mkdocs.yml` |
| 404 on GitHub Pages | Confirm the Pages source branch is `gh-pages`, not `main`, and that `site_url` in `mkdocs.yml` matches your repo name exactly |
| Workflow fails on push | Check the **Actions** tab logs; ensure `permissions: contents: write` is present in `deploy.yml` (already included) |

---

## 🔁 Everyday Workflow

1. Add/edit questions in the relevant `docs/<Topic>/*.md` file
2. Preview locally: `docker compose up --build`
3. Commit & push to `main`
4. GitHub Actions automatically rebuilds and republishes the live site

# Categorical Stability Explorer — Setup & Deployment

## How it works

```
Your machine                    GitHub Pages (free hosting)
────────────────                ────────────────────────────
export_data.py   →   site/      →   public website
  (runs once)      (static files)
```

Everything is precomputed. The website is just HTML + JSON — no server, no database.

---

## Step 1 — Precompute the data

Edit the CONFIG section at the top of `export_data.py`:

```python
TREEBANKS_FOLDER   = "/path/to/your/UD_treebanks_folder"
PATTERNS_TEXT_FILE = "/path/to/patterns_all_nodes.txt"
OUTPUT_DIR         = "./site/data"   # leave this as-is
```

Then run:

```bash
python export_data.py
```

This will create `site/data/index.json` and one folder per treebank:

```
site/
  index.html
  treebank.html
  data/
    index.json
    French-GSD/
      stability.json
      intruder.json
      bridge_pairs.json
    English-EWT/
      ...
```

Preview locally before deploying:

```bash
python -m http.server 8000 --directory site
# then open http://localhost:8000
```

---

## Step 2 — Deploy to GitHub Pages

### Option A — Simplest (recommended)

1. Create a new **public** GitHub repository, e.g. `ud-stability-explorer`
2. Copy the entire `site/` folder contents into the repo root
   (so `index.html` is at the root of the repo, and `data/` is a subfolder)
3. Push to GitHub
4. Go to **Settings → Pages → Source**: choose `main` branch, `/ (root)`
5. Click Save — your site will be live at `https://<your-username>.github.io/ud-stability-explorer/`

### Option B — Keep source code and site in one repo

Put your Python scripts in the root of the repo and the website in a `docs/` subfolder:

```
my-repo/
  export_data.py
  should_split.py
  docs/             ← rename "site/" to "docs/"
    index.html
    treebank.html
    data/
      ...
```

Then in GitHub Pages settings, choose **Source: `main` branch, `/docs` folder**.

### How to update the data

Re-run `export_data.py` and push the updated `data/` files. The website updates automatically.

---

## File size guide

Each treebank produces roughly:
- `stability.json`   ~5–20 KB
- `intruder.json`    ~5–20 KB  
- `bridge_pairs.json`  ~50–300 KB (depends on how many bridge pairs exist)

For 60 treebanks that's typically **20–50 MB total** — well within GitHub's free limits.

If `bridge_pairs.json` files are too large, increase `BRIDGE_THRESHOLD` in `export_data.py`
(e.g. `0.10`) to only export pairs with stronger overlap.

---

## Customising the website

Both HTML files are self-contained. The main things you might want to change:

- **`index.html`** header text — describes the project to visitors
- **`treebank.html`** feature table — currently shows `unit_value`, `rival_avg`, `diff`
- **Colors** — `YlOrRd` colourmap is used throughout; change `colorscale` in the Plotly calls
- **`BRIDGE_THRESHOLD`** in `export_data.py` — controls which pairs appear in the explorer

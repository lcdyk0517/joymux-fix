# ⚠️ Compatibility

This action is **only compatible with _arkos4clone_ builds dated **September 17, 2025** or later**.  
Please update if you’re on an older image.

---

# Patch ONE `odroidgo3-joypad` DTB (GitHub Actions)

Patch **a single DTB** in your own **fork** using GitHub Actions. The workflow:
- fixes the **AMUX** mapping based on what you *currently feel* on the sticks,
- **toggles** per-axis invert flags,
- updates 8 **tuning** values,
- and outputs **one** `.dtb` file whose **filename matches your input**.

> Workflow file: `.github/workflows/patch-one-odroidgo3.yml`  
> Python patch script: `tools/patch_og3_joypad.py`

---

## Why a fork?

Running in **your fork** keeps your DTB private and avoids sharing tokens or files with anyone else. All processing happens on GitHub’s runner in your own repo; the finished file is delivered as a private **Actions artifact** in your fork.

---

## Quick start (TL;DR)

1. **Fork** this repository.  
2. In your fork, **enable Actions** (Actions tab → “Enable workflows”).  
3. **Commit your DTB** to the fork (any path is fine).  
4. Go to **Actions → Patch ONE odroidgo3-joypad DTB → Run workflow**.  
5. Fill the inputs (see below) and run.  
6. When it finishes, open the run page and download the **artifact** (a zip containing only your patched `.dtb`, same filename as input).

> Default artifact retention is **1 day**. You can change `retention-days` inside the workflow if needed.

---

## Inputs (what to fill)

### 1) `dtb_path` (required)
**Path to your DTB inside the repository.** This must match the committed file exactly (case-sensitive).

- Examples:
  - `rk3326-xf36pro-linux.dtb` (file at repo root)
  - `dtbs/og3/odroidgo3.dtb` (file in a subfolder)

If the workflow says “DTB not found”, double-check the path and file name in your fork.

---

### 2) `amux_map` (recommended)
**Tell the action what you *currently feel*** at the four positions `[LX, LY, RX, RY]`.  
The script uses your observation to **reassign the existing four values to the correct mapping** and writes it back.

Accepted values:
- Any permutation of `LX, LY, RX, RY` (commas/spaces/full‑width separators OK; case‑insensitive)
- `unchanged` (do not modify AMUX)

Common recipes:
- `LX,LY,RX,RY` — everything *feels correct* → mapping kept as is.
- `LX,LY,RY,RX` — *right stick X/Y feel swapped* → script swaps the right half.
- `RX,RY,LX,LY` — *left stick ↔ right stick feel swapped* (whole sticks swapped).

> Intuition: you describe “what you *feel* at each position,” the script rebuilds the canonical mapping and writes it back to the DTB.

---

### 3) `invert_axes` (optional)
**Toggle (flip) these axes’ invert flags**. Use a CSV of any of: `lx`, `ly`, `rx`, `ry` (aliases `absx`, `absy`, `absrx`, `absry` also accepted).  
- If a listed invert flag already exists, it will be **removed**.  
- If it does not exist, it will be **added**.  
- Axes **not listed** remain unchanged.  
- Use `unchanged` to skip.

Examples:
- `unchanged` — don’t touch any inverts.
- `ly` — only toggle left‑Y.
- `lx,rx` — toggle X of both sticks.

---

### 4) `tuning` (optional)
**Dead‑zone/curve tunings**. Provide either:
- a **single integer** (applied to all 8 entries), default `200`, or
- **8 integers** (CSV) in this order: `x+, x-, y+, y-, rx+, rx-, ry+, ry-`.

Numbers are written into the DTS as hex cells (e.g., `200` → `<0xC8>`). Range: `0 … 4294967295`.

Examples:
- `200`
- `180,180,200,200,220,220,200,200`

---

## Output

- The action uploads **one** file: your patched **`.dtb`**, with the **same filename** as the input’s basename.  
  - Example: input `dtbs/og3/odroidgo3.dtb` → artifact contains `odroidgo3.dtb`.
- GitHub serves artifacts as zip containers; this zip contains **only that one file**.
- Default artifact retention: **1 day** (edit in the workflow if you need longer/shorter).

---

## Step‑by‑step (detailed)

1) **Fork**  
Click **Fork** in the top‑right of this repo to create your copy.

2) **Enable Actions in your fork**  
Open the **Actions** tab in the fork. If prompted, click **Enable workflows**.

3) **Add your DTB**  
Put your `.dtb` file somewhere in the repo (commit/push). Remember the **relative path**.

4) **Run the workflow**  
Actions → **Patch ONE odroidgo3-joypad DTB** → **Run workflow**.  
Fill:
- **`dtb_path`** — the exact path you committed (e.g., `rk3326-xf36pro-linux.dtb`).  
- **`amux_map`** — your observed order for `[LX,LY,RX,RY]` (or `unchanged`).  
- **`invert_axes`** — CSV like `ly` / `lx,rx` / `unchanged`.  
- **`tuning`** — `200` or 8 CSV numbers.

5) **Download**  
Open the run page; in the **Artifacts** box, download the zip (it contains just your patched `.dtb`).

---

## Examples

**Right stick X/Y feel swapped**  
- `amux_map`: `LX,LY,RY,RX`  
- `invert_axes`: `unchanged`  
- `tuning`: `200`

**Whole sticks feel swapped (left ↔ right)**  
- `amux_map`: `RX,RY,LX,LY`

**Only make RY feel inverted**  
- `amux_map`: `unchanged`  
- `invert_axes`: `ry`

**Uniform tuning**  
- `tuning`: `210` (applied to all 8 tuning entries)

---

## Troubleshooting

- **“DTB not found”**  
  Check that `dtb_path` matches a committed file in your **fork** (case‑sensitive). The job summary lists detected `.dtb` files to help you pick a valid path.

- **`dtc` / device‑tree‑compiler errors**  
  The workflow installs `device-tree-compiler`. If the install step failed, re‑run the job. If `dtc` parsing fails (e.g., “Properties must precede subnodes”), please share logs in an issue.

- **AMUX format errors**  
  `amux_map` must be a permutation of `LX,LY,RX,RY` or `unchanged`. Separators and case are flexible.

- **No artifact?**  
  Ensure your fork’s Actions are enabled and you ran the workflow from the **fork**, not the upstream repo. Confirm `tools/patch_og3_joypad.py` exists in your fork (the job checks this).

---

## Security & privacy

- Workflow permissions are `contents: read`. It does **not** push to your repo or publish releases.  
- Your DTB is processed on GitHub’s runner within your fork.  
- The only output is a private Actions artifact in **your** fork.  
- No external services or secrets are required.

---

## Customization (optional)

- **Artifact retention**: change `retention-days` in the “Upload patched DTB only” step (`1` by default).  
- **Add more outputs**: if you also want `before.dts`, `after.dts`, or a diff, adjust the workflow to keep and upload those files.  
- **Stricter AMUX validation**: you can restrict `amux_map` to a few common patterns in `tools/patch_og3_joypad.py` if you prefer failing fast on unusual permutations.

# ❤️ **Support the Project**

If you find ArkOS4Clone helpful and want to support future development:\ 👉 **https://ko-fi.com/lcdyk**

Every donation helps testing new devices, improving compatibility, and speeding up development.\ Thank you for your support! 🙏

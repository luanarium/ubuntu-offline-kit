Archiving system for APT and Python packages for offline use.

# Ubuntu-offline-kit

[See apt_archive.md](apt_archive.md) for automatic APT archiving.

Archiving python libraries themselves via pip, per project or per one long requirements.txt file for UV, as UV creates a global content-addressed cache so you only need to download the collection once.

(Archiving via UV)[archiving-python-libraries-via-uv]
(Archiving via PIP)[archiving-via-pip]

## Archiving via UV

This uses [UV project manager](https://github.com/astral-sh/uv) and is the recommended method.

For portable, reproducible environments (air-gapped or otherwise):

Build an explicit wheelhouse a project:

```bash
uv export --format requirements-txt | uv pip download -r /dev/stdin -d ./wheelhouse
```

Install fully offline later, elsewhere:

```bash
uv pip install --no-index --find-links ./wheelhouse -r requirements.txt
```

Confirm a backup is complete (fails loudly if anything's missing):

```bash
uv sync --offline
```

Keep `./wheelhouse` alongside `uv.lock` — the lockfile pins exact versions/hashes, so the pair gives you a fully reproducible, offline-installable environment rather than relying on `uv`'s global cache (`uv cache dir`), which mixes in packages from unrelated projects.

### Moving the backup to removable media

A wheelhouse is just a flat folder of `.whl`/`.tar.gz` files — no database, no symlinks, no baked-in absolute paths — so it copies cleanly to a USB drive or external disk.

Archive it as one file:

```bash
tar czf wheelhouse-backup.tar.gz wheelhouse uv.lock pyproject.toml
```

Copy to mounted removable media:

```bash
cp wheelhouse-backup.tar.gz /media/your-usb/
```

**Restoring on another machine, fully offline:**

```bash
tar xzf wheelhouse-backup.tar.gz
```

```bash
uv sync --offline --find-links ./wheelhouse
```

Notes:

- **Architecture matters.** Wheels are often platform-specific (e.g. `manylinux_x86_64` vs `macosx_arm64`). If the drive needs to serve different machines, download with `--python-platform`/`--python-version` flags on `uv pip download` so the wheelhouse contains wheels for the *target* machine, not just the one that built it. A single-architecture homelab (all x86_64 Linux) doesn't need to worry about this.
- **Always include `uv.lock`.** Without it, the wheelhouse is just "some package files," not a reproducible environment — the lockfile is what tells `uv` exactly which versions/hashes to expect.
- **Check size before copying** with `du -sh ./wheelhouse`; export with `--no-dev` if you only need runtime dependencies and want to trim dev/test tooling from the backup.

## Archiving via pip

**1. Create virtual environment**

```bash
python3 -m venv ~/offline-env
source ~/offline-env/bin/activate
```

**2. Create `requirements.txt`**

```
flask
requests>=2.30
numpy==1.26.4
```

**3. Download packages + dependencies into archive**

```bash
mkdir -p ~/offline-env/pip-archive
pip download -r requirements.txt -d ~/offline-env/pip-archive --only-binary=:all:
```

**4. Install packages from local archive**

```bash
pip install --no-index --find-links ~/offline-env/pip-archive -r requirements.txt
```

**5. (Optional) Serve archive to LAN clients**

```bash
cd ~/offline-env/pip-archive
python3 -m http.server 8080
```

Clients:

```bash
pip install --no-index --find-links http://<server-ip>:8080 -r requirements.txt
```

**6. Make the environment relocatable**
- Do this after the environment is finalized and ready to be shared.

```bash
pip install virtualenv
virtualenv --relocatable ~/offline-env
```

---

⚠️ **Caveat:**  
Pure-Python wheels are fully portable. Packages with **C extensions or compiled libraries** (e.g. `numpy`, `pandas`, `lxml`) depend on the target system’s ABI and architecture. These must match (e.g., same Ubuntu version and Python build) for the environment to remain functional offline.

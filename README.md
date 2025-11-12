# Ubuntu-offline-kit

[See apt_archive.md](apt_archive.md)

## Archive pip Packages

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

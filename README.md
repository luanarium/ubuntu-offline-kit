# 🖥️ Offline Ubuntu + Python Package Archive

---

This guide explains how to create a portable, offline-ready archive of Ubuntu system packages and Python libraries. It shows how to preserve APT caches to build a local repository of `.deb` packages, and how to download Python packages using `pip` with a `requirements.txt` file. Using these archives, you can reinstall software or set up new systems without Internet access, either directly from local storage or over a LAN/USB.
--- 

## 1. Persist APT Cache

```bash
echo 'APT::Clean-Installed "false";' | sudo tee /etc/apt/apt.conf.d/02noclean
```

Install packages normally — cached `.deb`s stay in `/var/cache/apt/archives/`.

---

## 2. Archive APT Packages

```bash
mkdir -p /srv/myrepo
cp /var/cache/apt/archives/*.deb /srv/myrepo/
cd /srv/myrepo
dpkg-scanpackages . /dev/null | gzip -9c > Packages.gz
```

Offline use:

```
deb [trusted=yes] file:///srv/myrepo ./
sudo apt update
sudo apt install vim htop
```

---

## 3. Archive pip Packages

Create `requirements.txt` listing desired packages, e.g.:

```text
flask
requests>=2.30
numpy==1.26.4
```

Or from an existing environment:

```bash
pip freeze > requirements.txt
```

Download packages:

```bash
mkdir ~/pip-archive
pip download -r requirements.txt -d ~/pip-archive
```

Offline install:

```bash
pip install --no-index --find-links ~/pip-archive -r requirements.txt
```

Optional LAN server:

```bash
python3 -m http.server 8080 --directory ~/pip-archive
```

Use `--find-links http://server-ip:8080` on clients.

---

**Result:** fully offline-ready APT + pip repository for system + Python packages.

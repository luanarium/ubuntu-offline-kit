# Personal Ubuntu repository for offline use

## 1. Create your directories
```bash
sudo mkdir /opt
sudo mkdir /opt/myrepo
```

## 2. Download packages and move to /opt/myrepo
```bash
sudo apt-get install [package] --download-only
sudo cp -u /var/cache/apt/archives/*.deb /opt/myrepo/
```

## 3. Build indexes for your repo. Packages and Packages.gz must be capitalized
```bash
sudo dpkg-scanpackages . /dev/null | sudo tee Packages > /dev/null
sudo dpkg-scanpackages . /dev/null | gzip -9c | sudo tee Packages.gz > /dev/null
```

## 4. If your Packages file is empty your repo will not work, check with:
```bash
cat Packages
```

## 5. Clear /opt/myrepo if you need to start fresh
```bash
sudo rm -rf /opt/myrepo/*
```

## 6. Ensure permissions are correct. Do this for Packges and Packages.gz each time you rebuild them
```bash
sudo chmod a+r /opt/myrepo/*.deb
sudo chmod a+r /opt/myrepo/Packages /opt/myrepo/Packages.gz
sudo chown -R root:root /opt/myrepo
sudo chmod -R a+rX /opt/myrepo
```

## 7. Add repo dir to sources.list and apt update
```bash
echo "deb [trusted=yes] file:///opt/myrepo ./" | sudo tee /etc/apt/sources.list.d/myrepo.list
sudo apt update
```

## 8. Create gzip archive for deployment. run in the root of your project directory
```bash
tar czf myrepo_$(date +%Y%m%d).tar.gz -C / opt/myrepo
```

## 9. Put your gzip archive into a project directory (e.g. project) with any other files you'd like accessible in your offline machine. Then build your iso
```bash
genisoimage -o myrepo_$(date +%Y%m%d).iso -R -J project
```

## 10. On your target machine, create your directories, extract your archive, set permissions, add your repo, then apt update
```bash
sudo mkdir /opt
sudo mkdir /opt/myrepo
sudo tar xzf myrepo_*.tar.gz -C / # run this in the root of your cdrom or mounted iso.
sudo chmod a+r /opt/myrepo/*.deb
sudo chmod a+r /opt/myrepo/Packages /opt/myrepo/Packages.gz
sudo chown -R root:root /opt/myrepo
sudo chmod -R a+rX /opt/myrepo
echo "deb [trusted=yes] file:///opt/myrepo ./" | sudo tee /etc/apt/sources.list.d/myrepo.list
sudo apt update
```

## Notes
I encountered a problem with two dependencies for VLC and discovered the issue was that I had both the amd64 and i368 versions of these packages. The solution was to delete the i368 versions and keep only the packages for amd64, as my architecture is amd64. Then rebuild my index.

## Aliases: Add these to your ~/.bashrc
```bash
alias rebuildrepo='sudo dpkg-scanpackages . /dev/null | sudo tee Packages > /dev/null && sudo dpkg-scanpackages . /dev/null | gzip -9c | sudo tee Packages.gz > /dev/null'
alias upgrades='sudo apt-get --download-only upgrade -y' # download packages for first update after installing ubuntu
alias packiso='genisoimage -o myrepo_$(date +%Y%m%d).iso -R -J project' # make iso containing myrepo.gz
alias pack='sudo cp -u /var/cache/apt/archives/*.deb /opt/myrepo/'
alias offline='sudo apt-get install --download-only'
```
## This alias checks if 'Packages' and 'Packages.gz' files exist, have correct case, Packages isn't empty, and checks for correct permisisons.

```bash
alias checkrepo='[ -d /opt/myrepo ] || { echo "⚠️ /opt/myrepo missing"; exit 1; };
find /opt/myrepo \( -type d ! -perm -a+rx -o -type f ! -perm -a+r -o ! -user root -o ! -group root \) | grep . && echo "⚠️ permission or ownership issue(s)";
[ ! -f /opt/myrepo/Packages ] && echo "⚠️ Packages missing";
[ ! -f /opt/myrepo/Packages.gz ] && echo "⚠️ Packages.gz missing";
[ -f /opt/myrepo/Packages ] && [ ! -s /opt/myrepo/Packages ] && echo "⚠️ Packages empty"'
```

## Usage examples:

## rebuild package index in /opt/myrepo
```bash
cd /opt/myrepo
rebuildrepo
```

## download all upgrade packages for caching
```bash
upgrades
```

## create ISO with repo archive
```bash
packiso
```

## copy newly downloaded .deb files into repo
```bash
pack
```

## download specific package(s) without installing
```bash
offline vlc
offline curl wget
```

## Here's a brief walkthrough of adding code to `.bashrc` via nano:

1. Open nano editor by running:

```bash
nano ~/.bashrc
````

2. Scroll to the bottom of the file.

3. Add your new code on a new line, for example:

```bash
# New addition
echo "Hello World"
```

4. Save and exit nano by pressing **Ctrl+X**, then **Y**, then **Enter**.

5. Reload `.bashrc` to apply changes:

```bash
source ~/.bashrc
```

**Key points:**

* `.bashrc` is a hidden file in your home directory (`~/.bashrc`)
* Use `nano ~/.bashrc` to open/edit it
* Add your new code at the end of the file
* Save with **Ctrl+X**, then **Y**, then **Enter**
* Reload with `source ~/.bashrc`

It's recommended to backup `.bashrc` before making major changes:

```bash
cp ~/.bashrc ~/.bashrc.bak
```

This creates a copy named `.bashrc.bak` in the same directory.

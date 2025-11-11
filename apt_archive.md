1. Create the repo folder

sudo mkdir -p /opt/myrepo
sudo chown root:root /opt/myrepo
sudo chmod a+rX /opt/myrepo


---

2a. Download .debs for currently installed packages (optional)

> Optional because default packages are already included in the Ubuntu live ISO.



dpkg --get-selections | awk '{print $1}' | xargs sudo apt-get install --download-only -y


---

2b. Download .debs for available upgrades

> Ensures your offline repo includes post-install updates.



sudo apt update
sudo apt-get --download-only upgrade -y


---

2c. Download .debs for additional single packages

> Use this for packages not installed by default or upgrades you want pre-cached.



sudo apt-get install --download-only <package-name>


---

3. Copy .debs to repo folder

sudo cp -u /var/cache/apt/archives/*.deb /opt/myrepo/
sudo chmod a+r /opt/myrepo/*.deb


---

4. Build Packages and Packages.gz indexes (with tee)

cd /opt/myrepo

# Uncompressed Packages
sudo dpkg-scanpackages . /dev/null | sudo tee Packages > /dev/null

# Compressed Packages.gz
sudo dpkg-scanpackages . /dev/null | gzip -9c | sudo tee Packages.gz > /dev/null

# Ensure readable by _apt
sudo chmod a+r Packages Packages.gz

> Important: filenames must be exactly Packages and Packages.gz.




---

5. Add the local repo to APT

echo "deb [trusted=yes] file:///opt/myrepo ./" | sudo tee /etc/apt/sources.list.d/myrepo.list
sudo apt update


---

6. Test offline install and upgrade

sudo apt install vim
sudo apt upgrade


---

✅ Notes

Step 2a is optional if you already have the ISO-provided default packages.

/opt/myrepo avoids _apt permission issues.

Using tee prevents “Permission denied” errors when writing Packages and Packages.gz.

Steps 2b–2c ensure upgrades and additional packages are available offline.

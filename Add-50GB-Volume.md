# Moodle on Ubuntu (EC2) — Add 50GB EBS Volume & Move Data
Complete, step-by-step `README.md` to attach a 50GB EBS volume, mount it at `/mnt/data`, move Moodle web files and `moodledata`, make the mount persistent, and optionally move MySQL data. Copy-paste this file into your GitHub repo.

---

## Summary
This guide helps you free up a full root volume by moving Moodle application files and user data to a new 50GB EBS volume mounted at `/mnt/data`. It includes safe backups, verification steps, permission fixes, fstab persistence, optional MySQL data migration, cleanup tips and rollback steps.

**Assumptions**
- Ubuntu on AWS EC2 (you have console access).
- You have `sudo` privileges on the instance.
- Web server: Apache or Nginx + PHP (adjust service names if different).
- New EBS device expected at `/dev/xvdf` (or similar).

**Important:** Take backups before starting. Expect a short maintenance window while files are moved.

---

## 0. (Strongly Recommended) Create Snapshots/Backups

Before changes, create at least a DB and web files backup.
```bash
# Check disk usage
df -h

# List block devices
lsblk

# Backup web files quickly to the new disk (if already mounted)
sudo mkdir -p /mnt/data/backups
sudo tar -C /var/www/html -czf /mnt/data/backups/moodle-www-$(date +%F).tar.gz moodle

# Backup DB (adjust user/password/db)
mysqldump -u root -p YOUR_MARIADB_ROOT_PASSWORD moodle > /mnt/data/backups/moodle-db-$(date +%F).sql
```
Also consider an EBS snapshot of the root volume from the AWS Console.

## 1. Create & Attach a 50 GB EBS Volume (AWS Console)

- EC2 → Elastic Block Store → Volumes → Create volume

- Size: 50 GiB, Type: gp3 (or gp2), AZ must match your instance (e.g., ap-south-1a).

- Create → Select the volume → Actions → Attach volume → Choose your instance.

- Device name: /dev/sdf (Linux will map it as /dev/xvdf). Attach.

Alternative device names for data disks: /dev/sd[f-p]. On some Nitro instances, it may appear as /dev/nvme1n1 instead of /dev/xvdf.

## 2. Verify Disk on the Instance
```bash
lsblk -o NAME,MAJ:MIN,RM,SIZE,RO,TYPE,MOUNTPOINTS
```
You should see a new 50G disk (e.g., xvdf).

## 3. Format the New Volume (ext4)

Skip if you’ve already formatted it.
```bash
sudo mkfs.ext4 /dev/xvdf
```
## 4. Create Mount Point & Mount

If root was 100% full and mkdir failed earlier, you can temporarily mount to /media, clean some space, then proceed.
```bash
# Create a dedicated mount point
sudo mkdir -p /mnt/data


# Mount the volume
sudo mount /dev/xvdf /mnt/data


# Verify
df -h | grep xvdf
```
Expected: /dev/xvdf 50G … /mnt/data

## 5. Make Mount Persistent (fstab)

Find the UUID and add it to /etc/fstab so it mounts on reboot.
```bash
sudo blkid /dev/xvdf
# Example output: /dev/xvdf: UUID="12345678-90ab-cdef-1234-567890abcdef" TYPE="ext4"


sudo nano /etc/fstab
```
Append a line (replace with your actual UUID):
```bash
UUID=12345678-90ab-cdef-1234-567890abcdef /mnt/data ext4 defaults,nofail,x-systemd.device-timeout=1min 0 2
```
Test the fstab entry:
```bash
sudo mount -a
```

## 6. Stop Web Services (Short Downtime)
```bash
# Apache
sudo systemctl stop apache2
# or Nginx (if you use Nginx as the web server)
sudo systemctl stop nginx
```

## 7. Move Moodle Web Code to the New Disk

Default path is often /var/www/html/moodle. Adjust if your path differs.

Simple move (works without extra packages)
```bash
sudo mv /var/www/html/moodle /mnt/data/moodle
```
Create a symlink so the original path still works:
```bash
sudo ln -s /mnt/data/moodle /var/www/html/moodle
```
Fix ownership and permissions (typical owner is www-data):
```bash
sudo chown -R www-data:www-data /mnt/data/moodle
```
Apache FollowSymLinks: Ensure the site vhost allows following symlinks (usually default). If restricted, add Options FollowSymLinks in the relevant <Directory> block and reload Apache.

## 8. Move moodledata to New Disk

moodledata can be very large. It’s usually outside the web root for security.

1. Find its path from Moodle config:
```bash
sudo grep dataroot /var/www/html/moodle/config.php
# Example: $CFG->dataroot = '/var/moodledata';
```
2. Move it and relink or repoint:


Keep same path with symlink
```bash
sudo mv /var/moodledata /mnt/data/moodledata
sudo ln -s /mnt/data/moodledata /var/moodledata
sudo chown -R www-data:www-data /mnt/data/moodledata
```
Change config.php to the new absolute path
```bash
# Edit the dataroot line to point to /mnt/data/moodledata
sudo nano /var/www/html/moodle/config.php
# $CFG->dataroot = '/mnt/data/moodledata';


# Move the directory
sudo mv /var/moodledata /mnt/data/moodledata
sudo chown -R www-data:www-data /mnt/data/moodledata
```
Security: Ensure moodledata is not web-accessible (outside /var/www/html).

## 9. Start Services & Verify
```bash
# Start web stack
sudo systemctl start apache2 || sudo systemctl start nginx
sudo systemctl start cron || true


# Quick checks
ls -ld /var/www/html/moodle
# should show -> /mnt/data/moodle


sudo -u www-data php /var/www/html/moodle/admin/cli/checks.php || true
```
Open the site in a browser (replace host/IP):
```bash
http://<your-ec2-public-ip>/moodle
```
If you changed moodledata path, go to Site administration → Server → Environment and verify no errors. Also try uploading a small file to confirm writes land in /mnt/data/moodledata.

## 10. Free Up Root Space (Safe Cleanup)
Run these to reclaim space on /:
```bash
# Show largest top-level dirs
sudo du -x -sh /* 2>/dev/null | sort -hr | head -n 15


# APT cache
sudo apt-get clean


# Journals/logs (keep 1 day)
sudo journalctl --vacuum-time=1d
sudo rm -rf /var/log/*.gz /var/log/*.1


# Old kernels & packages
sudo apt -y autoremove --purge
```
Re-check usage:
```bash
df -h
```

Done!

You now have a 50 GB data volume mounted at /mnt/data with Moodle (and optionally moodledata) living there, freeing the root disk and keeping things tidy and scalable.

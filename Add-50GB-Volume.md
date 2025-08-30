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

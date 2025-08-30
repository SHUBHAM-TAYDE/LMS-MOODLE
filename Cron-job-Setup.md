# Moodle Server Cron Jobs Setup

This document explains the cron jobs configured for the Moodle server on Ubuntu.

---

## 📌 Location of Cron Jobs
The cron jobs are configured under the `ubuntu` user using:

```bash
crontab -l
```

##  Weekly Moodle Database Backup
```bash
0 2 * * 0 mysqldump -u muser -p'moodle@123$' moodle | gzip > /mnt/data/moodle_backups/moodle_db_$(date +\%F).sql.gz
```
What it does :

- Runs every Sunday at 2:00 AM.

- Dumps the Moodle database (moodle) using mysqldump.

- Compresses the dump with gzip.

- Saves the backup file in /mnt/data/moodle_backups/ with the current date in the filename.

    Example: moodle_db_2025-08-21.sql.gz

🔹 Why this is important

- Ensures that a consistent backup of the Moodle database is available every week.

- Helps in disaster recovery if the database gets corrupted or data is lost.

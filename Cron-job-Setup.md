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

## Old Backup Cleanup (Retention Policy)
```bash
30 2 * * 0 find /mnt/data/moodle_backups/ -type f -name "*.sql.gz" -mtime +28 -delete
```
🔹 What it does :

- Runs every Sunday at 2:30 AM (after the backup job).

- Finds all .sql.gz files in /mnt/data/moodle_backups/ older than 28 days.

- Deletes them automatically.

🔹 Why this is important :

- Prevents the server from running out of storage space due to old backups.

- Ensures only the latest 4 weeks of backups are kept.


  ## Moodle Cron Job (Every 5 Minutes) for Notification
  ```bash
  */5 * * * *  /usr/bin/php8.3  /var/www/html/moodle/admin/cli/cron.php > /dev/null 2>&1
  ```
  
🔹 What it does :
- Runs every 5 minutes.

- Executes Moodle’s internal scheduled tasks via cron.php.

- Redirects all output and errors to /dev/null (to prevent filling up system logs).

🔹 Why this is important
- Moodle requires its cron to be run frequently to:

    - Send email notifications.

    - Process queued tasks (e.g., quiz attempts, forum digests).

    - Clean up temporary data.

    - Run automated course tasks.

⚠️ If this cron job is not running, Moodle will show a warning in the admin dashboard.

Verify Cron Jobs
To check if cron jobs are installed:
```bash
crontab -l
```
To check Moodle cron execution:
```bash
php /var/www/html/moodle/admin/cli/cron.php
```


## Cron Jobs Summary

| Job                  | Schedule              | Purpose                                |
|-----------------------|-----------------------|----------------------------------------|
| Moodle DB Backup      | Weekly @ 2:00 AM (Sunday)   | Creates compressed DB backup            |
| Old Backup Cleanup    | Weekly @ 2:30 AM (Sunday)   | Deletes backups older than 28 days      |
| Moodle Cron           | Every 5 minutes       | Runs Moodle background tasks (events, notifications, cleanup) |


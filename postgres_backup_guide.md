# Automatic PostgreSQL Database Backup with GitHub Push (Next.js + VPS)

This guide explains how to **automatically backup a PostgreSQL database** on a VPS running a Next.js + TypeScript project, store the backup locally, and push it to GitHub daily at 3 AM using a **fine-grained GitHub token**.

---

## **1. Prerequisites**

- VPS access (SSH) with root or user privileges  
- PostgreSQL installed and database ready  
- Git installed on VPS  
- A GitHub repository to store backups  
- Fine-grained GitHub personal access token (PAT)  

---

## **2. Directory Structure**

Assume your project is located at:

```
/home/srv1094267/apps/project3/
```

Create a `scripts` folder for the backup script:

```bash
cd /home/srv1094267/apps/project3
mkdir scripts
cd scripts
```

---

## **3. Create Backup Script**

Create `db_backup.sh` inside the `scripts` folder:

```bash
nano db_backup.sh
```

Paste the following script:

```bash
#!/bin/bash

# --------------------------
# Database configuration
# --------------------------
DB_NAME="cartex_db"
DB_USER="postgres"
DB_PASSWORD="root"

# Script directory (current folder)
SCRIPT_DIR="$(pwd)"

# Backup filename with timestamp
DATE=$(date +'%Y-%m-%d_%H-%M-%S')
BACKUP_FILE="${SCRIPT_DIR}/${DB_NAME}_backup_$DATE.sql"

# Export password for pg_dump
export PGPASSWORD=$DB_PASSWORD

# Delete old backups in folder
rm -f ${SCRIPT_DIR}/${DB_NAME}_backup_*.sql

# Create new backup
pg_dump -U $DB_USER -F p $DB_NAME > "$BACKUP_FILE"

# --------------------------
# GitHub configuration
# --------------------------
GITHUB_USERNAME="mesumali-dev"
GITHUB_TOKEN="your_fine_grained_token_here"
GITHUB_REPO="cartex_backup"

# Initialize git repo if not exists
if [ ! -d ".git" ]; then
    git init
    git branch -M main
    git remote add origin https://${GITHUB_USERNAME}:${GITHUB_TOKEN}@github.com/${GITHUB_USERNAME}/${GITHUB_REPO}.git
fi

# Add, commit, and push
git add .
git commit -m "Database backup $DATE" || echo "Nothing to commit"
git push -u origin main --force

echo "Backup completed and pushed to GitHub: $BACKUP_FILE"
```

> **Important:** Replace `your_fine_grained_token_here` with your actual fine-grained GitHub token.  

---

## **4. Make Script Executable**

```bash
chmod +x db_backup.sh
```

---

## **5. Test Script Manually**

Run the script:

```bash
./db_backup.sh
```

- A `.sql` backup file will be created in the `scripts` folder  
- It will push the backup to your GitHub repository  

Check:

```bash
ls
```

You should see:

```
cartex_db_backup_2025-11-15_09-34-56.sql  db_backup.sh
```

---

## **6. Set Up Automatic Daily Backup (Cron Job)**

Edit crontab:

```bash
crontab -e
```

Add the following line to run the backup daily at **3 AM**:

```bash
0 3 * * * cd /home/srv1094267/apps/project3/scripts && ./db_backup.sh >> backup.log 2>&1
```

- `0 3 * * *` → 3:00 AM daily  
- `cd ... && ./db_backup.sh` → run script in correct folder  
- `>> backup.log 2>&1` → logs output and errors to `backup.log`  

Save and exit:  

- **Nano:** `Ctrl + O` → `Enter` → `Ctrl + X`  
- **Vi/Vim:** `Esc` → `:wq` → `Enter`  

---

## **7. Verify Cron Job**

```bash
crontab -l
```

You should see your 3 AM job listed.  

---

## **8. GitHub Notes**

- Make sure your **fine-grained token** has `Contents → Read & Write` permission for the `cartex_backup` repository.  
- The script will **force push**, replacing old backups in GitHub with the latest one.  
- All local backups except the latest will be deleted automatically.  

---

## **9. Troubleshooting Tips**

- **“No such file or directory”** → Make sure the script path is correct and you are running it from the `scripts` folder  
- **403 error on push** → Token may be invalid or lacking permissions. Regenerate a **fine-grained token** with `repo access`.  
- **Cron not running** → Make sure the script path in crontab is absolute. Check logs:

```bash
cat /home/srv1094267/apps/project3/scripts/backup.log
```

- **Multiple backups needed** → Remove `rm -f` line in the script to keep all backups.  

---

## ✅ **Done!**

Now your **Next.js project database** is:

- Backed up daily at 3 AM  
- Old local backups deleted automatically  
- Latest backup pushed to GitHub securely using a fine-grained token  

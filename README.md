# Automated Database Backup

An automated MariaDB/MySQL database backup solution that backs up databases to Cloudflare R2 storage with automatic retention management and easy restoration capabilities.

## Project URL
https://roadmap.sh/projects/automated-backups

## Features

- **Automated Backups**: Dumps MariaDB/MySQL databases with timestamps
- **Compression**: Automatically compresses backups to save storage space
- **Cloud Storage**: Uploads backups to Cloudflare R2 (S3-compatible)
- **Retention Policy**: Automatically maintains only the 7 most recent backups
- **Easy Restoration**: Download and restore from the latest backup
- **Portable**: Scripts automatically detect their location and work from any directory
- **Error Handling**: Built-in error checking at each step

## Prerequisites

- MariaDB/MySQL server
- `mariadb-dump` (or `mysqldump`)
- AWS CLI configured for Cloudflare R2
- Bash shell

## Installation

1. Clone this repository:
   ```bash
   git clone <repository-url>
   cd automated_db_backup
   ```

2. Create a `.env` file in the project root with the following variables:
   ```bash
   DB_NAME=your_database_name
   DB_USER=your_database_user
   DB_PASS=your_database_password
   ```

3. Make the scripts executable:
   ```bash
   chmod +x backup.sh restore.sh
   ```

4. Configure AWS CLI for Cloudflare R2:
   ```bash
   aws configure
   ```
   Use your Cloudflare R2 Access Key ID and Secret Access Key.

## Usage

### Backup Database

Run the backup script from any location:

```bash
/path/to/backup.sh
# or from the project directory
./backup.sh
```

This script will:
1. Dump the database to a SQL file
2. Compress it into a `.tar.gz` archive with a timestamp
3. Upload the archive to your R2 bucket
4. Automatically remove old backups, keeping only the 7 most recent
5. Clean up temporary files

### Restore Database

Run the restore script from any location:

```bash
/path/to/restore.sh
# or from the project directory
./restore.sh
```

This script will:
1. List and identify the latest backup from R2
2. Download the backup to `~/restoredb`
3. Extract the compressed archive
4. Prompt you to confirm database restoration
5. Restore the database if confirmed

## Configuration

### Backup Settings

Edit [backup.sh](backup.sh) to customize:
- `BACKUP_DIR`: Temporary backup directory (default: `/tmp/maria_backup`)
- `BUCKET`: Your Cloudflare R2 bucket name
- `S3_API`: Your R2 endpoint URL
- Maximum backup retention (default: 7 backups)

### Restore Settings

Edit [restore.sh](restore.sh) to customize:
- `RESTORE_DIR`: Directory for downloaded backups (default: `~/restoredb`)
- `BUCKET`: Your Cloudflare R2 bucket name
- `S3_API`: Your R2 endpoint URL

## Portability Features

The scripts are now fully portable:
- **Automatic Path Detection**: Scripts automatically determine their location using `SCRIPT_DIR`
- **Relative .env Loading**: The `.env` file is loaded relative to the script location
- **User-Independent Paths**: Uses `$HOME` variable for restore directory instead of hardcoded paths
- **Run from Anywhere**: Can be executed from any directory or via cron without issues

## Automation

To automate backups, add a cron job (works regardless of script location):

```bash
# Edit crontab
crontab -e

# Add a line to run backup daily at 2 AM
0 2 * * * /home/hoang/projects/automated_db_backup/backup.sh
```

## Backup Retention

The backup script automatically maintains a maximum of **7 backups** in your R2 storage. When the 8th backup is created, the oldest backup is automatically deleted. To change this limit, modify the retention check in [backup.sh](backup.sh):

```bash
# Change 7 to your desired number
while [[ "$NUM_BACKUP" -gt 7 ]]; do
```

## Backup File Format

Backups are named with the following format:
```
YYYY-MM-DD_HH-MM-SS_backup_<database_name>.tar.gz
```

Example: `2026-03-03_02-00-00_backup_mydb.tar.gz`

## Security Notes

- Keep your `.env` file secure and never commit it to version control
- Add `.env` to your `.gitignore` file
- Use strong passwords for database access
- Restrict access to backup files and R2 bucket
- Consider encrypting backups for sensitive data
- The `.env` file must be in the same directory as the scripts

## Troubleshooting

### "cant dump your mariadb database"
- Verify database credentials in `.env`
- Ensure MariaDB/MySQL is running
- Check user permissions

### "cant upload file to storage"
- Verify AWS CLI is configured correctly
- Check R2 bucket name and endpoint URL
- Ensure you have write permissions to the bucket

### "cant determine the latest version"
- Verify R2 bucket contains backup files
- Check AWS CLI configuration
- Ensure bucket name is correct

### "cant find the oldest backup on storage"
- This occurs during backup retention management
- Check R2 bucket permissions
- Verify AWS CLI endpoint configuration

## Project Structure

```
automated_db_backup/
├── backup.sh       # Main backup script with retention management
├── restore.sh      # Restore script for database recovery
├── .env            # Database credentials (not in version control)
└── README.md       # This file
```

## License

This project is open source and available under the MIT License.

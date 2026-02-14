---
name: backup-recovery
description: "Implement comprehensive backup and disaster recovery strategies for databases, applications, and infrastructure with automated testing and verification."
---

# Backup & Recovery Skill

Implement comprehensive backup and disaster recovery strategies for production systems.

## When to Use

Use this skill when the user wants to:
- Implement database backup strategies
- Set up automated backup systems
- Create disaster recovery plans
- Implement point-in-time recovery
- Set up backup monitoring and alerting
- Implement backup encryption and security
- Create backup retention policies
- Test backup restoration procedures
- Implement cross-region backups
- Set up backup verification
- Implement incremental and differential backups
- Create business continuity plans

## Technology Stack

### Backup Tools

#### Database Backups
- **PostgreSQL**: pg_dump, pg_basebackup, WAL archiving
- **MySQL**: mysqldump, mysqlpump, Percona XtraBackup
- **MongoDB**: mongodump, MongoDB Atlas backups
- **Redis**: RDB snapshots, AOF persistence
- **Elasticsearch**: Snapshot and Restore API

#### File System Backups
- **rsync**: Incremental file synchronization
- **Restic**: Encrypted, deduplicated backups
- **Duplicity**: Encrypted bandwidth-efficient backup
- **Bacula**: Enterprise backup solution
- **Borgbackup**: Deduplicated, encrypted backups

#### Cloud Backups
- **AWS**: S3, EBS snapshots, RDS backups, AWS Backup
- **Azure**: Azure Backup, Blob snapshots
- **GCP**: Cloud Storage, Persistent Disk snapshots
- **Backblaze B2**: Cost-effective cloud storage

#### Orchestration
- **Cron**: Scheduled job execution
- **Systemd timers**: Modern job scheduling
- **Airflow**: Workflow orchestration
- **Jenkins**: CI/CD with backup jobs

## Backup Strategies

### 1. Full Backups

**PostgreSQL Full Backup:**
```bash
#!/bin/bash
# postgresql-full-backup.sh

set -euo pipefail

# Configuration
DB_NAME="${DB_NAME:-myapp}"
DB_USER="${DB_USER:-postgres}"
BACKUP_DIR="${BACKUP_DIR:-/var/backups/postgresql}"
RETENTION_DAYS="${RETENTION_DAYS:-30}"
S3_BUCKET="${S3_BUCKET:-s3://my-backups/postgresql}"

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Generate timestamp
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_${TIMESTAMP}.sql.gz"

# Create backup
echo "Creating backup of $DB_NAME..."
pg_dump -U "$DB_USER" \
    -h localhost \
    -F c \
    -b \
    -v \
    -f "$BACKUP_FILE.tmp" \
    "$DB_NAME"

# Compress backup
echo "Compressing backup..."
gzip "$BACKUP_FILE.tmp"
mv "$BACKUP_FILE.tmp.gz" "$BACKUP_FILE"

# Calculate checksum
echo "Calculating checksum..."
sha256sum "$BACKUP_FILE" > "$BACKUP_FILE.sha256"

# Upload to S3
echo "Uploading to S3..."
aws s3 cp "$BACKUP_FILE" "$S3_BUCKET/full/"
aws s3 cp "$BACKUP_FILE.sha256" "$S3_BUCKET/full/"

# Verify upload
echo "Verifying upload..."
aws s3 ls "$S3_BUCKET/full/$(basename $BACKUP_FILE)"

# Clean old local backups
echo "Cleaning old backups..."
find "$BACKUP_DIR" -name "${DB_NAME}_*.sql.gz" -mtime +$RETENTION_DAYS -delete
find "$BACKUP_DIR" -name "${DB_NAME}_*.sql.gz.sha256" -mtime +$RETENTION_DAYS -delete

# Log completion
echo "Backup completed successfully: $BACKUP_FILE"
echo "Size: $(du -h $BACKUP_FILE | cut -f1)"

# Send notification
curl -X POST https://hooks.slack.com/services/YOUR/WEBHOOK/URL \
    -H 'Content-Type: application/json' \
    -d "{\"text\":\"PostgreSQL backup completed: ${DB_NAME}_${TIMESTAMP}\"}"
```

### 2. Incremental Backups

**PostgreSQL WAL Archiving:**
```bash
#!/bin/bash
# postgresql-wal-archive.sh

set -euo pipefail

WAL_FILE=$1
WAL_PATH=$2
S3_BUCKET="s3://my-backups/postgresql/wal"

# Upload WAL file to S3
aws s3 cp "$WAL_PATH" "$S3_BUCKET/$WAL_FILE"

# Verify upload
if aws s3 ls "$S3_BUCKET/$WAL_FILE" > /dev/null 2>&1; then
    echo "WAL file archived successfully: $WAL_FILE"
    exit 0
else
    echo "ERROR: WAL file upload failed: $WAL_FILE"
    exit 1
fi
```

**PostgreSQL Configuration (postgresql.conf):**
```conf
# Enable WAL archiving
wal_level = replica
archive_mode = on
archive_command = '/usr/local/bin/postgresql-wal-archive.sh %f %p'
archive_timeout = 300  # Force archive every 5 minutes

# Retention
wal_keep_size = 1GB
max_wal_senders = 3
```

### 3. Continuous Backup

**Python Backup Manager:**
```python
import os
import subprocess
import logging
from datetime import datetime, timedelta
from pathlib import Path
import boto3
import hashlib
from typing import List, Dict

class BackupManager:
    def __init__(self, config: dict):
        self.config = config
        self.s3_client = boto3.client('s3')
        self.backup_dir = Path(config['backup_dir'])
        self.retention_days = config.get('retention_days', 30)

        # Setup logging
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(levelname)s - %(message)s'
        )
        self.logger = logging.getLogger(__name__)

    def create_postgres_backup(self, db_name: str) -> str:
        """Create PostgreSQL backup."""
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        backup_file = self.backup_dir / f"{db_name}_{timestamp}.sql.gz"

        self.logger.info(f"Creating backup: {backup_file}")

        # Create backup
        cmd = [
            'pg_dump',
            '-U', self.config['db_user'],
            '-h', self.config['db_host'],
            '-F', 'c',  # Custom format
            '-Z', '9',  # Maximum compression
            '-f', str(backup_file),
            db_name
        ]

        try:
            subprocess.run(cmd, check=True, capture_output=True)
            self.logger.info(f"Backup created: {backup_file}")
            return str(backup_file)
        except subprocess.CalledProcessError as e:
            self.logger.error(f"Backup failed: {e.stderr.decode()}")
            raise

    def calculate_checksum(self, file_path: str) -> str:
        """Calculate SHA256 checksum."""
        sha256_hash = hashlib.sha256()
        with open(file_path, "rb") as f:
            for byte_block in iter(lambda: f.read(4096), b""):
                sha256_hash.update(byte_block)
        return sha256_hash.hexdigest()

    def upload_to_s3(self, file_path: str, s3_key: str) -> bool:
        """Upload file to S3 with verification."""
        try:
            # Upload file
            self.s3_client.upload_file(
                file_path,
                self.config['s3_bucket'],
                s3_key,
                ExtraArgs={
                    'ServerSideEncryption': 'AES256',
                    'StorageClass': 'STANDARD_IA'
                }
            )

            # Upload checksum
            checksum = self.calculate_checksum(file_path)
            checksum_key = f"{s3_key}.sha256"
            self.s3_client.put_object(
                Bucket=self.config['s3_bucket'],
                Key=checksum_key,
                Body=checksum.encode()
            )

            self.logger.info(f"Uploaded to S3: {s3_key}")
            return True

        except Exception as e:
            self.logger.error(f"S3 upload failed: {e}")
            return False

    def verify_backup(self, backup_file: str) -> bool:
        """Verify backup integrity."""
        try:
            # For PostgreSQL custom format
            cmd = ['pg_restore', '--list', backup_file]
            subprocess.run(cmd, check=True, capture_output=True)

            self.logger.info(f"Backup verified: {backup_file}")
            return True

        except subprocess.CalledProcessError as e:
            self.logger.error(f"Backup verification failed: {e}")
            return False

    def cleanup_old_backups(self):
        """Remove backups older than retention period."""
        cutoff_date = datetime.now() - timedelta(days=self.retention_days)

        for backup_file in self.backup_dir.glob('*.sql.gz'):
            if backup_file.stat().st_mtime < cutoff_date.timestamp():
                self.logger.info(f"Removing old backup: {backup_file}")
                backup_file.unlink()

                # Remove checksum file if exists
                checksum_file = Path(f"{backup_file}.sha256")
                if checksum_file.exists():
                    checksum_file.unlink()

    def run_backup(self, db_name: str):
        """Execute complete backup workflow."""
        try:
            # Create backup
            backup_file = self.create_postgres_backup(db_name)

            # Verify backup
            if not self.verify_backup(backup_file):
                raise Exception("Backup verification failed")

            # Upload to S3
            timestamp = datetime.now().strftime('%Y/%m/%d')
            s3_key = f"backups/{db_name}/{timestamp}/{Path(backup_file).name}"

            if not self.upload_to_s3(backup_file, s3_key):
                raise Exception("S3 upload failed")

            # Cleanup old backups
            self.cleanup_old_backups()

            self.logger.info("Backup workflow completed successfully")

        except Exception as e:
            self.logger.error(f"Backup workflow failed: {e}")
            self._send_alert(f"Backup failed: {e}")
            raise

    def _send_alert(self, message: str):
        """Send alert notification."""
        # Implement notification (Slack, email, etc.)
        pass
```

### 4. Application Backup

**File System Backup Script:**
```python
import os
import subprocess
from datetime import datetime
from pathlib import Path

class FileSystemBackup:
    def __init__(self, config: dict):
        self.config = config
        self.backup_root = Path(config['backup_root'])

    def create_restic_backup(self, source_dirs: list):
        """Create encrypted backup with Restic."""
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')

        # Initialize repository if needed
        self._init_restic_repo()

        # Create backup
        cmd = [
            'restic',
            '-r', self.config['restic_repo'],
            'backup',
            *source_dirs,
            '--tag', timestamp,
            '--exclude-file', self.config.get('exclude_file', '/dev/null')
        ]

        env = os.environ.copy()
        env['RESTIC_PASSWORD'] = self.config['restic_password']

        result = subprocess.run(cmd, env=env, capture_output=True, text=True)

        if result.returncode == 0:
            print(f"Backup completed: {timestamp}")
            print(result.stdout)
        else:
            print(f"Backup failed: {result.stderr}")
            raise Exception("Restic backup failed")

    def _init_restic_repo(self):
        """Initialize Restic repository if it doesn't exist."""
        cmd = ['restic', '-r', self.config['restic_repo'], 'snapshots']
        env = os.environ.copy()
        env['RESTIC_PASSWORD'] = self.config['restic_password']

        result = subprocess.run(cmd, env=env, capture_output=True)

        if result.returncode != 0:
            # Repository doesn't exist, initialize it
            init_cmd = ['restic', '-r', self.config['restic_repo'], 'init']
            subprocess.run(init_cmd, env=env, check=True)

    def prune_old_snapshots(self, keep_daily=7, keep_weekly=4, keep_monthly=12):
        """Remove old snapshots according to retention policy."""
        cmd = [
            'restic',
            '-r', self.config['restic_repo'],
            'forget',
            '--keep-daily', str(keep_daily),
            '--keep-weekly', str(keep_weekly),
            '--keep-monthly', str(keep_monthly),
            '--prune'
        ]

        env = os.environ.copy()
        env['RESTIC_PASSWORD'] = self.config['restic_password']

        subprocess.run(cmd, env=env, check=True)
```

## Recovery Procedures

### 1. Point-in-Time Recovery (PITR)

**PostgreSQL PITR Script:**
```bash
#!/bin/bash
# postgresql-pitr.sh

set -euo pipefail

TARGET_TIME="${1:-}"
BACKUP_FILE="${2:-}"
S3_BUCKET="s3://my-backups/postgresql"
RECOVERY_DIR="/var/lib/postgresql/recovery"
DATA_DIR="/var/lib/postgresql/14/main"

if [ -z "$TARGET_TIME" ] || [ -z "$BACKUP_FILE" ]; then
    echo "Usage: $0 <target-time> <backup-file>"
    echo "Example: $0 '2024-01-15 14:30:00' backup_20240115_120000.sql.gz"
    exit 1
fi

# Stop PostgreSQL
echo "Stopping PostgreSQL..."
sudo systemctl stop postgresql

# Backup current data directory
echo "Backing up current data directory..."
sudo mv "$DATA_DIR" "${DATA_DIR}.old.$(date +%Y%m%d_%H%M%S)"

# Create new data directory
echo "Creating new data directory..."
sudo mkdir -p "$DATA_DIR"
sudo chown postgres:postgres "$DATA_DIR"

# Restore base backup
echo "Restoring base backup..."
aws s3 cp "$S3_BUCKET/full/$BACKUP_FILE" /tmp/
sudo -u postgres pg_restore -d postgres -C /tmp/$BACKUP_FILE

# Create recovery configuration
echo "Creating recovery configuration..."
cat > "$DATA_DIR/recovery.signal" << EOF
# Recovery configuration
EOF

cat > "$DATA_DIR/postgresql.auto.conf" << EOF
# Point-in-time recovery configuration
restore_command = 'aws s3 cp $S3_BUCKET/wal/%f %p'
recovery_target_time = '$TARGET_TIME'
recovery_target_action = 'promote'
EOF

# Start PostgreSQL in recovery mode
echo "Starting PostgreSQL in recovery mode..."
sudo systemctl start postgresql

# Wait for recovery to complete
echo "Waiting for recovery to complete..."
while sudo -u postgres psql -c "SELECT pg_is_in_recovery();" | grep -q "t"; do
    echo "Still recovering..."
    sleep 5
done

echo "Recovery completed successfully!"
echo "Database restored to: $TARGET_TIME"
```

### 2. Automated Recovery Testing

**Recovery Test Framework:**
```python
import subprocess
import tempfile
from datetime import datetime
from pathlib import Path

class RecoveryTester:
    def __init__(self, config: dict):
        self.config = config

    def test_backup_restore(self, backup_file: str) -> bool:
        """Test backup restoration in isolated environment."""
        with tempfile.TemporaryDirectory() as temp_dir:
            try:
                # Create test database
                test_db = f"recovery_test_{datetime.now().strftime('%Y%m%d_%H%M%S')}"

                # Create database
                subprocess.run([
                    'createdb',
                    '-U', self.config['db_user'],
                    '-h', self.config['test_db_host'],
                    test_db
                ], check=True)

                # Restore backup
                subprocess.run([
                    'pg_restore',
                    '-U', self.config['db_user'],
                    '-h', self.config['test_db_host'],
                    '-d', test_db,
                    '-v',
                    backup_file
                ], check=True)

                # Verify data
                result = subprocess.run([
                    'psql',
                    '-U', self.config['db_user'],
                    '-h', self.config['test_db_host'],
                    '-d', test_db,
                    '-c', 'SELECT COUNT(*) FROM users;'
                ], check=True, capture_output=True, text=True)

                # Check if we got data
                if 'rows' in result.stdout:
                    print(f"✓ Backup restoration successful: {backup_file}")
                    return True

                return False

            except Exception as e:
                print(f"✗ Backup restoration failed: {e}")
                return False

            finally:
                # Cleanup test database
                subprocess.run([
                    'dropdb',
                    '-U', self.config['db_user'],
                    '-h', self.config['test_db_host'],
                    '--if-exists',
                    test_db
                ])

    def run_weekly_recovery_test(self):
        """Run weekly recovery drill."""
        # Get latest backup
        latest_backup = self._get_latest_backup()

        # Test restoration
        success = self.test_backup_restore(latest_backup)

        # Record result
        self._record_test_result({
            'timestamp': datetime.now().isoformat(),
            'backup_file': latest_backup,
            'success': success
        })

        # Send report
        self._send_recovery_test_report(success, latest_backup)

        return success

    def _get_latest_backup(self) -> str:
        """Get path to latest backup file."""
        # Implementation to find latest backup
        pass

    def _record_test_result(self, result: dict):
        """Record test result for compliance."""
        # Implementation to record results
        pass

    def _send_recovery_test_report(self, success: bool, backup_file: str):
        """Send recovery test report to team."""
        # Implementation to send report
        pass
```

## Monitoring and Alerting

### Backup Monitoring

**Python Monitoring Script:**
```python
import boto3
from datetime import datetime, timedelta
from typing import List, Dict

class BackupMonitor:
    def __init__(self, config: dict):
        self.config = config
        self.s3_client = boto3.client('s3')

    def check_backup_freshness(self, max_age_hours: int = 25) -> Dict:
        """Verify backups are recent enough."""
        bucket = self.config['s3_bucket']
        prefix = self.config['backup_prefix']

        # Get latest backup
        response = self.s3_client.list_objects_v2(
            Bucket=bucket,
            Prefix=prefix,
            MaxKeys=1
        )

        if 'Contents' not in response:
            return {
                'status': 'CRITICAL',
                'message': 'No backups found'
            }

        latest_backup = response['Contents'][0]
        backup_age = datetime.now(latest_backup['LastModified'].tzinfo) - latest_backup['LastModified']

        if backup_age > timedelta(hours=max_age_hours):
            return {
                'status': 'CRITICAL',
                'message': f'Latest backup is {backup_age.total_seconds()/3600:.1f} hours old',
                'last_backup': latest_backup['Key']
            }

        return {
            'status': 'OK',
            'message': f'Latest backup is {backup_age.total_seconds()/3600:.1f} hours old',
            'last_backup': latest_backup['Key']
        }

    def verify_backup_integrity(self, backup_key: str) -> bool:
        """Verify backup checksum."""
        # Download checksum
        checksum_key = f"{backup_key}.sha256"

        try:
            checksum_obj = self.s3_client.get_object(
                Bucket=self.config['s3_bucket'],
                Key=checksum_key
            )
            expected_checksum = checksum_obj['Body'].read().decode()

            # Compare with S3 ETag (for simple uploads)
            backup_obj = self.s3_client.head_object(
                Bucket=self.config['s3_bucket'],
                Key=backup_key
            )

            # Verify size is reasonable
            if backup_obj['ContentLength'] < 1024:  # Less than 1KB
                return False

            return True

        except Exception as e:
            print(f"Integrity check failed: {e}")
            return False

    def check_retention_compliance(self) -> Dict:
        """Verify retention policy compliance."""
        bucket = self.config['s3_bucket']
        prefix = self.config['backup_prefix']

        # Expected backup counts
        expected_daily = 7
        expected_weekly = 4
        expected_monthly = 12

        # Get all backups
        backups = self._list_all_backups(bucket, prefix)

        # Categorize by age
        now = datetime.now()
        daily_backups = []
        weekly_backups = []
        monthly_backups = []

        for backup in backups:
            age_days = (now - backup['LastModified'].replace(tzinfo=None)).days

            if age_days <= 7:
                daily_backups.append(backup)
            elif age_days <= 30:
                weekly_backups.append(backup)
            elif age_days <= 365:
                monthly_backups.append(backup)

        issues = []
        if len(daily_backups) < expected_daily:
            issues.append(f"Missing daily backups: {len(daily_backups)}/{expected_daily}")

        if len(weekly_backups) < expected_weekly:
            issues.append(f"Missing weekly backups: {len(weekly_backups)}/{expected_weekly}")

        if len(monthly_backups) < expected_monthly:
            issues.append(f"Missing monthly backups: {len(monthly_backups)}/{expected_monthly}")

        if issues:
            return {
                'status': 'WARNING',
                'message': ', '.join(issues)
            }

        return {
            'status': 'OK',
            'message': 'Retention policy compliant'
        }

    def _list_all_backups(self, bucket: str, prefix: str) -> List[Dict]:
        """List all backups in S3."""
        backups = []
        continuation_token = None

        while True:
            params = {
                'Bucket': bucket,
                'Prefix': prefix
            }

            if continuation_token:
                params['ContinuationToken'] = continuation_token

            response = self.s3_client.list_objects_v2(**params)

            if 'Contents' in response:
                backups.extend(response['Contents'])

            if not response.get('IsTruncated'):
                break

            continuation_token = response.get('NextContinuationToken')

        return backups
```

## Disaster Recovery Plan

### DR Runbook Template

**disaster-recovery.md:**
```markdown
# Disaster Recovery Runbook

## Overview
- **RTO (Recovery Time Objective)**: 4 hours
- **RPO (Recovery Point Objective)**: 1 hour
- **Last Updated**: 2024-01-15
- **Last Tested**: 2024-01-10

## Contact Information

### On-Call Team
- Primary: ops-primary@company.com
- Secondary: ops-secondary@company.com
- Manager: ops-manager@company.com

### External Contacts
- AWS Support: 1-800-XXX-XXXX
- Database Vendor: support@vendor.com

## Disaster Scenarios

### Scenario 1: Database Corruption

**Detection:**
- Database errors in logs
- Application errors
- Data integrity checks fail

**Recovery Procedure:**
1. Declare incident and notify team
2. Stop application to prevent further corruption
3. Assess extent of corruption
4. Identify last known good backup
5. Execute point-in-time recovery
6. Verify data integrity
7. Resume application
8. Monitor for issues

**Commands:**
```bash
# Stop application
kubectl scale deployment myapp --replicas=0

# Start recovery
./scripts/postgresql-pitr.sh "2024-01-15 12:00:00" backup_20240115_100000.sql.gz

# Verify recovery
psql -c "SELECT COUNT(*) FROM users;"

# Resume application
kubectl scale deployment myapp --replicas=3
```

### Scenario 2: Region Failure

**Detection:**
- AWS health dashboard
- Monitoring alerts
- Application unavailable

**Recovery Procedure:**
1. Activate DR plan
2. Notify stakeholders
3. Failover to secondary region
4. Restore from cross-region backup
5. Update DNS
6. Verify application
7. Monitor performance

**Commands:**
```bash
# Failover DNS
aws route53 change-resource-record-sets ...

# Start services in DR region
terraform apply -var="region=us-west-2"

# Restore database
./scripts/restore-from-cross-region-backup.sh
```

## Testing Schedule

- **Quarterly**: Full DR drill
- **Monthly**: Backup restoration test
- **Weekly**: Backup verification
- **Daily**: Backup completion check
```

## Best Practices

### 1. Backup Strategy
- Implement 3-2-1 rule (3 copies, 2 media, 1 offsite)
- Encrypt backups at rest and in transit
- Use immutable backups where possible
- Implement automated backup testing
- Document backup procedures

### 2. Security
- Encrypt backups with strong encryption
- Secure backup credentials
- Implement access controls
- Audit backup access
- Secure backup transmission

### 3. Retention
- Define clear retention policies
- Automate retention enforcement
- Consider compliance requirements
- Balance cost and retention needs
- Document retention decisions

### 4. Testing
- Test backups regularly
- Automate recovery testing
- Document test results
- Practice disaster recovery drills
- Update procedures based on tests

### 5. Monitoring
- Monitor backup completion
- Alert on backup failures
- Track backup size trends
- Monitor storage usage
- Verify backup integrity

### 6. Documentation
- Document all procedures
- Keep runbooks updated
- Document recovery time objectives
- Maintain contact information
- Record test results

### 7. Automation
- Automate backup creation
- Automate backup verification
- Automate retention enforcement
- Automate recovery testing
- Automate alerting

## Common Anti-Patterns to Avoid

1. **Never testing restores**: Always test backup restoration
2. **No encryption**: Always encrypt sensitive backups
3. **Single location**: Always have offsite backups
4. **No monitoring**: Always monitor backup success
5. **Manual processes**: Automate backup procedures
6. **Ignoring retention**: Enforce retention policies
7. **No documentation**: Document all procedures

## Deliverables

When implementing backup and recovery, ensure you deliver:

1. **Backup System**
   - Automated backup scripts
   - Backup verification
   - Encryption configuration
   - Storage configuration

2. **Recovery Procedures**
   - Recovery scripts
   - PITR procedures
   - DR runbooks
   - Testing procedures

3. **Monitoring**
   - Backup success monitoring
   - Integrity verification
   - Retention compliance
   - Alert configuration

4. **Documentation**
   - Backup strategy document
   - Recovery procedures
   - DR plan
   - Testing results

5. **Compliance**
   - Retention policy
   - Encryption standards
   - Audit logs
   - Test records

## Quality Checklist

Before completing a backup implementation, verify:

- [ ] Automated backups configured and running
- [ ] Full and incremental backups implemented
- [ ] Backups encrypted in transit and at rest
- [ ] Offsite/cross-region backups configured
- [ ] Backup verification automated
- [ ] Recovery procedures documented
- [ ] Recovery procedures tested
- [ ] Point-in-time recovery tested
- [ ] Retention policies implemented
- [ ] Monitoring and alerting configured
- [ ] DR plan documented
- [ ] DR drill completed
- [ ] RTO and RPO defined and tested
- [ ] Team trained on procedures
- [ ] Compliance requirements met

# User Management V2 - Clean Code Edition

## 🎯 Overview

Enhanced user management role with clean code architecture, comprehensive validation, and advanced features.

## ✨ Features

- ✅ **Input Validation**: Username format, UID ranges, email validation
- 🔐 **Password Management**: Auto-generate secure passwords (configurable)
- 🛡️ **Error Handling**: Graceful error handling with rollback support
- 📧 **Email Notifications**: Thailand timezone, HTML/text templates
- 📝 **Audit Logging**: Complete audit trail
- 🧪 **Dry-run Mode**: Test without making changes
- 🔄 **Rollback Support**: Automatic rollback on errors
- 🎨 **Clean Code**: Modular design, separation of concerns

## 📋 Requirements

- Ansible >= 2.14
- Collections:
  - `community.general`
  - `ansible.posix`

## 🚀 Quick Start

### Basic Usage

```yaml
- name: Create users
  hosts: all
  become: yes
  roles:
    - role: company.infrastructure.user_management_v2
      vars:
        user_configs:
          - name: "appuser"
            groups: ["docker"]
            email: "admin@company.com"
            sr_number: "SR2024001"
```

### With Email Notification

```yaml
- name: Create users with notification
  hosts: all
  become: yes
  roles:
    - role: company.infrastructure.user_management_v2
      vars:
        enable_email_notification: true
        smtp_host: "smtp.company.com"
        smtp_port: 587
        smtp_use_tls: true
        email_template: "text"
        user_configs:
          - name: "appuser"
            email: "john@company.com"
            owner_name: "John Doe"
            sr_number: "SR2024001"
```

### Dry-run Mode

```yaml
- name: Test user creation
  hosts: all
  become: yes
  roles:
    - role: company.infrastructure.user_management_v2
      vars:
        dry_run: true
        user_configs:
          - name: "testuser"
```

## 📖 Variables

### Core Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `dry_run` | `false` | Enable dry-run mode |
| `debug_mode` | `false` | Enable debug output |
| `user_configs` | `[]` | List of users to create |

### Password Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `password_length` | `16` | Password length |
| `password_chars` | `ascii_letters,digits,punctuation` | Password character set |

### Email Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `enable_email_notification` | `false` | Enable email notifications |
| `smtp_host` | `localhost` | SMTP server |
| `smtp_port` | `25` | SMTP port |
| `email_template` | `text` | Email template (text/html) |

### Validation Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `validate_username_format` | `true` | Validate username format |
| `check_duplicate_usernames` | `true` | Check for duplicates |
| `fail_on_missing_groups` | `true` | Fail if groups missing |

## 🎨 User Configuration Format

```yaml
user_configs:
  - name: "username"              # Required
    uid: 5001                     # Optional (auto-assigned)
    password: ""                  # Optional (auto-generated if empty)
    groups: ["group1", "group2"]  # Optional
    home_directory: "/home/user"  # Optional (default: /home/username)
    shell: "/bin/bash"            # Optional (default: /bin/bash)
    email: "user@company.com"     # Required for notifications
    sr_number: "SR2024001"        # Service request number
    owner_name: "John Doe"        # Requester name
    tel: "081-234-5678"          # Contact number
    purpose: "App account"        # Purpose description
```

## 📊 Examples

### Example 1: Basic User Creation

```yaml
user_configs:
  - name: "appuser01"
    groups: ["docker", "developers"]
```

### Example 2: Full Configuration

```yaml
user_configs:
  - name: "dbadmin"
    uid: 5001
    password: "SecurePassword123!"
    groups: ["dba", "wheel"]
    home_directory: "/opt/dbadmin"
    shell: "/bin/zsh"
    email: "dba-team@company.com"
    sr_number: "SR2024-0042"
    owner_name: "Database Team"
    tel: "02-123-4567"
    purpose: "Database administration account"
```

### Example 3: Multiple Users with Notification

```yaml
enable_email_notification: true
smtp_host: "smtp.company.com"
email_template: "text"

user_configs:
  - name: "developer1"
    groups: ["developers"]
    email: "dev-lead@company.com"
    sr_number: "SR2024-100"
    owner_name: "Dev Lead"
  
  - name: "developer2"
    groups: ["developers"]
    email: "dev-lead@company.com"
    sr_number: "SR2024-100"
    owner_name: "Dev Lead"
```

## 🔧 Advanced Features

### Rollback on Error

```yaml
enable_rollback_on_error: true
```

### Audit Logging

```yaml
enable_audit_logging: true
# Logs saved to: /var/log/ansible/user_management/audit.log
```

### Save Report to File

```yaml
save_report_to_file: true
report_directory: "/var/log/ansible/user_management"
```

### Thailand Timezone

```yaml
use_thailand_timezone: true
timezone: "Asia/Bangkok"
timezone_display: "ICT"
```

## 📝 Output Examples

### Console Output

```
╔═══════════════════════════════════════════════════════╗
║   USER MANAGEMENT V2 - CLEAN CODE EDITION            ║
║   Version: 2.0.0                                      ║
║   Mode: PRODUCTION                                    ║
╚═══════════════════════════════════════════════════════╝

✅ ALL VALIDATIONS PASSED
├─ Users: 3
├─ Required groups: 2
└─ Missing groups: 0

👤 USER CREATION SUMMARY
├─ Total: 3
├─ Created: 3
├─ Skipped: 0
└─ Failed: 0

✅ USER MANAGEMENT COMPLETED SUCCESSFULLY
```

### Email Output

```
Dear John Doe,

Your user account creation request SR2024001 has been successfully completed.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 SERVICE REQUEST SUMMARY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SR Number    : SR2024001
Requester    : John Doe
Completed At : 9 October 2025 14:30:45 (ICT)

👥 USER ACCOUNTS CREATED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Username : appuser
  Password: Xk9#mP2$vL8qR   ⚠️ Please store this securely.
```

## 🎯 Tags

```bash
# Run only validation
ansible-playbook playbook.yml --tags validation

# Run user creation only
ansible-playbook playbook.yml --tags users

# Skip notifications
ansible-playbook playbook.yml --skip-tags notification
```

## 🐛 Troubleshooting

### Issue: Groups not found

**Solution**: Create groups manually or set:
```yaml
auto_create_missing_groups: true
fail_on_missing_groups: false
```

### Issue: Email not sending

**Solution**: Check SMTP settings and enable debug:
```yaml
debug_mode: true
smtp_timeout: 60
```

## 📚 License

MIT

## 👥 Author

IT Automation Team - Company Infrastructure Operations

** WIKI for setting up / managing AWS ec2 instances **

<a href="https://github.com/sosroInSpace/wordpress_wiki/wiki">View here</a>

# Creating a New WordPress Website from the Custom AMI

## Overview

A custom AMI named:

wordpress-starter-v1

has been created and contains:

* Amazon Linux 2023
* Nginx
* PHP 8.5
* MariaDB
* WordPress
* WP-CLI
* Default plugins installed
* SSH configured
* Backup policy enabled

Using this AMI allows a new WordPress server to be launched in approximately 5 minutes.

---

## Step 1 - Launch a New EC2 Instance

Navigate to:

AWS Console → EC2 → AMIs

Locate:

wordpress-starter-v1

Select:

Launch Instance from AMI

---

## Step 2 - Configure Instance

### Name

Example:

clientname-production

### Instance Type

Recommended:

t3.small

Suitable for:

* Business websites
* WooCommerce stores
* Low to medium traffic sites

### Key Pair

Select existing key pair:

perthpeptidepens.pem

(or another appropriate key)

### Storage

Recommended:

30 GB GP3

### Security Group

Allow:

* SSH (22)
* HTTP (80)
* HTTPS (443)

---

## Step 3 - Launch Instance

Click:

Launch Instance

Wait for:

Instance State = Running

Status Checks = 3/3 Passed

---

## Step 4 - Allocate Elastic IP

Navigate to:

EC2 → Elastic IPs

Select:

Allocate Elastic IP

Associate with the new instance.

Record the Elastic IP address.

This prevents IP changes if the server is restarted.

---

## Step 5 - Configure DNS

Create DNS records pointing to the Elastic IP.

Example:

A Record

example.com → Elastic IP

[www.example.com](http://www.example.com) → Elastic IP

Wait for DNS propagation.

---

## Step 6 - Connect via SSH

Add entry to local SSH config:

## SSH Configuration

Host clientname
    HostName SERVER_IP
    User ec2-user
    IdentityFile ~/.ssh/clientname.pem

Connect:

ssh clientname

---

## Step 7 - Create SSL Certificate

Install SSL using Certbot.

Example:

sudo dnf install -y certbot python3-certbot-nginx

sudo certbot --nginx -d example.com -d [www.example.com](http://www.example.com)

Verify:

https://example.com

---

## Step 8 - Log into WordPress

Visit:

https://example.com/wp-admin

Log in using the credentials configured during site setup.

---

## Step 9 - Install Theme

Upload:

* Custom theme
* Child theme
* Starter theme

as required.

---

## Step 10 - Import Website

Options:

### Option A - All In One WP Migration

Install plugin

Import backup

### Option B - UpdraftPlus

Restore backup

### Option C - Manual Migration

* Database import
* wp-content upload
* Search replace URLs

---

## Step 11 - Verify Backups

Navigate:

EC2 → Lifecycle Manager

Confirm daily snapshot policy exists.

Policy:

Daily WordPress Server Backups

Retention:

14 Days

All EBS volumes are automatically included.

No additional tagging required.

---

## Step 12 - Create Manual Snapshot Before Go Live

Navigate:

EC2 → Volumes

Select volume

Actions → Create Snapshot

Description:

Client Name - Initial Production Backup

This provides a restore point before launch.

---

## Server Maintenance Checklist

For every new site:

☑ Launch from wordpress-starter-v1

☑ Allocate Elastic IP

☑ Configure DNS

☑ Configure SSL

☑ Install theme

☑ Install required plugins

☑ Import website

☑ Verify backups

☑ Create manual snapshot

☑ Go live

---

## Updating the Base AMI

When improvements are made to the starter server:

1. Launch layercayk-wordpress-starter-v1.0.0
2. Apply updates
3. Create new image

Naming convention:

wordpress-starter-v1.0.1
wordpress-starter-v1.0.2
wordpress-starter-v1.0.3

Always keep at least one previous AMI available for rollback.


## Server Configuration

| Setting | Value |
|----------|----------|
| **OS** | Amazon Linux 2023 |
| **Web Server** | Nginx |
| **PHP** | PHP 8.5 |
| **Database** | MariaDB |
| **Instance Type** | t3.small |
| **Storage** | 30 GB GP3 |
| **Backups** | Daily EBS Snapshots (14 Day Retention) |
| **SSL** | Let's Encrypt |
| **SSH User** | ec2-user |
| **AMI** | wordpress-starter-v1 |

### Included Software

- WordPress (Latest Stable)
- WP-CLI
- Nginx
- PHP-FPM
- MariaDB
- Certbot (Let's Encrypt)
- Advanced Custom Fields Pro
- Default LayerCayk plugin stack

### AWS Configuration

- Region: ap-southeast-2 (Sydney)
- Public Access: Enabled
- Elastic IP: Recommended
- Security Group Ports:
  - 22 (SSH)
  - 80 (HTTP)
  - 443 (HTTPS)

### Backup Strategy

- Automated EBS snapshots via AWS Lifecycle Manager
- Snapshot Frequency: Daily
- Retention: 14 Days
- Manual snapshot created before major deployments
- AMI created after significant server configuration changes

### Deployment Standard

All new WordPress websites should be launched from:

`layercayk-wordpress-starter-v1.0.0`

This ensures consistent server configuration, security settings, plugins, backups, and deployment procedures across all LayerCayk-managed websites.

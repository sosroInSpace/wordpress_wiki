Below is a script that can be used to back up the wp-content folder, and the SQL database, for a Wordpress instance. Before using this script:

- Make sure that `INSTANCE_NAME` is set to the name of the instance.
- Make sure the AWS CLI is installed, `aws`. Try running just `aws` to see if its installed. You may need to install it with `sudo apt install awscli`.
- Make sure the EC2 Instance has a Role that allows upload (`putObject`) to `s3://okmg-developers/wordpress-backups/INSTANCE_NAME/*`. Use the default role called **"Wordpress default role"** or something like that. Otherwise you may need to create an IAM Role and assign it to the EC2 instance, then give the IAM Role an inline policy with `putObject` access on `s3://okmg-developers/wordpress-backups/INSTANCE_NAME/*`.
- Ensure the instance is in the right timezone (for cronjob). Run `date` to check. If it's different, configure the timezone and reboot the instance with `sudo reboot`.
- Use a cronjob with `crontab -e` to run this script:
```
# website backup scripts (Sunday 2:30 am).
30 2 * * 0 /home/bitnami/scripts/site-backup.sh backupsql
35 2 * * 0 /home/bitnami/scripts/site-backup.sh backupwp
```

### The Script.
[Available here](https://github.com/okmediagroup/wordpress-assets/blob/master/site-backup.sh)
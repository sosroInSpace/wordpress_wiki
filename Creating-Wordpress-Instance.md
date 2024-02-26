## Launch New EC2 Instance.

- Sign in to the [AWS Console](https://aws.amazon.com/console/)
- Go to Services -> EC2.
- Click **Launch Instance**.

**1. Choose AMI:**
- Go **AWS Marketplace** and find **WordPress Certified by Bitnami**.

**2. Choose Instance Type:**
- Select **t3a.micro** (or **t3.micro**) for the instance type.
- t3a.micro uses AMD CPUs and costs a little less.
- If a service limit is hit with t3a.micro, just use t3.micro.

**3. Configure Instance:**
Make sure the following settings are used:
- Auto-assign Public IP = **Use subnet settings (Enable)**.
- Shutdown Behaviour = **Stop**.
- Enable termination protection = **Yes**.
- T2/T3 Unlimited = **No**.

**4. Add Storage:**
Make sure the following settings are used:
- Size (GiB) = **10**.
- Volume Type = **gp2**.
- Delete on Termination = **No**.

**5. Add Tags:**
Add the following tags (case-sensitive):
- "Name": **"instanceName"**.
- "project": **"companyName"**.

**6. Configure Security Groups:**
- Pick **"Select an existing security group"**.
- Select **WordpressPublic** and **DeveloperSSH**.

**Launch Instance.**
- Choose **"Create a new key pair"**.
- Call the key pair the same name as the Name tag (step 5), **"instanceName"**.
- Click **Launch** and select _Create a new key pair_, and give the key pair a name like "company-name".
- Download the key, and save it somewhere on your machine. It is suggested to save all keys to **~/.ssh/**.

## Secure key with password.
- Create password-protected version of MY_KEY.pem with `openssl rsa -aes256 -in MY_KEY.pem -out MY_KEY-priv.pem`. **Ask Staff for what the password should be**.
- Delete the old unprotected MY_KEY.pem
- Run `chmod 400 MY_KEY-priv.pem` to stop a SSH security warning when attempting to SSH.
- Upload the key to S3 with `aws s3 cp ./MY_KEY-priv.pem s3://okmg-developers/keys/MY_KEY-priv.pem` so that others may download and use the key (requires aws-cli).
- (OPTIONAL) add an entry to `~/.ssh/config` for the instance to make SSH-ing easier.

## SSH in and set datetime.
- SSH in with `ssh {INSTANCE_NAME}`.
- Run `sudo dpkg-reconfigure tzdata` and pick `Australia/Perth`.
- Run `date` to confirm the date.
- Reboot the instance with `sudo reboot`. This is needed to make `crontab -e` work with the correct date.

## Remove the Bitnami banner and /bitnami pages.
- SSH in with `ssh {INSTANCE_NAME}`.
- Edit file with `nano /opt/bitnami/apache2/conf/httpd.conf`
- Near the bottom of the file, **comment out** the line like so: `# Include "/opt/bitnami/apps/bitnami/banner/conf/banner.conf"`.
- Restart apache with `sudo service bitnami restart apache`.
- Confirm that the site is live and the banner has been removed.
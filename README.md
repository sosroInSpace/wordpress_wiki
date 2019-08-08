# Wordpress Assets Wiki

Welcome to the OKMG Wordpress Assets. This wiki contains a bunch of information about running Wordpress on AWS, as well as using the assets on this github repo to your advantage.

## Creating New Wordpress Site

1. Sign in to the [AWS Console](https://aws.amazon.com/)
2. Go to Services -> EC2.
3. Click **Launch Instance**.
4. Under Step 1 (Choose AMI), go **AWS Marketplace** and select **WordPress Certified by Bitnami**.
5. Under Step 2 (Choose Instance Type), select **t3.micro** for the instance type.
6. Under Step 3 (Configure Instance), make sure the default VPC is selected ("VPC 1"). Also **Uncheck T2/T3 Unlimited**.
7. Under Step 6 (Configure Security Groups), select _"Select an **existing security** group"_ and pick the groups **Wordpress Public** and **Staff SSH**.
8. All other settings should be automatically filled correctly by Bitnami.
9. Click **Launch** and select _Create a new key pair_, and give the key pair a name like "company-name".
10. Download the key, and save it somewhere on your machine. It is suggested to save all keys to **~/.ssh/**
11. Secure MY_KEY.pem with `openssl rsa -aes256 -in MY_KEY.pem -out MY_KEY_ENCRYPTED.pem`.
12. Run `chmod 400 MY_KEY_ENCRYPTED.pem` to stop a SSH security warning when attempting to SSH.


## Setting Up New Wordpress Site Boiler Plate

1. Once a new instance with a fresh Wordpress install has been created install **OKMG's Boilerplate Theme**.

2. Download "wp-content.zip" via - https://www.dropbox.com/s/g8zzs9m7cftlxdj/wp-content.zip?dl=0

3. Download "ACF-export.json" via - https://www.dropbox.com/s/fikpvn4zb5rwtbi/acf-export-2018-10-23.json?dl=0

        - wp-content.zip (contains OKMG boilerplate theme and all associated plugins required to build on)
        - ACF-export.json (contains boilerplate importable custom fields)

4. **SFTP / SSH ** into the new Wordpress instance and replace the current "wp-content" with the version downloaded through drop-box.

5. Login to your Wordpress admin area and activate **Advanced Custom Fields** plugin as a minimum and any others needed.

6. Once Custom Fields have been activated a widget will appear within the left Wordpress panel.
 Navigate to **ACF** > **tools** > **import advanced custom fields** > import "ACF-export.json".
7. Done! your new Wordpress Boilerplate has been created.

**Notes**

1. Advanced Custom Fields allows you to create custom reusable components for Wordpress. These components can be 
   found once connected through SFTP within:
   **wp-content** > **themes** > **OKMG** > **flexible_content** > **templates**.

2. If a new component is created and you think it will be beneficial within OKMG's Boilerplate Wordpress Theme please save and update "wp-content" within the associated dropbox account.

3. Once a new component is created the corresponding stylesheet will need to be saved within "flexible_content/css" this stylesheet will need to be referenced through the boilerplates **functions.php** file. Each stylesheet will need to be added at the bottom of **functions.php** - **e.g.**

```
function enqueue_style() {
    wp_enqueue_style( 'main-style', get_template_directory_uri() . '/custom-style.css', null, time() );
    wp_enqueue_style( 'sub-style', get_template_directory_uri() . '/flexible_content/css/sub-style.css', null, time() );

}
add_action( 'wp_enqueue_scripts', 'enqueue_style' );

```  

     

## SSH Config

The following steps can be taken to make SSH-ing into the Wordpress instance easier. In the command-line:

1. `cd ~/.ssh/`
2. `nano config`
3. Copy + Paste the following to the bottom of the file:

```
Host SOME_NAME
    Hostname IP_ADDRESS
    User bitnami
    IdentityFile ~/.ssh/SSH_KEY_FILE_HERE
```

4. `Ctrl + O` to save.
5. `Ctrl + X` to quit.

Now, if you type `ssh SOME_NAME`, it will attempt to SSH into the given IP_ADDRESS using the details above.

## Install RSUB
Copy the following into the command line once you have SSH'd into the server. 

1. (sudo) wget -O /usr/local/bin/rsub \https://raw.github.com/aurora/rmate/master/rmate
2. (sudo) chmod a+x /usr/local/bin/rsub

## Bitnami Creating User

Bitnami's Wordpress comes with a default username and password (seen in the system logs for the EC2 instance), but these cannot be used anywhere. A new user should be created from scratch:

1. SSH into the server with `ssh SOME_NAME`.
2. Go to the wp-content directory with `cd ~/apps/wordpress/htdocs/wp-content/`.
3. Start a file with `mkdir mu-plugins`, `cd mu-plugins`, `nano create-admin-user.php`.
4. Copy in the following code, then use `Ctrl + O` to save, and `Ctrl + X` to quit:

```
<?php
add_action( 'init', function () {
  
	$username = 'temp_user';
	$password = 'testpassword123';
	$email_address = 'temp_user@example.com';
	if ( ! username_exists( $username ) ) {
		$user_id = wp_create_user( $username, $password, $email_address );
		$user = new WP_User( $user_id );
		$user->set_role( 'administrator' );
	}

} );
```

5. With the command line still open, login to Wordpress with the credentials above at `http://SITE_IP/admin`.
6. Set a new password for the existing user, "user".
7. Logout of Wordpress.
8. In the command line, exit the mu-plugins folder with `cd ..`, then delete it with `rm -rf mu-plugins`. This folder should only be temporary as it runs it's contents whenever the site is visited.
9. Login as "user" to see if it worked.
10. Delete old user "temp_user".

## Bitnami upload permissions with SFTP

Sometimes you may want to upload files to a theme folder, but cannot due to permissions. This is likely caused by ownership of the installed theme directory, which is set to `daemon:daemon` when downloaded and installed via Wordpress. With the SFTP client, the user `bitnami` is used, so to allow uploading:

1. SSH into the server with `ssh SOME_NAME`.
2. Go to the themes directory with `cd apps/wordpress/htdocs/wp-content/themes`.
3. Run `sudo chown -R bitnami:daemon THEME_DIR`. Uploading to this directory should now work.
4. Run `ls -l` to check if THEME_DIR directory now has `bitnami daemon`.

## Bitnami sendmail fix

The Bitnami instance may not be able to send emails, or may send them very slow (~ 1 minute). The following steps will fix this issue, and were based on the information seen in:

- [wordpress sendmail](https://community.bitnami.com/t/why-cant-i-use-phps-mail-function-on-this-bitnami-server/13319)
- [sendmail speed fix](https://stackoverflow.com/questions/7578952/sending-mail-takes-long-time-in-localhost/12279895)

Note that amazon limits the number of emails to 2000 per day. It is recommended that an external SMTP provider be used for emails.

1. SSH into the instance with `ssh SOME_NAME`.
2. run `sudo apt-get install sendmail`
3. run `sudo nano /etc/hosts` and insert the text `localhost localhost.localdomain` between the line `127.0.0.1 ip-XXX-XXX-XXX-XXX` to make it look like the line below. Use `Ctrl + O` to save, and `Ctrl + X` to quit.

```
127.0.0.1 localhost localhost.localdomain ip-XXX-XXX-XXX-XXX
```

4. run `sudo sendmailconfig` and answer 'Y' to everything.
5. run `nano /opt/bitnami/php/etc/php.ini` and search for the word "sendmail" using `Ctrl + O`.
6. Uncomment the line `sendmail_path = "env -i /usr/sbin/sendmail -t -i"` by removing the `;` at the start (or add the line if it doesn't exist).
7. Use `Ctrl + O` to save, and `Ctrl + X` to quit.
8. Restart bitnami service by running `sudo service bitnami restart`. This may take around 30 seconds.

The emails should be fixed now.

## Bitnami remove corner banner

The corner banner can be removed with the following steps, based on:

- [Bitnami Docs](https://docs.bitnami.com/aws/components/bninfo/)

1. SSH into the instance with `ssh SOME_NAME`.
2. run `sudo /opt/bitnami/apps/wordpress/bnconfig --disable_banner 1`
3. run `sudo /opt/bitnami/ctlscript.sh restart apache`

The corner banner that appears on first visit should be gone now.

## Bitnami allow HTTPS (once domain name is owned).
### NOTE: This guide may need to be updated, as lego has changed a bit.

The following steps can be taken to allow HTTPS through getting an SSL certificate from LetsEncrypt.

- [Bitnami Guide](https://docs.bitnami.com/aws/how-to/generate-install-lets-encrypt-ssl/)

1. SSH into the instance with `ssh SOME_NAME`.
2. Install the Lego client using the following 4 commands:
```
cd /tmp
curl -L -s https://api.github.com/repos/xenolf/lego/releases/latest | grep browser_download_url | grep linux_amd64 | cut -d '"' -f 4 | wget -i -
tar xf {downloaded_lego_file}.tar.gz
sudo mv lego /usr/local/bin/lego
```
3. Generate the SSL certificate with the following commands. Make sure to replace "DOMAIN-ONE" with your base domain name (Eg. `example.com`), and add an entry for every subdomain needed (`www.example.com`, `shop.example.com`, etc):
```
sudo /opt/bitnami/ctlscript.sh stop
sudo lego --email="EMAIL-ADDRESS" --domains="DOMAIN-ONE" --domains="DOMAIN-TWO" --path="/etc/lego" --http run
```
4. Move the old SSH certificates somewhere else, and link in the new ones with the following commands. Make sure to change "DOMAIN-ONE" to the correct name of the files ( make sure the generated certificates are within the correct folder path before doing this! ):
```
sudo mv /opt/bitnami/apache2/conf/server.crt /opt/bitnami/apache2/conf/server.crt.old
sudo mv /opt/bitnami/apache2/conf/server.key /opt/bitnami/apache2/conf/server.key.old
sudo mv /opt/bitnami/apache2/conf/server.csr /opt/bitnami/apache2/conf/server.csr.old
sudo ln -s /etc/lego/certificates/DOMAIN-ONE.key /opt/bitnami/apache2/conf/server.key
sudo ln -s /etc/lego/certificates/DOMAIN-ONE.crt /opt/bitnami/apache2/conf/server.crt
sudo chown root:root /opt/bitnami/apache2/conf/server*
sudo chmod 600 /opt/bitnami/apache2/conf/server*
```
5. Start all services again with `sudo /opt/bitnami/ctlscript.sh start`

Connect to the website now to check if HTTPS is correctly being used. No further configuration should be needed as the certificates are, by default, listed in `/opt/bitnami/apache2/conf/bitnami/bitnami.conf`.

### Auto-Renewal

1. Create a new file with `sudo nano /etc/lego/renew-certificate.sh`.
2. Paste in the following, and use `Ctrl + O` to save. The middle part should have the same information as step 3 above:
```
#!/bin/bash

sudo /opt/bitnami/ctlscript.sh stop apache
sudo /usr/local/bin/lego --email="EMAIL-ADDRESS" --domains="DOMAIN-ONE" --domains="DOMAIN-TWO" --path="/etc/lego" --http renew
sudo /opt/bitnami/ctlscript.sh start apache
```
4. Make the script executable with `sudo chmod +x /etc/lego/renew-certificate.sh`
5. Open the crontab with `sudo crontab -e`
6. Add the line `0 0 * * 0 /etc/lego/renew-certificate.sh 2> /dev/null` and save with `Ctrl + O` (if using nano). This will add a weekly cron job that runs on Sunday 12:00 am.

### Force HTTPS

Forcing the browser to user HTTPS is both secure and good for SEO. The following steps can be used to force HTTPS.

[Force All Applications](https://docs.bitnami.com/aws/components/apache/#how-to-force-https-redirection-for-an-application)

1. Open config file with `nano /opt/bitnami/apache2/conf/bitnami/bitnami.conf`.
2. Add the following lines directly below the top of the default host, `<VirtualHost _default_:80>`, and save with `Ctrl + O`:
```
DocumentRoot "/opt/bitnami/apache2/htdocs"
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```
3. Restart apache with `sudo /opt/bitnami/ctlscript.sh restart apache`

### Force www to non-www

Forcing the browser to use one domain name is good for SEO. The following steps can be used to force none-www e.g.('https://mydomain.com').

1. Open config file with `nano /opt/bitnami/apps/wordpress/conf/httpd-app.conf`.
2. Add the following lines at the top just after - `RewriteEngine On`:
```
RewriteCond %{HTTP_HOST} ^www\.(.+)$ [NC]
RewriteRule ^ http%{ENV:protossl}://%1%{REQUEST_URI} [L,R=301]
```

**(if you have run SSL https script)** 
1. Open config with sudo nano /opt/bitnami/apache2/conf/bitnami/bitnami.conf
2. Comment out rewrite rules by adding a # to the beginning of lines 18-21 and lines 60-63
**(end)**
3. Restart apache with `sudo /opt/bitnami/ctlscript.sh restart apache`

## Bitnami import MySQL database (optional for migrations).

1. Get database dump with `mysqldump -u root -p DB_NAME > PATH_TO_DB_FILE.sql`
2. Upload the database.sql file via SFTP.
3. SSH into the instance.
4. Find out the password for the root user (in ec2 management, select instance, Actions -> Instance Settings -> Get System Log).
5. Run `mysql -u root -p DB_NAME < PATH_TO_DB_FILE.sql`, where DB_NAME is the name of the DB to import to (usually bitnami_wordpress).
6. Enter the password from Step 3.

## Logging into phpMyAdmin from the browser

[Based on Bitnami Guide](https://docs.bitnami.com/virtual-machine/components/phpmyadmin/)

1. Run `ssh -N -L 8888:127.0.0.1:80 SOME_NAME`. If successful, will appear frozen (leave open for the duration of phpMyAdmin).
2. Go to `http://localhost:8888/phpmyadmin/` and login with `root` and `password` below.
3. Password is found on `AWS -> EC2 -> Actions -> Instance Settings -> Get System Log -> (Scroll down to find Password)`.

## Setting up SMTP with AWS

> REVO FITNESS NOTE:
> Please follow up to step 3, then click 'Retry' for the revo domain. All the rest has been done, and credentials are available in a previous email.

1. Login to AWS and find _Simple Email Service_.
2. Switch region to _EU (Ireland)_.
3. Go to **Domains**, and retry an existing domain or click **Verify a New Domain**.
4. Give the domain an unused subdomain name, like "mail.companyName.com.au"
5. Follow the verification instructions on the domain.
6. On the left, click **SMTP Settings**.
7. Click **Create My SMTP Credentials**.
8. Set the username to something like "companyName-SMTP".
9. Save the credentials of the new user somewhere! Maybe in an email to `joseph@okmg.com`.
10. Use these credentials with the SMTP Plugin for Wordpress.
11. SMTP will become active once the domain name has been verified on AWS.

## Finding out the root password for MySQL

1. Look in AWS -> EC2 -> Instance -> Actions -> Instance Settings -> Get System Log
2. Scroll down and find the password.
3. If the password cannot be found, it will have to be reset using the following [Binami Guide](https://docs.bitnami.com/installer/apps/edx/administration/change-reset-password/), **"Reset The MySQL Root Password"**. _Note: The Bitnami instances usually ship with 5.7.22_

## Setting up a remote database

1. SSH into the instance.
2. run `mysql -u root -p` to connect to the database.
3. create an empty database with `CREATE DATABASE bitnami_dev;`.
4. Create a new user to use remotely with `CREATE USER 'bn_dev'@'127.0.0.1' IDENTIFIED BY 'random_password_here';`. **Please don't use the usual password!** Use a made-up one like `M2on7Hdkem4` and write it down somewhere.
5. Grant **bn_dev** full permission on **bitnami_dev** with `GRANT ALL PRIVILEGES ON bitnami_dev.* TO 'bn_dev'@'127.0.0.1';`.
6. Exit mysql with `exit`.
7. Go to the home directory by running `cd`.
8. Clone the **bitnami_wordpress** to a local file with `mysqldump -u root -p bitnami_wordpress > temp.sql`.
9. Import the local file to **bitnami_dev** with `mysql -u root -p bitnami_dev < temp.sql`.

## Connecting to the remote database with local instance

1. Change your `wp-config.php` config to:
```
/** The name of the database for WordPress */
define('DB_NAME', 'bitnami_dev');

/** MySQL database username */
define('DB_USER', 'bn_dev');

/** MySQL database password */
define('DB_PASSWORD', 'random_password_here');

/** MySQL hostname */
define('DB_HOST', '127.0.0.1:8888');
```
2. Create an SSH tunnel with `ssh -N -L 8888:127.0.0.1:3306 SERVER_NAME`.
3. Run wordpress locally. It should work!

## Multisite setup.

Please see the following [Bitnami Guide](https://docs.bitnami.com/aws/apps/wordpress-multisite/configuration/configure-wordpress-multisite/) on how to do this.

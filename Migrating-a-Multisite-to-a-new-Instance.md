### Part 1: Download the old stuff.

_NOTE: Areas marked with {{DOUBLE_BRACKETS}} need to be replaced with your own fields._

1. Get the `wp-content` folder with `ssh {{OLD_INSTANCE}} 'cd /home/bitnami/apps/wordpress/htdocs && tar czf - wp-content' > ./wp-content.tar.gz`.
2. Get MySQL credentials for **step 3** with `ssh {{OLD_INSTANCE}} 'egrep -h "DB_NAME|DB_USER|DB_PASSWORD" /home/bitnami/apps/wordpress/htdocs/wp-config.php'`.
2. Get the database dump for the wordpress database with `ssh {{OLD_INSTANCE}} 'mysqldump -u bn_wordpress -p bitnami_wordpress' > ./database.sql`. You will be prompted for DB_PASSWORD gotten in **step 2**.

### Part 2: Stop bitnami on the new instance.

_We want to stop bitnami, as our changes will likely cause errors in the running bitnami daemons._

1. Stop bitnami with `ssh {{NEW_INSTANCE}} 'sudo /opt/bitnami/ctlscript.sh stop'`.

### Part 3: Install wp-content to new instance.

1. Upload `./wp-content.tar.gz` with `scp ./wp-content.tar.gz {{NEW_INSTANCE}}:~/wp-content.tar.gz`.
3. SSH in with `ssh {{NEW_INSTANCE}}`.
4. Rename `wp-content` to `old-wp-content`.
5. Install `./wp-content.tar.gz` with `tar xf ./wp-content.tar.gz -C /home/bitnami/apps/wordpress/htdocs`.
6. Give correct group to `wp-content` with `sudo chown -R bitnami:daemon /home/bitnami/apps/wordpress/htdocs/wp-content/`.
7. Give correct permissions to `wp-content` with `sudo chmod -R u+rw,g+rw /home/bitnami/apps/wordpress/htdocs/wp-content/`.
8. Get new MySQL credentials for **step 9** with `ssh {{OLD_INSTANCE}} 'egrep -h "DB_NAME|DB_USER|DB_PASSWORD" /home/bitnami/apps/wordpress/htdocs/wp-config.php'`.

### Part 4: Import database.

1. Upload `./database.sql` with `scp ./database.sql {{NEW_INSTANCE}}:~/database.sql`.
2. SSH in with `ssh {{NEW_INSTANCE}}`.
3. Get new MySQL credentials for **step 4** with `egrep -h "DB_NAME|DB_USER|DB_PASSWORD" /home/bitnami/apps/wordpress/htdocs/wp-config.php`.
4. Create a dump of the current database with `mysqldump -u bn_wordpress -p bitnami_wordpress > ~/old-database.sql`. You will be prompted for DB_PASSWORD gotten in **step 3**.
5. Import new database with `mysql -u bn_wordpress -p bitnami_wordpress < ~/database.sql`. You will be prompted for DB_PASSWORD gotten in **step 3**.

### Part 5: Start bitnami again.

1. Start bitnami with `ssh {{NEW_INSTANCE}} 'sudo /opt/bitnami/ctlscript.sh start'`.

### Part 6: Update the database if getting "ERR_TOO_MANY_REDIRECTS".

If switching from one domain to another, you might get **ERR_TOO_MANY_REDIRECTS** when visiting. To fix this, get into SQL using one of the following methods:

**With phpmyadmin**
1. Run one of the following commands depending on your scenario:
* `ssh -N -L 8888:127.0.0.1:80 {{INSTANCE}}`
* `ssh -N -L 8888:127.0.0.1:443 {{INSTANCE_NAME}}` (If instance redirects HTTP to HTTPS).
2. Visit `127.0.0.1:8888/phpmyadmin/`.
3. Login with a MySQL user. Use the credentials in **Part 4** -> **Step 3**.
4. Make sure **bitnami_wordpress** -> **wp_site** has the correct domain name.

**With MySQL**
1. SSH in with `ssh {{NEW_INSTANCE}}`.
2. Run `mysql -u {{DB_USER}} -p`. You will be prompted for DB_PASSWORD from **Part 4** -> **Step 3**.
3. In the SQL shell, run `use bitnami_wordpress;`.

**Once in, do the following:**
1. Run `select * from wp_site;` If the site is different, you need to update it with:
- `UPDATE wp_site SET domain='{{DOMAIN}}' WHERE id=1;
2. Run `select * from wp_blogs;` If the root sitename is different (usually `blog_id=1`), you need to update it with the new domain name:
- `UPDATE wp_blogs SET domain='{{DOMAIN}}' WHERE blog_id=1;
3. If there are subdomains in `wp_blogs`, update each of them to `{{SUBDOMAIN}}.{{NEW_DOMAIN}}` with blog_id=2,3,4,5....etc.
4. Run `SELECT option_name,option_value FROM wp_options WHERE option_name='siteurl' OR option_name='home';`. If the names are different, update them with:
- `UPDATE wp_options SET option_value='http://{{NEW_DOMAIN}}' WHERE option_name='siteurl' OR option_name='home';`.
5. Run `show tables;` and look for all the `wp_2_options`, `wp_3_options`, etc. For each of these, run:
- `SELECT option_name,option_value FROM wp_2_options WHERE option_name='siteurl' OR option_name='home';`.
- `UPDATE wp_2_options SET option_value='http://{{NEW_DOMAIN}}' WHERE option_name='siteurl' OR option_name='home';`.

### Part 7: Login to the Multisite.

1. Visit the `{{DOMAIN}}/wp-login.php` and login as an imported user from `database.sql`.


### Part 8: Associating Elastic IP

1. Associate Elastic IP
2. Visit new IP (website will be down)
3. SSH into instance and navigate to: /opt/bitnami/apps/wordpress > run sudo ./bnconfig --machine_hostname YOURELASTICIP
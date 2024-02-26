### How to access phpMyAdmin on a bitnami instance.

1. Run one of the following commands depending on your scenario:
* `ssh -N -L 8888:127.0.0.1:80 -i KEYFILE bitnami@SERVER-IP`
* `ssh -N -L 8888:127.0.0.1:80 INSTANCE_NAME`
* `ssh -N -L 8888:127.0.0.1:443 INSTANCE_NAME` (If instance redirects HTTP to HTTPS).
2. Visit `127.0.0.1:8888/phpmyadmin/`.
3. Login with a MySQL user. Run `egrep -h "DB_USER|DB_PASSWORD" /home/bitnami/apps/wordpress/htdocs/wp-config.php` to find the Wordpress DB User credentials.
4. Alternatively, login as root (username: `root`, password is inside `/home/bitnami/bitnami_credentials`). The username is usually `root` despite in the file it saying `user`.

*Note: Make sure you have shut down you localhost (XAMP or MAMP) otherwise it will create a conflict

### How to access MYSQL on a bitnami instance.

1. SSH in to the instance.
2. Run `sudo cat /home/bitnami/bitnami_credentials` to find the default password.
3. Run `mysql -u root -p`.
4. Enter the password.

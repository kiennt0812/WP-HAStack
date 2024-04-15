
## Load Balancer for Galera cluster

![]()
- WordPress is an open-source content management system (CMS) widely used to create and manage websites and blogs. WordPress provides a user-friendly interface for users to create content, including posts, pages, images, and videos.
- Nginx is an open-source web server widely used for serving websites and web applications. It's designed to efficiently handle concurrent web requests, providing high performance and resource efficiency

### Preparation :

| Hostname | OS | Role | IP address |
|--------|------|------|-------|
| app1 | ubuntu 20.04  | Active  | 192.168.64.16 |
| app2 | ubuntu 20.04  |  Active | 192.168.64.17 |
- Install PHP-FPM and and several other support packages. PHP-FPM is a program that has the ability to interpret PHP when running a Website for the Server.
    ```
    apt-get install php7.4 php7.4-cli php7.4-fpm php7.4-mysql php7.4-json php7.4-opcache php7.4-mbstring php7.4-xml php7.4-gd php7.4-curl
    ```

### 1. Download and Configure WordPress
- Start with create the root folder for your WordPress installation.   
    ```sh
    mkdir -p /var/www/html/wordpress/public_html
    ```
- Download the archived WordPress file using `wget` and unzip it to the root of the WordPress installation that we have created in the previous step.
     ```sh
    # cd /var/www/html/wordpress/public_html
    # wget https://wordpress.org/latest.tar.gz
    # tar -zxvf latest.tar.gz
    # mv wordpress/* .
    # rm -rf wordpress
    ```
- Change the ownership and apply correct permissions to the extracted WordPress files and folders. To do that, use the following command from the terminal.
    ```sh
    # cd /var/www/html/wordpress/public_html
    # chown -R www-data:www-data *
    # chmod -R 755 *
    ```
- Now provide the database name, database user and the password in the WordPress config file so that it can connect to the MySQL database that we had created earlier. By default, WordPress provides a sample configuration file and we will make use of it to create our own configuration file.
    ```sh
    cd /var/www/html/wordpress/public_html
    mv wp-config-sample.php wp-config.php
    vi wp-config.php
    ...
    ...
    define('DB_NAME', 'wordpress_db');                  
    define('DB_USER', 'haproxy_root');                        
    define('DB_PASSWORD', 'Trungkien1998@');           
    define( 'DB_HOST', '192.168.64.15' );        #Virtual IP          
    ```
- *Continue repeating the above steps with app2 server.*  
### 2. Install & Configure Nginx 
- On the HAProxy server app1 install the package.
    ```sh
    apt-get install nginx
    ```
- You can create a config file at `/etc/nginx/conf.d/wordpress.conf` with the following content :
    ```sh
    server {
        listen 80;
        root /var/www/html/wordpress/public_html;
        index index.php index.html;
        server_name localhost; 

        access_log /var/log/nginx/wptest.access.log;
        error_log /var/log/nginx/wptest.error.log;

        location / {
            try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        }
        
        location ~ /\.ht {
            deny all;
        }

        location = /favicon.ico {
            log_not_found off;
            access_log off;
        }

        location = /robots.txt {
            allow all;
            log_not_found off;
            access_log off;
        }

        location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
            expires max;
            log_not_found off;
        }
    }
    ```

- Don't forget to disable the sample config file located in `/etc/nginx/sites-enabled` :
    ```diff
    rm -rf /etc/nginx/site*  
    ```
- Once you’re done configuring, make sure the configuration file has correct syntax before reloading the configuration:
    ```diff
    nginx -t
    ```
    ```diff
    systemctl restart nginx 
    ```
- *Continue repeating the above steps with app2 server.*

### 3. Conclusion
WordPress is the most popular CMS and we learned how to install it with NGINX on a Ubuntu server. You can now proceed further to create your website with it.

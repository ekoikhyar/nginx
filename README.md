nginx
=====
Setup vps untuk wordpress dengan multidomain 
git remote add origin https://github.com/ekoikhyar/nginx.git
git push -u origin master

#how to install wordpress nginx (digitalosean)
Step One — Create a MySQL Database and User for WordPress
mysql -u root -p
CREATE DATABASE wordpress;
CREATE USER wordpressuser@localhost IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@localhost;
FLUSH PRIVILEGES;
exit

Step Two — Download WordPress to your Server
cd ~
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
sudo apt-get update
sudo apt-get install php5-gd libssh2-php

Step Three — Configure WordPress
cd ~/wordpress
cp wp-config-sample.php wp-config.php
nano wp-config.php
. . .
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', 'password');
. . .

Step Four — Copy the Files to the Document Root
sudo mkdir -p /var/www/html
sudo rsync -avP ~/wordpress/ /var/www/html/
cd /var/www/html/
sudo chown -R demo:www-data /var/www/html/*
mkdir wp-content/uploads
sudo chown -R :www-data /var/www/html/wp-content/uploads

Step Five — Modify Nginx Server Blocks
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/wordpress
sudo nano /etc/nginx/sites-available/wordpress

server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;

        root /var/www/html;
        index index.php index.html index.htm;

        server_name your_domain.com;

        location / {
                # try_files $uri $uri/ =404;
                try_files $uri $uri/ /index.php?q=$uri&$args;
        }

        error_page 404 /404.html;

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
                root /usr/share/nginx/html;
        }

        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass unix:/var/run/php5-fpm.sock;
                fastcgi_index index.php;
                include fastcgi_params;
        }
}

And now, We need to link our new file to the sites-enabled directory in order to activate it. We can do that like this:

sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
sudo service nginx restart
sudo service php5-fpm restart

Step Six — Complete the Installation through the Web Interface
http://your_domain.com
And next, install a wordpress.

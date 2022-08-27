## Setup LAMP Stack on EC2 with Ubuntu AMI
#### Setup Nginx
Update your local package index if this is your first interaction with the `apt` packaging in the terminal session.  
```
$ sudo apt-get update
```
Install Nginx
```
$ sudo apt-get install nginx
```
See [How to Nginx](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04) for  details.   

#### Setup a virtual host for a single website on Nginx
Create a directory for the site
```
$ sudo mkdir -p /var/www/example.com/html
```  
Change file ownership to you currently logged in account for convenience (you must not be logged in as root when doing this ).
```
$ sudo chown -R $USER:$USER /var/www/example.com/html
```
Create one or more sample HTML files
```
$ touch /var/www/example.com/html/index.html
$ echo "<p>Hello EC2</p>" >> /var/www/example.com/html/index.html
```  
Create you server block file for your domain by copying the default file.
```
$ sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/example.com
```
Update your server block
```
$ sudo nano  `/etc/nginx/sites-available/example.com`
```
It should look like this:
```
server {
        listen 80;
        listen [::]:80;

        # listen 443 ssl;
        # listen [::]:443 ssl;

        root /var/www/example.com/html;
        index index.html index.htm index.nginx-debian.html;

        server_name example.com, www.example.com;

        location / {
                try_files $uri $uri/ =404;
        }
  ## Other code...
}
```
Run Nginx test to make sure your server block is free of syntax error  
```
$ sudo nginx -t
```
Enable you server block
```
$ sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
```  
Restart Nginx
```
$ sudo systemctl restart nginx
```  
Add the entry to you host file (`/etc/hosts`)
```
127.0.0.1 example.com
```
Test the website  
```
$ curl http://example.com
```  
Learn more at [How to Virtual Host on Nginx](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-16-04)

#### Setup PHP  
Install PHP for Nginx  
```
$ sudo apt update
$ sudo apt install php7.4-fpm
```  
Check the status of PHP FPM (FastCGI Process Manager)
```
$ sudo systemctl status php7.4-fpm
```
Update you Nginx server block to enable PHP
```
server {

    # . . . other code
    root /var/www/example.com/html;
    index index.html index.htm index.php index.nginx-debian.html;

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.4-fpm.sock;
    }
}
```
Restart Nginx
```
$ sudo systemctl restart nginx
```  
Make a PHP file in the site's root directory  
```
$ touch /var/www/example.com/html/info.php
```  
Make the content of your PHP file (`info.php` in our case) like this
```
<?php  
phpinfo();
```  
Test the PHP page
```
$ curl example.com/info.php
```
Add an `A record` to your Route53 hosted zone with your public IP address as the `value`.  
Make sure the `A record`'s `name` is the same as the domain entry in your `/etc/hosts` file.

See more at [How to PHP](https://linuxize.com/post/how-to-install-php-on-ubuntu-20-04/)  

__Install PHP Extensions__  
Installing `curl`, `bcmath` and `mysql` PHP extensions
```
$ sudo apt-get install php7.4-curl
$ sudo apt-get install php7.4-bcmath
$ sudo apt-get install php7.4-mysql
```
Restart PHP fpm
```
$  sudo service php7.4-fpm restart
```

You may also need to uncomment some extension in the `php.ini` file to enable them

#### Setup MySQL
Install MySQL  
```
$ sudo apt update
$ sudo apt install mysql-server
```
Configure MySQL  
```
$ sudo mysql_secure_installation
```  
Check MySQL Server Status
```
$ sudo systemctl status mysql.service
```
Login to MySQL  
```
$ sudo mysql
```  
Add a new DB  
```
$ sudo mysql
mysql > CREATE DATABSE exampleDB;
```  
Create a new DB user
```
mysql > CREATE USER 'exampleUser'@'localhost' IDENTIFIED BY 'examplePassword';  
mysql >  GRANT ALL PRIVILEGES ON exampleDB.* TO 'exampleUser'@'localhost' WITH GRANT OPTION;
```  
To login using your newly created user
```
$ mysql -u exampleUser -p
```
Run a SQL script
```
mysql > use exampleDB;
mysql > source /path/to/script.sql
```  

To Copy File form you local computer to your EC2 instance
```
$ scp -i /path/to/keypair.pem /path/to/local/dir user@ec2-public-ip:path/to/remote/dir
```
For example
```
$ scp -r -i aws-linux2.pem d:/tochukwu/tables/ ubuntu@xx.xxx.xx.xxx:~/data/
```

###  Secure Nginx using Let's Ecrypt
First install Let's Ecrypt's Certbot and it's Nginx plugin
```
$ sudo apt install certbot python3-certbox-nginx
```
Make sure that the server block of your Nginx configuration file for the site have the `server_name` directive properly setup
```
...
server_name example.com www.example.com;
...
```  
If not, edit the config file, save and test for syntax correctness
```
$ sudo nginx -t
```
If you have firewall enabled, check your firewall status and allow Nginx Full profile which allows HTTP and HTTPS traffic
```
$ sudo ufw status
$ sudo ufw allow 'Nginx Full'
```  
You may  also delete the redundant `Nginx HTTP` only profile
```
$ sudo ufw delete allow 'Nginx HTTP'
```
Obtain an SSL Certificat using the certbot and nginx plugin
```
$ sudo certbot --nginx -d example.com -d www.example.com
```
Verify that certbot's auto-renewal systemd timer has been activated
```
$ sudo systemctl status certbot.timer
```
You can test the renewal process by doing a dry run
```
$ sudo certbot renew --dry-run
```
For detail see [Digital Ocean: How to Secure Nginx with Let's Encrypt in Ubuntu 20/04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04)

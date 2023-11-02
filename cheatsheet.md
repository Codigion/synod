# Cheatsheet
# Synod College Workshop

## Server Setup
//get to root environment

`sudo su -`

//refreshes the package database

`apt update`

//install PHP and it’s dependencies

`apt -y install php8.1-cli php8.1-common php8.1-fpm php8.1-mysql`

//install NGINX - a high-performance open-source web server and reverse proxy server used to serve web content and manage network traffic.

`apt -y install nginx`

//install MySQL

`apt -y install mysql-server`

//install Certbot - a command-line tool for automatically obtaining and renewing SSL/TLS certificates to enable secure, encrypted connections on web servers.

`apt -y install certbot python3-certbot-nginx`


## Clean-up defaults!
//Deletes the contents of the web server's document root.

`rm -rf /var/www/html`

//Removes the default Nginx configuration file.

`rm -rf /etc/nginx/sites-available/default`

//Deletes the symbolic link to the default Nginx configuration

`rm -rf /etc/nginx/sites-enabled/default`



## Clone your code!
Clones the 'weaver' repository from the 'Codigion' GitHub account to your local machine, creating a copy of the project for local development.

`cd /var/www`

`git clone https://github.com/Codigion/weaver`


## Get Inside Database
//Enter MySQL

`mysql`

//Creates a new database named "weaver."

`CREATE DATABASE weaver;`

//Creates a user named "weaver" who can connect from the "localhost" and uses the password "weaver@098" for authentication.

***CREATE USER "[USERNAME]"@"[HOST]" IDENTIFIED BY "[PASSWORD]";***

Example: `CREATE USER "weaver"@"localhost" IDENTIFIED BY "weaver@123";`

//Grants all privileges on the "weaver" database to the user "weaver" when connecting from the "localhost."

***GRANT ALL PRIVILEGES ON [DATABASE].* TO "[USERNAME]"@"[HOST]";***

Example: `GRANT ALL PRIVILEGES ON weaver.* TO "weaver"@"localhost";`

//Reloads the user privileges to ensure the changes take effect immediately.

`FLUSH PRIVILEGES;`

//Exit

`exit`


## Prepare to go LIVE!
//Create environment for Weaver project

`vi /var/www/weaver/Configurations/.env`

### .env File Content
```bash
## Environment
ENVIRONMENT=development

## MySQL Configuration
MYSQL_HOST=localhost
MYSQL_USERNAME=weaver
MYSQL_PASSWORD=weaver@123
MYSQL_DATABASE=weaver

## Mailer Configuration
SENDER_EMAIL=
SENDER_PASSWORD=

## Project Information
PROJECT_NAME=Project Name
```

## Open access to World!
//Create nginx Server Block file

`vi /etc/nginx/sites-available/weaver`

### "/etc/nginx/sites-available/weaver" File Content:
#### Note: Replace "[DOMAIN OR IP ADDRESS]" with Domain or IP Address.
```bash
server {
    #Listens on port 80 for HTTP requests and treats this server block as the default server.
    listen 80 default_server;
    #Specifies the domain name or IP address for which this server block will handle requests.
    server_name [DOMAIN OR IP ADDRESS];
    #Sets the document root directory to "/var/www/weaver," where web content is served from.
    root /var/www/weaver;
    #Defines the order in which index files (index.php and index.html) are looked for when accessing a directory.
    index index.php index.html;

    #Handles requests to the root path ("/") and performs a URL rewrite for friendly URLs.
    location / {
        rewrite ^/([^/]+)$ /?url=$1 last;
    }

    #Manages requests for PHP files, passing them to the PHP-FPM server for processing.
    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    #Blocks access to hidden files and folders (those starting with a dot) for security.
    location ~ /\. {
        deny all;
    }
}
```

//Create symbolic link to active for the web server to use

`ln -s /etc/nginx/sites-available/weaver /etc/nginx/sites-enabled/`

//Restart Nginx Server

`systemctl restart nginx`


## Secure, a little!

//Set the owner and group of the /var/www/weaver directory and its contents to www-data to grant web server permissions

`chown -R www-data:www-data /var/www/weaver`

//Change the directory's permissions to 700, which restricts access to only the owner while securing the web application.

`chmod -R 700 /var/www/weaver`




## Install SSL Certificate
In "/etc/nginx/sites-available/weaver" file, replace IP with Sub-Domain Name.

***certbot --nginx -d [DOMAIN NAME]***

Example: *certbot --nginx -d weaver.codigion.com*




# TechnicSolder
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
![Latest Stable Version](https://img.shields.io/badge/dynamic/json.svg?label=Latest%20Stable%20Version&url=https%3A%2F%2Fraw.githubusercontent.com%2FTheGameSpider%2FTechnicSolder%2Fmaster%2Fapi%2Fversion.json&query=version&colorB=brightgreen)
![Latest Dev Version](https://img.shields.io/badge/dynamic/json.svg?label=Latest%20Dev%20Version&url=https%3A%2F%2Fraw.githubusercontent.com%2FTheGameSpider%2FTechnicSolder%2FDev%2Fapi%2Fversion.json&query=version&colorB=orange)

TechnicSolder is an API that sits between a modpack repository and the Technic Launcher. It allows you to easily manage multiple modpacks in one single location.

Using Solder also means your packs will download each mod individually. This means the launcher can check MD5's against each version of a mod and if it hasn't changed, use the cached version of the mod instead. What does this mean? Small incremental updates to your modpack doesn't mean redownloading the whole thing every time!

Solder also interfaces with the Technic Platform using an API key you can generate through your account there. When Solder has this key it can directly interact with your Platform account. When creating new modpacks you will be able to import any packs you have registered in your Solder install. It will also create detailed mod lists on your Platform page! (assuming you have the respective data filled out in Solder) Neat huh?

-- Technic

# Installation
> ***Note: If you already have a working web server with mysql and zip extensions and enabled rewrite mod, you can [skip to step 6.](https://github.com/TheGameSpider/TechnicSolder#cloning-technicsolder-repository)***

**1. Install Ubuntu Server 18.04 (https://www.ubuntu.com/download/server)** <br />
**2. Login to Ubuntu with credentials you set.** <br />
**3. Become root**
```bash
sudo passwd root
```
Write your current password, then set new root password
```bash
exit
```
Login as root <br />
## Web server installation and configuration
**4. Install WEB Server**<br />
```bash
apt update
apt -y install apache2 php libapache2-mod-php mysql-server php-mysql php-dev zlib1g-dev
```
The above command can take a while to complete. Now, you need to install PHP ZIP extension.<br />
```bash
cd /
wget http://fr.archive.ubuntu.com/ubuntu/pool/universe/libz/libzip/libzip4_1.1.2-1.1_amd64.deb
wget http://fr.archive.ubuntu.com/ubuntu/pool/universe/libz/libzip/libzip-dev_1.1.2-1.1_amd64.deb
dpkg -i libzip4_1.1.2-1.1_amd64.deb
dpkg -i libzip-dev_1.1.2-1.1_amd64.deb
pecl install zip
nano /etc/php/7.2/apache2/php.ini
```
add `extension=zip.so` to the second line.
When you are finished, save and close the file by pressing Ctrl-X. You'll have to confirm the save by typing Y and then hit Enter to confirm the file save location.
```bash
service apache2 restart
```
Configure MySQL: 
```bash
mysql_secure_installation
```
You will be asked if you want to configure the VALIDATE PASSWORD PLUGIN. Answer **y**<br />
Then you'll be asked to select a level of password validation. Answer **0** and set your mysql password.<br />
You'll be asked for a few more things, answer **y** to all.
```bash
nano /etc/apache2/mods-enabled/dir.conf
```
We want to move the PHP index file to the first position after the DirectoryIndex specification: <br />
`DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm` -> `DirectoryIndex index.php index.html`<br />
Configuration will look like this:
```
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html
</IfModule>
```
When you are finished, save and close the file.
```bash
service apache2 restart
nano /var/www/html/index.php
```
This will open a blank file. We want to put the following text, which is valid PHP code, inside the file:
```php
<?php
phpinfo();
```
When you are finished, save and close the file.<br />
Now we can test whether our web server can correctly display content generated by a PHP script. To try this out, we just have to visit this page in our web browser. You'll need your server's public IP address.
```bash
curl http://icanhazip.com
```
Open in your web browser: `http://your_server_IP_address` <br />
This page basically gives you information about your server from the perspective of PHP. It is useful for debugging and to ensure that your settings are being applied correctly.<br />
<br />
You probably want to remove this file after this test because it could actually give information about your server to unauthorized users. To do this, you can type
```bash
rm /var/www/html/index.php
```
**5. Enable RewriteEngine**<br />
```bash
a2enmod rewrite
nano /etc/apache2/sites-enabled/000-default.conf
```
Add this before `</VirtualHost>` close tag:
```
    <Directory /var/www/TechnicSolder>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Require all granted
    </Directory>
```
Save and close the file
## Cloning TechnicSolder repository
**6. Clone TechnicSolder repository** 
```bash
cd /var/www/
git clone https://github.com/TheGameSpider/TechnicSolder.git
nano /etc/apache2/sites-enabled/000-default.conf
```
Change `DocumentRoot` to `/var/www/TechnicSolder`<br />
Restart apache
```bash
service apache2 restart
```
If you don't have remote access to the server (SSH), you can just download the master branch extract it to your document root folder<br />
Installation is complete. Now you need to confige TechnicSolder before using it.
# If you are using nginx:
*there is an example for nging configuration"*
```nginx
	location / {
        try_files   $uri $uri/ /index.php?$query_string;
        }

	location /api/ {
        try_files   $uri $uri/ /api/index.php?$query_string;
        }

    location ~* \.php$ {
            fastcgi_pass                    unix:/run/php/php7.2-fpm.sock;
            fastcgi_index                   index.php;
            fastcgi_split_path_info         ^(.+\.php)(.*)$;
            include                         fcgi.conf;
            fastcgi_param PATH_INFO         $fastcgi_path_info;
            fastcgi_param SCRIPT_FILENAME   $document_root$fastcgi_script_name;
    }

    location ~ /\.ht {
            deny all;
    }

    location ~ .*/\. {
            return 403;
    }

    error_page 403 /403.html;
    
    location ~* \.(?:ico|css|js|jpe?g|JPG|png|svg|woff)$ {
            expires 365d;
	}
```
# Configuration
**configure MySQL**
```bash
mysql -p -u root
```
Login with your password you set earlier. <br />
Create new user
```MYSQL
CREATE USER 'solder'@'localhost' IDENTIFIED BY 'secret';
```
> **NOTE: By writing *IDENTIFIED BY 'secret'* you set your password. Dont use *secret***

<br />

Create database solder and grant user *solder* access to it.

```MYSQL
CREATE DATABASE solder;
GRANT ALL ON solder.* TO 'solder'@'localhost';
exit
```

**Configure TechnicSolder** <br />

```bash
chown -R www-data TechnicSolder
```

Go to `http://your_server_IP_address` and fill up the form. If you followed these instructions, database name and username is `solder` <br />

The final step is to set your Solder URL in Solder Configuration (In your https://technicpack.net profile)

```
http://your_server_IP_address/api/
```

Click **Link solder**<br />

That's it. You have successfully installed and configured TechnicSolder. It's ready to use!

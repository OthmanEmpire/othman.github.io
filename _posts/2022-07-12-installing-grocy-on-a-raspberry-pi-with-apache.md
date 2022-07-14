---
layout: blogpost
author: Othman Alikhan
published: true
lang: en
title: Installing Bleeding Edge Grocy on Raspberry Pi OS (Debian 11)
category: sysadmin
tags:
  - raspberrypi
  - homeautomation
---


## Overview

We will be installing the latest release of Grocy (as of this article v3.3.1) on a 
Raspberry Pi 3 (running Rasperry Pi OS [Debian 11]).

The setup involves installing the backend PHP dependencies, the frontend javascript 
dependencies, and configuring an Apache server.

This article was inspired by *https://peppe8o.com/manage-your-home-stocks-like-a-pro-with-grocy-and-raspberry-pi/*

## Installing Backend Components

1. Update the OS list of downloadable packages.
    ```
    sudo apt update
    ```

2. Install Apache, SQLite, and PHP8
    ```
    sudo apt install apache2 sqlite3 php8.0
    ```

3. Clone the bleeding edge release of grocy.
    ```
    cd /var/www/
    sudo git clone https://github.com/grocy/grocy.git
    ```

4. Install PHP composer locally in the project (required to install PHP
   libraries automatically). PHP composer is a bit funny in how it works, 
   from official docs, do not download/install composer as root user for
   security reasons! 

   Therefore switch to the regular user that you created when
   you installed the Raspberry Pi OS. You can use `cat etc/passwd | grep -i
   1000` to find the username if you are really lost. We will later adjust
   permissions so only Apache service account can access `grocy/` directory.

    ```
    cd /var/www/grocy/

    # Temporary modifying permissions
    su <REGULAR_USER>
    sudo chown <REGULAR_USER>:<REGULAR_USER> -R /var/www/grocy/

    # Installing composer (Reference: https://getcomposer.org/download/)
    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    php -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
    php composer-setup.php
    php -r "unlink('composer-setup.php');"
    ```

    *QA: You should see a 'composer.phar' binary in your local directory.*

5. Use composer to fix any library version mismatches.
    ```
    php composer.phar update
    ```

6. Use composer to highlight any missing PHP extensions.
    ```
    php composer.phar install
    ```

    - 6.1. You should see a bunch of errors similar to the one below.
        ```
        Problem 1
            - slim/http is locked to version 1.2.0 and an update of this package was not requested.
            - slim/http 1.2.0 requires ext-simplexml * -> it is missing from your system. Install or enable PHP's simplexml extension.
        ```

    - 6.2. For each error, install the relevant library manually. So for the
      error above, `sudo apt install php8.0-simplexml`. Otherwise, just install these 
      ```
      sudo apt install php8.0-gd php8.0-simplexml php8.0-intl
      ```

    *QA: Re-run `php composer.phar install` and you should see no errors.*


## Installing Frontend Components


1. Install `yarn` to download frontend dependencies (install yarn via npm since the apt package 
   is a different tool altogether).
   ```
    sudo apt install npm
    sudo npm install -g yarn
   ```

2. Download frontend dependencies using `yarn`.
   ```
    cd /var/www/grocy
    sudo yarn install
   ```

## Configuring Apache


1. Prepare a grocy config file as mentioned in Grocy docs.
    ```
    cp /var/www/grocy/config-dist.php /var/www/grocy/data/config.php
    ```

2. Change ownership to Apache service account to restrict access.
    ```
    sudo chown www-data:www-data -R /var/www/grocy/
    ```

3. Configure apache to not ignore the .htaccess file in the grocy directory.
    ```
    sudo vi /etc/apache2/apache2.conf
    ```

    7.1. Under the '<Directory /var/www/>' section set AllowOverride from 'None' to 'All'
    ```
    ...
    <Directory /var/www/>
    ...
    AllowOverride All                   # Change this value is changed from 'None' to 'All'
    ...                                 # https://httpd.apache.org/docs/2.4/mod/core.html#allowoverride
    ```

4. Create an Apache config file for the grocy webapp.
    ```
    cd /etc/apache2/sites-available/
    sudo cp 000-default.conf grocy.conf
    sudo vi grocy.conf

    ...
    ServerName <IP>                             # Change to your (static) IP
    DocumentRoot /var/www/grocy/public          # Ensure no trailing slash!
    ...
    ```

5. Check for syntax errors
    ```
    sudo apache2ctl configtest

    ...
    Syntax OK
    ```

6. Enable the Apache sites and restart.
    ```
    sudo a2ensite grocy.conf
    sudo a2enmod rewrite                        # Enables the re-write module.
    sudo a2dissite 000-default.conf             # Disables the unused default Apache page
    sudo systemctl restart apache2
    ```


7. Finally, navigate to `http://<YOUR_IP>` to use your deployed grocy webapp.

## A quick guide for setting up a CentOS-based LAMP stack webserver powered by WordPress

This is a quick guide for setting up LAMP stack and installing  [Wordpress](https://wordpress.com/) on a [CentOS](https://www.centos.org/) machine. In order to follow this guide, you need to have access to CentOS machine either locally or remotely through ssh connection.

[CentOS](https://www.centos.org/) has long (10 years) release support. Each version of CentOS receives feature enhancements and new hardware support until 7 years, and security updates until 10 years after general availability. However, support for point releases (eg. CentOS 6.3) will be no longer available once a new point release (eg. CentOS 6.4) is available for the same version. At the time of this writing 7.3 is the latest version. However, 6.8 is the final point release for version 6. Hence, it is better use CentOS 6.8 since the final point release for version 7 is yet unknown. CentOS is available in two images: minimal and full DVD (contains all packages available for the respective version). The following instructions assume that you have installed the minimal version.

## STEP 1: INITIAL SERVER SETUP
After a fresh install, it is recommended to install updates.
    
    $ sudo yum update'

You may want to change the root password. To do that run the following command.
    
    $ sudo passwd root

The following instructions are optional. You can continue from _**STEP 2**_.

If you want to create a new user account, eg. _john_, then run this command.
    
    $ sudo adduser john

Then you can set password for _john_ (eg 123456) as follows
    
    $ sudo passwd john 123456
    
Even though you login with _john_'s credentials, you still need the root password in order to run some commands (eg. installing packages, making system configuration changes). You can avoid this by adding _john_ into the _sodoers_ group. The _sudoers_ group name in CentOS is _wheel_.
    
    $ sudo gpasswd -a john wheel
 
Now every time you run commands with sudo, you will not be prompted for the root password. Make sure that you use a strong password for any user (especially the ones that are in sudoers group).

## Step 2: LAMP SERVER SETUP

In this step we will install and configure [Apache](https://www.apache.org/) HTTP server, [PHP](http://www.php.net/) and [MariaDB](https://mariadb.org/) database server. MariaDB is an enhanced, drop-in replacement for [MySQL](https://www.mysql.com/) database server. From now on, all commands need to be executed as root (or with sudo privilege). If you don't want to type sudo for ever command, then switch to the root user.

    $ sudo su root
    
Now the command prompt sign changes from **$** to **#**.

First, install Apache server
    
    # yum install httpd
    
Start apache server
    
    # systemctl start httpd.service

Enable apache to start on boot

    # systemctl enable httpd.service

Configuration file for Apache is located at _/etc/httpd/conf/httpd.conf_. Open this file and search for _"DocumentRoot"_. By default, you will see the following

    DocumentRoot "/var/www/html/" ''

This means, Apache will serve websites that are hosted in _/var/www/html_ directory. By default, if Apache doesn't find an index file (eg. index.html), it will show directory listing of all files in _/var/www/html_ directory. This will expose all the files in this director to the public. In order to disable directory listing, open _/etc/httpd/conf/httpd.conf_ file and within `<Directory "/var/www/html">` directive, remove `"Indexes"` from the list of `Options`. Eg. If you find something like the following
    
    Options Indexes, FollowSymLinks

then change it to

    Options FollowSymLinks
i.e remove `Indexes`. You need to restart apache in order to apply configuration changes

    # systemctl restart httpd.service

The next step is to install [MariaDB]("https://mariadb.org/") database server.

    # yum install mariadb-server mariadb

Start the database server

    # systemctl start mariadb

By default, MySQL has EMPTY root password and comes with a test database. Run the following script and follow the instructions to override dangerous DEFAULT and lock down access to the database.

    # mysql_secure_installation

Enable MariaDB to start on boot

    # systemctl enable mariadb.service

Finally, install PHP

    # yum install php php-mysql

Restart Apache

    # systemctl restart httpd.service

## STEP 3: INSTALL WORDPRESS

First, create database for wordpress. To do that, login to mysql server as a root
    
    # mysql -u root -p

Then create a database. For example, the following command creates a database called <em>wp</em>

    > CREATE DATABASE wp;
   
Then create a user account which Wordpress will use to access the database. For example, the following command creates an account with username _wpUser_ and password _wpPass123_

    > CREATE USER wpUser@localhost IDENTIFIED BY 'wpPass123';
   
Grant the user access to the database we created earlier.

    > GRANT ALL PRIVILEGES ON wp.* TO wpUser@localhost IDENTIFIED BY 'wpPass123';

Flush privileges and close the connection.

    > FLUSH PRIVILEGES;
    > exit;
    
Install a module that allows wordpress to create tumbnails for uploaded images

    # yum install php-gd
    # systemctl restart httpd.service

Install _wget_ which is used to download files from the command line. This is a handy tool when you configure your server remotely.

    # yum install wget
    
Then download and extract latest Wordpress source files

    # wget http://wordpress.org/latest.tar.gz
    # tar xzvf latest.tar.gz

Extracted files are located at `wordpress` folder in the current working directory. Copy files from `wordpress/*` to a directory where you want to host your website. By default, Apache serves files that are hosted in`/var/www/html` directory. In order to keep this directory clean, create a sub directory `/var/www/html/wp` and copy Wordpress files to this directory. Use `rsync`to copy the files. This is because `rsync` keeps file permissions.

    # mkdir /var/www/html/wp
    # rsync -avP wordpress/ /var/www/html/wp
    
Create folder in _wp_ to store uploaded files.

    # mkdir /var/www/html/wp/wp-content/uploads
    
Grant ownership of all files in _wp_ to Apache's user and group.

    # chown -R apache:apache /var/www/html/wp/*
    
Create wordpress configuration file (Copy from the sample).

    # cd /var/www/html/wp
    # cp wp-config-sample.php wp-config.php
    
Open _wp-config.php_ and change `DB_NAME`, `DB_USER` and `DB_PASSWORD` accordingly.

Now change Apache's document root directory to the one we created. To do that, open _/etc/httpd/conf/httpd.conf_ and change Apache's document root as follows

    DocumentRoot "/var/www/html/wp"
        
Finally, open a browser and type the server's IP address. If you have local server, then type http://localhost to the URL. Then finish the installation of WordPress throught the web browser.

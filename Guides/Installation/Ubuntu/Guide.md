Ubuntu Web Guide.
============
>This guide was designed for Ubuntu 12.04+ and derivatives (Linux Mint/Elemantary OS/etc.), using this guide on Debian or other Linux Distributions might or might not function. This guide might have errors or be obsolete by the time you read it, doing some research on your part on how to fix issues if they arise is required.

### Notes:
>In this guide you will see commands within grey rectangles, example:
`this is an example`  
When you see these, you must type them into a command line interface (terminal window).

> PHP 5.4 and MySQL 5.5 are the minimum required versions, 
but higher versions (PHP 5.5 and MySQL 5.6 to be exact) are recommended.

### Step 1 *Updating your operating system:*
>Update all your sources:
>>`sudo apt-get update`

---

>Upgrade your applications:
>>`sudo apt-get upgrade`

---

>**Optional:** Upgrade some other stuff (like the kernel).
>>`sudo apt-get dist-upgrade`

---

>You must now reboot your server.

### Step 2 *Installing pre-requisite software:*
>These programs will be used later on to install additional software.  
They might already be installed on your operating system.  
>>`sudo apt-get install software-properties-common`  
`sudo apt-get install python-software-properties`

### Step 3 **[Optional]** *Adding a repository for the newest PHP and Apache:*
>This will give you the latest PHP and Apache, PHP 5.5 is highly recomended, 
if you operating system does not having PHP 5.5, please add this repository.
>>`sudo apt-get install software-properties-common`  
`sudo add-apt-repository ppa:ondrej/php5`  
`sudo apt-get update`

### Step 4 *Installing PHP and the required extensions:*
> Note that some extensions might be missing here, 
see INSTALL.txt in the nZEDb docs folder for all the required extensions.  
>>`sudo apt-get install php5 php5-dev php5-json php-pear php5-gd php5-mysqlnd php5-curl`

### Step 5 **[Mandatory]** *Apparmor:*
>Apparmor restricts certain programs, on nZEDb it stops us from using the 
MySQL LOAD DATA commands. 
You can read more on Apparmor [here](http://en.wikipedia.org/wiki/AppArmor).

>*You have two options*:  

---

>Option 1: Making Apparmor ignore MySQL
>>`sudo apt-get install apparmor-utils`  
`sudo aa-complain /usr/sbin/mysqld`

---

>Option 2: Disabling Apparmor
>>`sudo update-rc.d apparmor disable`

---

>**You MUST reboot your server after doing this!**

### Step 6 *Installing a MySQL server and client:*
>You have multiple choices when it comes to a MySQL server and client,
you will need to do some research to figure out which is the best for you.
We recommend MariaDB for most people.

>[MySQL](http://dev.mysql.com/doc/refman/5.6/en/features.html):
>>`sudo apt-get install mysql-server mysql-client libmysqlclient-dev`

---

>[MariaDB](https://mariadb.com/kb/en/mariadb-versus-mysql-compatibility/):
>>`sudo apt-get install mariadb-server mariadb-client libmysqlclient-dev`

---

>[Percona](http://www.percona.com/software/percona-server/feature-comparison):  

>Add the repo key:
>>`sudo apt-key adv --keyserver keys.gnupg.net --recv-keys 1C4CBDCDCD2EFD2A`  

>Create a text file to put in the repo source:  
>>`sudo nano /etc/apt/sources.list.d/percona.list`

>Paste the deb and deb-src lines, replacing VERSION with the name of your ubuntu: (12.04: precise,
12.10: quantal, 13.04: raring, 13.10: saucy, 14.04: trusty, 14.10: utopic).

>>`deb http://repo.percona.com/apt VERSION main`  
`deb-src http://repo.percona.com/apt VERSION main`

>Exit and save nano (press control+x, then type y and press Enter).

>Update your packages and install percona.
>>`sudo apt-get update`  
`sudo apt-get install percona-server-server-5.5 percona-server-client-5.5 libmysqlclient-dev`

### Step 7 *Configuring MySQL:*
>Edit my.cnf:
>>`sudo nano /etc/my.cnf`

>Add (or change them if they already exist) the following:
>>`max_allowed_packet = 16M`  
`group_concat_max_len = 8192`

>Consider raising the key_buffer_size to 256M

---

>**[Mandatory]** Add file permissions to your MySQL user.

>Log in to MySQL:
>>`sudo mysql -p`

>In the following line, change YourMySQLUsername for the username 
you will use to connect to MySQL in nZEDb (root for example).
Also change the YourMySQLServerIPAddress to the IP of the server 
(127.0.0.1 or localhost for example). Do not remove the single quotes.
>>`GRANT FILE ON *.* TO 'YourMySQLUsername'@'YourMySQLServerIPAddress';`

>Exit MySQL:
>>`\q`

### Step 8 *Installing and configuring a web server:*
>You have many options. We will however show you 2 options, Apache2 or Nginx.

>*[Apache](http://httpd.apache.org/):*  
>>`sudo apt-get install apache2`

>Now you need to check if you have apache 2.2 or apache 2.4, 
they require a different configuration.
>>`apache2 -v`

>**Apache 2.4**:  
>*If you have Apache 2.2, scroll down lower.*

>Create a site configuration file:
>>`sudo nano /etc/apache2/sites-available/nZEDb.conf`

>Paste the following into it (changing your paths as required):

    <VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName localhost
        DocumentRoot "/var/www/nZEDb/www"
        LogLevel warn
        ServerSignature Off
        ErrorLog /var/log/apache2/error.log
        <Directory "/var/www/nZEDb/www">
            Options FollowSymLinks
            AllowOverride All
            Require all granted
        </Directory>
        Alias /covers /var/www/nZEDb/resources/covers
    </VirtualHost>

>Save and exit nano.

>Edit apache to allow overrides in the /var/www folder (or the folder you will put nZEDb into):
>>`sudo nano /etc/apache2/apache2.conf`

>Under `<Directory /var/www/>`, change `AllowOverride None` to `AllowOverride All`

>Save and exit nano.

>Disable the default site, enable nZEDb, enable rewrite, restart apache:
>>`sudo a2dissite 00-default`  
`sudo a2dissite 000-default`  
`sudo a2ensite nZEDb.conf`  
`sudo a2enmod rewrite`  
`sudo service apache2 restart`  

>**Apache 2.2**  
>*Skip this if you have apache 2.4*

>Create a site configuration file:
>>`sudo nano /etc/apache2/sites-available/nZEDb`

>Paste the following into it (changing your paths as required):

    <VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName localhost
        DocumentRoot "/var/www/nZEDb/www"
        LogLevel warn
        ServerSignature Off
        ErrorLog /var/log/apache2/error.log
        <Directory "/var/www/nZEDb/www">
            Options FollowSymLinks
            AllowOverride All
            Order allow,deny
            allow from all
        </Directory>
        Alias /covers /var/www/nZEDb/resources/covers
    </VirtualHost>

>Save and exit nano.

>Disable the default site, enable nZEDb, enable rewrite, restart apache:
>>`sudo a2dissite default`  
`sudo a2ensite nZEDb`  
`sudo a2enmod rewrite`  
`sudo service apache2 restart`

---

>**Nginx:**  
>TODO (will be added in the future)

### Step 9 *Installing extra, optional software:*

>*[Unrar](http://en.wikipedia.org/wiki/Unrar):*

>You can install Unrar from the repositories, but it's quite old (version 4):
>> `sudo apt-get install unrar`

>You can also install it by downloading the newest version:  
>Go to http://www.rarlab.com/download.htm, look for the newest unrar version (currently RAR 5.10 beta 4), right click it and copy the link.  
>Replace the link below with the one you copied:
>>`mkdir -p ~/new_unrar`  
`cd ~/new_unrar`  
`wget http://www.rarlab.com/rar/rarlinux-x64-5.1.b4.tar.gz`
`tar -xzf rarlinux*.tar.gz`  
`sudo mv /usr/bin/unrar /usr/bin/unrar4`  
`sudo mv rar/unrar /usr/bin/unrar`  
`sudo chmod 755 /usr/bin/unrar`  
`cd ~/`  
`rm -rf ~/new_unrar`

---

>*[Mediainfo](http://mediaarea.net/en/MediaInfo):*

>Go to http://mediaarea.net/en/MediaInfo/Download/Ubuntu, download the deb files for 
libmediainfo, libzen0 and mediainfo (CLI)

>You can download by right clicking on them, copy link, then type `wget` and paste the link after.

>Once you have all 3 deb files, install them by typing `sudo dpkg -i` followed by the name of the file.

---

>*[Lame](http://lame.sourceforge.net/):*  
>>`sudo apt-get install lame`

---

>*[FFmpeg](http://www.ffmpeg.org/) or [avconv](http://libav.org/avconv.html):*

>On newer versions of ubuntu (14.04+) you can install avconv:
>>`sudo apt-get install libav-tools`

>You can alternatively install ffmpeg:
>>(manual compilation)  
https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu

>>(automated compilation, possibly unmaintained)  
https://github.com/jonnyboy/installers/blob/master/compile_ffmpeg.sh

---

>*[Memcached](http://memcached.org/)*  
>>`sudo apt-get install memcached php5-memcache`

>After git cloning and seting up the indexer, edit www/config.php, change MEMCACHE_ENABLED to true.

---

>*[yEnc](http://en.wikipedia.org/wiki/YEnc):*

>You have 3 choices:

>simple_php_yenc_decode is a PHP extension which offers the best performance of the 3 options.

>yydecode is an application written in C, it decodes yEnc files, it offers moderate performance.

>Alternatively, nZEDb has a PHP method which decodes yEnc but has very poor performance.

>You can change to any of the three at any time in site-edit part of the site.

>*[simple_php_yenc_decode](https://github.com/kevinlekiller/simple_php_yenc_decode):*

>>`sudo apt-get install git`  
`cd ~/`  
`git clone https://github.com/kevinlekiller/simple_php_yenc_decode`  
`cd simple_php_yenc_decode/`  
`sh ubuntu.sh`  
`cd ~/`  
`rm -rf simple_php_yenc_decode/`

>*[yydecode](http://yydecode.sourceforge.net/):*

>>`cd ~/`  
`mkdir -p yydecode`  
`cd yydecode/`  
`wget http://colocrossing.dl.sourceforge.net/project/yydecode/yydecode/0.2.10/yydecode-0.2.10.tar.gz`  
`tar -xzf yydecode-0.2.10.tar.gz`  
`cd yydecode-0.2.10/`  
`./configure`  
`make`  
`sudo make install`  
`make clean`  
`cd ~/`  
`rm -rf yydecode/`

---

### Step 10 *Acquiring nZEDb:*

>Install git:
>>`sudo apt-get install git`

>Clone the git:
>>`mkdir -p /var/www/`  
`cd /var/www/`  
`sudo git clone https://github.com/nZEDb/nZEDb.git`

>Set the permissions:

>During the install (next step of this guide) you can set perms to 777 to make things easier:
>>`sudo chmod -R 777 /var/www/nZEDb`

>After installation you can properly set your permissions.  
YourUnixUserName is the user you use in CLI.  
You can find this by typing : `echo $USER`  
>>`sudo chown -R YourUnixUserName:www-data /var/www/nZEDb`  
`sudo usermod -a -G www-data YourUnixUserName`  
`sudo chmod -R 774 /var/www/nZEDb`

### Step 11 *Setting up nZEDb:*

>Open up an internet browser, 
head to `http://IpAddressOfYourServer/install` 
changing IpAddressOfYourServer for the IP of your server.

>When you are done, create Amazon/Trakt/Tmdb/etc API keys 
(the keys we provide are used by many people so you will get nothing from them - 
all these services throttle requests).

>Next, head to the edit site section, fill out the keys, 
turn on header compression if your server supports it,
put in paths to optional software, like unrar/ffmpeg/etc..

>Go to the view groups section, turn on a group or 2 (alt.binaries.teevee for example).

>Run the scripts in /var/www/nZEDb/misc/update/ 
(update_binaries to download headers and update_releases to create NZB's from the headers).

>You can also run automated scripts, in /var/www/nZEDb/misc/update/nix

>For questions, check the [FAQ](https://github.com/nZEDb/nZEDb/blob/master/docs/FAQ.txt)/[Wiki](https://github.com/nZEDb/nZEDb/wiki)/[Forum](http://forums.nzedb.com/) or join IRC, server [Synirc](https://www.synirc.net/servers) channel #nZEDb
## GITHUB

Feel free to fork from https://github.com/Protonova/Blue-Economics-New.git

I've included below instructions for replicating the instance on a LAMP stack. Specifically Ubuntu 14.04 LTS Server which you can download from: http://www.ubuntu.com/download/server/thank-you?country=US&version=14.04.3&architecture=amd64 and you may also get VirtualBox from here: https://www.virtualbox.org/wiki/Downloads for experiementing around with this repo.

After you have developed and tested something, you can make a pull request, so your modification would be added to https://github.com/Protonova/Blue-Economics-New I will be making pull requests to master on a regular basis. You can choose to pull request to master directly https://github.com/raynor85/Blue-Economics-New , totally up to you!

## WEBSERVER

Since the website works under PHP and MYSQL, you need to install a webserver.
On a fresh host do the following:

#### Step 1

```
sudo apt-get -y update && sudo apt-get -y dist-upgrade && sudo apt-get -y autoclean && sudo apt-get -y --purge autoremove
```
This is needed to update, upgrade, remove leftovers for both packages and kernal.

#### Step 2
Install important packages!

```
sudo apt-get -y install vim wget ntp rsync open-vm-tools git
```
Note open-vm-tools is only for VMware users.

#### Step 3
Setup a static IP by opening up vim!

```
sudo vim /etc/network/interfaces
```
Put these in depending on your network's configuration.
```
# The Primary network interface
auto eth0
iface eth0 inet static
   address 182.168.1.10
   netmask 255.255.255.0
   network 192.168.1.0
   gateway 192.168.1.1
   dns-nameservers 208.67.220.220
```
Configure DNS
```
sudo vim /etc/resolv.conf
```
In Resolv.conf add the new servers:
```
nameserver 192.168.1.1
nameserver 208.67.220.220
nameserver 4.2.2.1
```

Restart your network service.
```
sudo service networking restart
```

#### Step 4
Modify your HOSTS:
```
sudo vim /etc/hosts
```
Add these:

```
192.168.1.10	BlueEco.home BlueEco BlueEco.local
```

Exit and save then run the below:
```
sudo hostname blueeco.local
sudo /etc/init.d/networking restart
```

#### Step 5
Configure SSH for sFTP or external access

```
sudo vim /etc/ssh/sshd_config
```
In there change the following parameters:
```
Port 22
Protocol 2
LoginGraceTime 15
Banner /etc/issue.net
```
Append at the final line:
```
# Restrict SSH to these users/groups
AllowUsers becom
```
*becom* is the name of my user, you should change it to yours.

#### Step 6
Configure the Firewall if you plan on opening your instance to the world. Run the following commands:

```
sudo apt-get -y install ufw
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 22/tcp
sudo ufw allow 123/udp
sudo ufw allow 3306/tcp
sudo ufw enable
sudo ufw default deny
sudo ufw status verbose
sudo ufw logging on
```

#### Step 7
Install Apache!

```
sudo apt-get -y install apache2
```

Secure Apache:

```
sudo vim /etc/apache2/apache2.conf
```
Change the user and group names to or anything you'd like just make to keep a consistant name/group:
```
User = http-web
Group = http-web
```
Exit and save then open the conf below:

```
sudo vim /etc/apache2/conf-enabled/security.conf
```
Change the following Server Signatures:
```
ServerSignature Off
ServerTokens Prod
TraceEnable Off
```

Make the apache group then add the new user to /bin/nologin and lock it to /var/www/:
```
sudo groupadd http-web
sudo useradd -d /var/www/ -g http-web -s /bin/nologin http-web
```

Enable the apache modules we will need for this project:
```
sudo a2enmod rewrite
sudo a2enmod headers
sudo a2enmod env
sudo a2enmod dir
sudo a2enmod mime
sudo service apache2 restart
```

#### Step 8
Install MariaDB!

Add the latest version of MySQL called MariaDB to our available package repo:
```
sudo apt-get install software-properties-common
sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xcbcb082a1bb943db
sudo add-apt-repository 'deb http://ftp.osuosl.org/pub/mariadb/repo/10.1/ubuntu trusty main'
````
Once the key is imported and the repository added you can install MariaDB with:
```
sudo apt-get -y update && sudo apt-get -y install mariadb-server
```
Now Secure the installation (add root password and remove all present tables/schemas:
```
mysql_secure_installation
```

#### Step 9
Install PHP!
```
sudo apt-get -y install php5 libapache2-mod-php5 php5-mcrypt php5-mysql php5-gd php5-intl php-pear php5-imap php5-memcache php5-xdebug php5-common php-apc php5-xmlrpc php5-cli php5-tidy php5-curl
```
Restart apache:
```
sudo service apache2 restart
```
Test PHP configuration
```
sudo su root
echo -e "<?php\nphpinfo();\n?>"  > /var/www/html/info.php
exit
```
then goto the browser and put in the IP given to your server instance. You should see the standard PHP info page.

Now go and change page index settings
```
sudo vim /etc/apache2/mods-enabled/dir.conf
```
Make it look like this:
```
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html
</IfModule>
```
Restart Apache
```
sudo service apache2 restart
```
Remove the phpinfo page it is no longer needed:
```
sudo rm /var/www/html/info.php
```

#### Step 10
Setup vHost!
Copy the default vhost into your new vhost like so:
```
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/BlueEco.com.conf
```
Now lets make the home directory for our project and fix permissions:
```
sudo mkdir -p /var/www/html/home/public_html/
sudo chown -R http-web:http-web /var/www/html/home/public_html
sudo chmod -R 755 /var/www
```
Now lets disable the default vhost and enable our new vhost:
```
sudo a2dissite 000-default.conf
sudo a2ensite BlueEco.com.conf
```
Now that everything setup, lets actually fill the vhost with out information!
```
sudo vim /etc/apache2/sites-available/BlueEco.com.conf
```
Within the vhost put in all the following:
```
<VirtualHost *:80>
	ServerName BlueEco.local

	ServerAdmin root@localhost
	DocumentRoot /var/www/html/home/public_html

	# Adding this will fix permalinks after install
	<Directory /var/www/html/home/public_html>
		Options All
		AllowOverride All
		Order allow,deny
		Allow from all
	</Directory>

	ErrorLog "${APACHE_LOG_DIR}/BlueEco.com_errors.log"
	CustomLog "${APACHE_LOG_DIR}/BlueEco.com_requests.log" combined
</VirtualHost>
```
Restart Apache:
```
sudo service apache2 restart
```

#### Step 11
Install BlueEconomics repo!
```
cd /tmp
git clone https://github.com/Protonova/Blue-Economics-New.git
sudo rsync -av ./ /var/www/html/home/public_html
rm -rf Blue-Economics-New
```
Fix permissions for the freshly placed files:
```
sudo chown -R http-web:http-web /var/www/html/home/public_html
sudo chmod -R 755 /var/www
```
Install NPM:
```
sudo apt-get -y install npm
sudo npm install gulp -g
```
Install NPM Dependancies:
```
sudo apt-get -y update
sudo apt-get -y install ruby-full rubygems-integration
sudo gem install sass
sudo npm install gulp --save-dev
sudo npm install gulp-ruby-sass gulp-autoprefixer gulp-minify-css gulp-rename --save-dev
sudo npm install webpack gulp-uglify gulp-jshint jshint-stylish gulp-util ng-annotate-webpack-plugin --save-dev
```

#### Step 12
Set up database
```
mysql -u root -p
```
Put in your root password then make the database:
```
CREATE DATABASE bedb_home;
CREATE USER bedb_homeu@localhost IDENTIFIED BY '&KD@BNezTiHVaBPJxkh7Bg3oH^O';
```
*Note* &KD@BNezTiHVaBPJxkh7Bg3oH^O was the password I made for the user. You can use: https://lastpass.com/generatepassword.php to make a secure password.
```
GRANT ALTER, CREATE, CREATE TEMPORARY TABLES, DELETE, DROP, INDEX, INSERT, LOCK TABLES, SELECT, UPDATE ON bedb_home.* to bedb_homeu@localhost;
FLUSH PRIVILEGES;
exit;
```

#### Step 13
Import the SQL dump provided in the git repo you cloned earlier.

Before importing it, make sure you've searched and replaced root with the name you gave your user.

```
mysql -u root -p -h localhost bedb_home < /tmp/Blue-Economics-New/dump/blueeconomics.sql
```

#### Step 14
Change credentials for the SQL connection:
```
/var/www/html/home/public_html/build/config/dev/mysql.ini
```

#### Step 15
Pat your self on the back! Job well done, you now have a perfectly working instance of BlueEconomics!


### FURTHER NOTES

If you have any issues with these instructions or the repo feel free to submit an issue or contact me via Skype (quicker) in the group chat setup by Sujith.
# Setting up a server with MariaDB, Apache2, PHP, phpMyAdmin and FTP services

##  0. Preparation

if on a regular pc

make sure that you have it connected to the internet

if in a vm

go to vms settings network select the network adapter 1 enable if not already and select in attached to and set it to bridget adapter and select your network card and under advanced Promiscuous mode set to allow all and tick cable connected

## 1. The OS

Our base install of debian
```
language English
Country United States
Keymap Americal English
change hostname to identify your server example Foma
setup a domain name if you have to, I do not so blank for me
set a root password that is secure and that only you and the people working on it will remember
setup a new user, give a special username for each user
setup a password for each user
set your timezone
nextup si drive partitioning select guided - use etire disk
select the disk to partition
in partitioning scheme select all files in one partition
		(in the enterprise space you may want to seperate your home, var and tmp partitions)
Select finish partitioning and write changes to disk
select yes on write changes to disks
during the install it will ask to configure the package manager by scanning the cd or dvd, select no
Set the debian archive nirror country to United States and debian archive mirror to deb.debian.org
leave http proxy empty
It will ask to configure popularity-contest select no
A window for Software selection will pop up and select the following
	standard system utilities
	print server
	ssh server
and untick
	Debian desktop enviroment if selected
		(you do not want a gui in some specific cases, in this case it will just make our server slower)
next it will install the grub boot loader to the master boot record, select yes and select your drive next
And with that the install is complete
```
it will reboot

log in

check if you have an ip
```
ip addr
```
and test internet with ping command
```
ping -c 3 google.com
```

## 2. setup sudo for our user

type in su to log in as root and type in the root password
```
apt install sudo
```
Open the sudoers file 
```
nano /etc/sudoers
```
under user privilege specification add your user like this
```
(usernameherewthouttheparenthesis)	ALL=(ALL:ALL) ALL
```
ctrl+s, ctrl+x to save and exit

logout of the root account
```
exit
```
Update the list of packages
```
sudo apt update
```


## 3. Installing MariaDB as MySQL replacement

First, we install MariaDB like this:
```
sudo apt -y install mariadb-server mariadb-client
```
Next, we will secure MariaDB with the mysql_secure_installation command. Run the below command and follow the wizard.
```
mysql_secure_installation
```
The recommended input is shown.
```
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
SERVERS IN PRODUCTION USE! PLEASE READ EACH STEP CAREFULLY!
In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.
```
```
Enter current password for root (enter for none): <-- Hit return
OK, successfully used password, moving on...
```
```
Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.
```
```
Set root password? [Y/n] <-- y
New password: <-- Enter the new password for the MariaDB root user
Re-enter new password: <-- Enter the password again
Password updated successfully!
Reloading privilege tables..
... Success!
```
```
By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them. This is intended only for testing, and to make the installation
go a bit smoother. You should remove them before moving into a
production environment.
```
```
Remove anonymous users? [Y/n] <-- y
... Success!
Normally, root should only be allowed to connect from 'localhost'. This
ensures that someone cannot guess at the root password from the network.
```
```
Disallow root login remotely? [Y/n] <-- y
... Success!
```
```
By default, MariaDB comes with a database named 'test' that anyone can
access. This is also intended only for testing, and should be removed
before moving into a production environment.
```
```
Remove test database and access to it? [Y/n] <-- y
- Dropping test database...
... Success!
- Removing privileges on test database...
... Success!
```
```
Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.
```
```
Reload privilege tables now? [Y/n] <-- y
... Success!
```
```
Cleaning up...
All done! If you've completed all of the above steps, your MariaDB
installation should now be secure.
Thanks for using MariaDB!
```
The MariaDB setup is secured now.

Launch mysql as root
```
sudo mysql -u root -p
```
Now, create a new user test (let’s say), set the password test for the user and grant all the privileges to the user with the following SQL statement:
```
GRANT ALL ON *.* TO 'test'@'localhost' IDENTIFIED BY 'test';
```
For the changes to take effect, run the following SQL statement:
```
FLUSH PRIVILEGES;
```
Now, exit out of the MariaDB shell as follows:
```
\q
```

## 4. Installing Apache web server

Apache is available as a Debian package, therefore we can install it like this:
```
sudo apt install apache2
```
Make apache2 start automatically, restart it and check the status
```
sudo systemctl enable apache2
sudo systemctl restart apache2
sudo systemctl status apache2
```
Next we need our ip address
```
ip addr
```
Apache's default document root is /var/www on Debian, and the configuration file is /etc/apache2/apache2.conf. Additional configurations are stored in subdirectories of the /etc/apache2 directory such as /etc/apache2/mods-enabled (for Apache modules), /etc/apache2/sites-enabled (for virtual hosts), and /etc/apache2/conf-enabled.

## 5. Installing PHP 7.3

We can install PHP and the Apache PHP module as follows:
```
sudo apt -y install php7.3 libapache2-mod-php7.3
```
We must restart Apache afterward:
```
service apache2 restart
```


## 6. Testing PHP / Getting details about your PHP installation

The document root of the default web site is /var/www/html. We will now create a small PHP file (info.php) in that directory and call it in a browser. The file will display lots of useful details about our PHP installation, such as the installed PHP version.

Create a info.php file. This file will show us all the information we need about php
```
sudo nano /var/www/html/info.php
```
/var/www/html/info.php
```
<?php
phpinfo();
```
Now we call that file in a browser (e.g. http://serveriphere/info.php):

As you see, PHP 7.3 is working, and it's working through the Apache 2.0 Handler, as shown in the Server API line. If you scroll further down, you will see all modules that are already enabled in PHP5. MySQL / MariaDB is not listed there which means we don't have MySQL support in PHP5 yet.

## 7. Getting MySQL and MariaDB Support in PHP

To get MySQL support in PHP, we will install the php7.3-mysql package. It's a good idea to install some other PHP modules as well as you might need them for your applications. You can search for available PHP 7 modules like this:
```
apt-cache search php7.3
```
Pick the ones you need and install them like this:
```
sudo apt-get -y install php7.3-mysql php-pear
```
Now restart Apache:
```
sudo systemctl restart apache2
```

## 8. phpMyAdmin

phpMyAdmin is a web interface through which you can manage your MySQL and MariaDB databases. It's a good idea to install it:

```
wget https://files.phpmyadmin.net/phpMyAdmin/5.0.2/phpMyAdmin-5.0.2-all-languages.zip
```

Now, extract the phpMyAdmin archive in the /opt directory with the following command

```
sudo unzip phpMyAdmin-5.0.2-all-languages.zip -d /opt
```

As you can see, a new directory is created in the /opt directory.

```
ls -lh /opt
```

For the sake of simplicity, rename the directory to just phpMyAdmin/ with the following command:

```
sudo mv -v /opt/phpMyAdmin-5.0.2-all-languages /opt/phpMyAdmin
```

Now, change the owner and group of the /opt/phpMyAdmin directory and all the contents of the directory to www-data.
```
sudo chown -Rfv www-data:www-data /opt/phpMyAdmin
```

> What is the www-data user?
>
> The owner and group of the /opt/phpMyAdmin directory and contents of the directory should be changed to www-data.
> For security.
> The files are not world writeable. They are restricted to the owner of the files for writing.
> The web server has to be run under a specific user. That user must exist.

## 9. Configuring Apache for phpMyAdmin:


Now, you have to configure Apache web server in order to use phpMyAdmin. I am going to configure phpMyAdmin on port 9000 in this article.

First, create a new site configuration file for phpMyAdmin with the following command:
```
sudo nano /etc/apache2/sites-available/phpmyadmin.conf
```
Now, type in the following lines to the phpmyadmin.conf file.
```
<VirtualHost *:9000>
ServerAdmin test@localhost
DocumentRoot /opt/phpMyAdmin

<Directory /opt/phpMyAdmin>
Options Indexes FollowSymLinks
AllowOverride none
Require all granted
</Directory>
ErrorLog ${APACHE_LOG_DIR}/error_phpmyadmin.log
CustomLog ${APACHE_LOG_DIR}/access_phpmyadmin.log combined
</VirtualHost>
```
ctrl+s, ctrl+x

Now, you have to tell Apache web server to listen to the port 9000.

To do that, edit the /etc/apache2/ports.conf configuration file with the following command:
```
sudo nano /etc/apache2/ports.conf
```
Now, add the line “Listen 9000” below the "Listen 80" line. Once you’re done, save the configuration file by pressing <Ctrl> + X followed by Y and <Enter>.

Now, enable the phpMyAdmin site that we’ve just configured with the following command:
```
sudo a2ensite phpmyadmin.conf
```
```
sudo systemctl reload apache2
```

## 10. Accessing phpMyAdmin:

Now, you should be able to access phpMyAdmin on port 9000.

From a browser, visit http://serveriphere:9000 and you should see the login page of phpMyAdmin.

If you want to access phpMyAdmin remotely, then replace localhost with the IP address of the Debian 10 machine where you’ve installed phpMyAdmin.

## 11. FTP

Now we will install the FTP daemon that we have chosen. In this case, it is VSFTPD. The advantage of using it is that it is easy to configure and is in the Debian 10 repositories.
```
sudo apt install vsftpd
```
Next, check the service status:
```
sudo systemctl status vsftpd
```
Next, make the service start at startup:
```
sudo systemctl enable vsftpd
```

Configure the FTP server

The VSFTPD configuration can be found at /etc/vsftpd.conf. Before modifying the contents of the configuration file, it is recommended to make a backup. If something goes wrong we will be able to restore and nothing bad will have happened.
```
sudo cp -r  /etc/vsftpd.conf /etc/vsftpd.conf.bak
```
Now, we can start with the work.
```
sudo nano /etc/vsftpd.conf
```
The file is somewhat extensive but it is because it is very well documented. The truth is simple to manipulate. First, make the service “listen” and if you don’t use IPV6, disable it.
```
listen=YES
#listen_ipv6=NO
```
By default, VSFTPD does not let you change the assigned folder. That is, it will only allow the download but not the upload. If you want to change this, leave the following directive like this:
```
write_enable=YES
```
The default port is 20 but you can change it:
```
#connect_from_port_20=YES
listen_port=21
```
Here comes a slightly delicate matter. It is normal that when we use an FTP server we have to create users who use it. The problem is that some users can upload and delete files from other users. This should be avoided. It is necessary to define what each user has his folder and that he cannot modify or see the others.
```
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd.chroot_list
```
Now you have to create the file with the list of users. Note that in this step, I have added a user called test1. That is the name of the user I will create later.
```
 sudo nano /etc/vsftpd.chroot_list
```
```
test
```
The pasv_min_port and pasv_max_port directives define the range of ports that will passively work FTP. These ports must be accessible if you use a Firewall.

if you do not want it, open your configuration file and define this:
```
anonymous_enable=NO
```
Next, reload the VSFTPD service.
```
sudo systemctl restart vsftpd
```
And check the status:
```
sudo systemctl status vsftpd
```

Test the FTP server on Debian 10

Open a web browser and go to ftp://your-server-ip/ and you will see this.

You can also use filezilla to access your ftp server.

## Bonus Final step monitoring our server using glances


on the server 
```
sudo apt install glances
```
```
glances -w &!
```
runs glances in web mode

> the &! part will enable us to run the command as a seperate process and then give us back the propmt to execute commands

this will run glances in server mode

the following will enable glances to autostart
```
sudo systemctl enable glances
```
.bashrc at the end
```
glances -w &!
```
on your machine 

go to your browser and type in url as follows
```
http://serveriphere:61208
```
this will run glances in client mode

You are done!

## Video for reference

[how to setup an epic server](https://www.youtube.com/watch?v=tgsUh6wsjvY&feature=youtu.be)

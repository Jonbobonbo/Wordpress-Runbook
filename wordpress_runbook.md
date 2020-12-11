# How to Install WordPress on Ubuntu 16.04
## Step 1 - Connect to your server
Open the terminal on your computer and connect to your Ubuntu server using the ssh command. Remember, if the server is using a different ssh port than 22 you'll need to include that in the command, as shown here:
```
C:\> ssh user@yourdomain.com -p portnumber
```
## Step 2 - Install Apache
### Now that you're connected to your Ubuntu server you need to install the next part of your LAMP stack, Apache.  
To install Apache use this command line, if you already have Apache installed you can skip to step 3:
```
user@server:/$ sudo apt-get install apache2
```
Enter the root password and then enter _y_ to continue.
Now you need to start apache and enable it to boot when the system starts:
```
user@server:/$ sudo systemctl start apache2
user@server:/$ sudo systemctl enable apache2
```
### Let's configure Apache to be compatible with your website.
First we need to configure Apache to point to your domain:
```
user@server:/$ sudo nano /etc/hosts
```
And now you will need to add the name of your domain with your local host ip to the list:
```
127.0.0.1       localhost
127.0.0.1       ubuntu
127.0.0.1       yourdomain.com
```
Now that your domain has been added to Apache, it needs a file for that site's configuration. You can copy the default conf file for the correct format like this:
```
user@server:/$ sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/yourdomain.com.conf
```
Now open your new conf file with the nano text editor and add the information for where you want your web root directory after **DocumentRoot** and your domain after **ServerName**:
```
<VirtualHost *:80>
    ServerName yourdomain.com
    DocumentRoot /var/www/html/
</VirtualHost>
```
Now disable the default configuration and enable your new site's configuration file and restart Apache:
```
user@server:/$ sudo a2dissite 000-default.conf
user@server:/$ sudo a2ensite yourdomain.com.conf
user@server:/$ sudo systemctl reload apache2
```
If you changed the default root directory in your conf file you will need to update the main Apache configuration file with the new root directory as well:
```
user@server:/$ sudo nano /etc/apache2/apache2.conf
```
And you will want to add a new Directory entry to the list of existing ones with the file path of your new root directory:
```
<Directory /new/file/path>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```
Save and exit nano.
Now restart Apache to update your changes:
```
user@server:/$ sudo systemctl reload apache2
```
If you want to avoid the need for sudo in front of all your commands or you want to ssh with an online text editor you can give yourself root permissions with this command: 
```
user@server:/$ chown -R user var/www
```
Enter the user and the directory that you want root access to. The default root directory is var/www, which is what is shown here. If you have changed the root directory to something else, you'll need to add the permissions to that directory instead.
## Step 3 - Install MYSQL 5.7
### The next part of your LAMP stack is MySQL. You will need to install and configure the database that Wordpress will use.
First you will need to run an update to enable Apache to find the newest version of MySQL:
```
user@server:/$ sudo apt-get update
``` 
Enter your password to continue.  
Now install MySQL:
```
user@server:/$ sudo apt-get install mysql-server
```
MySQL will now install. During the installation it will ask you to set a password for the database. Make sure to remember this password.  
**OPTIONAL**: For added security you should run the secure MySQL installation. Just run this code and answer the prompts: 
```
user@server:/$ mysql_secure_installation
```
## Step 4 - Install PHP 7.2
### The final part of your LAMP stack is PHP. This one is a bit trickier as we need to add a repository to download the file from.
The official repository for PHP on Ubuntu is from Ondřej Surý. This command will add that repository:
```
user@server:/$ sudo add-apt-repository ppa:ondrej/php
```
Press _enter_ when prompted.  
You need to update to make sure you have access to the most recent packages:
```
user@server:/$ sudo apt-get update
```
Now you can install PHP 7.2:
```
user@server:/$ sudo apt-get install php7.2
```
When it prompts you, type _y_ and submit. Once the install is complete check the version of PHP with:
```
user@server:/$ php -v
```
It should show that you are now running PHP 7.2. Well done so far!   
We will need to install 3 different extension packages to PHP in order to use it with the rest of the LAMP stack and Wordpress:
```
user@server:/$ sudo apt-get install php7.2-mysql
user@server:/$ sudo apt-get install php7.2-mbstring
user@server:/$ sudo apt-get install libapache2-mod-php7.2
```
If you get a message saying libapache2-mod-php7.2 is already installed, that's okay.  
Now you need to restart Apache so your changes can take affect:
```
user@server:/$ sudo service apache2 reload
```
## Step 5 - Installing WordPress
### Installing WordPress itself is fairly easy.
Navigate to your root directory, if you're not there already, and get the latest version of Wordpress from wordpress.org using the curl command (if your root directory is different than this, navigate there instead):
```
user@server:/$ cd /var/www
user@server:/var/www$ curl -O https://wordpress.org/latest.zip
```
You will also need a way to unzip the file you just downloaded:
```
user@server:var/www$ sudo apt-get install unzip
```
Unzip the file:
```
user@server:/var/www$ unzip latest.zip
```
You have now installed WordPress into your root directory!
## Step 5 - Set Up
### Let's start by configuring your MySQL database.
Now we have to enter the MySQL monitor with root permissions:
```
user@server:/$ mysql -u root -p
```
Enter your password and press _enter_.  
Now that you're in the MySQL monitor, we need to create a database for WordPress to use:
```
mysql> CREATE DATABASE wordpress;
```
The commands must be in uppercase and all lines must end with a semi-colon.  
Now we need to grant permissions for the user to access the database. For this guide, the user will be _wordpress_ but you can make that whatever you want:
```
mysql> GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpress'@'localhost' IDENTIFIED BY '<your-password>';
mysql> FLUSH PRIVILEGES;
mysql> EXIT;
```
You should now be back to your ssh command line.   
Don't forget to start the MySQL service now:
```
user@server:~$ sudo service mysql start
```
### To make things easier, let's set up our WordPress config file.
If you don't have a wp-config.php file when you visit your wordpress site for the first time, you will need to copy the code they give you and make one anyway. To avoid that extra step, navigate to your wordpress directory and change the name of the sample file:
```
user@server:~$ cd /var/www/wordpress
user@server:/var/www/wordpress$ sudo mv wp-config-sample.php wp-config.php
```
Now edit the config file with a text editor, I like to use nano, to tell WordPress to use the database you created with your MySQL:
```
user@server:/var/www/wordpress$ sudo nano wp-config.php
```
And change the sample database info to this:
```
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wordpress' );
define( 'DB_PASSWORD', '<your-password>' );
define( 'DB_HOST', 'localhost' );
define( 'DB_CHARSET', 'utf8' );
define( 'DB_COLLATE', '' );
```
## Step 6 - The Website
Navigate to your server's domain in your preferred web browser and remember to add the /wordpress filepath:
```
yourdomain.com/wordpress
```
Now just enter your information into the input fields and WordPress will do the rest.
## Congratulations, you have successfully installed WordPress!

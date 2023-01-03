
# Deploy-PHP-Laravel-Application-on-Linux-Centos-7-VPS-Server-Guidelines-
Create **Digital ocean** instance on https://m.do.co/c/fa76dfd4fd84 and get **200 USD Free Credit** !
# Initial Setup

> Here we will setup swap file and other necessary things before
> deploying laravel application

### Create Swap File
Check whether you have swap file or not. Run following command. 
	

    free -m
 Or
   

     swapon -s 
If you see nothing or swap data empty, then create a swap file first. Run following commands.

    dd if=/dev/zero of=/swapfile count=1024 bs=1M
Here we may create a swapfile having double size of the physical ram. So here in the above command count=1024 is used for 1GB size. If you need 2GB then count will be 2048 and so on. 
Now  setup the swap file we just created 

chmod 600 /swapfile

    mkswap /swapfile
Enable swapfile 

    swapon /swapfile
Now check the swap size again

    free -m
To enable swapfile after rebooting every time we need to edit the file system table file 

    nano /etc/fstab
Add the following line at the end of the file
> /swapfile   none    swap    sw    0   0






Now save by pressing ctrl + x, then press y and enter. 
If you see any error opening with nano can use vim or install **nano** if not available by following command 

    sudo yum install nano

## Install and configure Apache Server, PHP and its extensions, phpmyAdmin 
Now on centos 7 update repository to get the latest packages by running following commands. 

    rpm -Uvh https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-13.noarch.rpm
    
    rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
    
    sudo yum update 
   
   
   
How to Install EPEL repository on CentOS Stream 8
Step 1: To install EPEL repo on CentOS Stream 8, run the below command:

# dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

[OR]

# dnf config-manager --set-enabled PowerTools
# dnf install epel-release

Step 2: Update the software packages

dnf update

Install the Extra Packages for Enterprise Linux (EPEL) and EPEL-Next repositories:

# dnf config-manager --set-enabled crb
# dnf install epel-release epel-next-release -y

Upgrade the system, and reboot the machine to apply all updates:

# dnf upgrade --refresh -y
# dnf autoremove
# shutdown -r now

Install and Configure Apache
Install Apache and the required Apache modules:

$ sudo dnf install httpd httpd-tools mod_ssl mod_dav_svn -y
Disable the welcome page:

$ sudo sed -i 's/^/#&/g' /etc/httpd/conf.d/welcome.conf
Prevent Apache from exposing files in the document root directory:

$ sudo sed -i "s/Options Indexes FollowSymLinks/Options FollowSymLinks/" /etc/httpd/conf/httpd.conf
Start the Apache service and make it start on boot:

$ sudo systemctl start httpd.service
$ sudo systemctl enable httpd.service
4. Install and Configure Apache Subversion
Install Apache Subversion:

$ sudo dnf install subversion subversion-tools -y
$ svnserve --version
Specify the root directory of SVN repositories:

$ sudo mkdir /srv/svn
Create the first SVN repository named repo001:

$ sudo svnadmin create /srv/svn/repo001
$ sudo chown -R apache:apache /srv/svn/repo001
Create another SVN repository named repo002, if necessary:

$ sudo svnadmin create /srv/svn/repo002
$ sudo chown -R apache:apache /srv/svn/repo002
Set up a password database file /srv/svn/passwd to store user credentials. For example, the username is user001 for the first user, and the password is pass001.

$ sudo htpasswd -bcm /srv/svn/passwd user001 pass001 && history -d -1
$ sudo chown root:apache /srv/svn/passwd
$ sudo chmod 640 /srv/svn/passwd
If you need more users, create them the same way, but don't use the -c flag, which erases the credential file. For example, to create users user002, user003, and user004, use these commands:

$ sudo htpasswd -bm /srv/svn/passwd user002 pass002 && history -d -1
$ sudo htpasswd -bm /srv/svn/passwd user003 pass003 && history -d -1
$ sudo htpasswd -bm /srv/svn/passwd user004 pass004 && history -d -1
Set up a file /srv/svn/authz to store user permissions:

$ sudo cp /srv/svn/repo001/conf/authz /srv/svn/authz
For this example, assume that:

User001 is the administrator who has read and write permissions on all SVN repositories.
User002 and user003 are qualified users who have read and write permissions on both repo001 and repo002.
User004 is a trainee who has only read permission on repo002.
Open the user permissions file with the Nano editor:

$ sudo nano /srv/svn/authz
Edit the file as follows to grant the permissions mentioned above:

[groups]
admin = user001
user = user002, user003
trainee = user004

[/]
@admin = rw

[repo001:/]
@user = rw

[repo002:/]
@user = rw
@trainee = r
Press CTRL+O, ENTER, and CTRL+X to save the file and quit.

Forbid anonymous access to both SVN repositories:

$ sudo sed -i "s;# anon-access = read;anon-access = none;" /srv/svn/repo001/conf/svnserve.conf
$ sudo sed -i "s;# anon-access = read;anon-access = none;" /srv/svn/repo002/conf/svnserve.conf
Grant write permission on both SVN repositories to authorized users:

$ sudo sed -i "s;# auth-access = write;auth-access = write;" /srv/svn/repo001/conf/svnserve.conf
$ sudo sed -i "s;# auth-access = write;auth-access = write;" /srv/svn/repo002/conf/svnserve.conf
Specify the password database file for both SVN repositories:

$ sudo sed -i "s;# password-db = passwd;password-db = /srv/svn/passwd;" /srv/svn/repo001/conf/svnserve.conf
$ sudo sed -i "s;# password-db = passwd;password-db = /srv/svn/passwd;" /srv/svn/repo002/conf/svnserve.conf
Specify the user permissions file for both SVN repositories:

$ sudo sed -i "s;# authz-db = authz;authz-db = /srv/svn/authz;" /srv/svn/repo001/conf/svnserve.conf
$ sudo sed -i "s;# authz-db = authz;authz-db = /srv/svn/authz;" /srv/svn/repo002/conf/svnserve.conf
Define permission scope for both SVN repositories:

$ sudo sed -i "s;# realm = My First Repository;realm = /srv/svn/repo001;" /srv/svn/repo001/conf/svnserve.conf
$ sudo sed -i "s;# realm = My First Repository;realm = /srv/svn/repo002;" /srv/svn/repo002/conf/svnserve.conf
Specify the default root directory of all SVN repositories:

$ sudo sed -i "s;/var/svn;/srv/svn;" /etc/sysconfig/svnserve
Disable SELinux:

$ sudo setenforce 0
$ sudo sed -i "s/SELINUX=enforcing/SELINUX=disabled/" /etc/selinux/config
Start the SVN server and make it automatically start on boot:

$ sudo systemctl start svnserve.service
$ sudo systemctl enable svnserve.service
Define an available Apache virtual host:

$ sudo mkdir /etc/httpd/sites-available
$ sudo mkdir /etc/httpd/sites-enabled
$ sudo nano /etc/httpd/sites-available/svn.example.com.conf
Here after updating, we will install apache from remi,epel repository 
### Install apache2 server
   
  sudo dnf upgrade --refresh -y
  
  
### Install  mysql server [mariadb]
sudo dnf install mysql mysql-server -y

sudo dnf install mysql mysql-server mysql-devel -y
mysql --version

sudo systemctl enable mysqld --now

To stop the MySQL service:

sudo systemctl stop mysqld
To start the MySQL service:

sudo systemctl start mysqld
To disable the MySQL service at system startup:

sudo systemctl disable mysqld
To activate the MySQL service at system startup:

sudo systemctl enable mysqld
To restart the MySQL service:

sudo systemctl restart mysqld



### Install PHP 7.4 and some required extensions on centos 7 
Before installing php, we will install yum utils which can handle additional package's extensions. 

    sudo yum -y install yum-utils

Enable php 7.4 repository 

    sudo yum-config-manager --enable remi-php74
Now run the following command.

    sudo yum install php  php-cli php-fpm php-mysqlnd php-zip php-devel php-gd php-mcrypt php-mbstring php-curl php-xml php-pear php-bcmath php-json
You can add or remove extensions depending on your requirements. 
Restart apache/ httpd server 

     service httpd restart
### Install phpmyAdmin 
Run this command to install latest phpmyadmin 

    yum --enablerepo=remi install phpmyadmin

If you try to access phpmyAdmin by *serveripaddress/phpmyadmin* 
You will see some error. We need to allow access from outside. 
Edit phpmyadmin configuration file by running following command 

    sudo nano /etc/httpd/conf.d/phpMyAdmin.conf
    
Now change the following block by *<Directory /usr/share/phpMyAdmin/>*

      <Directory /usr/share/phpMyAdmin/>
       AddDefaultCharset UTF-8
       Require all granted
      </Directory>
	
Now restart apache server 

    sudo systemctl restart httpd.service
Now you can login remotely. Visit *serveripaddress/phpmyadmin* 
If you can't login using username and password of mysql, update your user's native password by running 

    mysql -uroot -p 
    
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';

### Test Simple PHP Script 
Run the following command 

    nano /var/www/html/test/index.php
Add the following lines in the *index.php* file 

    <?php 
    phpinfo();
    ?>
To access this you need to tell apache server the virtual host configuration. 
Now create a test configuration file on **/etc/httpd/conf.d/** directory 
Create **test.conf** and Add the following lines.

    <VirtualHost *:80>
           ServerName youdomain.com
           DocumentRoot /var/www/html/test
           <Directory /var/www/html/test>
                  AllowOverride All
           </Directory>
    </VirtualHost>
Now restart apache server 

    sudo systemctl restart httpd.service

Now try to access `youdomain.com` if you have configured your Domain's DNS's **blank**  and **www** A record pointing to your server ip. 
Congratulations you have succesfully configured php. Now to test mysql connection edit the /var/www/html/test/index.php file 

      <?php
    $mysqli = new mysqli("localhost","root","123456");
    
    // Check connection
    if ($mysqli -> connect_errno) {
      echo "Failed to connect to MySQL: " . $mysqli -> connect_error;
      exit();
    }
    else {
    echo "connected";
    }
    ?> 
Now save and reload the website. If you see **connected**, then you are good to go.

# # Deploy Laravel Application 
Now its time to deploy our laravel application. First we will install composer and git. 
### Install composer 

    curl -sS https://getcomposer.org/installer | php
    mv composer.phar /usr/bin/composer
    chmod +x /usr/bin/composer
### Install git 

    sudo yum -y install git
Now you can clone or upload your web applications file on /var/www/html directory in appropriate folder. 
For example we will simply just create an application to test login registration functionality. 

    composer create-project --prefer-dist laravel/laravel testapp "5.8.*"
#### Set file permission 

    chown -R apache.apache /var/www/html/testapp 
    chmod -R 755 /var/www/html/testapp 
    chmod -R 755 /var/www/html/testapp/storage
    chcon -R -t httpd_sys_rw_content_t /var/www/html/testapp/storage

Configure the .env file with database credentials. Run migrations 

    php artisan migrate 
To access using your domain , edit the test.conf file /etc/httpd/conf.d directory. You can add more conf files too.

    <VirtualHost *:80>
           ServerName yourdomain.com
           DocumentRoot /var/www/html/testapp/public
    
           <Directory /var/www/html/testapp>
                  AllowOverride All
           </Directory>
    </VirtualHost>
Here we added the document root which is the root directory of your laravel project's root directory. 
Now restart apache server 

    service httpd restart 
Now if you try to access the application by your domain you can access, But when trying to registering, you may get an **sql permission denied** error. To solve this run following command. 

    sudo -P setsebool httpd_can_network_connect_db 1
    
It will enable apache to connect with database. 

# Enjoy ! Keep eyes on this doc. 
### Future Updates 
- Configure Cloudflare SSL certificate
- Configure  SSH 
- Change phpmyadmin address 

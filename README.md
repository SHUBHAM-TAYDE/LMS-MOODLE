# Installing Moodle on Ubuntu
explains how to install Moodle, a Learning Management System (LMS), on an Ubuntu server. It includes instructions for setting up the required software and configuring Moodle.

### Prerequisites
> [!NOTE]
> Make sure you have the following installed:
> - A fresh installation of Ubuntu (Ubuntu 20.04 LTS or later recommended).
> - Root or Sudo access on the server.
> - A public IP address (or domain) for your Moodle instance.
> - A MySQL or MariaDB database server.

1. Update the System
   Before starting, update your system's package index and upgrade any outdated packages.
```bash
sudo apt update && sudo apt upgrade -y
```
2. Install Apache, MySQL, and PHP
   Moodle requires a web server (Apache), a database server (MySQL/MariaDB), and PHP with additional extensions.
   ### Install Apache:
```bash
sudo apt install apache2 -y
```
   ### Install MySQL:
```bash
sudo apt install mysql-server -y
```
   ### Install PHP and Required Extensions:
```bash
sudo apt install php php-mysql libapache2-mod-php php-gd php-xmlrpc php-intl php-json php-cli php-xml php-zip php-curl php-mbstring php-soap php-bz2 php-xmlrpc php-ldap php-mysqli php-imagick -y
```

3. Configure MySQL for Moodle
   Next, create a database and user for Moodle.
   ### Access MySQL:
```bash
sudo mysql -u root -p
```
   ### Create a Moodle database and user:
```bash
CREATE DATABASE moodle;
CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON moodle.* TO 'moodleuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
Note: Replace 'your_password' with a strong password.

4. Download and Install Moodle
   Download the latest stable version of Moodle:
```bash
cd /var/www/html
sudo wget https://download.moodle.org/stable/latest.tgz
sudo tar -xvzf latest.tgz
```
   Set the correct permissions:
```bash
sudo chown -R www-data:www-data /var/www/html/moodle
sudo chmod -R 755 /var/www/html/moodle
```
5. Create Moodle Data Directory
   Moodle requires a separate directory to store data files.
```bash
sudo mkdir /var/moodledata
sudo chown -R www-data:www-data /var/moodledata
sudo chmod 770 /var/moodledata
```
6. Configure Apache for Moodle
   Create a new Apache site configuration file for Moodle.
```bash
sudo vim /etc/apache2/sites-available/moodle.conf
```
Add the following content:
> [!IMPORTANT]
> Replace your_domain_or_IP with your domain or server's public IP address
```bash
<VirtualHost *:80>
    DocumentRoot /var/www/html/moodle
    ServerName your_domain_or_IP

    <Directory /var/www/html/moodle>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Enable the site configuration:
```bash
sudo a2ensite moodle.conf
```
Enable mod_rewrite:
```bash
sudo a2enmod rewrite
```
Restart Apache:
```bash
sudo systemctl restart apache2
```

7. Run Moodle Installation
1. Open your web browser and go to your Moodle site (e.g., http://your_domain_or_IP).
2. Follow the on-screen instructions:
Choose your language.
Configure the database settings (use the database, user, and password created earlier).
Set up an admin user and site details.


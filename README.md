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


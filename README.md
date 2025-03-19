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

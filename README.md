# osTicket
Installing osTicket system
# Installing osTicket on an Intel NUC

## Overview

This guide details how to install osTicket, a ticketing system, on an Intel NUC using Ubuntu Server. It includes configurations for MariaDB as the database, PHP for fetching data, Apache as the webserver, and Fail2Ban for added security against malicious IPs.

---

## Prerequisites

### Hardware

- **Intel NUC**
  - 16GB RAM (recommended)
  - 500GB storage (or sufficient for your needs)

### Software

- Ubuntu Server (latest version)
- osTicket package
- MariaDB (database server)
- PHP (with necessary extensions)
- Apache (web server)
- Fail2Ban (security)

---

## Installation Steps

### Step 1: Prepare the Ubuntu Server

1. Install Ubuntu Server on the Intel NUC.
2. Update the system:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
3. Set a static IP for the server (optional but recommended).

---

### Step 2: Install Apache Web Server

1. Install Apache:
   ```bash
   sudo apt install apache2 -y
   ```
2. Enable Apache to start on boot:
   ```bash
   sudo systemctl enable apache2
   ```
3. Adjust the firewall to allow HTTP and HTTPS traffic:
   ```bash
   sudo ufw allow 'Apache'
   sudo ufw enable
   ```

---

### Step 3: Install PHP and Extensions

1. Install PHP and required extensions:
   ```bash
   sudo apt install php php-cli php-mysql php-imap php-intl php-apcu php-gd php-mbstring php-xml php-curl unzip -y
   ```
2. Confirm PHP installation:
   ```bash
   php -v
   ```

---

### Step 4: Install MariaDB Database Server

1. Install MariaDB:

   ```bash
   sudo apt install mariadb-server -y
   ```

2. Secure the database server:

   ```bash
   sudo mysql_secure_installation
   ```

   - Set a strong root password.
   - Remove anonymous users and test databases as prompted.

3. Create the osTicket database and user:

   ```bash
   sudo mysql -u root -p
   CREATE DATABASE osticket;
   CREATE USER 'osticket_user'@'localhost' IDENTIFIED BY 'your_secure_password';
   GRANT ALL PRIVILEGES ON osticket.* TO 'osticket_user'@'localhost';
   FLUSH PRIVILEGES;
   EXIT;
   ```

---

### Step 5: Download and Configure osTicket

1. Download osTicket:
   ```bash
   wget https://github.com/osTicket/osTicket/releases/download/v1.17/osTicket-v1.17.zip
   ```
2. Extract the osTicket files:
   ```bash
   unzip osTicket-v1.17.zip -d /var/www/osticket
   ```
3. Set permissions:
   ```bash
   sudo chown -R www-data:www-data /var/www/osticket
   sudo chmod -R 755 /var/www/osticket
   ```
4. Configure Apache for osTicket:
   ```bash
   sudo nano /etc/apache2/sites-available/osticket.conf
   ```
   Add the following content:
   ```apache
   <VirtualHost *:80>
       ServerAdmin admin@example.com
       DocumentRoot /var/www/osticket
       ServerName osticket.local

       <Directory /var/www/osticket>
           Options FollowSymLinks
           AllowOverride All
           Require all granted
       </Directory>

       ErrorLog ${APACHE_LOG_DIR}/osticket_error.log
       CustomLog ${APACHE_LOG_DIR}/osticket_access.log combined
   </VirtualHost>
   ```
5. Enable the configuration and restart Apache:
   ```bash
   sudo a2ensite osticket.conf
   sudo systemctl restart apache2
   ```

---

### Step 6: Finalize osTicket Setup

1. Open your browser and go to the server’s IP or hostname (e.g., `http://your-server-ip`).
2. Follow the osTicket web-based installer to:
   - Connect to the MariaDB database.
   - Create an admin account.
   - Finalize installation.
3. Rename the `setup` directory:
   ```bash
   sudo mv /var/www/osticket/setup /var/www/osticket/setup.bak
   ```

---

### Step 7: Set Up Fail2Ban for Security

1. Install Fail2Ban:
   ```bash
   sudo apt install fail2ban -y
   ```
2. Configure Fail2Ban for Apache:
   ```bash
   sudo nano /etc/fail2ban/jail.local
   ```
   Add the following:
   ```ini
   [apache]
   enabled = true
   port = http,https
   filter = apache-auth
   logpath = /var/log/apache2/*error.log
   maxretry = 5
   bantime = 3600
   ```
3. Restart Fail2Ban:
   ```bash
   sudo systemctl restart fail2ban
   ```

---

## Troubleshooting

- **osTicket web page not loading**: Check Apache logs (`/var/log/apache2/error.log`).
- **Database connection issues**: Verify MariaDB credentials and ensure the database server is running.
- **Fail2Ban not banning IPs**: Test the configuration with a simulated login failure and check logs.

---

## Conclusion

You have successfully set up osTicket on an Intel NUC running Ubuntu Server. This setup provides a reliable ticketing system with added security using Fail2Ban. Enjoy your new helpdesk system!


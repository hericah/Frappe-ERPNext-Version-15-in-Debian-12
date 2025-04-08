# Frappe-ERPNext Version-15 in Debian 12 
#### Update repository database link
      sudo apt update
      
#### Upgrade system package to latest stable version:
      sudo apt upgrade
      
#### Create new user:
      useradd -m -s /bin/bash (New-Frappe-User)
      usermod -aG sudo (New-Frappe-User)
      su (New-Frappe-User)
      cd /home/(New-Frappe-User)

#### Reboot system:
      sudo reboot
-----


### STEP 1 Install dependencies
    sudo apt-get install git python3-dev python3.11-venv python3-setuptools python3-pip python3-virtualenv software-properties-common

### STEP 2 Install MariaDB
    sudo apt install mariadb-server
    sudo mysql_secure_installation
    
    
      In order to log into MariaDB to secure it, we'll need the current
      password for the root user. If you've just installed MariaDB, and
      haven't set the root password yet, you should just press enter here.

      Enter current password for root (enter for none): # PRESS ENTER
      OK, successfully used password, moving on...
      
      
      Switch to unix_socket authentication [Y/n] Y
      Enabled successfully!
      Reloading privilege tables..
       ... Success!
 
      Change the root password? [Y/n] Y
      New password: 
      Re-enter new password: 
      Password updated successfully!
      Reloading privilege tables..
       ... Success!

      Remove anonymous users? [Y/n] Y
       ... Success!
 
       Disallow root login remotely? [Y/n] Y
       ... Success!

       Remove test database and access to it? [Y/n] Y
       - Dropping test database...
       ... Success!
       - Removing privileges on test database...
       ... Success!
 
       Reload privilege tables now? [Y/n] Y
       ... Success!

### STEP 4 Edit the mariadb configuration ( unicode character encoding )

    sudo vi /etc/mysql/mariadb.conf.d/51-server.cnf

add this to the 51-server.cnf file

    
    [server]
    user = mysql
    pid-file = /run/mysqld/mysqld.pid
    socket = /run/mysqld/mysqld.sock
    basedir = /usr
    datadir = /var/lib/mysql
    tmpdir = /tmp
    lc-messages-dir = /usr/share/mysql
    bind-address = 127.0.0.1
    query_cache_size = 16M
    log_error = /var/log/mysql/error.log
    
    [mysqld]
    innodb-file-format=barracuda
    innodb-file-per-table=1
    innodb-large-prefix=1
    character-set-client-handshake = FALSE
    character-set-server = utf8mb4
    collation-server = utf8mb4_unicode_ci      
     
    [mysql]
    default-character-set = utf8mb4

Now press (Ctrl-X) to exit

    sudo systemctl restart mariadb

### STEP 5 install Redis
    
    sudo apt-get install redis-server

### STEP 6 install Node.js 20.X package

    sudo apt install curl 
    curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
    source ~/.profile
    nvm install 20

### STEP 7  install Yarn

    sudo apt-get install npm

    sudo npm install -g yarn

### STEP 8 install wkhtmltopdf

    sudo apt-get install xvfb libfontconfig wkhtmltopdf
    

### STEP 9 install frappe-bench

    sudo pip3 install frappe-bench --break-system-packages
    
    bench --version
    
### STEP 10 initilise the frappe bench & install frappe latest version 

    bench init frappe-bench --frappe-branch version-15
    
    cd frappe-bench/
    bench start
    
### STEP 11 create a site in frappe bench 
    
    bench new-site subdomain.mydomain.com
    
    bench --site subdomain.mydomain.com add-to-hosts

### STEP 15 install ERPNext latest version in bench & site

    
    bench get-app erpnext --branch version-15
    ###OR
    bench get-app https://github.com/frappe/erpnext --branch version-15

    bench --site subdomain.mydomain.com install-app erpnext

    bench use subdomain.mydomain.com
    
    bench start

Open url http://[Server IP]:8000 to login 
    
---
# Setting ERPNext for Production

### Install ansible
    sudo pip3 install ansible --break-system-packages

### Enable scheduler
    bench --site subdomain.mydomain.com enable-scheduler
    
### Disable maintenance mode
    bench --site subdomain.mydomain.com set-maintenance-mode off

### Setup production config
    sudo bench setup production erpnext

### Setup NGINX to apply the changes
    sudo bench setup nginx
    
#### Restart Supervisor and Launch Production Mode
    sudo supervisorctl restart all 
    sudo bench setup production erpnext

---
# Domain name and SSL
### Install certbot
    sudo apt install --reinstall certbot python3-certbot

### Enable multitenancy ERPNext
    bench config dns_multitenant on

### Refresh NGINX
    bench setup nginx
    sudo systemctl restart nginx

### Add domain
    bench setup add-domain subdomain.mydomain.com

### Add A Record in domain managament dashboard
    Open domain provider panel, example cloudflare
    Create new 'A' record, set IPv4 to ERPNext VPS server's

### Intiate SSL Certificate
    sudo bench setup lets-encrypt subdomain.mydomain.com

---
# Fixing Layout Not Rendering Properly
## Solution 1
### Check the NGINX error log
    tail -n 100 -f /var/log/nginx/error.log
    
### Change permission of the home directory
    chmod -R o+rx /home/[username, ex: erpnext]
    
## Solution 2
### Rebuild website assets
    bench build
    
### If during build encouter OUT OF MEMORY error, stop first some services
    sudo supervisorctl stop all
    sudo systemctl stop mariadb

### Ensure all database schema is up to date
    bench migrate

### Restart all required services
    sudo systemctl start mariadb
    sudo supervisorctl restart all

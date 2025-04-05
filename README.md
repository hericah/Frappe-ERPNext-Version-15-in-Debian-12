# Frappe-ERPNext Version-15 in Debian 12 
#### Update repository database link
      sudo apt update
      
#### Upgrade system package to latest stable version:
      sudo apt upgrade
      
#### Create new user:
      sudo adduser --home /home/(New-Frappe-User) (New-Frappe-User)
      usermod -aG sudo (New-Frappe-User)
      su (New-Frappe-User)
      cd /home/(New-Frappe-User)

#### Reboot system:
      sudo reboot
-----


### STEP 1 Install git
    sudo apt-get install git

### STEP 2 install python-dev

    sudo apt-get install python3-dev

### STEP 3 Install setuptools and pip (Python's Package Manager).

    sudo apt-get install python3-setuptools python3-pip

### STEP 4 Install virtualenv
    
    sudo apt install python3-virtualenv
    

### STEP 5 Install MariaDB

    sudo apt-get install software-properties-common
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

 
### STEP 6  MySQL database development files

    sudo apt-get install mariadb-client

### STEP 7 Edit the mariadb configuration ( unicode character encoding )

    sudo nano /etc/mysql/mariadb.conf.d/51-server.cnf

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

### STEP 8 install Redis
    
    sudo apt-get install redis-server

### STEP 9 install Node.js 20.X package

    sudo apt install curl 
    curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
    source ~/.profile
    nvm install 20

### STEP 10  install Yarn

    sudo apt-get install npm

    sudo npm install -g yarn

### STEP 11 install wkhtmltopdf

    sudo apt-get install xvfb libfontconfig wkhtmltopdf
    

### STEP 12 install frappe-bench

    sudo pip3 install frappe-bench --break-system-packages
    
    bench --version
    
### STEP 13 initilise the frappe bench & install frappe latest version 

    bench init frappe-bench --frappe-branch version-15
    
    cd frappe-bench/
    bench start
    
### STEP 14 create a site in frappe bench 
    
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
    sudo supervisorctl restart all sudo bench setup production erpnext

    

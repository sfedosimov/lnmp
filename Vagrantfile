# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # box name.  Any box from vagrant share or a box from a custom URL.
  config.vm.box = "ubuntu/trusty64"

  # box modifications, including memory limits and box name.
  config.vm.provider "virtualbox" do |vb|
     vb.name = "LNMP7"
     vb.memory = 4096
   vb.cpus = 2
  end

  ## IP to access box
  config.vm.network "private_network", ip: "192.168.192.168"
  config.vm.hostname = 'site.dev'
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true

  # If you want to share an additional folder, (such as a project root).
  # If you experience slow throughput or performance on the folder share, you
  # might have to use an OS specific share.
  #
  # Default Share:
  # config.vm.synced_folder "../data", "/vagrant_data"
  #
  # NFS Share, Good for Unix based OS (Mac OSX, Linux):
  # config.vm.synced_folder ".", "/vagrant", type: "nfs"
  #
  # SMB Share, good for Windows:
  # (If experiencing issues, upgrade PowerShell to V 3.0)
  # config.vm.synced_folder ".", "/vagrant", type: "smb"

#, create: true

  config.vm.synced_folder ".", "/var/www/html/site", create: true, type: "nfs"
  config.nfs.map_uid = Process.uid
  config.nfs.map_gid = Process.gid
  config.bindfs.bind_folder "/var/www/html/site", "/var/www/html/site"

  ## Bootstrap script to provision box.  All installation methods can go here.
  config.vm.provision "shell", inline: <<-SHELL
    HTTP_PATH="/var/www/html/site"
    PHP_INI_PATH="/etc/php/7.0/fpm/php.ini"
    MYSQL_CFG_PATH="/etc/mysql/mysql.conf.d/mysqld.cnf"
    MYSQL_PASSWORD="password1"

    echo "####################################################################"
    echo "############################# Edit PS1 #############################"
    echo "####################################################################"
    echo '
    PS1="\\[\\033[38;5;130m\\]\\u\\[$(tput sgr0)\\]\\[\\033[38;5;22m\\]@\\[$(tput sgr0)\\]\\[\\033[38;5;130m\\]\\h\\[$(tput sgr0)\\]\\[\\033[38;5;94m\\]:\\[$(tput sgr0)\\]\\[\\033[38;5;24m\\][\\w]:\\[$(tput sgr0)\\]\\[\\033[38;5;8m\\] \\[$(tput sgr0)\\]"
    ' >> /home/vagrant/.bashrc

    echo "####################################################################"
    echo "################### python-software-properties #####################"
    echo "####################################################################"
    apt-get install -y python-software-properties

    echo "####################################################################"
    echo "########################## UPDATING APT ############################"
    echo "####################################################################"
    add-apt-repository -y ppa:ondrej/mysql-5.7
    add-apt-repository -y ppa:ondrej/php
    apt-get update

    echo "####################################################################"
    echo "####################### INSTALLING NGINX ##########################"
    echo "####################################################################"
    apt-get -y install nginx
    openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -subj "/C=US/ST=Denial/L=Springfield/O=Dis/CN=*.dev" -keyout /etc/nginx/ssl.key -out /etc/nginx/ssl.crt

    echo "####################################################################"
    echo "######################## Setting Locales ###########################"
    echo "####################################################################"
    locale-gen en_US en_US.UTF-8 ru_RU ru_RU.UTF-8
    dpkg-reconfigure locales

    echo "####################################################################"
    echo "######################## INSTALLING MYSQL ##########################"
    echo "####################################################################"
    export DEBIAN_FRONTEND="noninteractive"
    debconf-set-selections <<< "mysql-server mysql-server/root_password password ${MYSQL_PASSWORD}"
    debconf-set-selections <<< "mysql-server mysql-server/root_password_again password ${MYSQL_PASSWORD}"
    apt-get -q -y install mysql-server-5.7 mysql-client-5.7

    echo "####################################################################"
    echo "######################### CONFIGURE MYSQL ##########################"
    echo "####################################################################"
    sed -i "s/.*bind-address.*/bind-address = 0.0.0.0/" ${MYSQL_CFG_PATH}
    echo "wait_timeout = 600" >> ${MYSQL_CFG_PATH}

    echo "####################################################################"
    echo "############## CREATING DATABASE & ADD % TO ROOT ###################"
    echo "####################################################################"
    mysql -u root -p${MYSQL_PASSWORD} -e "create database site;"
    mysql -u root -p${MYSQL_PASSWORD} -e "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '';"
    service mysql restart

    echo "####################################################################"
    echo "########################## INSTALLING PHP ##########################"
    echo "####################################################################"
    apt-get -y install php7.0-fpm

    echo "####################################################################"
    echo "###################### INSTALLING PHP EXTENSIONS ###################"
    echo "####################################################################"
    apt-get -y install php7.0-mcrypt php7.0-curl php7.0-cli php7.0-mysql php7.0-gd php7.0-intl php7.0-common php-pear php7.0-dev php7.0-xsl php-xdebug

    echo "####################################################################"
    echo "########################## CONFIGURE PHP ###########################"
    echo "####################################################################"
    echo "memory_limit = 1024M" >> ${PHP_INI_PATH}
    echo "max_execution_time = 600" >> ${PHP_INI_PATH}
    echo "always_populate_raw_post_data = -1" >> ${PHP_INI_PATH}
    echo "cgi.fix_pathinfo = 0" >> ${PHP_INI_PATH}
    echo "date.timezone = America/New_York" >> ${PHP_INI_PATH}
    echo "xdebug.remote_enable = 1" >> ${PHP_INI_PATH}
    echo "xdebug.remote_connect_back = 1" >> ${PHP_INI_PATH}

    service php7.0-fpm restart

    echo "####################################################################"
    echo "########################## INSTALLING GIT ##########################"
    echo "####################################################################"
    apt-get -y install git

    echo "####################################################################"
    echo "####################### INSTALLING COMPOSER ########################"
    echo "####################################################################"
    curl -sS https://getcomposer.org/installer | php
    mv composer.phar /usr/local/bin/composer

    echo "####################################################################"
    echo "######################## INSTALLING POSTFIX ########################"
    echo "####################################################################"
    debconf-set-selections <<< "postfix postfix/mailname string site.dev"
    debconf-set-selections <<< "postfix postfix/main_mailer_type string 'Internet Site'"
    apt-get -y install postfix

    echo "####################################################################"
    echo "########################### CONFIG NGINX ###########################"
    echo "####################################################################"
    echo "
    server {
        listen 80;
        listen 443 ssl;

        index index.php index.html index.htm;

        server_name site.dev;
        root /var/www/html/site;

        ssl_certificate     ssl.crt;
        ssl_certificate_key ssl.key;
        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers         HIGH:!aNULL:!MD5;

        location / {
            try_files \\$uri \\$uri/ =404;
        }

        error_page 404 /404.html;
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root /usr/share/nginx/html;
        }

        location ~ \\.php$ {
            try_files \\$uri =404;
            fastcgi_split_path_info ^(.+\\.php)(/.+)$;
            fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME \\$document_root\\$fastcgi_script_name;
            include fastcgi_params;
        }
    }
    " > /etc/nginx/sites-available/site.dev
    ln -s /etc/nginx/sites-available/site.dev /etc/nginx/sites-enabled/site.dev
    service nginx restart

    echo "####################################################################"
    echo "######################## INSTALL OTHER SOFT ########################"
    echo "####################################################################"
    apt-get -y install htop

    echo "####################################################################"
    echo "####################################################################"
    echo "################# PHP-Environment Vagrant Box ready! ###############"
    echo "################# Go to http://site.dev/             ###############"
    echo "####################################################################"
    echo "####################################################################"
  SHELL

  # If you need to forward ports, you can use this command:
  # config.vm.network "forwarded_port", guest: 80, host: 8080
end

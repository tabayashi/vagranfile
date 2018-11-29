# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = 'ubuntu/trusty64'
  config.vm.box_check_update = false
  config.vm.network :public_network, bridge: 'en1: Wi-Fi (AirPort)'
  config.vm.synced_folder '.', '/vagrant', disabled: true
  config.vm.synced_folder 'var/www/', '/var/www',
                          type: 'rsync',
                          rsync__exclude: [
                            '/var/www/default/node_modules',
                            '/var/www/default/public'
                          ],
                          rsync__args: ['--verbose', '--archive', '--compress'],
                          create: true

  config.vm.provider 'virtualbox' do |vb|
    vb.memory = 2048
  end

  config.vm.provision 'shell', inline: <<-UPDATE_APTGET
    apt-get update
  UPDATE_APTGET

  config.vm.provision 'shell', inline: <<-INSTALL_UTILS
    apt-get install -y git unzip
  INSTALL_UTILS

  config.vm.provision 'shell', inline: <<-INSTALL_APACHE2
    apt-get install -y apache2 apache2-dev
    echo 'ServerName 127.0.0.1' > /etc/apache2/conf-available/local-servername.conf
    a2enconf local-servername
  INSTALL_APACHE2

  config.vm.provision 'shell', inline: <<-INSTALL_MYSQL
    echo 'mysql-server mysql-server/root_password password vagrant' | debconf-set-selections
    echo 'mysql-server mysql-server/root_password_again password vagrant' | debconf-set-selections
    apt-get install -y mysql-server
    mkdir -p /var/log
    touch /var/log/slow.log
  INSTALL_MYSQL

  config.vm.provision 'shell', privileged: false, inline: <<-INITIALIZE_MYSQL
    echo "[mysqld]" >> "/home/vagrant/.my.cnf"
    echo "innodb_buffer_pool_size=512M" >> "/home/vagrant/.my.cnf"
    echo "innodb_additional_mem_pool_size=20M" >> "/home/vagrant/.my.cnf"
    echo "innodb_log_buffer_size=64M" >> "/home/vagrant/.my.cnf"
    echo "innodb_log_file_size=512M" >> "/home/vagrant/.my.cnf"
    echo "innodb_file_per_table=1" >> "/home/vagrant/.my.cnf"
    echo "query_cache_limit=16M" >> "/home/vagrant/.my.cnf"
    echo "query_cache_size=512M" >> "/home/vagrant/.my.cnf"
    echo "query_cache_type=1" >> "/home/vagrant/.my.cnf"
    echo "slow_query_log=ON" >> "/home/vagrant/.my.cnf"
    echo "long_query_time=3" >> "/home/vagrant/.my.cnf"
    echo "log-slow-queries=/var/log/slow.log" >> "/home/vagrant/.my.cnf"
    echo "join_buffer_size=256K" >> "/home/vagrant/.my.cnf"
    echo "max_allowed_packet=8M" >> "/home/vagrant/.my.cnf"
    echo "read_buffer_size=1M" >> "/home/vagrant/.my.cnf"
    echo "read_rnd_buffer_size=2M" >> "/home/vagrant/.my.cnf"
    echo "sort_buffer_size=4M" >> "/home/vagrant/.my.cnf"
    echo "max_heap_table_size=16M" >> "/home/vagrant/.my.cnf"
    echo "tmp_table_size=16M" >> "/home/vagrant/.my.cnf"
    echo "thread_cache_size=100" >> "/home/vagrant/.my.cnf"
    echo "wait_timeout=30" >> "/home/vagrant/.my.cnf"
  INITIALIZE_MYSQL

  config.vm.provision 'shell', privileged: false, inline: <<-INSTALL_ANYENV
    if [ ! -d /home/vagrant/.anyenv ]; then
      git clone https://github.com/riywo/anyenv /home/vagrant/.anyenv
      echo 'export PATH="$HOME/.anyenv/bin:$PATH"' >> /home/vagrant/.profile
      echo 'export ANYENV_PATH="$HOME/.anyenv"' >> /home/vagrant/.profile
      echo 'eval "$(anyenv init -)"' >> /home/vagrant/.profile
      git clone https://github.com/znz/anyenv-update.git /home/vagrant/.anyenv/plugins/anyenv-update
      git clone https://github.com/znz/anyenv-git.git /home/vagrant/.anyenv/plugins/anyenv-git
    fi
  INSTALL_ANYENV

  config.vm.provision 'shell', privileged: false, inline: <<-INSTALL_NODENV
    if [ ! -d /home/vagrant/.anyenv/envs/nodenv ]; then
      anyenv install nodenv
    fi
  INSTALL_NODENV

  config.vm.provision 'shell', privileged: false, inline: <<-INSTALL_NODEJS
    if [ ! -d /home/vagrant/.anyenv/envs/nodenv/versions/10.13.0 ]; then
      nodenv install 10.13.0
      nodenv global 10.13.0
    fi
  INSTALL_NODEJS

  config.vm.provision 'shell', privileged: false, inline: <<-INSTALL_PHPENV
    if [ ! -d /home/vagrant/.anyenv/envs/phpenv ]; then
      anyenv install phpenv
    fi
  INSTALL_PHPENV

  config.vm.provision 'shell', inline: <<-INSTALL_PHP7_0_10LIB
    apt-get install -y sqlite libxml2-dev libcurl4-openssl-dev libbz2-dev re2c libgd-dev libicu-dev libtidy-dev libreadline-dev pkg-config make libtool autoconf automake bison nasm
  INSTALL_PHP7_0_10LIB

  config.vm.provision 'shell', inline: <<-PRE_INSTALL_PHP
    if [ -d /etc/apache2 ]; then
      chmod oga+rw /usr/lib/apache2/modules
      chmod oga+rw /var/lib/apache2/module
      chmod -R oga+rw /etc/apache2
    fi
  PRE_INSTALL_PHP

  config.vm.provision 'shell', privileged: false, inline: <<-INSTALL_PHP7_0_10
    if [ ! -d /home/vagrant/.anyenv/envs/phpenv/versions/7.0.10  ]; then
      source /etc/apache2/envvars
      PHP_BUILD_CONFIGURE_OPTS="--with-pear --without-mcrypt --without-xsl --with-readline --enable-mbstring --enable-pcntl --with-gettext=/usr --with-gd --enable-sockets --with-jpeg-dir=/usr --with-png-dir=/usr --enable-exif --enable-zip --with-zlib --with-zlib-dir=/usr --with-bz2 --enable-intl --with-kerberos --with-openssl --enable-ftp --enable-cgi --with-curl=/usr --with-tidy --with-xmlrpc --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-apxs2=/usr/bin/apxs" PHP_BUILD_EXTRA_MAKE_ARGUMENTS=-j4 phpenv install --verbose -f 7.0.10
      phpenv global 7.0.10
      echo no | pecl install apcu
    fi
  INSTALL_PHP7_0_10

  config.vm.provision 'shell', inline: <<-POST_INSTALL_PHP
    if [ -d /etc/apache2 ]; then
      chmod og-w /usr/lib/apache2/modules
      chmod og-w /var/lib/apache2/module
      chmod -R og-w /etc/apache2
    fi
  POST_INSTALL_PHP

  config.vm.provision 'shell', privileged: false, inline: <<-INITIALIZE_PHP
    if ! grep -q extension=apcu.so /home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini; then
      perl -pe 's%^;?(default_charset)\s?=\s?.?$%$1 = "UTF-8"%' -i.bak "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      perl -pe 's%^;?(mbstring\.language)\s?=\s?.?$%$1 = Japanese%' -i.bak "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      perl -pe 's%^;?(mbstring\.encoding_translation)\s?=\s?.?$%$1 = Off%' -i.bak "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      perl -pe 's%^;?(mbstring\.detect_order)\s?=\s?.?$%$1 = UTF-8,SJIS,EUC-JP,JIS,ASCII%' -i.bak "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      perl -pe 's%^;?(date\.timezone)\s?=\s?.?$%$1 = Asia/Tokyo%' -i.bak "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      perl -pe 's%^;?(expose_php)\s?=\s?.?$%$1 = Off%' -i.bak "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      perl -pe 's%^;?(memory_limit)\s?=\s?.?$%$1 = 512M%' -i.bak "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      perl -pe 's%^;?(post_max_size)\s?=\s?.?$%$1 = 128M%' -i.bak "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      perl -pe 's%^;?(upload_max_filesize)\s?=\s?.?$%$1 = 128M%' -i.bak "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      echo "opcache.enable=1" >> "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      echo "opcache.memory_consumption=128" >> "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      echo "opcache.interned_strings_buffer=8" >> "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      echo "opcache.max_accelerated_files=4000" >> "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      echo "opcache.revalidate_freq=60" >> "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      echo "opcache.fast_shutdown=1" >> "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      echo "opcache.enable_cli=1" >> "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      echo "extension=apcu.so" >> "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      echo "apc.shm_size=128M" >> "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      echo "apc.ttl=86400" >> "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
      echo "apc.gc_ttl=86400" >> "/home/vagrant/.anyenv/envs/phpenv/versions/7.0.10/etc/php.ini"
    fi
  INITIALIZE_PHP

  config.vm.provision 'shell', privileged: false, inline: <<-INITIALIZE_COMPOSER
    composer config -g repos.packagist composer https://packagist.jp
    composer global require hirak/prestissimo
    composer global require psy/psysh:dev-master
  INITIALIZE_COMPOSER

  config.vm.provision 'shell', inline: <<-INSTALL_SYMFONY
    if [ ! -e /usr/local/bin/symfony ]; then
      mkdir -p /usr/local/bin
      curl -LsS https://symfony.com/installer -o /usr/local/bin/symfony
      chmod a+x /usr/local/bin/symfony
    fi
  INSTALL_SYMFONY

  config.vm.provision 'shell', inline: <<-INITIALIZE_APACHE2
    a2enmod expires
    a2enmod headers
    a2enmod session_cookie
    a2enmod rewrite
    a2enmod vhost_alias
    if [ ! -f /etc/apache2/mods-available/php7.conf ]; then
      echo 'AddType application/x-httpd-php .php' > /etc/apache2/mods-available/php7.conf
      a2enmod php7
    fi
    apachectl stop
    perl -pe 's%^(\t*)DocumentRoot /var/www/html$%$1DocumentRoot /var/www/default/public\n$1<Directory "/var/www/default/public">\n$1\tOptions All\n$1\tAllowOverride All\n$1\tRequire all granted\n$1</Directory>%' -i.bak /etc/apache2/sites-available/000-default.conf
    if [ ! -d /var/www/default ]; then
      mkdir -p /var/www/default
      mv /var/www/html /var/www/default/public
      echo '<?php phpinfo(); ?>' > /var/www/default/public/info.php
      chown -Rh vagrant:vagrant /var/www
    fi
  INITIALIZE_APACHE2

  config.vm.provision 'shell', inline: <<-INSTALL_NPMLIB
    apt-get install -y graphicsmagick imagemagick libjpeg-progs gifsicle optipng
  INSTALL_NPMLIB

  config.vm.provision 'shell', inline: <<-INSTALL_ZSH
    apt-get install -y zsh
    if [ ! -d /usr/local/share/zsh-completions ]; then
      git clone git://github.com/zsh-users/zsh-completions.git /usr/local/share/zsh-completions
    fi
    if [ ! -f /home/vagrant/.zshenv ]; then
      echo 'export FPATH="/usr/local/share/zsh-completions/src:$FPATH"' >> /home/vagrant/.zshenv
      echo 'export PATH="$HOME/.anyenv/bin:$PATH"' >> /home/vagrant/.zshenv
      echo 'export ANYENV_PATH="$HOME/.anyenv"' >> /home/vagrant/.zshenv
      chown -Rh vagrant:vagrant /home/vagrant/.zshenv
    fi
    if [ ! -f /home/vagrant/.zshrc ]; then
      echo 'PROMPT="%F{green}[%d]%f %# "' >> /home/vagrant/.zshrc
      echo 'alias ll="ls -alvF"' >> /home/vagrant/.zshrc
      echo 'alias rm="rm -i"' >> /home/vagrant/.zshrc
      echo 'alias cp="cp -i"' >> /home/vagrant/.zshrc
      echo 'alias mv="mv -i"' >> /home/vagrant/.zshrc
      echo 'alias mkdir="mkdir -p"' >> /home/vagrant/.zshrc
      echo 'alias reload="exec -l $SHELL"' >> /home/vagrant/.zshrc
      echo 'alias addr="ifconfig | grep inet"' >> /home/vagrant/.zshrc
      echo 'autoload -Uz compinit' >> /home/vagrant/.zshrc
      echo 'compinit -u' >> /home/vagrant/.zshrc
      echo 'eval "$(anyenv init -)"' >> /home/vagrant/.zshrc
      chown -Rh vagrant:vagrant /home/vagrant/.zshrc
    fi
  INSTALL_ZSH

  config.vm.provision 'shell', privileged: false, inline: <<-CHSH_ZSH
    echo vagrant | chsh -s /usr/bin/zsh vagrant
  CHSH_ZSH

  config.vm.provision 'shell', run: 'always', inline: <<-TEARDOWN
    apt-get autoremove -y
    if [ -x "$(command -v mysql)" ]; then
      service mysql restart
    fi
    if [ -x "$(command -v apache2)" ]; then
      service apache2 restart
    fi
  TEARDOWN
end

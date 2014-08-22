# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version.
VAGRANTFILE_API_VERSION = "2"

$mysql_root_password = "sadvWQEFasdv3!"
$mysql_user_password = "156fsdgEWER%!"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  
  config.vm.box = "chef/centos-6.5"

  config.vm.provider "virtualbox" do |v|
    v.customize ["modifyvm", :id, "--cpuexecutioncap", "70"]
    v.memory = 512
    v.cpus = 2
  end

  # Load latest chef
  config.omnibus.chef_version = :latest
  
  # Bershelf plugin
  config.berkshelf.enabled = true

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "forwarded_port", guest: 443, host: 4043

  # Disable default synced folder
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # Sync src
  config.vm.synced_folder "src", "/vagrant/src", create: true, :mount_options => ['dmode=777', 'fmode=777']

  # Sync vhosts
  config.vm.synced_folder "vhosts", "/etc/httpd/sites-enabled"

  # Chef provision
  config.vm.provision "chef_solo" do |chef|
    chef.add_recipe "yum"
    chef.add_recipe "yum-epel"
    chef.add_recipe "zip"
    chef.add_recipe "git"
    chef.add_recipe "python"
    chef.add_recipe "build-essential"

    chef.add_recipe "mysql::client"
    chef.add_recipe "mysql::server"

    chef.add_recipe "openssl"

    chef.add_recipe "apache2"
    chef.add_recipe "apache2::mod_rewrite"
    chef.add_recipe "apache2::mod_php5"
    chef.add_recipe "apache2::mod_wsgi"
    chef.add_recipe "apache2::mod_ssl"
    chef.add_recipe "apache2::mod_fastcgi"

    chef.add_recipe "redisio::install"
    chef.add_recipe "redisio::enable"

    chef.add_recipe "memcached"

    chef.add_recipe "hhvm"
    # chef.add_recipe "varnish"

    chef.json = {
      :apache => { :default_site_enabled => false },
      :mysql => {
        :server_root_password => $mysql_root_password,
        :server_repl_password => $mysql_root_password,
        :server_debian_password => $mysql_root_password
      }
    }
  end
  
  # Default DB setup
  config.vm.provision :shell, :inline => "mysql -u root -p#{$mysql_root_password} -e \"create database if not exists dev_db\""
  config.vm.provision :shell, :inline => "mysql -u root -p#{$mysql_root_password} -e \"GRANT ALL ON dev_db.* TO 'dev_user'@'localhost' IDENTIFIED BY '#{$mysql_user_password}'; FLUSH PRIVILEGES;\""
  # Optional import DB
  # config.vm.provision :shell, :inline => "cat /vagrant/src/dump.sql.z* > /vagrant/src/dump.complete.zip; unzip -p /vagrant/src/dump.complete.zip | mysql -h localhost -u dev_user -p#{$mysql_user_password} dev_db; rm -f /vagrant/src/dump.complete.zip"

  # PHPMyAdmin
  config.vm.provision :shell, :inline => "yum install -y phpmyadmin; echo 'Alias /phpmyadmin /usr/share/phpMyAdmin\nAlias /phpMyAdmin /usr/share/phpMyAdmin\n\n<Directory /usr/share/phpMyAdmin/>\nOrder allow,deny\nAllow from all\n</Directory>' > /etc/httpd/sites-enabled/phpmyadmin.conf"
  # libjpeg
  config.vm.provision :shell, :inline => "yum install -y libjpeg-devel"

  # PHP 5.4
  config.vm.provision :shell, :inline => <<-SH
    rpm -Uvh http://mirror.webtatic.com/yum/el6/latest.rpm

	yum install -y yum-plugin-replace

    yum replace -y php-common --replace-with=php54w-common
    yum install -y php54w-pecl-memcache
    yum install -y php54w-pecl-zendopcache
    yum install -y php54w-devel

    pecl install redis
    echo extension = redis.so>>/etc/php.d/redis.ini

    a2enmod actions alias
  SH

  # Start HHVM and restart Apache
  config.vm.provision :shell, :inline => "hhvm --mode daemon -vServer.Type=fastcgi -vServer.Port=9000", run: "always"
  config.vm.provision :shell, :inline => "service httpd restart", run: "always"

end
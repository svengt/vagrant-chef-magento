# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version.
VAGRANTFILE_API_VERSION = "2"

$mysql_root_password = "sadvWQEFasdv3!"
$mysql_user_password = "156fsdgEWER%!"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  
  config.vm.box = "chef/centos-6.5"
  
  # Load latest chef
  config.omnibus.chef_version = :latest
  
  # Bershelf plugin
  config.berkshelf.enabled = true

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "forwarded_port", guest: 8000, host: 8001
  
  # Sync src
  config.vm.synced_folder "src", "/home/vagrant/src", create: true
  # Sync vhosts
  config.vm.synced_folder "vhosts", "/etc/httpd/sites-enabled"
  
  # Disable default synced folder
  config.vm.synced_folder ".", "/vagrant", disabled: true
  
  # Chef provision
  config.vm.provision "chef_solo" do |chef|
    chef.add_recipe "yum"
    chef.add_recipe "yum-epel"
    chef.add_recipe "build-essential"
    chef.add_recipe "python"
    chef.add_recipe "mysql::client"
    chef.add_recipe "mysql::server"
    # chef.add_recipe "apache2::mod_rewrite"
    chef.add_recipe "openssl"
    chef.add_recipe "php"
    # chef.add_recipe "php::module_apc"
    chef.add_recipe "php::module_curl"
    chef.add_recipe "php::module_mysql"
    
    chef.add_recipe "apache2"
    chef.add_recipe "apache2::mod_wsgi"
    chef.add_recipe "apache2::mod_php5"
    
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
  
  # PHPMyAdmin
  config.vm.provision :shell, :inline => "sudo yum install -y phpmyadmin; echo 'Alias /phpmyadmin /usr/share/phpMyAdmin\nAlias /phpMyAdmin /usr/share/phpMyAdmin\n\n<Directory /usr/share/phpMyAdmin/>\nOrder allow,deny\nAllow from all\n</Directory>' > /etc/httpd/sites-enabled/phpmyadmin; sudo service httpd restart"
  
  # Application provision
  config.vm.provision :shell, :inline => "pip install -r ./src/requirements.txt", run: "always"
  
end

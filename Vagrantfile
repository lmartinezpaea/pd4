# Installation script for Apache2, MySQL and Ruby (rbenv)
# Date: Aug 30, 2016 
# Description: This script creates a Vagrant box preloaded with Linux Ubuntu 14.04 LTS, Apache 2.2.22, MySQL 5.5, rbenv and Ruby 2.3.1. For local development convenience on the host machine, the www root folder in Apache has been changed to /vagrant/www which mirrors the local folder on the host machine where the Vagrantfile resides (asuming that there is a www folder inside the same location of the Vagrantfile on the host).  
#
# Vagrant Box IP address: 172.28.128.128
#
# Virtualbox RAM memory: 2048Mb
#
# ************************************
# * Vagrant user password: vagrant   *
# *                                  *
# * MySQL:                           *
# *  root password: rootroot         *
# *                                  *
# * Hostname: 172.28.128.120         *
# * user: vagrant                    *
# * passwd: vagrant                  *
# ************************************
#  
# # Declare a "script" variable with all the necessary packages and configuration parameters
$script = <<SCRIPT
sudo usermod -aG sudo vagrant
sudo apt-get -qq -y update
sudo apt-get -y --quiet install build-essential

# Install GIT
sudo apt-get -y --quiet install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties libffi-dev

git clone git://github.com/sstephenson/rbenv.git .rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc

# Adds "install" plug-in to rbenv
sudo git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Installs Ruby as vagrant user 
sudo -H --user=vagrant bash -i -c 'rbenv install 2.3.1' 
sudo -H --user=vagrant bash -i -c 'rbenv global 2.3.1' 

sudo apt-get -y --quiet install unzip
sudo apt-get -y --quiet install apache2
sudo apt-get -y --quiet install debconf-utils
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password rootroot'
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password rootroot'
sudo apt-get -y --quiet install mysql-server libapache2-mod-auth-mysql 
sudo mysql_install_db

# Secure root access
echo "update mysql.user set password = password('rootroot') where user = 'root';" > mysql_script

# Secure anonymous users
echo "update mysql.user set password = password('rootroot') where user = '';" >> mysql_script

# Delete test databases 
echo "delete from mysql.db where Db like 'test%';" >> mysql_script

echo "flush privileges;" >> mysql_script

# Install MySQL development headers (Ruby Gem MySQL2 requires it)
sudo apt-get install libmysqlclient-dev

# Change www root folder to /vagrant/www
cd /etc/apache2/sites-available
sudo sed -i "s#/var/www/html#/vagrant/www#g" 000-default.conf
cd ..
sudo sed -i "s#/var/www/#/vagrant/www/#g" apache2.conf

# Add permit to upload CSV to MySQL databases
cd /etc/mysql
sudo sed -i "s#mysql]#mysql]\nlocal-infile=1\n#g" my.cnf
sudo service mysql restart

# Change ownership and permissions on /vagrant/www/
mkdir /vagrant/www
cd /vagrant/www
sudo chmod g+w /vagrant/www -R

# Create a basic HTML page
sudo echo "<h1>Welcome to /vagrant/www/index.html!</h1>" > /vagrant/www/index.html

# Install Rails
cd ~
sudo apt-get -y install nodejs
gem install rails

# Change Apache user/group to vagrant
cd /etc/apache2
sudo sed -i "s/www-data/vagrant/g" envvars 
cd /var/lock
sudo chown vagrant apache2

# Restart Apache
sudo service apache2 restart

SCRIPT

Vagrant.configure(2) do |config|
  
  # Configure RAM memory and CPU amount for the provider Virtualbox
  config.vm.provider "virtualbox" do |v|
    v.memory = 2048 
    v.cpus = 2
  end

  config.vm.define :vagrant_box do |vagrant_box|
    # Use Ubuntu 14.04 LTS 64-bit image 
    vagrant_box.vm.box = "ubuntu/trusty64"
   
    # Define an "private" IP address only reachable from the host machine and among other VMs within the same subnet and group "private_network" 
    vagrant_box.vm.network "private_network", ip: "172.28.128.128"
    config.vm.network :forwarded_port, guest: 3000, host: 3000 
    vagrant_box.vm.provision "shell", privileged: false, inline: $script
  end

end

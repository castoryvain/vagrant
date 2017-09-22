# Vagrant File for headless Debian + docker + docker compose
# encoding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :
# Box / OS
VAGRANT_BOX = 'debian/jessie64'
# Memorable name for your
VM_NAME = 'SoBee-VM'
# VM User — 'vagrant' by default
VM_USER = 'vagrant'
# Host folder to sync
HOST_PATH = '.'
# Where to sync to on Guest — 'vagrant' is the default user name
GUEST_PATH = '/vagrant'
# # VM Port — uncomment this to use NAT instead of DHCP
VM_PORT = 80
Vagrant.configure(2) do |config|
  # Vagrant box from Hashicorp
  config.vm.box = VAGRANT_BOX
  # Actual machine name
  config.vm.hostname = VM_NAME
  # Set VM name in Virtualbox
  config.vm.provider "virtualbox" do |v|
    v.name = VM_NAME
    v.memory = 2048
  end
  #DHCP — comment this out if planning on using NAT instead
  # config.vm.network "private_network", type: "dhcp"
  # Port forwarding — uncomment this to use NAT instead of DHCP
  config.vm.network "forwarded_port", guest: 80, host: VM_PORT
  config.vm.network "forwarded_port", guest: 3306, host: 3306
  config.vm.network "forwarded_port", guest: 8082, host: 8082
  config.vm.network "forwarded_port", guest: 9200, host: 9200
  # Sync folder
  config.vm.synced_folder HOST_PATH, GUEST_PATH, type: "virtualbox"
  # Install Git, etc ....
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y git
    apt-get install -y  apt-transport-https ca-certificates curl gnupg2  software-properties-common
    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
    apt-get update
    apt-get install -y docker-ce
    curl -L --fail https://github.com/docker/compose/releases/download/1.14.0/run.sh > /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose

    # Install SoBee Build Tools ....
    add-apt-repository -y "deb http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main"
    apt-get update -y
    echo debconf shared/accepted-oracle-license-v1-1 select true | sudo debconf-set-selections
    echo debconf shared/accepted-oracle-license-v1-1 seen true | sudo debconf-set-selections
    apt-get -y install oracle-java8-installer
    wget http://mirrors.ircam.fr/pub/apache/maven/maven-3/3.5.0/binaries/apache-maven-3.5.0-bin.tar.gz
    tar -xvf apache-maven-3.5.0-bin.tar.gz
    mv apache-maven-3.5.0 /opt/.
    ln -s /opt/apache-maven-3.5.0/bin/mvn /usr/bin/mvn
    rm apache-maven-3.5.0-bin.tar.gz
    apt-get upgrade -y
    apt-get autoremove -y
    cd /vagrant
    mv overriding.properties overriding.properties.old
    cp overriding.properties.vagrant overriding.properties
    # Build SoBee
    mvn clean install docker:build -DskipTests=true
    # Run SoBee
    docker-compose up -d db
    mvn flyway:clean && mvn flyway:migrate
    docker-compose up -d sobee nginx
  SHELL
end

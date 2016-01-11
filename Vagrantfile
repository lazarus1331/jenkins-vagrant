# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.require_version ">= 1.6.0"
require 'fileutils'
CONFIG = File.join(File.dirname(__FILE__), ".config.rb")
user=''
oauth=''
if File.exist?(CONFIG)
  require CONFIG
end

Vagrant.configure(2) do |config|
  # https://docs.vagrantup.com.

  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "vubuntu-15.04"
  config.vm.hostname = "jenkins.threatx.local"

  # Create a forwarded port mapping which allows access to a specific port
  config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "172.16.13.7", virtualbox__intnet: "threatxnet"

  # Create a public network, which generally matched to bridged network.
  # config.vm.network "public_network", bridge: "eth0"

  config.vm.synced_folder "./data", "/vagrant_data"

  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
 
    # Customize the amount of memory on the VM:
    vb.memory = "1024"
  end
  
  # Install pre-reqs
  config.vm.provision "shell", :privileged => true, inline: <<-SHELL
    apt-get update
	apt-get install -y openjdk-7-jre git python python-dev python-pip
  SHELL

  # Install jenkins
  config.vm.provision "shell", :privileged => true, inline: <<-SHELL
	wget -q -O - https://jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
	sh -c 'echo deb http://pkg.jenkins-ci.org/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
    apt-get update
	apt-get install -y jenkins
  SHELL

  # Install nginx 
  config.vm.provision "shell", :privileged => true, inline: <<-SHELL
	apt-get install -y nginx
	cd /etc/nginx/sites-available
	if [[ -e default ]];then rm default ../sites-enabled/default;fi
  SHELL

  # add nginx-jenkins config file
  config.vm.provision :file, :source => "./nginx-jenkins.conf", :destination => "/tmp/nginx-jenkins.conf"

  # config nginx and restart 
  config.vm.provision "shell", :privileged => true, inline: <<-SHELL
	cp /tmp/nginx-jenkins.conf /etc/nginx/sites-available/jenkins
	ln -s /etc/nginx/sites-available/jenkins /etc/nginx/sites-enabled/ || true
	service nginx restart
	systemctl status nginx 
  SHELL

  # copy jenkins plugins from /vagrant_data/ 
  config.vm.provision "shell", :privileged => true, inline: <<-SHELL
	cp -R /vagrant_data/plugins/* /var/lib/jenkins/plugins/
  SHELL
  
  # restart jenkins to install plugins
  config.vm.provision "shell", :privileged => true, inline: <<-SHELL
	service jenkins restart
	systemctl status jenkins
  SHELL

  # install latest ansible via pip
  config.vm.provision "shell", :privileged => true, inline: <<-SHELL
	pip install ansible
  SHELL

end

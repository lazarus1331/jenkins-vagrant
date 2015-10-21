# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'yaml'

current_dir    = File.dirname(File.expand_path(__FILE__))
configs        = YAML.load_file("#{current_dir}/.config.yaml")
$user_tx_platform = configs['configs']['user']
$oauth_tx_platform = configs['configs']['oauth']

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
  config.vm.provision "shell", :privileged => true, inline: <<-SHELL1
    apt-get update
	apt-get install -y openjdk-7-jre git python ansible
  SHELL1

  # Install jenkins
  config.vm.provision "shell", :privileged => true, inline: <<-SHELL2
	wget -q -O - https://jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
	sh -c 'echo deb http://pkg.jenkins-ci.org/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
    apt-get update
	apt-get install -y jenkins
  SHELL2

  # Install nginx 
  config.vm.provision "shell", :privileged => true, inline: <<-SHELL3
	apt-get install -y nginx
	cd /etc/nginx/sites-available
	if [[ -f default ]];then rm default ../sites-enabled/default;fi
  SHELL3

  # copy jenkins plugins from /vagrant_data/ 
  config.vm.provision "shell", :privileged => true, inline: <<-SHELL4
	cp -R /vagrant_data/* /var/lib/jenkins/plugins/
  SHELL4
  
  # add config file
  config.vm.provision :file, :source => "./nginx-jenkins.conf", :destination => "/tmp/nginx-jenkins.conf"

  # config nginx and restart 
  config.vm.provision "shell", :privileged => true, inline: <<-SHELL5
	cp /tmp/nginx-jenkins.conf /etc/nginx/sites-available/jenkins
	ln -s /etc/nginx/sites-available/jenkins /etc/nginx/sites-enabled/ || true
	service nginx restart
	systemctl status nginx 
  SHELL5

  # restart jenkins to install plugins
  config.vm.provision "shell", :privileged => true, inline: <<-SHELL6
	service jenkins restart
	systemctl status jenkins
  SHELL6

  # setup ansible playbooks
  config.vm.provision "shell", :privileged => true, inline: <<-SHELL7
	mkdir -p /opt/jenkins-git/platform-deploy
	echo $oauth_tx_platform | git clone https://$user_tx_platform@github.com/ThreatX/platform-deploy.git /opt/jenkins-git/platform-deploy
	cd /opt/jenkins-git/platform-deploy
	git checkout mrollins-vagrant
	chown -R jenkins.root /opt/jenkins-git
  SHELL7

end

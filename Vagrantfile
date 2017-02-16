# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "centos/7"

  config.vm.network "forwarded_port", guest: 80, host: 8080

  config.vm.provision "shell", inline: <<-SHELL
    yum install -y git epel-release
    yum install -y nginx
    systemctl start nginx
    systemctl enable nginx
  SHELL

  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    git init --bare ci-fun.git
  SHELL

  config.vm.provision "file", source: "post-receive",
                              destination: "ci-fun.git/hooks/post-receive"
end

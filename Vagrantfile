# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "centos/7"

  config.vm.network "forwarded_port", guest: 80, host: 8080

  config.vm.provision "shell", inline: <<-SHELL
    setenforce 0
    # setsebool -P httpd_read_user_content 1

    yum install -y epel-release
    yum install -y git nginx php-cli php-pdo php-xml php-fpm

    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    php -r "if (hash_file('SHA384', 'composer-setup.php') === '55d6ead61b29c7bdee5cccfb50076874187bd9f21f65d8991d46ec5cc90518f447387fb9f76ebae1fbbacf329e583e30') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
    php composer-setup.php --install-dir=/usr/local/bin --filename=composer
    php -r "unlink('composer-setup.php');"

    sed -e "s/;date\\.timezone =/date\\.timezone = Europe\\/Paris/" \
        -e "s/;cgi\\.fix_pathinfo=1/cgi\\.fix_pathinfo=0/" \
        -i /etc/php.ini

    sed -e "s/listen = \\(.*\\)/listen = \\/var\\/run\\/php5-fpm\\.sock/" \
        -e "s/;\\(listen\\.owner = nobody\\)/\\1/" \
        -e "s/;\\(listen\\.group = nobody\\)/\\1/" \
        -e "s/user = apache/user = nginx/" \
        -e "s/group = apache/group = nginx/" \
        -i /etc/php-fpm.d/www.conf

    mkdir -p /var/www/project/app
    cd /var/www/project

    mkdir app/config app/cache app/logs

    setfacl -R -m u:nginx:rwX -m u:`whoami`:rwX app/cache app/logs
    setfacl -dR -m u:nginx:rwX -m u:`whoami`:rwX app/cache app/logs
  SHELL

  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    git init --bare ci-fun.git
  SHELL

  config.vm.provision "file", source: "post-receive",
                              destination: "ci-fun.git/hooks/post-receive"

  config.vm.provision "file", source: "symfony.conf",
                              destination: "symfony.conf"

  config.vm.provision "file", source: "tiny-repo/app/config/parameters.yml",
                              destination: "parameters.yml"

  config.vm.provision "shell", inline: <<-SHELL
    mv parameters.yml /var/www/project/app/config
    mv symfony.conf /etc/nginx/conf.d/symfony.conf

    systemctl start php-fpm
    systemctl enable php-fpm

    systemctl start nginx
    systemctl enable nginx
  SHELL
end

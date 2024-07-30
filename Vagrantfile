# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.
  config.vm.define "Debian12" do |srv|

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
    srv.vm.box = "/home/max/vagrant/images/debian12"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  #  config.vm.network "private_network", ip: "192.168.33.10"
  # config.vm.network "private_network", ip: "192.168.33.10", adapter: "5", type: "ip"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.
  # system("virsh net-create /home/max/vagrant/vagrant-libvirt-mgmt.xml")
    srv.vm.provider "libvirt" do |lv|
      lv.memory = "2048"
      lv.cpus = "2"
      lv.title = "Debian12"
      lv.description = "Виртуальная машина на базе дистрибутива Debian Linux"
      lv.management_network_name = "vagrant-libvirt-mgmt"
      lv.management_network_address = "192.168.121.0/24"
      lv.management_network_keep = "true"
      lv.management_network_mac = "52:54:00:27:28:83"
      lv.storage :file, :size => '1G', :device => 'vdb', :allow_existing => false
    end
    srv.vm.provision "shell", inline: <<-SHELL
      brd='*************************************************************'
      scriptdir="/opt"
      spawnconfdir="/etc/spawn-fcgi"
      echo "$brd"
      echo 'Изменим ttl для работы через раздающий телефон'
      echo "$brd"
      sysctl -w net.ipv4.ip_default_ttl=66
     # echo "$brd"
     # echo 'Если ранее не были установлены, то установим необходимые  пакеты'
     # echo "$brd"
   # apt install -y apache2 dpkg-dev build-essential zlib1g-dev libpcre3 libpcre3-dev unzip git cmake reprepro
    echo "$brd"
    echo "$brd"
    echo "$brd"
    echo 'Создаём сервис watchlog, который будет искать запись "ALERT" в журнале /var/log/watchlog и записывать событие о найденой записи в системный журнал'
    echo "$brd"
    echo "$brd"
    
    echo "$brd"
    echo 'Создаём файл с конфигурацией для сервиса в директории /etc/default'
    echo "$brd"

    echo '# Configuration file for my watchlog service' >> /etc/default/watchlog
    echo '# Place it to /etc/default' >> /etc/default/watchlog
    echo '# File and word in that file that we will be monit' >> /etc/default/watchlog
    echo 'WORD="ALERT"' >> /etc/default/watchlog
    echo 'LOG=/var/log/watchlog.log' >> /etc/default/watchlog
    echo 'cat /etc/default/watchlog'
    
    cat /etc/default/watchlog

    echo "$brd"
    echo 'Создаём файл "журнала".'
    echo "$brd"
    touch /var/log/watchlog.log
    echo 'This is ALERT!!!' >> /var/log/watchlog.log
    echo 'cat /var/log/watchlog.log'
    
    cat /var/log/watchlog.log

    echo "$brd"
    echo 'Создадим скрипт'
    echo "$brd"

    if  [ ! -d $scriptdir ] 
        then mkdir $scriptdir
        fi
    echo '#!/bin/bash' >> /opt/watchlog.sh
    echo 'WORD=$1' >> /opt/watchlog.sh
    echo 'LOG=$2' >> /opt/watchlog.sh
    echo 'DATE=`date`' >> /opt/watchlog.sh
    echo 'if grep $WORD $LOG &> /dev/null; then echo "$DATE: I found word, Jedi!" | logger; else exit 0; fi' >> /opt/watchlog.sh
    echo 'cat /opt/watchlog.sh'
    
    cat /opt/watchlog.sh

    echo "$brd"
    echo 'Добавим права на запуск файла'
    echo "$brd"

    chmod u+x /opt/watchlog.sh
    echo 'ls -l /opt/watchlog.sh'
    
    ls -l /opt/watchlog.sh

    echo "$brd"
    echo 'Создадим юнит для сервиса'
    echo "$brd"

    echo '[Unit]' >> /etc/systemd/system/watchlog.service
    echo 'Description=My watchlog service' >> /etc/systemd/system/watchlog.service
    echo '' >> /etc/systemd/system/watchlog.service
    echo '[Service]' >> /etc/systemd/system/watchlog.service
    echo 'Type=oneshot' >> /etc/systemd/system/watchlog.service
    echo 'EnvironmentFile=/etc/default/watchlog' >> /etc/systemd/system/watchlog.service
    echo 'ExecStart=/opt/watchlog.sh $WORD $LOG' >> /etc/systemd/system/watchlog.service
    echo 'cat /etc/systemd/system/watchlog.service'
    
    cat /etc/systemd/system/watchlog.service

    echo "$brd"
    echo 'Создадим юнит для таймера'
    echo "$brd"

    echo '[Unit]' >> /etc/systemd/system/watchlog.timer
    echo 'Description=Run watchlog script every 30 second' >> /etc/systemd/system/watchlog.timer
    echo '' >> /etc/systemd/system/watchlog.timer
    echo '[Timer]' >> /etc/systemd/system/watchlog.timer
    echo '# Run every 30 second' >> /etc/systemd/system/watchlog.timer
    echo 'OnUnitActiveSec=30' >> /etc/systemd/system/watchlog.timer
    echo 'Unit=watchlog.service' >> /etc/systemd/system/watchlog.timer
    echo '' >> /etc/systemd/system/watchlog.timer
    echo '[Install]' >> /etc/systemd/system/watchlog.timer
    echo 'WantedBy=multi-user.target' >> /etc/systemd/system/watchlog.timer
    echo 'cat /etc/systemd/system/watchlog.timer'
    
    cat /etc/systemd/system/watchlog.timer

    echo "$brd"
    echo 'Запустим timer и service'
    echo "$brd"

    systemctl daemon-reload
    systemctl enable watchlog.timer
    systemctl start watchlog.service
    systemctl start watchlog.timer
    systemctl status watchlog.service
    systemctl status watchlog.timer

    echo "$brd"
    echo "$brd"
    echo "$brd"
    echo 'Создаём spawn-fcgi.sevice'
    echo "$brd"
    echo "$brd"
    echo "$brd"

    echo "$brd"
    echo 'Установим пакеты spawn-fcgi php php-cgi php-cli apache2 libapache2-mod-fcgid.'
    echo "$brd"
    
    echo 'apt install -y spawn-fcgi php php-cgi php-cli apache2 libapache2-mod-fcgid'
    
    apt install -y spawn-fcgi php php-cgi php-cli apache2 libapache2-mod-fcgid

    echo "$brd"
    echo 'Создадим файл конфигурации для spawn-fcgi'
    echo "$brd"
    if  [ ! -d $spawnconfdir ] 
        then mkdir $spawnconfdir
        fi
    echo '# You must set some working options before the "spawn-fcgi" service will work.' > /etc/spawn-fcgi/fcgi.conf
    echo '# If SOCKET points to a file, then this file is cleaned up by the init script.' >> /etc/spawn-fcgi/fcgi.conf
    echo '#' >> /etc/spawn-fcgi/fcgi.conf
    echo '# See spawn-fcgi(1) for all possible options.' >> /etc/spawn-fcgi/fcgi.conf
    echo '#' >> /etc/spawn-fcgi/fcgi.conf
    echo '# Example :' >> /etc/spawn-fcgi/fcgi.conf
    echo 'SOCKET=/var/run/php-fcgi.sock' >> /etc/spawn-fcgi/fcgi.conf
    echo 'OPTIONS="-u www-data -g www-data -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"' >> /etc/spawn-fcgi/fcgi.conf
    echo 'cat /etc/spawn-fcgi/fcgi.conf'
    
    cat /etc/spawn-fcgi/fcgi.conf

    echo "$brd"
    echo 'Создадим unit для spawn-fcgi.service'
    echo "$brd"
    echo '[Unit]' > /etc/systemd/system/spawn-fcgi.service
    echo 'Description=Spawn-fcgi startup service by Otus' >> /etc/systemd/system/spawn-fcgi.service
    echo 'After=network.target' >> /etc/systemd/system/spawn-fcgi.service
    echo '' >> /etc/systemd/system/spawn-fcgi.service
    echo '[Service]' >> /etc/systemd/system/spawn-fcgi.service
    echo 'Type=simple' >> /etc/systemd/system/spawn-fcgi.service
    echo 'PIDFile=/var/run/spawn-fcgi.pid' >> /etc/systemd/system/spawn-fcgi.service
    echo 'EnvironmentFile=/etc/spawn-fcgi/fcgi.conf' >> /etc/systemd/system/spawn-fcgi.service
    echo 'ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS' >> /etc/systemd/system/spawn-fcgi.service
    echo 'KillMode=process' >> /etc/systemd/system/spawn-fcgi.service
    echo '' >> /etc/systemd/system/spawn-fcgi.service
    echo '[Install]' >> /etc/systemd/system/spawn-fcgi.service
    echo 'WantedBy=multi-user.target' >> /etc/systemd/system/spawn-fcgi.service
    echo 'cat /etc/systemd/system/spawn-fcgi.service'
    
    cat /etc/systemd/system/spawn-fcgi.service

    echo "$brd"
    echo 'Разрешим и запустим сервис spawn-fcgi'
    echo "$brd"
    systemctl daemon-reload
    systemctl enable spawn-fcgi
    systemctl start spawn-fcgi
    echo 'systemctl status spawn-fcgi'
    systemctl status spawn-fcgi

    echo "$brd"
    echo "$brd"
    echo "$brd"
    echo 'Запускаем два экземпляра nginx'
    echo "$brd"
    echo "$brd"
    echo "$brd"

    echo 'Установим nginx'
    echo "$brd"

    echo 'apt install -y nginx'
    
    apt install -y nginx
    echo "$brd"
    echo 'Копируем файл сервиса nginx и редактируем полученную копию: /lib/systemd/system/nginx@.service'
    echo "$brd"
    cp -p /lib/systemd/system/nginx.service /lib/systemd/system/nginx@.service
    sed -i.bak 's/nginx.pid/nginx-%I.pid/' /lib/systemd/system/nginx@.service
    sed -i 's/nginx -t -q -g/nginx -t -c \\/etc\\/nginx\\/nginx-%I.conf -q -g/' /lib/systemd/system/nginx@.service
    sed -i 's/nginx -g/nginx -c \\/etc\\/nginx\\/nginx-%I.conf -g/' /lib/systemd/system/nginx@.service
    echo 'cat /lib/systemd/system/nginx@.service'
    
    cat /lib/systemd/system/nginx@.service

    echo "$brd"
    echo 'Копируем файл настроек nginx в две копии - nginx-first.conf и nginx-second.conf. Редактируем полученные копии.'
    echo "$brd"
    cp -p /etc/nginx/nginx.conf /etc/nginx/nginx-first.conf && cp -p $_ /etc/nginx/nginx-second.conf
    sed -i.bak 's/nginx.pid/nginx-first.pid/' /etc/nginx/nginx-first.conf
    sed -i.bak 's/nginx.pid/nginx-second.pid/' /etc/nginx/nginx-second.conf
    sed -i 's/http {/http { server { listen 8081; }/' /etc/nginx/nginx-first.conf
    sed -i 's/http {/http { server { listen 8082; }/' /etc/nginx/nginx-second.conf
    sed -i 's/include \\/etc\\/nginx\\/sites-en/#include \\/etc\\/nginx\\/sites-en/' /etc/nginx/nginx-first.conf
    sed -i 's/include \\/etc\\/nginx\\/sites-en/#include \\/etc\\/nginx\\/sites-en/' /etc/nginx/nginx-second.conf
    echo 'Мы изменили путь к pid-файлу, порты, на которых работают копии экземпляра nginx, а также убрали ссылки на файл сайта по умолчанию.'
    echo 'cat /etc/nginx/nginx-first.conf | grep ".pid"'
    cat /etc/nginx/nginx-first.conf | grep '.pid'
    echo 'cat /etc/nginx/nginx-second.conf | grep ".pid"'
    cat /etc/nginx/nginx-second.conf | grep ".pid"
    echo 'cat /etc/nginx/nginx-first.conf | grep "listen"'
    cat /etc/nginx/nginx-first.conf | grep 'listen'
    echo 'cat /etc/nginx/nginx-second.conf | grep "listen"'
    cat /etc/nginx/nginx-second.conf | grep 'listen'
    echo 'cat /etc/nginx/nginx-first.conf | grep "sites-en"'
    cat /etc/nginx/nginx-first.conf | grep 'sites-en'
    echo 'cat /etc/nginx/nginx-second.conf | grep "sites-en"'
    cat /etc/nginx/nginx-second.conf | grep 'sites-en'

    echo "$brd"
    echo 'Разрешим и запустим сервисы nginx@'
    echo "$brd"
    systemctl daemon-reload
    systemctl enable nginx@first.service
    systemctl enable nginx@second.service
    systemctl start nginx@first.service
    systemctl start nginx@second.service

    echo "$brd"
    echo 'Убедимся, что наши экземпляры nginx "слушают" нужные порты (8081 и 8082)'
    echo "$brd"
    
    ss -ntlp
    
    echo "$brd"
    echo 'Проверяем статус первого экземпляра nginx'
    echo "$brd"
    
    echo 'systemctl status nginx@first.service'
    
    systemctl status nginx@first.service
    
    echo "$brd"
    echo 'Проверяем статус второго экземпляра nginx'
    echo "$brd"
    
    echo 'systemctl status nginx@second.service'
    
    systemctl status nginx@second.service
    
    echo "$brd"
    echo 'Так как apache занимает порт 80/tcp, то изменим порт по умолчанию для основного экземпляра nginx на 8080. Для этого отредактируем файл /etc/nginx/sites-enabled/default'
    
    echo 'После этого запустим его и проверим прослушиваемые порты на хосте, а также статус nginx.service'
    echo "$brd"
    
    echo 'sed -i 's/listen 80 /listen 8080 /' /etc/nginx/sites-enabled/default'
    echo 'sed -i 's/listen \\[/#listen \\[/' /etc/nginx/sites-enabled/default'
    sed -i 's/listen 80 /listen 8080 /' /etc/nginx/sites-enabled/default
    sed -i 's/listen \\[/#listen \\[/' /etc/nginx/sites-enabled/default
    echo 'cat /etc/nginx/sites-enabled/default | grep "listen"'
    
    cat /etc/nginx/sites-enabled/default | grep "listen"
    
    systemctl start nginx.service
    echo 'ss -ntlp'
    
    ss -ntlp
    
    echo 'systemctl status nginx.service'
    
    systemctl status nginx.service

    echo "$brd"
    echo 'В задании "Сервис для мониторинга журнала", чтение лог-файла происходит каждые 30 сек. Проверим, что нужные записи попали в системный журнал'
    echo "$brd"
    
    echo 'journalctl -b | grep "Jedi"'
    
    journalctl -b | grep "Jedi"

      SHELL
  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
    end
  # config.vm.define "Debian12c" do |clnt|
  #   clnt.vm.box = "/home/max/vagrant/images/debian12"
  #   clnt.vm.provider "libvirt" do |lv|
  #   lv.memory = "2048"
  #   lv.cpus = "2"
  #   lv.title = "Debian12c"
  #   lv.description = "Виртуальная машина на базе дистрибутива Debian Linux"
  #   lv.management_network_name = "vagrant-libvirt-mgmt"
  #   lv.management_network_address = "192.168.121.0/24"
  #   lv.management_network_keep = "true"
  #   lv.management_network_mac = "52:54:00:27:28:84"
  #   lv.storage :file, :size => '1G', :device => 'vdb', :allow_existing => false
  # end
  # clnt.vm.provision "shell", inline: <<-SHELL
  #   brd='*************************************************************'
  #   echo "$brd"
  #   echo 'Изменим ttl для работы через раздающий телефон'
  #   echo "$brd"
  #   sysctl -w net.ipv4.ip_default_ttl=66
  #   echo "$brd"
  #   SHELL
  # end

  # config.vm.define "ArchLinux1" do |srv1|
  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
    # srv1.vm.box = "/home/max/vagrant/images/debian12"
  # config.vm.provider "libvirt" do |lv|
  #   lv.memory = "1024"
  #   lv.cpu = "2"
  # end
  # end
end

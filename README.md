### Systemd
  - ***systemd*** — набор базовых компонентов Linux-системы. Представляет собой менеджер системы и служб, который выполняется как процесс с PID 1 и запускает остальную часть системы. 
  - ***systemd*** обеспечивает возможности агрессивной параллелизации, сокетную и D-Bus активацию для запуска служб, запуск демонов по запросу, отслеживание процессов с помощью контрольных групп Linux, обслуживание точек (авто)монтирования, а также предлагает развитую транзакционную логику управления службами на основе зависимостей. 
  - ***systemd*** поддерживает сценарии инициализации SysV и LSB и работает как замена ***sysvinit***. Среди прочих элементов и функций — демон журнала, утилиты управления базовой конфигурацией системы (имя хоста, дата, языковой стандарт), ведение списков вошедших в систему пользователей, запущенных контейнеров, виртуальных машин, системных учётных записей, каталогов и настроек времени выполнения, а также демоны для управления несложными сетевыми конфигурациями, синхронизации времени по сети, пересылки журналов и разрешения имён.
[Источник](https://wiki.archlinux.org/title/Systemd_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9))

Цель работы:
  - понимать различие систем инициализации;
  - использовать основные утилиты systemd;
  - изучить состав и синтаксис systemd unit.


#### Подготовка окружения
В нашем примере используется гипервизор Qemu-KVM, библиотека Libvirt. В качестве хостовой системы - OpenSuse Leap 15.5. Автоматическое разворачивание стенда осуществляется с помощью Vagrant.

Для работы Vagrant с Libvirt установлен пакет vagrant-libvirt:
```
Сведения — пакет vagrant-libvirt:
---------------------------------
Репозиторий            : Основной репозиторий
Имя                    : vagrant-libvirt
Версия                 : 0.10.2-bp155.1.19
Архитектура            : x86_64
Поставщик              : openSUSE
Размер после установки : 658,3 KiB
Установлено            : Да
Состояние              : актуален
Пакет с исходным кодом : vagrant-libvirt-0.10.2-bp155.1.19.src
Адрес источника        : https://github.com/vagrant-libvirt/vagrant-libvirt
Заключение             : Провайдер Vagrant для libvirt
Описание               : 

    This is a Vagrant plugin that adds a Libvirt provider to Vagrant, allowing
    Vagrant to control and provision machines via the Libvirt toolkit.
```
Пакет Vagrant также устанавливаем из репозиториев. Текущая версия для OpenSuse Leap 15.5:
```
max@localhost:~/vagrant/vg3> vagrant -v
Vagrant 2.2.18
```
Образ операционной системы создан заранее, для этого установлен [Debian Linux из официального образа netinst](https://www.debian.org/distrib/netinst)

#### Vagrantfile
Все основные параметры виртуальных машин задаются в блоке ***vm.provider***. Для сервера, на котором будет создан репозиторий настройки выглядят таким образом:
```
Vagrant.configure("2") do |config|
  config.vm.define "Debian12" do |srv|
    srv.vm.box = "/home/max/vagrant/images/debian12"
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
```
Здесь заданы:
  - кол-во процессоров,
  - размер оперативной памяти,
  - настройки сетевого интерфейса,
  - дополнительгые дисковые устройства.

Для сетевого адаптера указан определенный MAC-адрес, на основании этого значения для виртуальной машины будет зарезервирован ip-адрес - ***192.168.121.10/32***.

#### Пост установочная настройка - Provisioning
#### Сервис для мониторинга журнала.
Здесь мы создаём сервис watchlog, который будет искать запись "ALERT" в журнале /var/log/watchlog и записывать событие о найденой записи в системный журнал'. 

Создаём файл с конфигурацией для сервиса в директории /etc/default.
```
    echo '# Configuration file for my watchlog service' >> /etc/default/watchlog
    echo '# Place it to /etc/default' >> /etc/default/watchlog
    echo '# File and word in that file that we will be monit' >> /etc/default/watchlog
    echo 'WORD="ALERT"' >> /etc/default/watchlog
    echo 'LOG=/var/log/watchlog.log' >> /etc/default/watchlog
```

Создадим "журнал" - файл, который будет содержать искомое ключевое слово.
```
    touch /var/log/watchlog.log
    echo 'This is ALERT!!!' >> /var/log/watchlog.log
```

Создадим скрипт и сделаем его исполняемым.
```
    if  [ ! -d $scriptdir ]
        then mkdir $scriptdir
        fi
    echo '#!/bin/bash' >> /opt/watchlog.sh
    echo 'WORD=$1' >> /opt/watchlog.sh
    echo 'LOG=$2' >> /opt/watchlog.sh
    echo 'DATE=`date`' >> /opt/watchlog.sh
    echo 'if grep $WORD $LOG &> /dev/null; then echo "$DATE: I found word, Jedi!" | logger; else exit 0; fi' >> /opt/watchlog.sh

    echo "$brd"
    echo 'Добавим права на запуск файла'
    echo "$brd"
    sleep 5

    chmod u+x /opt/watchlog.sh
```

Создадим юниты для сервиса и для таймера.
```
    echo "$brd"
    echo 'Создадим юнит для сервиса'
    echo "$brd"
    sleep 5

    echo '[Unit]' >> /etc/systemd/system/watchlog.service
    echo 'Description=My watchlog service' >> /etc/systemd/system/watchlog.service
    echo '' >> /etc/systemd/system/watchlog.service
    echo '[Service]' >> /etc/systemd/system/watchlog.service
    echo 'Type=oneshot' >> /etc/systemd/system/watchlog.service
    echo 'EnvironmentFile=/etc/default/watchlog' >> /etc/systemd/system/watchlog.service
    echo 'ExecStart=/opt/watchlog.sh $WORD $LOG' >> /etc/systemd/system/watchlog.service

    echo "$brd"
    echo 'Создадим юнит для таймера'
    echo "$brd"
    sleep 5

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
```

Запустим юнит таймера
```
    systemctl daemon-reload
    systemctl enable watchlog.timer
    systemctl start watchlog.service
    systemctl start watchlog.timer
```

#### Установка spawn-fcgi и создание unit-файла spawn-fcgi.sevice
Установим необходимые для работы пакеты.
```
    apt install -y spawn-fcgi php php-cgi php-cli apache2 libapache2-mod-fcgid
```
Создадим файл конфигурации для spawn-fcgi.
```
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
```
Создадим юнит и запустим сервис.
```
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

    systemctl daemon-reload
    systemctl enable spawn-fcgi
    systemctl start spawn-fcgi
```
#### Доработка unit-файла Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно
Ставим пакет ***nginx***
```
    apt install -y nginx
```
Копируем файл описания сервиса ***nginx*** и редактируем его копию для наших целей.
```
    cp -p /lib/systemd/system/nginx.service /lib/systemd/system/nginx@.service
    sed -i.bak 's/nginx.pid/nginx-%I.pid/' /lib/systemd/system/nginx@.service
    sed -i 's/nginx -t -q -g/nginx -t -c \\/etc\\/nginx\\/nginx-%I.conf -q -g/' /lib/systemd/system/nginx@.service
    sed -i 's/nginx -g/nginx -c \\/etc\\/nginx\\/nginx-%I.conf -g/' /lib/systemd/system/nginx@.service
```
Копируем основной файл конфигурации nginx и адаптируем копии:
```
    cp -p /etc/nginx/nginx.conf /etc/nginx/nginx-first.conf && cp -p $_ /etc/nginx/nginx-second.conf
    sed -i.bak 's/nginx.pid/nginx-first.pid/' /etc/nginx/nginx-first.conf
    sed -i.bak 's/nginx.pid/nginx-second.pid/' /etc/nginx/nginx-second.conf
    sed -i 's/http {/http { server { listen 8081; }/' /etc/nginx/nginx-first.conf
    sed -i 's/http {/http { server { listen 8082; }/' /etc/nginx/nginx-second.conf
    sed -i 's/include \\/etc\\/nginx\\/sites-en/#include \\/etc\\/nginx\\/sites-en/' /etc/nginx/nginx-first.conf
    sed -i 's/include \\/etc\\/nginx\\/sites-en/#include \\/etc\\/nginx\\/sites-en/' /etc/nginx/nginx-second.conf
```

Для ***systemd*** отправляем команду перчитывания конфигураций и запускаем наши дополнительные экземпляры nginx. Также изменим настройки для основного экземпляра nginx, чтобы избежть конфликта с ***apache***.
```
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
    systemctl status nginx@first.service
    echo "$brd"
    echo 'Проверяем статус второго экземпляра nginx'
    echo "$brd"
    systemctl status nginx@second.service
    echo "$brd"
    echo 'Так как apache занимает порт 80/tcp, то изменим порт по умолчанию для основного экземпляра nginx на 8080.'
    echo 'После этого запустим его и проверим прослушиваемые порты на хосте, а также статус nginx.service'
    echo "$brd"
    sed -i 's/listen 80 /listen 8080 /' /etc/nginx/sites-enabled/default
    sed -i 's/listen \\[/#listen \\[/' /etc/nginx/sites-enabled/default
    systemctl start nginx.service
    ss -ntlp
    systemctl status nginx.service
```

Также проверим, что в первом задании поиск ключевой фразы работает корректно.
```
    journalctl -b | grep "Jedi"
```
К данной работе прилагаю также запись консоли. Для того, чтобы воспроизвести выполненные действия,
необходимо скачать файлы [screenrecord-2024-07-30.script](screenrecord-2024-07-30.script) и [screenrecord-2024-07-30.time](screenrecord-2024-07-30.time),
после чего выполнить в каталоге с загруженными файлами команду:
```
scriptreplay ./screenrecord-2024-07-30.time ./screenrecord-2024-07-30.script
```
Спасибо за прочтение! :potted_plant:

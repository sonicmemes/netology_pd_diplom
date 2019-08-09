

## **установка СУБД**

    sudo nano  /etc/apt/sources.list.d/pgdg.list
    
    ----->
    deb http://apt.postgresql.org/pub/repos/apt/ bionic-pgdg main
    <<----
    
    
    wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
    
    sudo apt-get update
    
    sudo apt-get install postgresql-10 postgresql-server-dev-10
    
    sudo -u postgres psql postgres
    
    create user mploy with password 'password';
    
    alter role mploy set client_encoding to 'utf8';
    
    alter role mploy set default_transaction_isolation to 'read committed';
    
    alter role mploy set timezone to 'Asia/Almaty';
    
    create database mploy owner mploy;
    alter user mploy createdb;


## **Установка RabbitMQ**

    wget -O - 'https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc' | sudo apt-key add -
    
    echo "deb https://dl.bintray.com/rabbitmq/debian bionic main erlang" | sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list
    
    
    sudo apt-get update
    
    sudo apt-get install rabbitmq-server
    
    sudo rabbitmqctl add_user myuser mypassword
    
    sudo rabbitmqctl add_vhost myvhost
    
    sudo rabbitmqctl set_user_tags myuser mytag
    
    sudo rabbitmqctl set_permissions -p myvhost myuser ".*" ".*" ".*"

**Статус**

    sudo rabbitmqctl status

**Остановка**

    sudo rabbitmq-server -detached

## **Установка Redis**

    sudo apt install redis
    sudo nano /etc/redis/redis.conf
    ----->> 
    supervised systemd
    maxmemory 1500mb
    maxmemory-policy allkeys-lru
    <<----

## **Получить исходный код**

    git config --global user.name "YOUR_USERNAME"
    
    git config --global user.email "your_email_address@example.com"
    
    mkdir ~/mploy
    
    cd mploy
    
    git clone git@gitlab.com:A.Iskakov/jobstoday.git
    
    cd jobstoday
    
    sudo pip3 install  --upgrade pip
    
    sudo pip3 install -r req.txt
    
    python3 src/manage.py makemigrations
     
    python3 src/manage.py migrate
    
    python3 src/manage.py createsuperuser
    
    python3 src/manage.py collectstatic
    
    python3 src/manage.py download_photos
    
    python3 src/manage.py initial_db_data


**Создать полученный от админа файл настроек**

    nano src/hragency/local_settings.py

    python3 -m compileall src


**Проверяем работу модулей**

    cd src
    
    gunicorn -w 2 --threads 2 hragency.wsgi:application
    
    celery worker --app=hragency --loglevel=info
    
    celery -A hragency beat -l info --scheduler django_celery_beat.schedulers:DatabaseScheduler


**Создаем сервис gunicorn**

    sudo nano /etc/systemd/system/gunicorn.service
    ---->>

    [Unit]
    
    Description=gunicorn daemon
    After=network.target
    
    [Service]
    
    PIDFile=/run/gunicorn/pid
    User=ubuntu
    Group=ubuntu
    
    RuntimeDirectory=gunicorn
    WorkingDirectory=/home/ubuntu/mploy/jobstoday/src
    
    ExecStart=/usr/local/bin/gunicorn --pid /run/gunicorn/pid -w 2 hragency.wsgi:application
    
    ExecReload=/bin/kill -s HUP $MAINPID
    ExecStop=/bin/kill -s TERM $MAINPID
    PrivateTmp=true
    
    [Install]
    
    WantedBy=multi-user.target
    <<----


    sudo nano /etc/tmpfiles.d/gunicorn.conf
    
    d /run/gunicorn 0755 ubuntu ubuntu -

**Запускаем сервисы** 

    sudo systemctl enable gunicorn.service
    
    sudo systemctl start gunicorn.service
    
    sudo systemctl status gunicorn.service


**Создаем сервис celery**

    sudo nano /etc/systemd/system/celery.service
    ---->>

    [Unit]
    
    Description=celery daemon
    
    After=network.target
    
    [Service]
    
    PIDFile=/run/celery/pid
    
    User=ubuntu
    
    Group=ubuntu
    
    RuntimeDirectory=celery
    
    WorkingDirectory=/home/ubuntu/mploy/jobstoday/src
    
    ExecStart=/usr/local/bin/celery worker --app=hragency --loglevel=info
    
    ExecReload=/bin/kill -s HUP $MAINPID
    
    ExecStop=/bin/kill -s TERM $MAINPID
    
    PrivateTmp=true
    
    [Install]
    
    WantedBy=multi-user.target
    <<----

    sudo nano /etc/tmpfiles.d/celery.conf
    
    ---->>
    d /run/celery 0755 ubuntu ubuntu -
    <<----

**Запускаем сервисы** 

    sudo systemctl enable celery.service
    
    sudo systemctl start celery.service
    
    sudo systemctl status celery.service


**Создаем сервис celery-beat**

    sudo nano /etc/systemd/system/celery-beat.service
    ---->>

    [Unit]
    
    Description=celery-beat daemon
    
    After=network.target
    
    [Service]
    
    PIDFile=/run/celery-beat/pid
    
    User=ubuntu
    
    Group=ubuntu
    
    RuntimeDirectory=celery-beat
    
    WorkingDirectory=/home/ubuntu/mploy/jobstoday/src
    
    ExecStart=/usr/local/bin/celery -A hragency beat -l info --scheduler django_celery_beat.schedulers:DatabaseScheduler
    
    ExecReload=/bin/kill -s HUP $MAINPID
    
    ExecStop=/bin/kill -s TERM $MAINPID
    
    PrivateTmp=true
    
    [Install]
    
    WantedBy=multi-user.target
    <<----



    sudo nano /etc/tmpfiles.d/celery-beat.conf
    ---->>

    d /run/celery-beat 0755 ubuntu ubuntu -
    <<----

**Запускаем сервисы** 

    sudo systemctl enable celery-beat.service
    
    sudo systemctl start celery-beat.service
    
    sudo systemctl status celery-beat.service
    
    


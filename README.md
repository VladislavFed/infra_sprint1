# Запуск проекта Kittygram на удалённом сервере

Kittygram — социальная сеть для обмена фотографиями любимых питомцев. Это полностью рабочий проект, который состоит из бэкенд-приложения на Django и фронтенд-приложения на React. 

- Можно зарегистрироваться и авторизоваться, добавить нового котика на сайт или изменить существующего. 
- Есть возможность просмотреть записи других пользователей
### Технологии
- Python 3.10
- Django 3.2.3
- Django REST Framework 3.12.4
- Pillow 9.0.0
- Gunicorn 20.1.0
- Nginx 1.18.0
- certbot 2.6.0
### Загрузка проекта на удаленный сервер

- В директории .ssh создайте новую директорию для SSH-ключей
```
mkdir название-директории 
```
- Перенесите файлы SSH-ключей в только что созданную директорию
- Установите права доступа к файлам SSH-ключей с помощью команд:
```
chmod 600 название_файла_закрытого_SSH-ключа
```
```
chmod 644 название_файла_открытого_SSH-ключа.pub 
```
- Подключитесь к серверу
```
ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом_без_расширения login@ip
```
- Подключите сервер к аккаунту на GitHub
    - Установите на сервер Git
    - Сгенерируйте SSH ключ для Github
    ```
    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    ```
    - Cохраните открытый ключ в вашем аккаунте на GitHub.
    - Клонируйте проект на сервер
    ```
    git clone git@github.com:Ваш_аккаунт/infra_sprint1.git 
    ```
- Запустите бэкенд
    - Установите на сервер интерпретатор Python, менеджер пакетов pip, утилиту для создания виртуального окружения venv
    ```
    sudo apt install python3-pip python3-venv -y
    ```
    - Создайте виртуальное окружение
    ```
    python3 -m venv venv
    ```
    - Активируйте виртуальное окружение
    ```
    source venv/bin/activate
    ```
    - Устанавите зависимости
    ```
    pip install -r requirements.txt
    ```
    - Выполните миграции
    ```
    python3 manage.py migrate
    ```
    - Создайте суперпользователя
    ```
    python3 manage.py createsuperuser 
    ```
    - Добавьте в список ALLOWED_HOSTS внешний IP-адрес , адреса 127.0.0.1 и localhost
    ```
    ALLOWED_HOSTS = ['xxx.xxx.xxx.xxx', '127.0.0.1', 'localhost']
    ```
- Запустите фронтенд
    - Установите на сервер пакетный менеджер npm
    ```
    curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
    sudo apt-get install -y nodejs
    ```
    - установите зависимости для фронтенд-приложения. Перейдите в директорию infra_sprint1/frontend/ и выполните команду:
    ```
    npm i
    ```
- Установите и запустите Gunicorn
    - Установите пакет gunicorn:
    ```
    pip install gunicorn==20.1.0
    ```
    - Запустите gunicorn:
    ```
    gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi
    ```
    - Установите в файле settings.py для константы DEBUG значение False
     ```
     sudo nano settings.py
     ```

- Создайте юнит для Gunicorn
    - Создайте конфигурационный файл Gunicorn для проекта Kittygram
    ```
    sudo nano /etc/systemd/system/gunicorn_kittygram.service
    ```
    - В файле gunicorn_kittygram.service опишите конфигурацию процесса
    ```
    [Unit]

    Description=gunicorn daemon

    After=network.target

    [Service]

    User=yc-user

    WorkingDirectory=/home/yc-user/infra_sprint1/backend/

    ExecStart=/home/yc-user/infra_sprint1/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi

    [Install]

    WantedBy=multi-user.target
    ```
- Запустите процесс gunicorn_kittygram.service
    ```
    sudo systemctl start gunicorn
    ```
- Установите Nginx
    - Из любой директории выполните команду:
    ```
    sudo apt install nginx -y
    ```
    - Укажите файрволу, какие порты должны остаться открытыми:
    ```
    sudo ufw allow 'Nginx Full'
    sudo ufw allow OpenSSH 
    ```
    - Включите файрвол:
    ```
    sudo ufw enable
    ```
- Соберите статику фронтенд-приложения
    - Перейдите в директорию infra_sprint1/frontend и выполните команду:
    ```
    npm run build
    ```
    - Скопируйте в системную директорию /var/www/ содержимое папки .../frontend/build/:
    ```
    sudo cp -r <имя_пользователя>/infra_sprint1/frontend/build/. /var/www/kittygram/
    ```
    - Создайте директорию media в директории /var/www/kittygram/
    ```
    sudo mkdir media
    ```
    - В настройках бэкенда для константы MEDIA_ROOT укажите путь до созданной директории media:
    ```
    MEDIA_ROOT = '/var/www/kittygram/media'
    ```
    - Назначьте текущего пользователя владельцем директории media:
    ```
    sudo chown -R <имя_пользователя> /var/www/kittygram/media/
    ```
- Настройте файл конфигурации веб-сервера
    - Через редактор Nano откройте файл конфигурации веб-сервера:
    ```
    sudo nano /etc/nginx/sites-enabled/default
    ```
    - Добавьте новые настройки:
    ```
    server {

        server_name vladkittygram.hopto.org;

        location /api/ {
            proxy_pass http://127.0.0.1:8080;
        }

        location /admin/ {
            proxy_pass http://127.0.0.1:8080;
        }

        location /media/ {
            root /var/www/kittygram;
        }

        location / {
            root   /var/www/kittygram;
            index  index.html index.htm;
            try_files $uri /index.html;
        }

    }
    ```
    - Проверьте корректность конфигурации и перезагрузите её:
    ```
    sudo nginx -t
    sudo systemctl reload nginx
    ```
- Получите доменное имя, по которому будет доступно приложение
    - Используйте любой сервис, выдающий доменные имена, например No-IP
    - Добавьте доменное имя в настройки Django
    ```
    ALLOWED_HOSTS = ['xxx.xxx.xxx.xxx', '127.0.0.1', 'localhost', 'vladkittygram.hopto.org']
    ```
    - Перезапустите Gunicorn
    ```
    sudo systemctl restart gunicorn
    ```
    - Пропишите в конфигурационном файле Nginx выданное вам доменное имя:
    ```
        server {
    ...
        server_name <ваш-ip> vladkittygram.hopto.org;
    ...
    } 
    ```
    - Проверьте корректность конфигурации и перезагрузите её:
    ```
    sudo nginx -t
    sudo systemctl reload nginx
    ```
- Настройте шифрование запросов по протоколу HTTPS
    - Установите certbot на ваш удалённый сервер
    ```
    sudo apt install snapd
    sudo snap install core; sudo snap refresh core
    sudo snap install --classic certbot
    sudo ln -s /snap/bin/certbot /usr/bin/certbot
    ```
    - Чтобы начать процесс получения сертификата, введите команду:
    ```
    sudo certbot --nginx
    ```
    - Перезагрузите конфигурацию Nginx:
    ```
    sudo systemctl reload nginx
    ```
    - Настройте автоматическое обновление SSL-сертификата
    ```
    sudo certbot renew --dry-run
    ```
- На удалённом сервере сделайте push проекта Kittygram в свой репозиторий на GitHub
```
git add .
git commit -m '<комментарий>'
git push
```

### Автор
Федотов Владислав [GitHub](https://github.com/VladislavFed)

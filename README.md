# ������ ������� Kittygram �� �������� �������

Kittygram � ���������� ���� ��� ������ ������������ ������� ��������. ��� ��������� ������� ������, ������� ������� �� ������-���������� �� Django � ��������-���������� �� React. 

- ����� ������������������ � ��������������, �������� ������ ������ �� ���� ��� �������� �������������. 
- ���� ����������� ����������� ������ ������ �������������
### ����������
- Python 3.10
- Django 3.2.3
- Django REST Framework 3.12.4
- Pillow 9.0.0
- Gunicorn 20.1.0
- Nginx 1.18.0
- certbot 2.6.0
### �������� ������� �� ��������� ������

- � ���������� .ssh �������� ����� ���������� ��� SSH-������
```
mkdir ��������-���������� 
```
- ���������� ����� SSH-������ � ������ ��� ��������� ����������
- ���������� ����� ������� � ������ SSH-������ � ������� ������:
```
chmod 600 ��������_�����_���������_SSH-�����
```
```
chmod 644 ��������_�����_���������_SSH-�����.pub 
```
- ������������ � �������
```
ssh -i ����_��_�����_�_SSH_������/��������_�����_�_SSH_������_���_���������� login@ip
```
- ���������� ������ � �������� �� GitHub
    - ���������� �� ������ Git
    - ������������ SSH ���� ��� Github
    ```
    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    ```
    - C�������� �������� ���� � ����� �������� �� GitHub.
    - ���������� ������ �� ������
    ```
    git clone git@github.com:���_�������/infra_sprint1.git 
    ```
- ��������� ������
    - ���������� �� ������ ������������� Python, �������� ������� pip, ������� ��� �������� ������������ ��������� venv
    ```
    sudo apt install python3-pip python3-venv -y
    ```
    - �������� ����������� ���������
    ```
    python3 -m venv venv
    ```
    - ����������� ����������� ���������
    ```
    source venv/bin/activate
    ```
    - ���������� �����������
    ```
    pip install -r requirements.txt
    ```
    - ��������� ��������
    ```
    python3 manage.py migrate
    ```
    - �������� �����������������
    ```
    python3 manage.py createsuperuser 
    ```
    - �������� � ������ ALLOWED_HOSTS ������� IP-����� , ������ 127.0.0.1 � localhost
    ```
    ALLOWED_HOSTS = ['xxx.xxx.xxx.xxx', '127.0.0.1', 'localhost']
    ```
- ��������� ��������
    - ���������� �� ������ �������� �������� npm
    ```
    curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
    sudo apt-get install -y nodejs
    ```
    - ���������� ����������� ��� ��������-����������. ��������� � ���������� infra_sprint1/frontend/ � ��������� �������:
    ```
    npm i
    ```
- ���������� � ��������� Gunicorn
    - ���������� ����� gunicorn:
    ```
    pip install gunicorn==20.1.0
    ```
    - ��������� gunicorn:
    ```
    gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi
    ```
    - ���������� � ����� settings.py ��� ��������� DEBUG �������� False
     ```
     sudo nano settings.py
     ```

- �������� ���� ��� Gunicorn
    - �������� ���������������� ���� Gunicorn ��� ������� Kittygram
    ```
    sudo nano /etc/systemd/system/gunicorn_kittygram.service
    ```
    - � ����� gunicorn_kittygram.service ������� ������������ ��������
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
- ��������� ������� gunicorn_kittygram.service
    ```
    sudo systemctl start gunicorn
    ```
- ���������� Nginx
    - �� ����� ���������� ��������� �������:
    ```
    sudo apt install nginx -y
    ```
    - ������� ��������, ����� ����� ������ �������� ���������:
    ```
    sudo ufw allow 'Nginx Full'
    sudo ufw allow OpenSSH 
    ```
    - �������� �������:
    ```
    sudo ufw enable
    ```
- �������� ������� ��������-����������
    - ��������� � ���������� infra_sprint1/frontend � ��������� �������:
    ```
    npm run build
    ```
    - ���������� � ��������� ���������� /var/www/ ���������� ����� .../frontend/build/:
    ```
    sudo cp -r <���_������������>/infra_sprint1/frontend/build/. /var/www/kittygram/
    ```
    - �������� ���������� media � ���������� /var/www/kittygram/
    ```
    sudo mkdir media
    ```
    - � ���������� ������� ��� ��������� MEDIA_ROOT ������� ���� �� ��������� ���������� media:
    ```
    MEDIA_ROOT = '/var/www/kittygram/media'
    ```
    - ��������� �������� ������������ ���������� ���������� media:
    ```
    sudo chown -R <���_������������> /var/www/kittygram/media/
    ```
- ��������� ���� ������������ ���-�������
    - ����� �������� Nano �������� ���� ������������ ���-�������:
    ```
    sudo nano /etc/nginx/sites-enabled/default
    ```
    - �������� ����� ���������:
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
    - ��������� ������������ ������������ � ������������� �:
    ```
    sudo nginx -t
    sudo systemctl reload nginx
    ```
- �������� �������� ���, �� �������� ����� �������� ����������
    - ����������� ����� ������, �������� �������� �����, �������� No-IP
    - �������� �������� ��� � ��������� Django
    ```
    ALLOWED_HOSTS = ['xxx.xxx.xxx.xxx', '127.0.0.1', 'localhost', 'vladkittygram.hopto.org']
    ```
    - ������������� Gunicorn
    ```
    sudo systemctl restart gunicorn
    ```
    - ��������� � ���������������� ����� Nginx �������� ��� �������� ���:
    ```
        server {
    ...
        server_name <���-ip> vladkittygram.hopto.org;
    ...
    } 
    ```
    - ��������� ������������ ������������ � ������������� �:
    ```
    sudo nginx -t
    sudo systemctl reload nginx
    ```
- ��������� ���������� �������� �� ��������� HTTPS
    - ���������� certbot �� ��� �������� ������
    ```
    sudo apt install snapd
    sudo snap install core; sudo snap refresh core
    sudo snap install --classic certbot
    sudo ln -s /snap/bin/certbot /usr/bin/certbot
    ```
    - ����� ������ ������� ��������� �����������, ������� �������:
    ```
    sudo certbot --nginx
    ```
    - ������������� ������������ Nginx:
    ```
    sudo systemctl reload nginx
    ```
    - ��������� �������������� ���������� SSL-�����������
    ```
    sudo certbot renew --dry-run
    ```
- �� �������� ������� �������� push ������� Kittygram � ���� ����������� �� GitHub
```
git add .
git commit -m '<�����������>'
git push
```

### �����
������� ��������� [GitHub](https://github.com/VladislavFed)

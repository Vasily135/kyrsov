### Конышев Василий Юрьевич M30-312Б-18.  
  
  ## 1) Установим репозиторий EPEL

    [vasily@localhost project]$ sudo yum install epel-release

## 2)Установим pip диспетчер пакетов Python

    [vasily@localhost project]$ sudo yum install python3-pip python3-devel gcc nginx  
  
## 3) Создаём виртуальную среду Python.  
    [vasily@localhost project]$ sudo pip3 install virtualenv  
    [vasily@localhost project]$ virtualenv projectvenv  
    [vasily@localhost project]$ source projectvenv/bin/activate  
    (projectvenv) [vasily@localhost project]$  
  
## 4) Создание и настройка приложения Flask.  
    (projectvenv) [vasily@localhost project]$ pip3 install wheel uwsgi flask  
    (projectvenv) [vasily@localhost project]$ vi ~/project/app.py  


## 5) Установим модули для работы приложения:
    
    (projectvenv) [vasily@localhost project]$ pip3 install Pillow opencv-python mesa-liblibGL.x86_64
    (projectvenv) [vasily@localhost project]$ python3 ~/project/application.py &  
    (projectenv) [vasily@localhost project]$ curl -L http://10.0.2.15:5000 | head -n 8  
    
> \<!DOCTYPE html>  
>  \<html lang="ru">  
>  \<head>  
>   \<meta charset="UTF-8">  
>   \<title>Image processing</title>  
>  \</head>  
> \<body>  
>   \<p class="fadein1">Загрузите изображения для красного и инфракрасного спектров в формате tif:</p>  
      
## 6)Остановим Flask приложение:  

    (projectvenv) [vasily@localhost project]$ fg  
    python3 app.py  
    ^C  
  
## 7) Создание точки входа WSGI.  

    (projectvenv) [vasily@localhost project]$ vi ~/project/wsgi.py  
    import application  
    if __name__ == "__main__":  
      app.run()  
  
## 8) Настройка конфигурации uWSGI.  
    (projectvenv) [vasily@localhost project]$ uwsgi --socket 0.0.0.0:8000 --protocol=http -w wsgi &  
    (projectvenv) [vasily@localhost project]$ curl -L http://10.0.2.15:8000 | head -n 5  
    
> \<!DOCTYPE html>  
> \<html lang="ru">  
> \<head>  
> \<meta charset="UTF-8">  
> \<title>Image processing</title>  

## 9) Приостановим uwsgi:  

    (projectvenv) [vasily@localhost project]$ fg  
    uwsgi --socket 0.0.0.0:8000 --protocol=http -w wsgi
    ^C  
    (projectvenv) [vasily@localhost project]$ deactivate  
## 10) Создадим файл конфигурации uWSGI:  

    [vasily@localhost project]$ vi ~/project/project.ini  
    [uwsgi]  
    module = wsgi  
    master = true  
    processes = 10  
    socket = project.sock  
    chmod-socket = 660  
    vacuum = true  

  ## 11) Создание файла модуля systemd.  
Создадим файл службы:  

    [vasily@localhost project]$ sudo vi /etc/systemd/system/project.service  
    [Unit]  
    Description=uWSGI for project  
    After=network.target  
    [Service]  
    User=vasily  
    Group=nginx  
    WorkingDirectory=/home/vasily/project  
    Environment="PATH=/home/vasily/project/projectvenv/bin"  
    ExecStart=/home/vasily/project/projectvenv/bin/uwsgi --ini project.ini  
    [Install]  
    WantedBy=multi-user.target  
    [vasily@localhost project]$ sudo systemctl daemon reload  
    [vasily@localhost project]$ sudo systemctl start project  
    [vasily@localhost project]$ sudo systemctl enable project  
  
## 12) Настройка Nginx.  
    [vasily@localhost project]$ sudo vi /etc/nginx/nginx.conf  

    ...  
      http {  
      ...  
        include /etc/nginx/conf.d/*.conf;  
        *
        server {  
            listen 80 default_server;  
            }  
      ...  
      }  
    ...  
 ## 13) Выше него (*) создадим свой:  
    server {  
      listen 80;  
      server_name 10.0.2.15;  
      
      location / {
        include uwsgi_params;
        uwsgi_pass unix:/home/vasily/project/project.sock;
    }

    [vasily@localhost project]$ sudo usermod -a -G vladislav nginx  
    [vasily@localhost project]$ chmod 710 /home/vladislav  
## 14) Запускаем Nginx:  

    [vasily@localhost project]$ sudo systemctl start nginx  
    [vasily@localhost project]$ sudo systemctl enable nginx  
  

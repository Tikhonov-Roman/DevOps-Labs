# Лабораторная работа №1  
**Выполнили:** <br />
Журбина Марина К3239 <br />
Тихонов Роман К3239 <br />


## Подготовка
Прочитав задание и погуглив про nginx, решили, что будем выполнять работу на Ubuntu, в качестве проектов будем использовать предложенный в задании код.
Для первого проекта main.py:
``` python
from starlette.applications import Starlette
from starlette.responses import HTMLResponse
from starlette.routing import Route

async def home(request):
    return HTMLResponse(f"""<pre>
    Project 1
    url: {str(request.url)}
    client:
        host: {request.client.host}
        port: {request.client.port}
    headers:
        {('<br>' + '&nbsp' * 12).join(k + ': ' + v for k, v in request.headers.items())}
</pre>""")

app = Starlette(debug=True, routes=[Route('/', home)])
```

Для второго проекта main.py:
``` python
from starlette.applications import Starlette
from starlette.responses import HTMLResponse
from starlette.routing import Route

async def home(request):
    return HTMLResponse(f"""<pre>
    Project 2
    url: {str(request.url)}
    client:
        host: {request.client.host}
        port: {request.client.port}
    headers:
        {('<br>' + '&nbsp' * 12).join(k + ': ' + v for k, v in request.headers.items())}
</pre>""")

app = Starlette(debug=True, routes=[Route('/', home)])
```

## Ход работы: 

1. Обновляем список доступных пакетов и устанавливаем nginx
```bash
sudo apt update
sudo apt install nginx
```
2. Создаем папки для наших проектов: project1 и project2 и созадем там указанные выше файлы main.py для кажого проекта
3. Создаем директории для alias\
  3.1 Для первого проекта по пути /var/www/project1/static создадим test.html:\
  <img width="902" height="82" alt="image" src="https://github.com/user-attachments/assets/05c23ca5-3ddf-4a93-961a-e70d6bd1f23f" />\
\
  3.2 Аналогично для второго проекта по пути /var/www/project2/asstes создадим test.html:\
   <img width="897" height="128" alt="image" src="https://github.com/user-attachments/assets/0de5840a-d164-42da-88e4-90ab7061e1e8" />\
4. Создадим SSL сертификат с помощью openssl:
```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048  -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt
```
5. Напишем файл конфигурации nginx по пути /etc/nginx/sites-available/default\
5.1 Настройка переадресации с http на https:
```
server {
listen 80;
server_name project1.local project2.local;
return 301 https://$host$request_uri;
} 
```
 
5.2 Настройка первого проекта:
```
server {
    listen 443 ssl;
    server_name project1.local;
    
    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;
    
    location / {
        proxy_pass http://127.0.0.1:8001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    location /static/ {
        alias /var/www/project1/static/;
    }
}
```

5.3 Настройка второго проекта:
```
server {
    listen 443 ssl;
    server_name project2.local;
    
    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;
    
    location / {
        proxy_pass http://127.0.0.1:8002;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    location /assets/ {
        alias /var/www/project2/assets/;
    }
}
```
6. Добавим домены в /etc/hosts:\
<img width="898" height="466" alt="image" src="https://github.com/user-attachments/assets/245b27cd-e2ad-4347-aa27-e42a6b4d40b5" />\
7. Проверим и перезапустим Nginx:
<img width="809" height="170" alt="image" src="https://github.com/user-attachments/assets/f9511ee4-907c-48c9-ae52-3b39eb2e83e3" />\
8. Установим python3-venv, создадим и активируем виртуальное окружение для установки необходимых пакетов:
```
sudo apt install python3-venv python3-pip
python3 -m venv ~/devops-lab-env
source ~/devops-lab-env/bin/activate
pip install uvicorn starlette
```
9. Запустим проекты:\
  <img width="1661" height="193" alt="image" src="https://github.com/user-attachments/assets/3c4d4748-a953-4458-90fa-b08faad7f390" />\

10. Проверка работы:\
10.1 перенаправление HTTP-запросов на HTTPS для каждого проекта:\
<img width="1333" height="89" alt="image" src="https://github.com/user-attachments/assets/4ae1c934-63bd-4ac1-9d9c-f5879ef0bdb5" />\
Перенаправление на https:\
<img width="1340" height="436" alt="image" src="https://github.com/user-attachments/assets/6d07e371-01ab-408f-9951-e84a7ea1d0af" />\
Для второго проекта:\
<img width="1336" height="220" alt="image" src="https://github.com/user-attachments/assets/5bf74d38-c951-4c9f-a54f-7e2ddd0add6b" />\
Перенаправление на https:\
<img width="1412" height="489" alt="image" src="https://github.com/user-attachments/assets/5c15b53f-0b65-4354-b538-078db31ae7dc" />\
10.2 Проверка alias:\
    <img width="1341" height="178" alt="image" src="https://github.com/user-attachments/assets/fbf89203-da19-4709-aa69-4ad35dc2cb01" />\
    <img width="1359" height="157" alt="image" src="https://github.com/user-attachments/assets/0cdbbf88-2cd3-422b-8095-5470ec98064a" />\
Вывод:
В ходе выполнения лабораторной работы была успешно настроена конфигурация nginx, соответствующая всем требованиям технического задания.





   

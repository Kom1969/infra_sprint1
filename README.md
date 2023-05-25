# Kittygram
### Описание
Проект о наших домашних питомцах, вы можете зарегистрироваться и добавлять ваших питомцев, нужно указать имя и год рождения питомца, а так же добавить фотографии и указать его умения. 
### Технологии
- Python - язык программирования.
- Django - свободный фреймворк для веб-приложений на языке Python.
- Django REST Framework - мощный и гибкий набор инструментов для создания веб-API.
- Simple JWT - плагин аутентификации JSON Web Token для Django REST Framework.
- Gunicorn 20.1.0 
- Фронтенд-приложение на React 
- npm 9.5.1 
- База данных - SQLite3 
### Запуск проекта через Bash 
- Подключитесь к удаленному серверу по SSH-ключу
```
Например:
ssh -i D:/Dev/vm_access/yc-yakovlevstepan yc-user@153.0.2.10 
```
- Клонирование кода приложения с GitHub на сервер
```
git clone git@github.com:Ваш_аккаунт/kittygram.git
```
### Запуск бэкенд
- Установите на сервер пакетный менеджер и утилиту для создания виртуального окружения
```
sudo apt install python3-pip python3-venv -y 
```
- Установите и активируйте виртуальное окружение 
```
# Создаём виртуальное окружение.
python3 -m venv venv
# Активируем виртуальное окружение.
source venv/bin/activate
```
- Установите зависимости из файла requirements.txt
```
pip install -r requirements.txt
``` 
- Выполните миграции и создайте суперюзера
```
# Применяем миграции.
python manage.py migrate
# Создаём суперпользователя.
python manage.py createsuperuser 
```
- В папке с файлом manage.py выполните команду:
```
python manage.py runserver
```

### Локальное хранение секретных ключей
- Перейдите в директорию с файлом settings.py и создайте файл .env
```
# Открываем файл .env
sudo nano .env 
```
- Пернесите в него переменные окружения из файла settings.py, образец показан в файде .env.example

### Сборка статики бэкенд-приложения
 
- Для сбора статики, в файле `settings.py` укажите директорию, куда эту статику нужно сложить.  
```
	# Замените стандартное значение 'static' на 'static_backend', 
	# чтобы не было конфликта запросов к статике фронтенда и бэкенда. 
	STATIC_URL = 'static_backend' 
 
	STATIC_ROOT = BASE_DIR / 'static_backend' 
 
	MEDIA_URL = '/media/' 
 
	MEDIA_ROOT = BASE_DIR / '/var/www/kittygram/media/' 
``` 
 
- Установите зависимости для статистики бекенда. 
```
	python manage.py collectstatic 
``` 
- Статистику скопируйте в систеную директорию 
```
	sudo cp -r kittygram/backend/static_backend/ /var/www/kittygram/ 
``` 
### Установка и запуск Gunicorn 
```
	pip install gunicorn==20.1.0 
``` 
Чтобы systemd следил за работой WSGI-сервера, вам нужно создать **пользовательский юнит** и прописать в нём все необходимые конфигурации, **он должен**: 
- запускаться при старте системы; 
- работать непрерывно; 
- перезапускаться, если по какой-то причине отключится. 
 
```
	sudo nano /etc/systemd/system/gunicorn-kittygram.service  
``` 
 
```
	[Unit]  
	Description=gunicorn-kittygram daemon  
	After=network.target  
 
	[Service]  
	User=yc-user  
	WorkingDirectory=/home/yc-user/infra_sprint1/backend/  
	ExecStart=/home/yc-user/infra_sprint1/backend/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi  
 
	[Install]  
	WantedBy=multi-user.target 
``` 
 
- Запустите процесс gunicorn-kittygram.service: 
```
	sudo systemctl start gunicorn-kittygram 
``` 
- Добавьте процесс Gunicorn в список автозапуска 
``` 
	sudo systemctl enable gunicorn-kittygram 
``` 
- Проверьте работает ли демон Gunicorn: 
``` 
	sudo systemctl status gunicorn-kittygram 
``` 
- При изменении юнита перезагрузите демон Gunicorn: 
```
	sudo systemctl restart gunicorn-kittygram 
``` 
### Запуск фронтенд 
- Установите на сервер пакетный менеджер `npm`. 
```
	curl -fsSL https://deb.nodesource.com/setup_18.x | 	sudo -E bash - &&\ 
	sudo apt-get install -y nodejs 
``` 
 
- Установите зависимости для фронтенд-приложения: 
```
	npm i 
``` 
 
- Установите Nginx: 
```
	sudo apt install nginx -y 
``` 
 
- Запустите Nginx: 
```
	sudo systemctl start nginx 
``` 
 
### Сбор статики фронтенд-приложения 
- Перейдите в директорию `infra_sprint1/frontend` и выполните команду: 
```
	npm run build 
``` 
- Скопируйте в эту директорию содержимое папки `.../frontend/build/`: 
```
	sudo cp -r /home/yc-user/taski/frontend/build/. /var/www/taski/  
``` 
 
### Настройки для работы со статикой фронтенд-приложения 
 
``` 
	sudo nano /etc/nginx/sites-enabled/default 
``` 
 
```
server { 
 
		listen 80; 
		server_name <публичный_ip_вашего_удалённого_сервера> <домен>; 
     
		location /api/ { 
			proxy_pass http://127.0.0.1:8000; 
		} 
 
		location /admin/ { 
			proxy_pass http://127.0.0.1:8000; 
		} 
 
		location /media/ { 
			alias /var/www/kittygram/media/; 
		} 
		 
		location / { 
			root   /var/www/kittygram; 
			index  index.html index.htm; 
			try_files $uri /index.html; 
		}  
 
} 
``` 
- Проверьте файл конфигурации на ошибки: 
```
	sudo nginx -t 
	# если всё хорошо будет выведено 
	# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok 
	# nginx: configuration file /etc/nginx/nginx.conf test is successful 
``` 
- Далее перезагрузите конфигурацию Nginx: 
```
	sudo systemctl reload nginx 
``` 

### Автор
```
- Михаил Корюкин
```
```
 https://github.com/Kom1969
```


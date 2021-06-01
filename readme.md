
### Требования к разворачиванию проекта  
PHP 7.4  
MySQL 5.7  
RabbitMQ  
IMAP Extension 

### Разворачивание проекта
* composer install  
* cp .env .env.local  
* cp config/packages/framework.yaml.example config/packages/framework.yaml  
* Раскомментировать строку php.ini ``extension = imap``. Если не установлено расширение, то установить.  
* Установить [RabbitMQ](https://www.rabbitmq.com/download.html) 

### Настройка для работы с почтой  

``IMAP_MAILBOX_TRASH`` - попадают письма с ненужным форматом  
``IMAP_UPD_MAILBOX_APPROVE`` - УПД, которые прошли проверку  
``IMAP_UPD_MAILBOX_VALIDATION_ERROR`` - УПД, которые с ошибками валидации  
``IMAP_PRICE_MAILBOX_APPROVE`` - Прайс, которые прошли проверку  
``IMAP_PRICE_MAILBOX_VALIDATION_ERROR`` - Прайсы, которые с ошибками валидации  
``IMAP_ORDER_MAILBOX_CONFIRM`` - Подтвержденные заказы  
``IMAP_ORDER_MAILBOX_CHANGED`` - Измененные заказы  
``IMAP_ORDER_MAILBOX_CANCELED`` - Отмененные заказы  

````
**Пример .env**

IMAP_MAILBOX_TRASH="Trash"

IMAP_UPD_MAILBOX_APPROVE="UPD.Approve"  
IMAP_UPD_MAILBOX_VALIDATION_ERROR="UPD.ValidationError"  

IMAP_PRICE_MAILBOX_APPROVE="Prices.Approve"  
IMAP_PRICE_MAILBOX_VALIDATION_ERROR="Prices.ValidationError"  

IMAP_ORDER_MAILBOX_CONFIRM="Order.Confirm"  
IMAP_ORDER_MAILBOX_CHANGED="Orders.Changed"  
IMAP_ORDER_MAILBOX_CANCELED="Orders.Canceled"  
````

### Назначения переменных в .env
``EMAIL_FROM`` - почта, которая будет указана как отправитель  

``IMAP_KORONA_AUTO_USERNAME`` - логин IMAP   
``IMAP_KORONA_AUTO_PASSWORD`` - пароль IMAP  

``IMAP_ATTACHMENTS_PATH`` - путь, куда будут сохраняться прикрепленные документы  
``MAILER_DSN`` - подключение почты  
``DATABASE_URL`` - подключение в бд  

### Настройки соединения с RabbitMQ  
``AMQP_RABBITMQ_HOST``  
``AMQP_RABBITMQ_PORT``  
``AMQP_RABBITMQ_USER``  
``AMQP_RABBITMQ_PASSWORD``  
``AMQP_RABBITMQ_VHOST``  

### Настройка соединения с Redis
``REDIS_HOST`` - хост Redis (в докере redis)
``REDIS_PORT`` - внутренний порт Redis

### Настройки docker  
``DB_NAME`` - имя бд  
``DB_USER`` - пользователь бд  
``DB_PASSWORD`` - пароль пользователя бд  
``DB_ROOT_PASSWORD``  
``COMPOSE_DB_PORT`` - внешний порт базы данных  
``COMPOSE_API_PORT`` - внешний порт nginx  
``COMPOSER_RABBITMQ_PORT`` - внешний порт RabbitMQ  
``COMPOSE_REDIS_PORT`` - внешний порт Redis

### Доступные консольные команды
``order:check:new`` - проверка новых заказов из ИМ  
``order:check:send`` - проверка нужно ли отправить заказ поставщику   
``mail:check:order`` - проверка заказов, которые должны согласовать поставщики  
``mail:check:price`` - проверка и парсинг новых прайсов с почты  
``mail:check:upd`` - проверка и парсинг новых UPD с почты  
``rabbitmq:consumer:balance-history`` - синхронизация истории баланса  

## Разворачивание проекта Docker
Для начала работы необходимо развернуть front версию. 
Файл конфига nginx в параметре root ссылается на директорию фронта, 
которая формируется в процессе сборки ``root /home/user/sites/site/dist``   

* cp .env .env.local, заполняем все пременные:  
["Настройки docker"](#docker-variables)  
["Настройки соединения с RabbitMQ"](#rebbitmq-variables)  
["Назначения переменных в .env"](#default-variables)  
["Настройка для работы с почтой"](#imap-variables)  
* cp config/packages/framework.yaml.example config/packages/framework.yaml  
* docker-compose --env-file up -d --build
* Заходим в контейнер php и выполняем composer install
* Cron(etc/cron.d) должен быть с правами 644 и для пользователя root 
* Настроиваем сервер, чтобы было проксировани  
* Настройка nginx для проксирования докера
````
server {

    listen      80;

    server_name domain.com

    root /home/user/sites/site/dist;
    
    location /front/v1 {
    	proxy_pass http://0.0.0.0:8017/front/v1;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-for $remote_addr;
        proxy_set_header Host $host;
        proxy_connect_timeout 300;
        proxy_send_timeout 300;
        proxy_read_timeout 300;
        proxy_redirect off;
        proxy_set_header Connection close;
        proxy_pass_header Content-Type;
        proxy_pass_header Content-Disposition;
        proxy_pass_header Content-Length;
    }

    location ~* \.(jpg|jpeg|gif|png|css|ico|bmp|swf|js)$ {
        root /home/rolf/sites/korona-auto.loc/dist;
	expires max;
    }
  
    location ~ /\.ht    {return 404;}
    location ~ /\.svn/  {return 404;}
    location ~ /\.git/  {return 404;}
    location ~ /\.hg/   {return 404;}
    location ~ /\.bzr/  {return 404;}

    try_files $uri /index.html;

    disable_symlinks if_not_owner from=/home/rolf/sites/korona-auto.loc/public;
}
````
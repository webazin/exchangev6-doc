# راهنمای نصب برنامه صرافی ارز دیجیتال ورژن 6

## مرحله آپلود

ابتدا محتویلت داخل پوشه public_html  را در پوشه public سرور خود آپلود کنید

سپس یک پوشه به عقب برگردید و یک پوشه با نام application ایجاد کنید.

محتویات فایل application را در این پوشه آپلود کنید.

یک دیتابیس در مدیریت سرور خود بسازید و اطلاعات را در فایل .env موجود در پوشه application وارد کنید

در فایل های دریافتی یک فایل با نام database.sql وجود دارد در دیتابیس ساخته شده در مرحله قبل آپلود کنید

## مرحله راه اندازی سرور

به ترتیب کدهای زیر را از طریق ssh اجرا کنید.

*** توجه دستورات زیر جهت اجرا در سرور اوبونتو می باشد در صورتی که از سیستم عامل دیگری استفاده میکنید دستورات را طبق سیستم عامل خود اجرا کنید.

```
sudo apt-get update
```

###  نصب ردیس 

```
sudo apt-get install redis
```

### نصب composer 

```
cd ~
curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php

HASH=`curl -sS https://composer.github.io/installer.sig`

echo $HASH

php -r "if (hash_file('SHA384', '/tmp/composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

sudo php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer

```

### نصب node js , npm

```
curl -sL https://deb.nodesource.com/setup_12.x -o nodesource_setup.sh
```

```
sudo bash nodesource_setup.sh & sudo apt-get install -y nodejs
sudo apt install nodejs
```

### نصب pm2 , laravel-echo-server

```
npm install -g pm2
npm install -g laravel-echo-server
```

### ویرایش اطلاعات لاراول اکو 

اطلاعات فایل زیر را متناسب با اطلاعات مورد نیاز خود بروز کنید 

```
nano /home/USER/domains/DOMAIN/application/laravel-echo-server.conf
```
### نصب supervisor 

```
sudo apt-get install supervisor
```

سپس به مسیر پوشه application رفته و دستورات زیر را اجرا کنید

```
composer install --ignore-platform-reqs
npm install
```

#### راه اندازی سرویس supervisor

```
nano /etc/supervisor/conf.d/currencies.conf
```
و سپس اطلاعات زیر را در فایل بالا وارد کنید

```
[group:currencies]
programs=updateprice,updateusdtprice

[program:updateprice]
command=/usr/local/php80/bin/php80 /home/USER/domains/DOMAIN/application/artisan currency:getPrice
startsecs=0
events=TICK_60
numprocs=1
autostart=true
autorestart=true
user=root

[program:updateusdtprice]
command=/usr/local/php80/bin/php80 /home/USER/domains/DOMAIN/application/artisan currency:getUsdtPrice
startsecs=0
events=TICK_60
numprocs=1
autostart=true
autorestart=true
user=root

```

```
nano /etc/supervisor/conf.d/laravel.conf
```

```
[group:laravel]
programs=queue,schedule

[program:queue]
command=/usr/local/php80/bin/php80 /home/USER/domains/DOMAIN/application/artisan queue:work
numprocs=1
autostart=true
autorestart=true
user=root

[program:schedule]
command=/usr/local/php80/bin/php80 /home/USER/domains/DOMAIN/application/artisan schedule:run
numprocs=1
autostart=true
autorestart=true
user=root
startsecs=0
```

```
nano /etc/supervisor/conf.d/laravelEcho.conf
```

```
[program:laravelEcho]
command=/usr/bin/laravel-echo-server start --dir=/home/USER/domains/DOMAIN/application
numprocs=1
autostart=true
autorestart=true
user=root
```

```
nano /etc/supervisor/conf.d/market.conf
```

```
[group:markets]
programs=orderboockask,orderbockbid,ticker,orderupdate,lastorder


[program:lastorder]
command=/usr/local/php80/bin/php80 /home/USER/domains/DOMAIN/application/artisan market:redis-last-orders-subscribe
numprocs=1
autostart=true
autorestart=true
user=root

[program:orderboockask]
command=/usr/local/php80/bin/php80 /home/USER/domains/DOMAIN/application/artisan market:redis-ask-subscribe
numprocs=1
autostart=true
autorestart=true
user=root

[program:orderbockbid]
command=/usr/local/php80/bin/php80 /home/USER/domains/DOMAIN/application/artisan market:redis-bid-subscribe
numprocs=1
autostart=true
autorestart=true
user=root


[program:ticker]
command=/usr/local/php80/bin/php80 /home/USER/domains/DOMAIN/application/artisan market:redis-last-ticker-subscribe
numprocs=1
autostart=true
autorestart=true
user=root

[program:orderupdate]
command=/usr/local/php80/bin/php80 /home/USER/domains/DOMAIN/application/artisan market:redis-order-update
events=TICK_30
numprocs=1
autostart=true
autorestart=true
user=root
```

* اطلاعات خودتون را وارد کنید . DOMIN و USER به جای


و در نهایت دستورات زیر را اجرا کنید


```
supervisorctl reread
supervisorctl update
```

* برای اطمینان از مراحل بالا آدرس سایت خود به همراه port 6003 را وارد کنید باید نوشته ok ظاهر شود


سپس به پوشه /applicatio/nodejs رفته و فایل config را بروزرسانی و اطلاعات خود را وارد کنید

 در مسیر بالا دو پوشه با نامه های بایننس و کوکوین وجود دارد به پوشه های موجوذ رفته و دستورات زیر اجرا کنید.
 
```
pm2 start *
```

سپس 

```
pm2 save
pm2 startup
```

* توجه پس از وارد کردن بازار جدید کد زیر را اجرا کنید

```
pm2 restart all
suprevisorctl restart all
```
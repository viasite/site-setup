# site-setup
Scripts for setup drupal site on virtual hosting.

## Requirements
- [popstas/server-scripts](https://github.com/popstas/server-scripts)

# Installation
```
cd /usr/local/src
git clone https://github.com/popstas/site-setup.git
cd site-setup
./install.sh
```

It install scripts to `/usr/share/site-setup`.
Config placed to `/etc/site-setup.conf`

# Использование

Скрипт по автоматизации установки сайтов.

- Создает юзера в системе, в его папке создает папки log и www
- Скачивает движок сайта, распаковывает его в папку сайта, ставит нужные права файлам сайта
- Пишет конфиги для nginx и apache, активирует их
- Создает пользователя MySQL и БД, дает юзеру права на базу
- Делегирует домен, если он 2 уровня
- Запускает скрипт установки сайта
- Отправляет email админу


## Примеры

### Установка стандартного drupal сайта
```
site-setup --verbose \
 --domain example.com \
 --domain_test test.example.com \
 --engine drupal \
 --user site \
 --db site \
 --script drupal
```

или
```
site-setup --config default_drupal --user site --domain site.ru
```

### Копирование существующего сайта
`db-pass` нужно указывать текущий от юзера, иначе скрипт изменит пароль юзера mysql при установке!
```
site-setup --verbose \
--domain new.example.com \
--engine /home/example/www/old.example.com \
--user example \
--db example_new \
--db_source example_old \
--db_pass ASDlhkfew9
```

### Настройка пустого сайта (без БД, только apache & nginx)
```
site-setup --domain example.com --db example
```

### Настройка домена
```
site-setup-domain --domain example.com
```


## Параметры скрипта:
Можно посмотреть через site-setup --help

`--config` - конфиг в виде bash-скрипта. Конфиг переопределяет все, что указано параметрами к команде. 
Параметры-флаги --param нужно указывать как param=True  
Можно указывать полный путь к конфигу или только название конфига из /usr/share/site-setup/configs
Появился в v1.2.0

`--domain` - скрипт автоматом добавляет в dns сервера домен, если его там еще нет, site-setup-domain, Делегирование домена на стороне сервера

`--domain_test` - не обязателен, это обычно поддомен test.example.com

`--db_pass` - если установка идет в существующего юзера, нужно указать `--db_pass password_ot_db`, обязательно!

`--verbose` - для подробного вывода, рекомендуется добавлять всегда

`--debug` - для очень подробного вывода, рекомендуется для доработки скрипта. Также, при установке этого флага, не будут приходить email об установке сайта.

`--engine` - можно указать:
- drupal (установка последней версии через drush)
- project:commerce_kickstart (дистриба с drupal.org)
- архив tgz, tar.gz на локальном или удаленном сервере
- путь к корню существующего локального сайта
- путь к локальному .yml файлу, тогда будет запущен drush make - Всё о drush, drush make
- ничего не указывать, тогда установится пустой сайт

`--script` - команда для конечной инициализации движка, когда хосты настроены, файлы и база созданы.
На 11.09.2014 такой скрипт только один: drupal-setup. Скрипт вызывается с такими параметрами:
`$SCRIPT $DOMAIN --sitedir $sitedir --user $USER --db_pass $DB_PASS --db $DB --logfile $logfile --drupal_profile $DRUPAL_PROFILE`

`--drupal_profile` - профиль установки drupal




## Баги и фичи:
1. Сайты, создаваемые в подпапке `test_directory`, блокируются на IP адрес офиса, в корне сайта создается файл .excluded
2. site-setup-domain удобно запускать отдельно: `site-setup-domain --domain example.com`



## Тестирование
<a href="http://ci.viasite.ru/viewType.html?buildTypeId=SiteSetup_Build">
<img src="http://ci.viasite.ru/app/rest/builds/buildType:(id:SiteSetup_Build)/statusIcon"/></a>

Бесопаснее и быстрее всего это делать в докере, в подготовленном контейнере:
```
cd site-setup
docker run --rm \
 -v $PWD:/usr/local/src/site-setup \
 -w /usr/local/src/site-setup \
 popstas/squeeze bash ./docker-tests.sh
```

Без docker можно запускать run-tests.sh, тогда действия должны выполняться под root и могут испортить вам рабочую систему.
`docker-tests.sh` можно запускать и вне docker, он устанавливает server-scripts и site-setup, предполагает, что исходники лежат в /usr/local/src/site-setup



## TODO:
- Конструкции типа is_verbose && echo 'verbose output' не работают при set -e, программа падает, если запущена без --verbose

# goaccess-nginx-systemd

### Установка
Установить пакет GoAccess можно прямо из репозитория, но скорее всего там будет далеко не самая свежая версия. Так что лучше добавить репозиторий GoAccess и установить из него.

###### Debian/Ubuntu
    $ echo "deb http://deb.goaccess.io/ $(lsb_release -cs) main" | sudo tee -a /etc/apt/sources.list.d/goaccess.list
    $ wget -O - https://deb.goaccess.io/gnugpg.key | sudo apt-key add -
    $ sudo apt-get update
    $ sudo apt-get install goaccess
Для других дистрибутивов смотрите документацию.
Настраиваем согласно примера в предоставленном конфиге.

### Настройка VCOMBINED логов в NGINX
В Nginx можно настроить сколько угодно кастомных форматов логов. Формат указывается после имени файла, например `access_log /var/log/nginx/access.log main;`.

Для настройки формата **VCOMBINED** необходимо добавить в `nginx.conf` следующую конфигурацию:

    http {
        ...
    
         log_format  vcombined '$host:$server_port '
                               '$remote_addr - $remote_user [$time_local] '
                               '"$request" $status $body_bytes_sent '
                               '"$http_referer" "$http_user_agent"';
    
         ...
    }
И включить логирование в формате **VCOMBINED** в настройках virtual host, или в основном конфиге для всех:

    server {
        listen 80;
        listen [::]:80;
    
        server_name *.domain.ua;
    
        access_log /var/log/nginx/sitename.access.log vcombined;
    }

Теперь нужно скопировать файл в папку `/lib/systemd/system` и запустить сервис командой **systemctl start goaccess**:

    cp goaccess.service /lib/systemd/system
    systemctl daemon-reload 
    systemctl start goaccess
Теперь можно открыть страницу domain.ua/report.html и увидеть real-time статистику сервера.

Для автоматического запуска GoAccess демона после перезагрузки сервера следует включить его в systemd командой `systemctl enable goaccess`.

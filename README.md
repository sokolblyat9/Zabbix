Zabbix через Docker контейнер

 docker network create --subnet 172.20.0.0/16 --ip-range 172.20.240.0/20 zabbix-net



docker run --name POSTGRES_DB \
    -e POSTGRES_PASSWORD="password" \
    -e POSTGRES_USER="IlyaS" \
    -e POSTGRES_DB="zabbix-postgres" \
    -e PGDATA="/var/lib/postgresql/18/docker" \
    -v /home/ilyas/DOKER/postgresql:/var/lib/postgresql/18 \
    --network=zabbix-net \
    -d postgres:18.1





docker run --name ZABBIX_SNMPTRAPS \
    -v /home/ilyas/DOKER/snmptraps:/var/lib/zabbix/snmptraps \
    -v /home/ilyas/DOKER/mibs:/var/lib/zabbix/mibs \
    -p 162:1162/udp \
    --network=zabbix-net \
    -d zabbix/zabbix-snmptraps:7.4-alpine-latest






docker run --name ZABBIX_SERVER_POSTGRES \
    -e DB_SERVER_HOST="POSTGRES_DB" \
    -e DB_SERVER_PORT="5432" \
    -e POSTGRES_USER="IlyaS" \
    -e POSTGRES_PASSWORD="password" \
    -e POSTGRES_DB="zabbix-postgres" \
    -e ZBX_LOADMODULE="" \
    -e ZBX_LISTENIP="0.0.0.0" \
    -e ZBX_LISTENPORT=10051 \
    -e ZBX_ENABLE_SNMP_TRAPS="true" \
    -v /home/ilyas/DOKER/alertscripts:/usr/lib/zabbix/alertscripts \
    -v /home/ilyas/DOKER/externalscripts:/usr/lib/zabbix/externalscripts \
    -v /home/ilyas/DOKER/modules:/var/lib/zabbix/modules \
    -v /home/ilyas/DOKER/enc:/var/lib/zabbix/enc \
    -v /home/ilyas/DOKER/ssh_keys:/var/lib/zabbix/ssh_keys \
    -v /home/ilyas/DOKER/certs:/var/lib/zabbix/ssl/certs \
    -v /home/ilyas/DOKER/keys:/var/lib/zabbix/ssl/keys \
    -v /home/ilyas/DOKER/ssl_ca:/var/lib/zabbix/ssl/ssl_ca \
    --volumes-from ZABBIX_SNMPTRAPS \
    -v /home/ilyas/DOKER/export:/var/lib/zabbix/export \
    -p 10051:10051 \
    --network=zabbix-net \
    --init -d zabbix/zabbix-server-pgsql:alpine-7.4-latest






docker run --name ZABBIX_WEB_SERVICE \
    -e DB_SERVER_HOST="POSTGRES_DB" \
    -e DB_SERVER_PORT="5432" \
    -e POSTGRES_USER="IlyaS" \
    -e POSTGRES_PASSWORD="password" \
    -e POSTGRES_DB="zabbix-postgres" \
    -e ZBX_ALLOWEDIP="0.0.0.0" \
    -e ZBX_LISTENPORT=10053 \
    -p 8092:8080 \
    -d zabbix/zabbix-web-nginx-pgsql:7.4-alpine-latest



Логин - Admin
Пароль - zabbix

При такой настройке, если войти в zabbix, то он будет ругаться, что не отвечает Zabbix агент, которого я и не собирал, а так же будет ругаться на неправильно написанный/указанный ip/port:

<img width="1154" height="122" alt="image" src="https://github.com/user-attachments/assets/0183106d-e721-4947-b9cf-fa37bc7bf418" />

<br><br/>
<br><br/>

<img width="552" height="288" alt="image" src="https://github.com/user-attachments/assets/e2abbd82-8024-4e96-ba2a-25eebd3a8dd5" />

<br><br/>
DeepSeek показал пример настройки ZABBIX_WEB_SERVICE:
<br><br/>
docker run --name ZABBIX_WEB_SERVICE \
  -e DB_SERVER_HOST="POSTGRES_DB" \
  -e POSTGRES_USER="IlyaS" \
  -e POSTGRES_PASSWORD="password" \
  -e POSTGRES_DB="zabbix-postgres" \
  -e ZBX_SERVER_HOST="ZABBIX_SERVER_POSTGRES" \
  -e ZBX_SERVER_PORT=10051 \
  -e PHP_TZ="Europe/Moscow" \
  -p 8092:8080 \
  --network=zabbix-net \
  -d zabbix/zabbix-web-nginx-pgsql:alpine-7.4-latest

<br><br/>
<br><br/>
После этого в Zabbix выглядит так:
<br><br/>
<img width="555" height="288" alt="image" src="https://github.com/user-attachments/assets/531c4e39-757e-4260-93a3-ff629f4c27e0" />
<br><br/>
Документация - https://www.zabbix.com/documentation/current/en/manual/installation/containers#environment-variables



<br><br/>
<br><br/>
<br><br/>
### Рабочий конфиг для докера:

<br><br/>

docker network create --subnet 172.20.0.0/16 --ip-range 172.20.240.0/20 zabbix-net



docker run --name POSTGRES_DB \
    -e POSTGRES_PASSWORD="password" \
    -e POSTGRES_USER="IlyaS" \
    -e POSTGRES_DB="zabbix-postgres" \
    -e PGDATA="/var/lib/postgresql/18/docker" \
    -v /home/ilyas/DOKER/postgresql:/var/lib/postgresql/18 \
    --network=zabbix-net \
    -d postgres:18.1





docker run --name ZABBIX_SNMPTRAPS \
    -v /home/ilyas/DOKER/snmptraps:/var/lib/zabbix/snmptraps \
    -v /home/ilyas/DOKER/mibs:/var/lib/zabbix/mibs \
    -p 162:1162/udp \
    --network=zabbix-net \
    -d zabbix/zabbix-snmptraps:7.4-alpine-latest






docker run --name ZABBIX_SERVER_POSTGRES \
    -e DB_SERVER_HOST="POSTGRES_DB" \
    -e DB_SERVER_PORT="5432" \
    -e POSTGRES_USER="IlyaS" \
    -e POSTGRES_PASSWORD="password" \
    -e POSTGRES_DB="zabbix-postgres" \
    -e ZBX_LOADMODULE="" \
    -e ZBX_LISTENIP="0.0.0.0" \
    -e ZBX_LISTENPORT=10051 \
    -e ZBX_ENABLE_SNMP_TRAPS="true" \
    -v /home/ilyas/DOKER/alertscripts:/usr/lib/zabbix/alertscripts \
    -v /home/ilyas/DOKER/externalscripts:/usr/lib/zabbix/externalscripts \
    -v /home/ilyas/DOKER/modules:/var/lib/zabbix/modules \
    -v /home/ilyas/DOKER/enc:/var/lib/zabbix/enc \
    -v /home/ilyas/DOKER/ssh_keys:/var/lib/zabbix/ssh_keys \
    -v /home/ilyas/DOKER/certs:/var/lib/zabbix/ssl/certs \
    -v /home/ilyas/DOKER/keys:/var/lib/zabbix/ssl/keys \
    -v /home/ilyas/DOKER/ssl_ca:/var/lib/zabbix/ssl/ssl_ca \
    --volumes-from ZABBIX_SNMPTRAPS \
    -v /home/ilyas/DOKER/export:/var/lib/zabbix/export \
    -p 10051:10051 \
    --network=zabbix-net \
    --init -d zabbix/zabbix-server-pgsql:alpine-7.4-latest

docker run --name ZABBIX_WEB_SERVICE \
  -e DB_SERVER_HOST="POSTGRES_DB" \
  -e POSTGRES_USER="IlyaS" \
  -e POSTGRES_PASSWORD="password" \
  -e POSTGRES_DB="zabbix-postgres" \
  -e ZBX_SERVER_HOST="ZABBIX_SERVER_POSTGRES" \
  -e ZBX_SERVER_PORT=10051 \
  -e PHP_TZ="Europe/Moscow" \
  -p 8092:8080 \
  --network=zabbix-net \
  -d zabbix/zabbix-web-nginx-pgsql:alpine-7.4-latest




<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>
<br><br/>

Zabbix на систему


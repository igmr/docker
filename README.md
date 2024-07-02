# Docker

- [Instalación de docker](#install-docker)
- [Configuración de docker](#settings-docker)
- [Instalación de docker-compose](#install-docker-compose)
- [Contenedores](#container)
    - [SpeedTest Tracker](#install-speed-test-tracker)
    - [Portainer](#install-portainer-ce)
    - [Pi-Hole](#install-pi-hole)
    - [Netdata](#install-netdata)
    - [IT-Tools](#install-it-tools)
    - [mStream](#install-mstream)
    - [Ampache](#install-ampache)
    - [Panel de control](#dashboard)
        - [Flame](#install-flame)
        - [Heimdall](#install-heimdall) 
    - [Utilerias](#utils)
        - [Stirling PDF](#stirling-pdf)
        - [Flatnotes](#flatnotes)
    - [Servidor de archivos](#cloud)
        - [NextCloud](#install-nextcloud)
        - [OwnCloud](#install-owncloud)
    - [Base de datos](#database)
        - [MySQL](#install-mysql)
        - [MariaDB](#install-mariadb)
        - [PostgreSQL](#install-postgresql)
        - [SQLite](#install-sqlite)

<a name="install-docker"></a>

## Instalación de docker

```shell
# =============================================================================
# Añadir claves GPG
# =============================================================================
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
# =============================================================================
# Añadir fuente de respositorios al APT
# =============================================================================
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update -y
# =============================================================================
# Instalación de docker
# =============================================================================
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
<a name="settings-docker"></a>

## Configuración de docker

```shell
# =============================================================================
# Correr contenedor de prueba
# =============================================================================
sudo docker run hello-world
# =============================================================================
# Crear grupo docker
# =============================================================================
sudo groupadd docker
# =============================================================================
# Agregar usuario actual al grupo docker
# =============================================================================
sudo usermod -aG docker $USER
# =============================================================================
# Activar cambios del grupo
# =============================================================================
newgrp docker
# =============================================================================
# Correr contenedor de prueba
# =============================================================================
docker run hello-world
```

<a name="install-docker-compose"></a>

## Instalación de docker-compose

```shell
# =============================================================================
# Instalación de docker-compose
# =============================================================================
sudo apt install -y docker-compose
```
<a name="container"></a>

## Contenedores

<a name="install-speed-test-tracker"></a>

### SpeedTest Tracker

```yaml
services:
    speedtest-tracker:
        container_name: speedtest-tracker
        ports:
            - 8080:80
            - 8443:443
        environment:
            - PUID=1000
            - PGID=1000
            - APP_KEY= # https://speedtest-tracker.dev/
            - DB_CONNECTION=sqlite
            - SPEEDTEST_SCHEDULE=
            - SPEEDTEST_SERVERS=
            - PRUNE_RESULTS_OLDER_THAN=
            - CHART_DATETIME_FORMAT= 
            - DATETIME_FORMAT=
            - APP_TIMEZONE=
        volumes:
            - /path/to/data:/config
            - /path/to-custom-ssl-keys:/config/keys
        image: lscr.io/linuxserver/speedtest-tracker:0.20.6
        restart: unless-stopped
```

<a name="install-portainer-ce"></a>

### Portainer CE

```yaml
version: '3'
services:
    portainer:
        image: portainer/portainer-ce:latest
        container_name: portainer
        ports:
            - 9443:9443
        volumes:
            - /home/ubuntu/docker/portainer/data:/data
            - /var/run/docker.sock:/var/run/docker.sock

```

<a name="install-pi-hole"></a>

### Pi-Hole

```yaml
version: '3'

# More info at https://github.com/pi-hole/docker-pi-hole/ and https://docs.pi-hole.net/
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    # For DHCP it is recommended to remove these ports and instead add: network_mode: "host"
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "67:67/udp" # Only required if you are using Pi-hole as your DHCP server
      - "80:80/tcp"
    environment:
      TZ: 'America/Chicago'
      WEBPASSWORD: 'your-password'
    # Volumes store your data between container upgrades
    volumes:
      - ./etc-pihole:/etc/pihole
      - ./etc-dnsmasq.d:/etc/dnsmasq.d
    #   https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
    cap_add:
      - NET_ADMIN # Required if you are using Pi-hole as your DHCP server, else not needed
```

#### Synology NAS

```yaml
services:
    pihole:
        image: pihole/pihole:latest
        container_name: pihole
        restart: unless-stopped
        network_mode: host
        environment:
            - PIHOLE_UID= #CHANGE_TO_YOUR_UID
            - PIHOLE_GID= #CHANGE_TO_YOUR_GID
            - TZ= #CHANGE_TO_YOUR_TZ
            - WEBPASSWORD= #YOUR_PASSWORD
            - DNSMASQ_LISTENING=local
            - WEB_PORT= #ADD_PORT_LISTENER
            - DNSMASQ_USER=pihole
            - FTLCONF_LOCAL_IPV4= #YOUR_IP_ADDRESS_V4
        volumes:
            - /volume1/docker/pihole/dnsmasq.d:/etc/dnsmasq.d
            - /volume1/docker/pihole/pihole:/etc/pihole
```

<a name="install-netdata"></a>

### Netdata

```yaml
version: '3'
services:
    netdata:
        image: netdata/netdata
        container_name: netdata
        hostname: localhost
        ports:
          - 19999:19999
        cap_add:
            - SYS_PTRACE
        security_opt:
            - apparmor:unconfined
        volumes:
            - ./netdata/netdataconfig/netdata:/etc/netdata
            - ./netdata/netdatalib:/var/lib/netdata
            - ./netdata/netdatacache:/var/cache/netdata
            - /etc/passwd:/host/etc/passwd:ro
            - /etc/group:/host/etc/group:ro
            - /etc/localtime:/etc/localtime:ro
            - /proc:/host/proc:ro
            - /sys:/host/sys:ro
            - /etc/os-release:/host/etc/os-release:ro
            - /var/log:/host/var/log:ro
            - /var/run/docker.sock:/var/run/docker.sock:ro
```

<a name="install-it-tools"></a>

#### IT-Tools

```yaml
version: '3.9'
services:
    it-tools:
        image: 'corentinth/it-tools:latest'
        container_name: it-tools
        restart: unless-stopped
        ports:
            - '8010:80'

```

<a name="install-mstream"></a>

## mStream

```yaml
version: '3'
services:
    mstream:
        image: lscr.io/linuxserver/mstream:latest
        container_name: mstream
        environment:
            - PUID=1000
            - PGID=1000
            - TZ=Etc/UTC
        volumes:
            - /path/to/data:/config
            - /path/to/music:/music
        ports:
            - 3000:3000
        restart: unless-stopped

```
<a name="install-ampache"></a>

### Ampache

```yaml
version: '3'

services:
    ampache:
        image: 'ampache/ampache:nosql'
        container_name: ampache
        ports:
            - 8013:80
        volumes:
            - ./data/music:/media
            - ./data/config:/var/www/config
            - ./data/log:/var/log/ampache
            - ./data/mysql:/var/lib/mysql

```


<a name="Dashboard"></a>

### Panel de control

<a name="install-flame"></a>

#### Flame

```yaml
version: '3.6'

services:
    flame:
        image: pawelmalak/flame
        container_name: flame
        volumes:
            - /path/to/host/data:/app/data
            - /var/run/docker.sock:/var/run/docker.sock # optional but required for Docker integration
        ports:
            - 5005:5005
        secrets:
            - password # optional but required for (1)
        environment:
            - PASSWORD=flame_password
            - PASSWORD_FILE=/run/secrets/password # optional but required for (1)
        restart: unless-stopped

# optional but required for Docker secrets (1)
secrets:
    password:
        file: /path/to/secrets/password
```

<a name="install-heimdall"></a>

#### Heimdall

```yaml
version: '3'
services:
    heimdall:
        image: linuxserver/heimdall
        container_name: heimdall
        volumes:
            - /home/user/appdata/heimdall:/config
        environment:
            - PUID=1000
            - PGID=1000
            - TZ=Europe/London
        ports:
            - 80:80
            - 443:443
        restart: unless-stopped
```

<a name="utils"></a>

### Utilerias

<a name="stirling-pdf"></a>

#### Stirling PDF

```yaml
version: '3'

services:
    stirling-pdf:
        image: frooodle/s-pdf:latest
        ports:
            - '8080:8080'
        volumes:
            - /location/of/trainingData:/usr/share/tesseract-ocr/5/tessdata #Required for extra OCR languages
            - /location/of/extraConfigs:/configs
#          - /location/of/customFiles:/customFiles/
#          - /location/of/logs:/logs/
        environment:
            - DOCKER_ENABLE_SECURITY=false
```

<a name="flatnotes"></a>

```yaml
version: "3"

services:
    flatnotes:
        image: elestio/flatnotes:latest
        restart: always
        ports:
            - "8080:8080"
        environment:
            FLATNOTES_AUTH_TYPE: "password"
            FLATNOTES_USERNAME: ${ADMIN_EMAIL}
            FLATNOTES_PASSWORD: ${ADMIN_PASSWORD}
            FLATNOTES_SECRET_KEY: ${ADMIN_PASSWORD}
        volumes:
            ./data:/data
```

```env
ADMIN_EMAIL=demo@demo.com
ADMIN_PASSWORD=password
ADMIN_PASSWORD=otherpassword
```

### NextCloud

<a name="cloud"></a>

### Servidor de archivos

<a name="install-nextcloud"></a>

### NextCloud

```yaml
version: '3'

services:
    nextcloud_db:
        image: mariadb:10.6
        container_name: nextcloud_db
        command: --transaction-isolation=READ-COMMITTED --log-bin=binlog --binlog-format=ROW
        volumes:
            - ./mariadb:/var/lib/mysql
        environment:
            - MYSQL_ROOT_PASSWORD=password
            - MYSQL_PASSWORD=password
            - MYSQL_DATABASE=nextcloud
            - MYSQL_USER=nextcloud
    nextcloud:
        image: nextcloud
        container_name: nextcloud
        ports:
            - 443:443
            - 80:80
        links:
            - nextcloud_db
        volumes:
            - ./nextcloud:/var/www/html
        environment:
            - NEXTCLOUD_ADMIN_USER=nextcloud
            - NEXTCLOUD_ADMIN_PASSWORD=password
            - MYSQL_HOST=dbnextcloud
            - MYSQL_PASSWORD=password
            - MYSQL_DATABASE=nextcloud
            - MYSQL_USER=nextcloud
```


<a name="install-owncloud"></a>

#### OwnCloud

```yaml
version: "3"

services:
  owncloud:
    image: owncloud/server:${OWNCLOUD_VERSION}
    container_name: owncloud_server
    restart: always
    ports:
      - ${HTTP_PORT}:8080
    depends_on:
      - mariadb
      - redis
    environment:
      - OWNCLOUD_DOMAIN=${OWNCLOUD_DOMAIN}
      - OWNCLOUD_TRUSTED_DOMAINS=${OWNCLOUD_TRUSTED_DOMAINS}
      - OWNCLOUD_DB_TYPE=mysql
      - OWNCLOUD_DB_NAME=owncloud
      - OWNCLOUD_DB_USERNAME=owncloud
      - OWNCLOUD_DB_PASSWORD=owncloud
      - OWNCLOUD_DB_HOST=mariadb
      - OWNCLOUD_ADMIN_USERNAME=${ADMIN_USERNAME}
      - OWNCLOUD_ADMIN_PASSWORD=${ADMIN_PASSWORD}
      - OWNCLOUD_MYSQL_UTF8MB4=true
      - OWNCLOUD_REDIS_ENABLED=true
      - OWNCLOUD_REDIS_HOST=redis
    healthcheck:
      test: ["CMD", "/usr/bin/healthcheck"]
      interval: 30s
      timeout: 10s
      retries: 5
    volumes:
      - ./data:/mnt/data

  mariadb:
    image: mariadb:10.11 # minimum required ownCloud version is 10.9
    container_name: owncloud_mariadb
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=owncloud
      - MYSQL_USER=owncloud
      - MYSQL_PASSWORD=owncloud
      - MYSQL_DATABASE=owncloud
      - MARIADB_AUTO_UPGRADE=1
    command: ["--max-allowed-packet=128M", "--innodb-log-file-size=64M"]
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-u", "root", "--password=owncloud"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - ./mysql:/var/lib/mysql

  redis:
    image: redis:6
    container_name: owncloud_redis
    restart: always
    command: ["--databases", "1"]
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - ./redis:/data
```

Variables de entorno

```env
OWNCLOUD_VERSION=10.14
OWNCLOUD_DOMAIN=localhost:8080
OWNCLOUD_TRUSTED_DOMAINS=localhost
ADMIN_USERNAME=admin
ADMIN_PASSWORD=admin
HTTP_PORT=8080
```


<a name="database"></a>

### Base de datos

<a name="install-mysql"></a>

#### MySQL

```yaml
version: '3'
services:
    mysql:
        image: mysql:8.3.0
        container_name: mysql
        command: --default-authentication-plugin=mysql_native_password
        environment:
            MYSQL_ROOT_PASSWORD: password
            MYSQL_DATABASE: datatable
            MYSQL_USER: user
            MYSQL_PASSWORD: password
        volumes:
            - ./mysql:/var/lib/mysql
        ports:
            - 3306:3306
        expose:
            - 3306
    adminer:
        image: adminer
        container_name: adminer-mysql
        ports:
        - 8080:8080
```

<a name="install-mariadb"></a>

#### MariaDB

```yaml
version: '3'

services:
    mariadb:
        image: mariadb:11.1
        container_name: mariadb
        environment:
            MARIADB_ROOT_PASSWORD: password
            MARIADB_DATABASE: db
            MARIADB_USER: user
            MARIADB_PASSWORD: password
        volumes:
            - ./mariadb:/var/lib/mysql
        ports:
            - 3306:3306
    adminer:
        image: adminer
        container_name: adminer-mariadb
        ports:
            - 8080:8080
```

<a name="install-postgresql"></a>

#### PostgreSQL

```yaml
version: '3'

services:
    postgresql:
        image: postgres:16.2
        container_name: postgresql
        shm_size: 128mb
        environment:
            POSTGRES_USER: ubuntu
            POSTGRES_DB: db
            POSTGRES_PASSWORD: martinez
            PGDATA: /var/lib/postgresql/data/pgdata
        volumes:
            - ./data:/var/lib/postgresql/data
    adminer:
        image: adminer
        container_name: adminer-postgresql
        ports:
            - 8080:8080
```

<a name="install-sqlite"></a>

#### SQLite

```yaml
version: '3'

services:
    sqlitebrowser:
        image: lscr.io/linuxserver/sqlitebrowser:latest
        container_name: sqlitebrowser
        security_opt:
            - seccomp:unconfined #optional
        environment:
            - PUID=1000
            - PGID=1000
            - TZ=Etc/UTC
        volumes:
            - /path/to/config:/config
        ports:
            - 3000:3000
            - 3001:3001
```


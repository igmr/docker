# Docker

- [Instalación de docker](#install-docker)
- [Configuración de docker](#settings-docker)
- [Instalación de docker-compose](#install-docker-compose)
- [Contenedores](#container)
    - [Heimdall](#install-heimdall) 
    - [Portainer](#install-portainer-ce)
    - [Pi-Hole](#install-pi-hole)
    - [NextCloud](#install-nextcloud)
    - [Netdata](#install-netdata)
    - [IT-Tools](#install-it-tools)
    - [Stirling PDF](#install-stirling-pdf)
    - [mStream](#install-mstream)
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
sudo apt update -y
sudo apt -y install ca-certificates curl
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
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
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

<a name="install-heimdall"></a>

### Heimdall

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

<a name="install-stirling-pdf"></a>

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

<a name="#install-mstream"></a>

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


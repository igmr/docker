# Docker

- [Instalación de docker](#install-docker)
- [Configuración de docker](#settings-docker)
- [Instalación de docker-compose](#install-docker-compose)
- [Contenedores](#container)
  - [Portainer](#install-portainer-ce)
  - [Pi-Hole](#install-pi-hole)

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

<a name="install-portainer-ce"></a>

### Portainer CE

```yaml
version: "3"
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
version: "3"

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

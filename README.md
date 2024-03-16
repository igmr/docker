  - [Index](#Index)
    - [Instalación de docker](#install-docker)
    - [Instalación de docker-compose](#install-docker-compose)


# Contenedores Docker

## Index

<a name="install-docker"></a>

### Instalación de docker

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
sudo apt-get update
# =============================================================================
# Instalación de docker
# =============================================================================
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
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
### Instalación de docker-compose

```shell
sudo apt install -y docker-compose
```

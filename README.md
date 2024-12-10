 Monitoreo de Servicios con Nagios y Docker

Este proyecto implementa un sistema de monitoreo para servidores críticos utilizando Nagios Core y Docker. Se monitorean un servidor de base de datos MySQL y un servidor web NGINX.

 Requisitos Previos

- Ubuntu Linux
- Nagios Core 4.4.6 instalado
- Docker
- Git

 Instalación

1. Instalación de Docker
```bash
# Actualizar repositorios
sudo apt update

# Instalar Docker
sudo apt install docker.io

# Iniciar y habilitar Docker
sudo systemctl start docker
sudo systemctl enable docker

# Agregar usuario al grupo docker
sudo usermod -aG docker $USER
```

 2. Configuración de Contenedores Docker

 Crear red de Docker
```bash
sudo docker network create nagios-network
```

 Iniciar MySQL
```bash
sudo docker run -d \
  --name mysql-server \
  --network nagios-network \
  -e MYSQL_ROOT_PASSWORD=tupassword \
  -e MYSQL_DATABASE=test \
  -p 3306:3306 \
  mysql:8.0
```

Iniciar NGINX
```bash
sudo docker run -d \
  --name nginx-server \
  --network nagios-network \
  -p 8080:80 \
  nginx:latest
```

 3. Configuración de Nagios

 Crear archivo de configuración para los hosts Docker
```bash
sudo nano /usr/local/nagios/etc/objects/docker-servers.cfg
```

Contenido del archivo:
```text
# MySQL Server
define host {
    use                     linux-server
    host_name               mysql-server
    alias                   MySQL Docker Server
    address                 127.0.0.1
    max_check_attempts      5
    check_period           24x7
    notification_interval   30
    notification_period    24x7
}

define service {
    use                     generic-service
    host_name               mysql-server
    service_description     MySQL Port
    check_command           check_tcp!3306
    notifications_enabled   1
}

# NGINX Server
define host {
    use                     linux-server
    host_name               nginx-server
    alias                   NGINX Docker Server
    address                 127.0.0.1
    max_check_attempts      5
    check_period           24x7
    notification_interval   30
    notification_period    24x7
}

define service {
    use                     generic-service
    host_name               nginx-server
    service_description     NGINX HTTP
    check_command           check_http!-p 8080
    notifications_enabled   1
}
```

Actualizar configuración principal de Nagios
Agregar al final del archivo `/usr/local/nagios/etc/nagios.cfg`:
```text
cfg_file=/usr/local/nagios/etc/objects/docker-servers.cfg
```

Verificar y reiniciar Nagios
```bash
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
sudo systemctl restart nagios
```

Uso de Docker Compose

El archivo `docker-compose.yml` incluido permite levantar todos los servicios con un solo comando:

```bash
docker-compose up -d
```

Verificación

1. Acceder a la interfaz web de Nagios:
   ```
   http://localhost/nagios/
   ```

2. Los servicios monitoreados deberían aparecer en la sección "Services"

3. Para simular una falla y probar el monitoreo:
   ```bash
   sudo docker stop mysql-server
   ```

Estructura del Repositorio

```
.
├── README.md
├── docker-compose.yml
├── configs/
│   ├── nagios.cfg
│   └── docker-servers.cfg
└── capturas/
    ├── hosts.png
    ├── services.png
    └── alert.png
```



Notas Adicionales

- El servidor NGINX está configurado en el puerto 8080 para evitar conflictos con Nagios
- La contraseña de root de MySQL está configurada como "12345"
- Todos los servicios están configurados para reiniciarse automáticamente

Autor

JUAN CASTAÑEDA

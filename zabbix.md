# 🚨 Implementación de Zabbix para sistema de alertas

## Objetivo

El objetivo de esta fase del proyecto es implementar un sistema de alertas independiente de Prometheus y Grafana utilizando Zabbix.

Mientras que Grafana y Prometheus se encargan de la monitorización y visualización de métricas, Zabbix se utilizará exclusivamente para la generación de alertas ante posibles incidencias en el servidor.

---

## Arquitectura

La solución se compone de dos máquinas virtuales:

### Servidor principal

```text
IP: 192.168.1.150
```

Contiene:

* Kubernetes (K3s)
* Servidor Minecraft
* Prometheus
* Grafana

### Servidor de monitorización

```text
IP: 192.168.1.129
```

Contiene:

* Docker
* PostgreSQL
* Zabbix Server
* Zabbix Web

---

## Instalación de Docker

Actualizamos Ubuntu:

```bash
sudo apt update
sudo apt upgrade -y
```

Instalamos Docker:

```bash
sudo apt install docker.io -y
```

Comprobamos la instalación:

```bash
docker --version
```

Habilitamos Docker al arrancar:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Verificamos el correcto funcionamiento:

```bash
sudo docker run hello-world
```

### Captura recomendada

📸 Docker funcionando correctamente.

---

## Instalación de Docker Compose

Instalamos Docker Compose:

```bash
sudo apt install docker-compose -y
```

Comprobamos la instalación:

```bash
docker-compose --version
```

### Captura recomendada

📸 Resultado de `docker-compose --version`

---

## Creación del entorno Zabbix

Creamos una carpeta para almacenar la configuración:

```bash
mkdir ~/zabbix
cd ~/zabbix
```

Creamos el archivo:

```bash
nano docker-compose.yml
```

Introducimos la siguiente configuración:

```yaml
version: '3.8'

services:

  postgres:
    image: postgres:15
    container_name: zabbix-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbixpass
      POSTGRES_DB: zabbix
    volumes:
      - postgres_data:/var/lib/postgresql/data

  zabbix-server:
    image: zabbix/zabbix-server-pgsql:latest
    container_name: zabbix-server
    restart: unless-stopped
    depends_on:
      - postgres
    environment:
      DB_SERVER_HOST: postgres
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbixpass
      POSTGRES_DB: zabbix
    ports:
      - "10051:10051"

  zabbix-web:
    image: zabbix/zabbix-web-nginx-pgsql:latest
    container_name: zabbix-web
    restart: unless-stopped
    depends_on:
      - postgres
      - zabbix-server
    environment:
      DB_SERVER_HOST: postgres
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbixpass
      POSTGRES_DB: zabbix
      ZBX_SERVER_HOST: zabbix-server
      PHP_TZ: Europe/Madrid
    ports:
      - "8080:8080"

volumes:
  postgres_data:
```

### Captura recomendada

📸 Archivo `docker-compose.yml`

---

## Despliegue de Zabbix

Iniciamos los contenedores:

```bash
sudo docker compose up -d
```

Comprobamos que los servicios están funcionando:

```bash
sudo docker ps
```

### Captura recomendada

📸 Resultado de `docker ps`

---

## Acceso a la interfaz web

Accedemos desde el navegador:

```text
http://192.168.1.129:8080
```

Credenciales por defecto:

```text
Usuario: Admin
Contraseña: zabbix
```

### Captura recomendada

📸 Pantalla principal de Zabbix.

---

# Configuración del agente Zabbix

## Instalación del agente

En el servidor principal:

```bash
sudo apt install zabbix-agent -y
```

---

## Configuración

Editamos:

```bash
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Configuramos:

```text
Server=192.168.1.129
ServerActive=192.168.1.129
Hostname=ServidorMinecraft
```

Reiniciamos el servicio:

```bash
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent
```

Comprobamos:

```bash
sudo systemctl status zabbix-agent
```

### Captura recomendada

📸 Servicio Zabbix Agent activo.

---

# Alta del host en Zabbix

Accedemos a:

```text
Data collection
→ Hosts
→ Create host
```

Configuramos:

```text
Host name: ServidorMinecraft
```

Grupo:

```text
Linux servers
```

Interfaz:

```text
Agent
IP: 192.168.1.150
Port: 10050
```

Template:

```text
Linux by Zabbix agent
```

### Captura recomendada

📸 Configuración completa del host.

---

# Verificación de conectividad

Accedemos a:

```text
Monitoring
→ Hosts
```

Comprobamos que el indicador:

```text
ZBX
```

aparece en color verde.

### Captura recomendada

📸 Host conectado correctamente.

---

# Configuración de alertas

Las alertas seleccionadas para el proyecto fueron:

* High CPU utilization
* High memory utilization
* High swap space usage

### Captura recomendada

📸 Triggers configurados.

---

# Configuración de correo electrónico

Accedemos a:

```text
Alerts
→ Media types
```

Configuramos Gmail como proveedor SMTP.

Posteriormente asociamos la cuenta de correo al usuario administrador.

### Captura recomendada

📸 Configuración del Media Type Gmail.

---

# Resultado final

Con esta configuración Zabbix es capaz de:

* Detectar problemas de CPU.
* Detectar problemas de memoria.
* Detectar uso excesivo de swap.
* Detectar indisponibilidad del servidor.
* Enviar alertas por correo electrónico.

De esta forma se dispone de un sistema de alertas independiente de Grafana y Prometheus, aumentando la disponibilidad y capacidad de reacción ante incidencias.


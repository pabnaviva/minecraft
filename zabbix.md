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

## Instalación de Docker Compose

Instalamos Docker Compose:

```bash
sudo apt install docker-compose -y
```

Comprobamos la instalación:

```bash
docker-compose --version
```



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

<img width="1919" height="994" alt="zabbix yml" src="https://github.com/user-attachments/assets/c5224563-d125-40e5-9ef9-4dc92594c6e2" />


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
<img width="1911" height="1023" alt="zabbix server ip" src="https://github.com/user-attachments/assets/49dec6e5-7dac-4f5a-81ff-f6b624171a42" />
<img width="1905" height="987" alt="zabbix hostname y ipactive" src="https://github.com/user-attachments/assets/abe0c3d1-c35a-4718-970c-3d5bb907779f" />


Reiniciamos el servicio:

```bash
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent
```

Comprobamos:

```bash
sudo systemctl status zabbix-agent
```

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

<img width="1857" height="950" alt="zabix emparejamiento" src="https://github.com/user-attachments/assets/166e162b-6824-47f2-b1c3-0f500326c187" />


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

<img width="1690" height="615" alt="zabbix enable" src="https://github.com/user-attachments/assets/cc3ecb46-e7f1-41ae-bc06-fa89a88d0214" />


# Configuración de alertas

Las alertas seleccionadas para el proyecto fueron:

* High CPU utilization
* High memory utilization
* High swap space usage

<img width="1682" height="805" alt="Problemas zabbix mostrar" src="https://github.com/user-attachments/assets/85459879-5a12-4c7c-bd9b-6dc63b4a481c" />


# Configuración de correo electrónico

Accedemos a:

```text
Alerts
→ Media types
```

Configuramos Gmail como proveedor SMTP.

Posteriormente asociamos la cuenta de correo al usuario administrador.

<img width="745" height="542" alt="zabbix correo smtp" src="https://github.com/user-attachments/assets/06a8880e-6cb6-4627-b706-dc230cc4bc08" />




# Resultado final

Con esta configuración Zabbix es capaz de:

* Detectar problemas de CPU.
* Detectar problemas de memoria.
* Detectar uso excesivo de swap.
* Detectar indisponibilidad del servidor.
* Enviar alertas por correo electrónico.

De esta forma se dispone de un sistema de alertas independiente de Grafana y Prometheus, aumentando la disponibilidad y capacidad de reacción ante incidencias.


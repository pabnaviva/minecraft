# 📊 Monitorización del servidor Minecraft con Prometheus y Grafana

Una vez desplegado el servidor Minecraft sobre Kubernetes K3s, el siguiente paso fue implementar una solución de monitorización que permitiera supervisar en tiempo real el estado de la infraestructura y el consumo de recursos.

Para ello se utilizaron **Prometheus** como sistema de recopilación de métricas y **Grafana** como herramienta de visualización.

---

# Arquitectura de monitorización

```text
K3s Cluster
│
├── Minecraft Server
├── Prometheus
├── Grafana
├── kube-state-metrics
└── node-exporter
```

📸 Captura 1

* Diagrama de arquitectura.

---

# Acceso a Grafana

Una vez desplegado el stack de monitorización se accedió a Grafana mediante el NodePort configurado previamente.

```text
http://192.168.1.150:30030
```

<img width="1917" height="961" alt="image" src="https://github.com/user-attachments/assets/0dad72ae-8d31-4cc1-b842-e8125de4d117" />


---

# Importación de dashboard Grafana

Para acelerar la creación del sistema de monitorización se utilizó un dashboard publicado por la comunidad de Grafana.

Dashboard utilizado:

```text
15757
```


Proceso de importación:

1. Acceder a Grafana.
2. Seleccionar **Dashboards**.
3. Pulsar **Import**.
4. Introducir el ID:

```text
15757
```

5. Seleccionar la fuente de datos Prometheus.
6. Importar el dashboard.

Este dashboard proporciona métricas relacionadas con Kubernetes y sirvió como base para la personalización posterior del sistema de monitorización.

<img width="1897" height="907" alt="image" src="https://github.com/user-attachments/assets/aa472d7d-8c2d-41cc-8b61-1b2603d7e9d3" />


# Personalización de paneles

Una vez importado el dashboard se modificaron varios paneles para mostrar únicamente la información relevante para el proyecto.

<img width="1917" height="912" alt="image" src="https://github.com/user-attachments/assets/3c2f0645-6288-405b-a54d-8fd0ba83a802" />

---

# 🔥 Monitorización de CPU

Para visualizar el consumo de CPU se utilizó la siguiente consulta PromQL:

```promql
rate(container_cpu_usage_seconds_total{pod="minecraft-549bf5df76-fxbq6"}[5m]) * 100
```

Esta consulta permite visualizar:

* Namespace default (Servidor Minecraft)
* Namespace monitoring (Prometheus y Grafana)

📸 Captura 6

<img width="1912" height="907" alt="image" src="https://github.com/user-attachments/assets/049a882b-883d-41c9-982e-7e08ffdfc38a" />


---

# 🧠 Monitorización de memoria RAM

Para visualizar el consumo de memoria RAM se utilizó la siguiente consulta:

```promql
sum(container_memory_working_set_bytes{pod="minecraft-549bf5df76-fxbq6"}) / 1024 / 1024 / 1024
```



Esta consulta muestra la memoria utilizada por los componentes desplegados.

📸 Captura 7

<img width="1912" height="917" alt="image" src="https://github.com/user-attachments/assets/989c73f1-58c7-4f49-9f10-87eeb053020a" />


# Personalización de nombres

Para mejorar la visualización se utilizaron las opciones **Overrides** de Grafana.

Los nombres originales:

```text
default
monitoring
```

fueron sustituidos por:

```text
Minecraft
Grafana + Prometheus
```

Esto permite identificar fácilmente qué métricas corresponden al servidor Minecraft y cuáles pertenecen a la infraestructura de monitorización.

<img width="1437" height="286" alt="image" src="https://github.com/user-attachments/assets/780b01f0-3d15-4bfc-9ad4-e178761d626b" />


# Problema detectado

Durante las pruebas se observó que los jugadores eran expulsados del servidor Minecraft con el mensaje:

```text
Tiempo de espera agotado
```

Tras analizar el sistema se comprobó que la máquina virtual se encontraba prácticamente sin memoria disponible.

Comprobación:

```bash
free -h
```

Resultado obtenido:

```text
Memoria total: 4.2 GB
Memoria utilizada: 4.2 GB
Memoria disponible: 12 MB
Swap: 0B
```

<img width="977" height="95" alt="image" src="https://github.com/user-attachments/assets/5e5c61a9-9bdd-4e8b-abe6-312ba68b97ca" />



---

# Solución aplicada: memoria SWAP

Para evitar bloqueos cuando la memoria RAM se agotase se creó un archivo SWAP de 4 GB.

Creación:

```bash
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

Verificación:

```bash
free -h
```

Esta configuración permite a Linux utilizar espacio en disco como memoria auxiliar cuando la RAM física se encuentra al límite.

<img width="1045" height="86" alt="image" src="https://github.com/user-attachments/assets/91448617-b1fb-409f-a50a-161215de6183" />


# Dashboard final

# Función de cada componente

## Prometheus

Prometheus es el encargado de recopilar y almacenar métricas del clúster Kubernetes.

Entre las métricas recopiladas se encuentran:

* CPU
* Memoria RAM
* Estado de los pods
* Estado de los servicios
* Recursos consumidos

---

## Grafana

Grafana es el encargado de visualizar las métricas obtenidas por Prometheus mediante dashboards interactivos.

Permite:

* Crear paneles personalizados.
* Visualizar gráficas en tiempo real.
* Analizar tendencias.
* Detectar problemas de rendimiento.

---

# Conclusiones

La integración de Prometheus y Grafana permitió incorporar una capa completa de observabilidad al proyecto.

Gracias a esta solución es posible supervisar en tiempo real el estado del servidor Minecraft desplegado sobre Kubernetes K3s y detectar posibles problemas de rendimiento antes de que afecten a los usuarios.

Además, la implementación de memoria SWAP permitió mejorar la estabilidad del sistema evitando problemas derivados de la falta de memoria RAM.


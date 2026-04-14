

**Ejemplo completo, mínimo y listo para copiar** para monitorizar un servidor de Minecraft en Kubernetes (con **k3s + Prometheus + Grafana + exporter**). Está pensado para portátil, así que es **ligero y funcional**.


# 🧱 0. Requisitos

- k3s instalado 

- Helm instalado ( + info: https://helm.sh/es/ )

- kubectl configurado 


# 🚀 1. Namespace

```
apiVersion: v1

kind: Namespace

metadata:

  name: minecraft
```

```
kubectl apply -f namespace.yaml
```


# 🎮 2. Servidor de Minecraft (con métricas)

Usamos la imagen popular:

- itzg/minecraft-server 

```
apiVersion: apps/v1

kind: Deployment

metadata:

  name: minecraft

  namespace: minecraft

spec:

  replicas: 1

  selector:

    matchLabels:

      app: minecraft

  template:

    metadata:

      labels:

        app: minecraft

    spec:

      containers:

        - name: minecraft

          image: itzg/minecraft-server

          ports:

            - containerPort: 25565

            - containerPort: 9100 \# metrics exporter

          env:

            - name: EULA

              value: "TRUE"

            - name: TYPE

              value: "PAPER"

            - name: MEMORY

              value: "2G"

            - name: ENABLE\_RCON

              value: "true"
```


# 🌐 3. Service

```
apiVersion: v1

kind: Service

metadata:

  name: minecraft

  namespace: minecraft

  labels:

    app: minecraft

spec:

  selector:

    app: minecraft

  ports:

    - name: mc

      port: 25565

      targetPort: 25565

    - name: metrics

      port: 9100

      targetPort: 9100
```


# 📊 4. Prometheus + Grafana (ligero)

Instala stack reducido:

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm install monitoring prometheus-community/kube-prometheus-stack \\

  --namespace monitoring \\

  --create-namespace \\

  --set grafana.adminPassword=admin \\

  --set prometheus.prometheusSpec.retention=1d
```


# 🔎 5. ServiceMonitor (clave)

Esto conecta Prometheus con Minecraft:

```
apiVersion: monitoring.coreos.com/v1

kind: ServiceMonitor

metadata:

  name: minecraft

  namespace: monitoring

spec:

  selector:

    matchLabels:

      app: minecraft

  namespaceSelector:

    matchNames:

      - minecraft

  endpoints:

    - port: metrics

      interval: 15s
```

```
kubectl apply -f servicemonitor.yaml
```


# 📈 6. Acceder a Grafana

```
kubectl port-forward svc/monitoring-grafana -n monitoring 3000:80
```

Abrir:  
👉 http://localhost:3000  
Usuario: `admin`  
Password: `admin`


# 📊 7. Queries útiles (Prometheus)

En Grafana:

```
\# TPS

minecraft\_tps

\# jugadores

minecraft\_players\_online

\# RAM del pod

container\_memory\_usage\_bytes\{pod=~"minecraft.\*"\}

\# CPU

rate(container\_cpu\_usage\_seconds\_total\{pod=~"minecraft.\*"\}\[5m\])
```


# 🚨 8. Alerta básica

```
apiVersion: monitoring.coreos.com/v1

kind: PrometheusRule

metadata:

  name: minecraft-alerts

  namespace: monitoring

spec:

  groups:

    - name: minecraft

      rules:

        - alert: LowTPS

          expr: minecraft\_tps \< 18

          for: 2m

          labels:

            severity: warning

          annotations:

            summary: "Minecraft TPS bajo"
```


# 🧪 9. Verificar que funciona

```
kubectl get pods -A

kubectl get servicemonitors -A
```

En Prometheus UI:

```
kubectl port-forward svc/monitoring-kube-prometheus-prometheus -n monitoring 9090
```

👉 http://localhost:9090

Busca:

```
minecraft\_tps
```


# ⚠️ Nota importante (muy importante)

La imagen de itzg/minecraft-server **NO expone métricas por defecto**.

👉 Necesitas uno de estos:

### Opción A (recomendada)

- Plugin Prometheus para Paper/Spigot 

### Opción B

- Exporter sidecar (más avanzado) 


# 🧠 10. Versión ULTRA ligera (para portátil)

Si ves que va justo:

- Reduce memoria Minecraft → `1G` 

- Quita Grafana temporalmente 

- Reduce retention Prometheus → `12h` 


# 🧩 Resultado final

Tendrás:

- Kubernetes gestionando Minecraft 

- Prometheus recogiendo métricas 

- Grafana mostrando: 

  - TPS 

  - jugadores 

  - CPU/RAM 

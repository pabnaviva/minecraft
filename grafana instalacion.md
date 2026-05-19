# Monitorización Kubernetes con Prometheus y Grafana

En esta parte del proyecto vamos a desplegar un stack completo de monitorización utilizando Prometheus y Grafana sobre Kubernetes K3s.

El objetivo será monitorizar:

* Recursos del nodo Ubuntu
* Pods Kubernetes
* Estado del cluster
* Uso de CPU y RAM
* Estado del servidor Minecraft

---

# Arquitectura de monitorización

```text
K3s Cluster
│
├── Minecraft Pod
├── Prometheus
├── Grafana
├── kube-state-metrics
└── node-exporter
```

---

# Crear namespace monitoring

Lo primero que vamos a hacer es crear un namespace específico para la monitorización.

```bash
sudo k3s kubectl create namespace monitoring
```

Comprobamos que se ha creado correctamente.

```bash
sudo k3s kubectl get namespaces
```

<img width="792" height="207" alt="image" src="https://github.com/user-attachments/assets/f9519882-1727-4196-a94a-38800fd05d4a" />


---

# Instalación de Helm

Helm es el gestor de paquetes de Kubernetes y nos permitirá desplegar Prometheus y Grafana fácilmente.

## Instalar Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

## Verificar instalación

```bash
helm version
```

<img width="1471" height="97" alt="image" src="https://github.com/user-attachments/assets/08df1137-6b61-4c0a-8b8a-5fae7b0e5f26" />


---

# Configurar acceso Kubernetes para Helm

K3s utiliza un archivo de configuración específico.

## Configurar variable KUBECONFIG

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

## Permitir lectura del archivo

```bash
sudo chmod 644 /etc/rancher/k3s/k3s.yaml
```

## Hacer la configuración permanente

```bash
echo 'export KUBECONFIG=/etc/rancher/k3s/k3s.yaml' >> ~/.bashrc
source ~/.bashrc
```

---

# Añadir repositorio Prometheus

## Añadir repositorio Helm

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

## Actualizar repositorios

```bash
helm repo update
```

<img width="1200" height="300" alt="image" src="AQUI_CAPTURA_HELM_REPO" />

---

# Instalar Prometheus y Grafana

Ahora instalaremos el stack completo de monitorización.

## Instalación kube-prometheus-stack

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
--namespace monitoring
```

Este stack instala automáticamente:

* Prometheus
* Grafana
* Alertmanager
* node-exporter
* kube-state-metrics

<img width="1400" height="400" alt="image" src="AQUI_CAPTURA_INSTALL" />

---

# Verificar pods de monitorización

## Ver pods

```bash
sudo k3s kubectl get pods -n monitoring
```

## Resultado esperado

```text
Running
```

Todos los pods deben terminar en estado Running.

<img width="1500" height="500" alt="image" src="AQUI_CAPTURA_PODS" />

---

# Exponer Grafana mediante NodePort

Por defecto Grafana se despliega como ClusterIP, por lo que no es accesible desde fuera del cluster.

## Modificar servicio Grafana

```bash
sudo k3s kubectl patch svc monitoring-grafana -n monitoring -p '{"spec": {"type": "NodePort", "ports": [{"port":80,"targetPort":"grafana","nodePort":30030}]}}'
```

## Verificar servicio

```bash
sudo k3s kubectl get svc -n monitoring
```

## Resultado esperado

```text
monitoring-grafana   NodePort   ...   80:30030/TCP
```

<img width="1400" height="350" alt="image" src="AQUI_CAPTURA_GRAFANA_NODEPORT" />

---

# Acceder a Grafana

Una vez expuesto el servicio podremos acceder desde el navegador.

## Dirección Grafana

```text
http://192.168.1.150:30030
```

---

# Obtener contraseña Grafana

## Usuario

```text
admin
```

## Obtener contraseña

```bash
sudo k3s kubectl get secret --namespace monitoring monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

<img width="1400" height="700" alt="image" src="AQUI_CAPTURA_LOGIN_GRAFANA" />

---

# Dashboards Grafana

Grafana incluye dashboards automáticos para:

* CPU
* RAM
* Pods Kubernetes
* Red
* Estado del nodo
* Recursos consumidos

<img width="1600" height="900" alt="image" src="AQUI_CAPTURA_DASHBOARD" />

---

# Servicios desplegados

## Ver servicios monitoring

```bash
sudo k3s kubectl get svc -n monitoring
```

## Ver deployments

```bash
sudo k3s kubectl get deployments -n monitoring
```

## Ver pods

```bash
sudo k3s kubectl get pods -n monitoring
```

---

# Monitorización del servidor Minecraft

Gracias a Prometheus y Grafana ahora es posible monitorizar:

* Consumo de RAM del pod Minecraft
* CPU utilizada
* Estado del deployment
* Reinicios del pod
* Uso de red

---

# Reinicio automático

K3s reinicia automáticamente todos los servicios tras reiniciar Ubuntu.

Esto incluye:

* Minecraft
* Prometheus
* Grafana

Por tanto únicamente será necesario encender la máquina virtual para recuperar toda la infraestructura.

---

# Problemas encontrados

Durante el despliegue fue necesario configurar correctamente:

* Variable KUBECONFIG
* Permisos del archivo k3s.yaml
* Exposición NodePort Grafana

Además se detectaron problemas de networking con Minikube, por lo que se migró toda la infraestructura a K3s.

---

# Comandos utilizados

## Ver nodos Kubernetes

```bash
sudo k3s kubectl get nodes
```

## Ver pods monitoring

```bash
sudo k3s kubectl get pods -n monitoring
```

## Ver servicios monitoring

```bash
sudo k3s kubectl get svc -n monitoring
```

## Ver logs Grafana

```bash
sudo k3s kubectl logs -n monitoring deployment/monitoring-grafana
```

## Ver logs Prometheus

```bash
sudo k3s kubectl logs -n monitoring deployment/monitoring-kube-prometheus-operator
```

---

# Conclusiones

La integración de Prometheus y Grafana permitió convertir el proyecto en una infraestructura Kubernetes monitorizada y preparada para entornos reales.

El proyecto permitió trabajar:

* Kubernetes K3s
* Helm
* Observabilidad
* Monitorización
* Grafana
* Prometheus
* NodePort
* Networking
* Administración Linux

---

# Estructura recomendada del repositorio

```text
monitoring-k3s/
│
├── README.md
├── screenshots/
│   ├── grafana/
│   ├── prometheus/
│   ├── kubernetes/
│   └── dashboards/
```


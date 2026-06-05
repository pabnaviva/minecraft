# 🌍 Publicación del servidor Minecraft en Internet

Una vez desplegado el servidor Minecraft sobre Kubernetes K3s, el siguiente paso fue permitir el acceso desde Internet para que cualquier jugador pudiera conectarse desde fuera de la red local.

---

# Obtención de IP pública

Para permitir el acceso externo se contrató una dirección IP pública proporcionada por el operador.

Esta IP pública permite que los dispositivos de Internet puedan localizar el router de la red doméstica.

📸 Captura 1

212.230.181.105

# Comprobación de acceso local

Antes de publicar el servidor en Internet se verificó que el servidor Minecraft funcionaba correctamente dentro de la red local.

Comprobación del servicio:

```bash
sudo k3s kubectl get svc
```

Resultado esperado:

```text
minecraft-service   NodePort   ...   25565:30007/TCP
```

Esto indica que Kubernetes ha asignado el puerto interno:

```text
30007
```

al servicio Minecraft.



# Configuración de redirección NAT

Para permitir el acceso desde Internet se configuró una regla NAT (Port Forwarding) en el router.

Configuración aplicada:

```text
Nombre: Minecraft
Protocolo: TCP
Puerto externo: 25565
IP destino: 192.168.1.150
Puerto interno: 30007
```

Funcionamiento:

```text
Internet
      │
      ▼
IP Pública
      │
Puerto 25565
      │
      ▼
Router
      │
      ▼
192.168.1.150:30007
      │
      ▼
Minecraft Server
```

<img width="942" height="362" alt="Mineecraft" src="https://github.com/user-attachments/assets/9b86858e-21fe-4d1c-ae93-345c93ba5266" />


# Verificación del servicio Minecraft

Una vez aplicada la redirección se comprobó que el servidor continuaba funcionando correctamente.

Verificación:

```bash
sudo k3s kubectl get pods
```

Resultado esperado:

```text
minecraft   Running
```



# Comprobación desde Internet

Con la redirección configurada correctamente, cualquier jugador puede conectarse utilizando la IP pública proporcionada por el operador.

Formato de conexión:

```text
IP_PUBLICA:25565
```

Ejemplo:

```text
212.230.181.105:25565
```




# Arquitectura final

```text
Internet
      │
      ▼
IP Pública (Yoigo)
      │
      ▼
Router
      │
Puerto 25565
      │
      ▼
192.168.1.150
      │
      ▼
Kubernetes K3s
      │
      ▼
Minecraft Server
```

📸 Captura 8

* Diagrama final de arquitectura.

---

# Conclusiones

La asignación de una dirección IP pública y la configuración de una regla NAT permitieron exponer el servidor Minecraft a Internet.

Gracias a esta configuración cualquier jugador puede conectarse al servidor utilizando la dirección IP pública proporcionada por el operador, mientras que Kubernetes continúa gestionando internamente el despliegue y la disponibilidad del servicio.

# 📊 Publicación de Grafana en Internet

Además del servidor Minecraft, se decidió publicar Grafana para poder supervisar remotamente el estado de la infraestructura desde cualquier ubicación.

---

# Comprobación del servicio Grafana

Verificación:

```bash
sudo k3s kubectl get svc -n monitoring
```

Resultado esperado:

```text
monitoring-grafana   NodePort   ...   80:30030/TCP
```

Esto indica que Grafana se encuentra accesible internamente mediante el puerto:

```text
30030
```



# Configuración de redirección NAT para Grafana

Se creó una segunda regla NAT en el router.

Configuración aplicada:

```text
Nombre: Grafana
Protocolo: TCP
Puerto externo: 30030
IP destino: 192.168.1.150
Puerto interno: 30030
```



<img width="958" height="353" alt="Grafana" src="https://github.com/user-attachments/assets/5d324fba-5f80-42ad-bf63-169c53a1ab3e" />


---

# Acceso remoto a Grafana

Una vez configurada la redirección, Grafana puede consultarse desde Internet utilizando:

```text
http://212.230.181.105:30030
```


<img width="1907" height="965" alt="image" src="https://github.com/user-attachments/assets/e99dad59-3a84-4e4b-9524-43b4f5e72c6a" />


# Visualización de métricas en tiempo real

A través de Grafana es posible supervisar:

* Uso de CPU.
* Consumo de memoria RAM.
* Estado de Kubernetes.
* Estado de Minecraft.
* Estado de Prometheus y Grafana.

# Arquitectura final del proyecto

```text
Internet
      │
      ▼
IP Pública (Yoigo)
      │
      ▼
Router
      │
      ├── Puerto 25565 → Minecraft
      │
      └── Puerto 30030 → Grafana
                     │
                     ▼
               Ubuntu Server
                     │
                     ▼
                   K3s
                     │
      ├── Minecraft
      ├── Prometheus
      └── Grafana
```





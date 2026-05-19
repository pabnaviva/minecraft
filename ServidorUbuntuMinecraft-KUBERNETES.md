# Servidor Minecraft desplegado en Kubernetes con K3s

Lo primero que vamos a hacer es poner nuestra máquina virtual en adaptador puente para que Ubuntu pueda ser accesible desde nuestra red local.

---

## Instalación K3s

Lo siguiente es asegurarse de que Ubuntu está actualizado e instalar K3s.

```bash
sudo apt update && sudo apt upgrade -y
curl -sfL https://get.k3s.io | sh -
```

Comprobamos que Kubernetes funciona correctamente.

```bash
sudo k3s kubectl get nodes
```

Resultado esperado:

```text
NAME        STATUS   ROLES                  AGE   VERSION
ubuntu      Ready    control-plane,master   ...   ...
```

<img width="805" height="126" alt="image" src="https://github.com/user-attachments/assets/5cccc925-e02f-4b19-9ec5-ee0ed91a4e1c" />


---

## Crear carpeta del proyecto

Creamos una carpeta y accedemos a ella para meter la configuración que vamos a usar.

```bash
mkdir ~/minecraft-server
cd ~/minecraft-server
```

---

## Crear archivo YAML

Creamos el archivo de configuración Kubernetes.

```bash
nano minecraft.yaml
```

Una vez dentro pegamos esta configuración:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: minecraft

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

          env:
            - name: EULA
              value: "TRUE"

            - name: ONLINE_MODE
              value: "FALSE"

            - name: MEMORY
              value: "4G"

---

apiVersion: v1
kind: Service

metadata:
  name: minecraft-service

spec:
  type: NodePort

  selector:
    app: minecraft

  ports:
    - protocol: TCP
      port: 25565
      targetPort: 25565
      nodePort: 30007
```
<img width="1917" height="892" alt="image" src="https://github.com/user-attachments/assets/635ade88-71a0-4dc9-b488-638ac6377ad1" />
<img width="1910" height="567" alt="image" src="https://github.com/user-attachments/assets/88701322-2da7-4423-b3bf-a10a95573aa9" />



---

## Explicación de la configuración

```text
NodePort 30007 → expone Minecraft al exterior

EULA=TRUE → obligatorio para que el servidor arranque

ONLINE_MODE=FALSE → permite entrar desde TLauncher

MEMORY=4G → memoria RAM asignada al servidor
```

---

## Aplicar configuración Kubernetes

Aplicamos el Deployment y el Service.

```bash
sudo k3s kubectl apply -f minecraft.yaml
```

---

## Comprobar que el pod funciona

```bash
sudo k3s kubectl get pods
```

Resultado esperado:

```text
NAME                         READY   STATUS    RESTARTS   AGE
minecraft-xxxxxxxxxx-xxxxx   1/1     Running   0          ...
```
<img width="1917" height="77" alt="image" src="https://github.com/user-attachments/assets/54d62af8-7158-4ce0-8e1d-1d6d68106df1" />



---

## Comprobar el Service NodePort

```bash
sudo k3s kubectl get svc
```

Resultado esperado:

```text
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)
minecraft-service   NodePort    xxx.xxx.xxx.xxx <none>        25565:30007/TCP
```

<img width="1917" height="120" alt="image" src="https://github.com/user-attachments/assets/8df90935-bec7-4ffd-b81c-ff019aeec729" />


---

## Ver logs del servidor Minecraft

```bash
sudo k3s kubectl logs -f deployment/minecraft
```

Esperamos hasta ver:

```text
Done (...)! For help, type "help"
```

---

## Obtener IP local Ubuntu

```bash
ip a
```

Buscamos la IP local de Ubuntu.

```text
192.168.1.150
```

<img width="1237" height="301" alt="image" src="https://github.com/user-attachments/assets/fe8a35ee-003a-4a57-be46-925ec7b1dcf3" />


---

## Conexión al servidor Minecraft

Lo que queremos hacer ahora para comprobar que funciona es conectarnos desde otro dispositivo de la misma red local.

La dirección utilizada será:

```text
192.168.1.150:30007
```

<img width="932" height="567" alt="image" src="https://github.com/user-attachments/assets/53d2b95b-8eb6-4de2-bb6c-5f4aa48a80f9" />


---

## Conexión exitosa

Una vez conectado correctamente vemos que el servidor funciona perfectamente.

<img width="926" height="567" alt="image" src="https://github.com/user-attachments/assets/4def4900-c020-4356-9646-b5735dd8c9a4" />


---

# Configuración del router

Para permitir conexiones desde Internet configuramos una regla NAT / Port Forwarding.

## Configuración utilizada

| Parámetro | Valor |
|---|---|
| Puerto externo | 25565 |
| IP interna | 192.168.1.150 |
| Puerto interno | 30007 |
| Protocolo | TCP |

Funcionamiento:

```text
Internet
↓
Router :25565
↓
Ubuntu :30007
↓
K3s NodePort
↓
Minecraft :25565
```

<img width="966" height="205" alt="Config Ruter" src="https://github.com/user-attachments/assets/c1769c7e-04d7-4b00-ae9d-af3aa957041b" />


---

# Problemas encontrados

Durante el desarrollo se detectaron problemas de networking utilizando Minikube con driver Docker sobre VirtualBox.

Esto provocaba:

- Problemas de exposición NodePort
- Errores de conexión TCP persistente
- Problemas de acceso desde Windows hacia Minecraft

La solución aplicada fue migrar desde Minikube hacia K3s.

K3s permitió:

- Networking más estable
- NodePort funcional
- Compatibilidad correcta con Minecraft
- Exposición correcta del servidor

---

# Comandos Kubernetes utilizados

## Ver nodos

```bash
sudo k3s kubectl get nodes
```

## Ver pods

```bash
sudo k3s kubectl get pods
```

## Ver servicios

```bash
sudo k3s kubectl get svc
```

## Ver logs

```bash
sudo k3s kubectl logs -f deployment/minecraft
```

## Reiniciar deployment

```bash
sudo k3s kubectl rollout restart deployment minecraft
```

## Eliminar deployment

```bash
sudo k3s kubectl delete deployment minecraft
```

## Eliminar service

```bash
sudo k3s kubectl delete service minecraft-service
```

---

# Conclusiones

K3s permitió desplegar correctamente un entorno Kubernetes funcional capaz de ejecutar un servidor Minecraft accesible desde red local y preparado para futura exposición pública mediante dominio e IP pública.

El proyecto permitió trabajar conceptos reales de:

- Kubernetes
- Networking
- Virtualización
- NodePort
- Despliegue de contenedores
- Exposición de servicios
- Administración Linux

---

## 🎥 Demo del servidor Minecraft

[![Ver video](https://img.youtube.com/vi/Fxr7oufrTLM/0.jpg)](https://youtu.be/Fxr7oufrTLM)

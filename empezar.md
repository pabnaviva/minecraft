Lo primero que vamos a hacer es poner nuestra maquina virtual en adaptador puente.

Lo segundo es asegurarse de que ubuntu este actializado y instalar docker
```bash
sudo apt update
sudo apt install docker.io -y
```
Lo siguiente que tendremos que hacer es arrancar nuestro docker y comprobar que funciona todo bien 
```bash
sudo systemctl enable docker
sudo systemctl start docker
docker run hello-world
```

<img width="773" height="327" alt="image" src="https://github.com/user-attachments/assets/701f3f48-5001-4a51-82b3-2a07c9dba6da" />

Creamos una carpeta y accedemos a ella para meterle la configuracion que vamos a usar 
```bash
mkdir ~/minecraft-server
cd ~/minecraft-server
```

Una vez en la carpeta le metemos esta configuracion.
<img width="1522" height="667" alt="image" src="https://github.com/user-attachments/assets/a8a70bc0-4b07-4b5c-8b1c-055aaa050553" />

-p 25565:25565 → abre el puerto del Minecraft

EULA=TRUE → obligatorio para que arranque

-v → guarda datos (mundos)

-d → corre en segundo plano


Lo que queremos hacer ahora para comprbar que funciona es irnos a otra pc local ya que aun no esta lanzada al exterior y comprobar que podemos entrar, aclarar que tienen que estar en la misma red por lo menos por ahora.
<img width="922" height="559" alt="Captura de pantalla 2026-04-13 154527" src="https://github.com/user-attachments/assets/ad44c177-aff1-47a8-af26-e9d1aeeb11eb" />

Una vez puesto esto vemos que nos da conexion con nuestro servidro de minecraft!
<img width="939" height="190" alt="Captura de pantalla 2026-04-13 154600" src="https://github.com/user-attachments/assets/8008adc8-9130-44ea-83a5-dacec3013bde" />



Nos encontramos con un error de verificacion de usuario, y es porque no tengo el juego comprado
<img width="932" height="576" alt="Captura de pantalla 2026-04-13 160235" src="https://github.com/user-attachments/assets/6059dfac-80c6-4a1b-9904-82e5bfd6f6a3" />


Vamos a editar el archivo de configuracion.
```bash
nano ~/minecraft-server/data/server.properties
```

Ponemos el online mode en false para que cualquiera pueda entrar
<img width="1572" height="792" alt="image" src="https://github.com/user-attachments/assets/df4e15ce-f02b-4bbb-888e-c97dc91c8973" />

Y muy importante reniciar el contenedor
```bash
docker restart mc-server
```

Y PORFIN ESTAMOS DENTRO!!!!!!!!!!!!!
## 🎥 Demo del servidor Minecraft

[![Ver video](https://img.youtube.com/vi/Fxr7oufrTLM/0.jpg)](https://youtu.be/Fxr7oufrTLM)







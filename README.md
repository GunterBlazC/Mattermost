# 🖥️ M0375B6A2 - Servidor de Mensajería Mattermost
### Serveis de Xarxa i Internet

## 📌 Introducción

En esta práctica he instalado y configurado un servidor de mensajería Mattermost en una máquina Ubuntu Server. El objetivo era montar un entorno cliente-servidor funcional y comprobar que podía acceder al servicio desde otra máquina dentro de la misma red.

Para ello he trabajado con una máquina virtual: un servidor y un cliente. Las configuré en modo puente para que pudieran comunicarse entre ellas.

Durante el proceso instalé la base de datos necesaria, configuré la aplicación para que se conectara correctamente y creé el servicio para que el servidor se iniciara automáticamente al arrancar el sistema. También tuve que solucionar algunos errores relacionados con permisos y configuración.

Finalmente, el servidor quedó funcionando correctamente y es accesible desde el navegador del cliente a través del puerto 8065.

---

## 🧩 Entorno del laboratorio

El laboratorio está compuesto por:

- 🖥️ Ubuntu Server → Servidor Mattermost + PostgreSQL
- 💻 Desktop → Cliente (acceso por navegador)

<img width="797" height="668" alt="image" src="https://github.com/user-attachments/assets/452a6679-82fd-40c8-9a45-c9eec91c15fb" />

---

## 🗄️ Instalación y configuración de PostgreSQL

Primero instalé PostgreSQL:

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
````
Comprobé que el servicio estuviera activo:

sudo systemctl status postgresql:

<img width="758" height="160" alt="image" src="https://github.com/user-attachments/assets/486c270a-b265-45ee-9749-e635f74754c8" />

Después accedí a la consola de PostgreSQL:

````
sudo -u postgres psql
````

Y creé:

````
CREATE DATABASE mattermost;
CREATE USER mmuser WITH PASSWORD '1234';
ALTER DATABASE mattermost OWNER TO mmuser;
GRANT ALL PRIVILEGES ON DATABASE mattermost TO mmuser;
````
Muestro la tabla de los usuarios ya creados:


<img width="714" height="322" alt="image" src="https://github.com/user-attachments/assets/5bbfff2b-6748-4ed0-a9b6-c4c796e618fb" />



## 📦 Descarga e instalación de Mattermost

Descargué la versión oficial:
````
sudo wget https://releases.mattermost.com/9.5.3/mattermost-team-9.5.3-linux-amd64.tar.gz
````

# 🔐 Configuración de permisos

Creé el usuario del sistema:
````
sudo useradd --system --user-group mattermost
````
Configuré permisos:
````
sudo mkdir -p /opt/mattermost/data
sudo chown -R mattermost:mattermost /opt/mattermost
sudo chmod -R 750 /opt/mattermost
````
Esto fue necesario para evitar errores de escritura en el almacenamiento local.

⚙️ Configuración de conexión con la base de datos

Edité el archivo:

/opt/mattermost/config/config.json

Modifiqué la línea:

"DataSource": "postgres://mmuser:1234@localhost:5432/mattermost?sslmode=disable"

Con esto conecté la aplicación con PostgreSQL.

📷 Captura: config.json modificado


🛠️ Creación del servicio systemd

Para que Mattermost se ejecute como servicio del sistema, creé:

/etc/systemd/system/mattermost.service

Contenido:

[Unit]
Description=Mattermost
After=network.target

[Service]
Type=simple
User=mattermost
Group=mattermost
WorkingDirectory=/opt/mattermost
ExecStart=/opt/mattermost/bin/mattermost
Restart=always
RestartSec=10
LimitNOFILE=49152

[Install]
WantedBy=multi-user.target

Después ejecuté:

sudo systemctl daemon-reload
sudo systemctl enable mattermost
sudo systemctl start mattermost

Comprobación:

sudo systemctl status mattermost

Resultado:

Active: active (running)

📷 Captura: servicio activo


🌐 Acceso desde el cliente

Desde Ubuntu Desktop accedí mediante navegador a:

http://192.168.34.169:8065

Se completó el asistente de configuración:

Creación del usuario administrador

Creación del equipo

Acceso al dashboard

📷 Captura: panel principal


⚠️ Problemas encontrados y soluciones

Durante la instalación surgieron varios problemas:

Error 203/EXEC en systemd

Secciones mal escritas en el archivo .service (case-sensitive)

Problemas de permisos en /opt/mattermost/data

Se solucionaron corrigiendo sintaxis, permisos y recargando systemd.

✅ Conclusión

Se ha desplegado correctamente un servidor Mattermost funcional con PostgreSQL en Ubuntu Server.

El servicio está integrado en el sistema mediante systemd y accesible desde red.

Estado final del servicio:

Active: active (running)

La comunicación cliente-servidor funciona correctamente en el puerto 8065.


---

# 📸 Ahora qué hacer

1. En tu repo crea carpeta:

capturas/


2. Sube tus imágenes con estos nombres:
- 01_ping.png
- 02_postgres.png
- 03_config.png
- 04_service.png
- 05_dashboard.png

3. Haz commit.

---

Si quieres, ahora te reviso tu README antes de subirlo para que quede de 10.

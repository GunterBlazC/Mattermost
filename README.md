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

Aquí se muestra el estado del servicio PostgreSQL tras la instalación. Se puede comprobar que el servicio aparece como `active (running)`, lo que confirma que la base de datos está correctamente instalada y funcionando.

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

En esta captura se observa la creación de la base de datos `mattermost` y del usuario `mmuser`. Esto permite que la aplicación tenga una base de datos dedicada y no utilice el usuario administrador por defecto.

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

## ⚙️ Configuración de conexión con la base de datos

Edité el archivo:
````
/opt/mattermost/config/config.json
````

Modifiqué la línea:

"DataSource": "postgres://mmuser:1234@localhost:5432/mattermost?sslmode=disable"

Con esto conecté la aplicación con PostgreSQL.

<img width="795" height="600" alt="image" src="https://github.com/user-attachments/assets/d8c48595-f471-48ae-8d54-eba36becdf1d" />

se muestra la modificación del archivo `config.json`, donde se establece la conexión entre Mattermost y la base de datos PostgreSQL mediante el parámetro `DataSource`.

## 🛠️ Creación del servicio systemd

Para que Mattermost se ejecute como servicio del sistema, creé:

````
/etc/systemd/system/mattermost.service
````
Contenido:

<img width="798" height="594" alt="image" src="https://github.com/user-attachments/assets/cd8f88d4-88ea-41d8-87e7-890f7ce491cd" />

Aquí se puede ver el archivo `mattermost.service` creado manualmente. Este archivo permite que la aplicación se ejecute como un servicio del sistema y pueda iniciarse automáticamente al arrancar el servidor.

Después ejecuté:

````
sudo systemctl daemon-reload
sudo systemctl enable mattermost
sudo systemctl start mattermost
````
Comprobación:

````
sudo systemctl status mattermost
````

Resultado:

<img width="800" height="404" alt="image" src="https://github.com/user-attachments/assets/ea79557a-2037-4b03-98d0-b8181f456cba" />



## 🌐 Acceso desde el cliente

Desde Ubuntu Desktop accedí mediante navegador a:
````
http://192.168.34.169:8065
````
Inicie con el usuario y contraseña correspondiente y me encontre con el dashbord que funciona correctamente 

Acceso al dashboard:

<img width="1474" height="801" alt="image" src="https://github.com/user-attachments/assets/b67ebd99-7966-49cf-863a-64764944f871" />

se observa el panel principal de Mattermost una vez iniciado sesión. Esto confirma que la instalación, configuración de la base de datos y puesta en marcha del servicio han sido correctas.

## ⚠️ Problemas encontrados y soluciones

Durante la instalación surgieron varios problemas:

Error 203/EXEC en systemd

Secciones mal escritas en el archivo .service (case-sensitive)

Problemas de permisos en /opt/mattermost/data

Se solucionaron corrigiendo sintaxis, permisos y recargando systemd.

## ✅ Conclusión

Se ha desplegado correctamente un servidor Mattermost funcional con PostgreSQL en Ubuntu Server.

El servicio está integrado en el sistema mediante systemd y accesible desde red.

La comunicación cliente-servidor funciona correctamente en el puerto 8065.

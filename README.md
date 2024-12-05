# Despliegue de WordPress en Alta Disponibilidad en AWS 
Este proyecto consiste en el despliegue de un sistema CMS WordPress utilizando AWS con una arquitectura de tres capas en alta disponibilidad y escalabilidad. El diseño incluye un balanceador de carga público, servidores backend privados con un sistema de archivos compartido (NFS) y una base de datos privada.

## Arquitectura
### Capa 1: Pública (Balanceador de Carga)
Un servidor web Apache que actúa como balanceador de carga.
Acceso permitido desde el exterior únicamente a través de HTTP (puerto 80) y HTTPS (puerto 443).
### Capa 2: Privada (Backend + NFS)
Dos servidores backend Apache que alojan la aplicación web.
Un servidor NFS que contiene los recursos compartidos del CMS.
### Capa 3: Privada (Base de Datos)
Un servidor MariaDB/MySQL que gestiona la base de datos de WordPress.
Acceso restringido únicamente desde los servidores backend.
# Desarrolllo
## 1. Crear la VPC y Subredes
### Paso 1: Crearemos una VPC con tres subredes

Una subred pública para el balanceador de carga (Capa 1), que será el único punto accesible desde internet.
Dos subredes privadas, una para los servidores Backend y NFS (Capa 2) y otra para la base de datos MySQL (Capa 3), garantizando que estos recursos estén protegidos y solo accesibles internamente.

![image](https://github.com/user-attachments/assets/046f851d-1da4-4c15-97de-bbb22cbad298)

![image](https://github.com/user-attachments/assets/a4398091-b22e-45a5-903b-1ffcbef76611)
Despues de terminar con la configuracion de VPC le damos ha crear VPC

### Paso 2: Comprobamos que se ha creado la vpc y las subredes de esta de forma correcta
![image](https://github.com/user-attachments/assets/520611e7-cceb-4779-8e80-38b77bd937e3)

## 2. Creacion de las instancias 
Se crearán instancias con una AMI de Ubuntu Server 24.04 para un despliegue en tres capas: un balanceador en la primera capa, servidores backend en la segunda y un servidor MySQL en la tercera.

### Paso 1: Configuración de las instancias
Asignar un nombre a la instancia y seleccionar la AMI Ubuntu Server 24.04 LTS. En entornos de prueba, se pueden usar ISOs o imágenes equivalentes.

![image](https://github.com/user-attachments/assets/973d4bea-a08b-47c9-8a41-674d7fd2d82d)

### Paso 2: Definir el tipo de Instancia y crear las Claves de SSH

![image](https://github.com/user-attachments/assets/58d0c5a1-45e6-49b1-994b-f59310e3820b)
![image](https://github.com/user-attachments/assets/4bbf5985-ddc1-4074-a211-5f5be6802148)

### Paso 3: Configuración de Red
Se asignará a la instancia creada una red VPC previamente configurada. Al ser un balanceador de carga, se asociará a una subred pública para permitir el acceso SSH desde el equipo local.

![Captura de pantalla 2024-12-03 215323](https://github.com/user-attachments/assets/a1fde9de-c9a0-449c-bcfe-910f40480e39)

### Paso 4: Creación de Grupo de Seguridad para la Instancia

![image](https://github.com/user-attachments/assets/7394e885-7a66-4fd4-88a4-0cd7b3b629d2)
![image](https://github.com/user-attachments/assets/4af3c169-aab2-4201-818b-49208d85c752)
![image](https://github.com/user-attachments/assets/d0d48d1d-d86a-4e7a-bc57-2b88224d4710)

* Las instancias creadas
![image](https://github.com/user-attachments/assets/4d5d0aae-4a88-473a-8ff2-d2c0688b4013)

## 3. Configuración Servidores Web y Balanceador
Se configurarán los servidores web y el balanceador de carga. En el balanceador, se instalará Apache2 y, mediante una directiva proxy en un archivo .conf, se gestionará la distribución de carga entre los dos servidores web.

### Paso 1: Configuración del Balanceador de Carga
* Para optimizar el funcionamiento del balanceador, se ejecutarán dos comandos para actualizar la lista de repositorios y llevarlos a su versión más reciente. 
```
sudo apt-get update
```
![image](https://github.com/user-attachments/assets/5b4efa0f-d0ec-4f9c-b670-244e5231c0fd)

```
sudo apt-get upgrade -y
```
![image](https://github.com/user-attachments/assets/4d6624db-2d58-4b3a-aee2-0f82c31c04fa)

*  Instalación y Configuración de APACHE2
Instalamos Apache2 en el balanceador de carga para redirigir las solicitudes entre los servidores web, distribuyendo así la carga de manera eficiente.

```
sudo apt-get install apache2 -y
```
![image](https://github.com/user-attachments/assets/4b9f2bd1-8b79-4327-840f-ec157496857a)

* Realizamos la comprobación

```
sudo systemctl start apache2
```
![image](https://github.com/user-attachments/assets/6efa6527-fc98-423d-bf9a-b41762148758)

```
sudo systemctl enable apache2
```
![image](https://github.com/user-attachments/assets/0bb49be9-7a2a-4f41-8270-285446b823d3)

```
sudo systemctl status apache2
```
![image](https://github.com/user-attachments/assets/4db86cba-9900-43e6-8335-b2d46b2c97c3)

* Comprobamos si funciona el Apache2
![image](https://github.com/user-attachments/assets/63d35076-98dc-48f9-b760-463f9e12c4e3)

* Configuramos el fichero de Balanceo

```
sudo cp 000-default.conf Balanceo.conf
```
```
sudo a2enmod proxy proxy_http
```
* Editamos el fichero de Balanceo
```
sudo nano Balanceo.conf
```

![image](https://github.com/user-attachments/assets/d1fd907b-4064-42fe-9e3c-68e7f0c2d615)

* Habilitamos el nuevo archivo .conf y reiniciamos apache y comprobamos que esta funcionando de manera correcta

```
sudo a2ensite Balanceo.conf
```
```
sudo systemctl restart apache2
```
```
sudo systemctl reload apache2
```
```
sudo systemctl status apache2
```
![image](https://github.com/user-attachments/assets/b9f6c024-4d60-45b6-bf62-708a6efc2390)

### Paso 2: Configuración de los servidores WEBs

- Los servidores con Apache y PHP despliegan WordPress utilizando recursos alojados en una carpeta compartida en el servidor NFS, recibiendo solicitudes gestionadas por el balanceador de carga.
* Aprovisionamiento para los dos servidores que nos permitira configurar todo lo necesario para el perfecto funcionamiento 

```
# Actualizar e instalar Apache y PHP con módulos necesarios

apt update -y
apt install -y apache2 nfs-common php libapache2-mod-php php-mysql php-curl>

# Habilitar el módulo rewrite

a2enmod rewrite

# Configurar el sitio web para que use la carpeta compartida de NFS

sed -i 's|DocumentRoot .*|DocumentRoot /nfs/shared/wordpress|' /etc/apache2>

# Configurar permisos del directorio

sed -i '/<\/VirtualHost>/i \
<Directory /nfs/shared/wordpress>\
        Options Indexes FollowSymLinks\
        AllowOverride All\
        Require all granted\
</Directory>' /etc/apache2/sites-available/000-default.conf

# Crear configuración personalizada

cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-availab>

# Montar la carpeta compartida desde el servidor NFS

mkdir -p /nfs/shared
mount 172.40.131.73:/var/nfs/shared /nfs/shared

# Configurar montaje automático en /etc/fstab

echo "172.40.131.73:/var/nfs/shared /nfs/shared nfs auto,nofail,noatime,nol>mount -a

# Desactivar el sitio por defecto y activar el nuevo sitio

a2dissite 000-default.conf
a2ensite websv.conf

# Reiniciar Apache
systemctl restart apache2
```
![image](https://github.com/user-attachments/assets/32b22374-ccd2-4ff7-aaf6-8ad291a7b7d6)

![image](https://github.com/user-attachments/assets/b326921d-5a95-41fc-846b-63512b23cb59)

## 4. Configuración de NFS
El servidor NFS centralizará los archivos del CMS WordPress, compartiéndolos con los servidores backend para garantizar coherencia, escalabilidad y un acceso eficiente y seguro a los recursos.

### Paso 1: Aprovisionamiento de NFS
En el aprovisionanmiento se abarcara toda la configuracion necesria para el buen funcionamiento 
```
#!/bin/bash
# Actualizar e instalar NFS y utilidades
apt update -y
apt install -y nfs-kernel-server unzip curl php php-mysql mysql-client

# Crear carpeta compartida y asignar permisos
mkdir -p /var/nfs/shared
chown -R nobody:nogroup /var/nfs/shared

# Configurar acceso desde los servidores con las nuevas IPs
sed -i '$a /var/nfs/shared      172.40.143.60(rw,sync,no_subtree_check)' /e>
sed -i '$a /var/nfs/shared      172.40.143.230(rw,sync,no_subtree_check)' />
# Descargar e instalar WordPress
curl -O https://wordpress.org/latest.zip
unzip -o latest.zip -d /var/nfs/shared/
chmod 755 -R /var/nfs/shared/
chown -R www-data:www-data /var/nfs/shared/*

# Reiniciar el servidor NFS
systemctl restart nfs-kernel-server
```
![image](https://github.com/user-attachments/assets/233b8e90-671c-4907-a8fd-a8fec0b8335a)

## 4. Configuración del servidor DDBB
En el servidor de base de datos configuraremos MariaDB para gestionar la información del CMS WordPress. Crearemos una base de datos específica, un usuario con permisos adecuados, y habilitaremos conexiones remotas desde los servidores backend, asegurando un entorno seguro y optimizado para el manejo de datos.

* Aprovicionamiento del servidor DDBB
- En el aprovisionamiento del servidor de base de datos, instalaremos y configuraremos MariaDB, crearemos una base de datos para WordPress, asignaremos un usuario con los permisos necesarios y habilitaremos conexiones remotas desde los servidores backend, garantizando la comunicación segura y eficiente en la arquitectura desplegada.
```
#!/bin/bash
# Actualizar e instalar MySQL y PhpMyAdmin
apt update -y
apt install -y mysql-server phpmyadmin

# Configurar MySQL para permitir conexiones remotas desde las nuevas IPs
sed -i "s/^bind-address.*/bind-address = 172.40.139.44/" /etc/mysql/mysql.c>
# Reiniciar MySQL
systemctl restart mysql

# Crear base de datos y usuario con acceso desde los servidores
mysql <<EOF
CREATE DATABASE db_wordpress;
CREATE USER 'severino'@'%' IDENTIFIED BY '5826';
GRANT ALL PRIVILEGES ON db_wordpress.* TO 'severino'@'%';
FLUSH PRIVILEGES;
EOF
```
![image](https://github.com/user-attachments/assets/fc0446a9-baf6-4bf8-ad45-486347cf2037)

## 4. Funcionamiento del CMS WordPress 
Ya terminando con toda la tarea, con nuestra dirección pública, tendremos que ingresarla en un navegador para poder instalar nuestro WordPress de la siguiente manera:

![image](https://github.com/user-attachments/assets/11f735cf-7b4a-47a3-81ae-ede141772f57)

### Paso 1: 

* En este apartado selecionaremos el idioma:
  
![Captura de pantalla 2024-12-05 135920](https://github.com/user-attachments/assets/b5bc1ffd-f5c7-44c3-aa4c-b81f62b81a4d)

![Captura de pantalla 2024-12-05 140020](https://github.com/user-attachments/assets/7c2b12fe-6a74-423b-a396-f0800873dfc3)

* Aquí introducimos los datos de nuestra base de datos creada para establecer la conexión:

![Captura de pantalla 2024-12-05 140321](https://github.com/user-attachments/assets/5f67b801-cb35-4183-be9c-d32d923cc2f9)
![Captura de pantalla 2024-12-05 140353](https://github.com/user-attachments/assets/d63bd472-4c67-4ed4-9e63-17e37badbafd)

* Introducimos la información necesaria y asi para terminar con nuestra instalacion

![Captura de pantalla 2024-12-05 140908](https://github.com/user-attachments/assets/6824b5ed-f384-4565-8441-4626fb78075c)
![Captura de pantalla 2024-12-05 140946](https://github.com/user-attachments/assets/dc258571-fb29-4df2-9036-2719fcdedcaa)
![Captura de pantalla 2024-12-05 141137](https://github.com/user-attachments/assets/bac93870-3899-427b-a20a-c2dc4c906cc2)

### Paso 2: Creación de dominio con nuestra IP publica

Desde la pagina de [noip](https://www.noip.com/es-MX) podemos crear un dominio nuestro
![Captura de pantalla 2024-12-05 141822](https://github.com/user-attachments/assets/a8c722a2-d5e4-4c17-916d-233ba2ef65c9)

### Paso 3: La instalacion de certbot

Por último instalaremos Certbot para obtener y configurar un certificado SSL/TLS gratuito de Let's Encrypt en nuestro servidor. Esto nos permitirá habilitar HTTPS en nuestro sitio web, asegurando las conexiones y protegiendo la privacidad de los usuarios.


![Captura de pantalla 2024-12-05 142622](https://github.com/user-attachments/assets/98cfb68c-863d-424f-ab30-c3d587c46f8d)





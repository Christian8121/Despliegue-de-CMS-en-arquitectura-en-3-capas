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

### Paso 2: Comprobamos que se ha creado la vpc y las subredes de esta de forma correcta
![image](https://github.com/user-attachments/assets/520611e7-cceb-4779-8e80-38b77bd937e3)

## 2. Creacion de las instancias 
Se crearán instancias con una AMI de Ubuntu Server 24.04 para un despliegue en tres capas: un balanceador en la primera capa, servidores backend en la segunda y un servidor MySQL en la tercera.

### Paso 1: Configuración de la instancia
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
![image](https://github.com/user-attachments/assets/6d2040e3-3aa8-40f6-9017-9b09cc3826ce)

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
* Aprovisionamiento de los dos servidores
Los servidores con Apache y PHP despliegan WordPress utilizando recursos alojados en una carpeta compartida en el servidor NFS, recibiendo solicitudes gestionadas por el balanceador de carga.

* actualizar la lista de repositorios y llevarlos a su versión más reciente.

```
sudo apt-get update
```
![image](https://github.com/user-attachments/assets/bea9e838-a9a6-4c77-a67c-50e3a2973a91)

```
sudo apt-get upgrade -y
```
![image](https://github.com/user-attachments/assets/c1886106-9949-41ba-b320-88db65976ca2)

* Instalación de Apache y PHP (+ Librerías)
```
sudo apt-get install apache2 -y
```
![image](https://github.com/user-attachments/assets/dbe70fe5-831b-4970-957a-26b2ef701aa2)

```
sudo apt install -y software-properties-common
```
![image](https://github.com/user-attachments/assets/935d8ef6-41c2-4df4-a397-57a8a6d503ba)

```
sudo add-apt-repository ppa:ondrej/php
```
![image](https://github.com/user-attachments/assets/601f5255-48ba-45fe-9a23-b21cbc7daff7)

```
sudo apt-get update
```
![image](https://github.com/user-attachments/assets/c64580dc-eba9-4e61-9ffd-0d70a1528862)

```
sudo apt install php7.4
```
![image](https://github.com/user-attachments/assets/bf77dbfe-8045-4faf-8b7c-ccc6502ecb9d)

```
sudo apt install php7.4-gd php7.4-intl
```
![image](https://github.com/user-attachments/assets/37b68bef-26f1-44ee-995a-86af32667914)

```
sudo apt install php7.4-cli php7.4-common php7.4-curl php7.4-zip php7.4-gd php7.4-mysql php7.4-xml php7.4-mbstring php7.4-json php7.4-intl libapache2-mod-php7.4
```
![image](https://github.com/user-attachments/assets/9cee2a35-0b1e-457e-8565-83a4b5288d38)

```
sudo update-alternatives --set php /usr/bin/php7.4
```
![image](https://github.com/user-attachments/assets/e8e35489-725b-4119-80d7-3b8849e81afc)

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




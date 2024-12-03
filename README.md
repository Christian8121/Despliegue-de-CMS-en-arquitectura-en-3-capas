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

### Paso 2: Definir el tipo de Instancia y crear las Claves de SSH:

![image](https://github.com/user-attachments/assets/58d0c5a1-45e6-49b1-994b-f59310e3820b)
![image](https://github.com/user-attachments/assets/4bbf5985-ddc1-4074-a211-5f5be6802148)







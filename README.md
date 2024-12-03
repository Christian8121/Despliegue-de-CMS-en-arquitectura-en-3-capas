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
Crearemos una VPC con tres subredes:

Una subred pública para el balanceador de carga (Capa 1), que será el único punto accesible desde internet.
Dos subredes privadas, una para los servidores Backend y NFS (Capa 2) y otra para la base de datos MySQL (Capa 3), garantizando que estos recursos estén protegidos y solo accesibles internamente.

![image](https://github.com/user-attachments/assets/046f851d-1da4-4c15-97de-bbb22cbad298)

![image](https://github.com/user-attachments/assets/a4398091-b22e-45a5-903b-1ffcbef76611)

## 2. Comprobamos que se ha creado la vpc y las subredes de esta de forma correcta
![image](https://github.com/user-attachments/assets/520611e7-cceb-4779-8e80-38b77bd937e3)

## Creacion de las instancias 




## Practica instalación Lamp:

![Captura desde 2024-02-14 13-00-26](https://github.com/Scosrom/Ansible-aws/assets/114906778/9087263d-4e83-420f-bfb0-0dab177f21b5)

En esta práctica, vamos a instalar una pila LAMP (Linux, Apache, MySQL/MariaDB, PHP) en dos instancias de AWS utilizando Ansible. Esto nos permitirá configurar rápidamente un entorno de desarrollo web en nuestras máquinas virtuales.

### Paso 1: Preparación del entorno

Antes de comenzar, asegúrate de tener acceso a las instancias de AWS y de tener la clave SSH necesaria para conectarte a ellas.

### Paso 2: Creación del inventario

Creamos un archivo llamado inventario donde especificamos las direcciones IP de nuestras instancias AWS y algunas variables comunes:

Archivo inventario: 

```
[aws]
34.229.151.76
3.87.125.51

[all:vars]
ansible_user=admin
ansible_ssh_private_key_file=/home/2asir/Escritorio/notepierdas.pem
ansible_ssh_common_args='-o StrictHostKeyChecking=accept-new'
```
Aquí hemos definido el grupo aws con las direcciones IP de nuestras instancias y hemos establecido algunas variables comunes como el usuario de SSH y la ruta de la clave privada

### Paso 3: Creación del playbook

Creamos un archivo llamado install_lamp.yaml que contendrá nuestro playbook de Ansible para instalar la pila LAMP:

Archivo install_lamp.yaml:

```
---
- name: Playbook para instalar la pila LAMP
  hosts: aws
  become: yes

  tasks:

    - name: Actualizar los repositorios
      apt:
        update_cache: yes

    - name: Instalar el servidor web Apache
      apt:
        name: apache2
        state: present

    - name: Instalar el sistema gestor de bases de datos MariaDB
      apt:
        name: mariadb-server
        state: present

    - name: Instalar PHP y los módulos necesarios
      apt:
        name:
          - php
          - php-mysql
          - libapache2-mod-php
        state: present

    - name: Reiniciar el servidor web Apache
      service:
        name: apache2
        state: restarted

    - name: Ejecutar el script de implantación
      script: implantacion_script.sh
```
En este playbook, definimos una serie de tareas para actualizar los repositorios, instalar Apache, MariaDB, PHP y sus módulos, reiniciar Apache y ejecutar un script de implantación.

### Paso 4: Creación del script de implantación

Creamos un script llamado implantacion_script.sh que contiene los comandos necesarios para configurar MariaDB, instalar phpMyAdmin, Git, clonar un proyecto PHP y copiar los archivos al directorio del servidor web:

implantacion_script.sh:

```
#!/bin/bash

# Install MariaDB 
export DEBIAN_FRONTEND=noninteractive
apt-get install -y mariadb-server

# Start MariaDB service
systemctl start mariadb

# Connect to MariaDB and execute SQL commands
mysql -u root <<myscript
CREATE DATABASE IF NOT EXISTS lamp_db CHARSET utf8mb4;
USE lamp_db;
CREATE TABLE IF NOT EXISTS users (
  id int(11) NOT NULL auto_increment,
  name varchar(100) NOT NULL,
  age int(3) NOT NULL,
  email varchar(100) NOT NULL,
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
CREATE USER IF NOT EXISTS 'lamp_user'@'%';
ALTER USER 'lamp_user'@'%' IDENTIFIED BY 'lamp_password';
GRANT ALL PRIVILEGES ON lamp_db.* TO 'lamp_user'@'%';
FLUSH PRIVILEGES;
myscript

# Install phpMyAdmin, Git
apt-get install -y phpmyadmin git

# Clone your PHP
git clone https://github.com/Scosrom/php-MariaDB-Apache2.git 

# Copy files to web server directory
cp -r php-MariaDB-Apache2/src/* /var/www/html

# Restart Apache
systemctl restart apache2

# Delete index
rm -r /var/www/html/index.html
```
Este script instala MariaDB, crea una base de datos y un usuario, instala phpMyAdmin, Git, clona un proyecto PHP y copia los archivos al directorio del servidor web.

* Este script esta sacado del repositorio: https://github.com/Scosrom/Implantacion_web/ y adaptado a esta practica.

### Paso 5: Ejecución del playbook

Finalmente, ejecutamos el playbook con el siguiente comando:

```
ansible-playbook -i inventario install_lamp.yaml
```
Esto ejecutará las tareas definidas en el playbook en todas las instancias especificadas en el inventario, configurando así nuestra pila LAMP automáticamente.


Información adicional: 

La variable de entorno DEBIAN_FRONTEND se utiliza en entornos donde se realiza la instalación de paquetes Debian de forma automatizada, como en scripts de instalación o en la ejecución de herramientas de gestión de paquetes como apt-get.

Cuando se establece DEBIAN_FRONTEND=noninteractive, se configura el frontend de Debian para que funcione en modo no interactivo. Esto significa que el sistema no espera ninguna entrada del usuario durante la instalación de los paquetes, sino que utiliza la configuración predeterminada o las opciones por defecto para todos los diálogos o mensajes de configuración que puedan aparecer durante el proceso de instalación.

En resumen, al establecer DEBIAN_FRONTEND=noninteractive, se asegura que las instalaciones de paquetes en sistemas Debian se realicen de forma automática, sin necesidad de intervención manual por parte del usuario, lo que es útil en escenarios de automatización o instalación masiva de software.

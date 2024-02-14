## Practica instalación Lamp:

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

Comando:

```
ansible-playbook -i inventario install_lamp.yaml
```
La variable de entorno DEBIAN_FRONTEND se utiliza en entornos donde se realiza la instalación de paquetes Debian de forma automatizada, como en scripts de instalación o en la ejecución de herramientas de gestión de paquetes como apt-get.

Cuando se establece DEBIAN_FRONTEND=noninteractive, se configura el frontend de Debian para que funcione en modo no interactivo. Esto significa que el sistema no espera ninguna entrada del usuario durante la instalación de los paquetes, sino que utiliza la configuración predeterminada o las opciones por defecto para todos los diálogos o mensajes de configuración que puedan aparecer durante el proceso de instalación.

En resumen, al establecer DEBIAN_FRONTEND=noninteractive, se asegura que las instalaciones de paquetes en sistemas Debian se realicen de forma automática, sin necesidad de intervención manual por parte del usuario, lo que es útil en escenarios de automatización o instalación masiva de software.

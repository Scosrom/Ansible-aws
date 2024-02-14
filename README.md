# Ansible-aws

### 1. Conexión SSH con instancias EC2 de AWS:

Para conectarte a las instancias EC2 de AWS desde tu nodo principal, necesitas una clave SSH privada proporcionada por AWS, generalmente llamada vockey.pem. Antes de usarla, asegúrate de darle permisos de solo lectura al propietario con el comando:

```
chmod 400 vockey.pem
```

Luego, puedes conectarte a una instancia con el comando SSH, usando el usuario adecuado (en nuestro caso, ubuntu) y la dirección IP pública de la instancia:

```
ssh -i vockey.pem ubuntu@18.206.58.248
```

### 2. Cómo indicar el usuario y la clave privada SSH en Ansible:

Puedes especificar el usuario y la clave privada SSH de varias maneras en Ansible:

2.1. Desde la línea de comandos usando los parámetros **--user** y **--private-key**.
2.2. En el archivo de inventario usando las variables **ansible_user** y **ansible_ssh_private_key_file**.


Ejemplo desde la linea de coomandos:

```
ansible-playbook -i inventario playbook.yaml --user admin --private-key /path/to/private_key.pem
```

Ejemplo con archivo inventario: 

```
[aws]
18.206.58.248
52.5.11.241

[aws:vars]
ansible_user=admin
ansible_ssh_private_key_file=/home/2asir/Escritorio/miclave.pem
```
### 3. Gestión de la validación del fingerprint de la clave pública SSH:

Al conectar por SSH por primera vez con una instancia tendremos que aceptar el fingerprint de la
clave SSH pública de la instancia remota.
Si en el archivo de inventario tiene varias instancias, al ejecutar nuestro playbook tendremos un
error porque no podemos aceptar el fingerprint de todas las instancias.
En el siguiente ejemplo se muestra lo que ocurre cuando ejecutamos un playbook sobre un archivo
de inventario que tiene dos instancias. En este ejemplo, solo podemos aceptar el fingerprint de la
última instancia del inventario.

```

TASK [Gathering Facts]
******************************************************************************************************************
The authenticity of host '34.226.122.155 (34.226.122.155)' can't be established.
ECDSA key fingerprint is SHA256:AjA3b6U0HwxjtJ3PhhelcplOf0u5xoY8fiEuSATMAJg.

The authenticity of host '44.206.245.114 (44.206.245.114)' can't be established.
ECDSA key fingerprint is SHA256:FJS7iV7GpDIuZPykQ9VQfGRj8l0dfLU1FiZVYJ+gIjI.
Are you sure you want to continue connecting (yes/no/[fingerprint])?

```

Existen varias soluciones para solucionar este problema, veamos algunas.

#### Solucion 1: Configurar la variable de entorno ANSIBLE_HOST_KEY_CHECKING

La primera solución es configurar la variable de entorno ANSIBLE_HOST_KEY_CHECKING a False para
saltarnos el paso de aceptar el fingerprint de las instancias remotas.

```
export ANSIBLE_HOST_KEY_CHECKING=False
```

Y ya podremos ejecutar:

```
ansible-playbook -i inventario install_lamp.yaml --user ubuntu --private-key /home/josejuan/Lab/vockey.pem
```

### Solucion 2: Configurar SSH para que acepte por defecto el fingerprint de las nuevas instancias. 

Podemos utilizar un parámetro en la conexión SSH para aceptar por defecto el fingerprint de las
nuevas instancias a las que vamos a conectarnos.

EJEMPLO:

```
ansible-playbook -i inventario install_lamp.yaml --user ubuntu --private-key /home/josejuan/Lab/vockey.pem --ssh-common
-args '-o StrictHostKeyChecking=accept-new'
```

### Solucion 3: Desde el archivo de inventario

En el archivo de inventario podemos utilizar el parámetro **ansible_ssh_common_args** para configurar
SSH y aceptar por defecto el fingerprint de las nuevas instancias a las que vamos a conectarnos.

```
[aws]
34.226.122.155
44.206.245.114

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/josejuan/Lab/vockey.pem
ansible_ssh_common_args='-o StrictHostKeyChecking=accept-new'
```

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

Archivo yaml:

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

Script:

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
** La variable de entorno DEBIAN_FRONTEND se utiliza en entornos donde se realiza la instalación de paquetes Debian de forma automatizada, como en scripts de instalación o en la ejecución de herramientas de gestión de paquetes como apt-get.

Cuando se establece DEBIAN_FRONTEND=noninteractive, se configura el frontend de Debian para que funcione en modo no interactivo. Esto significa que el sistema no espera ninguna entrada del usuario durante la instalación de los paquetes, sino que utiliza la configuración predeterminada o las opciones por defecto para todos los diálogos o mensajes de configuración que puedan aparecer durante el proceso de instalación.

En resumen, al establecer DEBIAN_FRONTEND=noninteractive, se asegura que las instalaciones de paquetes en sistemas Debian se realicen de forma automática, sin necesidad de intervención manual por parte del usuario, lo que es útil en escenarios de automatización o instalación masiva de software.**





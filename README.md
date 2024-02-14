# Ansible-aws

![image](https://github.com/Scosrom/Ansible-aws/assets/114906778/506333d7-276f-4aad-92c4-02bc95fa261a)


### 1. Conexión SSH con instancias EC2 de AWS:

Para conectarte a las instancias EC2 de AWS desde tu nodo principal, necesitas una clave SSH privada proporcionada por AWS, generalmente llamada vockey.pem. Antes de usarla, asegúrate de darle permisos de solo lectura al propietario con el comando:

```
chmod 400 vockey.pem
```

Luego, puedes conectarte a una instancia con el comando SSH, usando el usuario adecuado y la dirección IP pública de la instancia:

```
ssh -i vockey.pem ubuntu@18.206.58.248
```

### 2. Cómo indicar el usuario y la clave privada SSH en Ansible:

Puedes especificar el usuario y la clave privada SSH de varias maneras en Ansible:

- 2.1. Desde la línea de comandos usando los parámetros **--user** y **--private-key**.

- 2.2. En el archivo de inventario usando las variables **ansible_user** y **ansible_ssh_private_key_file**.


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
ansible-playbook -i inventario install_lamp.yaml --user ubuntu --private-key /home/2asir/Escritorio/miclave.pem
```

### Solucion 2: Configurar SSH para que acepte por defecto el fingerprint de las nuevas instancias. 

Podemos utilizar un parámetro en la conexión SSH para aceptar por defecto el fingerprint de las
nuevas instancias a las que vamos a conectarnos.

EJEMPLO:

```
ansible-playbook -i inventario install_lamp.yaml --user ubuntu --private-key /home/2asir/Escritorio/miclave.pem --ssh-common
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
ansible_user=admin
ansible_ssh_private_key_file=/home/2asir/Escritorio/miclave.pem
ansible_ssh_common_args='-o StrictHostKeyChecking=accept-new'
```

Comando:

```
ansible all -i inventario -m ping
```

## Practicas

1. [Practica 1- Pila lamp](practica1.md)




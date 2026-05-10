---
title: Secure Shell (SSH).
author: 
    - porucz 
date:
    created: 2024-07-26
tags:
    - ssh 
    - keys
    - linux
---

## Que es SSH
Hola en este post les enseñare a como utilizar el protocolo de de Secure Shell (SSH). Para ello necesitaremos saber que es SSH.

SSH es un protocolo que envía comandos de manera segura a un servidor o computadora, a través de la red. SSH cifra y autentica la comunicación punto a punto, también permite la tunelizacion o re direccionamiento de paquetes que de otro modo no podrían cruzar. El protocolo SSH es amplia mente utilizado en la gestión remota de infraestructura y servidores. 

## Como usar SSH 

Para poder utilizar ssh es necesario tenerlo instalado dentro de nuestro sistema operativo, para ello utilizaremos OpenSSH debido a que la implementación de Secure Shell es de código privativo y OpenSSH es una alternativa libre y de código abierto:

### Instalacion

- **Windows**

Verificamos que no exista una instalación de powershell ya en nuestro sistema operativo:

```shell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
```

La salida que arroja powershell nos indica el estatus del cliente y el servidor:


```ruby
Name  : OpenSSH.Client~~~~0.0.1.0
State : NotPresent

Name  : OpenSSH.Server~~~~0.0.1.0
State : NotPresent
```

Para poder realizar la instalación utilizaremos los siguientes comandos en base a nuestras necesidades, es decir si solo necesitamos conectarnos a otro servidor podemos usar únicamente el cliente, pero si requerimos conectarnos a el equipo utilizaremos también el servidor: 

```shell 
# Instalar el cliente SSH
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

# Instalar el servidor SSH
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```
Iniciaremos los servicios para OpenSSH:

```shell
Start-Service sshd
```
Si queremos que el cliente ssh se inicie al encender el equipo utilizaremos el siguiente comando:


```shell 
Set-Service -Name sshd -StartupType 'Automatic'
```
**Linux**
La mayoría de distribuciones GNU/Linux ya tienen instalado por defecto el cliente de OpenSSH, por lo que en caso de que lo requieras solo seria necesario instalar el servidor OpenSSH.

**Debian/Ubuntu**

```shell
apt install openssh-server
```

**Fedora**

```shell
dnf install openssh-server
```

**Alpine Linux**

```shell
apk add openssh
```

**Arch Linux**


```bash
pacman -S openssh 
```

**OpenSUSE**
```bash
zypper install --no-confirm openssh
```

**Levantar el servicio para el servidor OpenSSH para GNU/Linux**

**Systemd**

```bash
#Iniciar el servicio
systemctl start sshd.service

#Habilitar al inicio 
systemctl enable sshd.service

```
**OpenRC**

```bash
#Iniciar el servicio
rc-service sshd start

#Habilitar al inicio 
rc-update add sshd
```
**Runit**

```bash
#Iniciar el servicio 
sv up sshd.service

#Habilitar al inicio
ln -s /etc/sv/<service> /var/service/
```
**S6**

```bash
#Iniciar el servicio
s6-rc -u change sshd

#Habilitar al inicio
s6-rc-db default + sshd
s6-rc-update
```
### Uso

Para poder hacer una conexión SSH sera necesario contar con un servidor (IP o Hostname), la sintaxis es la siguiente:

```bash
#ssh la llamada al programa
#user el usuario dentro del servidor
#@ indica el redireccionamiento
#IP o Hostname es la direccion del servidor
ssh <user>@<IP or Hostname>
```
Ejemplo:

```bash 
ssh operator@10.10.1.3 
```
## Como generar llaves rsa y ed25519 para ssh

La seguridad es muy importante al momento de utilizar el protocolo SSH por lo que es una excelente practica utilizar llaves generadas por pares con algoritmos como RSA o ed25519 ambos bastante robustos.

### RSA 

Para crear una llave RSA utilizaremos el siguiente comando:

```bash
 ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
Donde:

| Argumento | Descripcion |
|--------------- | --------------- |
| -t | Algoritmo utilizado para generar la llave |
| -b | Tamaño de bits a utilizar |
| -c | Correo electronico asociado para la llave |

### ed25519

Para generar una clave ed25519 utilizaremos el siguiente comando:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```
### Copiar en el servidor

Una vez generada nuestra clave nos genera dos archivos, una llave privada y la llave publica dentro de nuestra carpeta `.ssh`. 

- Para el caso de Linux se encuentra en `/home/<tu usuario>/.ssh` 
- Para el caso de Windows se encuentra en `C:\Windows\Users\<tu usuario>\.ssh`

```
.
├── id_rsa
├── id_rsa.pub
```

Solo debe de compartirse la llave publica con el servidor, bajo ningún concepto debe de compartirse la llave privada.

## Archivo .config para ssh 

Si se utilizan múltiples llaves ssh es una buena practica generar nuestro archivo .config gestionarlas de manera inteligente. Para ello generaremos un nuevo archivo llamado `config` dentro de nuestra carpeta `.ssh` el contenido es el siguiente:

```ruby
# Ejemplo para una cuenta de github
Host github.com                     # Este es el alias que le asignaremos.
    HostName github.com             # La direccion a la que apunta
    User git                        # El usuario de github en este caso
    IdentityFile ~/.ssh/id_rsa      # Ruta de la llave privada
    IdentitiesOnly yes              # Indica que solo debe usar la clave especificada en IdentityFile

# Ejemplo para una cuenta de Gitlab
Host gitlab.com
    HostName gitlab.com
    User gitlab
    IdentityFile ~/.ssh/id_gitlab_rsa
    IdentitiesOnly yes

# Ejemplo para un servidor o infraestructura
Host aws.prod1.global.mobile
    HostName 52.55.222.44
    User certops
    IdentityFile ~/.ssh/id_aws_glomo_rsa
    IdentitiesOnly yes
```

## Copiar archivos con SCP

SCP (Secure Copy Protocol) es un comando que nos permite hacer una copia segura a través de SSH entre dos clientes o servidores. La sintaxis esta dada de la siguiente manera: 

```bash
#File es el archivo o directorio a compartir 
#User es el usuario ssh para realizar la conexion
#Host es el cliente o servidor destino
#Path es la ruta dentro del servidor o cliente donde se almacenara el archivo
scp <file> <user>@<host>:<path>
```
Ejemplo

```bash
scp linux-zen-kernel.tar.gz operator@10.10.1.3:/home/operator/Downloads
```
Donde: 

| Argumento | Descripcion |
| -------------- | --------------- |
| linux-zen-kernel.tar.gz | Archivo |
| operator | usuario que corresponde al servidor 10.10.1.3 |
| 10.10.1.3 | IP del servidor o cliente |
| /home/operator/Downloads | Ruta donde sera almacendo el archivo |

## Troubleshooting

Dentro de los errores mas comunes existe el equivocarse al colocar la contraseña al ingresar por SSH. Por lo que para solucionarlo el siguiente comando es de utilidad:


```bash
ssh-keygen -R <host>
```
Ejemplo:


```bash 
ssh-keygen -R 10.10.1.3
```
Donde:

| Argumento | Descripcion |
| -------------- | --------------- |
| -R | Elimina el host dentro del archivo know_hosts |
|10.10.1.3 | Host a eliminar |

## Importante 

> Este post se seguirá actualizando a manera de robustecer la información proporcionada.
{: .prompt-info }


# Practica 2: Xen

## Tarea 1: Puesta a punto de la infraestructura

### Preinstalación

El primer paso es comprobar que las extensiones de virtualización (VT-x) esten habilitadas para inspeccionamos el fichero /proc/cpuinfo
```bash
cat /proc/cpuinfo | grep vmx
```
Lo siguiente es buscar una partición que no se esté utilizando. En este caso la partición sda5 contiene ubuntu, la partición sda6 contiene un debian y la partición sda7 otro debian, el que estoy utilizado. Para cerciorarme de que no hay nada importante en la partición sda6 la monté en el directorio /mnt con el comando `mount /dev/sda6 /mnt`, tras revisarlo se desmontó con el comando `umount /mnt`

En este caso, como el espacio no era suficiente para crear todos los volúmenes necesarios, tuve que que reducir la partición de ubuntu. Para ello era necesario acceso a la cuenta root, por lo que monté el volumen de ubuntu en el directorio /mnt desde debian y modifiqué el fichero /etc/shadow para quitar la contraseña al usuario root de ubuntu y poder acceder. Una vez dentro de ubuntu reduje la partición con ayuda de la herramienta gráfica gparted.

Por último, con las particiones listas creé el volumen lógico que se pide siguiendo estos pasos:
```sh
#Creación de volumenes fisicos
pvcreate /dev/sda6
pvcreate /dev/sda8
#Creación del grupo de volumenes
vgcreate vol1 /dev/sda6 /dev/sda8
#Creación del volumen logico
lvcreate -L15Gi -n base vol1
```

### Instalación

El siguiente paso es intalar Xen de los respositorios de debian, crear una entrada en el GRUB para poder acceder al Dom0 y configurar los limites de CPU y memoria del Dom0. Para hacer esto se siguieron los siguientes pasos:
```bash
# Instalar xen
apt update;apt install xen-hypervisor
# Anhadir entrada al GRUB
update grub
# Configurar CPU y memoria
vi /etc/xen/xl.conf # memory=1024
xl vcpu-set Dom0 2 # Esta es la manera de hacerlo dinámico, se puede hacer también estatico modificando el fichero como hice con la memoria
```
Después de esto para que se apliquen los cambios introducidos en el fichero /etc/xen/xl.conf es necesario reiniciar el servicio xen o reiniciar la máquina. Sin embargo, al hacer eso hay que volver a ejecutar el comando `xl vcpu-set Dom0 2`, ya que es un cambio en tiempo de ejecución y al reiniciar se pierde.

Para concluir el proceso de instalación se pide que comprobar los errores o adventencias, para ello solo hay que ejecutar el comando `dmesg`.

### Post-Instalación

Para configurar la red hay que crear un bridge, pero con las herramientas tradicionales no ha funcionado, así que ha sido necesario recurrir al paquete de software libvirt mediante la siguiente secuencia de comandos:

```sh
apt install libvirt-clients libvirt-daemon-system iptables ebtables dnsmasq-base
virsh net-start default
virsh net-autostart default
```
Tras instalar todos los paquetes necesarios, se le indica a xen el bridge mediante la interfaz virbr0 en el fichero /etc/xen/xl.conf.
## Tarea 2: Creación de DomU's

### Instalación de DomU
Para esta tarea utilicé el volumen lógico vol1-base creado en la tarea anterior y ejecutando los siguientes comandos instalé debian buster.
```bash
mkfs.ext4 /dev/mapper/vol1-base
mount /dev/mapper/vol1-base /mnt
debootstrap --arch amd64 buster /mnt/ http://ftp.debian.org/debian
```
Después de preparar el volumen, generé el fichero de configuración de DomU a partir de uno creado con las herramientas de automatización xen-tools, indicando el kernel, la particion de root, el disco, el nombre del dominio y el comportamiento deseado cuando el Dom0 es apagado, reiniciado o crashea.

Para arrancar el DomU y comprobar que funciona correctamente hay que usar el comando `xl create -c <pathAlFichero>`. La consola da problemas por lo que una de las primeras cosas que tuve que hacer fue intalar `openssh`, modificar la configuración para poder conectarme como root y crear un par de claves para no tener que usar la contraseña.

Con todo ya funcinando desplegué los benchmarks de la Practica 1.

El siguiente paso es crear dos clones del DomU: uno mediante un snapshot seguro (del mismo tamaño que el volumen original) y otro creando un fichero de backup y restaurandolo en un fichero monolítico. En mi caso realicé dos clones mediante snapshot porque no disponia de espacio sufiente en el disco para generar el fichero comprimido de backup. Los pasos para crear los clones son los siguientes:
```bash
# Safe snapshot
lvcreate -L15Gi -s -n clone1 /dev/vol1/base
# Backup
dd if=/dev/vol1/base | gzip /home/ricardo/base_backup.img.gzip
# Monolitic file
gzip -dc | dd of=/xen/domains/clone2.img
```
Por último, depues de generar los discos es necesario copiar el fichero de configuración cambiando los parámetros *disk* y *name*
## Tarea 3: Gestión de DomU's

### Uso

En esta versión de xen todos los dominios son gestionados. No existe el new. xl create -p. Para tomar las medidas se utiliza el comando `time` de linux.

#### Boot
```
real 0m0.676s
user 0m0.539s
sys 0m0.140s
```
#### Shutdown
```
real 0m0.013
user 0m0.006
sys 0m0.007
```
#### Checkpoint 
```
real 0m13.948s
user 0m0.019s
sys 0m1.198s
```
#### Restore
```
real 0m12.794s
user 0m0.374s
sys 0m0.848s
```
### Arranque

#### PV GRUB
```
En el Dom0
	kernel          = /usr/lib/grub-xen/grub-x86_64-xen.bin
	extra           = '(hd0)/boot/grub/grub.cfg'
En el DomU
	mkdir /boot/grub
	apt-get install linux-image-amd64
	Install grub2-common, and run update-grub to create /boot/grub/grub.cfg
```

#### Kernel from backports: 
```sh
# add backports repository to /etc/apt/sources.list
apt install linux-image-5.4.8
```

### QoS de recursos

Para ejecutar los tres SPECcpu se utiliza el comando `parallel-ssh` que se incluye en el paquete pssh. Para instalarlo `apt install pssh`.

#### Asignando un core a cada DomU

Para asignar un core a cada DomU se hace lo siguiente:
```bash
xl vcpu-set Domain-0 1
xl vcpu-set debian10 1
xl vcpu-set clone1 1
xl vcpu-set clone2 1

xl vcpu-pin Domain-0 0 0
xl vcpu-pin debian10 0 1
xl vcpu-pin clone1 0 2
xl vcpu-pin clone2 0 3
```

#### Restringir las cpu 2 y 3 para los DomU

Para restringir las cpu 2 y 3 para los DomU:
```bash
xl vcpu-pin debian10 0 2-3
xl vcpu-pin clone1 0 2-3
xl vcpu-pin clone2 0 2-3
```

#### Incrementar el peso de base para el planificador

Para incrementar el peso de la base para el planificador:

```bash
xl sched-credit # Para ver los valores iniciales (por defecto el peso es 256)
xl sched-credit -d debian10 -w 4096
```
#### Resultados

En la tabla se mostrarán los resultados de los tres DomU, para comparar los cuatro casos propuestos. Para esta tarea solo se ha ejecutado el test h264ref.

| Caso | debian10 | clone 1 | clone 2|
|---|---|---| --- |
|Base||||
|1 core por DomU||||
|Cores 2 y 3 para DomU||||
|Incrementar pero DomU base||||


### Limpieza

Para eliminar los snapshot y clones:

```bash
lvremove /dev/mapper/vol1-clone1
lvremove /dev/mapper/vol1.clone2
```
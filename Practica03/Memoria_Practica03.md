# Practica 3: Máquinas Virtuales: Optimización de rendimiento

## Tarea 1

### Crear una instalación mínima de un DomU HVM 
Para crear una máquina virtual de tipo HVM creamos una copia del fichero de configuración de ejemplo que existe en el directorio /etc/xen/, modificando algunos de los parámetros. A continuación, descargamos un disco de instalación de debian 10 buster e instalamos un cliente vnc para poder acceder al menú de instalación que aparecerá al arrancar el DomU. Una vez descargada la imagen de instalación, añadimos un nuevo disco de tipo CDROM en el fichero de configuración con el path a la imagen y ajustamos el orden de boot para poner en primer lugar el CD. Creamos, además, un volumen lógico de pequeño tamaño donde se realizará la instalación del sistema.
```
type=hvm
name=debian_hvm
boot=cda
vif = [ bridge=virbr0 ]
disk = [ 'phy:/dev/mapper/vol1-hvm,xvda,rw','file:/xen/netinst.iso,xvdb:cdrom,rw' ]
vnc=1
```

Tras todo este proceso, arrancamos el DomU y nos conectamos mediante el cliente vnc viewer . Al hacer esto se abrirá una ventana en la que veremos el instalador de Debian en modo texto. Siguiendo los pasos, realizamos una instalación mínima.

Al finalizar, modificamos el fichero de configuración de la máquina HVM para quitar el CDROM del listado de discos de máquina y evitar así que vuelva a iniciarse el instalador. Con esto debería ser posible iniciar la máquina virtual con el comando `xl create -c /etc/xen/debian-hvm.cfg`.

### Configurar la máquina HVM para tener acceso al disco de la máquina PV (reutilizar /)

Para conseguir que la máquina HVM tenga acceso al disco de la máquina PV es necesario realizar 2 cambios:
 - Añadir el disco al fichero de configuración de la máquina
    ```
    disk = [ 'phy:/dev/mapper/vol1-hvm,xvda,rw','file:/xen/netinst.iso,xvdb:cdrom,rw', 'phy:/dev/mapper/vol1-debian10,xvdc,rw' ]
    ```
 - Una vez dentro de la máquina, instalar grub y modificar la entrada del grub para que monte el directorio raiz sobre el dispositivo en el que se encuenta mapeado el volumen lógico de la máquina PV.
    ```
    root = /dev/xvdc
    ```
## Tarea 2: Cargas de trabajo limitadas por CPU

### Comparar el rendimiento de SPECcpu en los siguientes casos: Nativo, PV sin VT-x, PV con VT-x y HVM on PV

Para este apartado ya contamos con los resultados del benchmark en la máquina nativa y en este caso ejecutar PV con VT-x y ejecutar sin VT-x nos ofrecería un rendimiento similar. Por tantom nos centramos en ejecutar solamente los casos de PV con VT-x y HVM on PV.

|Tipo de virtualizacion |h264ref |calculix|
|---|---|---|
|Nativo|||
|PV con VT-x|||
|HVM on PV|||

#### ¿Por qué se produce la perdida de rendimiento en el caso de PV sin VT-x?

Para constestar a este apartado repetimos las pruebas realizadas con en la máquina virtual PV con VT-x pero analizando la ejecución con perf. Concretamente, analizaremos los siguientes eventos: evento 1, evento 2, evento 3, evento 4. Estos eventos nos mostrarán el número de veces que tiene que intervenir el hipervisor para realizar determinadas operaciones.

##### h264ref

| Evento | Ocurrencias |
|---|---:|
|xen:xen_mc_entry|7451|
|xen:xen_mmu_flush_tlb_one_user|1238|
|xen:xen_mmu_alloc_ptpage|849|
|xen:xen_mmu_realease_ptpage|852|

##### calculix

| Evento | Ocurrencias |
|---|---:|
|xen:xen_mc_entry|6038|
|xen:xen_mmu_flush_tlb_one_user|1340|
|xen:xen_mmu_alloc_ptpage|527|
|xen:xen_mmu_realease_ptpage|530|

Como se puede ver en las tablas hacen muchas llamadas al hipervisor, debido a los accesos a memoria o al uso de llamadas al sistema y esto introduce un overhead que causa un descenso del rendimiento.

## Tarea 3: QoS para SPECjbb 

### Configurar dos DomU para ejecutar SPECjbb

### Ejecutar SPECjbb simultaneamente para intentar alcanzar un 80% del rendimiento nativo en uno de ellos

Para este caso se ha optado por utilizar clones como los de la práctica 2. Para sincronizar la ejecución se ha utilizado parallel-ssh. Y se han probado 2 configuraciónes diferentes para tratar de alcanzar el 80% del rendimiento nativo:
 - Dom0 limitado a un core fisico y compartiendolo con el clone y máquina base con 4 vcores.
 - Misma configuración que en el caso anterior pero aumentando los creditos del planificador para la máquina base de manera que tenga 16 veces más prioridad

 Los datos se muestran en la siguiente tabla:

| Nativo | 80% | Alternativa 1 | Alternativa 2 |
| :---: | :---: | :---: | :---: |
| 109855 | 87884 | 70801 | 83317 |

Vemos como en ninguno de los dos intentos hemos sido capaces de alcanzar el 80% del rendimiento, pero en uno de los casos, si que ha sido posible quedarse relativamente cerca. Con esta alternativa podemos garantizar un rendimiento cercano al 75,84% en una de las dos máquinas.   

## Tarea 4: SPECweb

### Comparar el rendimiento de SPECweb en los siguientes casos

En este apartado se pide configurar y ejecutar el benchmark para correr sobre dos máquina virtuales de tipo HVM-on-PV: una para el servidor web y otra para el cliente y el BackEnd Simulator. Posteriormente hay que modificar el fichero de configuración para añadir PCI Pass Through a la red y repetir el proceso con el objetivo de comparar el rendimiento de ambas configuraciones. Por último, sobre esta configuración modificar algunos párametros de QoS de las máquinas virtuales como en el apartado anterior y repetir el proceso.

Tras crear una segunda máquina virtual HVM-on-PV procedí a configurar las diferentes partes del benchmark. Pero, en el momento de ejecutar, me apareció un fallo que no me había surgido primero y, que consistía en que tanto el servidor web como el BackEnd Simulator estaban disponibles pero se producia un error con el generador de carga de trabajo. Debido a una mala gestión del tiempo por mi parte no he podido resolver el problema y ejecutar el benchmark. Así que describiré el proceso de configuración seguido y los pasos que creo que habrían sido necesarios para añadir PCI-Pass-Through.

##### HVM-on-PV

Para configurar este caso es necesario configurar las partes de SPECweb necesarias en cada máquina. En mi caso, descargue el benchmark y configuré el sevidor web apache y, posteriormente, creé un clon de esa máquina virtual. 

El proceso es igual que en el caso de desplegarlo en una sola máquina, con la diferencia de que en una solo hay que poner los scripts php y el contenido estático y en la otra el BackEnd Simulator. Con esto montado, hay que modificar las direcciones IP en el fichero Test.config para que el cliente del benchmark pueda comunicarse con el servidor web y el servidor web con el BackEnd Simulator.

##### PCI-Pass-Through

Para activar esta característica es necesario modificar el fichero de configuración de los DomU y reiniciarla.

```sh
# Buscar los dispositivos de Ethernet
lspci | grep Ethernet
# Desconectar el dispositivo del kernel de manera dinámica
modprobe xen-pciback; update-initramfs -u BDF=0000:02.01.0 
# Desconectar el dispositivo del Dom0 
echo -n $BDF > /sys/bus/pci/devices/$BDF/driver/unbind 
# Añadir un nuevo slot al listado de PCI  
echo -n $BDF > /sys/bus/pci/drivers/pciback/new_slot 
# Por ultimo, se asigna el dispositivo al slot creado 
echo -n $BDF > /sys/bus/pci/drivers/pciback/bind
# Modificar el fichero de configuración de los DomU's y comprobar que se han asignado correctamente
vi /etc/xen/debian_hvm.cfg # pci =['02:05.0']
vi /etc/xen/debian_hvm2.cfg # pci =['02:05.0']
xl pci-list debian_hvm
xl pci-list debian_hvm2
```

##### QoS 

Para este aparado se habria que utilizar los mismos comando que se emplearon en la practica anterior para SPECcpu:
- `xl vcpu-set`
- `xl cpu-pin `




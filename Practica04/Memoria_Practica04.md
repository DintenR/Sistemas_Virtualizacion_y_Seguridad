# Practica 4: Contenedores y servicios de cloud publicos

## Tarea 1: LXC

### Repetir la tarea 3 de la practica 3 intentando alcanzar un SLA del 75%

El primer paso que hay que realizar para llevar a cabo esta tarea es instalar lxc. Para ello:

```bash
apt update
apt install lxc
```

Lo siguiente es crear dos contenedores debian. Para crear el contenedor utilizo la plantilla que hay disponible en /usr/share/lxc/templates

```bash
lxc-create -t debian -n debian_base
```
Con esto se nos genera un directorio en /var/lib/lxc con el nombre debian_base que contiene la configuración y el sistema de ficheros raiz del contendor.


También es necesario configurar la red en modo brigde para lo que seguimos los pasos que aparecen en las diapositivas de la asignatura y modificamos el fichero /etc/lxc/default.conf asignando los siguientes parámetros:

```
lxc.net.0.type = veth 
lxc.net.0.link = virbr0 
lxc.net.0.flags = up 
lxc.apparmor.profile = generated 
lxc.apparmor.allow_nesting = 1

```

En cada contenedor se introducen java y el benchmark. Se inician los contendores y para intentar que uno alcance una puntuación alta, cercana a la nativa, se modifican dos parametros de los cgroups de los contenedores:
- En debian_clone para restringirlo a una sola cpu
- En debian_base dar mayor prioridad al contendor debian

Hay dos alternativas, una temporal para realizarlo con el contendor en ejecución y que al parar el contenedor se reincia y otra estática en la configuración del contenedor.

```sh
echo 0 > /sys/fs/cgruops/cpuset/lxc/debian_clone/cpuset.cpus
echo 64 > /sys/fs/cgruops/cpu/lxc/debian_clone/cpu.shares
echo 0-3 > /sys/fs/cgruops/cpuset/lxc/debian_base/cpuset.cpus
echo 1024 > /sys/fs/cgruops/cpu/lxc/debian_base/cpu.shares
```

```sh
#En /var/lib/lxc/debian_clone/config
lxc.cgroups.cpuset.cpus = 0
lxc.cgrups.cpu.shares = 64
#En /var/lib/lxc/debian_base/config
lxc.cgroups.cpuset.cpus = 0-3
lxc.cgrups.cpu.shares = 1024
```
Los resultados obtenidos son los que se muestran en la siguiente tabla:

| Nativo | 75% | Alternativa 1 | Alternativa 2 |
| :---: | :---: | :---: | :---: |
| 109855 | 82391 | 83317 | 89583 |

A pesar de haber alcanzado el objetivo con la primera de las alternativas se probó la segunda configuración utilizada en la práctica anterior, ya que se vió que ofrecía mayor rendimiento. En este caso al contrario que en el anterior se ha conseguido alcanzar el objetivo marcado, incluso se ha conseguido alcanzar el 80% que se pedía en el ejercicio de la práctica anterior. Esto nos demuestra que la mayor ligereza de los contenedores frente a las máquinas virtuales repercute en un mejor rendimiento.
## Tarea 2: Docker

### Desplegar SPECweb en 3 contenedores Docker (servidor web, cliente y BeSim) usando imagenes incrementales

El primer paso para realizar esta tarea es instalar docker. 
```sh
su -i
curl -sSL https://get.docker.com/ | sh
usermod -aG docker riki3897
```
Lo siguiente es escoger una imagen base para generar las imagenes incrementales necesarias. En este caso se toma como base debian:buster.
En primer lugar hay que definir las cosas necesarias en las imagenes:
 - En la imagen para el servidor es necesario instalar apache2 y php5.6.
 ```sh
 docker run -dit --name webserver debian:buster
 ```
 - En la imagen del BackEnd simulator es necesario instalar apache2 y el modulo fcgi.
 ```sh
 docker run -dit --name besim debian:buster
 ```
 - Por último en la imagen del cliente es necesario instalar curl y java 5.
 ```sh
 docker run -dit --name client debian:buster
 ```
Una vez instalado el software necesario en cada uno de los contendores, los subo al repositorio privado para tenerlos accesibles. Como en cada uno de ellos es necesario tener una serie de fichero e incluirlos en la imagen incrementa demasiado su tamaño, se subiran las imagenes solamente con el software y se montarán los directorios de la máquina dentro del contenedor a la hora de crearlo.
```sh
# Subir imagenes al repositorio remoto
docker login
docker commit webserver dintenr/svs:webserver
docker push dintenr/svs:webserver
docker commit besim dintenr/svs:besim
docker push dintenr/svs:besim
docker commit client dintenr/svs:client
docker push dintenr/svs:client

# Eliminar los contenedores
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)

# Crear los contenedores con el mapeo de directorios y puertos apropiados
docker run -dit --publish 80:80 --name webserver --volume /var/www:/var/www dintenr/svs:webserver
docker run -dit --publish 81:80 --name besim --volume /var/www:/var/www dintenr/svs:besim
docker run -dit --name client --volume /tmp/client:/tmp dintenr/svs:client
```
Como se ve en el ejemplo anterior se mapea el puerto 80 del contendor con el 80 de la máquina en el webserver y el 80 del contendor con el 81 de la máquina en el besim para evitar que se solapen y provoque un error. Estos cambios así como las direccion IP hay que indicarlas en el fichero de configuración Test.config

### Ejecutar y comparar con el mejor Xen

Se realizan varias ejecuciones del benchmark con distinto número de sesiones simultáneas hasta encontrar el punto en el que el porcentaje de respuestas correctas baja de 98%. Comenzando en 100 y ajustando el número a la vista de los resultados.

#### Test Support

**Primera ejecución**: 100 sesiones simultaneas -> 100% 

**Segunda ejecución**: 125 sesiones simultaneas -> 95%

**Tercera ejecución**: 110 sesiones simultaneas -> 100%

**Cuarta ejecución** : 120 sesiones simultaneas -> 94%

A la vista de estos resultados podemos determinar que, en Test Support, el benchmark consigue una puntuación entre 110 y 120 cuando se ejecuta sobre docker, frente a una puntuación de xxx cuando se ejecuta sobre máquina virtual

#### Test Banking

**Primera ejecución**: 100 sesiones simultaneas -> 100% 

**Segunda ejecución**: 120 sesiones simultaneas -> 100%

**Tercera ejecución**: 150 sesiones simultaneas -> 80%

**Cuarta ejecución** : 130 sesiones simultaneas -> 100%

**Quinta ejecución** : 140 sesiones simultaneas -> 96%

A la vista de estos resultados podemos determinar que, en Test Banking, el benchmark consigue una puntuación entre 130 y 140 cuando se ejecuta sobre docker, frente a una puntuación de xxx cuando se ejecuta sobre máquina virtual

### Subir los contenedos a Dockerhub de manera privada

## Tarea 3: GCE/Amazon EC2 

### Desplegar los contenedores creados en la tarea 2 en AMI separados en Amazon EC2 o  Google Compute Engine
Para esta tarea es necesario desplegar 3 máquinas virtuales en Google Compute Engine y en cada una correr uno de los contenedores Docker. El proceso que he seguido para ello ha sido desplegar  primero una máquina y descargar e instalar en ella lo necesario: docker, SPECweb, besim, ejecutar Wafgen...

Cuando ya he terminado de configurar adecuadamente la máquina he creado una snapshot y, a partir de la snapshot he creado 2 réplicas de la máquina. En cada una de las máquina he desplegado un contenedor y he modificado el fichero Test.config de SPECweb para indicarle cual es la dirección ip de cada uno de los servidores. Estas direcciones ip se corresponden con las direcciones ip externas que aparecen en la consola de Google Compute Engine.
### Medir el rendimiento   

Al igual que en el caso anterior, se toma 100 sesiones como referencia y se va ajustando el número de sesiones en función de los resultados obtenidos. Para hacer la medida de rendimiento se han usado 3 máquinas N1 standard con 1cpu y 3.75GB de memoria

#### Test Support

**Primera ejecución**: 100 sesiones simultáneas -> 100% 
**Segunda ejecución**: 150 sesiones simultáneas -> 70%
**Tercera ejecución**: 130 sesiones simultáneas -> 95%
**Cuarta ejecución** : 115 sesiones simultáneas -> 99%
**Quinta ejecución** : 120 sesiones simultáneas -> 99%

A la vista de estos resultados podemos ver que el servidor es capaz de cumplir un 98% de las peticiones cuando se le somete a una carga de entre 120 y 130 sesiones simultáneas. Esto es sorprendente, ya que es un número similar al obtenido en el laboratorio donde se estaba empleando una máquina con 4 cores y en este la máquina que aloja el servidor cuenta con un unico core. Esto me hace pensar en tres posibles causas: 
- El core de esta máquina es mucho más potente que los empleado en el laboratorio debido a que se trata de un procesador pensado para servidores.
- El benchmark tiene alguna limitación que impide obtener mejores resultados
- El servidor o el benchmark no aprovechan los cuatro núcleos del procesador de manera adecuada.

#### Test Banking

**Primera ejecución**: 100 sesiones simultaneas -> 100% 
**Segunda ejecución**: 140 sesiones simultaneas -> 93%
**Tercera ejecución**: 120 sesiones simultaneas -> 100%
**Cuarta ejecución** : 130 sesiones simultaneas -> 100%
**Quinta ejecución** : 135 sesiones simultaneas -> 99%

En el caso del Test Banking la máquina soporta entre 135 y 140 sesiones simultáneas para un SLA del 98%. Lo cuál sigue siendo sorprendente porque, nuevamente, se parece bastante a los resultados obtenidos en el laboratorio.
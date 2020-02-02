# Practica01: Evaluación del Rendimiento
En esta práctica se va a realizar una evaluación de rendimiento utilizando los benchmarks SPECcpu y SPECweb. En la primera de las partes,
se descarga, compila y ejecuta el benchmark SPECcpu, con el objetivo de evaluar el efecto de las diferentes opciones de compilación.
En la segunda parte, se descarga compila y ejecuta SPECweb para medir el efecto que tiene en el rendimiento la elección de del web-server 
y la JVM.


## Parte 1: Evaluación con SPECcpu2006
El primer paso ha sido descargar SPECcpu2006 y los compiladores gcc, g++ y gfortran. Después, se ha ejecutado el instalador de SPECcpu2006 y, a partir de un fichero de configuración que se pueda utilizar para el procesador de la maquina se ha generado el fichero ricardo-linux.cfg. En ese fichero se han actualizado las rutas de los compiladores y se han añadido las opciones de compilación -O3 -march=native con el objetivo de optimizar al máximo el rendimiento del programa.

Una vez definido el fichero de configuración se han asignado las variables de entorno necesarias con el comando source shrc y se ha probado a compilar un test de enteros para comprobar que el sistema funciona. 
### Test de enteros
Con el sistema configurado, se ha procedido a compilar todos los test de enteros y se han obtenido fallos en 2 de ellos: xalancbmk y perlbench.
El error de xalancbmk estaba relacionado con algunas librerias que no estaban disponibles a la hora de realizar la compilación. Se ha solucionado añadiendo al fichero ricardo-linux.cfg dos flags: 

El error de perlbench no se ha solucionado.
### Test de coma flotante
Tras compilar los test de enteros, se han compilado los test de coma flotante. La primera vez que se intentó compilar hubo muchos fallos debido a que se había especificado mal la ruta del compilador de fortran. Tras solucionar esto aparecieron errores en 3 test: 416.gamess, 447.dealII y 450.

En el caso de 416.gamess hay un error de compatibilidad debido a la version del compilador y se ha solucionado añadiendo la opción -std=legacy a las opciones de compilación.

En el caso de 417.dealII fue necesario introducir los siguientes flags: -Wdeprecated, -fpermisive, -Wdeprecated-declarations, -include cstring.

En el caso del test 450. no se hizo nada ya que es un problema complejo que supundría demasiado tiempo solucionar y no aprota valor real a la practica, simplemente se omite este test a la hora de ejecutar el benchmark.

Por último, ocurria un problema también con el test 481.wrf. Este problema se puede solucionar añadiendo la opción wrf_data_header_size=4. Ya que, por defecto, este toma valor 8 y provoca un error.

### Resultados

En este apartado se presentan los resultados de las diferentes ejecuciones de SPECcpu, antes de ejecutar el benchmark se ha usado un script obtenido en GitHub que permite desactivar la función TurboBoost, de manera que esta no interfiera con los resultados. 

La primera de las ejecuciones ha sido con todos los test y con las opciones indicadas anteriormente. Obteniendo los siguientes resultados:

#### INT
| Benchmarks   |Base Ref. | Base Run Time  | Ratio |     
| -------------- | ------ | --------- | ---------  |
|400.perlbench   | 9770   |      --   |         CE |                                 
|401.bzip2       | 9650   | 361       |  26.8   *  |                                 
|403.gcc         | 8050   | 186       |  43.3   *  |                                 
|429.mcf         | 9120   | 196       |  46.5   *  |                                 
|445.gobmk       |10490   | 368       |  28.5   *  |                                 
|456.hmmer       | 9330   | 125       |  74.6   *  |                                 
|458.sjeng       |12100   | 396       |  30.6   *  |                                 
|462.libquantum  |20720   | 223       |  93.0   *  |                                 
|464.h264ref     |22130   | 361       |  61.3   *  |                                 
|471.omnetpp     | 6250   | 244       |  25.6   *  |                                 
|473.astar       | 7020   | 318       |  22.1   *  |                                 
|483.xalancbmk   | 6900   | 145       |  47.4   *  |

#### FP
                            
| Benchmarks   |Base Ref. | Base Run Time  | Ratio |     
| -------------- | ------ | --------- | ---------  |  
| 410.bwaves     | 13590  |  246      |   55.2   * |                                 
| 416.gamess     | 19580  |   50.5    |          VE|                                 
| 433.milc       |  9180  |  234      |   39.2   * |                                 
| 434.zeusmp     |  9100  |  152      |   60.0   * |                                 
| 435.gromacs    |  7140  |  213      |   33.5   * |                                 
| 436.cactusADM  | 11950  |  156      |   76.7   * |                                 
| 437.leslie3d   |  9400  |  133      |   70.9   * |                                 
| 444.namd       |  8020  |  251      |   32.0   * |                                 
| 447.dealII     | 11440  |  219      |   52.3   * |                                 
| 450.soplex     |  8340  |       --  |          CE|                                 
| 453.povray     |  5320  |   95.3    |   55.8   * |                                 
| 454.calculix   |  8250  |  547      |   15.1   * |                                 
| 459.GemsFDTD   | 10610  |  203      |   52.3   * |                                 
| 465.tonto      |  9840  |  235      |   42.0   * |                                 
| 470.lbm        | 13740  |  188      |   73.1   * |                                 
| 481.wrf        | 11170  |  148      |   75.6   * |                                 
| 482.sphinx3    | 19490  |  372      |   52.4   * | 

En la segunda de las ejecuciones, se cambió el flag march. En la ejecución anterior se puso -march=native para que la compilación se hiciera de acuerdo con la arquitectura del procesador empleado. En este caso, se trata de un procesado de 64-bits. En esta ejecución se ha modificado ese parámetro para que la compilación se realize pensando en el ISA de un procesador x86 de 32bits. El objetivo es ver como afecta al rendimiento general del sistema el conjunto de intrucciones disponibles a la hora de compilar. Los resultados obtenidos son los siguientes:

#### INT
| Benchmarks   |Base Ref. | Base Run Time  | Ratio |     
| -------------- | ------ | --------- | ---------  |
|400.perlbench   |         |            |       NR  |                 
|401.bzip2       | 9650    |363         |26.6   *   |                               
|403.gcc         |         |            |      NR   |      
|429.mcf         | 9120    |113         |80.8   *   |                               
|445.gobmk       |10490    |374         |28.1   *   |                               
|456.hmmer       | 9330    |134         |69.7   *   |                               
|458.sjeng       |12100    |432         |28.0   *   |                               
|462.libquantum  |20720    |438         |47.3   *   |                               
|464.h264ref     |22130    |375         |59.0   *   |                               
|471.omnetpp     | 6250    |244         |25.6   *   |                               
|473.astar       | 7020    |330         |21.3   *   |                               
|483.xalancbmk   | 6900    |149         |46.4   *   |

#### FP
| Benchmarks   |Base Ref. | Base Run Time  | Ratio |     
| -------------- | ------ | --------- | ---------  |
|410.bwaves      |13590   | 275       |  49.3   *  |                                
|416.gamess      |        |           |         NR |                                
|433.milc        | 9180   | 286       |  32.1   *  |                                
|434.zeusmp      | 9100   | 205       |  44.3   *  |                                
|435.gromacs     | 7140   | 305       |  23.4   *  |                                
|436.cactusADM   |        |           |         NR |                                
|437.leslie3d    | 9400   | 137       |  68.5   *  |                                
|444.namd        | 8020   | 335       |  24.0   *  |                                
|447.dealII      |11440   | 231       |  49.6   *  |                                
|450.soplex      |        |           |         NR |                                
|453.povray      |        |           |         NR |                                
|454.calculix    | 8250   | 630       |  13.1   *  |                                
|459.GemsFDTD    |10610   | 323       |  32.8   *  |                                
|465.tonto       | 9840   | 401       |  24.6   *  |                                
|470.lbm         |13740   | 241       |  57.0   *  |                                
|481.wrf         |11170   | 209       |  53.5   *  |                                
|482.sphinx3     |19490   | 331       |  58.9   *  |

Como podemos ver en las tablas, al recompilar y ejecutar el benchmark con el parámetro `-m32` algunos de los benchmark no compilan correctamente y lo que lo hacen tienen un peor rendimiento al tener un conjunto de instrucciones menor que desaprovecha algunas de las mejoras introducidas con las arquitecturas de 64bits.

### Perf

En este apartado se ejecutará un benchmark de cada suit, es decir, uno de la suite de enteros y otro de la suite de coma flotante. En el caso de enteros usaremos 464.h264ref y en el caso de coma flotante usaremos 454.calculix. En este caso los ejecutaremos con tamaño de problema ref y una iteración para analizarlos con perf y determinan cual esta limitado por memoria y cual por cpu principalmente y cuanta energia se malgasta debido a la especulación:

En el primero de los casos se ejecutan los benchmarks usando la información que muestra perf por defecto con los siguientes comandos:

```sh
#Test de enteros
perf stat -d runspec --config=ricardo-linux.cfg --tune=base --noreportable --size=ref --iterations=1 h264ref

#Test de coma flotante
perf stat -d runspec --config=ricardo-linux.cfg --tune=base --noreportable --size=ref --iterations=1 calculix
```

Los resultados se muestan en las siguientes imagenes:

![Resultados perf stat para el test de enteros](https://github.com/DintenR/Sistemas_Virtualizacion_y_Seguridad/blob/master/Practica01/Imagenes/SPECcpu_nativo_fp_perf.PNG)

![Resultados perf stat para el test de coma flotante](https://github.com/DintenR/Sistemas_Virtualizacion_y_Seguridad/blob/master/Practica01/Imagenes/SPECcpu_nativo_fp_perf.PNG)

Como se puede apreciar en las imagenes, en el test de numeros enteros se producen un mayor número de cambios de contexto, fallos de pagina y lectutas de L1, lo que nos indica que esta más limitado por la memoria que el test de coma flotante.

Para el segundo caso no es suficiente con mostrar los datos básicos de perf. Para poder calcular el malgasto producido por la especulación es necesario recurrir al manual del procesador para encontrar el evento que deseamos analizar y después con `perf list` comprobar el nombre exacto. En este caso necesitamos saber cuantas instrucciones se ejecutan y cuantas se descartan. Estos datos los obtenemos de los contadores `uops.exetuted` y `uops.issued`. Además, se necesita saber el consumo energético, pero en este procesador no estan soportados esos contadores por tanto el malgasto energético se mostrará en porcentaje. Una vez conocidos los contadores hay que ejecutar los siguientes comandos

```sh
#Test de enteros
perf stat -e uops_executed.thread,uops_issued.any runspec --config=ricardo-linux.cfg --tune=base --noreportable --size=ref --iterations=1 h264ref

#Test de coma flotante
perf stat -e uops_executed.thread,uops_issued.any --config=ricardo-linux.cfg --tune=base --noreportable --size=ref --iterations=1 calculix
```

Los resultados son los que se muestran en las imágenes:

![Resultados del análisis de la especulación con perf para el test de enteros](https://github.com/DintenR/Sistemas_Virtualizacion_y_Seguridad/blob/master/Practica01/Imagenes/perf_uops_int.png)

![Resultados del análisis de la especulación con perf para el test de coma flotante](https://github.com/DintenR/Sistemas_Virtualizacion_y_Seguridad/blob/master/Practica01/Imagenes/perf_uops_fp.png)

Como se puede ver, en el caso de los enteros, el procesador ejecuta un total de 3.511.686.768.719 uops y retira 3.203.017.076.374 uops, por tanto esto supone un malgasto de 8.79 %. En el caso de coma flotante, el procesador ejecuta un total de 3.821.039.136.496 uops y retira 3.481.674.726.732 uops, por tanto esto supone un malgasto de 8.89 %.

## Parte 2.1: SPECjbb2006

En este apartado hay que descargar e instalar el benchmark SPECjbb2006. El proceso es más sencillo que en el caso de SPECcpu y de SPECweb. Solamente hay que descargar, descomprimir y ejecutar. Previamente es necesario instalar una version del jdk de java.

```sh
# Decargar en instalar jdk
wget https://www.ce.unican.es/%7evpuente/SVS/jdk-1_5_0_22-linux-amd64.bin
chmod +x jdk-1_5_0_22-linux-amd64.bin
./jdk-1_5_0_22-linux-amd64.bin
export PATH=/home/ricardo/jdk1.5.0_22/bin:$PATH

# Descargar y descomprimir SPECjbb2006
https://www.ce.unican.es/%7evpuente/SVS/SPECjbb2005.tar.gz
tar -xvf SPECjbb2005.tar.gz
```
### Resultados

Durante la ejecución el benchmark va aumentando el número de warehouses, por defecto, de 1 a 8. Esta configuración es suficiente ya que el procesador cuenta con 4 cores y se puede ver como el rendimiento va aumentando hasta llegar a 4 cores donde alcanza el máximo y comienza a decaer.

| Warehouses    |           Thrput    |
| ------------- | ------------------- |
|              1|                57682|
|              2|                95410|
|              3|               113135|
|            * 4|               119574|
|            * 5|               111684|
|            * 6|               105852|
|            * 7|               107408|
|            * 8|               104757|

## Parte 2.2: SPECweb2005

En este apartado se explicará el proceso seguido para intalar, configurar y ejecutar el benchmark SPECweb y se mostrarán los resultados de su ejecución.

### Instalación y configuración

Para instalar el software simplemente ha sido necesario descargar el archivo comprimido de la web del profesor y descomprimirlo. Al hacer esto aparece un directorio que contiene casi todos los elementos necesarios para ejecutar el benchmark, excepto el servidor web o servidor de aplicaciones.

Llegados a este punto hay que tomar una decisión y escoger la versión de basada en jsp o la versión basada en php. Dado que, según el profesor, la versión basada en jsp ofrece un mejor rendimiento y es algo más actual, en un primer intento se optó por utilizar esta. Esto implicaba descargar el servidor de aplicaciones Apache Tomcat, java 1.5, apache2 y el modulo cgi para el backend simulator y configurarlo. Sin embargo, tras bastante tiempo intentando hacer funcionar el benchmark seguían apareciendo errores en el servidor de aplicaciones, por tanto, se optó por empezar nuevamente y utilizar esta vez la versión basada en php, que aunque ofrece un rendimiento algo peor, es más fácil de configurar y se ha utilizado con anterioridad en la asignatura.

Para el caso de la versión basada en php ha sido necesario descargar apache2 con el modulo fast-cgi, un jdk 1.5 y php5. Inicialmente se probó con versiones más recientes de php (php7) y java (java 1.8), pero presentaban incompatibilidades con SPECweb2005, por tanto, se descartó la opción de usar las versiones actuales y se volvió a las indicadas en el manual.

Una vez descagardo todo se procedió a a configurar cada una de las piezas que componen el sitema: servidor web, wafgen, BeSim y cliente.

#### Servidor web

Lo primero que se hizo con el sevidor web fue modificar el fichero de configuración del sitio para establecer la raiz en el directorio /var/www en lugar de /var/www/html, que es el directorio por defecto. Además, se copiaron los ficheros del directorio scripts de SPECweb2005 al /var/www

```sh
#Añadir repositorios con php5.6

apt-get install apt-transport-https lsb-release ca-certificates
wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list

apt-get install libapache2-mod-php5.6  php5.6 curl

a2enmod php5.6
vi /etc/apache2/sites-enabled/000-default.conf # Cambiar DocRoot to “/var/www"

cp -r /tmp/SPECweb2005/scripts/php/* /var/www/
```

#### Wafgen

Wafgen es un generador de contenido estático para el benchmark. SPECweb2005 contiene 3 tests diferentes: Support, Banking y E-commerce. Cada uno de ellos necesita una serie de ficheros. Para generarlos hay que definir una serie de propiedades en los ficheros de configuracion wafgen para cada test y luego ejecutar Wafgen. Antes de ejecutar el generador es necesario añadir el directorio /bin del jdk a la variable PATH

```sh
#Ejemplo para el caso support
cd /tmp/SPECweb2005/wafgen
vi  unix/support_image_props.rc (cambiar DOCROOT=/var/www/)
vi unix/support_downloads_props.rc (cambiar DOCROOT=/var/www/ y seleccionar un numero de sesiones simultanes ej. SIMULTANEOUS_SESSIONS=100)

export PATH=/tmp/jdk1.5.0_22/bin/:$PATH

./Wafgen unix/support_image_props.rc  
./Wafgen unix/support_downloads_props.rc 

```

#### BeSim
Para el Backend Simulator es necesario activar el modulo fast-cgi en apache y modificar el fichero de configuración del sitio para añadir la opciones ExecCGI. Una vez hecho esto, se crea el directorio /var/www/besim y desde el directorio del besim que se encuenta dentro de SPECweb2005 se compila besim con las opciones apropiadas. Por último, se da permisos a todos los ficheros en /var/www para que se puedan leer y escribir y se lanza el test de BeSim para comprobar que funciona.

```sh
apt-get install libapache2-mod-fcgid  libfcgi-dev
a2enmod  fcgid
service apache2 restart
vi /etc/apache2/apache2.conf

mkdir /var/www/besim/
make fcgi TARGET='clean all install' DEST=/var/www/besim/

bash ./test_besim_support.sh http://127.0.0.1/besim/besim_fcgi.fcgi

chmod -R a+rw /var/www
bash ./test_besim_support.sh http://127.0.0.1/besim/besim_fcgi.fcgi
```
#### Cliente 

A la hora de ejecutar el cliente hay que fijar una serie de parámetros y escoger cual de los 3 test se va a ejecutar. Cada test tiene una serie de características. En este caso se van ejecutar solamente Support y Banking, aunque en el caso de Banking se va a lanzar sin ssl porque el cypher está obsoleto y habria que recompilar una versión antigua de openssl que lo soporte. Support se basa en paginas web estaticas y, por tanto, esta más afectado por operaciones de entrada salida, mientras que Banking pone una mayor carga sobre el procesador.

En ambos test, se seguirá el mismo patrón de ejecución para ver cuantas sesiones simultaneas es capaz de soportar el sistema. Se comenzará con 100 sesiones simultaneas y en función de los resultados se irá ajustando el numero de sesiones simultaneas hasta alcanzar el punto en el que comienzan a descartarse peticiones, estableciendo el limite en un 98% de casos correctos.

 

#### Test Support

**Primera ejecución**: 100 sesiones simultaneas -> 100%

**Segunda ejecución**: 200 sesiones simultáneas -> 1%

**Tercera ejecución**: 150 sesiones simultáneas -> 8%

**Cuarta ejecución** : 125 sesiones simultáneas -> 79%

**Quinta ejecución**: 120 sesiones simultaneas -> 90%

**Sexta ejecución**: 110 sesiones simultaneas -> 98%

A la vista de estos resultados podemos determinar que, en Test Support, el benchmark consigue una puntuación de 110.

#### Test Banking

**Primera ejecución**: 100 sesiones simultaneas -> 100% 

**Segunda ejecución**: 200 sesiones simultáneas -> 0,5%

**Tercera ejecución**: 150 sesiones simultáneas -> 5%

**Segunda ejecución**: 130 sesiones simultaneas -> 97%

**Tercera ejecución**: 125 sesiones simultaneas -> 99%

A la vista de estos resultados podemos determinar que, en Test Banking, el benchmark consigue una puntuación entre 125 y 130.

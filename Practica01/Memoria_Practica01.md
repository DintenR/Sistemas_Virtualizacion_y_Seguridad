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



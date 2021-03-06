# TEMA 3: GESTIÓN DE MEMORIA
## REPASO
### Elementos hardware

**▷** **MEMORIAS CACHÉ:**
memorias intermedias que mantienen instrucciones/datos
previamente accedidos y que son más rápidas que la RAM.

**▷** Para cachés, definimos:

 · **Acierto de cache:** la instrucción/dato buscado está en ella. 
 
 · **Fallo de caché:** la instrucción/dato no esta en la misma.
 
 - **Funcionamiento de una caché:** Las memorias cachés funcionan gracias a la localidad de las referencias del código, datos, etc, es decir, normalmente, si un item es referenciado, las direcciones próximas a el tienden a ser referenciadas, por lo tanto, **localidad espacial**, y si un item es referenciado, suele volver a usarse, por lo tanto, **localidad temporal**.

**▷TIEMPO DE ACCESO EFECTIVO(TAE)**: Coste de acceder a memoria.
**TAE = p·ta + (1-p)·tf**   donde: **p**=probabilidad de acierto; **ta**=tiempo de acceso si
hay acierto;  **1-p**= probabilidad de fallo, y **tf**=tiempo de
acceso si hay fallo.


**▷ESPACIOS LÓGICOS Y FÍSICOS**

· La necesidad de poder reubicar un programa en
memoria hace necesario separar el espacio de
direcciones generadas por el compilador, espacio
lógico o virtual, del espacio físico en el que se
carga, el espacio de direcciones físicas (en RAM).

· **Dirección lógica** - la generada por la CPU;
también conocida como virtual.

· **Dirección física** - dirección que se pasa al
controlador de memoria.

**▷ TRADUCCIÓN DE DIRECCIONES**

La separación de espacios me obliga a realizar una
traducción de direcciones:

FALTA IMAGEN

+ **UNIDAD DE GESTIÓN DE MEMORIA MMU (Memory Management Unit):**
 Dispositivo hardware que se encarga de: 
 1. Traducir direcciones virtuales en direcciones físicas.
 2. Implementar la protección.
 
La forma de la MMU dependerá del esquema de gestión
de memoria implementado en hardware. En el esquema
más simple, contendrá un registro de reubicación que
almacena el valor a sumar a cada dirección generada por
el proceso de usuario al mismo tiempo que es enviado a
memoria.

  - **Funcionamiento:**
           Cuando el procesador genera una dirección virtual, es realmente el compilador el que genera el espacio
           del proceso, por lo tanto, es el que asigna direcciones. Lo que lee la CPU del código binario son
           direcciones de memoria virtuales y es la MMU la que traduce esas direcciones virtuales en físicas y si no
           puede acceder a ella, genera una excepción.
           
    **Problema:**  Fragmentación: fracción de memoria que no es asignable debido al propio mecanismo de gestión de memoria.
     
   Los SOs actuales suelen utilizar paginación como esquema básico de gestión de memoria, si bien, dependiendo del procesador,              deben también utilizar segmentación. Por ejemplo, los procesadores Intel implementan segmentación como esquema básico de                gestión de memoria (protección:modos de funcionamiento del procesador) y opcionalmente se puede activar o no la paginación. Existen dos tipos de fragmentación:
1. **Fragmentación interna**: se da en los sistemas que usan un particionamiento fijo de la memoria. La memoria está dividia en bloques de igual tamaño. Un proceso se carga en un bloque, ocupándolo entero, por pequeño que sea el proceso. Por ejemplo, un programa de menos de 2Mb podría cargarse en un bloque de 8Mb, desperdiciando la memoria. Esto es lo que se conoce como **fragmentación interna**.
2. **Fragmentación externa**: en el particionamiento dinámico, a un proceso se le asigna en memoria principal exactamente lo que ocupa. Con el tiempo, van quedando huecos pequeños de memoria que no pueden aporvecharse ya que no cabe ningún proceso, fenómeno conocido como **fragmentación externa**. *Para solucionarlo, se utiliza la compactación.*
   
   - **Paginación**
   La MMU “divide” el programa en bloques del mismo tamaño, denominados páginas, para cargarlos en bloques de memoria principal del mismo tamaño, denominados marcos. El SO mantiene la pista de cuales son los marcos que contienen las páginas de un programa mediante una estructura de datos por proceso denominada tabla de páginas (TP). Esta estructura tiene una entrada de TP (PTE) por cada página del proceso, donde cada entrada indica cual es la dirección base de memoria principal del marco que la contiene. También contiene información de protección de la página.
   
Dos tipos:

Paginación simple: Se introduce una caché hardware en la MMU para acelerar la traducción, el TLB. 
Translation Lookaside Buffer (TLB) es una memoria caché administrada por la MMU, que contiene partes de la tabla de paginación, es decir, relaciones entre direcciones lógicas y físicas. Posee un número fijo de entradas y se utiliza para obtener la traducción rápida de direcciones. Si no existe una entrada buscada, se deberá revisar la tabla de paginación y tardará varios ciclos más, sobre todo si la página que contiene la dirección buscada no está en memoria principal. Si en la tabla de paginación no se encuentra la dirección buscada, saltará una interrupción conocida como fallo de página.

Paginación multinivel: Desarrollada en el tema.

  - **Segmentación**
  Troceamos el programa en unidades lógicas de programación (procedimientos, pilas, código, datos, tabla de símbolos, etc.)
denominadas segmentos.Cada segmento suele tener una tamaño diferente del resto. Ahora, una dirección lógica es una tupla:
<número_de_segmento, desplazamiento>. Cada programa tiene una Tabla de Segmentos dondecada entrada tiene los
siguientes elementos:

 base - dirección física donde reside el inicio del segmento en memoria.

 límite - longitud del segmento.
   
## 1. Paginación Multinivel

Justificación: Para una arquitectura de 32 bits, con un espacio de direccones de 4GB=2^32, necesitaremos tablas de página de 2^20 entradas, con 4B cada una, lo que nos dará lugar a una tabla de página (TP) de 4MB (¡GIGANTE!). 
Para solucionar este problema, se decidió paginar la tabla de páginas. Consiste en partir la zona en la que describo la tabla de página  en 2^n partes de k bits.

**Paginación a 2 niveles**

![](https://chsos20141911030.files.wordpress.com/2014/05/sin-tc3adtulo43.png?w=700)

- ¿Cómo funciona?

![](https://chsos20141911030.files.wordpress.com/2014/06/sin-tc3adtulo5.png?w=700)
1) Se busca en la tabla de primer nivel (N1) la "dir" que apunta a la tabla de segundo nivel.
2) Se carga la taba de página de segundo nvel (N2). 
3) Se busca en la tabla (N2) la página de N2.
4) Se carga la página y con el offset encontramos la dirección física buscada.

Para calcular las entradas de las tablas de primer y segundo nivel y el offset de la posición de memoria indicada por una dirección lógica se sigue el siguiente procedimiento:
1) Dividimos la dirección lógica entre el número de entradas de cada tabla de segundo nivel multiplicado por el tamaño de página. 
2) El cociente de esa división es el índice dentro de la tabla de primer nivel al que corresponde la dirección (dicha posición en la tabla de primer nivel redirecciona a la tabla de segundo nivel correspondiente).
3) Dividimos el resto de la división anterior entre el tamaño de página. 
4) El cociente es el índice de la tabla de segundo nivel correspondiente a la dirección.
5) El resto de esta división es la posición dentro de la página a la que señala la dirección.

- Ventajas:

 1.Las tablas de segundo nivel no tienen porqué estar en memoria, pueden estar en disco e ir construyéndose mientras vayan haciendo falta. Por ejemplo, si yo tengo todo el código de tratamiento de código en disco y en la ejecución de mi programa no se produce nungún error, no será necesario crear las páginas de errores. 
 
 2. Los espacios reales de los procesos son dispersos(tienen muchos huecos que no se usan), entonces para aquellas páginas que sean huecos, no necesito construirlas, es decir, siempre construyo las páginas de primer nivel pero puedo evitar construir aquellas páginas de segundo nivel que no referencien ninguna información.
 
 3. Los TLB mantienen un TAE razonable. 

## Concepto previo: Unidad de Gestión de Memoria (no sé donde meterlo exactamente)

La **MMU (Memory Management Unit)** es un dispositivo hardware que traduce direcciones virtuales a direcciones físicas. Este dispositivo está gestionado por el SO.

En el esquema MMU más simple, el valor del registro base se añade a cada dirección generada por el proceso de usuario al mismo tiempo que es enviado a memoria.

El programa de usuario trata solo con direcciones lógicas, nunca con direcciones reales.

Además de la traducción, el MMU deberá detectar si la dirección aludida se encuentra o no en memoria principal y generar una excepción si no se encuentra. La MMU cuenta con sus propios registros. 

   
## 3. Gestión de Memoria en Linux

En un sistema monopogramado, la memoria se divide en dos partes: una parte para el sistema operativo (monitor residente, núcleo) y una parte para el programa actualmente en ejecución. En un sistema multiprogramado, la parte "usuario" de la memoria se debe subdividir posteriormente para acomodar múltiples procesos. El sistema operativo es el encargado de la tarea de subdivisión y a esta tarea se le denomina **gestión de memoria**, vital en un sistema multiprogramado para el buen aporvechamiento de la CPU y ejecucuión de los procesos. 

### Niveles de Gestión de Memoria.

Existen dos niveles con requisitos diferentes:

* **Gestor de memoria de SO:**
	
	· Asigna porciones de memoria a los procesos (es
	no crítica).
	
	· Asigna memoria a los subsistemas del SO (crítica).

* **Gestor de memoria de procesos:** gestión dinámica 
de los procesos (malloc, free, ...)

### Gestor de memoria en Linux

Tiene dos interfaces:

· Interfaz de llamadas al sistema: interfaz de usuario.
	
· Interfaz intra-kernel: interfaz de mm al resto del kernel.

### Elementos de Gestión.

La implementación Linux de gestión de memoria cubre:

* Memoria kernel:
	1. Distribuidor sistema amigo para asignar grandes bloques contiguos 
	de memoria.

	2. Distribuidores tableta(slab, slob, slub¹), para asignar memoria 
	inferior a una página.
			
	3.Mecanismo `vmalloc()` para asignar bloques no contiguos de memoria.

*¹Son los tres distribuidores de memoria (memory allocators) existentes en el kernel de Linux. Cada uno de ellos se usa con un objetivo concreto a la hora de optimizar la memoria principal*
	
* Memoria de usuario:
			
	1. Mecanismos de construcción y gestión de los espacios de direcciones 
	de los procesos.

### Gestión de memoria para procesos.

La asignación de memoria dinámica a los procesos de usuario tiene requisitos 
diferentes que a los del kernel:

* Las peticiones de procesos no se consideran urgentes: el kernel intenta 
diferir su asignación, ya que la solicitud no indica utilización inmediata.
	
* Los procesos de usuario no son confiables: el kernel debe estar preparado 
para atrapar errores de direccionamiento.
	
* Cada proceso tiene su propio espacio de direcciones separado del resto de 
procesos.
	
* El kernel puede anadir/suprimir rangos de direcciones lineales.
	
### Regiones de memoria	
	
El kernel representa un intervalo de direcciones lineales contiguas con el mismo
tipo de protección mediante un recurso denominado región de memoria.

Estas se caracterizan por su dirección de inicio, su longitud y los derechos de 
acceso. Por eficiencia, estas tienen un tamaño múltiplo del tamaño de página.

EJEMPLO:

> * región de código: permisos lectura-ejecución.
> * región de datos : permisos de lectura-escritura.
> * región de pila  : lectura-escritura, crecimiento.

Las tablas de páginas NO son adecuadas para representar espacios de direcciones
grandes, especialmente si son dispersos, que entonces se superpone otra gestión 
de memoria sobre la paginación.

Linux representa cada región de memoria con una estructura denominada vm-área.
Las vm-área contienen información necesaria para poder establecer la traducción
de una dirección que la TP no pueda realizar.

El ED de un proceso se representa como una lista de vm-áreas. Si el número de
vm-áreas se organizan en un árbol rojo-negro.

Las vm-áreas no tienen contador de referencias, por lo que solo pueden pertenecer 
a un proceso
	
	   


## Linux y VM-areas

Linux representa cada región de memoria con una estructura denominada
`vm-area` (virtual memory area). Contienen la información para
realizar la traducción de una dirección que la TP no pueda
realizar. El espacio de direcciones de un proceso se representa como
una lista de vm-areas. Si es muy elevado se representa en un arbol
rojo-negro. **NO** tienen contador de referencias (solo pueden pertenecer a un proceso).

Cada vm-area viene definida por un struct. Sus componentes son:

* Un rango de dirección determinado por una dirección de inicio y otra de fin.
* Indicadores VM. Palabra que contiene entre otros los permisos para
  un proceso (`VM_READ`, `VM_WRITE`, `VM_EXEC`)
* Información de enlace: Punteros al subarbol derecho e izquierdo de la lista de vm-areas.
* Información de archivo proyectado: Si el vm-area proyecta un
  archivo, este componente almacena el puntero a archivo y el
  desplazamiento necesario para localizar los datos.
* Operaciones VM y datos privados. Contiene:
  
  - Datos privados.
  - Puntero a operaciones VM: apunta a funciones. Proporciona a las
    vm-areas característias de orientación a objetos. Cada vm-area
    puede tener distintos manejadores.
  - Cualquier objeto (sistema de archivos, dispositivos de caracteres,
    ...) proyectado en memoria de usuario (`mmap`) puede suministrar
    sus propias operaciones. Algunas son: `open() / close()` para
    crear y destruir un vm-area respectivamente y `fault()`manejador
    de falta de página que no está en la TP.
	
![Espacio virtual en x86](images/espacio_virtual.png)


### Llamadas relacionadas con regiones

Las llamadas relacionadas con la creación, destrucción o modificación de regiones de memoria son:

* `brk()` - Cambia el tamaño del heap de un proceso.
* `exec()` - Carga un nuevo programa ejecutable.
* `exit()` - Termina el proceso actual.
* `fork()` - Crea un nuevo proceso.
* `mmap()` - Crea una proyección de memoria para un archivo.
* `munmap()` - Destruye una proyección de memoria para un archivo.
* `shmat()` - Crea una región de memoria compartida.
* `shmdt()` - Destruye una región de memoria compartida.

El descriptor de mm_struct es

```c
struct mm_struct {
	struct vm_area_struct *mmap; //Lista de áreas de memoria (VMAs)
	struct rb_root mm_rb //Árbol red-black de VMAs, para buscar un elemento concreto
	struct list_head mmlist; //Lista con todas las mm_struct: espacio de direcciones
	atomic_t mm_users //Número de procesos utilizando este espacio de direcciones
	atomic_t mm_count /*Contador que se activa con la primera referencia al espacio 
	de direcciones y se desactiva cuando mm_users vale 0*/
	
	//Límites de las secciones principales
	unsigned long start_code; //Primera dirección de código
	unsigned long end_code; //Última dirección de código
	unsigned long start_data; //Primera dirección de datos
	unsigned long end_data; //Última dirección de datos
	unsigned long start_brk; //Primera dirección de heap
	unsigned long brk; //Última dirección de heap
	unsigned long start_stack; //Primera dirección de la pila
	unsigned long arg_start; //Principio de los argumentos
	unsigned long arg_end; //Final de los argumentos
	unsigned long env_start; //Principio del ámbito del proceso
	unsigned long env_end; //Final del ámbito de proceso
	
	//Información relacionada con las páginas
	pgd_t *pgd //Directorio global de páginas
	unsigned long rss; //Páginas contenidas
	unsigned long total_vm; //Número de páginas totales
}
```
![ejemplo de vm-area](images/vm_area.png)

#### Explicación del ejemplo anterior

Suponemos que un proceso proyecta los primeros 32KB (8 páginas) de un archivo en la dirección virtual 0x2000

Los pasos del dibujo anterior:
+ Intenta leer la dirección 0x6008. Linux localiza la vm-área que cubre la dirección que provoca la falta (página 3).
+ Linux inicia la transferencia de la página 3.
+ Linux copia el contenido del archivo en el marco adecuado y actualiza la tabla de páginas. El proceso reanuda su ejecución. 

![Descriptor ED](images/Desc_ED.PNG)


![gestion falta de pagina](images/Gestion_falt.PNG)

### Mecanismo COW

El mecanismo COW (copy-on-write) permite reducir:
+ Las operaciones de copia en memoria.
+ El número de copias de marcos.
+ El mecanismo permite que varios procesos compartan un
marco de memoria. Si alguno de los procesos intenta escribir
en la página compartida COW, el kernel asigna una nueva
copia de la página al proceso escritor y realiza la escritura.

> Implementación: La página compartida COW se marca
como de solo lectura pero la vm-área correspondiente
permite la escritura.


![ejemplo COW](images/Ejemplo%20COW.PNG)

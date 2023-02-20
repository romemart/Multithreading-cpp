## Multithreading en C++
Un proceso es un programa que se ejecuta en la PC. En las PC's de hoy en día, muchos procesos se ejecutan al mismo tiempo. Un programa se puede dividir en subprocesos para que los subprocesos se ejecuten al mismo tiempo. Estos subprocesos se denominan Threads. Los Threads deben ejecutarse como partes de un programa. Algunos programas requieren más de una entrada simultáneamente. Tal programa necesita Threads. Si los Threads se ejecutan en paralelo, la velocidad general del programa aumenta. Los Threads también comparten datos entre ellos. Este intercambio de datos genera conflictos sobre qué resultado es válido y cuándo es válido. Este conflicto es una carrera de datos y se puede resolver. 

Dado que los Threads tienen similitudes con los procesos, el compilador g ++ compila un programa de Threads de la siguiente manera:
```cpp
g++ -std=c++17 main.cpp -lpthread -o main
```
Donde <font color="red">main.cpp</font> es archivo de codigo fuente, y <font color="red">main</font> es el archivo ejecutable y compilado bajo el estandar <span style="color:green;font-size:12px">(C++17)</span>.

Un programa que usa hilos, se inicia de la siguiente manera:
```cpp
#include <iostream>
#include <thread>
using namespace std;
```
Note el uso de `#include <thread>`.

Este artículo explica los conceptos básicos de Multi-thread y carreras de datos en C++. El lector debe tener conocimientos básicos de C++, su Programación Orientada a Objetos y su función lambda; para apreciar el resto de este artículo.

#### Tabla de contenidos

- [Librearia Thread](#1).
- [Objeto Miembro de Thread](#2)
- [Thread Retornando un Valor](#3)
- [Comunicación entre Threads](#4)
- [Especificador thread_local](#5)
- [Secuencias, Sincrónico, Asincrónico, Paralelo, Concurrente, Orden](#6)
- [Bloqueando un Thread](#7)
- [Concepto de Locking](#8)
- [Libreria Mutex](#9)
- [Timeout en C++](#10)
- [Requerimientos para bloquear](#11)
- [Tipos de Mutex](#12)
- [Carrera de datos](#13)
- [Acerca de Locks](#14)
- [Funcion Call Once](#15)
- [Libreria Condition Variable](#16)
- [Libreria Future](#17)
- [Conclusión](#18)

<div id='1'/>

#### Librearia Thread
El flujo de control de un programa puede ser único o múltiple. Cuando es único, es un Thread de ejecución o simplemente, Thread. Un programa simple es un Thread. Este Thread tiene la función main() como su función de nivel superior. Este Thread se puede llamar el Thread principal. En términos simples, un Thread es una función de nivel superior, con posibles llamadas a otras funciones.
Cualquier función definida en el ámbito global es una función de nivel superior. Un programa tiene la función main() y puede tener otras funciones de nivel superior. Cada una de estas funciones de nivel superior se puede convertir en un Thread encapsulándolas en un objeto de Thread. Un objeto Thread es un código que convierte una función en un Thread y gestiona el Thread. Se crea una instancia de un objeto de Thread desde la clase de Thread.
Entonces, para crear un Thread, ya debería existir una función de nivel superior. Esta función es el Thread efectivo. Luego se crea una instancia de un objeto Thread. El ID del objeto de Thread sin la función encapsulada es diferente del ID del objeto de Thread con la función encapsulada. El ID también es un objeto instanciado, aunque se puede obtener su valor de cadena.
Si se necesita un segundo subproceso más allá del subproceso principal, se debe definir una función de nivel superior. Si se necesita un tercer Thread, se debe definir otra función de nivel superior para eso, y así sucesivamente.

<div id='2'/>

#### Objeto Miembro de Thread
El thread principal ya está allí y no es necesario volver a crearlo. Para crear otro thread, su función de nivel superior ya debería existir. Si la función de nivel superior aún no existe, debe definirse. A continuación, se crea una instancia de un objeto thread, con o sin la función. La función es el thread efectivo (o el thread efectivo de ejecución). El siguiente código crea un objeto thread con una thrdFn (con una función):
```cpp
#include <iostream>
#include <thread>
using namespace std;

void thrdFn() {
    cout << "seen" << '\n';
}  

int main(int argc, char const *argv[])
{
    thread thr(&thrdFn);

    return 0;
}
```
El nombre del thread es <font color="red">thr</font>, instanciado desde la clase thread, thread. Recuerde: para compilar y ejecutar un thread, use un comando similar al anterior.

El constructor de la clase thread toma una referencia de la función como argumento.

Este programa ahora tiene dos threads: el thread principal y el thread del objeto thr. La salida de este programa debe ser: <font color="red">seen</font> (imprimiendo desde la función thrdFn). Este programa tal como está no tiene ningún error de sintaxis; está bien tipeado. Este programa, tal como es, se compila con éxito. Sin embargo, si se ejecuta este programa, es posible que el thread (función, thrdFn) no muestre ningún resultado; es posible que se muestre un mensaje de error.
Esto es porque el thread, <font color="red">thrdFn()</font> y el <font color="red">main()</font>
<div id='3'/>

#### Thread Retornando un Valor

<div id='4'/>

#### Comunicación entre Threads

<div id='5'/>

#### Especificador thread_local

<div id='6'/>

#### Secuencias, Sincrónico, Asincrónico, Paralelo, Concurrente, Orden

<div id='7'/>

#### Bloqueando un Thread

<div id='8'/>

#### Concepto de Locking

<div id='9'/>

#### Libreria Mutex

<div id='10'/>

#### Timeout en C++

<div id='11'/>

#### Requerimientos para bloquear

<div id='12'/>

#### Tipos de Mutex

<div id='13'/>

#### Carrera de datos

<div id='14'/>

#### Acerca de Locks

<div id='15'/>

#### Funcion Call Once

<div id='16'/>

#### Libreria Condition Variable

<div id='17'/>

#### Libreria Future

<div id='18'/>

#### Conclusión

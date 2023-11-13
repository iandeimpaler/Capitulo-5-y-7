# Capitulo-5-y-7

# Capítulo 5: Hash, diseño consistente.

Para obtener un escalamiento horizontal es importante distribuir las solicitudes de información de manera eficiente.  El proceso de hashing es comúnmente usado para obtener resultados alineados con este propósito,  veamos de manera profunda el problema.


## El problema de rehashing

Si tienes n  servidores caché,  una manera común de equilibrar la carga es usar el siguiente método:

serverIndex = hash(key) % N donde N es el tamaño del recipiente del servidor.
Un ejemplo para ver como funciona. En la tabla 5-1 mostramos 4 servidores y 8 llaves de texto con su correspondiente hash.

<img width="309" alt="image" src="https://github.com/iandeimpaler/Capitulo-5-y-7/assets/60924631/f39f2746-6a90-4fb4-aafd-952b3d8343e2">



Procedemos con una operación modular f(key) % 4. De esta manera  hash(key0) % 4 = 1 Significa que el cliente debe contactar al servidor número uno para buscar la data en caché.
La figura 5-1 muestra la distribución de llaves basado en la tabla 5-1.

<img width="275" alt="image" src="https://github.com/iandeimpaler/Capitulo-5-y-7/assets/60924631/ac9cb03f-13ab-4f9e-b19c-4943b194d6f1">


Esta Aproximación funciona bien cuando el tamaño del recipiente del servidor es inamovible y la distribución de la información es homogénea.  de todas maneras el problema surge cuando nuevos servidores son añadidos o cuando servidores existentes son removidos.  Por ejemplo si el servidor 1 entra en un estado offline el tamaño del recipiente del servidor se transforma en tres.  utilizando la misma función Hash obtenemos el valor Hash para una llave.  pero aplicando operaciones modulares nos dan diferentes índices del servidor porque el nombre del servidor ha sido reducido en una unidad. Obtenemos resultados como los mostrados en la tabla 5-2, aplicando hash%3:


<img width="272" alt="image" src="https://github.com/iandeimpaler/Capitulo-5-y-7/assets/60924631/965af29e-7969-40dc-bb24-9e05247a6158">



La figura 5-2 muestra la nueva distribución de llaves basado en la tabla 5-2.

<img width="296" alt="image" src="https://github.com/iandeimpaler/Capitulo-5-y-7/assets/60924631/07c3f14e-1719-4ea5-9a6e-b0f4818009eb">


Como se muestra en la figura 5-2, La mayoría de las llaves han sido redistribuidas, no solo las que estaban originalmente almacenadas en el servidor offline (servidor 1).  Esto significa que cuando el servidor 1 está offline la mayoría del caché del cliente será conectada con servidores incorrectos para buscar la información.  Esto causa una tormenta de intentos de caché errados.  un hashing consistente es una forma efectiva de mitigar el problema.

## Hashing consistente
El hashing consistente es una manera especial de hashing la cual permite que cuando una tabla con valores Hash está siendo redimensionada y a la vez esta tiene un hashing consistente en su implementación solo las llaves K/N  tienen la necesidad de ser remapeadas,  donde K es el número de llaves,  y n es el número de espacios.  por el contrario en las tablas tradicionales de Hash el cambio en un número dentro del array provoca que casi todas las llaves se han remapeadas.

## Hash space y hash ring
Ahora que entendemos la definición del haching consistente nos permitiremos descubrir cómo funciona. Asumiendo SHA-1 es usado en la funcion hash F, y la salida del rango de la funcion hash es:  x0, x1, x2, x3, …, xn. En criptografía SHA-1 hash, va de 0 a 2^160-1. Eso significa que X0 corresponde a 0, XN a 2^160-1, y todos los demas valores hash en la escala del medio entre 0 y 2^160-1. La figura 5-3 muestra el hash space. 

<img width="314" alt="image" src="https://github.com/iandeimpaler/Capitulo-5-y-7/assets/60924631/e42a6f89-6ae7-48cf-8b8c-d3750b682a54">


## Servidores Hash

Usando la misma función f, cuando mapeamos el servidor basado en IP o nombre dentro del anillo.La figura 5-5 muestra que 4 servidores están mapeados en el hash ring o anillo.

<img width="299" alt="image" src="https://github.com/iandeimpaler/Capitulo-5-y-7/assets/60924631/90685459-a2f6-4bb9-9567-377b10d6115a">


## Llaves Hash

Otra cosa que vale la pena mencionar Es que la función Hash está siendo usada de diferente manera con respecto a la función Hash en el problema del rehashing problem, Y no existe ninguna operación modular. Como se muestra en la figura 5-6, 4 llaves caché ((key0, key1, key2, and key3) están posicionadas en el anillo. 

<img width="308" alt="image" src="https://github.com/iandeimpaler/Capitulo-5-y-7/assets/60924631/ed3fd9f7-4865-4d1e-9bef-88c498442bc6">


## Bloqueo de servidor

Para determinar qué servidor tiene una llave guardada debemos ir a favor del sentido del reloj desde la posición de la llave en el anillo Hasta que el servidor es encontrado. Figura 5-7 explica este proceso. Vamos en sentido horario desde la llave cero guardada en el servidor 0,  llave 1 guardada en el servidor 1, llave 2 guardada en el servidor 2 y llave 3 guardada en el servidor 3 

<img width="289" alt="image" src="https://github.com/iandeimpaler/Capitulo-5-y-7/assets/60924631/49127be4-c7db-4183-8879-6cc64bea7e09">


## Añadir un servidor

Usando la lógica descrita arriba Añadir un servidor solo requerirá la redistribución de una fracción de las llaves.

 en la figura 5-8, después de un nuevo servidor cuarto añadido,  solo la llave cero necesita ser redistribuida.  k1, k2 y k3 se mantienen en el mismo servidor.  Antes de que el servidor 4 sea añadido la llave cero es guardada en el servidor 0.  Ahora la llave cero será guardada en el servidor 4 debido a que el servidor 4 es el primer servidor que se encontrará si vamos en sentido horario desde la posición de la llave 0 en el anillo.  Las otras llaves no han sido redistribuidas dada  la consistencia del algoritmo Hash.
 

<img width="302" alt="image" src="https://github.com/iandeimpaler/Capitulo-5-y-7/assets/60924631/d6ed7ac3-82d1-45b4-acca-64d62fd7b43a">



##  Remover un servidor

Para remover un servidor solo necesitamos distribuir una porción pequeña de las llaves.
En la figura 5-9, cuando el servidor 1 es removido solo la llave 1 debe ser reasignada al servidor 2, las demás llaves no son afectadas.

<img width="305" alt="image" src="https://github.com/iandeimpaler/Capitulo-5-y-7/assets/60924631/92bb03cd-1d57-41b0-b5c2-a02202e11d72">


## Dos problemas en el acercamiento básico

La consistencia del algoritmo Hash fue introducida por Karger et al. En el MIT
 los pasos básicos son los siguientes:
- mapear el servidor y las llaves en el anillo usando de manera uniforme la distribución de la función Hash.
- encontrar qué servidor está mapeado aquella vez para ir en sentido horario de la posición de la llave Hasta que el primer servidor en el anillo sea encontrado.

Los dos problemas identificados con este aproximamiento.  El primero si es imposible mantener el mismo tamaño de particiones dentro del anillo para todos los servidores considerando que el servidor puede ser añadido o removido.  Una partición en el espacio Hash entre los servidores adyacentes. Es posible que el tamaño de las particiones en el anillo asignadas a cada servidor sea demasiado pequeña o apenas lo suficientemente grande. En la figura 5-10,  si s1 uno es removido la partición de s2 sera el doble de larga que la partición en s0 y s3.

<img width="303" alt="image" src="https://github.com/iandeimpaler/Capitulo-5-y-7/assets/60924631/4c162a0d-1574-4aa0-9ba8-ee6940c926fb">


Segundo,  es posible no tener una distribución no uniforme en el anillo,  por lo tanto si los servidores han sido posicionados dentro del mapa en la siguiente lista como muestra la figura 5-11,  la mayoría de las llaves serán almacenadas en el servidor número 2.  de todas maneras el servidor 1 y el servidor 3 no tienen información. 

<img width="307" alt="image" src="https://github.com/iandeimpaler/Capitulo-5-y-7/assets/60924631/88e5c048-58f0-4937-bb8d-4402cbdfaba3">


## Nodos virtuales

Un nodo virtual se refiere al nodo real y cada servidor es representado por múltiples nodos virtuales dentro del anillo.  en la figura 5 - 12, Ambos servidor cero y servidor uno tienen tres nodos virtuales.  El número 3 se elige arbitrariamente; y en sistemas del mundo real, el número de nodos virtuales es mucho mayor. En lugar de utilizar s0, tenemos s0_0, s0_1 y s0_2 para representar el servidor 0 en el anillo. De manera similar, s1_0, s1_1 y s1_2 representan al servidor 1 en el anillo. Con nodos virtuales, cada servidor es responsable de múltiples particiones. Las particiones (bordes) con la etiqueta s0 son gestionadas por el servidor 0. Por otro lado, las particiones con la etiqueta s1 son gestionadas por el servidor 1.


<img width="314" alt="image" src="https://github.com/iandeimpaler/Capitulo-5-y-7/assets/60924631/54296055-c56f-401f-b0a4-1fa2982144b9">




Para encontrar en qué servidor está guardada una llave debemos recorrer en sentido horario desde la posición de la llave y encontrar el primer nodo virtual dentro del anillo.  en la figura 5 - 13, Para encontrar en qué servidor k0 está guardado vamos en sentido horario desde la posición de casero hasta encontrar el primer nodo virtual s1-1, que se refiere al servidor 1. 

<img width="325" alt="image" src="https://github.com/iandeimpaler/Capitulo-5-y-7/assets/60924631/ae9d9213-806a-4c6b-873b-3767a995759e">


Mientras el número de nodos virtuales aumenta,  la distribución de las llaves se torna más balanceada.  esto es debido a que la desviación estándar tiende a ser más pequeña Mientras más nodos virtuales existan,  apuntar a una distribución balanceada.  el estándar de desviación mide como la información es esparcida.  el resultado de un experimento llevado por una  un búsqueda virtual muestra que con uno o 200 nodos virtuales la desviación estándar es del 5% 200 nodos virtuales y 10% 100 nodos virtuales de todo el producto. La desviación estándar será menor cuando incrementemos el número de nodos virtuales.  esto llegaría a ser una transacción la cual podemos modificar en base al número de nodos virtuales que sea adecuado para el requerimiento de nuestro sistema.

## Find qué has dicho affected keys

 cuando un servidor es añadido o removido una fracción de la información necesita ser redistribuida. 
 en la figura 5 - 14, Servidor 4 es añadido dentro del anillo.  el rango afectado empieza en el servidor 4 y se mueve antihorario alrededor del anillo Hasta que el servidor 3 colapsa junto con el servidor 4 de esta manera las llaves localizadas entre los dos servidores necesitan ser redistribuidas al punto de colisión en el servidor 4. 

<img width="315" alt="image" src="https://github.com/iandeimpaler/Capitulo-5-y-7/assets/60924631/9f48350b-fa0a-4e2a-8843-a2d3c191fe90">


Cuando un servidor s1 es removido,  el rango afectado empieza desde el servidor 1 y se mueve en sentido antihorario alrededor del anillo Hasta que encuentre el servidor 0 de la misma manera pero al opuesto las llaves entre el servidor 0 y el uno deben ser redistribuidos en el servidor 2.



##  Material de referencia 

[1] Consistent hashing: https://en.wikipedia.org/wiki/Consistent_hashing [2] Consistent Hashing: https://tom-e-white.com/2007/11/consistent-hashing.html [3] Dynamo: Amazon’s Highly Available Key-value Store: https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf [4] Cassandra - A Decentralized Structured Storage System: http://www.cs.cornell.edu/Projects/ladis2009/papers/Lakshmanladis2009.PDF [5] How Discord Scaled Elixir to 5,000,000 Concurrent Users: https://blog.discord.com/scaling-elixir-f9b8e1e7c29b [6] CS168: The Modern Algorithmic Toolbox Lecture #1: Introduction and Consistent Hashing: http://theory.stanford.edu/~tim/s16/l/l1.pdf [7] Maglev: A Fast and Reliable Software Network Load Balancer: https://static.googleusercontent.com/media/research.google.com/en//pubs/ar chive/44824.pdf 


# Capítulo 7: diseña un generador de ID único en sistemas distribuidos

En este capítulo se te solicita diseñar un generador de ID único dentro de los sistemas distribuidos.  tu primer pensamiento será el de probablemente utilizar una llave primaria con el atributo de auto incrementación como en una base de datos tradicional.  Pero de hecho el auto incremento no funciona en un ambiente distribuido porque una base de datos única no es suficiente y generar ideas únicas a lo largo de varias bases de datos con un delay mínimo es desafiante. 

## Paso 1 - Entender el problema y establecer un espacio de diseño 

Preguntar acerca de la incertidumbre en la clarificación es el primer paso para aproximarnos al diseño de cualquier sistema,  en especial si estamos dentro de una entrevista. 

Candidato: ¿Cuáles son las características de los IDs únicos?
Entrevistador: Los IDs deben ser únicos y ordenables.
Candidato: ¿Para cada nuevo registro, el ID se incrementa en 1?
Entrevistador: El ID se incrementa con el tiempo, pero no necesariamente solo se incrementa en 1. Los IDs creados por la noche son mayores que los creados por la mañana en el mismo día.
Candidato: ¿Los IDs solo contienen valores numéricos?
Entrevistador: Sí, eso es correcto.
Candidato: ¿Cuál es el requisito de longitud del ID?
Entrevistador: Los IDs deben ajustarse a 64 bits.
Candidato: ¿Cuál es la escala del sistema?
Entrevistador: El sistema debería ser capaz de generar 10,000 IDs por segundo.
Las preguntas anteriores son ejemplos que puedes hacerle a tu entrevistador.
Es importante entender los requisitos y aclarar cualquier ambigüedad. Para esta pregunta de la entrevista, los requisitos se enumeran de la siguiente manera:
• Los IDs deben ser únicos.
• Los IDs son solo valores numéricos.
• Los IDs se ajustan a 64 bits.
• Los IDs están ordenados por fecha.
• Capacidad para generar más de 10,000 IDs únicos por segundo.

## Paso 2 - Proponer un diseño de alto nivel y alta demanda

Se pueden utilizar múltiples opciones para generar IDs únicos en sistemas distribuidos. Las opciones que consideramos son:
• Replicación multi-maestro
• Identificador único universal (UUID)
• Servidor de tickets
• Enfoque Twitter snowflake
Veamos cada una de ellas, cómo funcionan y las ventajas/desventajas de cada opción.




## Replicacion Multi-Master

Este enfoque utiliza la función auto_increment de las bases de datos. En lugar de aumentar el próximo ID en 1, lo incrementamos en k, donde k es el número de servidores de bases de datos en uso. Como se ilustra en la Figura 7-2, el próximo ID a generar es igual al ID anterior en el mismo servidor más 2. Esto resuelve algunos problemas de escalabilidad porque los IDs pueden adaptarse al número de servidores de bases de datos. Sin embargo, esta estrategia tiene algunas desventajas importantes:
• Difícil de escalar con múltiples centros de datos.
• Los IDs no aumentan con el tiempo en varios servidores.
• No escala bien cuando se agrega o elimina un servidor.


## UUID

Un UUID es otra manera fácil de obtener IDs únicos. Un UUID es un número de 128 bits utilizado para identificar información en sistemas informáticos. Un UUID tiene una probabilidad muy baja de colisión. Citando a Wikipedia, "después de generar 1 mil millones de UUIDs cada segundo durante aproximadamente 100 años, la probabilidad de crear un duplicado alcanzaría el 50%" [1].

Ventajas: 
Generar un UUID es sencillo. No se necesita coordinación entre los servidores entonces no habrá ningún problema de sincronización.
El sistema es más fácil de escalar debido a que cada servidor web es responsable de generar las ids que consuman. El generador Id puede escalar fácilmente con servidores web.
Desventajas:
Los IDs tienen una longitud de 128 bits, pero nuestro requisito es de 64 bits.
Los IDs no aumentan con el tiempo.
Los IDs podrían no ser numéricos.


## Servidor Ticket


Los servidores de tickets son otra forma interesante de generar IDs únicos. Flicker desarrolló servidores de tickets para generar claves primarias distribuidas [2]. Vale la pena mencionar cómo funciona el sistema.

La idea es utilizar una función de autoincremento centralizada en un único servidor de base de datos (Servidor de Tickets). Para obtener más información al respecto, consulta el artículo del blog de ingeniería de Flicker [2].

Ventajas:
 IDs numéricos.
 Fácil de implementar y funciona para aplicaciones de pequeña a mediana escala.

Desventajas:
Punto único de fallo. Un único servidor de tickets significa que si el servidor de tickets falla, todos los sistemas que dependen de él enfrentarán problemas. Para evitar un punto único de fallo, podemos configurar varios servidores de tickets. Sin embargo, esto introducirá nuevos desafíos, como la sincronización de datos.

## Aproximación Twitter snowflake(copo de nieve)

Las aproximaciones mencionadas anteriormente nos proporcionan algunas ideas sobre cómo funcionan diferentes sistemas de generación de IDs. Sin embargo, ninguna de ellas cumple con nuestros requisitos específicos; por lo tanto, necesitamos otro enfoque. El sistema de generación de IDs único de Twitter llamado "snowflake" [3] es inspirador y puede satisfacer nuestros requisitos.

La estrategia de "dividir y conquistar" es nuestra aliada. En lugar de generar un ID directamente, dividimos un ID en diferentes secciones.

Cada sección se explica a continuación.
 Bit de signo: 1 bit. Siempre será 0. Esto está reservado para usos futuros y potencialmente se puede utilizar para distinguir entre números con signo y sin signo.
Marca de tiempo: 41 bits. Milisegundos desde la época o época personalizada. Utilizamos la época predeterminada de Twitter snowflake 1288834974657, equivalente al 04 de noviembre de 2010, 01:42:54 UTC.
 ID del centro de datos: 5 bits, lo que nos da 2 ^ 5 = 32 centros de datos.
 ID de máquina: 5 bits, lo que nos da 2 ^ 5 = 32 máquinas por centro de datos.
Número de secuencia: 12 bits. Por cada ID generado en esa máquina/proceso, el número de secuencia se incrementa en 1. El número se reinicia a 0 cada milisegundo.

## Paso 3 - Diseño vista profunda

En el diseño de alto nivel, discutimos varias opciones para diseñar un generador de IDs único en sistemas distribuidos. Optamos por un enfoque basado en el generador de IDs Twitter snowflake. Profundicemos en el diseño. Para refrescar nuestra memoria, el diagrama de diseño se vuelve a listar a continuación.

Los IDs de los centros de datos y las máquinas se eligen en el momento de inicio, generalmente fijos una vez que el sistema está en funcionamiento. Cualquier cambio en los IDs de centros de datos y máquinas requiere una revisión cuidadosa, ya que un cambio accidental en esos valores puede provocar conflictos de IDs. Las marcas de tiempo y los números de secuencia se generan cuando el generador de IDs está en funcionamiento.

Marca de tiempo: Los 41 bits más importantes componen la sección de la marca de tiempo. A medida que las marcas de tiempo crecen con el tiempo, los IDs son ordenables por tiempo. La Figura 7-7 muestra un ejemplo de cómo se convierte la representación binaria a UTC. También puedes convertir UTC de nuevo a la representación binaria utilizando un método similar.

La marca de tiempo máxima que se puede representar en 41 bits es 2^41 - 1 = 2199023255551 milisegundos (ms), lo que nos da aproximadamente 69 años (2199023255551 ms / 1000 segundos / 365 días / 24 horas / 3600 segundos). Esto significa que el generador de IDs funcionará durante 69 años y tener una época personalizada cercana a la fecha actual retrasa el tiempo de desbordamiento. Después de 69 años, necesitaremos una nueva época o adoptar otras técnicas para migrar IDs.

Número de secuencia: El número de secuencia es de 12 bits, lo que nos da 2^12 = 4096 combinaciones. Este campo es 0 a menos que se generen más de un ID en un milisegundo en el mismo servidor. En teoría, una máquina puede admitir un máximo de 4096 nuevos IDs por milisegundo.

## Paso 4 - Envolver

En este capítulo, hablamos sobre diferentes formas de diseñar un generador de IDs único: replicación multi-maestro, UUID, servidor de tickets y un generador de IDs único similar al Twitter snowflake. Optamos por snowflake porque cubre todos nuestros casos de uso y es escalable en un entorno distribuido.

Si hay tiempo extra al final de la entrevista, aquí hay algunos puntos adicionales para discutir:

Sincronización de relojes: En nuestro diseño, asumimos que los servidores de generación de IDs tienen el mismo reloj. Esta suposición puede no ser cierta cuando un servidor se está ejecutando en múltiples núcleos. El mismo desafío existe en escenarios de varias máquinas. Las soluciones para la sincronización de relojes están fuera del alcance de este libro, pero es importante entender que el problema existe. El Protocolo de Tiempo de Red es la solución más popular para este problema. Para lectores interesados, consulten el material de referencia [4].
Ajuste de la longitud de la sección: Por ejemplo, menos números de secuencia pero más bits de marca de tiempo son efectivos para baja concurrencia y aplicaciones a largo plazo.
Alta disponibilidad: Dado que un generador de IDs es un sistema crítico para la misión, debe ser altamente disponible.
¡Felicidades por llegar tan lejos! ¡Ahora, date una palmadita en la espalda! ¡Buen trabajo!

## Materiales de referencia

[1] Universally unique identifier: https://en.wikipedia.org/wiki/Universally_unique_identifier [2] Ticket Servers: Distributed Unique Primary Keys on the Cheap: https://code.flickr.net/2010/02/08/ticket-servers-distributed-unique-primarykeys-on-the-cheap/ [3] Announcing Snowflake: https://blog.twitter.com/engineering/en_us/a/2010/announcingsnowflake.html [4] Network time protocol: https://en.wikipedia.org/wiki/Network_Time_Protocol 




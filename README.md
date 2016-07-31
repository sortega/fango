# Saliendo del fango

Traducción al castellano del artículo "Out of the Tar Pit"

**Traducción en progreso**

Autores:

 - Ben Moseley ben@moseley.name
 - Peter Marks public@indigomail.net

Traducción:
 - Sebastián Ortega
 - Álvaro Castellanos

## Resumen

El mayor obstáculo en el desarrollo de sistemas software de envergadura es la
complejidad. En consonancia con Brooks, distinguimos entre complejidad
accidental y esencial pero no aceptamos su premisa de que la mayoría de la
complejidad que se puede encontrar en los sistemas actuales es esencial.
Identificamos las causas más comunes de la complejidad y discutimos
aproximaciones generales para minimizar la complejidad de naturaleza
accidental. Para hacer más concreta la exposición, se bosqueja una potencial
aproximación a la minimización de complejidad basada en la *programación
funcional* y en el *modelo de datos relacional de Codd*.

## 1. Introducción

La "crisis del software" fue identificada por primera vez en 1968 [NR69, p70] y
en las décadas que han transcurrido desde entonces se ha profundizado en lugar
de aliviarse. El mayor problema en el desarrollo y mantenimiento de grandes
sistemas software es la complejidad: estos sistemas son difíciles de comprender.
Creemos que en muchos casos el mayor contribuyente para esta complejidad es la
gestión del *estado* y el lastre que añade cuando se intenta analizar y
razonar acerca del sistema. Otros contribuyentes estrechamente relacionados son
el *volumen de código* y, de forma especialmente preocupante, el *flujo de
control* a través del sistema.

Las estrategias clásicas para gestionar el estado incluyen a la programación
orientada a objetos, que acopla estrechamente estado y comportamiento, y la
programación funcional, que en su forma más pura evita completamente el estado y
los efectos laterales. Estas estrategias padecen de varios (y diferentes)
problemas cuando se aplican al diseño de grandes sistemas de forma tradicional.

Defendemos que es posible tomar ideas útiles de ambas estrategias y que,
en combinación con algunas ideas del mundo de las bases de datos relacionales,
se puede tomar una aproximación con un significativo potencial para simplificar
la construcción de grandes sistemas software.

Este artículo está dividido en dos mitades. En la primera mitad nos centramos en
la complejidad. En la sección 2, examinamos la complejidad en general y
justificamos nuestra creencia de que ésta es la raíz de la crisis para, a
continuación examinar cómo se intenta comprender un sistema en la actualidad en
la sección 3. En la sección 4, examinamos las causas de la complejidad (i.e.
aquello que dificulta la compresión) antes de discutir los enfoques clásicos
para gestionar estas causas en la sección 5. En la sección 6, definimos qué
entendemos por "accidental" y "esencial" y después en la sección 7 damos
recomendaciones para abordar de forma alternativa las causas de la complejidad
(con énfasis en prevenir los problemas en lugar de afrontarlos).

En la segunda mitad del artículo detallamos un enfoque que se deriva de nuestra
estrategia recomendada. Comenzamos con una revisión del modelo relacional en la
sección 8 y una descripción de una potencial estrategia en la sección 9. En la
sección 10 proveemos un breve ejemplo de cómo éste podría ser usado.

Finalmente contrastamos nuestra aproximación con otras posibles en la sección 11
y concluimos en la sección 12.

## 2. Complejidad

En su clásico artículo, "No Silver Bullet", Brooks [Bro86] identificó cuatro
propiedades de los sistemas software que hacen el desarrollo de software
difícil: Complejidad, Conformidad, Facilidad de cambio e Invisibilidad. De entre
ellas, creemos que la Complejidad es la *única* significativa: las otras pueden
bien *ser clasificadas* como formas de complejidad o bien considerarse
problemáticas únicamente *a causa* de la complejidad en el sistema.

La complejidad es *la* causa última de la inmensa mayoría de los problemas
relacionados con el software hoy en día. Inestabilidad, retrasos, falta de
seguridad, a menudo hasta los problemas de rendimiento, pueden ser vistos como
el producto de una complejidad inmanejable en estos grandes sistemas. Esta
clasificación de la complejidad como *la* causa principal es consecuencia del
hecho de que ser capaces de *comprender* un sistema es un pre-requisito para
evitar todos estos problemas y la complejidad incapacita esta comprensión.

La relevancia de la complejidad es ampliamente reconocida. Como dijo Dijkstra [Dij97, EWD1243]:

> "...tenemos que mantenerlo ordenado, desenmarañado y simple si queremos
> evitar ser aplastado por las complejidades de nuestra propia obra..."
    
...y The Economist dedicó un artículo completo a la complejidad del software
[Eco04] señalando que algunas estimaciones valoran en 59 miles de millones de
dólares anuales de coste para la economía americana de los problemas
relacionados con el software.

La capacidad de pensar y razonar sobre nuestros sistemas (en particular sobre
los efectos de los cambios que se les realicen) es de una *crucial* importancia.
Los peligros de la complejidad y la importancia de la simplicidad han sido un tema popular en las exposiciones del premio Turing de la ACM. En su exposición de 1990, Corbato dijo [Cor91]:

> "El problema general con los sistemas ambiciosos es la complejidad.",
> "...es importante enfatizar el valor de la simplicidad y la elegancia, dado
> que la complejidad es un camino de dificultades creciente"
    
En 1997, Backus [Bac78] habló sobre las "complejidades y debilidades" de los lenguajes tradiciones y apuntó que:

> "hay una necesidad desesperada de una metodología poderosa que nos ayude a
> razonar acerca de los programas. ... los lenguajes convencionales crean una
> confusión innecesaria en nuestra forma de razonar sobre los programas"

Finalmente en su discurso de aceptación del premio Turing en 1980, Hoare [Hoa81]
hizo esta observación:

> "...hay una cualidad que no puede ser comprada... y es la fiabilidad. El
> precio de la fiabilidad es la búsqueda de la mayor simplicidad"

y

> "Llego a la conclusión de que hay dos formas de construir un diseño
> software: o bien se hace algo tan simple que *obviamente* no tiene defectos
> o bien se hace tan complicado que no hay deficiencias *obvias*. El primer
> método es mucho más difícil."
    
Esta es la desafortunada realidad:

> La simplicidad es *difícil*.
    
...pero el propósito de este artículo es dar motivos para el optimismo.

Nuestro argumento final es que el tipo de complejidad del que estamos
discutiendo en este artículo es la que hace a los grandes sistemas *difíciles de
comprender*. Este es el motivo que nos hace consumir recursos ingentes para
*crear y mantener* dichos sistemas. Este tipo de complejidad no tiene nada que
ver con teoría de la complejidad (la rama de la ciencia de la computación que
estudia los recursos consumidos *por una máquina al ejecutar* un programa).
No tienen relación alguna: es sencillo escribir un pequeño programa de pocas
líneas que sea increíblemente simple (en nuestro sentido) y que pertenezca a la
mayor clase de complejidad (en el sentido de la teoría de la complejidad). De
aquí en adelante sólo nos referiremos a complejidad del primer tipo.

Debemos examinar las que consideramos como principales causas comunes de la
complejidad (aspectos que dificultan la comprensión) después de examinar de qué
formas tratamos de *comprender* los sistemas habitualmente.

## 3. Formas de comprender un sistema

Defendíamos antes que el peligro de la complejidad lo trae su impacto en nuestra
capacidad para *comprender* sistemas. Por esto, es interesante considerar los
mecanismos que comúnmente se utilizan para tratar de comprender estos sistemas.
A continuación, consideraremos el impacto que las posibles causas de la
complejidad tienen en estos enfoques. Los dos enfoques más utilizados para la
compresión de los sistemas (o componentes de los sistemas) son:

 - **Pruebas**. Estas tratan de comprender un sistema desde fuera, como una
   "caja negra". Se determinan aspectos del sistema en base a observaciones
   sobre su comportamiento ante determinadas situaciones. Estas pruebas pueden
   ser desempeñadas manual o automáticamente. Lo primero es más
   común en las pruebas del sistema al completo y lo segundo a nivel de
   componente individual.

 - **Razonamiento informal**. Se trata de intentar comprender el sistema
   examinándolo desde dentro. Se espera que esta información adicional permita
   llegar a una compresión más precisa.

De los dos, el razonamiento informal es el más importante de lejos. Esto se debe
a que, como veremos más adelante, hay límites inherentes a lo que puede lograrse
por medio de pruebas, y por ello el razonamiento informal (en virtud de ser una
parte inherente del proceso de desarrollo) *siempre* se utiliza. La otra
justificación es que un mejor razonamiento informal lleva a *introducir menos
errores* mientras que un mejor proceso de pruebas tan solo lleva a *detectar más
errores*. Como Dijkstra dijo en su discurso de aceptación del premio Turing
[Dij72, EWD340]:

> "Aquellos que realmente quieren software fiable descubrirán que para empezar
> tienen que encontrar cómo evitar la mayoría de los errores."

y como O'Keefe (quien también hizo hincapié en la importancia de "*comprender tu
problema*" y que "*La elegancia no es opcional*") dijo [O'K90]:

> "Nuestra respuesta a los errores debería ser buscar formas de evitarlos, no
> culpar a la naturaleza de las cosas."

El principal problema con las pruebas es que una prueba (de cualquier tipo) que
utiliza un particular conjunto de entradas no dice *nada en absoluto* sobre
el comportamiento del sistema o el componente cuando se le de un conjunto
diferente de entradas. El gran número de posibles entradas diferentes
descarta la posibilidad probar exhaustivamente en el caso general, de ahí que
sea inevitable preguntarse al probar: *¿has probado los casos* adecuados*?*.
Sólo se puede responder con certeza a esta pregunta en el caso negativo, cuando
el sistema falla. De nuevo, como Dijkstra observó [Dij71, EWD303]:

> "las pruebas son desesperanzadoramente inadecuadas... pueden ser usadas muy
> efectivamente para demostrar la presencia de errores pero nunca para demostrar
> su ausencia."

Coincidimos con Dijkstra. *Confía* en pruebas bajo tu propia responsabilidad.
Esto *no* quiere decir que probar no sirva para nada. La conclusión es que
*todas* las formas de intentar comprender un sistema tienen sus limitaciones (y
esto incluye tanto al *razonamiento informal*, limitado en su alcance, impreciso
y por tanto propenso al error; como al *razonamiento formal*, que depende de la
precisión de la especificación). Debido a estas limitaciones puede ser
recomendable emplear pruebas *y* razonamiento de forma conjunta en muchas
situaciones.

Es precisamente *debido* a las limitaciones de todos estos enfoques que la
*simplicidad* es vital. Cuando se considera junto a las pruebas y el
razonamiento, la simplicidad es más importante que cualquiera de estos dos. Si
se tuviese que elegir entre invertir en pruebas o en simplicidad, la segunda
opción es con frecuencia la mejor ya que facilita *todos* los futuros intentos
de comprender el sistema, sean del tipo que sean.

## 4. Causas de la complejidad

En cualquier sistema no trivial hay una cierta cantidad de complejidad que es
inherente al problema a resolver. Sin embargo, en los grandes sistemas nos
encontramos con complejidad sospechosa de no ser "inherente al problema". Ahora
consideraremos algunas de las causas de esta complejidad.

### 4.1 Complejidad causada por el estado

Cualquiera que haya llamado al servicio de soporte de algún sistema software y
haya sido invitado a "probar de nuevo", "recargar el documento", "reiniciar el
programa", "reiniciar el ordenador", "reinstalar el programa" o incluso
"reinstalar el sistema operativo y después el programa" tiene experiencia de
primera mano sobre los problemas que el *estado*[^1] produce cuando se
intenta escribir software robusto y comprensible.

[^1]: Por "estado" nos referimos específicamente a *estado mutable*. Es decir,
    excluyendo cosas como por ejemplo las variables inmutables que sólo se
    pueden asignar una vez y que ofrecen los lenguajes de programación lógicos.
    
El motivo por el que estas situaciones son familiares para tanta gente es que
son habitualmente propuestas porque normalmente resuelven el problema. Esto
sucede porque muchos sistemas gestionan su estado de forma errónea. A su vez,
estos errores son introducidos porque la existencia del propio estado hace los
programas difíciles de *comprender*. Los hace complejos.

En relación al estado, nos mostramos de acuerdo con la opinión de Brooks cuando
afirma [Bro86]:

> "De la complejidad nace la dificultad para enumerar, ni siquiera comprender,
> todos los posibles estados del programa y de ahí nace falta de fiabilidad"

Estamos de acuerdo con esto pero creemos que es la mera existencia de tantos
posibles estados lo que en primer lugar produce complejidad, y:

> "computadoras... tienen un número muy grande de estados. Esto hace que
> concebirlos, describirlos y probarlos sea difícil. Los sistemas software
> tienen órdenes de magnitud más estados que las propias computadoras."

#### 4.1.1 Impacto del estado en las pruebas

La gravedad del impacto del *estado* en las pruebas que advirtió Brooks es
difícil de exagerar. El estado afecta a todos los tipos de pruebas, desde las
pruebas a nivel de sistema (donde el desarrollador padece los mismos problemas
que el desafortunado usuario que se mencionó) pasando por las de nivel de
componente y hasta las de unidad. El principal problema es que una prueba (de
cualquier tipo) en un sistema o componente que está en un *estado* concreto
aporta *cero información* acerca del comportamiento del sistema o componente en
cualquier otro *estado*.

La estrategia más común para probar un sistema con estado (a nivel de componente
o de sistema) pasa por establecer un estado "limpio" o "inicial" (no obstante
mayormente *oculto*), realizar las pruebas utilizando unas entradas y asumir (de
forma infundada en caso de error) que el sistema se va a comportar siempre de la
misma forma, *independientemente de su estado interno oculto*, cada vez que se
prueben esas entradas.

Simple y llanamente, esta estrategia barre bajo la alfombra el problema del
estado. Los motivos por los que se hace esto son, en primer lugar, que es
posible hacer progresos con esta estrategia y, en segundo lugar, porque no hay
otra alternativa cuando se trata de probar un sistema con un estado interno
oculto y complicado.

Por supuesto, no *siempre* es posible "hacer progresos" con esta estrategia. Si
alguna secuencia de eventos (entradas) puede hacer que el sistema "alcance un
estado *malo*" (específicamente un estado oculto *diferente* del que se utilizó
durante las pruebas) la situación puede salir mal y saldrá mal. Esto es lo que
le sucede al usuario del hipotético servicio de soporte citado al principio de
esta sección. Las soluciones propuestas son todas intentos de forzar que el
sistema vuelva a un "estado interno bueno".

Este problema (que las pruebas en un *estado* no te dicen *nada en absoluto*
acerca del sistema en un *estado* diferente) es completamente análogo a uno de
los problemas discutidos antes: probar con un *conjunto de entradas* no te dice
*nada en absoluto* sobre el comportamiento con otro *conjunto de entradas*. Sin
embargo, el problema que causa el estado suele ser peor, especialmente cuando se
prueban porciones grandes de un sistema, porque aunque el espacio de *entradas*
pueda ser muy grande, el espacio de *estados* tiende a ser incluso mayor.

Estos dos problemas similares, uno intrínseco al hecho de probar y el otro fruto
del *estado*, se combinan *horriblemente*. Cada uno de ellos introduce una
ingente cantidad de incertidumbre y por lo tanto son muy pocas las cosas que
*podemos* dar por seguras si el sistema/componente bajo escrutinio tiene estado.

#### 4.1.2 Impacto del estado en el razonamiento informal

Además de causar problemas a la hora de comprender un sistema desde fuera, el
estado le hace la vida más difícil al desarrollador que tiene que, "desde
dentro", intentar *razonar* (normalmente de forma informal) sobre el
comportamiento esperado del sistema.

Los procesos mentales utilizados para este razonamiento informal suelen girar
alrededor de una simulación mental por cada caso: "si esta variable está en este
estado, tendremos este resultado que es correcto y en otro caso, tendremos este
otro resultado que también lo es". Según aumenta el número de estados y por
tanto de escenarios a considerar, la efectividad de este ejercicio mental se
reduce tan rápido como la de las pruebas (con la ventaja de que se pueden
abstraer conjuntos de valores similares que se tratan de forma idéntica).

Uno de los problemas que afecta tanto a las pruebas como al razonamiento es la
tasa exponencial de crecimiento del número de estados posibles. Cada nuevo *bit*
de estado añadido *dobla* el número de estados posibles. Otro problema que
afecta especialmente al razonamiento informal es la *contaminación*.

Tomemos como ejemplo un sistema formado por procedimientos en los que algunos
mantienen estado y otros no. Ya hemos tratado la dificultad de comprender las
partes de un sistema que tienen estado y podríamos esperar que las demás fuesen
más fáciles de comprender. Tristemente, no es cierto. Si un procedimiento que no
mantiene estado utiliza a cualquier otro que si lo mantenga, *aunque sea
indirectamente*, lo más probable es que el primero quede *contaminado* y sólo
pueda ser entendido en el contexto de ese estado. Si intentamos ignorar ese
estado corremos el riesgo de padecer todos los problemas detallados
anteriormente. El problema con el estado es que *"cuando dejas que la nariz del
camello se asome a la tienda de campaña, el resto del camello suele llegar
detrás"*.

Todos estos motivos fundamentan nuestra creencia de que la mayor causa
individual de la complejidad en los grandes sistemas modernos es el *estado* y
que cuanto más hagamos para *limitarlo* y *gestionarlo* mejor.

### 4.2 Complejidad causada por el flujo de control

De forma cruda, el flujo de control o simplemente control se trata del *orden*
en el que se suceden los acontecimientos.

El problema con este control es que, con frecuencia, nos gustaría
despreocuparnos del mismo. Obviamente, dado que queremos construir un sistema
real en el que efectivamente *pasen* cosas, en algún momento el orden será
relevante para alguien, pero corremos un riesgo significativo de ocuparnos de
este tema innecesariamente.

La mayoría de lenguajes de programación tradicionales *fuerzan* a preocuparse
por el orden y en la mayoría de los casos el orden en el que las cosas *suceden*
es el mismo en el que se hayan escrito las sentencias en el texto del programa.
Este orden se ve modificado por instrucciones de explícitas de control (que
pueden ser condicionales) y por la existencia de subrutinas que se invocan
formando una pila implícita.

Es obvio que múltiples órdenes de evaluación son posibles pero sin embargo
existe muy poca variedad al respecto entre los lenguajes más extendidos.

Cuando el control es un aspecto implícito del lenguaje (como casi siempre es)
surge la dificultad de que cada parte del programa debe ser comprendido dentro
de ese contexto, incluso cuando el programador nunca quiso especificar un orden.
Cuando un programador es obligado (al usar un lenguaje con flujo de control
implícito) a especificar el control, se le obliga a especificar *cómo* debe
funcionar el sistema en lugar de simplemente *qué* se desea que haga. El
resultado es la obligación de *sobre especificar* el problema. Consideremos el
siguiente pseudo-código:

    a := b + 3
    c := d + 2
    e := f * 4
    
En este caso es claro que el programador es indiferente con respecto al orden
(i.e. *cómo*) en el que se ejecuten. Lo único que le interesa es la *relación*
entre ciertos valores pero además de esto ha sido obligado a especificar un
flujo de control arbitrario. Con frecuencia, en casos como éste, el compilador
llega al extremo de tener que analizar si este orden (que sólo ha sido
especificado por la semántica del lenguaje) puede ser ignorado con seguridad.

En casos simples como el anterior se le da poca importancia a este problema pero
es importante darse cuenta de los dos innecesarios procesos que se dan: primero
se obliga a determinar un orden artificial y luego se hace trabajo adicional
para eliminarlo.

Este asunto, aparentemente inocuo, puede complicar el proceso de razonamiento
informal de forma realmente significativa. Esto es porque la persona que lee el
código anterior debe reproducir el trabajo que haría el hipotético compilado.
Debe empezar (debido a la semántica del lenguaje) asumiendo que el orden *es*
significativo para luego analizar si realmente lo era (en casos menos triviales
que el anterior esto puede ser muy difícil). Esto es problemático porque los
errores de juicio haciendo este análisis pueden llevar a introducir errores muy
sutiles y difíciles de encontrar.

Es importante recalcar que el problema no está en el propio *texto* del programa
anterior, que debe ser escrito en algún orden. El problema es causado por la
*semántica* del hipotético lenguaje imperativo que hemos tomado como ejemplo. Es
*posible* imaginar este mismo fragmento de código como válido en un lenguaje de
programación cuya semántica no defina el orden en tiempo de ejecución mediante
el orden en el texto del programa[^2].

[^2]: De hecho las primeras versiones del lenguaje Oz (con concurrencia *implícita* a nivel de sentencia) seguían un modelo similar [vRH04, p809].

Tras considerar el impacto del control en el *razonamiento informal* ahora
trataremos otro problema asociado al control, la concurrencia, que afecta
también al proceso de *pruebas*.

Al igual que las estructuras de control básicas como los condicionales y a
diferencia de orden implícito dado a las sentencias, la concurrencia es
típicamente especificada de forma *explícita* en la mayoría de lenguajes. El
modelo más común es el de "concurrencia con estado compartido" en el que se debe
especificar explícitamente aspectos de sincronización. El impacto que esto tiene
en la capacidad de razonar informalmente es bien conocido. Según aumenta el
*número de escenarios* que deben ser considerados mentalmente al leer el
programa va aumentando la dificultad para razonar. Cómo se mencionó
anteriormente, este problema es similar al que provoca el propio estado
aumentando el número de escenarios a considerar.

La concurrencia también tiene impacto en las pruebas ya que se pierde la
capacidad de obtener el mismo resultado al repetir las pruebas aunque se
garantice un estado inicial consistente. Ejecutar una prueba bajo condiciones de
concurrencia *no aporta información* sobre qué pasará en la siguiente ejecución
de la misma prueba, las mismas entradas y el mismo estado inicial... No se puede
llegar a una situación peor.

### 4.3 Complejidad causada por el volumen de código

La última causa de la complejidad que vamos a examinar en detalle es el propio
volumen de código.

En muchos sentidos es un efecto secundario ya que mucho código está dedicado a
gestionar *estado* o a especificar el *flujo de control*. Por este motivo
omitimos con frecuencia el volumen de código en el resto del análisis. Sin
embargo hay dos buenos motivos para considerarlo de forma independiente: primero
porque es la forma de complejidad más fácil de *medir* y segundo porque
interactúa negativamente con las otras causas de la complejidad.

Brooks advirtió en [Bro86]:

> "Muchos de los problemas clásicos para desarrollar productos software derivan
> de esta complejidad esencial y su crecimiento no linear con respecto a su
> tamaño"

A grandes trazos, estamos de acuerdo en que *en la mayoría de sistemas actuales*
esto es cierto (discrepamos en el uso de la palabra "esencial" como ya se dijo).
Es decir, en la mayoría de los sistemas la complejidad *muestra* un incremento
no linear con respecto a su tamaño (en volumen de código). A su vez, esta falta
de linealidad significa que es vital reducir la cantidad de código a su mínima
expresión.

También queremos recalcar la siguiente idea de Dijkstra [Dij72, EWD340] al
respecto de este tema:

> "Se ha sugerido que hay una especie de ley de la naturaleza por la que la
> cantidad de trabajo intelectual necesario es proporcional al cuadrado de la
> longitud del programa. Pero, afortunadamente, nadie ha sido capaz de probar
> esta ley. Y esto es porque no tiene por qué ser cierta. ... Tiendo a asumir,
> y de momento no he sido corregido por la experiencia, que aplicando
> adecuadamente nuestra capacidad de abstracción, el trabajo intelectual para
> concebir o para comprender un programa no tiene por qué crecer más que
> proporcionalmente a la longitud del código."

Estamos completamente de acuerdo con esto y es la razón de nuestra matización
anterior ("en la mayoría de sistemas actuales"). Creemos que mediante la gestión
efectiva de las dos principales causas de la complejidad, estado y control, es
mucho menos claro que la complejidad tenga que crecer de forma no lineal con
respecto al volumen de código.

### 4.4 Otras causas de la complejidad

Para concluir, existen otras causas para la complejidad como por ejemplo: código
duplicado, código que nunca llega a ejercitarse ("código muerto"), **abstracción
innecesaria**[^3], oportunidades para abstraer no aprovechadas, modularización
deficiente, documentación de baja calidad...

[^3]: Particularmente abstracción de datos innecesaria. Argumentaremos que esto pasa con la *mayoría* de la abstracción de datos en la sección 9.2.4.

Todas estas otras causas son el resultado de estos tres interrelacionados
principios:

 - **La complejidad crea más complejidad.** Hay un variado conjunto de causas
   *secundarias* de complejidad. Esto incluye toda la complejidad que se
   introduce *como resultado* de no ser capaz de comprender con claridad el
   sistema. La *duplicación* es un ejemplo prototípico de esto: si debido al
   estado, control o volumen del código no está claro si una funcionalidad
   existente hace exactamente lo que se requiere para otros casos habrá una
   fuerte tendencia a duplicarla. Esto es aún más cierto cuando hay presiones
   temporales.
   
  - **La simplicidad es *difícil*.** Como se indicó anteriormente, es necesario
    un esfuerzo significativo para alcanzar la simplicidad. La primera solución
    que se prueba no suele ser la más simple, especialmente si hay complejidades
    preexistentes o presiones temporales. La simplicidad sólo se puede conseguir
    si se admite este hecho y por ésta es buscada y apreciada.
  
  - **El poder corrompe.** A lo que nos referimos es que, si no hay garantías a
    nivel del lenguaje de programación (i.e. restricciones a las *capacidades o
    poderes* que el lenguaje ofrece) *se producirán* errores (y abusos). Por
    este motivo los recolectores de basura son buenos ya que el *poder* de
    gestionar manualmente la memoria es eliminado. El mismo principio es
    relevante para el *estado*, otro tipo de poder o capacidad. A este respecto
    tenemos que ser muy cuidadosos con cualquier lenguaje que *permita* tener
    estado, independientemente de cuánto desincentive su uso (ML y Scheme son
    ejemplos obvios). La conclusión es que cuanto más *poderoso* es un lenguaje
    (i.e. cuantas más cosas es *posible* expresar con el mismo), más difícil es
    *comprender* los sistemas construidos con ese lenguaje.
    
Algunas de estas causas son de naturaleza humana, otras son ambientales pero
todas pueden, en nuestra opinión, ser aliviadas en gran medida si se hace
énfasis en la gestión efectiva de las causas de la complejidad tratadas en las
secciones 4.1 a 4.3.

## 5. Aproximaciones clásicas a la gestión de la complejidad

Una buena forma de comprender las diferentes aproximaciones clásicas a la
gestión de la complejidad es examinar la propuesta de los lenguajes de
programación pertenecientes a los tres principales estilos (imperativo,
funcional y lógico). Tomaremos la aproximación orientada a objetos como un
ejemplo representativo del estilo imperativo.

### 5.1 Orientación a objetos

La orientación a objetos es esencialmente una aproximación imperativa a pesar de
ser un término muy amplio que cubre lenguajes basados en clases al estilo de
Java hasta lenguajes basados en prototipos al estilo de Self y desde lenguajes
con single-dispatch hasta lenguajes con multiple dispatch como CLOS y desde los
tradicionales objetos pasivos hasta los objetos activo o actores. Ésta ha
evolucionado hasta llegar a ser la forma predominante en la que se desarrolla el
software en computadores tradicionales (de arquitectura von-Neumann) y muchas de
sus características surgen de facilitar el estilo von-Neumann, es decir,
computaciones basadas en estado.

#### 5.1.1 Estado

En la mayoría de sus formas, en la programación orientada a objetos (POO), un
objeto consiste en algo de estado y una serie de procedimientos para acceder y
manipular ese estado.

Esto es esencialmente equivalente a la (anterior) idea de *tipo abstracto de
datos* (TAD) y es uno de los puntos fuertes de la POO cuando se compara con
otras aproximaciones imperativas menos estructuradas. En el contexto de la POO
esto es conocido como *encapsulación* y permite al programador garantizar
restricciones de integridad sobre el estado de un objeto al regular el acceso al
estado mediante procedimientos de acceso ("métodos").

Un problema con este esquema es que, si varios procedimientos acceden o
manipulan la misma porción de estado, habrá varios puntos donde la integridad
debe ser garantizada. Estos procedimientos pueden estar o no en el mismo fichero
dependiendo del lenguaje y de qué características, como la herencia, se están
usando. Otro gran problema[^4] es que las garantías basadas en restricciones
garantizadas mediante encapsulación están muy sesgadas hacia restricciones a
nivel de un único objeto y es difícil garantizar restricciones más complicadas
que involucren a varios objetos (para empezar no está claro dónde deben residir
estas restricciones).

[^4]: Este problema en particular no aplica a los lenguajes orientados a objetos que se basan en funciones genéricas (como CLOS) ya que no comparten el mismo concepto de encapsulación.

##### Identidad y estado

Otro aspecto intrínseco a la POO que está íntimamente relacionado con el estado
es el concepto de *identidad del objeto*.

En la POO, cada objeto es una entidad únicamente identificable sin importar el
valor de sus atributos. Esto se denomina como identidad *intensiva*[^nt1] (en
contraste con la identidad *extensiva* en la que la igualdad se basa en si los
atributos coinciden). Citando a Baker [Bak93]:

[^nt1]: N.T. *intensional identity* en el original.

> En cierto sentido, dar identidad a los objetos puede considerarse como
> rechazar el punto de vista del "álgebra relacional" en el que dos objetos sólo
> pueden ser distinguidos si sus atributos difieren.

Dar esta identidad a los objetos *tiene* sentido cuando son utilizados para
crear una abstracción con estado (mutable). Esto es así porque dos objetos
distintos pueden ser modificados para contener diferente estado incluso si sus
atributos (el estado que contienen) eran inicialmente idénticos.

Sin embargo, cuando no se necesita mutabilidad (como por ejemplo al representar
un simple valor numérico), la aproximación orientada a objetos se ve forzada a
adoptar técnicas como la creación de "objetos valor" dónde se intenta subvertir
la *identidad del objeto* y re-introducir la identidad extensiva. En estos casos
es común recurrir a procedimientos de acceso (métodos) para determinar si dos
objetos son equivalentes de una forma alternativa y específica del dominio. Un
riesgo, a parte del volumen adicional de código para soportar esto, es la falta
de garantías de que la equivalencia específica del dominio sea conforme al
concepto estándar de relación de equivalencia (por ejemplo no tiene por qué
garantizarse la transitividad).

El concepto de *identidad del objeto* es intrínseco al uso de estado y es, por
formar parte del propio paradigma, inevitable. Este concepto adicional de
identidad añade complejidad a la tarea de razonar sobre los sistemas
desarrollados en el estilo orientado a objetos (es necesario alternar
mentalmente entre ambos conceptos de equivalencia y confundirlos puede producir
graves errores).

##### Estado en la programación orientada a objetos

La conclusión es que todas las formas de programación orientada a objetos se
basan en el estado (contenido en los propios objetos) y en general, todo el
comportamiento es influenciado por este estado. A consecuencia de esto, la
programación orientada a objetos padece de forma directa los problemas asociados
con el estado que se describieron anteriormente y, por tanto, creemos que no
proporciona los fundamentos necesarios para evitar la complejidad.

#### 5.1.2 Control

La mayoría de los lenguajes orientados a objetos proveen un flujo de control
secuencial estándar y muchos de ellos, también ofrecen mecanismos para la
clásica "concurrencia de estado compartido" junto con todos los problemas de
complejidad asociados. Los lenguajes con modelo de actores se diferencian de los
anteriores en que ofrecen un modelo de concurrencia basado en "paso de mensajes"
(se asocian hilos de control con objetos individuales y los mensajes circulan
entre ellos). Este modelo puede llevar a un razonamiento informal más accesible
en algunos casos pero los lenguajes con modelo de actores no se han llegado a
popularizar.

#### 5.1.3 POO: resumen

Los programas imperativos y orientados a objetos sufren enormemente de la
complejidad derivada tanto del estado como del flujo de control.

### 5.2 Programación funcional

Mientras que la programación orientada a objetos fue motivada para tratar mejor
con la clásica arquitectura de von-Neumann basada en el estado, la programación
funcional tiene sus raíces en el cálculo lambda de Church (si descartamos
aproximaciones aún más simples basadas en lógica combinatoria), que es
completamente ajeno al estado. El lambda cálculo no tipado es equivalente en
potencia a la abstracción computacional basada en estado por antonomasia: la
máquina de Turing.

#### 5.2.1 Estado

Los lenguajes de programación funcionales modernos son habitualmente
clasificados como "puros", aquellos que como Haskell [PJ+03] evitan el estado y
los efectos laterales completamente, e "impuros", aquellos que como ML empujan a
evitar el estado y los efectos laterales pero los permiten. Cuando no se
mencione explícitamente nos referiremos a la programación funcional en su forma
pura.

La principal fortaleza de la programación funcional es que al evitar el estado
(y los efectos laterales) el sistema resultante adquiere *transparencia
referencial*. Esto implica que a partir de unos argumentos dados, una función
siempre producirá el mismo resultado (o más informalmente, siempre tendrá el
mismo comportamiento). Todo aquello que pueda influir en el resultado siempre
estará presente en los parámetros de entrada.

Esta férrea garantía evita uno de los problemas asociados a las pruebas ya
discutidos. Consecuentemente, incluso a pesar de que el otro problema persiste
(probar con unas entradas *no dice nada* sobre el comportamiento con otras), el
proceso de pruebas es mucho más efectivo en un sistema desarrollado en un estilo
funcional.

Al evitar el estado, la programación funcional, también evita todos los otros
problemas asociados a la gestión del estado ya presentados por lo que, entre
otras cosas, el razonamiento informal se vuelve mucho más efectivo.

#### 5.2.2 Control

La mayoría de lenguajes funcionales tienen un orden secuencial implícito (de
izquierda a derecha al evaluar los argumentos de las funciones) y por tanto
padecen muchos de los problemas ya mencionados. Sin embargo, los lenguajes
funcionales si que disfrutan de una ligera ventaja en lo relativo al control ya
que incentivan el uso de estructuras de control más abstractas de orden superior
(como `fold` / `map`) en lugar de iteración explícita.

También existen versiones concurrentes de muchos lenguajes funcionales y el
hecho de que, en general, evitan el estado puede dar réditos en este área. Por
ejemplo, en un lenguaje funcional puro es siempre seguro evaluar todos los
argumentos de una función en paralelo.

#### 5.2.3 Tipos de estado

En la mayoría de este artículo cuando mencionamos "estado" a lo que realmente
nos referimos es a "estado mutable".

En los lenguajes que no soportan (o desincentivan) el estado mutable es común
conseguir efectos similares pasando parámetros adicionales a los procedimientos
(funciones). Consideremos un procedimiento que realiza algún tipo de cálculo con
estado que devuelve un resultado. Quizás el procedimiento implementa un contador
y devuelve un valor incrementado con cada llamada:

    procedure int getNextCounter()
      // 'counter' is declared and initialized elsewhere in the code
      counter := counter + 1
      return counter

La forma en la que esto se suele implementar en un lenguaje funcional básico
consiste en reemplazar el procedimiento sin parámetros basado en estado con una
función que recibe un argumento y devuelve un par de valores como resultado.

    function (int,int) getNextCounter(int oldCounter)
      let int result = oldCounter + 1
      let int newCounter = oldCounter + 1
      return (newCounter, result)

Esto impone a quién invoque la función la obligación de asegurarse de que la
siguiente vez que se llame a `getNextCounter` se le proporcione el `newCounter`
devuelto en la anterior invocación. En efecto, lo que está pasando es que el
estado que se ocultaba dentro del procedimiento `getNextCounter` ha sido
reemplazado por un nuevo parámetro tanto en la entrada como en la salida de la
función `getNextCounter`. Este parámetro adicional no es *mutable* en forma
alguna (la entidad que es referida como `oldCounter` es un *valor* diferente
cada vez que se invoca la función).

Como hemos discutido, la versión funcional de este programa *tiene*
transparencia referencial al contrario que la versión imperativa. Por lo tanto
el llamante del *procedimiento* `getNextCounter` desconoce que puede influir en
el resultado que recibe (en principio, podría depender de un gran número de
variables mutables ocultas) mientras que el llamante de la *función*
`getNextCounter` puede ver inmediatamente que el resultado depende sólo de los
valores proporcionados a la función.

A pesar de esto, es un hecho que estamos usando valores funcionales para simular
estado. No hay nada que, *en principio*, impida a los programas funcionales
pasar y devolver un único parámetro adicional en cada una de las funciones del
sistema. Si este parámetro fuese una colección (valor compuesto) podría usarse
para *simular* un gran conjunto de variables mutables. Como resultado, este
método recrea un conjunto de variables globales y, por lo tanto, aunque se
mantiene la transparencia referencial se pierde la facilidad de razonamiento
(seguimos sabiendo que cada función depende sólo de sus argumentos pero uno de
ellos es muy grande *y contiene valores irrelevantes* por lo que este
conocimiento como ayuda para la compresión es poco valioso). Este es, sin
embargo, un ejemplo extremo y no menoscaba la potencia que en general tiene la
aproximación funcional.

Merece la pena mencionar brevemente que, aunque no se disfrute de la *garantía*
de la transparencia referencial, no hay ninguna razón por la que el estilo de
programación funcional no se pueda adoptar en los lenguajes con estado (i.e.
imperativos y funcionales impuros). De forma más general podíamos argumentar
que, sin importar el lenguaje utilizado, es beneficioso evitar el estado
mutable, implícito y oculto.

#### 5.2.4 Estado y modularidad

A veces se argumenta (e.g. [vRH4, p315]) que el estado es importante porque
permite un cierto tipo de modularidad, lo que es cierto. Dentro de un marco de
trabajo con estado es posible añadir estado adicional a cualquier componente sin
modificar los componentes que lo invocan. Desde el marco de trabajo funcional el
mismo efector sólo puede conseguirse modificando cada componente que lo invoque
de forma que se haga llegar la nueva información adicional (como en el caso de
la función `getNextCounter` vista anteriormente).

Hay un compromiso fundamental entre ambas aproximaciones. En la aproximación
funcional (cuando se intenta tener resultados parecidos a la basada en estado)
eres forzado a cambiar cada parte del programa que podría ser afectada
(añadiendo el nuevo parámetro relevante) mientras que en la basada en estado no
es necesario.

Sin embargo, esto significa que en un programa funcional *siempre puedes
determinar que controla el resultado de un procedimiento (i.e. función)*
simplemente mirando a los argumentos proporcionados en su invocación. En un
programa basado en estado, esta propiedad (de nuevo, a consecuencia de la
*transparencia referencial*) es completamente destruida, nunca puedes saber qué
controlará el resultado, y *potencialmente* tendrás que inspeccionar cada
fragmento de código de *todo el sistema* para determinarlo.

El compromiso se da entre la *complejidad* (con la habilidad de tomar un atajo
con determinados tipos de cambio) y la *simplicidad* (con *enormes* mejoras en
términos de pruebas y comprensión). Al igual que con la disciplina del tipado
(estático), se intercambia una única inversión por adelantado a cambio de
futuras ganancias continuadas y seguridad ("por adelantado" ya que cada
fragmento de código es escrito una vez y leído, razonado y probado
continuamente).

Un problema adicional con el argumento de la modularidad es que algunos
ejemplos, como el del uso de la cuenta de invocaciones a un procedimiento
(función) para depuración/mejora de rendimiento, parecen más apropiados para la
infraestructura/lenguaje en lugar de para el sistema en sí mismo (preferimos
recomendar una separación clara entre las tareas administrativas/de diagnóstico
y el núcleo del sistema).

Aún así es un hecho que estos argumentos han sido insuficientes para una
adopción extendida de la programación funcional. Por tanto, debemos concluir que
el principal punto débil de la programación funcional es el reverso de su
principal fortaleza: los problemas que se dan cuando el sistema debe mantener
estado de algún tipo (lo que es muy común).

Es inevitable preguntarse si hay alguna forma de "estar en misa y repicando".
Una posibilidad es el elegante sistema de mónadas utilizado por Haskell [Wad95].
Este sistema permite evitar el problema anterior pero se puede abusar del mismo
para crear un sub-lenguaje con estado y efectos laterales (y, por tanto,
re-introducir todos los problemas que se buscaba evitar) en Haskell, aunque al
menos deja su marca en los tipos de las funciones. De nuevo, a pesar de sus
puntos fuertes, las mónadas han sido insuficientes para permitir una adopción
mayoritaria de las técnicas funcionales.

#### 5.2.5 Programación funcional: resumen

### 5.3 Programación lógica
#### 5.3.1 Estado
#### 5.3.2 Control
#### 5.3.3 Programación lógica: resumen

## 6. Accidentes y esencia

Brooks considera como dificultades "esenciales" las inherentes a la naturaleza
del software y el resto las clasifica como "accidentales".

Normalmente usaremos los términos con el mismo signicado, pero preferimos
comenzar considerando a la complejidad del problema en sí misma incluso antes de
siquiera mencionar al software. Es por ello que definimos los siguientes dos
tipos de complejidad:

- **Complejidad esencial** es inherente y la esencial del *problema* (como se
  percibe por los usuarios).

- **Complejidad accidental** es todo lo demás, la complejidad con la que el
  equipo de desarrollo no tendría que luchar en un mundo ideal (e.g. complejidad
  derivada de los problemas de rendimiento, lenguajes suboptimos e
  infrastructura).

Nótese que la definición de *esencial* es deliberadamente más estricta que en su
uso común. Especialemente cuando usamos el término *esencial*, estrictamente
queremos decir esencial a *los problemas de los usuarios* (en lugar de, quizás,
esencial *para algo específico, implementado, del sistema,* o incluso, esencial
*para el software en general*). Por ejemplo, de acuerdo a la terminología que
usaremos en este artículo, bits, bytes, transistores, electricidad y ordenadores
en sí mismos *no* son en ningún sentido esenciales (porque no tienen nada que
ver con los problemas de los usuarios).

Igualmente, el término "accidente" es comúnmente más usado con la connotación de
"percance". Aquí (al igual que Brooks) lo usamos en el sentido más general de
"algo no esencial que está presente".

Con el fin de justificar estas dos definiciones comenzamos considerando el papel
del equipo de desarrollo de software, concretamente para producir (utilizando un
lenguaje e infrastructura determinados) y mantener un sistema de software que
cumple los propositos de sus usuarios. La complejidad en la cual estamos
interesados es la de este propósito y esto es lo que tratamos de clasificar como
accidental o esencial. Hasta ahora vemos la complejidad esencial como "la
complejidad, a la que el equipo *tendrá* que prestarle atención, incluso en un
mundo ideal".

Mencionar que el "tener que" partir de esta observación es fundamental, si hay
cualquier otra forma *posible* de que el equipo pudiera producir un sistema que
los usuarios consideraran correcto *sin* tener en consideración un tipo de
complejidad dada, entonces esa complejidad no sería esencial.

Dado que en el mundo real no todas las *aproximaciones* son prácticas, se deduce
que cualquier desarrollo necesitará lidiar con algo de complejidad accidental.
La definición no pretende negar esto, simplemente identificar su naturaleza
secundaria.

En última instancia (como veremos más adelante en la sección 7) nuestra
definición es equivalente a decir que lo que es esencial para el equipo es lo
que a los *usuarios* les preocupa. Esto es porque en un mundo ideal estaríamos
usando un lenguaje o infrastructura la cual nos dejaría expresar directamente
los problemas de los usuarios sin tener que expresar nada más, y así es como
llegamos a las definiciones dadas anteriormente.

El argumento podría ser presentado como que, en un mundo *ideal* podríamos
encontrar una infrastructura que ya resuelva el problema de los usuarios por
completo. Si bien, es posible imaginar que alguien ya ha hecho el trabajo, no es
particularmente esclarecedor, puede ser mejor considerar una restricción
implícita de que el hipotético lenguaje e infrastructura sean de uso general y
neutral al dominio.

Una consecuencia de esta definición es que si el usuario ni siquiera *sabe qué
es algo* (e.g. un grupo de hilos o un contador de un bucle, por seleccionar dos
ejemplos arbitrarios) no es posible que sea esencial dada nuestra definición
(estamos suponiendo, por supuesto, por desgracia la posibilidad con algo de
optimismo, que los usuarios, de hecho, conocen y comprenden el problema que
quieren resolver).

Brooks afirma [Bro86] (y otros como Booch corroboran [Boo91]) que "la
complejidad del software es una propiedad esencial, no accidental". Esto sugiere
que la mayoría (al menos) de la complejidad que encontramos en los grandes
sistemas contemporáneos es de tipo esencial.

Disentimos. La complejidad no es un propiedad inherente al software (o
esencial) por sí misma (es perfectamente posible escribir software simple y sin
embargo que siga siendo software), es más, mucha de la complejidad que vemos en
el software existente no es esencial (para el problema). Cuando se trata de
complejidad accidental y esencial creemos firmemente que la primera existe y que
el objetivo de la ingeniería del software tiene que ser eliminar tanta como sea
posible, y a la vez, ayudar con la segunda.

Debido a esto, es vital que examinemos cuidadosamente la *complejidad*
*accidental*. Ahora intentaremos clasificar la complejidad en accidental o
esencial para ciertos casos.

## 7. Recomendaciones generales

Dado que nuestras recomendaciones principales giran alrededor de intentar
*evitar* tanta complejidad accidental como sea posible, necesitamos preguntarnos
*qué* complejidad deben ser considerada accidental y cuál esencial.

Debemos responder esta pregunta considerando qué complejidad es *imposible*
evitar incluso en un mundo ideal (esto es básicamente nuestra *definición* de
esencial). A continuación veremos cómo de realista es este mundo ideal y
finalmente daremos algunas recomendaciones.

### 7.1 Mundo ideal

En un mundo ideal no estamos interesados en el rendimiento,o si nuestro lenguaje
e infrastructura nos proveé todo el soporte que deseamos. Es en ese contexto
donde vamos a examinar el estado y el control. En concreto, vamos a identificar
el estado como estado accidental, si podemos omitirlo en este mundo ideal, y lo
mismo realizaremos para el control.

Incluso en un mundo ideal necesitamos empezar por alguna parte, y parece
razonable suponer que tenemos que comenzar con una serie de *requisitos
informales* de los potenciales usuarios.

Nuestra siguiente observación es que debido a que en última instancia,
necesitamos que algo *suceda*, i.e., vamos a necesitar tener nuestro sistema de
procesado mecánico (en nuestro ordenador), necesitaremos *formalizar*.
Vamos a necesitar derivar los requisitos formales de los informales.

Entonces, en resumen, esto significa que incluso en un mundo ideal tenemos:

>Requisitos informales -> Requisitos formales

Tenga en cuenta que estamos buscando una mayor simplicidad, es crucial que la
formalización se haga sin añadir ningún tipo de aspecto *accidental* en
absoluto. En concreto, esto significa que en un mundo ideal, la formalización
debe hacerse *sin vistas a la ejecución de ningún tipo*. La única preocupación
cuando se realizan los requisitos formales debe ser garantizar que no hay
ambigüedad *relevante*[^6] en los requisitos informales (i.e. que no existen
omisiones).

[^6]: Incluimos la palabra "relevante" aquí porque en muchos casos, puede haber muchas soluciones posibles que sean aceptadas, y en tales casos, los requisitos pueden ser ambiguos en este sentido, sin embargo, eso no se considera que sea una ambigüedad "relevante", i.e., no lo hace corresponder a una *omisión* errónea de los requisitos.

Así, después de haber elaborado los requisitos formales, ¿Cuál debería ser el
siguiente paso a realizar? Dado que estamos considerando un mundo ideal, no es
razonable asumir que el siguiente paso es simplemente *ejecutar* estos
requisitos de forma directa en nuestra infrastructura de uso general
subyacente.[^7]

[^7]: En presencia de ambigüedades *irrelevantes* esto significará que la infrastructura debe *elegir* una de las posibilidades, o tal vez incluso proporcionar todas las soluciones posibles.

Esta situación es la simplicidad *absoluta*, no parece concebible que podamos
realizar nada mejor que esto, incluso en un mundo ideal.

Es interesante observar que efectivamente lo que acabamos de describir es, en
efecto, la verdadera *esencia* de la *programación declarativa*, i.e., que sólo
es necesario especificar *qué* se requiere, no *cómo* debe ser logrado.

Ahora vamos a considerar las implicaciones de este enfoque "ideal" para los
tipos complejidad discutidos anteriormente.

#### 7.1.1 Estado en el mundo ideal
#### 7.1.2 Control en el mundo ideal
#### 7.1.3 Resumen

### 7.2 Limitaciones teóricas y prácticas
#### 7.2.1 Lenguajes de especificación formal
#### 7.2.2 Facilidad de expresión
#### 7.2.3 Complejidad accidental requerida

### 7.3 Recomendaciones
#### 7.3.1 Complejidad accidental requerida
#### 7.3.2 Separación y relaciones entre los componentes

### 7.4 Resumen

## 8. Modelo relacional

A pesar de sus orígenes, el modelo relacional [Cod70] *en si mismo* no tiene
nada que ver con bases de datos. No es más que una forma elegante de
*estructurar* datos, una forma de *manipular* los mismos y un mecanismo para
mantener estado *íntegro* y consistente. Estas características son aplicables
a estado y datos en cualquier contexto.

A parte de estas tres áreas [Cod79, section 2.1], [Dat04, p109], un cuarto
punto fuerte del modelo relacional es el énfasis en tener una marcada
separación entre la capa lógica y física del sistema. Esto se traduce en que
el diseño del modelo lógico (minimizando la complejidad) es atacado de forma
separada del diseño del modelo de almacenamiento físico y de la correspondencia
entre ambos[^15]. Este principio se denomina *independencia de datos* y es una
parte crucial del modelo relacional [Cod70, section 1.1].

[^15]: Desafortunadamente la mayoría de sistemas gestores de bases de datos
están bastante limitados en el grado de flexibilidad permitida por la
correspondencia física/lógica. Desafortunadamente, los asuntos relativos al
rendimiento físico pueden invadir el diseño lógico incluso cuando evitar esto
era uno de los principales objetivos originales de Codd.

Vemos el modelo relacional compuesto de los siguientes cuatro aspectos:

 - **Estructura.** Se utilizan *relaciones* par representar todos los datos.
 - **Manipulación.** Un medio para especificar datos derivados.
 - **Integridad.** Un medio para especificar restricciones de datos de
   obligado cumplimiento.
 - **Independencia de datos.** Hay una separación clara entre los datos
   lógicos y su representación física.

Trataremos brevemente cada uno de estos aspectos. [Dat04] ofrece una visión de
conjunto sobre el modelo relacional más exhaustiva.

Como comentario final, recordar al lector que SQL, en cualquier versión, no
sigue fielmente el modelo relacional [Cod90, p371, Serious flaws in SQL],
[Dat04, p xxiv] por lo que aconsejamos no equiparar ambos.

### 8.1 Estructura

#### 8.1.1 Relaciones

Como se ha mencionado previamente, las relaciones son la única forma de
estructurar datos en el modelo relacional. Una relación se puede ver como un
*conjunto de registros* homogéneo en el que cada registro es conjunto
heterogéneo de *atributos* con nombre (ligeramente distinto del concepto
matemático de relación como conjunto de tuplas cuyos componentes se identifican
por su posición en lugar de por nombre).

Esta definición implica que una relación, por el hecho de ser un conjunto, no
puede contener duplicados y no tiene orden. Ambas restricciones contrastan con
el uso cotidiano de la palabra *tabla* que obviamente puede contener filas
duplicadas (y nombres de columna) y, por el hecho de ser una entidad visual en
una página, inevitablemente sus filas y columnas aparecen en un orden
determinado.

Las relaciones puede ser de estos tipos:

 - **Relaciones base**. Aquellas directamente almacenadas.
 - **Relaciones derivadas** (conocidas como *vistas*). Definidas en términos de
   otras relaciones (a su vez base o derivadas). Ver sección 8.2.

Como propone Date [Dat04], es útil pensar en cada relación como un único *valor*
(aunque compuesto) y considerar el estado mutable no como una "relación mutable"
sino como una *variable* que en cada momento está asociada a una relación
*valor* concreta que cambia con el tiempo. Date llama a estas variables
*variables relación* o *relvars*, lo que lleva a los términos *relvar base* y
*relvar derivada*. Utilizaremos su terminología de aquí en adelante. (Sin
embargo, nuestra definición de relación es ligeramente distinta ya que,
siguiendo las prácticas estándar en tipado estático, no consideramos el tipo
como parte del valor.)

#### 8.1.2 Beneficios estructurales de las relaciones - Independencia de la forma de acceso

Estructurar datos mediante relaciones es una idea atractiva porque no es
necesario tomar decisiones *subjetivas* de antemano acerca de la *forma de
acceso* que será más adelante utilizada para consultar y procesar los datos.

Para comprender que entendemos por *forma o camino de acceso*, consideremos un
ejemplo simple. Supongamos que queremos representar información acerca de
empleados y los departamentos en los que éstos trabajan. Un sistema en el que
elegir la estructura de los datos implique decidir las "rutas" posibles entre
las instancias de los datos (por ejemplo *desde* un empleado dado *hasta* un
departamento) es *dependiente del camino de acceso*.

Las dos principales formas de estructurar datos que precedieron al modelo
relacional (los modelos jerárquico y en red) eran ambos dependientes del camino
en este sentido. Por ejemplo, en el modelo jerárquico se debe tomar de forma
temprana una decisión sobre si el departamento es el nivel raíz (cada
departamento "contiene" sus empleados) o viceversa (los empleados "contienen"
sus departamentos). Esta decisión impactará todos los usos futuros de los datos.
En el primer caso, será fácil enumerar todos los empleados de un determinado
departamento (siguiendo el camino de acceso) pero será más difícil enumerar el
departamento de un empleado dado (habrá que usar alguna otra técnica que
implique buscar en todos los departamentos). En el segundo caso tendremos el
problema inverso.

El modelo en red alivió el problema en cierto modo al permitir múltiples caminos
de acceso entre las instancias de datos (con lo que se podría decidir tener
ambos caminos desde departamento a empleado y viceversa). El problema consiste
en que es imposible predecir de antemano cuales serán los caminos de acceso y
por este motivo siempre habrá una discrepancia entre:

 - **Requisitos de recuperación primarios.** Éstos pueden ser previstos y
   satisfechos simplemente siguiendo los caminos de acceso definidos.
 - **Requisitos de recuperación secundarios.** Éstos puede ser no previstos o
   simplemente no tener un soporte específico y por lo tanto sólo pueden ser
   provistos mediante mecanismos alternativos como búsquedas.
   
Una de las razones principales por las que el modelo relacional prevaleció sobre
los modelos jerárquico y en red es la capacidad de *evitar* completamente el
concepto de camino de acceso.

También es interesante considerar brevemente lo que implica aplicar una
aproximación orientada a objetos (POO) a nuestro ejemplo. Podemos elegir entre
estas opciones:

 - Dar a los objetos `Employee` una referencia a su `Department`.
 - Dar a los objetos `Department` un conjunto (o array) de referencias a sus
   objetos `Employee`.
 - Todas las anteriores.
 
Si elegimos la tercera opción nos exponemos en el mejor caso al trabajo
adicional de mantener referencias redundantes y en el peor a introducir errores.

Hay inquietantes similitudes entre las aproximaciones de la POO y el XML a cómo
estructurar datos y los modelos jerárquico y en red.

Finalmente, una ventaja de las relaciones como forma de estructurar datos en
contraposición a aproximaciones como el modelo entidad-relación de Chen [Che76]
es que no se hace distinción alguna entre *entidad* y *relación*. Utilizar
dicha distinción puede ser problemático porque puede ser muy subjetivo
clasificar algo como *entidad* o como *relación*.

### 8.2 Manipulación

Codd introdujo dos mecanismos diferentes para expresar la manipulación de datos
en el modelo relacional: el cálculo relacional y el álgebra relacional. Ambos
son formalmente equivalentes en el sentido de que expresiones de cada uno pueden
convertirse a otras equivalentes del otro y por lo tanto nos bastará con
considerar solo el álgebra.

El álgebra relacional (en una forma ligeramente diferente de cómo lo presentó
Codd originalmente) consiste en las siguientes ocho operaciones:

 - **Restricción (restrict).** Operación unaria que permite la selección de un
   subconjunto de los registros de acuerdo a un criterio dado.
 - **Proyección (project).** Operación unaria que produce una nueva relación en
   la que algunos atributos han sido eliminados.
 - **Producto (product).** Operación binaria que se corresponde con el producto
   cartesiano tal y como se define en matemáticas.
 - **Unión (union).** Operación binaria que produce una relación que contiene
   todos los registros que figuran en uno u otro operando.
 - **Intersección (intersection).** Operación binaria que produce una relación
   con todos los registros que figuran en ambos operandos simultáneamente.
 - **Diferencia (difference).** Operación binaria que produce una relación con
   los registros del primer operando que no figuran en el segundo.
 - **Combinar (join).** Operación binaria que produce todos los posibles
   registros que resultan juntar registros de ambos operandos cuyo valor de
   ciertos atributos coincide.
 - **Dividir (divide).** Es una operación ternaria que devuelve todos los
   registros del primer argumento que aparecen en el segundo asociados con *cada
   uno* de los registros del tercero.
   
Uno de los beneficios más significativos de este lenguaje de manipulación
(aparte de su simplicidad) es que tiene la propiedad de *clausura*: todos los
operandos y resultados son de la misma clase (relaciones). Por lo tanto, las
operaciones se pueden anidar de forma arbitraria (lo que de hecho es inherente a
cualquier álgebra monotipo).

### 8.3 Integridad

La integridad en el modelo relacional se mantiene simplemente con especificar,
de forma puramente declarativa, una serie de restricciones que deben cumplirse
en todo momento.

Cualquier infraestructura que implemente el modelo relacional debe garantizar
que estas restricciones se cumplen, específicamente en los intentos de modificar
el estado que puedan contravenir las restricciones tienen que ser rechazadas o
restringidas a operar dentro de los límites.

Las restricciones más comunes son aquellas que definen claves *candidatas* o
*primarias* y claves *extranjeras*. Las restricciones pueden llegar a ser
arbitrariamente complejas, involucrar múltiples relaciones y ser construidas
utilizando tanto el álgebra como el cálculo relacional.

Finalmente, muchos sistemas gestores de bases de datos comerciales proveen
mecanismos *imperativos* como los disparadores (*triggers*) para mantener la
integridad de los datos. Estos mecanismos sufren de los problemas sobre flujo de
control comentados en la sección 4.2 y *no se consideran* parte del modelo
relacional.

### 8.4 Independencia de datos

La *independencia de datos* es el principio por el que se separa el modelo
lógico de los datos de la representación física almacenada, y fue una de las
motivaciones que originaron el modelo relacional.

Es interesante darse cuenta de que el principio de *independencia de datos* es
de hecho análogo a la separación entre lo accidental y lo esencial recomendada
en la sección 7.3.2. Esto mismo es una de las razones que motivan la adopción de
la Programación Relacional Funcional (ver sección 9).

### 8.5 Extensiones

A pesar de ser flexible, el álgebra relacional es un lenguaje restrictivo en
términos computacionales (no es Turing-completo) y de forma habitual es
extendido de diversas formas en sus implementaciones prácticas. Algunas
extensiones comunes son:

 - **Capacidades de computación generales.** Como por ejemplo aritmética simple
   y capacidad de utilizar funciones definidas por el usuario.
 - **Operadores de agregación.** Como `MAX`, `MIN`, `COUNT`, `SUM`, etc.
 - **Capacidades de agrupado y resumen.** Para poder aplicar fácilmente las
   operaciones de agregación.

## 9. Programación relacional funcional

## 10. Ejemplo de un sistema FRP

## 11. Trabajo relacionado

## 12. Conclusiones

Hemos presentado como la *complejidad* es la mayor fuente de problemas para los
grandes sistemas software. También hemos discutido cómo *puede* ser domesticada
pero sólo a través de un esfuerzo consciente de *evitarla* en la cuando es
posible y *separándola* cuando no lo es. Específicamente, hemos determinado que
un sistema puede ser fructíferamente *separado* en tres partes principales: el
*estado esencial*, la *lógica esencial* y el *estado y control accidentales*.

Creemos que la aplicación de estos principios al diseño del alto nivel de un
sistema (utilizando diferentes *lenguajes* especializados para los diferentes
componentes) puede ofrecer más en términos de simplicidad que la adopción no
estructura de un único lenguaje de propósito general cualquiera (bien
imperativo, lógico o funcional). La argumentación de este punto incluye un
repaso de cada uno de los paradigmas de programación más comunes con especial
atención a los puntos débiles de la orientación a objetos como un ejemplo
particular de la aproximación imperativa.

En aquellos casos (como en los grandes sistemas preexistentes) donde esta
*separación* no puede ser aplicada directamente creemos que se debe hacer
énfasis en evitar el estado, el control explícito en la medida de lo posible y
luchar por todos los medios de *librarse del código*.

Finalmente, ¿cómo salir del fango? ¿Cuál es la bala de plata? ...podría no ser
el FRP pero creemos indudablemente que es la *simplicidad*.

## Referencias

- [Bac78] John W. Backus. Can programming be liberated from the von Neumann
  style? a functional style and its algebra of programs. *Commun. ACM*,
  21(8):613–641, 1978.

- [Bak93] Henry G. Baker. Equal rights for functional objects or, the more
  things change, the more they are the same. *Journal of Object-Oriented
  Programming*, 4(4):2–27, October 1993.

- [Boo91] G. Booch. *Object Oriented Design with Applications*.
  Benjamin/Cummings, 1991.

- [Bro86] Frederick P. Brooks, Jr. No silver bullet: Essence and accidents of
  software engineering. *Information Processing 1986, Proceedings of the Tenth
  World Computing Conference*, H.-J. Kugler, ed.: 1069–76. Reprinted in IEEE
  Computer, 20(4):10-19, April 1987, and in Brooks, The Mythical Man-Month:
  Essays on Software Engineering, Anniversary Edition, Chapter 16,
  Addison-Wesley, 1995.

- [Che76] P. P. Chen. "The Entity-Relationship Model". *ACM Trans. on Database
  Systems (TODS)*, 1:9–36, 1976.

- [Cod70] E. F. Codd. A relational model of data for large shared data banks.
  *Comm. ACM*, 13(6):377–387, June 1970.

- [Cod79] E. F. Codd. Extending the database relational model to capture more
  meaning. *ACM Trans. on Database Sys.*, 4(4):397, December 1979.

- [Cod90] E. F. Codd. *The Relational Model for Database Management, Version
  2*.  Addison-Wesley, 1990.

- [Cor91] Fernando J. Corbató. On building systems that will fail. *Commun.
  ACM*, 34(9):72–81, 1991.

- [Dat04] C. J. Date. *An Introduction to Database Systems*. Addison Wesley,
  8th edition, 2004.

- [DD00] Hugh Darwen and C. J. Date. *Foundation for Future Database Systems:
  The Third Manifesto*. Addison-Wesley, 2nd edition, 2000.

- [Dij71] Edsger W. Dijkstra. On the reliability of programs. circulated
  privately, 1971.

- [Dij72] Edsger W. Dijkstra. The humble programmer. *Commun. ACM*,
  15(10):859–866, 1972.

- [Dij97] Dijkstra. The tide, not the waves. In Peter J. Denning and Robert M.
  Metcalfe, editors, *Beyond Calculation: The Next Fifty Years of Computing,
  Copernicus, 1997*. 1997.

- [Eco04] Managing complexity. *The Economist*, 373(8403):89–91, 2004.

- [EH97] Conal Elliott and Paul Hudak. Functional reactive animation. In
  *Proceedings of the ACM SIGPLAN International Conference on Functional
  Programming (ICFP-97)*, volume 32,8 of *ACM SIGPLAN Notices*, pages 263–273,
  New York, June 9–11 1997. ACM Press.

- [HJ89] I. Hayes and C. Jones. Specifications are not (necessarily) executable.
  *IEE Software Engineering Journal*, 4(6):330–338, November 1989.

- [Hoa81] C. A. R. Hoare. The emperor’s old clothes. *Commun. ACM*, 24(2):75–83,
  1981.

- [Kow79] Robert A. Kowalski. Algorithm = logic + control. *Commun. ACM*,
  22(7):424–436, 1979.

- [Mer85] T. H. Merrett. Persistence and Aldat. In *Data Types and Persistence
  (Appin)*, pages 173–188, 1985.

- [NR69] P.Naur and B.Randell. Software engineering report of a conference
  sponsored by the NATO science committee Garmisch Germany 7th-11th October
  1968, January 01 1969.

- [OB88] A. Ohori and P. Buneman. Type inference in a database programming
  language. In *Proceedings of the 1988 ACM Conference on LISP and Functional
  Programming, Snowbird, UT*, pages 174–183, New York, NY, 1988. ACM.

- [O’K90] Richard A. O’Keefe. *The Craft of Prolog*. The MIT Press, Cambridge,
  1990.

- [PJ+03] Simon Peyton Jones et al., editors. *Haskell 98 Language and
  Libraries, the Revised Report*. CUP, April 2003.

- [SS94] Leon Sterling and Ehud Y. Shapiro. *The Art of Prolog - Advanced
  Programming Techniques, 2nd Ed.* MIT Press, 1994.

- [SU96] Randall B. Smith and David Ungar. A simple and unifying approach to
  subjective objects. TAPOS, 2(3):161–178, 1996.

- [vRH04] Peter van Roy and Seif Haridi. *Concepts, Techniques, and Models of
  Computer Programming*. MIT Press, 2004.

- [Wad95] Philip Wadler. Monads for functional programming. In *Advanced
  Functional Programming*, pages 24–52, 1995.

- [Won00] Limsoon Wong. Kleisli, a functional query system. *J. Funct.
  Program*, 10(1):19–56, 2000.

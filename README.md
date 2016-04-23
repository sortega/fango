# Saliendo del pozo de  brea

Traducción al castellano del artículo "Out of the Tar Pit"

**Traducción en progreso**

Autores:

 - Ben Moseley ben@moseley.name
 - Peter Marks public@indigomail.net

Traducción:
 - Sebastián Ortega

## Resumen

El mayor obstáculo en el desarrollo de sistemas software de envergadura es la
complejidad. En consonancia con Brooks, distinguimos entre complejidad
accidental y esencial pero no aceptamos su premisa de que la mayoría de la
complejidad que se puede encontrar en los sistemas actuales es esencial.
Identificamos las causas más comunes de la complejidad y discutimos
aproximaciones generales para minimizar la complejidad de naturaleza
accidental. Para hacer más concreta la exposición, se bosqueja una potencial
aproximación a la minimización de complejidad basada en programación funcional
y el modelo de datos relacional de Codd.

## 1. Introducción

## 2. Complejidad

## 3. Formas de comprender un sistema

## 4. Causas de la complejidad

## 5. Aproximaciones clásicas a la gestión de la complejidad

## 6. Accidentes y esencia

## 7. Recomendaciones generales

## 8. Modelo relacional

A pesar de sus orígenes, el modelo relacional [Cod70] *en si mismo* no tiene
nada que ver con bases de datos. No es más que una forma elegante de
*estructurar* datos, una forma de *manipular* los mismos y  un mecanismo para
mantener estado *íntegro* y consistente. Estas características son aplicables
a estado y datos en cualquier contexto.

A parte de estas tres áreas [Cod79, section 2.1], [Dat04, p109], un cuarto
punto fuerte del modelo relacional es el énfasis en tener una marcada
separación entre la capa lógica y física del sistema. Esto se traduce en que
el diseño del modelo lógico (minimizando la complejidad) es atacado de forma
separada del diseño del modelo de almacenamiento físico y de la correspondencia
entre ambos[^15]. Este principio de denomina *independencia de datos* y es una
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
aproximación orientada a objetos (OOP) a nuestro ejemplo. Podemos elegir entre
estas opciones:

 - Dar a los objetos `Employee` una referencia a su `Department`.
 - Dar a los objetos `Department` un conjunto (o array) de referencias a sus
   objetos `Employee`.
 - Todas las anteriores.
 
Si elegimos la tercera opción nos exponemos en el mejor caso al trabajo
adicional de mantener referencias redundantes y en el peor a introducir errores.

Hay inquietantes similitudes entre las aproximaciones de la OOP y el XML a cómo
estructurar datos y los modelos jerárquico y en red.

Finalmente, una ventaja de las relaciones como forma de estructurar datos en contraposición a aproximaciones como el modelo entidad-relación de Chen [Che76] es que no se hace distinción alguna entre *entidad* y *relación*. (Utilizar dicha distinción puede ser problemático porque puede ser muy subjetivo clasificar algo como *entidad* o como *relación*.)

### 8.2 Manipulación

Codd introdujo dos mecanismos diferentes para expresar la manipulación de datos
en el modelo relacional: el cálculo relacional y el álgebra relacional. Ambos
son formalmente equivalentes en el sentido de que expresiones de cada uno pueden
convertirse a otras equivalentes del otro y por lo tanto nos bastará con
considerar solo el álgebra.

El álgebra relacional (en una forma ligeramente diferente de cómo lo presentó
Codd originalmente) consiste en las siguientes ocho operaciones:

 - **Restricción (restrict).** Operación unaria que permite la selección de un
   subconjunto
   de los registros de acuerdo a un criterio dado.
 - **Proyección (project).** Operación unaria que produce una nueva relación en
   la que
   algunos atributos han sido eliminados.
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
(aparte de su simplicidad) es que tiene la propiedad de *clausura*: todos los operandos y resultados son de la misma clase (relaciones). Por lo tanto, las operaciones se pueden anidar de forma arbitraria (lo que de hecho es inherente a cualquier álgebra monotipo).

### 8.3 Integridad

La integridad en el modelo relacional se mantiene simplemente con especificar,
de forma puramente declarativa, una serie de restricciones que deben cumplirse
en todo momento.

Cualquier infraestructura que implemente el modelo relacional debe garantizar
que estas restricciones se cumplen, específicamente en los intentos de modificar
el estado que puedan contravenir las restricciones tienen que ser rechazadas o
restringidas a operar dentro de los límites.

Las restricciones más comunes son aquellas identificando claves *candidatas* o
*primarias* y claves *extrajeras*. Las restricciones pueden llegar a ser
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

- [Che76] P. P. Chen. The Entity-Relationship Model. *ACM Trans. on Database
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
  Programming (ICFP-97)*, volume 32,8 of ACM SIGPLAN Notices, pages 263–273,
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
  Programming*, Snowbird, UT, pages 174–183, New York, NY, 1988. ACM.

- [O’K90] Richard A. O’Keefe. *The Craft of Prolog*. The MIT Press, Cambridge,
  1990.

- [PJ+03] Simon Peyton Jones et al., editors. *Haskell 98 Language and
  Libraries, the Revised Report*. CUP, April 2003.

- [SS94] Leon Sterling and Ehud Y. Shapiro. *The Art of Prolog - Advanced
  Programming Techniques*, 2nd Ed. MIT Press, 1994.

- [SU96] Randall B. Smith and David Ungar. A simple and unifying approach to
  subjective objects. TAPOS, 2(3):161–178, 1996.

- [vRH04] Peter van Roy and Seif Haridi. *Concepts, Techniques, and Models of
  Computer Programming*. MIT Press, 2004.

- [Wad95] Philip Wadler. Monads for functional programming. *In Advanced
  Functional Programming*, pages 24–52, 1995.

- [Won00] Limsoon Wong. Kleisli, a functional query system. *J. Funct.
  Program*, 10(1):19–56, 2000.

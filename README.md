# Saliendo del pozo de  brea

Traducción al castellano del artículo "Out of the Tar Pit"

**Traducción en progreso**

Autores:

 - Ben Moseley ben@moseley.name
 - Peter Marks public@indigomail.net

Traductores:
 - ???

## Resumen

El mayor obstáculo en el desarrollo de sistemas software de envergadura es la
complejidad. En consononacia con Brooks, distinguimos entre complejidad
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
*estruturar* datos, una forma de *manipular* los mismos y  un mecanismo para
mantener estado *íntegro* y consistente. Estas caracterísiticas son aplicables
a estado y datos en cualquier contexto.

A parte de estas tres áreas [Cod79, section 2.1], [Dat04, p109], un cuarto
punto fuerte del modelo relacional es el énfasis en tener una marcada
separación entre la capa lógica y física del sistema. Esto se traduce en que
el diseño del modelo lógico (minimizando la complejidad) es atacado de forma
separada del diseño del modelo de almcenamiento físico y de la correspondencia
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
#### 8.1.2 Beneficios estructurales de las relaciones - Independencia del camino de acceso
### 8.2 Manipulación
### 8.3 Integridad
### 8.4 Independencia de datos
### 8.5 Extensions

## 9. Programación relacional funcional

## 10. Exemplo de un sistema FRP

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
  Essays on Software Engineer- ing, Anniversary Edition, Chapter 16,
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
  Programming (ICFP-97)*, volume 32,8 of ACM SIG- PLAN Notices, pages 263–273,
  New York, June 9–11 1997. ACM Press.

- [HJ89] I. Hayes and C. Jones. Specifications are not (necessarily) exe-
  cutable. *IEE Software Engineering Journal*, 4(6):330–338, Novem- ber 1989.

- [Hoa81] C. A. R. Hoare. The emperor’s old clothes. *Commun. ACM*, 24(2):75–83,
  1981.

- [Kow79] Robert A. Kowalski. Algorithm = logic + control. *Commun. ACM*,
  22(7):424–436, 1979.

- [Mer85] T. H. Merrett. Persistence and Aldat. In *Data Types and Persis-
  tence (Appin)*, pages 173–188, 1985.

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

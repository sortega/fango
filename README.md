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

---
tipo: informe-preliminar
id: INF-001
fecha_entrega: 2026-09-11
autores:
  - Elmer Mosquera
  - Nataly Orozco
proyecto: Sweet Crush (Desafío I)
---

# Informe Preliminar: Análisis y Diseño de Sweet Crush

> *"Todo proyecto empieza con una conversación, no con un documento".* 
> Este informe detalla el proceso de pensamiento, los retos de diseño y las decisiones arquitectónicas tomadas por nuestro equipo (Nataly y yo) durante la primera fase del Desafío I.

## 📑 Índice
1. [Contextualización y Entorno de Trabajo](#1-contextualización-y-entorno-de-trabajo)
2. [De qué se trata el proyecto](#2-de-qué-se-trata-el-proyecto)
2. [Análisis del problema y consideraciones](#3-análisis-del-problema-y-consideraciones)

---

# Informe preliminar: análisis y diseño de Sweet Crush


![Nota de análisis preliminar ](../adjuntos/NOTA-ANALISIS.jpg) *Figura 1: Apuntes diluidos de la primera sesión de análisis estructurando DOMER y las reglas del tablero.*
## 1. Contextualización y entorno de trabajo

Este documento presenta el informe del análisis preliminar del proyecto descrito en VA-DESAFIO-I, del curso Informática II 2026-2, UdeA Regiones.

Mi compañera y yo trabajamos sobre dos sistemas operativos (Linux y Windows). Nuestra primera conversación sobre el reto fue el lunes 7 de septiembre, cuando discutimos las impresiones iniciales tras leer la guía. En esa sesión decidimos adoptar la metodología DOMER (Documentación Organizada, Metódica, Evolutiva y Rastreable) con Obsidian y GitHub, con el fin de conservar el porqué de cada decisión sin que se diluya en el código.

Para este desafío decidimos documentar el proceso al mismo tiempo que lo implementábamos. El repositorio se divide en `docs/` y `software/`, lo que separa la planeación lógica de la implementación en C++ y permite rastrear cada requisito desde su origen.

## 2. De qué se trata el proyecto

Sweet Crush es un juego de combinaciones tipo match-3. El tablero es una matriz rectangular de F filas por C columnas. Cada posición contiene una ficha de uno de seis tipos posibles. El jugador elimina fichas y el sistema detecta combinaciones de tres o más fichas iguales consecutivas en horizontal o vertical, las elimina, reorganiza lo que queda y rellena los huecos con fichas nuevas. Ese relleno puede producir combinaciones sin intervención del jugador, y el proceso se repite hasta que el tablero queda estable. A ese encadenamiento se le llama cascada.

La particularidad del reto está en cómo se almacena el tablero. Cada ficha ocupa exactamente 3 bits, sin alineación por ficha, en una secuencia continua de bits dentro de un arreglo dinámico de bytes. Los bits de distintas fichas comparten byte cuando la aritmética lo permite. Eso obliga a leer y escribir cada ficha con operadores a nivel de bits (`&`, `|`, `^`, `~`, `<<`, `>>`) y a manejar tres casos de frontera entre bytes según dónde caiga la ficha. La manipulación bitwise es el eje del desafío, no una restricción añadida.

El programa también debe permitir modificar la estructura del tablero durante la partida: agregar o eliminar filas y columnas en cualquier posición, no solo en los extremos. Esa modificación tiene que reflejarse en la memoria física reservada. Cuando el tablero se reduce, la memoria solo se libera si la ocupación cae por debajo del 65 % del último dimensionamiento.

El código se escribe en C++ dentro del entorno de desarrollo Qt, pero el programa es de consola. Qt se usa como editor y depurador, no como framework de interfaz gráfica. No se permite POO (`struct`, `class`, `template`), ni STL (`std::vector`, `std::string`, `std::map`), ni sintaxis de ANSI C. La solución se organiza en archivos `.h` y `.cpp` con funciones puras, punteros y arreglos dinámicos.

## 3. Análisis del problema y consideraciones

Esta sección recoge las nociones surgidas de la lectura preliminar de la guía. Tres puntos concentraron la discusión inicial:

- Cómo distinguir las fichas entre sí. Revisamos la forma del tablero, los seis tipos de ficha y la codificación en 3 bits. Nos quedó claro que los caracteres que se muestren en pantalla deben ser distinguibles en ASCII.


	```
	
							= & ) # & ) = & ) # & ) @ # % ) = &
							) # @ = % % ) = % & ) @ = & % % ) =
							% & ) @ = = & # @ = % % @ # % ) & =
							% & ) @ = # ) # @ = % % = & ) # & )
							@ # % ) ) # @ = % & ) = % ) # @ = %
							% ) = % & ) = = # % ) # @ = % % ) =
							% & ) @ = # % % ) = % ) = & ) @ = #
							= % ) = ) % # @ = % ) = @ ) # @ = %
							% ) = % & ) @ = ) # @ = % % ) = % &
							) @ = # % = ) = # % % ) = % & ) @ =
							# % % ) = = % & ) @ = # ) # @ = % %
	
	```

- Cómo recibir las entradas del usuario. Evaluamos dos alternativas: entrada por teclado (viable) y detección de clic (descartada por el entorno de ejecución). La terminal fue el factor decisivo.


- Cómo se ve el tablero de referencia. La guía menciona Sweet Crush como base. A partir de una búsqueda preliminar encontramos juegos similares y quedé con la impresión de que habría que mover fichas con el mouse. Esa suposición desvió parte de la reunión hacia una complejidad que el enunciado no exige. Después quedó claro que el juego no requiere arrastrar fichas: el jugador indica una posición y esa ficha se elimina.

El balance de la sesión fue más de preguntas que de respuestas. La complejidad aparente del reto nos resultó alta, en parte por la experiencia del semestre anterior con el desarrollo, aún pendiente, del tetris binario.

### 3.1 La barrera del paradigma restrictivo

Se nos pide pensar en términos de C++ estructurado. Dados los temas vistos hasta ahora en el curso, esa forma de construir software se daba por sentada. Las decisiones sobre cómo prescindir de `stl`, `string` y arreglos multidimensionales `[i][j]` han representado un reto de diseño que seguimos abordando. Los operadores bitwise, en cambio, son el mecanismo central de la solución, no una limitación.

### 3.2 Dudas resueltas en clase

En la sesión del 8 de septiembre aclaramos varios puntos que estaban abiertos después de la primera reunión:

- La clase `bitset` no está permitida.
- El umbral del 65 % aplica para reducir memoria física, y ese porcentaje se calcula respecto al último dimensionamiento, no respecto al tamaño original.
- Los seis tipos de ficha deben ser distinguibles en pantalla usando caracteres ASCII.
- Las funciones de eliminación deben permitir eliminar una ficha individual, eliminar una fila, eliminar una columna, agregar una fila y agregar una columna.
- Es posible usar otro tipo de dato distinto a `unsigned char` para la reserva de memoria, si se prefiere, siempre que el empaquetado siga siendo a nivel de bits.
- El programa es de consola. La mención de Qt en el enunciado se refiere al entorno de desarrollo y depuración, no a una interfaz gráfica.

Estas aclaraciones resolvieron dudas que habíamos dejado abiertas sobre las fronteras entre bytes cuando se elimina una fila y sobre el entorno de ejecución, y confirmaron la decisión de usar entradas por consola para la jugabilidad.

## 4. Conclusiones

Hasta aquí llega el informe preliminar. Con las dudas de la sesión del 8 resueltas, el equipo se encuentra, mediante DOMER, definiendo la estructura del proyecto y los primeros documentos de arquitectura y requisitos. Ese trabajo excede el alcance del informe y corresponde a la metodología de implementación y seguimiento que adoptamos como parte de nuestra formación.

Solo ponermos usar iostream

# ADR - Restricción de librerías

## Contexto
El enunciado prohíbe expresamente el uso de struct, class, template,
objetos definidos por el estudiante, y estructuras dinámicas de librerías
externas (incluyendo la STL). No se permite el uso de objetos string.

## Decisión
Permitimos únicamente:
- `<iostream>` — entrada/salida por consola (cin, cout).
- `<cstdlib>` — generación de números aleatorios (rand, srand) y
  utilidades de memoria 
  new/delete por ser C++).
- `<ctime>` — semilla para el generador aleatorio (time()).

Explícitamente prohibido: <string>, cualquier cabecera de la STL
(<vector>, <map>, <algorithm>, <memory>, etc.), y cualquier librería que
provea contenedores o estructuras dinámicas ya resueltas.

## Justificación
El objetivo del desafío es implementar la gestión de memoria y las
estructuras de datos manualmente, no delegarlas a una librería. iostream,
cstdlib y ctime son utilidades básicas de entrada/salida y aleatoriedad
que no comprometen ese objetivo, ya que no proveen contenedores ni
estructuras de datos.
# Sweet Crush — Descripción del producto

## Qué es
Sweet Crush es un juego de tipo "match-3": el jugador elimina fichas de un
tablero rectangular y, cuando quedan 3 o más fichas iguales alineadas
(horizontal o verticalmente), se eliminan automáticamente, el tablero se
reacomoda y pueden generarse nuevas combinaciones en cascada.

## Tablero
- Rectangular, F filas × C columnas, definidas al iniciar la ejecución.
- Cada posición almacena exactamente 3 bits (ver [[codificacion-de-fichas]]
  y el ADR correspondiente).
- Se muestra en dos formatos: binario (secuencia de bits/bytes) y visual
  (representación por fichas), según piden los criterios de evaluación.

## Fichas y estados
6 tipos de ficha (A-F) + 2 estados adicionales usando los códigos sobrantes
de 3 bits. Propuesta inicial (a confirmar con Elmer):

| Código | Significado |
|--------|-------------|
| 000    | Ficha A |
| 001    | Ficha B |
| 010    | Ficha C |
| 011    | Ficha D |
| 100    | Ficha E |
| 101    | Ficha F |
| 110    | Posición vacía |
| 111    | Ficha especial / bloqueada |

## Mecánica de juego
1. El jugador selecciona una posición del tablero y elimina la ficha ahí.
2. El sistema revisa si esa eliminación generó una combinación válida
   (3+ fichas iguales, horizontal o vertical).
3. Si hay combinación, se eliminan las fichas involucradas (una ficha puede
   pertenecer a una combinación horizontal y vertical simultáneamente).
4. Las fichas restantes "caen" verticalmente para llenar los espacios
   vacíos; se generan fichas nuevas aleatoriamente para completar el tablero.
5. Se revisa de nuevo si la caída generó nuevas combinaciones (cascada);
   se repite el proceso hasta que no haya más combinaciones automáticas.

## Modificación dinámica del tablero
Durante la partida se puede:
- Agregar/eliminar una fila en cualquier posición (no solo en los extremos).
- Agregar/eliminar una columna en cualquier posición.

Estas operaciones reorganizan físicamente la memoria del tablero (no solo
lógicamente), ver ADR de gestión de memoria.

## Estado del juego (a trackear)
- Dimensiones actuales del tablero
- Cantidad de eliminaciones hechas por el usuario
- Cantidad total de fichas eliminadas
- Cantidad de combinaciones detectadas
- Cantidad de cascadas de la eliminación actual

## Objetivo pedagógico (contexto del desafío)
Práctica de operadores bitwise, gestión de memoria dinámica y programación
estructurada (sin struct/class/template/STL/string), en C++ con Qt.

## Aclaraciones de clase (8 sept)
- La codificación de bits para las 6 fichas es libre; no hay un mapeo obligatorio.
- Agregar fila/columna → siempre dispara redimensionamiento de memoria.
- Eliminar fila/columna → redimensiona solo si el uso cae bajo el 65%.
- El tablero se debe poder visualizar en dos formatos: bits crudos y fichas.
- Al eliminar una ficha, las fichas de las filas superiores caen para llenar el hueco.
- Las combinaciones se detectan y eliminan automáticamente, incluyendo cascadas.


## Visualización
El tablero se muestra por consola (texto), en dos formatos:
1. **Binario**: la secuencia cruda de bits/bytes tal como está almacenada
   en memoria (útil para verificar que el empaquetado es correcto).
2. **Fichas**: representación legible para el jugador, usando un carácter
   o símbolo distinto por cada tipo de ficha/estado (ej: A, B, C... o
   símbolos ASCII), organizados en filas y columnas como una cuadrícula
   de texto.

No hay interfaz gráfica (GUI). El proyecto usa Qt/CMake como entorno de
build y organización del código, pero toda la interacción con el jugador
(selección de posiciones, menús, mensajes) es por consola.
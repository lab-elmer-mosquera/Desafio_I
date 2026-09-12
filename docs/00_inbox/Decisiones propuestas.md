Son 6 fichas.

| Código | Significado    |
| ------ | -------------- |
| 000    | Posición vacía |
| 001    | # (35)         |
| 010    | & (38)         |
| 011    | @ (64)         |
| 100    | ) (41)         |
| 101    | = (61)         |
| 110    | % (36)         |
| 111    |                |
### Tablero de ejemplo en caracteres

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

## Funciones de visualización

- Vista de bist
- Vista de tablero
---
## Tipo de dato base

unsigned char \*

¿Porque unsigned char \*?

Se usa `unsigned char*` como tipo base del buffer del tablero porque garantiza tamaño de 1 byte, evita comportamiento indefinido/inesperado de extensión de signo al hacer shifts (`>>`, `<<`), y es el tipo estándar en C++ para representar bytes crudos manipulados a nivel de bits.


FORMAS DE ALEATORIEDAD QUE PODEMOS USAR, PROPUESTA:


Opción A: Usar `QRandomGenerator` de Qt (La opción recomendada)

El framework Qt incluye la clase `QRandomGenerator`, la cual utiliza internamente semillas del sistema operativo para generar números pseudoaleatorios de alta calidad con distribución uniforme.

- **¿Cómo se usa?:**

```
#include <QRandomGenerator>

// Genera un entero aleatorio uniforme en el rango [3]
unsigned char ficha_aleatoria = QRandomGenerator::global()->bounded(0, 6);
```

- **Ventajas:** Cumple al 100% con la restricción de usar Qt y evitar sintaxis  tradicional, no requiere inicializar semillas manualmente y garantiza una distribución uniforme.

Opción B: Usar `<random>` de C++11 (`std::mt19937`)

Podemos mantenernos dentro del estándar C++ puro sin recurrir a métodos de objetos Qt para este módulo, podemos utilizar el motor **Mersenne Twister** junto a una distribución uniforme de enteros.
El motor Mersenne Twister (std::mt19937) es un algoritmos de generación de números pseudoaleatorios.

- **¿Cómo se usa?:**

```
#include <random>

// Generador y distribución (se pueden inicializar una sola vez)
std::random_device rd;
std::mt19937 gen(rd());
std::uniform_int_distribution<int> dist(0, 5);

unsigned char ficha_aleatoria = dist(gen);
```

- **Ventajas:** Es el estándar moderno de C++ para aleatoriedad estadística precisa.  teniendo en cuenta que no podemos usar contenedores o clases no permitidas, solo los tipos primarios de la cabecera.

Opción C: Implementar un Generador Congruencial Lineal (LCG) propio

Consiste en programar nuestra propia función matemática de aleatoriedad basada en la fórmula $X_{n+1} = (a \cdot X_n + c) \bmod m$.

- **¿Cómo se usa?:**

```
static unsigned int semilla = 123456789; // Cambia con el tiempo de la máquina

unsigned char generar_ficha_lcg() {
    semilla = (1103515245 * semilla + 12345) % 2147483648;
    return (semilla / 65536) % 6; // Mapea a un valor entre 0 y 5
}
```

- **Ventajas:** Muestra un dominio absoluto de la programación desde cero sin depender de librerías externas.

---

2. Estrategias para poblar el tablero con las fichas generadas

Una vez obtenido el código de la ficha (un número entre `000` y `101`), existen dos formas de aplicar esta aleatoriedad a la lógica de juego:

Estrategia 1: Generación Pura / Directa (Tablero Caótico)

Rellena cada celda del tablero secuencialmente asignando un valor aleatorio independiente.

- **Comportamiento:** Al arrancar el tablero de $F \times C$, es muy probable que ya existan combinaciones de 3 o más fichas iguales alineadas.
- **Manejo:** dejar que la lógica de cascadas del juego se active inmediatamente desde el inicio, destruyendo las combinaciones iniciales y haciendo "caer" nuevas fichas automáticamente hasta que el tablero se estabilice antes del primer turno del usuario.

Estrategia 2: Generación Controlada / Sin Combinaciones Iniciales

Genera un tablero inicial listo para jugar, asegurando que ninguna ficha cree un trío horizontal o vertical al momento de colocarse.

- **Algoritmo:**
    1. Para la celda, fila,columna se genera una ficha aleatoria.
    2. Se verifica si coincide con las 2 fichas a su izquierda o con las 2 fichas arriba de ella que ya fueron colocadas.
    3. Si coincide y forma un trío, se vuelve a generar la ficha o se elige la siguiente hasta que sea válida.
- **Ventaja:** Garantiza que el jugador inicie la partida con un tablero donde **él** tenga que hacer el primer movimiento.

---

3. Integración con el empaquetado de bits

Independientemente del método aleatorio que se elija, el valor devuelto (un entero entre `0` y `5`) debe guardarse usando los **operadores a nivel de bits** (`<<`, `|`, etc.) para empaquetarlo en el arreglo dinámico `unsigned char*`.


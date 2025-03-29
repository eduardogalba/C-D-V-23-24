# Proyecto C (Comprime) D (Descomprime) V (Verifica)
## Descripción del Proyecto
Este proyecto consiste en la implementación de un compresor de texto elemental en lenguaje ensamblador. El objetivo es reducir el tamaño de un texto dado mediante la identificación y sustitución de cadenas de caracteres repetidas, utilizando un método sencillo de compresión.

## Método de Compresión
El proceso de compresión se basa en los siguientes principios:
1. Identificación de cadenas repetidas: El algoritmo busca secuencias de caracteres que ya hayan aparecido previamente en el texto.
2. Sustitución por referencias: Las cadenas repetidas se reemplazan por una referencia a su posición anterior (`P`) y su longitud (`L`).
- `P` se almacena como un valor de 16 bits (sin signo) que indica la distancia desde el inicio del texto.
- `L` se almacena como un valor de 8 bits (sin signo) que representa la longitud de la cadena (entre 1 y 255 caracteres).
3. Uso de un mapa de bits: Un mapa de bits (`vb[i]`) indica si un carácter se copia directamente (`vb[i] = 0`) o forma parte de una cadena referenciada (`vb[i] = 1`).
4. Parámetros configurables:
- `M`: Define el número inicial de caracteres que siempre se copian sin comprimir (múltiplo de 8).
- `N`: Longitud mínima de una cadena para que sea comprimida (en este proyecto, `M = 1` y `N = 4`).
## Estructura del Texto Comprimido
El texto comprimido se almacena en una estructura con la siguiente organización:
1. Cabecera (5 bytes):
- 2 bytes: Longitud del texto original sin comprimir (máximo 65535 caracteres).
- 1 byte: Valor de `M` (almacenado como `M/8`).
- 2 bytes: Desplazamiento al inicio del vector de bytes (`vBYT[j]`).
2. Mapa de bits (`vb[i]`):
- Indica cómo se han comprimido los caracteres (directamente o por referencia).
- Los primeros `M` bits son implícitamente `0` y no se almacenan.
3. Vector de bytes (`vBYT[j]`):
- Contiene caracteres individuales (si `vb[i] = 0`) o referencias a cadenas previas (si `vb[i] = 1`).

# Representación de la Figura 2 - Compresión de Texto

## Estructura del Texto Comprimido
**Ejemplo con parámetros:**
- `M = 8` (primeros 8 caracteres sin comprimir)
- `N = 6` (longitud mínima para compresión)
- Texto original: 24 caracteres

### 1. Texto Original (`v.entrada`)
"Como quieres que te quiera, si el que quiero que no me quiera..."
### 2. Texto Comprimido (`v.salida`)

#### Cabecera (5 bytes)
| Posición | Contenido          | Valor (ejemplo) | Descripción                     |
|----------|--------------------|-----------------|---------------------------------|
| 0-1      | Longitud original  | `0x0041`        | 65 caracteres (16 bits)         |
| 2        | M/8                | `0x01`          | M=8 → 8/8 = 1                   |
| 3-4      | Offset a `vBYT[j]`  | `0x000C`        | Desplazamiento = 12 bytes       |

#### Mapa de Bits (`vb[i]`)
- **7 bytes** (para 50 bits)
- Los primeros 8 caracteres se copian directamente. El mapa empieza en la posición 8.
- Bits:
  - `0`: Carácter copiado directamente
  - `1`: Inicio de cadena comprimida


# Representación de la Compresión con Tablas

## 1. Texto Original 
| Pos | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10| 11| 12| 13| 14| 15| 16| 17| 18| 19|
|-----|---|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|----|----|----|
| Chr| C | O | M | O |   | Q | U | I | E | R | E | S |   | Q | U | E |   | T | E |   |
| Col| 🟧 | 🟧 | 🟧 | 🟧 | 🟧 | 🟧 | 🟧 | 🟧 | ⬜️ | ⬜️ | ⬜️ | ⬜️ | 🟩 | 🟩 | 🟩 | 🟩 | 🟩 | ⬜️ | 🟦 | 🟦 |

| Pos | 20| 21| 22| 23| 24| 25| 26| 27| 28| 29| 30| 31| 32| 33| 34| 35| 36| 37| 38| 39|
|-----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
| Chr| Q | U | I | E | R | A | , |   | S | I |   | E | L |   | Q | U | E |   | Q | U |
| Col| 🟦 | 🟦 | 🟦 | 🟦 | 🟦 | 🟦 | ⬜️ | ⬜️ | ⬜️ | ⬜️ | ⬜️ | ⬜️ | ⬜️ | 🟩 | 🟩 | 🟩 | 🟩 | 🟩 | ⬜️ | ⬜️ |

| Pos | 40| 41| 42| 43| 44| 45| 46| 47| 48| 49| 50| 51| 52| 53| 54| 55| 56| 57| 58| 59|
|-----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
| Chr| I | E | R | O |   | Q | U | E |   | N | O |   | M | E |    | Q | U | I | E | R |
| Col| ⬜️ | ⬜️ | ⬜️ | ⬜️ | 🟩 | 🟩 | 🟩 | 🟩 | 🟩 | ⬜️ | ⬜️ | ⬜️ | ⬜️ | 🟦 | 🟦 | 🟦 | 🟦 | 🟦 | 🟦 | 🟦 |

| Pos | 60| 61| 62| 63|
|-----|----|----|----|----|
| Chr| A | . | . | . |
| Col| 🟦 | ⬜️ | ⬜️ | ⬜️ |

**Leyenda:**
- 🟧 8 primeros caracteres
- ⬜️ Caracteres copiados directamente
- 🟩 " que " 
- 🟦 "e quiera"

## 2. Texto Comprimido
| Pos | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10| 11| 12| 13| 14| 15| 16| 17| 18| 19|
|-----|---|---|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|----|----|----|
| Chr| C | O | M | O |   | Q | U | I | E | R | E | S |   | Q | U | E |   | T | E |   |
| Col| 🟧 | 🟧 | 🟧 | 🟧 | 🟧 | 🟧 | 🟧 | 🟧 | ⬜️ | ⬜️ | ⬜️ | ⬜️ | 🟩 | 🟩 | 🟩 | 🟩 | 🟩 | ⬜️ | 🟦 | 🟦 |

| Pos | 20| 21| 22| 23| 24| 25| 26| 27| 28| 29| 30| 31| 32| 33| 34| 35| 36| 37| 38| 39|
|-----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
| Chr| Q | U | I | E | R | A | , |   | S | I |   | E | L | **4**  | **5** | Q | U | I | E | R |
| Col| 🟦 | 🟦 | 🟦 | 🟦 | 🟦 | 🟦 | ⬜️ | ⬜️ | ⬜️ | ⬜️ | ⬜️ | ⬜️ | ⬜️ | 🟩 | 🟩 | ⬜️ | ⬜️ | ⬜️ | ⬜️ | ⬜️ |

| Pos | 40| 41| 42| 43| 44| 45| 46| 47| 48| 49| 50| 51|
|-----|----|----|----|----|----|----|----|----|----|----|----|----|
| Chr| O | **4** | **5** | N | O |   | M | **10** | **8** | . | . | . |
| Col| ⬜️ | 🟩 | 🟩 | ⬜️ | ⬜️ | ⬜️ | ⬜️ | 🟦 | 🟦 | ⬜️ | ⬜️ | ⬜️ |

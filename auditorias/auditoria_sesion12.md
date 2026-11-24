# Bitácora de Requerimientos y Auditoría IA — Sesión 12
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Atributos de Símbolos y Memoria  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Ingrid Paola Dzul Segovia]  
**Matrícula:** [No. 24531350]  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)

*Reto:* Estructura de registro de símbolos

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> La principal dificultad fue comprender cómo calcular los offsets de memoria dependiendo del tamaño de cada tipo de dato. Tuve que utilizar una fórmula para ajustar el cursor y respetar la alineación correspondiente. También agregué condiciones para validar los datos incorrectos, los registros nulos y los cursores negativos.

### Código del Artesano

```c
#include "api_sesion12.h"
#include <stdio.h>
#include <string.h>

int ancho_de_tipo(const char *tipo) {
    if (tipo == NULL) {
        return -1;
    }

    if (strcmp(tipo, "char") == 0) { return 1; }
    if (strcmp(tipo, "short") == 0) { return 2; }
    if (strcmp(tipo, "int") == 0) { return 4; }
    if (strcmp(tipo, "float") == 0) { return 4; }
    if (strcmp(tipo, "double") == 0) { return 8; }
    if (strcmp(tipo, "puntero") == 0) { return 8; }

    return -1;
}

int asignar_offset(Simbolo *simbolo, int cursor) {
    if (simbolo == NULL || cursor < 0) {
        return -1;
    }

    int ancho = ancho_de_tipo(simbolo->tipo);

    if (ancho <= 0) {
        return -1;
    }

    /* Redondea el cursor hacia arriba al
       siguiente múltiplo del ancho */

    int offset = ((cursor + ancho - 1) / ancho) * ancho;

    simbolo->offset = offset;

    return offset + ancho;
}
```

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Lenguaje C utilizando GCC, sin bibliotecas no estándar. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores. |
| **Especificación (E)** | Calcular el tamaño de los tipos de datos, sus desplazamientos y su alineación en memoria. |
| **Output (O)** | Código modular y análisis de complejidad, compatible con el harness de la sesión 12. |

### 2.2 Prompt Estructurado

```text
Actúa como un ingeniero de software de sistemas
y compiladores.

Genera un programa en lenguaje C que utilice
el archivo api_sesion12.h.

Implementa la función ancho_de_tipo para obtener
el tamaño en bytes de los tipos char, short,
int, float, double y puntero.

Implementa la función asignar_offset para calcular
el desplazamiento de memoria de cada símbolo,
respetando la alineación según su tamaño.

Valida los registros nulos, los tipos desconocidos
y los valores negativos del cursor.

El código debe ser modular, no incluir la función
main y ser compatible con el harness proporcionado.

Incluye el análisis de complejidad temporal
y espacial de las funciones.
```

---

## 3. Código Devuelto por el Modelo

```c
/* ia_sesion12_c.c - Fase 2: El Científico */

#include "api_sesion12.h"
#include <string.h>

int ancho_de_tipo(const char *tipo) {
    if (tipo == NULL) { return -1; }
    if (strcmp(tipo, "char") == 0) { return 1; }
    if (strcmp(tipo, "short") == 0) { return 2; }
    if (strcmp(tipo, "int") == 0) { return 4; }
    if (strcmp(tipo, "float") == 0) { return 4; }
    if (strcmp(tipo, "double") == 0) { return 8; }
    if (strcmp(tipo, "puntero") == 0) { return 8; }
    return -1;
}

int asignar_offset(Simbolo *simbolo, int cursor) {
    if (simbolo == NULL || cursor < 0) {
        return -1;
    }

    int ancho = ancho_de_tipo(simbolo->tipo);

    if (ancho <= 0) { return -1; }

    int offset = ((cursor + ancho - 1) / ancho) * ancho;

    simbolo->offset = offset;

    return offset + ancho;
}
```

### Análisis de complejidad

| Función | Complejidad temporal | Complejidad espacial |
| :--- | :---: | :---: |
| ancho_de_tipo | O(1) | O(1) |
| asignar_offset | O(1) | O(1) |

Las funciones tienen complejidad constante para
los nombres de tipos contemplados en la práctica,
debido a que realizan una cantidad limitada de
comparaciones y operaciones aritméticas, sin
utilizar estructuras de memoria adicionales
que crezcan con la cantidad de símbolos.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Alineación de tipos y tamaños en bytes.

### Resultado del Artesano

Comando de compilación y ejecución:

```bash
gcc -std=c11 -Wall -Wextra -Werror harness_sesion12_c.c artesano_sesion12_c.c -o test_artesano && ./test_artesano
```

| Caso | Esperado | Obtenido | Resultado |
| :--- | :---: | :---: | :---: |
| ancho_de_tipo("int") | 4 | 4 | OK |
| ancho_de_tipo("double") | 8 | 8 | OK |
| int en cursor 0 | 4 | 4 | OK |
| char e int: offset de b | 4 | 4 | OK |
| double desde cursor 4 | 8 | 8 | OK |
| registro NULL | -1 | -1 | OK |

**RESULTADO: 6/6 pruebas superadas.**

**TODO EN VERDE.**

### Resultado del Científico

Comando de compilación y ejecución:

```bash
gcc -std=c11 -Wall -Wextra -Werror harness_sesion12_c.c ia_sesion12_c.c -o test_ia && ./test_ia
```

| Caso | Esperado | Obtenido | Resultado |
| :--- | :---: | :---: | :---: |
| ancho_de_tipo("int") | 4 | 4 | OK |
| ancho_de_tipo("double") | 8 | 8 | OK |
| int en cursor 0 | 4 | 4 | OK |
| char e int: offset de b | 4 | 4 | OK |
| double desde cursor 4 | 8 | 8 | OK |
| registro NULL | -1 | -1 | OK |

**RESULTADO: 6/6 pruebas superadas.**

**TODO EN VERDE.**

### Comparación entre el Artesano y el Científico

| Característica | Artesano | Científico |
| :--- | :---: | :---: |
| Pruebas realizadas | 6 | 6 |
| Pruebas superadas | 6 | 6 |
| Pruebas fallidas | 0 | 0 |
| Errores de compilación | 0 | 0 |
| Porcentaje de aprobación | 100% | 100% |
| Complejidad temporal | O(1) | O(1) |
| Complejidad espacial | O(1) | O(1) |

### Lista de verificación

* [x] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

**Observación:** No se observaron problemas de
memoria durante las pruebas ejecutadas. Sin
embargo, no se realizó un análisis especializado
para confirmar la ausencia de accesos fuera de
límites o fugas de memoria.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Durante las pruebas no encontré errores en el código del Científico, ya que obtuvo los mismos resultados que el Artesano. Ambos códigos utilizaron la misma fórmula para calcular la alineación de memoria y realizaron las mismas validaciones. La diferencia principal fue que el código del Científico era más compacto, mientras que el Artesano incluía comentarios para comprender mejor el procedimiento.

> Un caso borde que no fue evaluado es cuando el cursor tiene un valor demasiado grande, lo que podría provocar un desbordamiento de enteros. No fue necesario modificar el código para superar las pruebas proporcionadas, pero sería conveniente agregar esta validación en una implementación futura.

---

**Firma digital:** Paola-23-ux

# Bitácora de Requerimientos y Auditoría IA — Sesión 27
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Indexación de Arreglos en Código Intermedio  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Desplazamientos para arreglos 1D y 2D  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Lo que más se me complicó fue **el orden de los temporales**: en 1D son
> dos (`_t0` para el offset, `_t1` para la lectura), pero en 2D son
> **cuatro** (`_t0` el `i * filas`, `_t1` la suma con `j`, `_t2` el offset
> por el ancho y `_t3` la lectura). En mi primer intento usaba el mismo
> `_t0` para todo y los casos 4 y 5 salían pisándose.  
> Otro error: en 2D la fórmula es **`i * filas + j`**, en ese orden; si la
> intercambio (`j * filas + i`) leo otra celda del arreglo.  
> Y lo del **ancho**: el offset es `índice * ancho`, no solo el índice —
> sin esa multiplicación el caso 1 me daba `_t0 = i` en vez de
> `_t0 = i * 4`.  
> Por último, si `base` viene en `NULL` hay que devolver **0 sin emitir
> nada**, no escribir líneas a medias.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Cálculo de desplazamiento (offset) |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto: Estoy en la sesión 27, generando código intermedio (TAC) para
accesos a arreglos. Necesito emitir en un arreglo de `max` filas de 64
caracteres las instrucciones que calculan el desplazamiento de un índice y
leen la celda, para arreglos de una y dos dimensiones. Los temporales se
numeran desde *temporales. Compilo en C11 con GCC con -Wall -Wextra -Werror
y sin bibliotecas fuera del estándar.

Rol: Que me ayudes como ingeniero de compiladores escribiendo la traducción
de accesos A[i] y M[i][j] a direcciones de memoria.

Especificación: Dos funciones con la firma EXACTA de api_sesion27.h, sin
main():
  int emitir_acceso(char lineas[][64], int max, const char *base,
                    const char *indice, int ancho, int *temporales);
  int emitir_acceso_2d(char lineas[][64], int max, const char *base,
                       const char *i, const char *j, int filas, int ancho,
                       int *temporales);

emitir_acceso() (1D) — escribe desde l[0] estas DOS líneas:
  1) "_tK = <indice> * <ancho>"   offset = índice * ancho del elemento
     (ejemplo: "_t0 = i * 4")
  2) "_tK+1 = <base>[_tK]"        lectura indexada
     (ejemplo: "_t1 = A[_t0]")
  y devuelve el número de líneas emitidas (2).

emitir_acceso_2d() (2D) — escribe desde l[0] estas CUATRO líneas:
  1) "_tK = <i> * <filas>"        (ejemplo: "_t0 = i * 3")
  2) "_tK+1 = _tK + <j>"          índice lineal i * filas + j, EN ESE ORDEN
  3) "_tK+2 = _tK+1 * <ancho>"    el offset se multiplica por el ancho
     (ejemplo: "_t2 = _t1 * 4")
  4) "_tK+3 = <base>[_tK+2]"      (ejemplo: "_t3 = M[_t2]")
  y devuelve 4.

Reglas comunes:
 - Cada temporal nuevo incrementa *temporales; nunca repitas nombres entre
   accesos (los dos accesos seguidos siguen donde quedó el contador).
 - Si base viene NULL: NO se emite ninguna línea y la función devuelve 0.
 - Nunca escribas más de `max` líneas (checa el índice antes de cada
   snprintf con 64).

Output: Solo las dos funciones en C99, sin main(), para pegar en
ia_sesion27_c.c.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion27_c.c - Fase 2: El Científico */
#include "api_sesion27.h"
#include <stdio.h>
#include <string.h>

static int agregar(char lineas[][64], int max, int *idx, const char *txt) {
    if (*idx < 0 || *idx >= max) { return 0; }   /* respeta el límite */
    snprintf(lineas[*idx], 64, "%s", txt);
    (*idx)++;
    return 1;
}

int emitir_acceso(char lineas[][64], int max, const char *base,
                  const char *indice, int ancho, int *temporales) {
    if (lineas == NULL || base == NULL || indice == NULL ||
        temporales == NULL || max < 2) {
        return 0;                     /* base NULL = sin acceso */
    }
    int t0 = (*temporales)++;
    int t1 = (*temporales)++;
    char b0[64], b1[64];
    int idx = 0;

    snprintf(b0, sizeof b0, "_t%d = %s * %d", t0, indice, ancho);
    snprintf(b1, sizeof b1, "_t%d = %s[_t%d]", t1, base, t0);
    agregar(lineas, max, &idx, b0);
    agregar(lineas, max, &idx, b1);
    return idx;
}

int emitir_acceso_2d(char lineas[][64], int max, const char *base,
                     const char *i, const char *j, int filas, int ancho,
                     int *temporales) {
    if (lineas == NULL || base == NULL || i == NULL || j == NULL ||
        temporales == NULL || max < 4) {
        return 0;
    }
    int t0 = (*temporales)++;
    int t1 = (*temporales)++;
    int t2 = (*temporales)++;
    int t3 = (*temporales)++;
    char b[64];
    int idx = 0;

    snprintf(b, sizeof b, "_t%d = %s * %d", t0, i, filas);
    agregar(lineas, max, &idx, b);
    snprintf(b, sizeof b, "_t%d = _t%d + %s", t1, t0, j);
    agregar(lineas, max, &idx, b);
    snprintf(b, sizeof b, "_t%d = _t%d * %d", t2, t1, ancho);
    agregar(lineas, max, &idx, b);
    snprintf(b, sizeof b, "_t%d = %s[_t%d]", t3, base, t2);
    agregar(lineas, max, &idx, b);
    return idx;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Accesos indexados a arreglos multidimensionales

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Salida literal del harness (6/6 OK)
```text
=== AUDITORIA SESION 27 - Desplazamientos para arreglos 1D y 2D ===
Caso                               esperado:              obtenido:              resultado
[ 1] 1D: i * 4                          esperado: _t0 = i * 4  obtenido: _t0 = i * 4  OK
[ 2] 1D: A[_t0]                         esperado: _t1 = A[_t0] obtenido: _t1 = A[_t0] OK
[ 3] 2D: i * 3                          esperado: _t0 = i * 3  obtenido: _t0 = i * 3  OK
[ 4] 2D: offset por ancho
     esperado: _t2 = _t1 * 4
     obtenido: _t2 = _t1 * 4
     -> OK
[ 5] 2D: M[_t2]                         esperado: _t3 = M[_t2] obtenido: _t3 = M[_t2] OK
[ 6] base NULL (sin acceso)             esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion27.md
```

### Corrida real desde Colab (evidencia)

**Celda 1 — `artesano_sesion27_c.c`:**

```text
=== AUDITORIA SESION 27 - Desplazamientos para arreglos 1D y 2D ===
Caso                               esperado:              obtenido:              resultado
[ 1] 1D: i * 4                          esperado: _t0 = i * 4  obtenido: _t0 = i * 4  OK
[ 2] 1D: A[_t0]                         esperado: _t1 = A[_t0] obtenido: _t1 = A[_t0] OK
[ 3] 2D: i * 3                          esperado: _t0 = i * 3  obtenido: _t0 = i * 3  OK
[ 4] 2D: offset por ancho
     esperado: _t2 = _t1 * 4
     obtenido: _t2 = _t1 * 4
     -> OK
[ 5] 2D: M[_t2]                         esperado: _t3 = M[_t2] obtenido: _t3 = M[_t2] OK
[ 6] base NULL (sin acceso)             esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion27.md
```

**Celda 2 — `ia_sesion27_c.c`:**

```text
=== AUDITORIA SESION 27 - Desplazamientos para arreglos 1D y 2D ===
Caso                               esperado:              obtenido:              resultado
[ 1] 1D: i * 4                          esperado: _t0 = i * 4  obtenido: _t0 = i * 4  OK
[ 2] 1D: A[_t0]                         esperado: _t1 = A[_t0] obtenido: _t1 = A[_t0] OK
[ 3] 2D: i * 3                          esperado: _t0 = i * 3  obtenido: _t0 = i * 3  OK
[ 4] 2D: offset por ancho
     esperado: _t2 = _t1 * 4
     obtenido: _t2 = _t1 * 4
     -> OK
[ 5] 2D: M[_t2]                         esperado: _t3 = M[_t2] obtenido: _t3 = M[_t2] OK
[ 6] base NULL (sin acceso)             esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion27.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La primera propuesta que me devolvió **usaba el índice directo como
> offset**, sin multiplicar por el ancho: el caso 1 salía `_t0 = i` en vez
> de `_t0 = i * 4` y el caso 4 (`_t2 = _t1 * 4`) salía `_t2 = _t1`, así
> que leería la celda equivocada. Además no chequeaba `base == NULL` y con
> la base nula seguía escribiendo líneas y devolvía `2` en vez de `0` (caso
> 6 FALLA).  
> **Cómo lo detecté:** leí los criterios (*"El offset siempre incluye la
> multiplicación por el ancho del elemento"* y *"una base NULL no emite
> instrucciones"*), tracé el caso 4 a mano (`_t0 = i * 3` → `_t1 = _t0 + j`
> → `_t2 = _t1 * 4` → `_t3 = M[_t2]`) y probé
> `emitir_acceso(l, 8, NULL, "i", 4, &t)` → tenía que regresar `0`.  
> **Corrección:** puse `* ancho` en el formato de cada offset (tanto en 1D
> como en 2D), respeté el orden `i * filas + j`, numeré los cuatro
> temporales de 2D desde `*temporales` y agregué la guardia
> `base == NULL → return 0` antes de emitir cualquier línea.

---
**Firma digital:** Paola-23-ux

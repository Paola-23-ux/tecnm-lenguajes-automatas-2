# Bitácora de Requerimientos y Auditoría IA — Sesión 22
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Generación de TAC para Expresiones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350.  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Traductor de nodos AST a TAC  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Lo primero que me falló fue el **orden del recorrido**: mi primer intento
> era en preorden (primero el padre, después los hijos) y me generaba
> `_t0 = a + _t0`... o sea, usaba un temporal que aún no existía. Hay que
> recorrer en **postorden**: primero traduzco el hijo izquierdo, luego el
> derecho, y **al final** emito la línea del operador. Con eso la expresión
> `a + b * c` me da `_t0 = b * c` primero y `_t1 = a + _t0` después, que es
> justo la precedencia.  
> Otro detalle: **un solo contador de temporales para todo el árbol**, no uno
> por nivel; si no, se repiten los `_t0` en ramas distintas.  
> Y no olvidar los límites: si el árbol tiene más operadores que `max`, no se
> puede escribir pasando `lineas[max]` (desbordamiento), y cada línea tiene
> que caber en 63 caracteres.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Conservación de precedencia de operadores |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto: Estoy en la sesión 22, generando código de tres direcciones (TAC)
desde un árbol de expresiones. Tengo un AST con nodos que traen tipo ('H'
hoja o 'B' binario), operador, valor y punteros a izq/der. Debo escribir una
línea TAC por operador en un arreglo de `max` filas de 64 caracteres y
devolver cuántas líneas emití. Compilo en C11 con GCC con -Wall -Wextra
-Werror, sin librerías fuera del estándar.

Rol: Que me ayudes como ingeniero de compiladores que está escribiendo la
generación de código intermedio a partir del AST.

Especificación: La función con la firma EXACTA de api_sesion22.h, sin
main():
  int generar_tac(const Nodo *raiz, char lineas[][64], int max);
Reglas:
 - Recorrido en POSTORDEN: primero se traducen los hijos y DESPUÉS se emite
   la línea del operador, así se conserva la precedencia.
 - Cada hoja no emite línea: devuelve su valor ("a", "b", ...).
 - Cada operador genera exactamente una línea con el formato
     "_tK = izq op der"   (ejemplo: "_t0 = b * c", "_t2 = _t0 * _t1")
   y devuelve el nombre del temporal para que el padre lo use.
 - Los temporales salen de la función nuevo_temporal(p, contador) que ya
   viene en el archivo: UN contador compartido para todo el árbol, que se
   incrementa en cada operador (_t0, _t1, _t2...).
 - Nunca escribas más de `max` líneas ni más de 63 caracteres por línea
   (usa snprintf con 64).
 - Devuelve el total de líneas emitidas; una hoja sola devuelve 0; raíz NULL
   devuelve 0.

Output: Solo la función en C99 sin main(), más el uso del Pool y
nuevo_temporal que ya están en artesano_sesion22_c.c, lista para pegar en
ia_sesion22_c.c.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion22_c.c - Fase 2: El Científico */
#include "api_sesion22.h"
#include <stdio.h>
#include <string.h>

/* Los temporales ya estan resueltos: usalos tal cual. */
typedef struct {
    char nombre[64][16];
    int usados;
} Pool;

const char *nuevo_temporal(Pool *p, int *contador) {
    if (p->usados >= 64) return "_t?";
    snprintf(p->nombre[p->usados], sizeof p->nombre[0], "_t%d", (*contador)++);
    return p->nombre[p->usados++];
}

static const char *traducir(const Nodo *n, char lineas[][64], int max,
                            int *nlineas, Pool *p, int *contador) {
    if (n == NULL) { return "_"; }
    if (n->tipo == 'H') { return n->valor; }   /* hoja: no emite línea */

    /* postorden: primero los hijos */
    const char *izq = traducir(n->izq, lineas, max, nlineas, p, contador);
    const char *der = traducir(n->der, lineas, max, nlineas, p, contador);

    const char *t = nuevo_temporal(p, contador);
    if (*nlineas < max) {                      /* respeta el límite */
        snprintf(lineas[*nlineas], 64, "%s = %s %c %s", t, izq, n->op, der);
        (*nlineas)++;
    }
    return t;
}

int generar_tac(const Nodo *raiz, char lineas[][64], int max) {
    if (raiz == NULL || lineas == NULL || max <= 0) { return 0; }
    Pool p = {0};
    int contador = 0;
    int nlineas = 0;
    traducir(raiz, lineas, max, &nlineas, &p, &contador);
    return nlineas;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Expresiones anidadas con paréntesis

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Salida literal del harness (6/6 OK)
```text
=== AUDITORIA SESION 22 - Traductor de AST a TAC con precedencia ===
Caso                               esperado:              obtenido:              resultado
[ 1] a + b * c                          esperado: _t0 = b * c  obtenido: _t0 = b * c  OK
[ 2] a + b * c (2a linea)
     esperado: _t1 = a + _t0
     obtenido: _t1 = a + _t0
     -> OK
[ 3] (a + b) * c
     esperado: _t1 = _t0 * c
     obtenido: _t1 = _t0 * c
     -> OK
[ 4] x * (y - z) + w
     esperado: _t2 = _t1 + w
     obtenido: _t2 = _t1 + w
     -> OK
[ 5] hoja sin operaciones               esperado: 0            obtenido: 0            OK
[ 6] (a + b) * (c - d)
     esperado: _t2 = _t0 * _t1
     obtenido: _t2 = _t0 * _t1
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion22.md
```


### Corrida real desde Colab (evidencia)

**Celda 1 - `artesano_sesion22_c.c`:**

```text
=== AUDITORIA SESION 22 - Traductor de AST a TAC con precedencia ===
Caso                               esperado:              obtenido:              resultado
[ 1] a + b * c                          esperado: _t0 = b * c  obtenido: _t0 = b * c  OK
[ 2] a + b * c (2a linea)
     esperado: _t1 = a + _t0
     obtenido: _t1 = a + _t0
     -> OK
[ 3] (a + b) * c
     esperado: _t1 = _t0 * c
     obtenido: _t1 = _t0 * c
     -> OK
[ 4] x * (y - z) + w
     esperado: _t2 = _t1 + w
     obtenido: _t2 = _t1 + w
     -> OK
[ 5] hoja sin operaciones               esperado: 0            obtenido: 0            OK
[ 6] (a + b) * (c - d)
     esperado: _t2 = _t0 * _t1
     obtenido: _t2 = _t0 * _t1
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion22.md
```

**Celda 2 - `ia_sesion22_c.c`:**

```text
=== AUDITORIA SESION 22 - Traductor de AST a TAC con precedencia ===
Caso                               esperado:              obtenido:              resultado
[ 1] a + b * c                          esperado: _t0 = b * c  obtenido: _t0 = b * c  OK
[ 2] a + b * c (2a linea)
     esperado: _t1 = a + _t0
     obtenido: _t1 = a + _t0
     -> OK
[ 3] (a + b) * c
     esperado: _t1 = _t0 * c
     obtenido: _t1 = _t0 * c
     -> OK
[ 4] x * (y - z) + w
     esperado: _t2 = _t1 + w
     obtenido: _t2 = _t1 + w
     -> OK
[ 5] hoja sin operaciones               esperado: 0            obtenido: 0            OK
[ 6] (a + b) * (c - d)
     esperado: _t2 = _t0 * _t1
     obtenido: _t2 = _t0 * _t1
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion22.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La primera propuesta que me devolvió recorría el árbol en **preorden**:
> emitía la línea del padre antes que la de los hijos, así que en el caso 1
> (`a + b * c`) el `l[0]` salía mal y el temporal del padre se usaba antes de
> existir; además no chequeaba `max`, así que con un árbol más grande que el
> arreglo escribía fuera de `lineas[max]`.  
> **Cómo lo detecté:** tracé el caso 1 a mano esperando `_t0 = b * c` (el hijo
> `b * c` tiene que salir **antes** que `a + _t0`) y revisé el criterio
> *"El orden de las líneas respeta la precedencia del árbol"* junto con la
> regla *"Nunca escribas más de max líneas"*.  
> **Corrección:** reescribí la función recursiva en postorden (izq → der →
> emitir) con un único contador compartido vía el `Pool`, y agregué
> `if (*nlineas < max)` antes de cada `snprintf(..., 64, ...)` para respetar
> tanto el número de líneas como los 63 caracteres por línea.

---
**Firma digital:** Paola-23-ux

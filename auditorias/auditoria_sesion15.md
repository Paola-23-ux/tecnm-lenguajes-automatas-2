# Bitácora de Requerimientos y Auditoría IA — Sesión 15
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Bloques Anidados y Ciclo de Vida  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350.  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Operaciones Push y Pop de ámbito  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> La restricción más dura fue **distinguir a quién pertenece la memoria**. El
> ámbito raíz lo crea el harness en la pila (`Ambito raiz = {"global", NULL};`),
> pero los ámbitos del bloque los crea mi `push` con `malloc`. Si `pop_ambito()`
> hace `free()` sobre la raíz, el programa **cae con *crash*** (liberar memoria
> que no me pertenece), justo el caso 4 del harness.  
> Solución: usar `padre == NULL` como marca de raíz — todo ámbito creado por
> `push` tiene `padre` enlazado, así que solo el raíz lo tiene en `NULL`; en ese
> caso se devuelve `NULL` **sin** liberar.  
> Segunda restricción: guardar `actual->padre` en una variable **antes** de
> `free(actual)`, o se leería un puntero ya liberado (*use-after-free*).  
> Tercera: `<stdlib.h>` no venía en el esqueleto del artesano (traía `stdio.h` y
> `string.h`); sin él, `malloc`/`free` no compilan con `-Wall -Wextra -Werror`.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Liberación de memoria al cerrar bloque |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
C: Estoy implementando el gestor de ámbitos de un compilador en C11
(GCC, -Wall -Wextra -Werror). La raíz vive en la pila del caller:
  Ambito raiz = {"global", NULL};
y los bloques anidados se crean en el heap. Dispongo de:
  typedef struct Ambito { const char *nombre; struct Ambito *padre; } Ambito;

R: Ingeniero de sistemas y compiladores (gestión de memoria y ámbitos).

E: Implementa EXACTAMENTE estas firmas, sin cambiarlas y sin main():
  Ambito *push_ambito(Ambito *actual, const char *nombre);
  Ambito *pop_ambito(Ambito *actual);
- push_ambito(): reserva un Ambito con malloc, le asigna el nombre recibido,
  lo enlaza con padre = actual y devuelve el nuevo ámbito. Devuelve NULL si
  actual es NULL, nombre es NULL o malloc falla. No modifica `actual`.
- pop_ambito(): devuelve el padre del ámbito recibido y libera SOLO el ámbito
  recibido. Si actual es NULL devuelve NULL. Si actual->padre == NULL (raíz,
  memoria que no me pertenece) devuelve NULL sin llamar free(). Guarda el padre
  ANTES de free para no leer memoria liberada.
- Cada push debe emparejarse con un pop: no dejar ámbitos colgando.

O: Código C99 modular sin main(), con complejidad O(1) por operación, listo
para pegar en artesano_sesion15_c.c, sin fugas ni dobles liberaciones.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion15_c.c - Fase 2: El Científico */
#include "api_sesion15.h"
#include <stdlib.h>

Ambito *push_ambito(Ambito *actual, const char *nombre) {
    if (actual == NULL || nombre == NULL) { return NULL; }
    Ambito *nuevo = (Ambito *)malloc(sizeof(Ambito));
    if (nuevo == NULL) { return NULL; }
    nuevo->nombre = nombre;
    nuevo->padre = actual;
    return nuevo;
}

Ambito *pop_ambito(Ambito *actual) {
    if (actual == NULL) { return NULL; }
    Ambito *padre = actual->padre;   /* se guarda ANTES de free */
    if (padre == NULL) { return NULL; } /* raíz: no se libera memoria ajena */
    free(actual);
    return padre;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Aislamiento de variables fuera de bloque

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Salida literal del harness (6/6 OK)
```text
=== AUDITORIA SESION 15 - Push y pop de ambitos ===
Caso                               esperado:              obtenido:              resultado
[ 1] nombre del ambito nuevo            esperado: if           obtenido: if           OK
[ 2] profundidad con dos push           esperado: 3            obtenido: 3            OK
[ 3] pop devuelve el padre              esperado: 1            obtenido: 1            OK
[ 4] pop sobre la raiz                  esperado: 1            obtenido: 1            OK
[ 5] push con padre NULL                esperado: 1            obtenido: 1            OK
[ 6] profundidad tras pop               esperado: 2            obtenido: 2            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion15.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La primera propuesta de la IA **omitía la raíz en la pila**: hacía
> `free(actual)` de forma incondicional en `pop_ambito()`. Como el harness
> declara `Ambito raiz` en la pila, el caso 4 (`pop sobre la raíz`) provocaban
> un *crash* por `free()` sobre memoria no alojada con `malloc`.  
> **Cómo lo detecté:** leí el harness antes de ejecutarlo y vi que la raíz es
> `Ambito raiz = {"global", NULL}` (automática) y que el caso 4 hace
> `pop_ambito(&raiz) == NULL`; probé además que un push siempre nace con
> `padre != NULL`, por lo que `padre == NULL` es marca exacta de raíz.  
> **Corrección:** guardar `padre = actual->padre` antes de `free(actual)` y
> cortar con `if (padre == NULL) return NULL;` antes de liberar. Con eso el
> caso 4 devuelve `1` sin *crash*, y los casos 1, 2, 3 y 6 liberan exactamente
> los ámbitos que crearon (sin fugas ni dobles liberaciones).

---
**Firma digital:** Paola-23-ux

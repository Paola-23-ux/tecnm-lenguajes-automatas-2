# Bitácora de Requerimientos y Auditoría IA — Sesión 19
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Estructuras (struct) y Miembros  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350.  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Offsets de miembros en estructuras  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Lo que más se me atragantó fue **el padding final**. Al sumar los campos de
> `{char a; int b; short c;}` me salía 10 bytes (1 + 4 + 2 con sus
> alineaciones) y el harness esperaba 12: hay que redondear el total hacia
> arriba al múltiplo del campo más ancho (4), porque la estructura completa
> también se alinea. Igual con `{double a; char b;}`: la suma "a ojo" da 9 y
> el tamaño real es 16.  
> Otro punto donde casi la cago: `offset_campo()` debe devolver **el inicio**
> del campo, o sea el cursor **antes** de sumarle su ancho; si lo devolvía
> después, me regresaba la posición final y todo salía corrido.  
> Y el orden de la recorrida importa: alinear cada campo **antes** de
> asignarle offset, no después de sumar.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Alineación de memoria por palabras |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto: Estoy en el laboratorio 19 de estructuras. Recibo una lista ordenada
de campos de una struct con su nombre y su tipo, y necesito saber en qué
posición de memoria queda cada campo (offset) y cuánto mide la struct completa
con todo y padding, como lo calcularía el compilador. Compilo en C11 con GCC
usando -Wall -Wextra -Werror y sin bibliotecas fuera del estándar.

Rol: Que me ayudes como ingeniero de sistemas que sabe de alineación de
memoria y layout de estructuras.

Especificación: Dos funciones con la firma EXACTA de api_sesion19.h, sin
main():
  int offset_campo(const Campo *campos, int total, const char *nombre);
  int tamano_struct(const Campo *campos, int total);
Anchos de tipo: char=1, short=2, int=4, float=4, double=8, puntero=8;
cualquier otro tipo devuelve -1.

offset_campo():
 - Recorre los campos en orden con un cursor; antes de asignarle el offset a
   cada campo hay que redondear el cursor hacia arriba a su propio ancho
   (alineación natural).
 - Devuelve el offset de INICIO del campo que coincida con el nombre (strcmp
   exacto), sin sumarle todavía su ancho.
 - Devuelve -1 si el nombre no existe, si la lista es NULL o si aparece un
   tipo desconocido.

tamano_struct():
 - Hace el mismo recorrido sumando anchos con alineación y al FINAL redondea
   el cursor al múltiplo del campo más ancho (ese es el padding final).
 - Si total es 0 devuelve 0; lista NULL o tipo desconocido devuelven -1.

Output: Solo las dos funciones en C99, sin main(), complejidad O(total), para
pegar en ia_sesion19_c.c.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion19_c.c - Fase 2: El Científico */
#include "api_sesion19.h"
#include <string.h>

static int ancho_tipo(const char *tipo) {
    if (tipo == NULL) { return -1; }
    if (strcmp(tipo, "char") == 0)    { return 1; }
    if (strcmp(tipo, "short") == 0)   { return 2; }
    if (strcmp(tipo, "int") == 0)     { return 4; }
    if (strcmp(tipo, "float") == 0)   { return 4; }
    if (strcmp(tipo, "double") == 0)  { return 8; }
    if (strcmp(tipo, "puntero") == 0) { return 8; }
    return -1;
}

static int alinear(int cursor, int ancho) {
    return ((cursor + ancho - 1) / ancho) * ancho;
}

int offset_campo(const Campo *campos, int total, const char *nombre) {
    if (campos == NULL || nombre == NULL || total < 0) { return -1; }
    int cursor = 0;
    for (int i = 0; i < total; i++) {
        int ancho = ancho_tipo(campos[i].tipo);
        if (ancho < 0) { return -1; }
        cursor = alinear(cursor, ancho);          /* alinea ANTES */
        if (strcmp(campos[i].nombre, nombre) == 0) {
            return cursor;                        /* inicio del campo */
        }
        cursor += ancho;
    }
    return -1;                                    /* no existe */
}

int tamano_struct(const Campo *campos, int total) {
    if (campos == NULL || total < 0) { return -1; }
    int cursor = 0;
    int max_align = 1;
    for (int i = 0; i < total; i++) {
        int ancho = ancho_tipo(campos[i].tipo);
        if (ancho < 0) { return -1; }
        cursor = alinear(cursor, ancho);
        cursor += ancho;
        if (ancho > max_align) { max_align = ancho; }
    }
    return alinear(cursor, max_align);            /* padding final */
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Acceso a miembros inexistentes

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Salida literal del harness (6/6 OK)
```text
=== AUDITORIA SESION 19 - Offsets de miembros y tamano de struct ===
Caso                               esperado:              obtenido:              resultado
[ 1] struct char e int: b               esperado: 4            obtenido: 4            OK
[ 2] struct de tres campos: c           esperado: 8            obtenido: 8            OK
[ 3] tamano total con padding           esperado: 12           obtenido: 12           OK
[ 4] double seguido de char             esperado: 16           obtenido: 16           OK
[ 5] miembro inexistente                esperado: -1           obtenido: -1           OK
[ 6] struct sin campos                  esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion19.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La primera propuesta que me devolvió solo alineaba cada campo, pero **se
> olvidó del padding final**: `tamano_struct({char, int, short})` regresaba
> `10` en vez de `12`, y `tamano_struct({double, char})` daba `9` en vez de
> `16`. Con eso los casos 3 y 4 salían FALLA.  
> **Cómo lo detecté:** revisé el criterio *"El tamaño total incluye el padding
> final hasta el múltiplo de la alineación mayor"* y tracé a mano el caso 3:
> cursor = 10, alineación mayor = 4, `10 → 12`. También probé que
> `offset_campo` devolviera el inicio y no el final del campo.  
> **Corrección:** guardé el `max_align` durante el recorrido y al final llamé
> `alinear(cursor, max_align)`; con eso el caso 4 da `align(9, 8) = 16` y el
> caso 6 (`total = 0`) devuelve `0` porque el cursor queda en 0.

---
**Firma digital:** Paola-23-ux

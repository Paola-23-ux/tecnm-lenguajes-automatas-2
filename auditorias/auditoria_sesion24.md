# Bitácora de Requerimientos y Auditoría IA — Sesión 24
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Control de Flujo: If e If-Else  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Generador de saltos condicionales con cortocircuito  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Lo que más se me complicó fue **el cortocircuito**: mi primer intento
> emitía los dos saltos siempre, o sea evaluaba `b` aunque `a` ya hubiera
> decidido la condición, y eso es exactamente lo contrario de lo que hace
> un `&&` de verdad. Hay que emitir el salto de `a` y solo meter el de `b`
> si `b` existe.  
> Otro error tonto que tuve: confundí **IFFALSE con IFTRUE**. Para `&`
> (AND) si el operando es falso nos vamos a la salida (`IFFALSE`), pero
> para `|` (OR) si el operando es **verdadero** nos vamos a la salida
> (`IFTRUE`); los puse al revés y los casos 4 y 5 FALLA.  
> Y el caso `b = NULL`: hay que devolver **1** (un solo salto) sin inventar
> una comparación rara ni escribir `(null)` en la línea.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Cortocircuito y salto condicional |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto: Estoy en la sesión 24 de control de flujo. Estoy generando el
esqueleto de saltos de una condición compuesta (AND/OR) para un compilador
que solo sabe avanzar instrucción por instrucción: recibo dos operandos, un
operador lógico (& o |) y la etiqueta de salida, y tengo que escribir en un
arreglo de `max` filas de 64 caracteres los saltos que la evalúan. Compilo
en C11 con GCC con -Wall -Wextra -Werror y sin bibliotecas fuera del
estándar.

Rol: Que me ayudes como ingeniero de compiladores que está escribiendo la
generación de código intermedio para sentencias if / if-else.

Especificación: La función con la firma EXACTA de api_sesion24.h, sin
main():
  int emitir_condicion(char lineas[][64], int max, const char *a,
                       const char *b, char op, const char *etiqueta);
Reglas:
 - op == '&': cada operando se evalúa con IFFALSE (si el operando es falso
   se salta a la etiqueta de salida).
 - op == '|': cada operando se evalúa con IFTRUE (si el operando es
   verdadero se salta a la etiqueta de salida).
 - Formato exacto de cada línea: "IFFALSE a GOTO L9" o "IFTRUE a GOTO L9"
   (operador, espacio, operando, espacio, GOTO, espacio, etiqueta).
 - Cortocircuito: se emite el salto del operando a; SOLO si b viene distinto
   de NULL se emite el segundo salto con el mismo formato y el mismo
   predicado que el primero.
 - Si b es NULL se emite UNA sola línea y no se inventa ninguna comparación.
 - Devuelve el número de líneas emitidas (1 o 2); nunca escribas más de
   `max` líneas ni más de 63 caracteres por línea (usa snprintf con 64).
 - Si lineas, a o etiqueta vienen NULL o max <= 0, devuelve 0 sin escribir.

Output: Solo la función en C99, sin main(), complejidad O(1), lista para
pegar en ia_sesion24_c.c.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion24_c.c - Fase 2: El Científico */
#include "api_sesion24.h"
#include <stdio.h>
#include <string.h>

int emitir_condicion(char lineas[][64], int max, const char *a,
                     const char *b, char op, const char *etiqueta) {
    if (lineas == NULL || a == NULL || etiqueta == NULL || max <= 0) {
        return 0;
    }
    const char *predicado = (op == '&') ? "IFFALSE" : "IFTRUE";

    snprintf(lineas[0], 64, "%s %s GOTO %s", predicado, a, etiqueta);
    if (b == NULL) {
        return 1;                    /* condición simple: un solo salto */
    }
    if (max < 2) {
        return 1;
    }
    snprintf(lineas[1], 64, "%s %s GOTO %s", predicado, b, etiqueta);
    return 2;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Saltos adelante y atrás estructurados

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Salida literal del harness (6/6 OK)
```text
=== AUDITORIA SESION 24 - Saltos condicionales con cortocircuito ===
Caso                               esperado:              obtenido:              resultado
[ 1] a && b (primer salto)
     esperado: IFFALSE a GOTO L9
     obtenido: IFFALSE a GOTO L9
     -> OK
[ 2] a && b (segundo salto)
     esperado: IFFALSE b GOTO L9
     obtenido: IFFALSE b GOTO L9
     -> OK
[ 3] a && b (dos saltos)                esperado: 2            obtenido: 2            OK
[ 4] a || b (primer salto)
     esperado: IFTRUE a GOTO L9
     obtenido: IFTRUE a GOTO L9
     -> OK
[ 5] a || b (segundo salto)
     esperado: IFTRUE b GOTO L9
     obtenido: IFTRUE b GOTO L9
     -> OK
[ 6] condicion simple (b NULL)
     esperado: IFFALSE a GOTO L0
     obtenido: IFFALSE a GOTO L0
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion24.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La primera propuesta que me devolvió emitía **los dos saltos siempre**,
> aunque `b` viniera en `NULL`: con eso el caso 6 (`condicion simple`)
> escribía una segunda línea con `(null)` y la función regresaba `2` en vez
> de `1`. Además usaba `IFFALSE` para ambos operadores, así que con `|`
> los casos 4 y 5 salían `IFFALSE a GOTO L9` en vez de `IFTRUE`.  
> **Cómo lo detecté:** tracé el caso 6 a mano (`b = NULL` → solo una
> línea, sin comparación inventada) y comparé los casos 4 y 5 contra la
> salida literal: para OR el predicado tiene que ser `IFTRUE` (si es
> verdadero, salta). También revisé el criterio *"El segundo operando se
> evalúa con un salto propio"* y el de *"una condición sin segundo operando
> emite una sola instrucción"*.  
> **Corrección:** elegí el predicado con `const char *predicado = (op == '&')
> ? "IFFALSE" : "IFTRUE";` y usé el mismo para ambos saltos; el segundo
> `snprintf` solo se ejecuta si `b != NULL` y `max >= 2`, y la función
> devuelve el conteo real de líneas emitidas (1 o 2).

---
**Firma digital:** Paola-23-ux

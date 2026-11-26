# Bitácora de Requerimientos y Auditoría IA — Sesión 25
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Control de Flujo: Bucles While y For  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Esqueleto TAC de while y for  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Lo que más me costó fue **el orden de las líneas del for**: mi primer
> intento ponía el `post` (la actualización del iterador) *después* del
> `GOTO` de regreso, o sea que el incremento quedaba fuera del ciclo y el
> bucle sería infinito; el caso 5 (`i = i + 1` en la posición 3) me salía
> mal. El orden correcto es: init → marca → salto → **post** → GOTO → marca
> de fin.  
> Otro detalle: **cada ciclo consume dos etiquetas** (`L0`/`L1`, después
> `L2`/`L3`), así que el contador de etiquetas hay que incrementarlo
> **dos veces por ciclo**; si lo dejo fijo, el segundo while me reutiliza
> `L0` y el caso 3 FALLA.  
> Y el `NULL` del `post`: hay que **omitir** la línea (el for sin post
> emite 5, no 6), sin escribir `(null)` en el buffer.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Estructura de bucles con etiquetas únicas |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto: Estoy en la sesión 25 de control de flujo, generando el esqueleto
TAC de bucles while y for para un compilador. Las funciones agregan líneas a
un arreglo de `max` filas de 64 caracteres en la posición *n y avanzan ese
contador; las etiquetas salen de *etiquetas y hay que consumir dos por
ciclo. Compilo en C11 con GCC con -Wall -Wextra -Werror y sin bibliotecas
fuera del estándar.

Rol: Que me ayudes como ingeniero de compiladores escribiendo la generación
de código intermedio para sentencias de repetición.

Especificación: Dos funciones con la firma EXACTA de api_sesion25.h, sin
main():
  void emitir_while(char lineas[][64], int max, int *n,
                    const char *cond, int *etiquetas);
  void emitir_for(char lineas[][64], int max, int *n, const char *inicio,
                  const char *cond, const char *post, int *etiquetas);

emitir_while(): emite EXACTAMENTE cuatro líneas en este orden:
  1) "LK:"          marca de inicio (K = valor actual de *etiquetas)
  2) "IFFALSE <cond> GOTO LK+1"   salto condicional a la marca de fin
  3) "GOTO LK"      regreso a la marca de inicio
  4) "LK+1:"        marca de fin
  y suma 2 a *etiquetas.

emitir_for(): desazúcar el for en este orden:
  1) si inicio != NULL: la línea "<inicio>" (ejemplo "i = 0"); si es NULL
     se omite
  2) "LK:"          marca de inicio
  3) "IFFALSE <cond> GOTO LK+1"
  4) si post != NULL: la línea "<post>" (ejemplo "i = i + 1") ANTES del
     GOTO de regreso; si es NULL se omite la línea
  5) "GOTO LK"      regreso a la marca de inicio
  6) "LK+1:"        marca de fin
  y suma 2 a *etiquetas.

Reglas comunes:
 - Nunca reutilices nombres de etiqueta: dos ciclos seguidos usan L0/L1
   y luego L2/L3.
 - Nunca escribas más de `max` líneas (usa snprintf con 64 y checa *n <
   max antes de cada línea).
 - Un argumento NULL en inicio o post significa que esa instrucción NO se
   emite; jamás escribas "(null)".
 - Si lineas, n o etiquetas vienen NULL, no hace nada.

Output: Solo las dos funciones en C99, sin main(), para pegar en
ia_sesion25_c.c.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion25_c.c - Fase 2: El Científico */
#include "api_sesion25.h"
#include <stdio.h>
#include <string.h>

static void agregar(char lineas[][64], int max, int *n, const char *txt) {
    if (*n < 0 || *n >= max) { return; }        /* respeta el límite */
    snprintf(lineas[*n], 64, "%s", txt);
    (*n)++;
}

void emitir_while(char lineas[][64], int max, int *n,
                  const char *cond, int *etiquetas) {
    if (lineas == NULL || n == NULL || cond == NULL || etiquetas == NULL) {
        return;
    }
    int ini = (*etiquetas)++;
    int fin = (*etiquetas)++;                  /* dos etiquetas por ciclo */
    char buf[64];

    snprintf(buf, sizeof buf, "L%d:", ini);
    agregar(lineas, max, n, buf);
    snprintf(buf, sizeof buf, "IFFALSE %s GOTO L%d", cond, fin);
    agregar(lineas, max, n, buf);
    snprintf(buf, sizeof buf, "GOTO L%d", ini);
    agregar(lineas, max, n, buf);
    snprintf(buf, sizeof buf, "L%d:", fin);
    agregar(lineas, max, n, buf);
}

void emitir_for(char lineas[][64], int max, int *n, const char *inicio,
                const char *cond, const char *post, int *etiquetas) {
    if (lineas == NULL || n == NULL || cond == NULL || etiquetas == NULL) {
        return;
    }
    int ini = (*etiquetas)++;
    int fin = (*etiquetas)++;
    char buf[64];

    if (inicio != NULL) {                      /* NULL = se omite */
        snprintf(buf, sizeof buf, "%s", inicio);
        agregar(lineas, max, n, buf);
    }
    snprintf(buf, sizeof buf, "L%d:", ini);
    agregar(lineas, max, n, buf);
    snprintf(buf, sizeof buf, "IFFALSE %s GOTO L%d", cond, fin);
    agregar(lineas, max, n, buf);
    if (post != NULL) {                        /* ANTES del GOTO de regreso */
        snprintf(buf, sizeof buf, "%s", post);
        agregar(lineas, max, n, buf);
    }
    snprintf(buf, sizeof buf, "GOTO L%d", ini);
    agregar(lineas, max, n, buf);
    snprintf(buf, sizeof buf, "L%d:", fin);
    agregar(lineas, max, n, buf);
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Bucles con saltos de regreso

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Salida literal del harness (6/6 OK)
```text
=== AUDITORIA SESION 25 - Esqueleto TAC de while y for ===
Caso                               esperado:              obtenido:              resultado
[ 1] while: marca de inicio             esperado: L0:          obtenido: L0:          OK
[ 2] while: condicion
     esperado: IFFALSE a < b GOTO L1
     obtenido: IFFALSE a < b GOTO L1
     -> OK
[ 3] dos ciclos sin colision            esperado: L2:          obtenido: L2:          OK
[ 4] for: inicializacion                esperado: i = 0        obtenido: i = 0        OK
[ 5] for: actualizacion                 esperado: i = i + 1    obtenido: i = i + 1    OK
[ 6] for sin post (NULL)                esperado: 5            obtenido: 5            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion25.md
```


### Corrida real desde Colab (evidencia)

**Celda 1 - `artesano_sesion25_c.c`:**

```text
=== AUDITORIA SESION 25 - Esqueleto TAC de while y for ===
Caso                               esperado:              obtenido:              resultado
[ 1] while: marca de inicio             esperado: L0:          obtenido: L0:          OK
[ 2] while: condicion
     esperado: IFFALSE a < b GOTO L1
     obtenido: IFFALSE a < b GOTO L1
     -> OK
[ 3] dos ciclos sin colision            esperado: L2:          obtenido: L2:          OK
[ 4] for: inicializacion                esperado: i = 0        obtenido: i = 0        OK
[ 5] for: actualizacion                 esperado: i = i + 1    obtenido: i = i + 1    OK
[ 6] for sin post (NULL)                esperado: 5            obtenido: 5            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion25.md
```

**Celda 2 - `ia_sesion25_c.c`:**

```text
=== AUDITORIA SESION 25 - Esqueleto TAC de while y for ===
Caso                               esperado:              obtenido:              resultado
[ 1] while: marca de inicio             esperado: L0:          obtenido: L0:          OK
[ 2] while: condicion
     esperado: IFFALSE a < b GOTO L1
     obtenido: IFFALSE a < b GOTO L1
     -> OK
[ 3] dos ciclos sin colision            esperado: L2:          obtenido: L2:          OK
[ 4] for: inicializacion                esperado: i = 0        obtenido: i = 0        OK
[ 5] for: actualizacion                 esperado: i = i + 1    obtenido: i = i + 1    OK
[ 6] for sin post (NULL)                esperado: 5            obtenido: 5            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion25.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La primera propuesta que me devolvió **siempre emitía la línea del `post`**,
> aunque viniera en `NULL`: escribía `(null)` en el buffer y el for sin
> actualización regresaba `6` en vez de `5` (caso 6 FALLA). Y en el `while`
> incrementaba el contador de etiquetas solo **una** vez, así que el segundo
> ciclo volvía a usar `L0`/`L1` y los dos bucles colisionaban (caso 3
> FALLA).  
> **Cómo lo detecté:** leí los criterios del laboratorio (*"Dos ciclos
> seguidos usan etiquetas distintas (L0/L1 y L2/L3)"* y *"si post es NULL,
> se omite la actualización"*), conté a mano las líneas del caso 6
> (init + marca + salto + GOTO + fin = **5**) y tracé el caso 3: el segundo
> `emitir_while` tiene que empezar en `l[4]` con `L2:`. También verifiqué
> el orden del caso 5: `i = i + 1` debe caer en `l[3]`, **antes** del
> `GOTO L0`.  
> **Corrección:** puse `if (post != NULL)` / `if (inicio != NULL)` antes de
> escribir esas líneas, tomé las etiquetas con `ini = (*etiquetas)++; fin =
> (*etiquetas)++;` (dos por ciclo) y pasé cada línea por un helper
> `agregar()` que solo escribe si `*n < max`.

---
**Firma digital:** Paola-23-ux

# Bitácora de Requerimientos y Auditoría IA — Sesión 23
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Etiquetas y Saltos Incondicionales  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Generador de etiquetas e instrucciones goto  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Aquí me confundí con **el punto y el espacio**: para el GOTO el formato es
> `"GOTO L2"` (sin nada al final), pero para la marca es `"L2:"` (con los
> dos puntos y **sin** espacio después). Me tardé porque en mi primer intento
> le puse dos puntos al GOTO y el caso 3 me salía `GOTO L2:` en vez de
> `GOTO L2`.  
> Otro detalle: la etiqueta puede venir en `NULL` y ahí hay que escribir
> **cadena vacía** (`""`), no `"L"` ni `(null)`; si mando `NULL` directo a
> `snprintf` con `%s` en glibc me imprime `(null)` y el caso 5 revienta.  
> Y otra vez el contador: se **usa y después se incrementa**, igual que en la
> sesión 21; si incremento primero, la primera llamada me daría `L1` en vez
> de `L0`.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Trazabilidad del flujo de control |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto: Estoy en la sesión 23 de código de tres direcciones, en la parte
de las etiquetas y los saltos incondicionales. Tengo un contador de etiquetas
y dos funciones que escriben en un buffer de destino con su capacidad: una
genera la etiqueta nueva y otra escribe el GOTO o la marca de destino.
Compilo en C11 con GCC usando -Wall -Wextra -Werror y sin bibliotecas fuera
del estándar; nada de printf suelto ni malloc.

Rol: Que me ayudes como ingeniero de compiladores que está escribiendo la
etapa de generación de código intermedio (flujo de control).

Especificación: Dos funciones con la firma EXACTA de api_sesion23.h, sin
main():
  void nueva_etiqueta(char *dst, int cap, int *contador);
  void emitir_flujo(char *dst, int cap, const char *etiqueta, int es_salto);

nueva_etiqueta():
 - Escribe en dst "L" seguido del valor actual de *contador, con snprintf
   para respetar cap (ejemplo: contador = 0 -> "L0", 5 -> "L5", 10 -> "L10").
 - Usa el contador y DESPUÉS lo incrementa, para que las etiquetas salgan
   únicas y consecutivas (L0, L1, L2...).
 - No escribe nada si dst, cap o contador vienen inválidos (NULL o cap <= 0).

emitir_flujo():
 - Si es_salto es distinto de cero escribe "GOTO " + etiqueta, sin dos puntos
   ni espacio al final (ejemplo: "GOTO L2").
 - Si es_salto es cero escribe la marca de destino con dos puntos al final
   y sin espacio después (ejemplo: "L2:").
 - Si etiqueta viene NULL se escribe cadena vacía "" (nunca "(null)").
 - Nunca escribas fuera de cap.

Output: Solo las dos funciones en C99, sin main(), usando snprintf, listas
para pegar en ia_sesion23_c.c.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion23_c.c - Fase 2: El Científico */
#include "api_sesion23.h"
#include <stdio.h>

void nueva_etiqueta(char *dst, int cap, int *contador) {
    if (dst == NULL || cap <= 0 || contador == NULL) { return; }
    snprintf(dst, (size_t)cap, "L%d", *contador);
    (*contador)++;                  /* se usa primero, se incrementa después */
}

void emitir_flujo(char *dst, int cap, const char *etiqueta, int es_salto) {
    if (dst == NULL || cap <= 0) { return; }
    if (etiqueta == NULL) { dst[0] = '\0'; return; }   /* cadena vacía */
    if (es_salto) {
        snprintf(dst, (size_t)cap, "GOTO %s", etiqueta);
    } else {
        snprintf(dst, (size_t)cap, "%s:", etiqueta);
    }
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
=== AUDITORIA SESION 23 - Etiquetas y saltos incondicionales ===
Caso                               esperado:              obtenido:              resultado
[ 1] nueva_etiqueta(L0)                 esperado: L0           obtenido: L0           OK
[ 2] nueva_etiqueta(L5)                 esperado: L5           obtenido: L5           OK
[ 3] emitir_flujo(GOTO L2)              esperado: GOTO L2      obtenido: GOTO L2      OK
[ 4] emitir_flujo(marca L2)             esperado: L2:          obtenido: L2:          OK
[ 5] emitir_flujo(NULL)                 esperado:              obtenido:              OK
[ 6] nueva_etiqueta(L10)                esperado: L10          obtenido: L10          OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion23.md
```


### Corrida real desde Colab (evidencia)

**Celda 1 - `artesano_sesion23_c.c`:**

```text
=== AUDITORIA SESION 23 - Etiquetas y saltos incondicionales ===
Caso                               esperado:              obtenido:              resultado
[ 1] nueva_etiqueta(L0)                 esperado: L0           obtenido: L0           OK
[ 2] nueva_etiqueta(L5)                 esperado: L5           obtenido: L5           OK
[ 3] emitir_flujo(GOTO L2)              esperado: GOTO L2      obtenido: GOTO L2      OK
[ 4] emitir_flujo(marca L2)             esperado: L2:          obtenido: L2:          OK
[ 5] emitir_flujo(NULL)                 esperado:              obtenido:              OK
[ 6] nueva_etiqueta(L10)                esperado: L10          obtenido: L10          OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion23.md
```

**Celda 2 - `ia_sesion23_c.c`:**

```text
=== AUDITORIA SESION 23 - Etiquetas y saltos incondicionales ===
Caso                               esperado:              obtenido:              resultado
[ 1] nueva_etiqueta(L0)                 esperado: L0           obtenido: L0           OK
[ 2] nueva_etiqueta(L5)                 esperado: L5           obtenido: L5           OK
[ 3] emitir_flujo(GOTO L2)              esperado: GOTO L2      obtenido: GOTO L2      OK
[ 4] emitir_flujo(marca L2)             esperado: L2:          obtenido: L2:          OK
[ 5] emitir_flujo(NULL)                 esperado:              obtenido:              OK
[ 6] nueva_etiqueta(L10)                esperado: L10          obtenido: L10          OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion23.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La primera propuesta que me devolvió metía `etiqueta` directo a `snprintf`
> aunque viniera `NULL`, así que el caso 5 (`emitir_flujo(NULL)`) escribía
> `(null)` en vez de cadena vacía, y el formato de marca salía con espacio
> de más (`L2: ` en vez de `L2:`). Con eso los casos 4 y 5 FALLA.  
> **Cómo lo detecté:** comparé contra la salida literal del laboratorio,
> donde el caso 5 espera **nada** (cadena vacía) y el caso 4 espera `L2:`
> exacto; probé a mano `emitir_flujo(b, 32, NULL, 1)` → tenía que ser `""`,
> y revisé que `nueva_etiqueta` con contador en 0 diera `L0` (usar el valor
> **antes** de incrementar).  
> **Corrección:** puse `if (etiqueta == NULL) { dst[0] = '\0'; return; }`
> antes de cualquier formato, usé `snprintf` con `cap` para no pasarme del
> buffer, y dejé el contador con el patrón "formatear primero, incrementar
> después" para que las etiquetas sean L0, L1, L2... consecutivas.

---
**Firma digital:** Paola-23-ux

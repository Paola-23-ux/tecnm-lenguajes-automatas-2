# Bitácora de Requerimientos y Auditoría IA — Sesión 21
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Fundamentos de Código de 3 Direcciones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350.  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Representación en Cuádruplos (op,a,b,r)  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Me costó entender que estas funciones **no imprimen**: escriben en un buffer
> que les pasas, así que no puede haber `printf` suelto ni `malloc`. Usé
> `snprintf(dst, cap, ...)` porque respeta la capacidad del buffer y deja el
> `\0` aunque el texto no quepa (con `sprintf` me salía de la spec).  
> Lo otro: el orden del contador. Dice "se usa y **después** se incrementa",
> o sea que hay que formatear con `*contador` y luego hacer
> `(*contador)++`; si incremento primero, la primera llamada me daría `_t1`
> en vez de `_t0`.  
> Y los argumentos `NULL`: en el formato tienen que salir como guion bajo
> (`_`), no omitidos ni como texto raro. Por eso cada campo pasa por un
> `x ? x : "_"`.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Generación de temporales únicos (_t0..) |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto: Estoy en la sesión 21 de código de tres direcciones (TAC). Estoy
armando el generador de cuádruplos de un compilador en C11 con GCC
(-Wall -Wextra -Werror). Las dos funciones reciben un buffer de destino con
su capacidad, no imprimen en pantalla, no reservan memoria y no usan globales.

Rol: Que me ayudes como ingeniero de compiladores que está escribiendo la
etapa de generación de código intermedio.

Especificación: Dos funciones con la firma EXACTA de api_sesion21.h, sin
main():
  void nuevo_temporal(char *dst, int cap, int *contador);
  void emitir_cuadruplo(char *dst, int cap, const char *op,
                        const char *a1, const char *a2, const char *res);

nuevo_temporal():
 - Escribe en dst el texto "_t" seguido del valor actual de *contador
   (snprintf con cap, para respetar el tamaño).
 - Usa el contador y DESPUÉS lo incrementa: la primera llamada con
   contador = 0 produce "_t0", con 7 produce "_t7", y así con _t10, _t20...
 - No debe escribir nada si dst, cap o contador son inválidos.

emitir_cuadruplo():
 - Escribe en dst el texto con el formato exacto: (op, a1, a2, res)
   con coma y espacio entre cada campo, por ejemplo "(+, a, b, _t0)".
 - Si alguno de op, a1, a2 o res viene NULL se escribe como guion bajo "_",
   sin omitir el campo ni cambiar el formato
     (ejemplo: emitir(..., "=", "a", NULL, "x") -> "(=, a, _, x)"
      emitir(..., "goto", "L1", NULL, NULL) -> "(goto, L1, _, _)")

Output: Solo las dos funciones en C99, sin main(), usando snprintf para no
desbordar el buffer, listas para pegar en ia_sesion21_c.c.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion21_c.c - Fase 2: El Científico */
#include "api_sesion21.h"
#include <stdio.h>

void nuevo_temporal(char *dst, int cap, int *contador) {
    if (dst == NULL || cap <= 0 || contador == NULL) { return; }
    snprintf(dst, (size_t)cap, "_t%d", *contador);
    (*contador)++;                  /* se usa primero, se incrementa después */
}

void emitir_cuadruplo(char *dst, int cap, const char *op,
                      const char *a1, const char *a2, const char *res) {
    if (dst == NULL || cap <= 0) { return; }
    snprintf(dst, (size_t)cap, "(%s, %s, %s, %s)",
             op  ? op  : "_",
             a1  ? a1  : "_",
             a2  ? a2  : "_",
             res ? res : "_");
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Expresiones aritméticas compuestas

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Salida literal del harness (6/6 OK)
```text
=== AUDITORIA SESION 21 - Cuadruplos y temporales unicos ===
Caso                               esperado:              obtenido:              resultado
[ 1] nuevo_temporal(_t0)                esperado: _t0          obtenido: _t0          OK
[ 2] nuevo_temporal(_t7)                esperado: _t7          obtenido: _t7          OK
[ 3] cuadruplo de a + b
     esperado: (+, a, b, _t0)
     obtenido: (+, a, b, _t0)
     -> OK
[ 4] cuadruplo de _t0 * c
     esperado: (*, _t0, c, _t1)
     obtenido: (*, _t0, c, _t1)
     -> OK
[ 5] asignacion simple                  esperado: (=, a, _, x) obtenido: (=, a, _, x) OK
[ 6] goto sin operandos
     esperado: (goto, L1, _, _)
     obtenido: (goto, L1, _, _)
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion21.md
```


### Corrida real desde Colab (evidencia)

**Celda 1 - `artesano_sesion21_c.c`:**

```text
=== AUDITORIA SESION 21 - Cuadruplos y temporales unicos ===
Caso                               esperado:              obtenido:              resultado
[ 1] nuevo_temporal(_t0)                esperado: _t0          obtenido: _t0          OK
[ 2] nuevo_temporal(_t7)                esperado: _t7          obtenido: _t7          OK
[ 3] cuadruplo de a + b
     esperado: (+, a, b, _t0)
     obtenido: (+, a, b, _t0)
     -> OK
[ 4] cuadruplo de _t0 * c
     esperado: (*, _t0, c, _t1)
     obtenido: (*, _t0, c, _t1)
     -> OK
[ 5] asignacion simple                  esperado: (=, a, _, x) obtenido: (=, a, _, x) OK
[ 6] goto sin operandos
     esperado: (goto, L1, _, _)
     obtenido: (goto, L1, _, _)
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion21.md
```

**Celda 2 - `ia_sesion21_c.c`:**

```text
=== AUDITORIA SESION 21 - Cuadruplos y temporales unicos ===
Caso                               esperado:              obtenido:              resultado
[ 1] nuevo_temporal(_t0)                esperado: _t0          obtenido: _t0          OK
[ 2] nuevo_temporal(_t7)                esperado: _t7          obtenido: _t7          OK
[ 3] cuadruplo de a + b
     esperado: (+, a, b, _t0)
     obtenido: (+, a, b, _t0)
     -> OK
[ 4] cuadruplo de _t0 * c
     esperado: (*, _t0, c, _t1)
     obtenido: (*, _t0, c, _t1)
     -> OK
[ 5] asignacion simple                  esperado: (=, a, _, x) obtenido: (=, a, _, x) OK
[ 6] goto sin operandos
     esperado: (goto, L1, _, _)
     obtenido: (goto, L1, _, _)
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion21.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La primera propuesta que me devolvió hacía `(*contador)++` **antes** de
> formatear, así que la primera llamada con contador en 0 escribía `_t1` en
> vez de `_t0` (caso 1 FALLA); y metía los punteros `NULL` directo a
> `snprintf`, que en glibc los imprime como `(null)`, dejando
> `(=, a, (null), x)` en vez de `(=, a, _, x)` (caso 5 FALLA).  
> **Cómo lo detecté:** leí el contrato (*"El contador se usa y después se
> incrementa"* y *"Un argumento NULL se escribe como guion bajo"*) y comparé
> caso por caso contra la salida literal que pide el laboratorio,
> especialmente el 5 (`asignacion simple`) y el 6 (`goto sin operandos`).  
> **Corrección:** formateé primero con el valor actual y luego incrementé, y
> cada campo del cuádruplo pasa por el ternario `x ? x : "_"`. También usé
> `snprintf(dst, cap, ...)` en vez de `sprintf` para respetar la capacidad
> del buffer.

---
**Firma digital:** Paola-23-ux

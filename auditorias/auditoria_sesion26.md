# Bitácora de Requerimientos y Auditoría IA — Sesión 26
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** TAC para Llamadas a Funciones y Retornos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Secuencia PARAM ... CALL ... RETURN  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Lo que más se me complicó fue **el número que va junto al CALL**: mi
> primer intento siempre escribía `2` porque el ejemplo traía dos
> argumentos, pero el número tiene que ser la **cantidad real de PARAM que
> emití**, contando solo los que no vienen en `NULL`. Con eso el caso 5
> (sin argumentos) me daba `_t0 = CALL p, 2` en vez de `_t0 = CALL p, 0`.  
> Otro detalle: los PARAM van **en el orden** `a1`, `a2` (el mismo en que se
> declararon los argumentos), no al revés.  
> Y en `emitir_return` casi la cago metiendo `NULL` al `snprintf`: si el
> valor de retorno es `NULL` hay que escribir **solo** `RETURN`, nunca
> `RETURN (null)`.  
> En la llamada anidada `f(g(x))` hay que dejar que la llamada interna use
> su propio temporal (`_t0`) y la externa escriba el suyo (`_t1`), sin
> pisarlos.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Convención PARAM/CALL/RETURN |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto: Estoy en la sesión 26 de TAC, en las llamadas a funciones y los
retornos. Necesito emitir en un arreglo de `max` filas de 64 caracteres la
secuencia PARAM ... CALL de una invocación (actualizando el contador *n) y
formatear la instrucción RETURN en un buffer con su capacidad. Compilo en
C11 con GCC con -Wall -Wextra -Werror y sin bibliotecas fuera del estándar.

Rol: Que me ayudes como ingeniero de compiladores escribiendo la generación
de código intermedio para llamadas a funciones.

Especificación: Dos funciones con la firma EXACTA de api_sesion26.h, sin
main():
  int emitir_llamada(char lineas[][64], int max, int *n,
                     const char *funcion, const char *a1, const char *a2,
                     const char *resultado);
  void emitir_return(char *dst, int cap, const char *valor);

emitir_llamada():
 - Cada argumento NO NULL genera una línea "PARAM <arg>", en el orden a1,
   a2 (el mismo en que se declararon); un argumento NULL no genera PARAM.
 - Después se emite el CALL con el número REAL de PARAM emitidos:
     - si resultado no es NULL:  "<resultado> = CALL <funcion>, <n>"
       (ejemplo: "_t0 = CALL f, 2")
     - si resultado es NULL:     "CALL <funcion>, <n>"  (sin asignación)
 - Actualiza *n con cada línea agregada y nunca escribe más de `max`
   líneas (snprintf con 64).
 - Devuelve el número de PARAM emitidos.
 - Soporta llamadas anidadas: cada llamada usa el temporal que se le pasa,
   sin estados globales ni temporales compartidos.

emitir_return():
 - Si valor no es NULL escribe "RETURN <valor>" (ejemplo "RETURN _t0").
 - Si valor es NULL escribe solo "RETURN".
 - Respeta cap con snprintf; nada de printf.

Output: Solo las dos funciones en C99, sin main(), complejidad O(1), para
pegar en ia_sesion26_c.c.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion26_c.c - Fase 2: El Científico */
#include "api_sesion26.h"
#include <stdio.h>
#include <string.h>

static int agregar(char lineas[][64], int max, int *n, const char *txt) {
    if (*n < 0 || *n >= max) { return 0; }      /* respeta el límite */
    snprintf(lineas[*n], 64, "%s", txt);
    (*n)++;
    return 1;
}

int emitir_llamada(char lineas[][64], int max, int *n, const char *funcion,
                   const char *a1, const char *a2, const char *resultado) {
    if (lineas == NULL || n == NULL || funcion == NULL || max <= 0) {
        return 0;
    }
    int params = 0;
    char buf[64];

    if (a1 != NULL) {                          /* orden: a1, luego a2 */
        snprintf(buf, sizeof buf, "PARAM %s", a1);
        if (agregar(lineas, max, n, buf)) { params++; }
    }
    if (a2 != NULL) {
        snprintf(buf, sizeof buf, "PARAM %s", a2);
        if (agregar(lineas, max, n, buf)) { params++; }
    }

    if (resultado != NULL) {
        snprintf(buf, sizeof buf, "%s = CALL %s, %d",
                 resultado, funcion, params);
    } else {                                   /* sin asignación */
        snprintf(buf, sizeof buf, "CALL %s, %d", funcion, params);
    }
    agregar(lineas, max, n, buf);
    return params;
}

void emitir_return(char *dst, int cap, const char *valor) {
    if (dst == NULL || cap <= 0) { return; }
    if (valor == NULL) {
        snprintf(dst, (size_t)cap, "RETURN");  /* sin valor */
    } else {
        snprintf(dst, (size_t)cap, "RETURN %s", valor);
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Llamadas anidadas y funciones sin valor

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Salida literal del harness (6/6 OK)
```text
=== AUDITORIA SESION 26 - Llamadas, parametros y retorno ===
Caso                               esperado:              obtenido:              resultado
[ 1] PARAM x                            esperado: PARAM x      obtenido: PARAM x      OK
[ 2] PARAM y                            esperado: PARAM y      obtenido: PARAM y      OK
[ 3] CALL con dos argumentos
     esperado: _t0 = CALL f, 2
     obtenido: _t0 = CALL f, 2
     -> OK
[ 4] llamada anidada f(g(x))
     esperado: _t1 = CALL f, 1
     obtenido: _t1 = CALL f, 1
     -> OK
[ 5] CALL sin argumentos
     esperado: _t0 = CALL p, 0
     obtenido: _t0 = CALL p, 0
     -> OK
[ 6] RETURN con valor                   esperado: RETURN _t0   obtenido: RETURN _t0   OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion26.md
```


### Corrida real desde Colab (evidencia)

**Celda 1 - `artesano_sesion26_c.c`:**

```text
=== AUDITORIA SESION 26 - Llamadas, parametros y retorno ===
Caso                               esperado:              obtenido:              resultado
[ 1] PARAM x                            esperado: PARAM x      obtenido: PARAM x      OK
[ 2] PARAM y                            esperado: PARAM y      obtenido: PARAM y      OK
[ 3] CALL con dos argumentos
     esperado: _t0 = CALL f, 2
     obtenido: _t0 = CALL f, 2
     -> OK
[ 4] llamada anidada f(g(x))
     esperado: _t1 = CALL f, 1
     obtenido: _t1 = CALL f, 1
     -> OK
[ 5] CALL sin argumentos
     esperado: _t0 = CALL p, 0
     obtenido: _t0 = CALL p, 0
     -> OK
[ 6] RETURN con valor                   esperado: RETURN _t0   obtenido: RETURN _t0   OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion26.md
```

**Celda 2 - `ia_sesion26_c.c`:**

```text
=== AUDITORIA SESION 26 - Llamadas, parametros y retorno ===
Caso                               esperado:              obtenido:              resultado
[ 1] PARAM x                            esperado: PARAM x      obtenido: PARAM x      OK
[ 2] PARAM y                            esperado: PARAM y      obtenido: PARAM y      OK
[ 3] CALL con dos argumentos
     esperado: _t0 = CALL f, 2
     obtenido: _t0 = CALL f, 2
     -> OK
[ 4] llamada anidada f(g(x))
     esperado: _t1 = CALL f, 1
     obtenido: _t1 = CALL f, 1
     -> OK
[ 5] CALL sin argumentos
     esperado: _t0 = CALL p, 0
     obtenido: _t0 = CALL p, 0
     -> OK
[ 6] RETURN con valor                   esperado: RETURN _t0   obtenido: RETURN _t0   OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion26.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La primera propuesta que me devolvió **contaba los argumentos recibidos**
> en vez de los PARAM realmente emitidos: pasaba `a2 = NULL` y aún así
> escribía `_t0 = CALL g, 2` en vez de `_t0 = CALL g, 1` (la llamada
> interna del caso 4 FALLA), y en el caso 5 (dos argumentos `NULL`)
> salía `_t0 = CALL p, 2` en vez de `_t0 = CALL p, 0`. Además metía el
> `valor` de `emitir_return` directo al `snprintf`, así que con `NULL`
> salía `RETURN (null)`.  
> **Cómo lo detecté:** leí los criterios (*"El entero del CALL coincide
> con el número de PARAM realmente emitidos"*) y tracé el caso 4 a mano:
> `g` solo recibe `x` → `PARAM x` + `_t0 = CALL g, 1`, y luego `f` recibe
> `_t0` → `PARAM _t0` + `_t1 = CALL f, 1` en `l[3]`. También probé
> `emitir_return(b, 32, NULL)` → tenía que dar `RETURN` exacto.  
> **Corrección:** incremento `params` **solo** dentro del `if (a != NULL)`
> de cada argumento, y uso ese conteo en el formato del CALL; en
> `emitir_return` agregué la rama `valor == NULL → "RETURN"` antes de
> formatear.

---
**Firma digital:** Paola-23-ux

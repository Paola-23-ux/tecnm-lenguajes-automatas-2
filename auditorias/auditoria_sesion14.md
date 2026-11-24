# Bitácora de Requerimientos y Auditoría IA — Sesión 14
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Declaración vs Uso de Variables  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Nombre Completo]  
**Matrícula:** [No. de Control]  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Detección de variables no declaradas  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> El problema real no fue insertar, sino **comparar bien**. El primer intento usó
> `strncmp(nombre_declarado, nombre, 1)` y el caso "prefijo" (`usar(&t, "xy")`
> con "x" declarado) regresaba `1` en lugar de `0`: el compilador aceptaría
> `xy` como si fuera `x`. Se corrigió con `strcmp()` (comparación exacta),
> donde `"x"` y `"xy"` son cadenas distintas.  
> Segunda restricción: la redeclaración **no debe** incrementar `total`; si se
> incrementaba, el caso 4 (`total tras redeclarar`) fallaba porque la tabla
> registraba 2 símbolos para un solo nombre.  
> Tercera restricción: el nombre vacío `""` debe rechazarse con `-1` **antes** de
> tocar la tabla, o quedaría un símbolo fantasma insertado.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Invariante de definición previa |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
C: Implemento un compilador en C11 (GCC, -Wall -Wextra -Werror) y necesito una
tabla de símbolos mínima para validar la regla de declaración previa.
Dispongo de: typedef struct Tabla { const char *nombres[TABLA_MAX]; int total; }
Tabla, con TABLA_MAX = 16, inicializada con memset(0).

R: Ingeniero de sistemas y compiladores.

E: Implementa EXACTAMENTE estas firmas sin cambiarlas y sin main():
  int declarar(Tabla *tabla, const char *nombre);
  int usar(const Tabla *tabla, const char *nombre);
- declarar(): devuelve 1 si insertó; 0 si el nombre YA existe en la tabla
  (redeclaración, sin incrementar total); -1 si tabla/nombre es NULL o nombre
  vacío, o si la tabla está llena. La comparación debe ser EXACTA: "x" y "xy"
  son nombres distintos (prohibido strncmp/strstr/prefijos).
- usar(): devuelve 1 solo si el nombre ya fue declarado; 0 en cualquier otro
  caso (incluido NULL), sin modificar la tabla.
- Ninguna función debe escribir memoria fuera de nombres[0..TABLA_MAX-1].

O: Código C99 modular sin main(), con complejidad O(n) por operación
(n = total de símbolos), listo para pegar en artesano_sesion14_c.c.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion14_c.c - Fase 2: El Científico */
#include "api_sesion14.h"
#include <string.h>

int declarar(Tabla *tabla, const char *nombre) {
    if (tabla == NULL || nombre == NULL || nombre[0] == '\0') { return -1; }
    if (tabla->total < 0 || tabla->total >= TABLA_MAX) { return -1; }
    for (int i = 0; i < tabla->total; i++) {
        if (tabla->nombres[i] != NULL && strcmp(tabla->nombres[i], nombre) == 0) {
            return 0;
        }
    }
    tabla->nombres[tabla->total] = nombre;
    tabla->total++;
    return 1;
}

int usar(const Tabla *tabla, const char *nombre) {
    if (tabla == NULL || nombre == NULL || nombre[0] == '\0') { return 0; }
    for (int i = 0; i < tabla->total; i++) {
        if (tabla->nombres[i] != NULL && strcmp(tabla->nombres[i], nombre) == 0) {
            return 1;
        }
    }
    return 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Uso antes de declarar y redeclaración

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Salida literal del harness (6/6 OK)
```text
=== AUDITORIA SESION 14 - Declaracion previa y redeclaracion ===
Caso                               esperado:              obtenido:              resultado
[ 1] usar sin declarar                  esperado: 0            obtenido: 0            OK
[ 2] usar despues de declarar           esperado: 1            obtenido: 1            OK
[ 3] redeclarar el mismo nombre         esperado: 0            obtenido: 0            OK
[ 4] total tras redeclarar              esperado: 1            obtenido: 1            OK
[ 5] prefijo no es el mismo nombre      esperado: 0            obtenido: 0            OK
[ 6] declarar nombre vacio              esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion14.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La primera propuesta de la IA **omitía el caso de tabla llena**: solo
> validaba `tabla == NULL` y el nombre vacío, así que una decimoséptima
> declaración escribía fuera de `nombres[16]` (desbordamiento de arreglo /
> *buffer overflow*).  
> **Cómo lo detecté:** probé la invariante de capacidad fuera del harness:
> declaré 17 nombres distintos en la misma tabla; las 16 primeras llamadas
> devolvieron `1` y la decimoséptima debía devolver `-1` sin incrementar
> `total`. También probé `usar(&t, NULL)`, que debe responder `0` en lugar de
> hacer *crash*.  
> **Corrección:** agregar `if (tabla->total < 0 || tabla->total >= TABLA_MAX)
> return -1;` antes de insertar y el guardián `nombre == NULL` en `usar()`.

---
**Firma digital:** [Usuario de GitHub]

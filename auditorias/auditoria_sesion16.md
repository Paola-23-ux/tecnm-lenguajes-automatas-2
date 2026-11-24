# Bitácora de Requerimientos y Auditoría IA — Sesión 16
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Firmas de Función y Parámetros  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350.  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Registro de prototipos y aridad  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> El detalle que casi me hace fallar fue el **orden de los diagnósticos**: hay
> tres salidas posibles (`1`, `0`, `-1`) y dos causas distintas de `-1`
> (parámetros inválidos **y** nombre no registrado). Si se devuelve `0` cuando
> el nombre no existe, el compilador reportaría "aridad incorrecta" en vez de
> "función no declarada" — justo lo que mide el caso 4.  
> Segunda restricción: la búsqueda debe ser **exacta** con `strcmp` y recorrer
> los `total` elementos del catálogo, sin asumir que el nombre está en el
> índice 0 ni usar comparación de prefijo.  
> Tercera: la función con **cero parámetros** (`main`) debe aceptar solo
> `nargs == 0`; no puede tratar el `0` como "sin información".  
> Cuarta: validar `catalogo == NULL` **antes** del bucle, o se haría
> desreferencia de puntero nulo.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Coincidencia estricta de parámetros |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
C: Estoy construyendo la fase de verificación de llamadas de un compilador en
C11 (GCC, -Wall -Wextra -Werror). Tengo un catálogo de prototipos ya
registrado y debo validar una llamada contra él. Dispongo de:
  typedef struct Funcion { const char *nombre; int aridad; } Funcion;

R: Ingeniero de sistemas y compiladores (análisis de firmas y aridad).

E: Implementa EXACTAMENTE esta firma, sin cambiarla y sin main():
  int verificar_llamada(const Funcion *catalogo, int total,
                        const char *nombre, int nargs);
- Devuelve 1 si existe una función con ese nombre EXACTO y su aridad declarada
  es igual a nargs (búsqueda lineal con strcmp sobre los `total` elementos).
- Devuelve 0 si el nombre existe pero la aridad no coincide.
- Devuelve -1 si: catalogo es NULL, nombre es NULL, nargs es negativo, o el
  nombre no está en el catálogo (función no registrada).
- Una función declarada con aridad 0 solo es válida con nargs == 0.
- No modificar el catálogo ni salirse de [0, total).

O: Código C99 modular sin main(), complejidad O(total), listo para pegar en
artesano_sesion16_c.c, con los tres valores de retorno bien distinguidos.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion16_c.c - Fase 2: El Científico */
#include "api_sesion16.h"
#include <string.h>

int verificar_llamada(const Funcion *catalogo, int total,
                      const char *nombre, int nargs) {
    if (catalogo == NULL || nombre == NULL || nargs < 0) {
        return -1;
    }
    for (int i = 0; i < total; i++) {
        if (catalogo[i].nombre != NULL &&
            strcmp(catalogo[i].nombre, nombre) == 0) {
            return (catalogo[i].aridad == nargs) ? 1 : 0;
        }
    }
    return -1; /* función no registrada */
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Llamadas con número incorrecto de args

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Salida literal del harness (6/6 OK)
```text
=== AUDITORIA SESION 16 - Registro de prototipos y aridad ===
Caso                               esperado:              obtenido:              resultado
[ 1] suma con 3 argumentos              esperado: 1            obtenido: 1            OK
[ 2] suma con 2 argumentos              esperado: 0            obtenido: 0            OK
[ 3] suma con 4 argumentos              esperado: 0            obtenido: 0            OK
[ 4] funcion no registrada              esperado: -1           obtenido: -1           OK
[ 5] main sin argumentos                esperado: 1            obtenido: 1            OK
[ 6] nombre NULL                        esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion16.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La primera propuesta de la IA **omitía el catálogo nulo y la aridad
> negativa**: abría el `for` directamente, así que `verificar_llamada(NULL,
> 1, "suma", 3)` caía con desreferencia de puntero nulo, y con `nargs = -1`
> devolvía `0` ("aridad incorrecta") en lugar de `-1`, contradiciendo el
> contrato.  
> **Cómo lo detecté:** confronté el código con la línea del contrato
> *"Un catálogo NULL, un nombre NULL o una aridad negativa devuelven -1"* y
> probé fuera del harness:
> `verificar_llamada(c, 1, "suma", -1)` → debía ser `-1`, no `0`; y
> `verificar_llamada(NULL, 1, "suma", 3)` → debía ser `-1`, no *crash*.  
> **Corrección:** agregar el guardián
> `if (catalogo == NULL || nombre == NULL || nargs < 0) return -1;` antes del
> bucle y distinguir dentro del bucle `aridad == nargs ? 1 : 0`, dejando el
> `-1` final para el nombre no registrado (caso 4).

---
**Firma digital:** Paola-23-ux

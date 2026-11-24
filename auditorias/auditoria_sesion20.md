# Bitácora de Requerimientos y Auditoría IA — Sesión 20
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Hito 2: Analizador Semántico y Scopes  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350.  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Integración de Tabla de Símbolos y AST  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Este es el Hito 2 y se nota: hay que juntar lo de ámbito (sesión 13) con lo
> de la tabla de símbolos. Lo que más me complicó fue **contar sin repetir**:
> mi primera idea era sumar `total` de cada ámbito, pero como `y` está en el
> ámbito interno Y en el padre, me salía 4 en vez de 3. Tuve que recorrer la
> cadena y, por cada nombre, checar si ya apareció en un nivel más interno
> antes de sumarlo.  
> Otro detalle: en `profundidad_de()` hay **dos** errores distintos (`-1` no
> existe, `-2` argumento NULL) y los primeros intentos me regresaban `-1` para
> todo, lo cual rompía el caso 6. El guardián de NULL tiene que ir **antes**
> de cualquier recorrido.  
> Y en la profundidad el conteo empieza en **0** para el ámbito actual (no en
> 1), subiendo uno por cada `padre`.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Contrato de Validación Semántica Integral |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto: Estoy en el Hito 2 del laboratorio (sesión 20). Tengo una cadena de
ámbitos enlazados por padre, donde cada ámbito guarda sus símbolos en un
arreglo de hasta AMBITO_MAX (8) nombres con su total. Necesito dos cosas:
saber a cuántos niveles está un nombre desde el ámbito actual y contar cuántos
nombres distintos se ven desde un punto del programa. Compilo en C11 con GCC
usando -Wall -Wextra -Werror, sin bibliotecas fuera del estándar.

Rol: Que me ayudes como ingeniero de sistemas que está armando el analizador
semántico de un compilador (resolución de nombres y shadowing).

Especificación: Dos funciones con la firma EXACTA de api_sesion20.h, sin
main():
  int profundidad_de(const Ambito *actual, const char *nombre);
  int contar_visibles(const Ambito *actual);

profundidad_de():
 - Devuelve -2 si actual o nombre vienen en NULL (guardián ANTES de
   recorrer, sin desreferenciar nada).
 - Recorre la cadena desde actual hacia el padre contando niveles desde 0
   (0 = ámbito actual, 1 = padre, 2 = abuelo...).
 - Devuelve el nivel del PRIMER ámbito que declara el nombre (el más interno
   gana en shadowing) o -1 si ningún ámbito de la cadena lo tiene.

contar_visibles():
 - Cuenta cada nombre UNA sola vez aunque aparezca en varios ámbitos (si "y"
   está en el interno y en el padre, cuenta una sola vez).
 - Devuelve -1 si actual es NULL.

Output: Solo las dos funciones en C99, sin main(), sin memoria dinámica, con
complejidad O(n²) aceptable por el tamaño de la cadena, para pegar en
ia_sesion20_c.c.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion20_c.c - Fase 2: El Científico */
#include "api_sesion20.h"
#include <string.h>

int profundidad_de(const Ambito *actual, const char *nombre) {
    if (actual == NULL || nombre == NULL) { return -2; }
    int nivel = 0;
    for (const Ambito *a = actual; a != NULL; a = a->padre, nivel++) {
        for (int i = 0; i < a->total; i++) {
            if (a->simbolos[i] != NULL &&
                strcmp(a->simbolos[i], nombre) == 0) {
                return nivel;          /* el más interno gana */
            }
        }
    }
    return -1;                         /* nadie lo declaró */
}

int contar_visibles(const Ambito *actual) {
    if (actual == NULL) { return -1; }
    int cuenta = 0;
    for (const Ambito *a = actual; a != NULL; a = a->padre) {
        for (int i = 0; i < a->total; i++) {
            const char *s = a->simbolos[i];
            if (s == NULL) { continue; }
            /* ¿ya salió antes (ámbito más interno o mismo ámbito, índice
               anterior)? entonces no se vuelve a contar */
            int ya_visto = 0;
            for (const Ambito *b = actual; b != NULL && b != a && !ya_visto;
                 b = b->padre) {
                for (int j = 0; j < b->total && !ya_visto; j++) {
                    if (b->simbolos[j] != NULL &&
                        strcmp(b->simbolos[j], s) == 0) {
                        ya_visto = 1;
                    }
                }
            }
            for (int j = 0; j < i && !ya_visto; j++) {
                if (a->simbolos[j] != NULL &&
                    strcmp(a->simbolos[j], s) == 0) {
                    ya_visto = 1;
                }
            }
            if (!ya_visto) { cuenta++; }
        }
    }
    return cuenta;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Casos complejos de shadowing y ámbito

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Salida literal del harness (6/6 OK)
```text
=== AUDITORIA SESION 20 - Resolucion de nombres y visibilidad ===
Caso                               esperado:              obtenido:              resultado
[ 1] nombre en el ambito actual         esperado: 0            obtenido: 0            OK
[ 2] nombre dos niveles arriba          esperado: 2            obtenido: 2            OK
[ 3] shadowing gana el interno          esperado: 0            obtenido: 0            OK
[ 4] nombre no visible                  esperado: -1           obtenido: -1           OK
[ 5] visibles sin repetir               esperado: 3            obtenido: 3            OK
[ 6] ambito NULL                        esperado: -2           obtenido: -2           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion20.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La primera propuesta que me devolvió **sumaba los `total` de cada ámbito**
> para `contar_visibles()`: con `global = {x, y}` y `bloque = {y, z}` daba
> `2 + 2 = 4` porque contaba `y` dos veces (caso 5 FALLA), y además
> `profundidad_de(NULL, "x")` regresaba `-1` en vez de `-2`, mezclando "no
> existe" con "argumento inválido" (caso 6 FALLA).  
> **Cómo lo detecté:** leí los criterios del laboratorio (*"contar cada nombre
> una sola vez"* y *"los argumentos NULL se reportan con -2"*) y tracé el caso
> 5 a mano: `y` está en los dos niveles, así que solo puede contar una vez;
> probé también `profundidad_de(NULL, "x")` → `-2`.  
> **Corrección:** en `contar_visibles()` antes de sumar un nombre verificó si
> ya apareció en un ámbito más interno (o en el mismo ámbito antes), y en
> `profundidad_de()` puse `if (actual == NULL || nombre == NULL) return -2;`
> como primera línea, antes de tocar cualquier puntero.

---
**Firma digital:** Paola-23-ux

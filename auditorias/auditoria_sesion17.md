# Bitácora de Requerimientos y Auditoría IA — Sesión 17
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Comprobación Semántica de Retornos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350.  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Validación de tipo de sentencia return  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Lo que más me costó fue **el orden en que reviso las cosas**. Al principio
> yo revisaba primero si todas las rutas tenían return y después el tipo, pero
> con eso el caso 2 (`void devuelve valor`) me salía como "todo bien" porque las
> rutas sí cuadraban (1 de 1) y nunca llegaba a comparar tipos. Tuve que dejar
> claro que **el tipo se evalúa antes que las rutas**.  
> Otra parte fea fue la tabla de anchos: para que un return sea válido el tipo
> destino tiene que ser igual o más ancho que el origen (`double` acepta `int`,
> pero `int` nunca acepta `double`). Y `void` es raro: una función `void` que
> devuelve algo siempre es error de tipo.  
> Por último, los conteos: `rutas_totales = 0` o más retornos que rutas no
> tienen sentido y hay que devolver `-1` antes de hacer cualquier otra cosa.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Rutas de ejecución con retorno garantizado |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto: Estoy resolviendo el laboratorio 17 sobre validación de retornos.
Trabajo en C con GCC usando -Wall -Wextra -Werror y sin bibliotecas fuera del
estándar. Recibo el tipo que promete la función, el tipo de la expresión que se
devuelve y el conteo de rutas de ejecución contra las que sí tienen return.

Rol: Quiero que me ayudes como ingeniero de compiladores (análisis semántico),
porque la función es para detectar errores de return como los que haría el
compilador.

Especificación: Necesito exactamente esta función, sin cambiar la firma y sin
main():
  int diagnosticar_retorno(const char *tipo_funcion, const char *tipo_expresion,
                           int rutas_totales, int rutas_con_return);
Que regrese:
 - 0 si todo está bien (el tipo coincide y todas las rutas retornan).
 - 1 si el tipo no es compatible: solo se acepta cuando el destino es igual de
   ancho o más ancho que el origen (double acepta int, pero int no acepta
   double), y una función void que devuelve un valor siempre es error de tipo.
 - 2 si el tipo está bien pero sobra alguna ruta sin return
   (rutas_con_return < rutas_totales), que es el caso del if sin else.
 - -1 si tipo_funcion o tipo_expresion vienen en NULL, o si los conteos no
   tienen sentido: cero rutas, más retornos que rutas o retornos negativos.
Dos reglas que no me quiero saltar: comparar EL TIPO ANTES QUE LAS RUTAS (un
return incompatible no puede reportarse como ruta faltante) y una función void
solo es válida si la expresión devuelta también es void.

Output: Solo la función en C99, sin main(), que pueda pegar tal cual en
ia_sesion17_c.c, con complejidad O(1) por llamada.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion17_c.c - Fase 2: El Científico */
#include "api_sesion17.h"
#include <string.h>

static int ancho_tipo(const char *tipo) {
    if (strcmp(tipo, "char") == 0)  { return 1; }
    if (strcmp(tipo, "short") == 0) { return 2; }
    if (strcmp(tipo, "int") == 0)   { return 4; }
    if (strcmp(tipo, "float") == 0) { return 4; }
    if (strcmp(tipo, "long") == 0)  { return 8; }
    if (strcmp(tipo, "double") == 0){ return 8; }
    return -1;
}

int diagnosticar_retorno(const char *tipo_funcion, const char *tipo_expresion,
                         int rutas_totales, int rutas_con_return) {
    if (tipo_funcion == NULL || tipo_expresion == NULL) { return -1; }
    if (rutas_totales <= 0 || rutas_con_return < 0 ||
        rutas_con_return > rutas_totales) { return -1; }

    int es_void_func = (strcmp(tipo_funcion, "void") == 0);
    int es_void_expr = (strcmp(tipo_expresion, "void") == 0);
    if (es_void_func || es_void_expr) {
        /* void solo cuadra con void; todo lo demás es error de tipo */
        return (es_void_func && es_void_expr) ? 0 : 1;
    }

    int w_dest = ancho_tipo(tipo_funcion);
    int w_orig = ancho_tipo(tipo_expresion);
    if (w_dest < 0 || w_orig < 0) { return -1; }

    if (w_orig > w_dest) { return 1; }  /* tipo primero */

    if (rutas_con_return < rutas_totales) { return 2; }  /* ruta abierta */
    return 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Funciones sin retorno o retornos mixtos

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Salida literal del harness (6/6 OK)
```text
=== AUDITORIA SESION 17 - Validacion de retornos y rutas ===
Caso                               esperado:              obtenido:              resultado
[ 1] int con int y dos rutas            esperado: 0            obtenido: 0            OK
[ 2] void devuelve valor                esperado: 1            obtenido: 1            OK
[ 3] ruta sin retorno                   esperado: 2            obtenido: 2            OK
[ 4] double acepta int                  esperado: 0            obtenido: 0            OK
[ 5] int no acepta double               esperado: 1            obtenido: 1            OK
[ 6] cero rutas de ejecucion            esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion17.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La primera propuesta que me devolvió solo comparaba los conteos de rutas y
> no tenía los guardias de argumentos inválidos: en el caso 2
> (`void devuelve valor`) regresaba `0` en vez de `1` porque las rutas cuadraban
> 1 de 1 y nunca llegó a comparar el tipo, y en el caso 6 (`cero rutas`) en vez
> de `-1` devolvía `0` porque "0 == 0".  
> **Cómo lo detecté:** leí los criterios de aceptación del laboratorio (ahí
> dice textual que el tipo se compara antes que las rutas) y probé a mano
> `diagnosticar_retorno("void", "int", 1, 1)` → tenía que ser `1`, y
> `diagnosticar_retorno("int", "int", 0, 0)` → `-1`.  
> **Corrección:** puse primero los guardias de NULL y de conteos sin sentido,
> luego la comparación de tipos con la tabla de anchos (con `void` tratado
> aparte) y solo al final la revisión de rutas, quedando el orden
> `-1 → 1 → 2 → 0`.

---
**Firma digital:** Paola-23-ux

# Bitácora de Requerimientos y Auditoría IA — Sesión 18
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Arreglos y Verificación de Límites  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350.  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Cálculo de tamaño de arreglo en memoria  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> El clásico **off-by-one**: yo había escrito la condición como
> `indice <= elementos`, pensando en "hasta la posición 5", y con eso el caso 5
> (`indice 5 de 5`) me regresaba `1` cuando un arreglo de 5 solo tiene
> posiciones 0 a 4. Hay que usar `indice < elementos`.  
> Otro detalle: si hubiera hecho `elementos - 1` con `elementos = 0`, me daría
> `-1` y un índice `-1` parecería válido; por eso valido primero que haya
> elementos y que el índice no sea negativo. También cuidado con convertir a
> `unsigned`: `-1` se vuelve enorme y pasa como "en rango".  
> En `bytes_arreglo`, lo feo fue multiplicar sin fijarme en dimensiones
> negativas: `int[-4]` tendría que devolver `-1`, no un tamaño raro.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Indexación entera no negativa |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto: Estoy en el laboratorio 18 de arreglos. Necesito calcular cuántos
bytes ocupa un arreglo estático según su tipo y cuántos elementos tiene, y
también validar si un índice se puede leer sin salir del arreglo. Compilo con
GCC en C11 con -Wall -Wextra -Werror y sin librerías fuera del estándar.

Rol: Que me ayudes como ingeniero de sistemas que sabe de memoria y límites
de arreglos, porque esto es para la parte del compilador que calcula el
tamaño de un arreglo.

Especificación: Dos funciones con la firma exacta de api_sesion18.h, sin
main():
  long bytes_arreglo(const char *tipo, int elementos);
  int  indice_en_rango(int indice, int elementos);
bytes_arreglo():
 - Tipos válidos: char (1), short (2), int (4), float (4), double (8) y
   puntero (8); cualquier otro nombre, incluido NULL, devuelve -1.
 - Si elementos es negativo devuelve -1.
 - Si elementos es 0 devuelve 0 bytes.
 - Si no, devuelve elementos * ancho_del_tipo.
indice_en_rango():
 - Devuelve 1 si 0 <= indice < elementos (el índice válido más alto es
   elementos - 1, nunca elementos).
 - Devuelve 0 si el índice es negativo o si elementos es 0 o negativo.
 - Ojo: sin castear a unsigned ni usar elementos - 1 cuando elementos = 0,
   que ahí se me atraviesa el -1.

Output: Solo las dos funciones en C99, sin main(), complejidad O(1), para
pegarlas en ia_sesion18_c.c.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion18_c.c - Fase 2: El Científico */
#include "api_sesion18.h"
#include <string.h>

long bytes_arreglo(const char *tipo, int elementos) {
    if (tipo == NULL || elementos < 0) { return -1; }

    long ancho;
    if      (strcmp(tipo, "char") == 0)   { ancho = 1; }
    else if (strcmp(tipo, "short") == 0)  { ancho = 2; }
    else if (strcmp(tipo, "int") == 0)    { ancho = 4; }
    else if (strcmp(tipo, "float") == 0)  { ancho = 4; }
    else if (strcmp(tipo, "double") == 0) { ancho = 8; }
    else if (strcmp(tipo, "puntero") == 0){ ancho = 8; }
    else { return -1; }

    return ancho * (long)elementos;
}

int indice_en_rango(int indice, int elementos) {
    if (elementos <= 0) { return 0; }   /* arreglo vacío: nada es válido */
    if (indice < 0)     { return 0; }   /* índice negativo: se rechaza */
    return (indice < elementos) ? 1 : 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Límites estáticos y dimensiones inválidas

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Salida literal del harness (6/6 OK)
```text
=== AUDITORIA SESION 18 - Tamano de arreglo y limites ===
Caso                               esperado:              obtenido:              resultado
[ 1] int[10]                            esperado: 40           obtenido: 40           OK
[ 2] double[3]                          esperado: 24           obtenido: 24           OK
[ 3] int[-4]                            esperado: -1           obtenido: -1           OK
[ 4] indice 0 de 5                      esperado: 1            obtenido: 1            OK
[ 5] indice 5 de 5                      esperado: 0            obtenido: 0            OK
[ 6] indice negativo                    esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion18.md
```

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La primera propuesta que me regresó solo checaba `indice < elementos` y no
> validaba la dimensión: `indice_en_rango(-1, 5)` devolvía `1` porque `-1 < 5`
> es cierto en C, y `bytes_arreglo("int", -4)` regresaba `-16` en vez de `-1`
> porque multiplicaba sin revisar que los elementos no fueran negativos.  
> **Cómo lo detecté:** comparé contra los casos 5 y 6 del harness y contra el
> criterio *"Los índices negativos se rechazan sin desbordar la
> comparación"*; probé a mano `indice_en_rango(-1, 5)` (tenía que ser `0`) y
> `indice_en_rango(5, 5)` (tenía que ser `0`, no `1`).  
> **Corrección:** puse primero `if (elementos <= 0) return 0;` para no caer en
> `elementos - 1` con arreglo vacío, luego `if (indice < 0) return 0;` sin
> convertir a `unsigned`, y dejé la comparación estricta
> `indice < elementos`. En `bytes_arreglo` agregué la guardia
> `elementos < 0 → -1` antes de multiplicar.

---
**Firma digital:** Paola-23-ux

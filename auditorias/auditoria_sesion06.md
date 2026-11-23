# Bitácora de Requerimientos y Auditoría IA — Sesión 06

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Sistema de Tipos Primitivos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350  
**Track:** C  
**Fecha:** 23 de noviembre de 2026

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Chequeo de compatibilidad de tipos.

### Problema o restricción identificada al codificar a mano

Durante esta fase realicé la implementación manual de las funciones `compatible()` y `nombre_tipo()` en lenguaje C. Al principio tuve dificultades con la sintaxis, porque algunas instrucciones quedaron fuera de las funciones, lo que provocó errores de compilación.

También tuve que comprender cómo funciona la jerarquía de tipos `CHAR < INT < FLOAT`, ya que las asignaciones deben realizarse de un tipo de menor rango a otro de igual o mayor rango.

Para solucionar los errores corregí la estructura de las funciones, revisé las llaves y utilicé condiciones para rechazar `TIPO_ERROR`.

Después de realizar las correcciones, ejecuté el harness y conseguí que las seis pruebas fueran satisfactorias.

**Resultado del Artesano: 6/6 pruebas superadas.**

### Código implementado manualmente

```c
#include "api_sesion06.h"

int compatible(Tipo destino, Tipo origen) {
    if (destino == TIPO_ERROR || origen == TIPO_ERROR) {
        return 0;
    }

    return destino >= origen;
}

const char *nombre_tipo(Tipo t) {
    switch (t) {
        case TIPO_CHAR:
            return "char";
        case TIPO_INT:
            return "int";
        case TIPO_FLOAT:
            return "float";
        case TIPO_ERROR:
            return "error";
        default:
            return "desconocido";
    }
}
```

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

En esta fase utilicé inteligencia artificial para generar una segunda implementación del sistema de compatibilidad de tipos, respetando las mismas funciones y reglas establecidas en el contrato.

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| Contexto (C) | C99 / GCC, sin bibliotecas no estándar |
| Rol (R) | Ingeniero de software de sistemas y compiladores |
| Especificación (E) | Matriz de compatibilidad estática de tipos primitivos |
| Output (O) | Código modular y análisis de complejidad |

### 2.2 Prompt Estructurado

**Contexto (C):**

Estoy desarrollando una práctica de Lenguajes y Autómatas II sobre el sistema de tipos primitivos y la verificación semántica en lenguaje C. Necesito implementar una tabla de compatibilidad para determinar si una asignación entre dos tipos es válida.

**Rol (R):**

Actúa como un ingeniero de software especializado en compiladores, análisis semántico y programación en lenguaje C.

**Especificación (E):**

Desarrolla las funciones `compatible(Tipo destino, Tipo origen)` y `nombre_tipo(Tipo t)` respetando las declaraciones de `api_sesion06.h`.

La jerarquía de tipos es `CHAR < INT < FLOAT`.

La función de compatibilidad debe aceptar asignaciones cuando el destino tenga un rango igual o mayor que el origen. Debe rechazar asignaciones que puedan perder información y cualquier operación que involucre `TIPO_ERROR`.

La función `nombre_tipo()` debe devolver el nombre correspondiente a cada tipo.

Utiliza una función auxiliar para identificar los rangos y una estructura diferente de la implementación manual.

**Output (O):**

Genera código modular en C, sin función `main()`, compatible con GCC y las opciones `-std=c11 -Wall -Wextra -Werror`. Incluye un análisis breve de la complejidad de las funciones.

---

## 3. Código Devuelto por el Modelo

La inteligencia artificial generó una implementación que utiliza una función auxiliar llamada `obtener_rango()` para identificar la jerarquía de los tipos. También utiliza un arreglo de cadenas para devolver sus nombres.

```c
#include "api_sesion06.h"

static int obtener_rango(Tipo tipo) {
    switch (tipo) {
        case TIPO_CHAR:
            return 1;
        case TIPO_INT:
            return 2;
        case TIPO_FLOAT:
            return 3;
        default:
            return -1;
    }
}

int compatible(Tipo destino, Tipo origen) {
    int rango_destino = obtener_rango(destino);
    int rango_origen = obtener_rango(origen);

    if (rango_destino == -1 || rango_origen == -1) {
        return 0;
    }

    if (rango_destino >= rango_origen) {
        return 1;
    }

    return 0;
}

const char *nombre_tipo(Tipo t) {
    static const char *nombres[] = {
        "error",
        "char",
        "int",
        "float"
    };

    if (t >= TIPO_ERROR && t <= TIPO_FLOAT) {
        return nombres[t];
    }

    return "desconocido";
}
```

### Análisis de complejidad

La función `obtener_rango()` tiene complejidad temporal O(1), porque utiliza un número fijo de casos.

La función `compatible()` también tiene complejidad O(1), ya que solamente realiza dos consultas de rango y una comparación.

La función `nombre_tipo()` utiliza un arreglo de tamaño fijo, por lo que su acceso también tiene complejidad O(1).

En general, ambas implementaciones tienen complejidad temporal y espacial constante, O(1).

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Rechazo de asignaciones incompatibles y verificación de los nombres de los tipos.

Durante esta fase utilicé el archivo `harness_sesion06_c.c` para comprobar el funcionamiento de las implementaciones del Artesano y del Científico.

El harness ejecutó seis pruebas automáticas y comparó los resultados obtenidos con los valores esperados.

### Resultados del Artesano

| Caso | Esperado | Obtenido | Resultado |
|---|---|---|---|
| compatible(INT,INT) | 1 | 1 | OK |
| compatible(FLOAT,INT) | 1 | 1 | OK |
| compatible(INT,FLOAT) | 0 | 0 | OK |
| compatible(INT,ERROR) | 0 | 0 | OK |
| nombre_tipo(INT) | int | int | OK |
| nombre_tipo(ERROR) | error | error | OK |

**RESULTADO: 6/6 pruebas superadas.**

**TODO EN VERDE.**

### Resultados del Científico

| Caso | Esperado | Obtenido | Resultado |
|---|---|---|---|
| compatible(INT,INT) | 1 | 1 | OK |
| compatible(FLOAT,INT) | 1 | 1 | OK |
| compatible(INT,FLOAT) | 0 | 0 | OK |
| compatible(INT,ERROR) | 0 | 0 | OK |
| nombre_tipo(INT) | int | int | OK |
| nombre_tipo(ERROR) | error | error | OK |

**RESULTADO: 6/6 pruebas superadas.**

**TODO EN VERDE.**

### Verificación de los criterios de auditoría

- [x] Los seis casos del harness fueron superados sin errores de ejecución.
- [x] Las dos implementaciones compilaron con las opciones solicitadas.
- [x] La complejidad teórica de las funciones es constante, O(1).
- [ ] No existen accesos fuera de límites ni fugas de memoria: pendiente de una verificación específica.

### Comparación entre Artesano y Científico

El Artesano utiliza una implementación sencilla basada en la comparación directa de los valores del tipo enumerado y una estructura `switch` para obtener los nombres.

En cambio, el Científico utiliza una función auxiliar que permite obtener el rango de cada tipo y un arreglo de cadenas para identificar los nombres.

Aunque ambas implementaciones son diferentes, las dos consiguieron superar las seis pruebas del harness.

Esto demuestra que un mismo problema puede resolverse mediante diferentes estructuras de programación y producir los mismos resultados.

---

## 5. Reporte Crítico

### ¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?

Durante las pruebas realizadas no se encontró ningún error en los seis casos evaluados por el harness. La inteligencia artificial implementó correctamente las reglas de compatibilidad establecidas.

Un caso importante fue la validación de `TIPO_ERROR`, ya que este tipo no debe ser compatible con ningún otro. La implementación de IA lo rechaza mediante la función auxiliar `obtener_rango()`, que devuelve `-1` cuando el tipo no es válido.

También observé que el harness solamente evalúa seis casos, por lo que no comprueba todas las combinaciones posibles. Para mejorar la auditoría sería conveniente agregar pruebas como `compatible(CHAR,FLOAT)`, `compatible(FLOAT,CHAR)` y `compatible(ERROR,ERROR)`.

No fue necesario modificar el código generado por la IA para superar las pruebas originales.

### Reflexión sobre el uso de inteligencia artificial

La inteligencia artificial facilitó la creación de una implementación modular y permitió utilizar una estructura diferente de la solución manual.

Sin embargo, comprendí que no es suficiente obtener código de una IA, porque también es necesario revisarlo, compilarlo y comprobar su funcionamiento mediante pruebas.

El harness permitió verificar de manera objetiva si las funciones cumplían con los requerimientos establecidos.


---

**Firma digital:** [Paola-23-ux]

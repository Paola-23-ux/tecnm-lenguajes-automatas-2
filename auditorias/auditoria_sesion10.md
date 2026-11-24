
# Bitácora de Requerimientos y Auditoría IA — Sesión 10

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Hito 1: Mini-Evaluador de Expresiones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350  
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Integración AST y Tipado Estático.

Durante esta fase tuve dificultades para implementar las funciones encargadas de evaluar las expresiones matemáticas, principalmente al establecer la jerarquía de los operadores y reconocer los paréntesis.

Necesité ayuda de la inteligencia artificial para elaborar una primera versión del código, en la cual solamente se reconocían números enteros, espacios, sumas y restas.

La implementación inicial no contemplaba las multiplicaciones, divisiones, paréntesis ni la detección de división entre cero.

### Código inicial del Artesano

```c
#include "api_sesion10.h"
#include <stdio.h>
#include <ctype.h>

typedef struct {
    const char *s;
    int error;
} Parser;

static void saltar(Parser *p) {
    while (*p->s == ' ') {
        p->s++;
    }
}

long expr(Parser *p);

long primaria(Parser *p) {
    saltar(p);

    long numero = 0;

    if (!isdigit((unsigned char)*p->s)) {
        p->error = 2;
        return 0;
    }

    while (isdigit((unsigned char)*p->s)) {
        numero = numero * 10 + (*p->s - '0');
        p->s++;
    }

    return numero;
}

long termino(Parser *p) {
    long valor = primaria(p);

    // Pendiente: multiplicacion y division.

    return valor;
}

long expr(Parser *p) {
    long valor = termino(p);

    saltar(p);

    while (*p->s == '+' || *p->s == '-') {
        char operador = *p->s;
        p->s++;

        long numero = termino(p);

        if (operador == '+') {
            valor = valor + numero;
        } else {
            valor = valor - numero;
        }

        saltar(p);
    }

    return valor;
}

long evaluar_expresion(const char *fuente, int *error) {
    static int sin_error;

    if (error == NULL)
        error = &sin_error;

    *error = 0;

    if (fuente == NULL) {
        *error = 2;
        return 0;
    }

    Parser p = {fuente, 0};

    long v = expr(&p);

    saltar(&p);

    if (p.error == 0 && *p.s != '\0')
        p.error = 2;

    *error = p.error;

    return p.error != 0 ? 0 : v;
}
```

### Correcciones realizadas

Se modificó la función primaria() para reconocer los paréntesis mediante llamadas recursivas a expr().

También se completó la función termino() para realizar multiplicaciones y divisiones antes de las sumas y restas.

Se agregó una condición para detectar cuando el divisor es cero y devolver el código de error 1.

Finalmente, se mejoró la función expr() para evitar continuar evaluando cuando se detecta un error.

Estas modificaciones permitieron que la versión final del Artesano superara las seis pruebas del harness.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| Contexto (C) | Lenguaje C, estándar C11, compilador GCC y Google Colab, sin bibliotecas no estándar. |
| Rol (R) | Ingeniero de software especializado en compiladores y análisis sintáctico. |
| Especificación (E) | Implementar un evaluador de expresiones matemáticas que respete la precedencia de operadores, reconozca paréntesis y detecte errores. |
| Output (O) | Código modular compatible con la API, utilizando strtol(), isspace() y switch, sin incluir main(). |

### 2.2 Prompt Estructurado

**Contexto:**

Estoy realizando la sesión 10 de Lenguajes y Autómatas II sobre el mini-evaluador de expresiones matemáticas en lenguaje C.

**Rol:**

Actúa como un programador especializado en compiladores y análisis sintáctico.

**Especificación:**

Implementa un evaluador de expresiones que reconozca números enteros, espacios, paréntesis, sumas, restas, multiplicaciones y divisiones.

Respeta la precedencia de operadores.

Devuelve error 1 para división entre cero y error 2 para expresiones inválidas.

Respeta las firmas declaradas en api_sesion10.h.

**Output:**

Entrega código compatible con C11, utilizando strtol(), isspace() y switch.

No incluyas la función main().

El código debe ser compatible con el harness proporcionado por el docente.

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion10.h"
#include <stdlib.h>
#include <ctype.h>
#include <errno.h>

typedef struct {
    const char *cursor;
    int error;
} Analizador;

static void espacios(Analizador *a) {
    while (isspace((unsigned char)*a->cursor)) {
        a->cursor++;
    }
}

static long expresion(Analizador *a);

static long primaria(Analizador *a) {
    espacios(a);

    switch (*a->cursor) {
        case '(': {
            a->cursor++;

            long valor = expresion(a);
            espacios(a);

            if (*a->cursor == ')') {
                a->cursor++;
            } else if (a->error == 0) {
                a->error = 2;
            }

            return valor;
        }

        default: {
            if (!isdigit((unsigned char)*a->cursor)) {
                a->error = 2;
                return 0;
            }

            char *fin;
            errno = 0;

            long valor = strtol(a->cursor, &fin, 10);

            if (errno == ERANGE) {
                a->error = 2;
                return 0;
            }

            a->cursor = fin;
            return valor;
        }
    }
}

static long termino(Analizador *a) {
    long valor = primaria(a);

    while (a->error == 0) {
        espacios(a);

        char op = *a->cursor;

        if (op != '*' && op != '/') {
            break;
        }

        a->cursor++;
        long derecho = primaria(a);

        if (a->error != 0) {
            return 0;
        }

        switch (op) {
            case '*':
                valor *= derecho;
                break;

            case '/':
                if (derecho == 0) {
                    a->error = 1;
                    return 0;
                }

                valor /= derecho;
                break;
        }
    }

    return valor;
}

static long expresion(Analizador *a) {
    long valor = termino(a);

    while (a->error == 0) {
        espacios(a);

        char op = *a->cursor;

        if (op != '+' && op != '-') {
            break;
        }

        a->cursor++;
        long derecho = termino(a);

        if (a->error != 0) {
            return 0;
        }

        switch (op) {
            case '+':
                valor += derecho;
                break;

            case '-':
                valor -= derecho;
                break;
        }
    }

    return valor;
}

long evaluar_expresion(const char *fuente, int *error) {
    int auxiliar = 0;

    if (error == NULL) {
        error = &auxiliar;
    }

    if (fuente == NULL) {
        *error = 2;
        return 0;
    }

    Analizador a = {fuente, 0};

    long resultado = expresion(&a);

    espacios(&a);

    if (a.error == 0 && *a.cursor != '\0') {
        a.error = 2;
    }

    *error = a.error;

    return a.error == 0 ? resultado : 0;
}
```

### Análisis de complejidad

**Complejidad temporal: O(n).**

El evaluador recorre los caracteres de la expresión matemática para identificar números, operadores, espacios y paréntesis.

El tiempo de ejecución depende de la longitud de la expresión, representada por n.

**Complejidad espacial auxiliar: O(d).**

La implementación utiliza llamadas recursivas para evaluar expresiones que contienen paréntesis, donde d representa la profundidad de anidamiento.

En el peor caso, cuando existen muchos paréntesis anidados, la complejidad espacial puede llegar a ser O(n).

### Comparación entre el Artesano y el Científico

| Característica | Artesano | Científico |
|---|---|---|
| Estructura principal | Parser | Analizador |
| Lectura de números | Dígito por dígito | strtol() |
| Reconocimiento de espacios | Comparación con espacio simple | isspace() |
| Estructuras de control | if y else | switch y case |
| Detección de números fuera de rango | No incluida | errno y ERANGE |
| Detección de división entre cero | Sí | Sí |
| Reconocimiento de paréntesis | Sí | Sí |
| Complejidad temporal | O(n) | O(n) |
| Complejidad espacial auxiliar | O(d) | O(d) |
| Pruebas superadas | 6/6 | 6/6 |

Ambas implementaciones permiten evaluar expresiones matemáticas respetando la precedencia de operadores. Sin embargo, el Científico incorpora funciones de la biblioteca estándar que facilitan la lectura de números y la detección de algunos errores.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Seis casos de evaluación de expresiones matemáticas, precedencia de operadores, paréntesis, espacios y detección de errores para cada implementación.

### Resultado del Artesano

```text
=== AUDITORIA SESION 10 - Mini-evaluador de expresiones (Hito 1) ===

[ 1] "2+3*4"
     esperado: 14
     obtenido: 14
     OK

[ 2] "(2+3)*4"
     esperado: 20
     obtenido: 20
     OK

[ 3] "10/0" -> error 1
     esperado: 1
     obtenido: 1
     OK

[ 4] "2+" -> error 2
     esperado: 2
     obtenido: 2
     OK

[ 5] "  7 "
     esperado: 7
     obtenido: 7
     OK

[ 6] "5-10"
     esperado: -5
     obtenido: -5
     OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion10.md
```

### Resultado del Científico

```text
=== AUDITORIA SESION 10 - Mini-evaluador de expresiones (Hito 1) ===

[ 1] "2+3*4"
     esperado: 14
     obtenido: 14
     OK

[ 2] "(2+3)*4"
     esperado: 20
     obtenido: 20
     OK

[ 3] "10/0" -> error 1
     esperado: 1
     obtenido: 1
     OK

[ 4] "2+" -> error 2
     esperado: 2
     obtenido: 2
     OK

[ 5] "  7 "
     esperado: 7
     obtenido: 7
     OK

[ 6] "5-10"
     esperado: -5
     obtenido: -5
     OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion10.md
```

### Tabla de resultados

| Caso evaluado | Esperado | Artesano | Científico | Estado |
|---|---|---|---|---|
| 2+3*4 | 14 | 14 | 14 | OK |
| (2+3)*4 | 20 | 20 | 20 | OK |
| 10/0 | Error 1 | Error 1 | Error 1 | OK |
| 2+ | Error 2 | Error 2 | Error 2 | OK |
| Número 7 con espacios | 7 | 7 | 7 | OK |
| 5-10 | -5 | -5 | -5 | OK |

**Resultado del Artesano:** 6/6 pruebas superadas.

**Resultado del Científico:** 6/6 pruebas superadas.

**Resultado total:** 12/12 pruebas superadas entre ambas ejecuciones.

### Verificación de criterios

- [x] Caso base y límites evaluados superados sin Crash.
- [ ] Ausencia de accesos fuera de límites y Memory Leaks comprobada mediante herramientas especializadas.
- [x] La complejidad teórica coincide con la implementada.
- [x] El Artesano superó las seis pruebas.
- [x] El Científico superó las seis pruebas.
- [x] Se respetó la precedencia de operadores en los casos evaluados.
- [x] Se reconocieron correctamente los paréntesis.
- [x] Se detectó la división entre cero.
- [x] Se identificó la expresión incompleta.
- [x] No se presentaron errores de compilación.

No se realizaron pruebas especializadas de memoria. Aunque ambas implementaciones superaron el harness, todavía existen casos adicionales que podrían evaluarse.

### Reporte Crítico

**¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?**

Durante la revisión del código generado por la inteligencia artificial se identificó un posible problema relacionado con el desbordamiento de números enteros durante las operaciones aritméticas.

Aunque el Científico utiliza strtol() y comprueba ERANGE para detectar números fuera del rango del tipo long, no verifica si el resultado de una suma, resta, multiplicación o división excede los límites permitidos.

Por ejemplo, una expresión formada por el valor máximo de long seguido de una suma de 1 podría provocar un desbordamiento.

En una plataforma donde long sea de 64 bits, un ejemplo sería:

```text
9223372036854775807+1
```

Este caso podría producir un comportamiento indefinido en lenguaje C.

El problema se identificó mediante el análisis del código y no mediante una prueba ejecutada en el harness.

### Propuesta de solución

Una posible solución consiste en utilizar la biblioteca limits.h, que contiene las constantes LONG_MAX y LONG_MIN.

Antes de realizar una operación aritmética se pueden comprobar los límites permitidos.

Por ejemplo, para una suma:

```c
#include <limits.h>

if ((derecho > 0 && valor > LONG_MAX - derecho) ||
    (derecho < 0 && valor < LONG_MIN - derecho)) {
    a->error = 2;
    return 0;
}

valor += derecho;
```

También sería necesario incorporar validaciones específicas para las restas, multiplicaciones y divisiones.

Esta modificación se presenta únicamente como una propuesta de mejora. No fue incorporada ni ejecutada durante las pruebas originales.

---

**Firma digital:** Paola-23-ux

# Bitácora de Requerimientos y Auditoría IA — Sesión 07

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Evaluación de Expresiones Aritméticas  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350  
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Evaluador recursivo de operaciones

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

Durante la implementación del evaluador recursivo, una de las dificultades fue comprender cómo recorrer los nodos del árbol y realizar las operaciones correctamente. También fue necesario verificar los valores antes de realizar divisiones y evitar que el programa fallara cuando existiera una división entre cero. Para resolverlo se utilizaron condiciones `if` y se agregaron validaciones para identificar los errores.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| Contexto (C) | C99 / GCC, sin bibliotecas no estándar |
| Rol (R) | Ingeniero de software de sistemas y compiladores |
| Especificación (E) | Manejo estricto de división entre cero |
| Output (O) | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado

**Contexto (C):** Estoy realizando una práctica de Lenguajes y Autómatas II en lenguaje C, utilizando GCC. Necesito implementar un evaluador recursivo de expresiones aritméticas mediante árboles AST.

**Rol (R):** Actúa como un ingeniero de software especializado en sistemas y compiladores.

**Especificación (E):** Implementa la función `int evaluar(ASTNode *n, int *error)` respetando el archivo `api_sesion07.h`. Debe evaluar sumas, restas, multiplicaciones, divisiones y módulo. Si existe división o módulo entre cero, debe asignar error 1. Si el nodo es inválido, debe asignar error 2. Los errores deben propagarse sin continuar evaluando los nodos superiores.

**Output (O):** Proporciona únicamente código modular en C, sin función `main()`, compatible con GCC y sin advertencias de compilación. Incluye un análisis de complejidad.

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion07.h"
#include <stddef.h>

int evaluar(ASTNode *n, int *error) {
    int a, b;

    if (error == NULL) {
        return 0;
    }

    *error = 0;

    if (n == NULL) {
        *error = 2;
        return 0;
    }

    switch (n->tipo) {
        case NODO_NUM:
            return n->valor;

        case NODO_BINOP:
            a = evaluar(n->izq, error);
            if (*error != 0) {
                return 0;
            }

            b = evaluar(n->der, error);
            if (*error != 0) {
                return 0;
            }

            switch (n->op) {
                case '+':
                    return a + b;

                case '-':
                    return a - b;

                case '*':
                    return a * b;

                case '/':
                    if (b == 0) {
                        *error = 1;
                        return 0;
                    }
                    return a / b;

                case '%':
                    if (b == 0) {
                        *error = 1;
                        return 0;
                    }
                    return a % b;

                default:
                    *error = 2;
                    return 0;
            }

        default:
            *error = 2;
            return 0;
    }
}
```

**Análisis de complejidad:** La complejidad temporal es O(n), donde n representa el número de nodos visitados. La complejidad espacial auxiliar es O(h), donde h es la altura del árbol debido a las llamadas recursivas.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Expresiones complejas y división por 0.

- [x] Caso base y límites superados sin Crash.
- [ ] No existen accesos fuera de límites ni Memory Leaks. Pendiente de verificar con herramientas específicas.
- [x] La complejidad teórica coincide con el análisis de la implementación.

**Resultados del Test Harness**

| Caso | Resultado esperado | Artesano | Científico |
|---|---|---|---|
| (2+3)*4 | 20 | OK | OK |
| 7/0 | Error 1 | OK | OK |
| 7%0 | Error 1 | OK | OK |
| Árbol NULL | Error 2 | OK | OK |
| 10-2-3 | 5 | OK | OK |
| (8/2)+1 | 5 | OK | OK |

**Resultado del Artesano:** 6/6 pruebas superadas.

**Resultado del Científico:** 6/6 pruebas superadas.

### Salida literal del harness (corrida real desde Colab)

**Celda 1 — `artesano_sesion07_c.c`:**

```text
=== AUDITORIA SESION 07 - Evaluador recursivo con deteccion de errores ===
Caso                               esperado:              obtenido:              resultado
[ 1] (2+3)*4                            esperado: 20           obtenido: 20           OK
[ 2] 7/0 -> error 1                     esperado: 1            obtenido: 1            OK
[ 3] 7%0 -> error 1                     esperado: 1            obtenido: 1            OK
[ 4] arbol NULL -> error 2              esperado: 2            obtenido: 2            OK
[ 5] 10-2-3                             esperado: 5            obtenido: 5            OK
[ 6] (8/2)+1                            esperado: 5            obtenido: 5            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion07.md
```

**Celda 2 — `ia_sesion07_c.c`:**

```text
=== AUDITORIA SESION 07 - Evaluador recursivo con deteccion de errores ===
Caso                               esperado:              obtenido:              resultado
[ 1] (2+3)*4                            esperado: 20           obtenido: 20           OK
[ 2] 7/0 -> error 1                     esperado: 1            obtenido: 1            OK
[ 3] 7%0 -> error 1                     esperado: 1            obtenido: 1            OK
[ 4] arbol NULL -> error 2              esperado: 2            obtenido: 2            OK
[ 5] 10-2-3                             esperado: 5            obtenido: 5            OK
[ 6] (8/2)+1                            esperado: 5            obtenido: 5            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion07.md
```

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

Durante la compilación del código generado por la IA se detectó un error porque la constante `NULL` no estaba declarada. Esto ocurrió porque faltaba incluir la biblioteca `<stddef.h>`. Para solucionar el problema agregué esta biblioteca al código y ejecuté nuevamente la compilación. Después de realizar la corrección, el programa compiló correctamente y superó las seis pruebas del harness. Esto permitió comprobar que el código generado por IA necesita ser revisado y probado antes de utilizarse.

---

**Firma digital:** [Paola-23-ux]

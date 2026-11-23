
# Bitácora de Requerimientos y Auditoría IA — Sesión 05

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Recorridos del AST y Notación Polaca Inversa (RPN)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia.  
**Matrícula:** 24531350. 
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Impresión en Postorden / RPN.

### Dificultades encontradas

Durante la implementación manual de la función `postorden()`, la principal dificultad fue comprender el recorrido recursivo de un árbol de sintaxis abstracta (AST).

Se identificó que el recorrido debía seguir el orden:

1. Visitar el subárbol izquierdo.
2. Visitar el subárbol derecho.
3. Procesar la raíz.

Otra dificultad fue administrar correctamente la memoria del buffer, evitando escribir fuera de su capacidad.

También fue necesario asegurar que los elementos estuvieran separados por un solo espacio y que la cadena terminara con el carácter nulo `'\0'`.

### Solución implementada

Se utilizó una función auxiliar recursiva para recorrer los nodos del árbol y almacenar los números y operadores en un buffer.

Se controló la cantidad de caracteres escritos y se verificó el espacio disponible antes de agregar cada elemento.

### Resultado

La implementación del Artesano compiló correctamente y superó las cinco pruebas del harness.

**Resultado: 5/5 pruebas superadas.**

---

## 2. Fase 2: El Científico
### Ingeniería de Requerimientos C-R-E-O

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| Contexto (C) | Lenguaje C, GCC y árboles de sintaxis abstracta |
| Rol (R) | Ingeniero de software especializado en compiladores |
| Especificación (E) | Recorrido recursivo en postorden y generación de RPN |
| Output (O) | Código modular en C y análisis de complejidad |

### 2.2 Prompt Estructurado

**Contexto (C):**

Estoy desarrollando una actividad de Lenguajes y Autómatas II sobre árboles de sintaxis abstracta (AST) y generación de notación polaca inversa (RPN), utilizando lenguaje C.

**Rol (R):**

Actúa como un ingeniero de software especializado en lenguaje C, compiladores, estructuras de datos y algoritmos recursivos.

**Especificación (E):**

Desarrolla una función llamada `postorden()` que reciba un puntero a un nodo AST, un buffer de caracteres y su capacidad.

La función debe cumplir las siguientes condiciones:

1. Recorrer el árbol en postorden: izquierda, derecha y raíz.
2. Escribir los números y operadores en un buffer.
3. Separar los elementos con un solo espacio.
4. No agregar espacios al final de la cadena.
5. Si el árbol es NULL, producir una cadena vacía.
6. Respetar la capacidad del buffer.
7. Evitar desbordamientos de memoria.
8. Garantizar la terminación con `'\0'`.
9. No utilizar `printf()` para imprimir el resultado.
10. No incluir una función `main()`.

La firma debe ser exactamente:

`void postorden(ASTNode *n, char *buf, int cap);`

**Output (O):**

Proporciona el código completo en lenguaje C, compatible con C11 y GCC.

El código debe compilar utilizando las opciones:

`-Wall -Wextra -Werror`

También debe incluir una explicación breve de su funcionamiento y complejidad.

---

## 3. Código Devuelto por el Modelo

La IA proporcionó una implementación que utiliza recursividad para recorrer el AST y generar la notación polaca inversa.

### Archivo: ia_sesion05_c.c

```c
#include "api_sesion05.h"
#include <stdio.h>
#include <string.h>

static void generar_rpn(ASTNode *n, char *buf,
                        int cap, int *pos) {
    if (n == NULL) {
        return;
    }

    generar_rpn(n->izq, buf, cap, pos);
    generar_rpn(n->der, buf, cap, pos);

    char token[32];
    int longitud;

    if (n->tipo == NODO_NUM) {
        longitud = snprintf(token, sizeof(token),
                            "%d", n->valor);
    } else if (n->tipo == NODO_BINOP) {
        longitud = snprintf(token, sizeof(token),
                            "%c", n->op);
    } else {
        return;
    }

    if (longitud < 0 ||
        longitud >= (int)sizeof(token)) {
        return;
    }

    int separador = (*pos > 0) ? 1 : 0;
    int necesario = longitud + separador;

    if (necesario >= cap - *pos) {
        return;
    }

    if (separador) {
        buf[*pos] = ' ';
        (*pos)++;
    }

    memcpy(buf + *pos, token, (size_t)longitud);

    *pos += longitud;
    buf[*pos] = '\0';
}

void postorden(ASTNode *n, char *buf, int cap) {
    if (buf == NULL || cap <= 0) {
        return;
    }

    buf[0] = '\0';

    int pos = 0;

    generar_rpn(n, buf, cap, &pos);
}
```

### Explicación del código

La función `generar_rpn()` recorre recursivamente los nodos del árbol.

Primero visita el subárbol izquierdo, después el derecho y finalmente procesa el nodo actual.

La función `snprintf()` convierte los números y operadores en cadenas.

La función `memcpy()` copia los elementos al buffer.

Se verifica el espacio disponible para evitar desbordamientos.

### Análisis de complejidad

**Complejidad temporal: O(n)**

Cada nodo del árbol se visita una vez, donde n representa la cantidad de nodos.

**Complejidad espacial: O(h)**

La recursividad utiliza memoria proporcional a la altura del árbol.

En un árbol balanceado, la altura es aproximadamente O(log n).

En un árbol degenerado, puede alcanzar O(n).

---

## 4. Fase 3: El Auditor
### Resultados del Test Harness

Se utilizó el archivo `harness_sesion05_c.c` para comprobar el funcionamiento de las implementaciones del Artesano y de la IA.

### 4.1 Auditoría del Artesano

Comando utilizado:

```bash
gcc -std=c11 -Wall -Wextra -Werror \
harness_sesion05_c.c artesano_sesion05_c.c \
-o test_artesano

./test_artesano
```

**Resultado obtenido:**

`RESULTADO: 5/5 pruebas superadas`

### 4.2 Auditoría de la IA

Comando utilizado:

```bash
gcc -std=c11 -Wall -Wextra -Werror \
harness_sesion05_c.c ia_sesion05_c.c \
-o test_ia

./test_ia
```

**Resultado obtenido:**

`RESULTADO: 5/5 pruebas superadas`

### 4.3 Casos evaluados

| Caso | Expresión | Resultado esperado | Estado |
|---|---|---|---|
| 1 | Hoja 42 | 42 | OK |
| 2 | 3+(5*2) | 3 5 2 * + | OK |
| 3 | Árbol NULL | Cadena vacía | OK |
| 4 | (1+2)*(3+4) | 1 2 + 3 4 + * | OK |
| 5 | 8-3 | 8 3 - | OK |

### 4.4 Validación

- [x] Los cinco casos del harness fueron superados.
- [x] El árbol NULL produjo una cadena vacía.
- [x] El recorrido respetó el orden izquierda, derecha y raíz.
- [x] Los resultados no presentaron espacios finales.
- [ ] Verificar buffers de capacidad reducida mediante pruebas adicionales.
- [ ] Verificar ausencia de accesos fuera de límites con herramientas de memoria.
- [x] Se realizó el análisis teórico de complejidad.

---

## 5. Reporte Crítico

### ¿Qué caso borde o problema omitió la IA y cómo se solucionó?

Durante la primera ejecución del código generado por la IA se presentó un error de compilación:

`error: unknown type name 'ASTNode'`

El problema ocurrió porque el archivo de implementación no reconocía la estructura ASTNode declarada en la cabecera del proyecto.

### Solución aplicada

Se corrigió el archivo `ia_sesion05_c.c` agregando la siguiente instrucción:

```c
#include "api_sesion05.h"
```

Después de guardar nuevamente el archivo, se ejecutó la auditoría y la implementación superó las cinco pruebas.

### Caso borde identificado

También se identificó que el harness proporcionado no incluye pruebas específicas con buffers de capacidad reducida.

Este caso es importante porque una implementación debe evitar escribir fuera del espacio disponible.

Aunque el código contiene verificaciones de capacidad, se requiere una prueba adicional para confirmar su comportamiento con buffers pequeños.

Por lo tanto, no se puede afirmar que exista un fallo de memoria únicamente con los resultados actuales.

### Comparación entre Artesano e IA

| Característica | Artesano | IA |
|---|---|---|
| Lenguaje | C | C |
| Recorrido postorden | Correcto | Correcto |
| Generación RPN | Correcta | Correcta |
| Árbol NULL | Correcto | Correcto |
| Pruebas superadas | 5/5 | 5/5 |
| Compilación final | Exitosa | Exitosa |

---


**Resultado final:**

- Artesano: 5/5 pruebas superadas.
- Científico (IA): 5/5 pruebas superadas.
- Auditor: cinco casos verificados correctamente.

---

## 7. Evidencias

Archivos utilizados:

- `api_sesion05.h`
- `artesano_sesion05_c.c`
- `ia_sesion05_c.c`
- `harness_sesion05_c.c`
- `auditorias/auditoria_sesion05.md`

Se conservaron las salidas de las auditorías del Artesano y de la IA, ambas con cinco pruebas superadas.

---

8. Firma Digital: Paola-23-ux

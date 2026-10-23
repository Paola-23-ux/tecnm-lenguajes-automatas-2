# Bitácora de Requerimientos y Auditoría IA — Sesión 02
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Gestión en GitHub Web y Nodos Dinámicos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350  
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Lista enlazada de tokens en Heap.

**Problema o restricción identificada:**

El principal reto fue comprender cómo utilizar memoria dinámica para crear una lista enlazada de tokens en lenguaje C.

Una de las dificultades fue manejar correctamente los punteros y reservar memoria mediante malloc para cada nodo.

También fue necesario comprender cómo conectar los nodos mediante el puntero siguiente y cómo liberar la memoria sin perder la referencia al siguiente elemento.

La implementación utiliza cuatro funciones: crear_nodo, agregar_token, contar_tokens y liberar_lista.

Al ejecutar el programa, se superaron las seis pruebas del harness.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Desarrollo de una lista enlazada de tokens en C11 utilizando GCC y memoria dinámica. |
| **Rol (R)** | Ingeniero de software especializado en lenguaje C, estructuras de datos y compiladores. |
| **Especificación (E)** | Implementar crear_nodo, agregar_token, contar_tokens y liberar_lista utilizando malloc y free, respetando api_sesion02.h. |
| **Output (O)** | Código modular en C11, sin main(), compatible con -Wall -Wextra -Werror. |

### 2.2 Prompt Estructurado

```text
Contexto:
Estoy realizando una práctica de Lenguajes y Autómatas II sobre listas enlazadas de tokens en memoria dinámica.

Rol:
Actúa como un programador experto en lenguaje C.

Especificación:
Implementa las funciones crear_nodo, agregar_token, contar_tokens y liberar_lista, respetando exactamente las firmas de api_sesion02.h.

Utiliza malloc, inicializa los punteros correctamente, evita fugas de memoria y permite trabajar con listas vacías.

Output:
Devuelve únicamente código C compatible con C11, sin función main(), que compile con -Wall -Wextra -Werror.
```

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion02.h"
#include <stdlib.h>
#include <string.h>

NodoToken *crear_nodo(const char *lexema) {
    if (lexema == NULL) {
        return NULL;
    }

    NodoToken *nodo = malloc(sizeof(NodoToken));

    if (nodo == NULL) {
        return NULL;
    }

    strncpy(nodo->lexema, lexema, sizeof(nodo->lexema) - 1);
    nodo->lexema[sizeof(nodo->lexema) - 1] = '\0';
    nodo->siguiente = NULL;

    return nodo;
}

void agregar_token(NodoToken **cabeza, const char *lexema) {
    if (cabeza == NULL) {
        return;
    }

    NodoToken *nuevo = crear_nodo(lexema);

    if (nuevo == NULL) {
        return;
    }

    if (*cabeza == NULL) {
        *cabeza = nuevo;
        return;
    }

    NodoToken *actual = *cabeza;

    while (actual->siguiente != NULL) {
        actual = actual->siguiente;
    }

    actual->siguiente = nuevo;
}

int contar_tokens(NodoToken *cabeza) {
    int total = 0;

    while (cabeza != NULL) {
        total++;
        cabeza = cabeza->siguiente;
    }

    return total;
}

void liberar_lista(NodoToken *cabeza) {
    while (cabeza != NULL) {
        NodoToken *siguiente = cabeza->siguiente;
        free(cabeza);
        cabeza = siguiente;
    }
}
```

### Análisis de complejidad

- crear_nodo: O(1), porque reserva memoria para un nodo y copia una cantidad limitada de caracteres.
- agregar_token: O(n), porque recorre la lista hasta encontrar el último nodo.
- contar_tokens: O(n), porque recorre todos los elementos.
- liberar_lista: O(n), porque libera cada nodo individualmente.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Creación de nodos, conteo de tokens, orden de elementos, punteros NULL y liberación de memoria.

### Resultados obtenidos

| Caso | Prueba | Artesano | IA |
| :--- | :--- | :--- | :--- |
| 1 | crear_nodo("if") | OK | OK |
| 2 | contar_tokens(3 tokens) | OK | OK |
| 3 | Orden del segundo token | OK | OK |
| 4 | Último nodo apunta a NULL | OK | OK |
| 5 | contar_tokens(NULL) | OK | OK |
| 6 | liberar_lista(NULL) | OK | OK |

**Resultado Artesano:** 6/6 pruebas superadas.

**Resultado IA:** 6/6 pruebas superadas.

Ambos programas compilaron correctamente y finalizaron con código de salida 0.

### Evidencia del Harness

```text
=== AUDITORIA SESION 02 - Lista enlazada de tokens en heap ===

[ 1] crear_nodo("if")->lexema       OK
[ 2] contar_tokens(3 tokens)        OK
[ 3] orden: segundo token           OK
[ 4] ultimo->siguiente == NULL      OK
[ 5] contar_tokens(NULL)            OK
[ 6] liberar_lista(NULL)            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

### Lista de verificación

- [x] Los seis casos del harness fueron superados sin errores.
- [x] contar_tokens(NULL) devuelve 0.
- [x] liberar_lista(NULL) no provoca fallos.
- [x] El último nodo apunta a NULL.
- [x] La complejidad teórica coincide con la implementación.
- [ ] Verificación completa de fugas de memoria y accesos inválidos mediante herramientas especializadas.

### Reporte Crítico

**¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?**

Al revisar el código generado por la inteligencia artificial, identifiqué que no informa cuando un lexema supera la capacidad del arreglo de 16 caracteres.

La implementación utiliza strncpy para copiar como máximo 15 caracteres y agrega el carácter terminador '\0'. Esto evita escribir fuera del arreglo, pero puede ocasionar que un token largo se almacene incompleto.

El harness no evalúa este caso, por lo que sería necesario agregar una prueba adicional.

Como mejora, se propone validar la longitud del lexema antes de reservar memoria y rechazar los tokens que excedan la capacidad permitida.

También se comprobó que el programa maneja correctamente listas vacías y libera los nodos conservando temporalmente el puntero al siguiente elemento.

---

## 5. Conclusión

En esta práctica aprendí cómo funcionan las listas enlazadas y la importancia de utilizar correctamente la memoria dinámica en lenguaje C.

Comprendí que cada token necesita un nodo independiente y que los punteros permiten conectar los elementos para formar una lista.

También aprendí a utilizar la inteligencia artificial como herramienta de apoyo y a verificar el funcionamiento del código mediante pruebas automatizadas.

Los resultados fueron satisfactorios, ya que tanto la implementación manual como la generada con IA superaron las seis pruebas del harness.

Finalmente, comprendí que es importante revisar los casos límite y el manejo de memoria para desarrollar programas más seguros y confiables.

---

**Firma digital:** @Paola-23-ux

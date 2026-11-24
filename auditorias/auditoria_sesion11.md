
# Bitácora de Requerimientos y Auditoría IA — Sesión 11

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Arquitectura de la Tabla de Símbolos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350  
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Tabla Hash con encadenamiento.

Durante esta fase tuve dificultades para comprender cómo calcular la posición de un símbolo dentro de una tabla hash y cómo resolver las colisiones cuando dos nombres llegan a la misma cubeta.

También tuve dificultades con el manejo de punteros, la asignación dinámica de memoria y la utilización de listas enlazadas.

Con apoyo de la inteligencia artificial elaboré una primera versión que calculaba el hash mediante un ciclo while. Posteriormente, completé la función de inserción para verificar si un nombre ya existía y evitar que se almacenara dos veces.

Utilicé malloc() para reservar memoria, strcmp() para comparar los nombres y strcpy() para copiar el nombre del símbolo.

Finalmente, incorporé una validación para evitar almacenar nombres que excedieran la capacidad del arreglo.

### Código del Artesano

```c
#include "api_sesion11.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

unsigned int hash_djb2(const char *clave) {

    unsigned int hash = 5381;
    int i = 0;

    while (clave[i] != '\0') {
        hash = hash * 33 + (unsigned char)clave[i];
        i++;
    }

    return hash;
}

int insertar_simbolo(Simbolo *tabla[],
                     unsigned int cubetas,
                     const char *nombre,
                     int tipo,
                     int offset) {

    if (tabla == NULL || nombre == NULL || cubetas == 0) {
        return -1;
    }

    unsigned int posicion;
    posicion = hash_djb2(nombre) % cubetas;

    Simbolo *actual = tabla[posicion];

    while (actual != NULL) {

        if (strcmp(actual->nombre, nombre) == 0) {
            return 0;
        }

        actual = actual->sig;
    }

    if (strlen(nombre) >= sizeof(actual->nombre)) {
        return -1;
    }

    Simbolo *nuevo = malloc(sizeof(Simbolo));

    if (nuevo == NULL) {
        return -1;
    }

    strcpy(nuevo->nombre, nombre);

    nuevo->tipo = tipo;
    nuevo->offset = offset;

    nuevo->sig = tabla[posicion];
    tabla[posicion] = nuevo;

    return 1;
}
```

La implementación final del Artesano logró superar las seis pruebas proporcionadas por el docente.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| Contexto (C) | Lenguaje C, estándar C11, compilador GCC y Google Colab, sin bibliotecas no estándar. |
| Rol (R) | Ingeniero de software de sistemas especializado en estructuras de datos y compiladores. |
| Especificación (E) | Implementar una función de dispersión DJB2 y una tabla hash con encadenamiento, evitando símbolos duplicados y buscando un rendimiento promedio O(1) respecto al número de símbolos, bajo condiciones adecuadas. |
| Output (O) | Código modular compatible con la API, utilizando ciclos for, punteros y calloc(), acompañado de un análisis de complejidad. |

### 2.2 Prompt Estructurado

**Contexto (C):**

Estoy realizando la sesión 11 de Lenguajes y Autómatas II sobre la arquitectura de la tabla de símbolos en lenguaje C.

**Rol (R):**

Actúa como un ingeniero de software especializado en estructuras de datos y compiladores.

**Especificación (E):**

Implementa las funciones hash_djb2() e insertar_simbolo().

Utiliza una tabla hash con encadenamiento, evita nombres duplicados, conserva el tipo y offset de cada símbolo y valida los argumentos incorrectos.

**Output (O):**

Entrega código compatible con C11, utilizando ciclos for, punteros y calloc().

Respeta las firmas de api_sesion11.h y no incluyas main().

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion11.h"
#include <stdlib.h>
#include <string.h>

unsigned int hash_djb2(const char *clave) {
    unsigned int resultado = 5381;

    if (clave == NULL) {
        return resultado;
    }

    for (const unsigned char *p =
         (const unsigned char *)clave; *p; ++p) {

        resultado = (resultado << 5) + resultado + *p;
    }

    return resultado;
}

int insertar_simbolo(Simbolo *tabla[],
                     unsigned int cubetas,
                     const char *nombre,
                     int tipo,
                     int offset) {

    if (!tabla || !nombre || cubetas == 0) {
        return -1;
    }

    unsigned int indice = hash_djb2(nombre) % cubetas;

    Simbolo **enlace = &tabla[indice];

    for (; *enlace != NULL; enlace = &(*enlace)->sig) {

        if (strcmp((*enlace)->nombre, nombre) == 0) {
            return 0;
        }
    }

    size_t longitud = strlen(nombre);

    if (longitud >= sizeof(((Simbolo *)0)->nombre)) {
        return -1;
    }

    Simbolo *nodo = calloc(1, sizeof(Simbolo));

    if (!nodo) {
        return -1;
    }

    memcpy(nodo->nombre, nombre, longitud + 1);

    nodo->tipo = tipo;
    nodo->offset = offset;
    nodo->sig = NULL;

    *enlace = nodo;

    return 1;
}
```

### Funcionamiento del código

La función hash_djb2() utiliza un valor inicial de 5381 y recorre los caracteres del nombre mediante un puntero.

El desplazamiento de bits hacia la izquierda en cinco posiciones, seguido de una suma, equivale a multiplicar el valor por 33.

La función insertar_simbolo() calcula el índice de la cubeta mediante el operador módulo.

Posteriormente, recorre la lista enlazada correspondiente para verificar si el nombre ya existe. Si encuentra un símbolo repetido, devuelve 0.

Cuando el símbolo no existe, reserva memoria mediante calloc(), almacena el nombre, tipo y offset, y agrega el nodo al final de la lista.

Si la inserción se realiza correctamente, devuelve 1. Si ocurre un error, devuelve -1.

### Análisis de complejidad

**Complejidad de la función hash: O(m).**

La función recorre todos los caracteres de la clave, donde m representa la longitud del nombre.

**Complejidad promedio de inserción: O(1).**

Considerando nombres de longitud limitada, una distribución adecuada de las claves y un factor de carga controlado, la inserción tiene un costo promedio constante respecto al número de símbolos.

**Complejidad en el peor caso: O(n).**

Si todos los nombres llegan a la misma cubeta, será necesario recorrer todos los nodos de la lista para verificar si existe un símbolo duplicado.

**Complejidad espacial: O(n + b).**

La tabla requiere memoria para almacenar n símbolos y b cubetas.

### Comparación entre el Artesano y el Científico

| Característica | Artesano | Científico |
|---|---|---|
| Ciclo utilizado | while | for |
| Cálculo del hash | Multiplicación por 33 | Desplazamiento de bits |
| Recorrido de caracteres | Índices | Punteros |
| Reserva de memoria | malloc() | calloc() |
| Copia del nombre | strcpy() | memcpy() |
| Manejo de enlaces | Puntero simple | Puntero doble |
| Inserción del nodo | Inicio de la lista | Final de la lista |
| Detección de duplicados | strcmp() | strcmp() |
| Manejo de colisiones | Encadenamiento | Encadenamiento |
| Complejidad promedio | O(1) | O(1) |
| Complejidad en el peor caso | O(n) | O(n) |
| Pruebas superadas | 6/6 | 6/6 |

Ambas implementaciones cumplen con las pruebas establecidas, aunque utilizan diferentes estructuras de control y técnicas para manejar la memoria y los punteros.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Manejo de claves duplicadas y colisiones, cálculo de la función hash, conservación del offset y validación de argumentos incorrectos.

### Resultado del Artesano

```text
=== AUDITORIA SESION 11 - Tabla hash con encadenamiento ===

[1] hash_djb2("")
    esperado: 5381
    obtenido: 5381
    OK

[2] hash_djb2("abc")
    esperado: 193485963
    obtenido: 193485963
    OK

[3] insertar x dos veces
    esperado: 0
    obtenido: 0
    OK

[4] dos claves, una cubeta
    esperado: 2
    obtenido: 2
    OK

[5] offset de beta en cadena
    esperado: 4
    obtenido: 4
    OK

[6] insertar en tabla NULL
    esperado: -1
    obtenido: -1
    OK

RESULTADO: 6/6 pruebas superadas

TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion11.md
```

### Resultado del Científico

```text
=== AUDITORIA SESION 11 - Tabla hash con encadenamiento ===

[1] hash_djb2("")
    esperado: 5381
    obtenido: 5381
    OK

[2] hash_djb2("abc")
    esperado: 193485963
    obtenido: 193485963
    OK

[3] insertar x dos veces
    esperado: 0
    obtenido: 0
    OK

[4] dos claves, una cubeta
    esperado: 2
    obtenido: 2
    OK

[5] offset de beta en cadena
    esperado: 4
    obtenido: 4
    OK

[6] insertar en tabla NULL
    esperado: -1
    obtenido: -1
    OK

RESULTADO: 6/6 pruebas superadas

TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion11.md
```

### Tabla de resultados

| Caso evaluado | Esperado | Artesano | Científico | Estado |
|---|---|---|---|---|
| Hash de cadena vacía | 5381 | 5381 | 5381 | OK |
| Hash de abc | 193485963 | 193485963 | 193485963 | OK |
| Insertar x dos veces | 0 | 0 | 0 | OK |
| Dos claves en una cubeta | 2 | 2 | 2 | OK |
| Offset de beta en cadena | 4 | 4 | 4 | OK |
| Insertar en tabla NULL | -1 | -1 | -1 | OK |

**Resultado del Artesano:** 6/6 pruebas superadas.

**Resultado del Científico:** 6/6 pruebas superadas.

**Resultado total:** 12/12 pruebas superadas entre ambas implementaciones.

**Porcentaje de aprobación:** 100 %.

### Verificación de criterios

- [x] Caso base y límites evaluados superados sin Crash.
- [ ] Ausencia de accesos fuera de límites y Memory Leaks verificada mediante herramientas especializadas.
- [x] La complejidad teórica coincide con la implementada según el análisis del código.
- [x] La función hash devuelve los valores esperados.
- [x] Se evita la inserción de nombres duplicados.
- [x] Se manejan correctamente las colisiones evaluadas.
- [x] Se conserva el offset del símbolo.
- [x] Se valida una tabla NULL.
- [x] El Artesano superó las seis pruebas.
- [x] El Científico superó las seis pruebas.

No se realizaron pruebas especializadas para detectar fugas de memoria o accesos fuera de límites.

### Reporte Crítico

**¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?**

Durante la revisión del código generado por la inteligencia artificial identifiqué que la función insertar_simbolo() valida que el nombre no sea NULL, pero no comprueba si el nombre es una cadena vacía.

Aunque la función hash_djb2() debe aceptar una cadena vacía y devolver 5381, esto no significa que una cadena vacía deba ser aceptada como identificador dentro de una tabla de símbolos.

El código del Científico podría reservar memoria e insertar un símbolo sin nombre, lo cual representaría un problema si la tabla solamente debe almacenar identificadores válidos de un lenguaje de programación.

Este caso no fue incluido en las seis pruebas originales del harness.

### Propuesta de solución

Una posible solución consiste en agregar una condición para verificar que el nombre contenga al menos un carácter antes de calcular su posición dentro de la tabla.

```c
if (!tabla || !nombre || cubetas == 0) {
    return -1;
}

if (nombre[0] == '\0') {
    return -1;
}
```

Con esta validación se podría rechazar la inserción de símbolos con nombres vacíos sin modificar el comportamiento de hash_djb2().

La modificación se presenta como una propuesta de mejora, siempre que el contrato de la API establezca que los nombres vacíos no son válidos.

No fue incorporada ni ejecutada durante las pruebas originales.

---

**Firma digital:** Paola-23-ux

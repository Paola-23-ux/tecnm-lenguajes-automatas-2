# Bitácora de Requerimientos y Auditoría IA — Sesión 13
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Manejo de Ámbitos (Scope Global/Local)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia.  
**Matrícula:** No. 24531350  
**Track:** [ C | Rust ]  

---

## 1. Fase 1: El Artesano (Dificultad Manual)

*Reto:* Pila de tablas de símbolos (Stack Scopes).

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> La principal dificultad fue comprender cómo realizar la búsqueda de una variable entre diferentes ámbitos, comenzando por el ámbito actual y continuando hacia los ámbitos padres. También fue necesario controlar la declaración de variables duplicadas en un mismo nivel, respetar la capacidad máxima de cada ámbito y validar los punteros nulos. Para resolverlo utilicé ciclos for y while, que permiten recorrer los nombres registrados y realizar una búsqueda jerárquica.

### Código del Artesano

```c
/* artesano_sesion13_c.c - Fase 1: El Artesano (sin IA)
 * Pila de ámbitos y búsqueda jerárquica.
 *
 * Implementa lo que declara api_sesion13.h.
 * El harness compila este archivo junto con el suyo:
 * no escribas main().
 */

#include "api_sesion13.h"
#include <stdio.h>
#include <string.h>

int declarar(Ambito *ambito, const char *nombre) {
    if (ambito == NULL || nombre == NULL ||
        nombre[0] == '\0') {
        return -1;
    }

    if (ambito->total < 0 ||
        ambito->total >= AMBITO_MAX) {
        return -1; /* ámbito lleno */
    }

    /* ¿Ya existe en ESTE ámbito?
       (no consulta al padre) */

    for (int i = 0; i < ambito->total; i++) {
        if (ambito->nombres[i] != NULL &&
            strcmp(ambito->nombres[i], nombre) == 0) {
            return 0; /* duplicado en el mismo nivel */
        }
    }

    ambito->nombres[ambito->total] = nombre;
    ambito->total++;

    return 1;
}

const char *buscar(const Ambito *ambito,
                   const char *nombre) {
    if (ambito == NULL || nombre == NULL ||
        nombre[0] == '\0') {
        return NULL;
    }

    const Ambito *actual = ambito;

    while (actual != NULL) {
        for (int i = 0; i < actual->total; i++) {
            if (actual->nombres[i] != NULL &&
                strcmp(actual->nombres[i], nombre) == 0) {
                return actual->nombre;
            }
        }

        actual = actual->padre; /* sube al padre */
    }

    return NULL;
}
```

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Lenguaje C con GCC, utilizando el estándar C11 y bibliotecas estándar. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores. |
| **Especificación (E)** | Implementar la declaración de variables, búsqueda jerárquica de identificadores y ocultamiento de variables (Shadowing). |
| **Output (O)** | Código modular y análisis de complejidad, sin función main y compatible con el harness de la sesión 13. |

### 2.2 Prompt Estructurado

```text
Actúa como un ingeniero de software especializado
en sistemas y compiladores.

Desarrolla una implementación en lenguaje C
utilizando el archivo api_sesion13.h.

Implementa la función declarar para registrar
identificadores dentro del ámbito actual.

La función debe evitar declaraciones duplicadas
en un mismo nivel y respetar la capacidad máxima
establecida por AMBITO_MAX.

Implementa la función buscar para localizar un
identificador desde el ámbito actual hasta los
ámbitos padres.

Respeta el comportamiento de Shadowing, dando
prioridad a las variables declaradas en el ámbito
más cercano.

Valida los punteros nulos y los nombres vacíos.

Utiliza bibliotecas estándar y entrega un código
modular que no incluya la función main.

El código debe ser compatible con el harness
de la sesión 13.

Incluye un análisis de complejidad temporal
y espacial de las funciones.
```

---

## 3. Código Devuelto por el Modelo

```c
/* ia_sesion13_c.c - Fase 2: El Cientifico */

#include "api_sesion13.h"
#include <string.h>

int declarar(Ambito *ambito, const char *nombre) {
    if (!ambito || !nombre || nombre[0] == '\0')
        return -1;

    for (int i = 0; i < ambito->total; i++) {
        if (strcmp(ambito->nombres[i], nombre) == 0)
            return 0;
    }

    if (ambito->total >= AMBITO_MAX)
        return -1;

    ambito->nombres[ambito->total++] = nombre;

    return 1;
}

const char *buscar(const Ambito *ambito,
                   const char *nombre) {
    if (!nombre || nombre[0] == '\0')
        return NULL;

    for (const Ambito *actual = ambito;
         actual != NULL;
         actual = actual->padre) {

        for (int i = 0; i < actual->total; i++) {
            if (strcmp(actual->nombres[i], nombre) == 0)
                return actual->nombre;
        }
    }

    return NULL;
}
```

### Análisis de complejidad

| Función | Complejidad temporal | Complejidad espacial auxiliar |
| :--- | :---: | :---: |
| declarar() | O(n × L) | O(1) |
| buscar() | O(d × n × L) | O(1) |

Donde:

- n representa la cantidad máxima de identificadores
  registrados por ámbito.
- d representa la cantidad de ámbitos recorridos.
- L representa la longitud máxima de los nombres
  comparados.

La función declarar realiza una búsqueda lineal
dentro del ámbito actual para verificar si existe
un identificador duplicado.

La función buscar recorre los identificadores
de cada ámbito hasta encontrar una coincidencia
o llegar a la raíz.

Ambas funciones utilizan una cantidad constante
de memoria auxiliar, ya que no realizan
asignaciones dinámicas ni llamadas recursivas.

Debido a que AMBITO_MAX limita la capacidad
a ocho identificadores, si también se considera
constante la longitud de los nombres, declarar
tiene complejidad O(1) y buscar tiene complejidad O(d).

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Búsqueda jerárquica de identificadores.

### Resultado del Artesano

Comando de compilación y ejecución:

```bash
gcc -std=c11 -Wall -Wextra -Werror harness_sesion13_c.c artesano_sesion13_c.c -o test_artesano && ./test_artesano
```

```text
=== AUDITORIA SESION 13 - Pila de ambitos y busqueda jerarquica ===

[1] declarar(&g, "x")
    esperado: 1
    obtenido: 1
    resultado: OK

[2] buscar en el mismo ambito
    esperado: global
    obtenido: global
    resultado: OK

[3] buscar subiendo al padre
    esperado: global
    obtenido: global
    resultado: OK

[4] shadowing en el hijo
    esperado: bloque
    obtenido: bloque
    resultado: OK

[5] tres niveles hasta la raiz
    esperado: raiz
    obtenido: raiz
    resultado: OK

[6] nombre no declarado
    esperado: (nulo)
    obtenido: (nulo)
    resultado: OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

### Resultado del Científico

Comando de compilación y ejecución:

```bash
gcc -std=c11 -Wall -Wextra -Werror harness_sesion13_c.c ia_sesion13_c.c -o test_ia && ./test_ia
```

```text
=== AUDITORIA SESION 13 - Pila de ambitos y busqueda jerarquica ===

[1] declarar(&g, "x")
    esperado: 1
    obtenido: 1
    resultado: OK

[2] buscar en el mismo ambito
    esperado: global
    obtenido: global
    resultado: OK

[3] buscar subiendo al padre
    esperado: global
    obtenido: global
    resultado: OK

[4] shadowing en el hijo
    esperado: bloque
    obtenido: bloque
    resultado: OK

[5] tres niveles hasta la raiz
    esperado: raiz
    obtenido: raiz
    resultado: OK

[6] nombre no declarado
    esperado: (nulo)
    obtenido: (nulo)
    resultado: OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

### Tabla de resultados

| Caso de prueba | Esperado | Artesano | Científico | Resultado |
| :--- | :---: | :---: | :---: | :---: |
| Declarar variable x | 1 | 1 | 1 | OK |
| Buscar en el mismo ámbito | global | global | global | OK |
| Buscar subiendo al padre | global | global | global | OK |
| Shadowing en el hijo | bloque | bloque | bloque | OK |
| Tres niveles hasta la raíz | raiz | raiz | raiz | OK |
| Nombre no declarado | NULL | NULL | NULL | OK |

### Comparación entre el Artesano y el Científico

| Característica | Artesano | Científico |
| :--- | :---: | :---: |
| Pruebas realizadas | 6 | 6 |
| Pruebas superadas | 6 | 6 |
| Pruebas fallidas | 0 | 0 |
| Errores en la compilación final | 0 | 0 |
| Porcentaje de aprobación | 100% | 100% |
| Recorrido de ámbitos | while | for |
| Validación de total negativo en declarar | Sí | No |
| Validación de elementos nulos | Sí | No |
| Memoria auxiliar | O(1) | O(1) |
| Comentarios explicativos | Sí | Limitados |

### Lista de verificación

* [x] Caso base y límites evaluados superados sin Crash.
* [ ] No existen accesos fuera de límites ni Memory Leaks.
* [x] La complejidad teórica coincide con la implementada.

**Observación:** Ambas implementaciones superaron
las seis pruebas del harness sin presentar fallos
durante su ejecución. Sin embargo, no se realizaron
pruebas especializadas para confirmar la ausencia
de accesos fuera de límites o fugas de memoria.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Al comparar el código del Artesano con el código del Científico, observé que ambos superaron todas las pruebas del harness, pero presentan diferencias en sus validaciones. El Artesano verifica que los elementos almacenados dentro del ámbito no sean nulos antes de utilizar strcmp, mientras que el Científico realiza la comparación directamente. Esto podría generar un error si se encuentra un elemento nulo.

> Otra diferencia es que el Científico no valida si la variable total contiene un valor negativo y revisa la capacidad máxima después de recorrer los identificadores. El Artesano realiza estas verificaciones antes del recorrido en la función declarar.

> También observé que el Artesano rechaza cualquier declaración cuando el ámbito está lleno, incluso si el identificador ya existe. El Científico, en cambio, comprueba primero si existe un duplicado.

> Durante la compilación inicial del Científico se presentó el error "No such file or directory", porque el archivo ia_sesion13_c.c todavía no estaba disponible. Para solucionarlo ejecuté la celda encargada de crear el archivo y posteriormente volví a ejecutar el harness, obteniendo 6/6 pruebas superadas.

> No fue necesario modificar el algoritmo generado por la IA para superar las pruebas. Sin embargo, como mejora futura, sería conveniente validar los elementos nulos y los límites del contador total antes de recorrer los ámbitos.

---

**Firma digital:** Paola-23-ux

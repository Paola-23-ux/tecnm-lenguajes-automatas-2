# Bitácora de Requerimientos y Auditoría IA — Sesión 04

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Anatomía de un Árbol Sintáctico (AST)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350  
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Estructura de nodo binario (ASTNode).

### Problema o restricción identificada

Durante la implementación del Árbol Sintáctico Abstracto, uno de los principales retos fue comprender cómo reservar memoria dinámicamente para crear nodos y cómo conectar los hijos izquierdo y derecho mediante punteros.

También fue necesario comprender el funcionamiento de la recursividad para evaluar las operaciones matemáticas almacenadas en el árbol.

Al ejecutar las pruebas, se presentó un error de compilación relacionado con la función `malloc()`, debido a que faltaba incluir la biblioteca `<stdlib.h>`.

Para solucionarlo, agregué dicha biblioteca al código y ejecuté nuevamente la compilación. Después de corregir el problema, las seis pruebas se completaron correctamente.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Lenguaje C compatible con GCC y `api_sesion04.h`, utilizando estructuras, memoria dinámica y recursividad. |
| **Rol (R)** | Ingeniero de software especializado en lenguaje C, estructuras de datos y compiladores. |
| **Especificación (E)** | Implementar `crear_numero`, `crear_binario` y `evaluar`, permitiendo construir un AST y evaluar operaciones de suma, resta y multiplicación. Si el nodo es NULL, retornar -1. |
| **Output (O)** | Código modular en C, sin función main(), compatible con el archivo de cabecera y con complejidad de evaluación O(n). |

### 2.2 Prompt Estructurado

Actúa como un ingeniero de software especializado en lenguaje C y compiladores.

Necesito implementar un Árbol Sintáctico Abstracto (AST) utilizando la estructura ASTNode definida en el archivo `api_sesion04.h`.

El programa debe implementar tres funciones:

1. `crear_numero(int v)`: Crear un nodo numérico y almacenar su valor.
2. `crear_binario(char op, ASTNode *izq, ASTNode *der)`: Crear un nodo de operación binaria y conectar sus hijos.
3. `evaluar(ASTNode *n)`: Evaluar recursivamente el árbol utilizando los operadores +, - y *.

Requisitos:
- Utilizar memoria dinámica.
- Verificar errores de reserva de memoria.
- Retornar -1 cuando el nodo sea NULL.
- Mantener las firmas originales.
- No incluir la función main().
- Generar código compatible con GCC.
- Explicar la complejidad de las funciones.

---

## 3. Código Devuelto por el Modelo

La IA proporcionó una implementación en lenguaje C que utiliza `calloc()` para reservar e inicializar memoria y recursividad para evaluar el árbol.

### Función crear_numero

~~~c
ASTNode *crear_numero(int v) {
    ASTNode *nodo = calloc(1, sizeof(ASTNode));

    if (nodo == NULL) {
        return NULL;
    }

    nodo->tipo = NODO_NUM;
    nodo->valor = v;

    return nodo;
}
~~~

### Función crear_binario

~~~c
ASTNode *crear_binario(char op, ASTNode *izq, ASTNode *der) {
    ASTNode *nodo = calloc(1, sizeof(ASTNode));

    if (nodo == NULL) {
        return NULL;
    }

    nodo->tipo = NODO_BINOP;
    nodo->op = op;
    nodo->izq = izq;
    nodo->der = der;

    return nodo;
}
~~~

### Función evaluar

~~~c
int evaluar(ASTNode *n) {
    if (n == NULL) {
        return -1;
    }

    if (n->tipo == NODO_NUM) {
        return n->valor;
    }

    if (n->tipo == NODO_BINOP) {
        int izquierdo = evaluar(n->izq);
        int derecho = evaluar(n->der);

        switch (n->op) {
            case '+':
                return izquierdo + derecho;
            case '-':
                return izquierdo - derecho;
            case '*':
                return izquierdo * derecho;
            default:
                return -1;
        }
    }

    return -1;
}
~~~

### Análisis de complejidad

- **crear_numero():** O(1), porque crea un nodo.
- **crear_binario():** O(1), porque crea un nodo y enlaza sus hijos.
- **evaluar():** O(n), porque recorre recursivamente los nodos del árbol.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Árbol vacío, evaluación de nodos numéricos, operaciones binarias y verificación de los hijos.

### Resultados obtenidos

~~~text
[ 1] evaluar(nodo 42)                  esperado: 42   obtenido: 42   OK
[ 2] evaluar(3+5*2)                    esperado: 13   obtenido: 13   OK
[ 3] hijo izquierdo de (* 5 2)        esperado: 5    obtenido: 5    OK
[ 4] operador guardado                esperado: *    obtenido: *    OK
[ 5] evaluar(NULL)                     esperado: -1   obtenido: -1   OK
[ 6] evaluar(10-4)                     esperado: 6    obtenido: 6    OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
~~~

### Comparación de resultados

| Implementación | Pruebas superadas | Estado |
| :--- | :--- | :--- |
| El Artesano | 6/6 | Correcto |
| El Científico | 6/6 | Correcto |

### Lista de verificación

- [x] Caso base NULL y operaciones del harness superados sin errores.
- [ ] Ausencia de accesos fuera de límites y Memory Leaks: no verificada.
- [x] Complejidad teórica analizada y consistente con la implementación.

### Reporte Crítico

**¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?**

Durante la revisión del código generado por la IA, identifiqué que la función `evaluar()` comprueba si el nodo principal es NULL, pero no distingue los errores de evaluación de los resultados matemáticos válidos cuando procesa los nodos hijos.

Por ejemplo, si uno de los hijos es NULL, la función devuelve -1, pero ese valor puede utilizarse posteriormente en una operación matemática como si fuera un número válido.

Este caso no fue detectado por las seis pruebas del harness, ya que solamente se evaluó directamente un nodo NULL y operaciones con hijos válidos.

Como solución propuesta, se puede implementar un mecanismo que permita distinguir los errores de evaluación de los resultados válidos, respetando el contrato de las funciones.

También identifiqué un error de compilación en la implementación manual, debido a que faltaba incluir la biblioteca `<stdlib.h>`, necesaria para utilizar `malloc()`.

Después de agregar la biblioteca y ejecutar nuevamente el programa, se obtuvieron las seis pruebas superadas.

---

**Firma digital:** Paola-23-ux  

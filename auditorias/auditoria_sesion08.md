# Bitácora de Requerimientos y Auditoría IA — Sesión 08

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Inferencia y Promoción de Tipos  
**Docente Titular:** MCC. Iván Márquez Larios (ijmarquezl)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350  
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Reglas de casteo implícito.

Durante esta fase tuve dificultades para implementar las funciones de promoción de tipos y pérdida de información, ya que al principio no comprendía cómo comparar los rangos de CHAR, INT y FLOAT ni cómo manejar el tipo ERROR.

Intenté realizar el código por mi cuenta, pero necesité ayuda de la inteligencia artificial para completar la implementación. Utilicé estructuras if y else para comparar los tipos de datos y devolver el resultado correspondiente.

Comprendí que cuando se combinan dos tipos diferentes se debe devolver el de mayor rango, excepto cuando alguno de los operandos es ERROR.

### Código del Artesano

```c
#include "api_sesion08.h"

Tipo promover(Tipo a, Tipo b) {

    if (a == TIPO_ERROR || b == TIPO_ERROR) {
        return TIPO_ERROR;
    }

    if (a > b) {
        return a;
    } else {
        return b;
    }
}

int hay_perdida(Tipo destino, Tipo origen) {

    if (destino < origen) {
        return 1;
    } else {
        return 0;
    }
}

const char *nombre_tipo(Tipo t) {

    if (t == TIPO_CHAR) {
        return "char";
    } else if (t == TIPO_INT) {
        return "int";
    } else if (t == TIPO_FLOAT) {
        return "float";
    } else {
        return "error";
    }
}
```

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| Contexto (C) | Lenguaje C, Google Colab, compilador GCC y estándar C11. |
| Rol (R) | Programador especializado en lenguaje C y compiladores. |
| Especificación (E) | Implementar las funciones de promoción de tipos, pérdida de información y nombre del tipo, respetando la jerarquía CHAR < INT < FLOAT. |
| Output (O) | Código en lenguaje C utilizando operadores ternarios y switch, respetando las firmas de la API y sin incluir main(). |

### 2.2 Prompt Estructurado

**Contexto:**

Estoy realizando la sesión 08 de Lenguajes y Autómatas II sobre inferencia y promoción de tipos de datos en lenguaje C.

**Rol:**

Actúa como un programador especializado en lenguaje C y compiladores.

**Especificación:**

Implementa las funciones promover, hay_perdida y nombre_tipo, respetando la jerarquía CHAR < INT < FLOAT.

Si alguno de los operandos es ERROR, promover debe devolver ERROR.

Respeta las firmas declaradas en api_sesion08.h.

**Output:**

Entrega código en lenguaje C utilizando el operador ternario y switch. No incluyas la función main(). El código debe ser compatible con el harness proporcionado.

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion08.h"

Tipo promover(Tipo a, Tipo b) {
    return (a == TIPO_ERROR || b == TIPO_ERROR)
           ? TIPO_ERROR : (a > b ? a : b);
}

int hay_perdida(Tipo destino, Tipo origen) {
    return destino < origen;
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

### Análisis de complejidad

La complejidad temporal de las tres funciones es O(1), debido a que realizan una cantidad constante de operaciones y no utilizan ciclos.

La complejidad espacial también es O(1), porque no requieren arreglos ni asignación dinámica de memoria.

### Comparación entre el Artesano y el Científico

| Característica | Artesano | Científico |
|---|---|---|
| Estructura de control | if y else | Operador ternario y switch |
| Complejidad temporal | O(1) | O(1) |
| Complejidad espacial | O(1) | O(1) |
| Extensión del código | Más extenso | Más compacto |
| Pruebas superadas | 6/6 | 6/6 |

Ambas implementaciones cumplen con los resultados esperados del harness. La principal diferencia se encuentra en las estructuras utilizadas para resolver el problema.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Reglas de promoción de tipos y pérdida de información.

### 4.1 Resultado del Artesano

```text
=== AUDITORIA SESION 08 - Reglas de promocion y perdida ===

[ 1] promover(CHAR,INT)       esperado: int    obtenido: int    OK
[ 2] promover(INT,FLOAT)      esperado: float  obtenido: float  OK
[ 3] promover(FLOAT,FLOAT)    esperado: float  obtenido: float  OK
[ 4] promover(ERROR,INT)      esperado: error  obtenido: error  OK
[ 5] hay_perdida(CHAR,INT)    esperado: 1      obtenido: 1      OK
[ 6] hay_perdida(INT,CHAR)    esperado: 0      obtenido: 0      OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion08.md
```

### 4.2 Resultado del Científico

```text
=== AUDITORIA SESION 08 - Reglas de promocion y perdida ===

[ 1] promover(CHAR,INT)       esperado: int    obtenido: int    OK
[ 2] promover(INT,FLOAT)      esperado: float  obtenido: float  OK
[ 3] promover(FLOAT,FLOAT)    esperado: float  obtenido: float  OK
[ 4] promover(ERROR,INT)      esperado: error  obtenido: error  OK
[ 5] hay_perdida(CHAR,INT)    esperado: 1      obtenido: 1      OK
[ 6] hay_perdida(INT,CHAR)    esperado: 0      obtenido: 0      OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion08.md
```

### 4.3 Tabla de resultados

| Caso evaluado | Esperado | Artesano | Científico | Estado |
|---|---|---|---|---|
| promover(CHAR,INT) | int | int | int | OK |
| promover(INT,FLOAT) | float | float | float | OK |
| promover(FLOAT,FLOAT) | float | float | float | OK |
| promover(ERROR,INT) | error | error | error | OK |
| hay_perdida(CHAR,INT) | 1 | 1 | 1 | OK |
| hay_perdida(INT,CHAR) | 0 | 0 | 0 | OK |

**Resultado del Artesano:** 6/6 pruebas superadas.

**Resultado del Científico:** 6/6 pruebas superadas.

**Resultado total:** 12/12 pruebas superadas entre ambas ejecuciones.

### 4.4 Verificación de criterios

- [x] El Artesano superó las seis pruebas.
- [x] El Científico superó las seis pruebas.
- [x] Se respetaron las firmas de la API.
- [x] No se presentaron errores de compilación.
- [x] Se respetó la jerarquía de promoción en los casos evaluados.
- [x] Se conservó el centinela ERROR en el caso evaluado.
- [x] Se detectó correctamente la pérdida de información en los casos evaluados.
- [x] Ambas implementaciones finalizaron correctamente.

---

## 5. Reporte Crítico

### ¿Qué caso borde omitió la IA?

Al revisar el código generado por la inteligencia artificial, identifiqué un caso borde relacionado con la función hay_perdida cuando alguno de los tipos es TIPO_ERROR.

Por ejemplo:

```c
hay_perdida(TIPO_ERROR, TIPO_FLOAT);
```

De acuerdo con la comparación de rangos, esta función devolvería 1, debido a que TIPO_ERROR tiene un valor de 0 y TIPO_FLOAT tiene un valor de 3.

Sin embargo, ERROR representa un estado inválido y no un tipo de dato válido dentro de la jerarquía CHAR, INT y FLOAT.

### ¿Cómo lo detecté?

Este caso se identificó mediante la revisión de la lógica del código y la comparación de los valores establecidos en la API.

El harness original no incluye una prueba específica para hay_perdida cuando interviene TIPO_ERROR.

### ¿Cómo se podría solucionar?

Primero sería necesario establecer en el contrato qué resultado debe devolver la función cuando alguno de los tipos es ERROR. Después se podría agregar una validación y una prueba adicional al harness.

No modifiqué el código porque el contrato original no especifica el comportamiento de esta función para dicho caso.

Aunque ambas implementaciones superaron las seis pruebas, esto no significa que se hayan evaluado todas las combinaciones posibles.

---

**Firma digital:** Paola-23-ux

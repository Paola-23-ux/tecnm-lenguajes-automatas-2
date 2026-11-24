
# Bitácora de Requerimientos y Auditoría IA — Sesión 09

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Reporte Formal de Errores Semánticos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350  
**Track:** C  

---

## 1. Fase 1: El Artesano (Dificultad Manual)

**Reto:** Diagnóstico con línea y columna.

Durante esta fase tuve dificultades para construir el mensaje completo de diagnóstico utilizando la función snprintf().

Con ayuda de la IA elaboré una primera versión que mostraba únicamente el nombre del archivo, la línea y la columna, pero faltaba incorporar la palabra error y el contenido de la variable mensaje.

Después comprendí cómo utilizar los especificadores %s y %d para incorporar cadenas de texto y números enteros en el diagnóstico.

También aprendí que es importante respetar la capacidad del buffer para evitar escribir fuera de los límites de memoria.

### Código inicial del Artesano

```c
#include "api_sesion09.h"
#include <stdio.h>

void reportar(const char *archivo, int linea, int columna,
              const char *mensaje, char *salida, int cap) {

    if (salida != NULL && cap > 0) {

        snprintf(salida, cap, "%s:%d:%d",
                 archivo, linea, columna);

        // Pendiente: agregar el mensaje de error.
    }

    (void)mensaje;
}
```

### Corrección realizada

Se modificó el formato de snprintf() para incluir la descripción del error.

```c
snprintf(salida, cap, "%s:%d:%d: error: %s",
         archivo, linea, columna, mensaje);
```

También se eliminó la instrucción (void)mensaje porque después de la corrección el parámetro ya se utilizaba.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| Contexto (C) | Lenguaje C, estándar C11, compilador GCC y Google Colab, sin bibliotecas no estándar. |
| Rol (R) | Ingeniero de software especializado en sistemas y compiladores. |
| Especificación (E) | Implementar reportar() utilizando el formato estándar de errores GCC, incluyendo archivo, línea, columna y mensaje. Respetar la capacidad del buffer. |
| Output (O) | Código modular compatible con la API, sin función main(), acompañado de un análisis de complejidad. |

### 2.2 Prompt Estructurado

**Contexto:**

Estoy realizando la sesión 09 de Lenguajes y Autómatas II sobre el reporte formal de errores semánticos y diagnóstico en C.

**Rol:**

Actúa como un programador especializado en lenguaje C y compiladores.

**Especificación:**

Implementa la función reportar() respetando la firma declarada en api_sesion09.h.

Genera mensajes con el nombre del archivo, número de línea, número de columna y descripción del error.

Evita desbordar el buffer y garantiza la terminación nula cuando la capacidad sea positiva y el buffer sea válido.

**Output:**

Entrega código compatible con C11 utilizando snprintf() y validaciones.

No incluyas la función main(). El código debe superar las cuatro pruebas del harness.

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion09.h"
#include <stdio.h>

void reportar(const char *archivo, int linea, int columna,
              const char *mensaje, char *salida, int cap) {

    if (salida == NULL || cap <= 0) {
        return;
    }

    int resultado = snprintf(
        salida,
        (size_t)cap,
        "%s:%d:%d: error: %s",
        archivo,
        linea,
        columna,
        mensaje
    );

    if (resultado < 0) {
        salida[0] = '\0';
    }
}
```

### Análisis de complejidad

**Complejidad temporal: O(n).**

La función snprintf() procesa las cadenas correspondientes al nombre del archivo y al mensaje de error. El tiempo de ejecución depende de la longitud de estas cadenas.

**Complejidad espacial auxiliar: O(1).**

La función utiliza una variable local para almacenar el resultado de snprintf() y escribe directamente en el buffer proporcionado, sin realizar asignaciones dinámicas de memoria en nuestra implementación.

### Comparación entre el Artesano y el Científico

| Característica | Artesano | Científico |
|---|---|---|
| Validación del buffer | Condición positiva | Retorno anticipado |
| Uso de snprintf() | Directo | Almacena el resultado |
| Manejo de errores de formato | No incluido | Comprueba el valor retornado |
| Conversión de capacidad | Implícita | Explícita mediante size_t |
| Complejidad temporal | O(n) | O(n) |
| Complejidad espacial auxiliar | O(1) | O(1) |
| Pruebas superadas | 4/4 | 4/4 |

Ambas implementaciones generan los mismos resultados en los casos evaluados. Sin embargo, el código del Científico incorpora una validación adicional para detectar posibles errores durante el formateo.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

**Pruebas ejecutadas:** Diagnósticos de errores semánticos con línea, columna y truncamiento del buffer.

### Resultado del Artesano

```text
=== AUDITORIA SESION 09 - Diagnostico con linea y columna ===

[ 1] main.c:14:5
     esperado: main.c:14:5: error: tipo incompatible
     obtenido: main.c:14:5: error: tipo incompatible
     -> OK

[ 2] a.c:1:1
     esperado: a.c:1:1: error: x
     obtenido: a.c:1:1: error: x
     -> OK

[ 3] linea 0
     esperado: x.c:0:0: error: y
     obtenido: x.c:0:0: error: y
     -> OK

[ 4] cap=10 trunca
     esperado: main.c:14
     obtenido: main.c:14
     -> OK

RESULTADO: 4/4 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion09.md
```

### Resultado del Científico

```text
=== AUDITORIA SESION 09 - Diagnostico con linea y columna ===

[ 1] main.c:14:5
     esperado: main.c:14:5: error: tipo incompatible
     obtenido: main.c:14:5: error: tipo incompatible
     -> OK

[ 2] a.c:1:1
     esperado: a.c:1:1: error: x
     obtenido: a.c:1:1: error: x
     -> OK

[ 3] linea 0
     esperado: x.c:0:0: error: y
     obtenido: x.c:0:0: error: y
     -> OK

[ 4] cap=10 trunca
     esperado: main.c:14
     obtenido: main.c:14
     -> OK

RESULTADO: 4/4 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion09.md
```

### Tabla de resultados

| Caso evaluado | Artesano | Científico |
|---|---|---|
| Diagnóstico main.c:14:5 | OK | OK |
| Diagnóstico a.c:1:1 | OK | OK |
| Línea y columna en cero | OK | OK |
| Truncamiento con capacidad 10 | OK | OK |

**Resultado del Artesano:** 4/4 pruebas superadas.

**Resultado del Científico:** 4/4 pruebas superadas.

**Resultado total:** 8/8 pruebas superadas entre ambas ejecuciones.

### Verificación de criterios

- [x] Caso base y límites evaluados superados sin Crash.
- [ ] Ausencia de accesos fuera de límites y Memory Leaks comprobada mediante herramientas especializadas.
- [x] La complejidad teórica coincide con la implementada.
- [x] El Artesano superó las cuatro pruebas.
- [x] El Científico superó las cuatro pruebas.
- [x] Se respetó la capacidad del buffer en el caso de truncamiento evaluado.
- [x] No se presentaron errores de compilación.

La implementación no utiliza memoria dinámica directamente. Sin embargo, no se ejecutaron herramientas especializadas para verificar todos los posibles problemas de memoria.

### Reporte Crítico

**¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?**

Al analizar el código proporcionado por la inteligencia artificial se identificó un posible caso borde relacionado con los parámetros archivo y mensaje cuando contienen un puntero NULL.

Aunque la función verifica que el buffer de salida sea válido y que su capacidad sea mayor que cero, no comprueba si archivo o mensaje son punteros nulos antes de utilizarlos con el especificador %s.

Esta situación podría provocar un comportamiento indefinido, dependiendo de la implementación de la biblioteca de C.

El problema se identificó mediante la revisión del código, ya que el harness original no incluye pruebas específicas para estos parámetros.

Como propuesta de solución se podrían validar los parámetros antes de llamar a snprintf().

```c
if (archivo == NULL || mensaje == NULL) {
    salida[0] = '\0';
    return;
}
```

Esta validación tendría que colocarse después de comprobar que salida sea válido y que cap sea positivo.

La corrección se presenta como una propuesta de mejora, ya que no fue incorporada ni evaluada en las ejecuciones realizadas. Ambas implementaciones superaron las cuatro pruebas originales.

---

**Firma digital:** Paola-23-ux

# Bitácora de Requerimientos y Auditoría IA — Sesión 01

**Materia:** Lenguajes y Autómatas II (SCD-1016)

**Tema:** Encuadre, Diagnóstico de Punteros y Metodología IA

**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)

**Estudiante:** Ingrid Paola Dzul Segovia

**Matrícula:** 24531350

**Track:** C

---

## 1. Fase 1: El Artesano (Implementación Manual)

### Objetivo

Desarrollar una función en lenguaje C que permita invertir una cadena de caracteres directamente en su espacio de memoria, sin utilizar arreglos adicionales ni reservar memoria dinámica.

### Problema o restricción identificada

El principal reto fue comprender cómo invertir los caracteres de una cadena utilizando dos índices, uno al inicio y otro al final.

También fue necesario evitar accesos incorrectos a la memoria y comprobar que la función pudiera trabajar con cadenas vacías y punteros nulos.

Para resolver el problema, se utilizó una variable temporal que permite intercambiar los caracteres de ambos extremos hasta llegar al centro de la cadena.

### Descripción del procedimiento

1. Comprobar que el puntero recibido no sea NULL.
2. Identificar el inicio y el final de la cadena.
3. Intercambiar los caracteres utilizando una variable temporal.
4. Avanzar el índice inicial y retroceder el índice final.
5. Repetir el procedimiento hasta completar la inversión.

### Evidencia de ejecución

- Cadena original: `HOLA`
- Cadena invertida: `ALOH`
- Resultado: Correcto.

**Resultado de la implementación manual:** 6 de 6 pruebas superadas.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición técnica |
|---|---|
| Contexto (C) | Desarrollo de una función en lenguaje C compatible con GCC que invierta una cadena modificable en su misma memoria. |
| Rol (R) | Ingeniero de software especializado en programación C, punteros y manejo de memoria. |
| Especificación (E) | Implementar `void invertir(char *s)`, utilizar únicamente dos índices y una variable temporal, manejar NULL y cadenas vacías, sin malloc ni arreglos auxiliares. |
| Output (O) | Código fuente en C compatible con el archivo `api_sesion01.h`, sin función main(), acompañado de una explicación de funcionamiento y complejidad. |

### 2.2 Prompt estructurado

Actúa como un ingeniero de software especializado en lenguaje C y manejo de punteros.

Necesito implementar una función llamada `invertir(char *s)` que invierta una cadena de caracteres directamente en su mismo espacio de memoria.

La función debe cumplir las siguientes condiciones:

- Utilizar dos índices y una variable temporal.
- No utilizar malloc ni arreglos auxiliares.
- Modificar directamente la cadena original.
- Manejar correctamente punteros NULL y cadenas vacías.
- Tener complejidad O(n) en tiempo y O(1) en memoria auxiliar.
- Respetar la firma declarada en `api_sesion01.h`.
- No incluir la función main().

Devuelve el código en lenguaje C compatible con GCC y explica brevemente su funcionamiento.

---

## 3. Código Devuelto por el Modelo

La inteligencia artificial proporcionó una implementación que cumple con la firma requerida y permite invertir los caracteres sin crear una copia de la cadena.

**Archivo:** `ia_sesion01_c.c`

```c
#include "api_sesion01.h"
#include <string.h>

void invertir(char *s) {
    if (s == NULL) {
        return;
    }

    int inicio = 0;
    int fin = (int)strlen(s) - 1;

    while (inicio < fin) {
        char temporal = s[inicio];
        s[inicio] = s[fin];
        s[fin] = temporal;

        inicio++;
        fin--;
    }
}
```

### Explicación del funcionamiento

Primero se verifica que el puntero no sea NULL. Después se calcula la longitud de la cadena y se establecen dos índices.

Mediante un ciclo while se intercambian los caracteres de los extremos y se desplazan los índices hacia el centro hasta completar la inversión.

### Análisis de complejidad

**Tiempo O(n):** La función calcula la longitud de la cadena y realiza los intercambios necesarios para invertirla.

**Memoria O(1):** Se utilizan dos índices y una variable temporal, sin crear arreglos adicionales ni reservar memoria dinámica.

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

### Objetivo

Comprobar que las implementaciones manual y generada con inteligencia artificial cumplan con los requisitos establecidos mediante pruebas automáticas.

### Pruebas ejecutadas

| N.º | Entrada | Resultado esperado | Resultado obtenido | Estado |
|---|---|---|---|---|
| 1 | HOLA | ALOH | ALOH | OK |
| 2 | abc | cba | cba | OK |
| 3 | Cadena vacía | Cadena vacía | Cadena vacía | OK |
| 4 | AB | BA | BA | OK |
| 5 | a | a | a | OK |
| 6 | NULL | Sin error | Sin error | OK |

### Resultados de la implementación manual

```text
=== AUDITORIA SESION 01 - Inversion de buffer in-place ===

[ 1] invertir("HOLA")  esperado: ALOH       obtenido: ALOH       OK
[ 2] invertir("abc")   esperado: cba        obtenido: cba        OK
[ 3] invertir("")      esperado:           obtenido:           OK
[ 4] invertir("AB")    esperado: BA         obtenido: BA         OK
[ 5] invertir("a")     esperado: a          obtenido: a          OK
[ 6] invertir(NULL)    esperado: sin crash  obtenido: sin crash  OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

### Resultados de la implementación con IA

Se ejecutó el mismo conjunto de pruebas utilizando el archivo `ia_sesion01_c.c`.

**Resultado:** 6/6 pruebas superadas.

Todos los casos mostraron el estado OK y no se presentaron errores de compilación ni fallos visibles durante la ejecución.

### Comprobaciones de aceptación

- [x] La cadena original se modifica directamente.
- [x] No se utiliza malloc ni un arreglo auxiliar.
- [x] Los seis casos del harness mostraron OK.
- [x] La función maneja NULL sin provocar un fallo de ejecución.
- [x] La cadena vacía se procesa correctamente.
- [x] Ambas implementaciones se compilaron con `-Wall -Wextra -Werror`.

---

## 5. Reporte Crítico

Durante la auditoría se comprobó que el código generado por inteligencia artificial cumplía con los requisitos establecidos para invertir cadenas de caracteres.

Uno de los casos borde más importantes fue `invertir(NULL)`, ya que intentar acceder a un puntero nulo podría provocar un error en el programa. Sin embargo, la IA incluyó una condición para evitar este problema.

También se evaluaron las cadenas vacías y las cadenas de un solo carácter, comprobando que ambas se procesaran correctamente.

La IA no omitió ninguno de los casos borde evaluados. Esto se comprobó mediante el test harness, que ejecutó seis pruebas automáticas y mostró el resultado OK en todas ellas.

Aunque todas las pruebas fueron correctas, comprendí que es importante revisar el código generado por inteligencia artificial, ya que pueden existir errores que no se detecten con las pruebas realizadas.

---

## 6. Relación con el Entorno Profesional

Esta práctica se relaciona con el desarrollo profesional de software porque permite comprender la importancia de trabajar correctamente con memoria, punteros y estructuras de datos.

En un entorno laboral, los programadores deben asegurarse de que sus aplicaciones funcionen correctamente y no presenten errores que puedan afectar el funcionamiento de otros sistemas.

También es importante utilizar herramientas de prueba automática para detectar problemas antes de entregar un programa.

El uso de inteligencia artificial puede facilitar algunas tareas de programación, pero es responsabilidad del desarrollador comprobar que el código cumpla con los requisitos establecidos.

---

## 7. Conclusión

En esta práctica aprendí a invertir cadenas de caracteres en lenguaje C utilizando índices y una variable temporal, sin necesidad de crear otro arreglo.

También comprendí la importancia de definir correctamente los requisitos antes de utilizar inteligencia artificial y de comprobar que el código generado funcione adecuadamente.

Durante la ejecución de las pruebas automáticas, tanto la implementación manual como la generada con IA obtuvieron seis resultados correctos de seis posibles.

Esta actividad me permitió reforzar mis conocimientos sobre punteros, manejo de memoria y pruebas de software. Además, comprendí que la inteligencia artificial puede ser una herramienta de apoyo, pero siempre es necesario revisar sus resultados para garantizar un funcionamiento correcto.

---

**Firma digital:** `Paola-23-ux`

**Resultado final de la práctica:** 6/6 pruebas superadas en ambas implementaciones.

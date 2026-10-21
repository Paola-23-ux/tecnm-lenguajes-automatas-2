
# Bitácora de Requerimientos y Auditoría IA — Sesión 01

**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Encuadre, Diagnóstico de Punteros y Metodología IA  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350  
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)

*Reto: Inversión de buffer in-place (char\*)*

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

El principal reto fue comprender cómo utilizar dos punteros para recorrer una cadena de caracteres desde sus extremos e intercambiar sus valores sin utilizar otro arreglo.

También fue necesario considerar el carácter terminador `\0`, ya que este no debe intercambiarse con las letras. Para resolverlo, se utilizó una variable temporal y se movieron los punteros hacia el centro hasta que se encontraron o cruzaron.

La ejecución permitió comprobar que la palabra `HOLA` se invirtió correctamente a `ALOH`.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|---|---|
| **Contexto (C)** | C99 / GCC, sin bibliotecas no estándar. |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores. |
| **Especificación (E)** | Punteros `char*`, O(n) tiempo, O(1) memoria auxiliar, manejo de NULL y cadenas vacías. |
| **Output (O)** | Código modular en C y análisis de complejidad. |

### 2.2 Prompt Estructurado

```text
Actúa como un ingeniero de software especializado
en lenguaje C y manejo de punteros.

Necesito desarrollar una función que invierta
un buffer de caracteres directamente en memoria
(in-place), utilizando un puntero char*.

La solución debe cumplir estos requisitos:

- Complejidad O(n) en tiempo.
- Complejidad O(1) en memoria auxiliar.
- No utilizar arreglos adicionales.
- Manejar correctamente punteros NULL.
- Funcionar con cadenas vacías.
- Funcionar con longitudes pares e impares.

Devuelve el código completo en C compatible
con GCC y explica brevemente su complejidad.
```

---

## 3. Código Devuelto por el Modelo

```c
#include <stdio.h>
#include <string.h>

void invertir(char *texto) {
    if (texto == NULL) {
        return;
    }

    size_t longitud = strlen(texto);

    for (size_t i = 0; i < longitud / 2; i++) {
        char temporal = texto[i];
        texto[i] = texto[longitud - i - 1];
        texto[longitud - i - 1] = temporal;
    }
}

int main(void) {
    char palabra[] = "MUNDO";

    printf("Original: %s\n", palabra);

    invertir(palabra);

    printf("Invertida: %s\n", palabra);

    return 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas: Valores NULL, buffers vacíos y longitudes impares.*

- [x] Caso base y límites probados sin Crash.
- [ ] No existen accesos fuera de límites ni Memory Leaks. (Pendiente de comprobar con herramientas especializadas).
- [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

La solución generada por la inteligencia artificial contempla el manejo de valores NULL y cadenas vacías, por lo que no se identificaron errores en los cinco casos ejecutados.

Las pruebas realizadas fueron:

1. Cadena normal: `HOLA` → `ALOH`.
2. Longitud impar: `ABCDE` → `EDCBA`.
3. Cadena vacía: `""` → `""`.
4. Puntero `NULL`: retorno sin error.
5. Un carácter: `A` → `A`.

**Resultado obtenido:** TODAS LAS PRUEBAS PASARON.

Sin embargo, se identificó una limitación en la auditoría: las pruebas se realizaron sobre una copia de la función y no directamente sobre el archivo original generado por IA.

Para mejorar la validación, se recomienda ejecutar las pruebas contra la implementación original y utilizar herramientas de análisis de memoria.

También es importante verificar que el buffer recibido sea modificable y que contenga el terminador `\0`.

---

**Firma digital:** `Paola-23-ux`

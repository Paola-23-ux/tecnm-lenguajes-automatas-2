Bitácora de Requerimientos y Auditoría IA — Sesión 01
Materia: Lenguajes y Autómatas II (SCD-1016)  
Tema: Encuadre, Diagnóstico de Punteros y Metodología IA  
Docente Titular: MCC. Iván Márquez Larios (`ijmarquezl`)  
Estudiante: Ingrid Paola Dzul Segovia  
Matrícula: [24531350]  
Track: C
---
1. Fase 1: El Artesano (Dificultad Manual)
Reto: Inversión de buffer in-place (`char*`).
Problema o restricción identificada:  
El principal reto fue comprender cómo mover dos punteros dentro del mismo arreglo de caracteres sin utilizar otro buffer. También fue necesario evitar que el puntero del final apuntara al carácter terminador `\0` y detener los intercambios cuando ambos punteros se encontraran o cruzaran. Para ello se utilizó una variable temporal y se avanzó desde los extremos hacia el centro.
Evidencia de ejecución: La cadena `HOLA` se transformó en `ALOH`.
2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)
2.1 Matriz de Especificación Formal
Elemento C-R-E-O	Definición Técnica
Contexto (C)	Lenguaje C compatible con GCC; invertir un arreglo de caracteres modificable en su misma memoria.
Rol (R)	Ingeniero de software especializado en C, punteros y análisis de algoritmos.
Especificación (E)	Usar `char*`, tiempo O(n), memoria auxiliar O(1), manejar `NULL`, cadenas vacías y longitudes impares, sin asignar otro arreglo.
Output (O)	Código en C y análisis de complejidad.
2.2 Prompt Estructurado
> Actúa como ingeniero de software especializado en lenguaje C y manejo de punteros. Necesito una función que invierta un buffer modificable de caracteres directamente en memoria (*in-place*), usando `char*`. La solución debe tener complejidad O(n) en tiempo y O(1) en memoria auxiliar, sin crear un segundo arreglo. Debe manejar un puntero `NULL`, cadenas vacías y cadenas de longitud par o impar. Devuelve código C compatible con GCC y explica brevemente su complejidad.
3. Código Devuelto por el Modelo
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
Evidencia de ejecución: `Original: MUNDO` → `Invertida: ODNUM`.
4. Fase 3: El Auditor (Resultados del Test Harness)
Pruebas ejecutadas en el archivo `harness_sesion01_c.c`:
[x] Cadena normal: `HOLA` → `ALOH`.
[x] Longitud impar: `ABCDE` → `EDCBA`.
[x] Cadena vacía: `""` → `""`.
[x] Puntero `NULL`: la función retorna sin intentar desreferenciarlo.
[x] Cadena de un carácter: `A` → `A`.
Salida observada: `TODAS LAS PRUEBAS PASARON`.
Comprobaciones de la plantilla:
[x] Los casos probados terminaron sin fallos visibles (crash).
[ ] Ausencia de accesos fuera de límites y fugas de memoria verificada con herramientas especializadas (pendiente: no se ejecutó AddressSanitizer ni Valgrind).
[x] La complejidad teórica de la función revisada es O(n) en tiempo y O(1) en memoria auxiliar.
Reporte Crítico
La solución de IA considera el caso `NULL` y funciona con las entradas probadas. Una limitación importante de la auditoría realizada es que el test harness contiene una copia de la función `invertir` y no enlaza directamente el archivo `ia_sesion01_c.c`. Por eso, aunque las cinco pruebas pasaron, todavía es recomendable ejecutar las pruebas contra la implementación original de IA. Además, la función presupone que recibe una cadena válida, terminada en `\0` y almacenada en memoria modificable; no debe usarse sobre literales de cadena ni buffers sin terminador.
Firma digital: `Paola-23-ux`

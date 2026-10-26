
# Bitácora de Requerimientos y Auditoría IA — Sesión 03
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Google Colab, Mágicos y Flujo de Entrega  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Ingrid Paola Dzul Segovia  
**Matrícula:** 24531350  
**Track:** C

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Manipulación de archivos y llamadas OS.

*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> Una de las principales dificultades fue comprender cómo abrir, escribir, leer y cerrar archivos correctamente en lenguaje C. También fue necesario entender cómo contar las líneas de un archivo, especialmente cuando la última línea no termina con un salto de línea.
>
> Otro aspecto importante fue identificar cómo obtener el código de salida de un comando utilizando `system()`, `WIFEXITED` y `WEXITSTATUS`.
>
> Para resolver estos problemas se utilizaron funciones de manejo de archivos y condiciones para detectar errores y devolver los valores correspondientes.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | Lenguaje C11 utilizando GCC en Google Colab para manipular archivos y ejecutar comandos. |
| **Rol (R)** | Ingeniero de software especializado en lenguaje C y sistemas POSIX. |
| **Especificación (E)** | Implementar tres funciones para escribir texto, contar líneas y ejecutar comandos, respetando las firmas de `api_sesion03.h`, manejando errores y devolviendo códigos de salida correctos. |
| **Output (O)** | Código completo en C, sin función `main()`, compatible con GCC y sin advertencias de compilación. |

### 2.2 Prompt Estructurado

```text
Contexto:
Estoy realizando una práctica de Lenguajes y Autómatas II
en Google Colab utilizando lenguaje C11 y GCC.

Rol:
Actúa como un programador especializado en lenguaje C
y sistemas POSIX.

Especificación:
Necesito implementar tres funciones:

int escribir_texto(const char *ruta, const char *texto);
int contar_lineas(const char *ruta);
int ejecutar(const char *comando);

La primera función debe escribir texto en un archivo
y devolver 0 si funciona o -1 si ocurre un error.

La segunda debe contar las líneas de un archivo,
incluyendo la última cuando no tenga salto de línea.
Si el archivo no existe, debe devolver -1.

La tercera debe ejecutar comandos externos y obtener
el código real de salida utilizando WEXITSTATUS.

Los archivos deben cerrarse correctamente.
No se debe incluir main().

Output:
Proporciona el código completo en lenguaje C,
compatible con GCC y sin advertencias de compilación.
```

---

## 3. Código Devuelto por el Modelo

```c
#include "api_sesion03.h"
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>

int escribir_texto(const char *ruta, const char *texto) {
    if (ruta == NULL || texto == NULL) {
        return -1;
    }

    FILE *archivo = fopen(ruta, "w");
    if (archivo == NULL) {
        return -1;
    }

    int resultado = fputs(texto, archivo);
    int cierre = fclose(archivo);

    if (resultado == EOF || cierre == EOF) {
        return -1;
    }

    return 0;
}

int contar_lineas(const char *ruta) {
    if (ruta == NULL) {
        return -1;
    }

    FILE *archivo = fopen(ruta, "r");
    if (archivo == NULL) {
        return -1;
    }

    int lineas = 0;
    int caracter;
    int ultimo = '\n';

    while ((caracter = fgetc(archivo)) != EOF) {
        if (caracter == '\n') {
            lineas++;
        }
        ultimo = caracter;
    }

    int error = ferror(archivo);
    int cierre = fclose(archivo);

    if (error || cierre == EOF) {
        return -1;
    }

    if (ultimo != '\n') {
        lineas++;
    }

    return lineas;
}

int ejecutar(const char *comando) {
    if (comando == NULL) {
        return -1;
    }

    int resultado = system(comando);

    if (resultado == -1) {
        return -1;
    }

    if (WIFEXITED(resultado)) {
        return WEXITSTATUS(resultado);
    }

    return -1;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)

*Pruebas ejecutadas:* Compilación GCC utilizando `-Wall -Wextra -Werror` y ejecución de seis casos de prueba.

| Caso | Esperado | Obtenido | Resultado |
| :--- | :---: | :---: | :---: |
| Escribir texto en archivo | 0 | 0 | OK |
| Contar archivo de 3 líneas | 3 | 3 | OK |
| Contar líneas sin salto final | 2 | 2 | OK |
| Archivo inexistente | -1 | -1 | OK |
| Ejecutar comando `true` | 0 | 0 | OK |
| Ejecutar comando `false` | 1 | 1 | OK |

- [x] Casos base y límites evaluados superados sin errores.
- [ ] No existen accesos fuera de límites ni *Memory Leaks* (pendiente de verificación especializada).
- [x] La complejidad teórica es coherente con la implementación.

**Resultados:**

- Fase 1 — El Artesano: 6/6 pruebas superadas.
- Fase 2 — El Científico: 6/6 pruebas superadas.

**Salida del harness:**

```text
=== AUDITORIA SESION 03 - Archivos y codigos de salida ===
[ 1] escribir_texto(...)==0         esperado: 0    obtenido: 0    OK
[ 2] contar_lineas(3 lineas)         esperado: 3    obtenido: 3    OK
[ 3] contar_lineas(sin salto final)  esperado: 2    obtenido: 2    OK
[ 4] contar_lineas(inexistente)      esperado: -1   obtenido: -1   OK
[ 5] ejecutar("true")                esperado: 0    obtenido: 0    OK
[ 6] ejecutar("false")               esperado: 1    obtenido: 1    OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

### Reporte Crítico

*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> Al revisar el código generado por la IA, identifiqué que era importante comprobar el caso de un archivo cuya última línea no termina con un salto de línea. También revisé el manejo de archivos inexistentes y los códigos de salida de los comandos.
>
> La implementación generada ya contemplaba estos casos, por lo que no fue necesario realizar correcciones después de ejecutar las pruebas. El harness confirmó que los seis casos devolvieron los resultados esperados.
>
> Esta práctica me ayudó a comprender la importancia de revisar el código generado por inteligencia artificial y comprobar que funcione correctamente mediante pruebas.

---

**Firma digital:** Paola-23-ux
  

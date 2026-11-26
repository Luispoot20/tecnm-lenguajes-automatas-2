# Bitácora de Requerimientos y Auditoría IA — Sesión 03
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Google Colab, Mágicos y Flujo de Entrega  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Emiliano Xooc Hau  
**Matrícula:** 24531400  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Escritura y conteo de líneas en archivos  
En esta práctica tuve que trabajar con archivos de texto desde C. Al principio pensé que contar líneas consistía únicamente en buscar el carácter \n, pero al probar diferentes archivos encontré una situación que requería más atención: cuando un archivo termina sin un salto de línea, todavía debe contabilizarse su última línea.

Para resolverlo utilicé fgetc() y guardé el último carácter leído. También agregué una variable para saber si el archivo contenía información. Esto me permitió diferenciar entre un archivo vacío y otro que contiene texto, aunque no termine con un salto de línea.

Otra parte importante fue comprobar que fopen() funcionara correctamente y cerrar el archivo con fclose() al terminar. Aprendí que no basta con conseguir que el programa funcione; también hay que considerar qué sucede cuando un archivo no puede abrirse.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Salida estándar y códigos de retorno ABI |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Necesito desarrollar dos funciones en lenguaje C para
trabajar con archivos de texto.

Contexto:
El programa será utilizado en un laboratorio de
Lenguajes y Autómatas II y debe compilar con GCC.

Funciones requeridas:

1. escribir_texto:
   - Recibe una ruta y una cadena de texto.
   - Crea o sobrescribe el archivo.
   - Devuelve 0 cuando la escritura termina correctamente.
   - Devuelve -1 cuando existe algún problema.

2. contar_lineas:
   - Recibe la ruta de un archivo.
   - Cuenta las líneas existentes.
   - Debe reconocer archivos vacíos.
   - Debe contar la última línea aunque no tenga
     un salto de línea al final.
   - Devuelve -1 cuando no puede abrir o leer el archivo.

Restricciones:
- Respetar las firmas del archivo api_sesion03.h.
- Utilizar fopen, fgetc, fwrite y fclose cuando corresponda.
- Comprobar los errores de apertura, lectura y escritura.
- No incluir main.
- Cerrar los archivos después de utilizarlos.
- Evitar advertencias con -Wall -Wextra -Werror.

Entrega el código completo y explica cómo funcionan
las dos funciones y su complejidad.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion03.h"
#include <stdio.h>
#include <string.h>

int escribir_texto(const char *ruta, const char *texto) {
    if (ruta == NULL || texto == NULL) {
        return -1;
    }

    FILE *archivo = fopen(ruta, "w");

    if (archivo == NULL) {
        return -1;
    }

    size_t longitud = strlen(texto);
    size_t escritos = fwrite(texto, 1, longitud, archivo);
    int error = ferror(archivo);
    int cierre = fclose(archivo);

    if (escritos != longitud || error || cierre != 0) {
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
    int hay_caracteres = 0;

    while ((caracter = fgetc(archivo)) != EOF) {
        hay_caracteres = 1;

        if (caracter == '\n') {
            lineas++;
        }

        ultimo = caracter;
    }

    int error = ferror(archivo);
    int cierre = fclose(archivo);

    if (error || cierre != 0) {
        return -1;
    }

    if (hay_caracteres && ultimo != '\n') {
        lineas++;
    }

    return lineas;
}


```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Archivo vacío, sin salto final y ruta inexistente

### Salida del harness
```text
=== AUDITORIA SESION 03 - Archivos: escribir y contar lineas ===
Caso                               esperado:              obtenido:              resultado
[ 1] escribir_texto(...)==0             esperado: 0            obtenido: 0            OK
[ 2] contar_lineas(3 lineas)            esperado: 3            obtenido: 3            OK
[ 3] contar_lineas(sin salto final)     esperado: 2            obtenido: 2            OK
[ 4] contar_lineas(archivo vacio)       esperado: 0            obtenido: 0            OK
[ 5] contar_lineas(inexistente)         esperado: -1           obtenido: -1           OK
[ 6] escribir_texto(ruta invalida)      esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
Al analizar los resultados, me di cuenta de que las pruebas cubrían los errores más comunes, pero no comprobaban directamente qué sucedía cuando las funciones recibían un puntero NULL.

Revisé ambas funciones y encontré que ya existían condiciones para controlar esa situación: escribir_texto() verifica tanto la ruta como el contenido, mientras que contar_lineas() comprueba la ruta antes de intentar abrir el archivo.

También noté que el código verifica los posibles errores durante la lectura, escritura y cierre de los archivos. Por eso no fue necesario modificar la implementación para superar las pruebas disponibles.

Como mejora para otra prueba, agregaría casos específicos para los argumentos NULL y para errores de lectura o escritura que ocurran después de abrir un archivo. Esto permitiría comprobar situaciones que el harness actual no evalúa.
> 

---
**Firma digital:** Luis Enrique Noh Poot

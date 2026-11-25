# Bitácora de Requerimientos y Auditoría IA — Sesión 01
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Encuadre, Diagnóstico de Punteros y Metodología IA  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:**   Luis Enrique Noh Poot
**Matrícula:**  24531400
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Inversión de buffer in-place (char*)  
Al realizar el ejercicio manualmente, mi principal dificultad fue comprender cómo recorrer una cadena utilizando posiciones y modificar sus caracteres sin crear otra cadena. También tuve que identificar cómo obtener la última posición mediante strlen() y cómo intercambiar los caracteres utilizando una variable temporal.

Para resolverlo utilicé dos variables, una para comenzar desde el inicio y otra para avanzar desde el final. Después intercambié los caracteres hasta que ambas posiciones se encontraron. También agregué una condición para comprobar si el puntero era NULL y evitar que el programa presentara un error.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Punteros, O(n) tiempo, O(1) memoria |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como un ingeniero de software especializado en programación
en C y desarrollo de compiladores.

Necesito implementar una función llamada invertir(char *s)
que permita invertir una cadena de caracteres directamente
en su mismo espacio de memoria.

Requisitos:
1. Utilizar el lenguaje C.
2. No utilizar malloc ni crear arreglos auxiliares.
3. Utilizar una variable temporal para intercambiar caracteres.
4. La función debe manejar correctamente cadenas vacías.
5. Si el puntero es NULL, debe terminar sin provocar errores.
6. Mantener exactamente la firma void invertir(char *s).
7. Tener complejidad O(n) en tiempo y O(1) en memoria.
8. No incluir una función main.

Entrega el código completo y explica brevemente
cómo funciona y cuál es su complejidad.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion01.h"
#include <string.h>
#include <stddef.h>

void invertir(char *s) {
    int inicio = 0;
    int final;
    char temporal;

    /* Verificar si el puntero es NULL */
    if (s == NULL) {
        return;
    }

    /* Obtener la ultima posicion de la cadena */
    final = (int)strlen(s) - 1;

    /* Invertir los caracteres */
    while (inicio < final) {
        temporal = s[inicio];
        s[inicio] = s[final];
        s[final] = temporal;

        inicio++;
        final--;
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Valores NULL, buffers vacios y longitudes impares

### Salida del harness
```text
=== AUDITORIA SESION 01 - Inversion de buffer in-place ===
Caso                               esperado:              obtenido:              resultado
[ 1] invertir("HOLA")                   esperado: ALOH         obtenido: ALOH         OK
[ 2] invertir("abc")                    esperado: cba          obtenido: cba          OK
[ 3] invertir("")                       esperado:              obtenido:              OK
[ 4] invertir("AB")                     esperado: BA           obtenido: BA           OK
[ 5] invertir("a")                      esperado: a            obtenido: a            OK
[ 6] invertir(NULL)                     esperado: sin crash    obtenido: sin crash    OK

RESULTADO: 6/6 pruebas superadas
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
Al revisar el código generado por la IA, observé que resolvía correctamente los casos principales, incluyendo las cadenas vacías y el puntero NULL. Sin embargo, identifiqué que utilizaba una variable de tipo int para guardar el resultado de strlen(), que devuelve un valor de tipo size_t. En cadenas extremadamente grandes, esa conversión podría provocar problemas por el límite de representación de int.

Para esta actividad, el código superó las seis pruebas del laboratorio. Como mejora, sería conveniente utilizar size_t para manejar las posiciones y evitar conversiones innecesarias.

También comprobé que tanto mi implementación manual como la implementación generada por la IA obtuvieron el mismo resultado: 6 de 6 pruebas superadas.
> 

---
**Firma digital:** Luis Enrique Noh Poot

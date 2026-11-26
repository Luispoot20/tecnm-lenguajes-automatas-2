# Bitácora de Requerimientos y Auditoría IA — Sesión 06
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Sistema de Tipos Primitivos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Luis Enrique Noh Poot  
**Matrícula:** 24531400  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Chequeo de compatibilidad de tipos  
En esta actividad me enfoqué en entender cómo un compilador puede decidir si una asignación entre dos variables es válida. Para ello trabajé con tres tipos de datos: CHAR, INT y FLOAT.

Lo primero que tuve que identificar fue el orden de estos tipos. No todos tienen la misma capacidad para representar información. Por ejemplo, guardar un valor entero dentro de un tipo flotante es una conversión permitida, pero intentar guardar un flotante en un entero puede provocar pérdida de información.

Para resolverlo organicé las condiciones según el tipo de destino y el tipo de origen. También tuve que considerar el valor TIPO_ERROR, porque no debe aceptarse en ninguna asignación.

La parte que más atención requirió fue evitar que una comparación numérica sencilla aceptara combinaciones que realmente deberían rechazarse.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Matriz de compatibilidad estática |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como un ingeniero de software especializado
en compiladores y análisis semántico.

Necesito implementar un sistema de compatibilidad
de tipos primitivos en lenguaje C.

El archivo api_sesion06.h declara el siguiente enumerado:

typedef enum {
    TIPO_ERROR = 0,
    TIPO_CHAR = 1,
    TIPO_INT = 2,
    TIPO_FLOAT = 3
} Tipo;

Debes implementar exactamente estas funciones:

int compatible(Tipo destino, Tipo origen);
const char *nombre_tipo(Tipo t);

REQUISITOS:

1. Respetar exactamente las firmas indicadas.
2. La jerarquía debe ser:
   CHAR < INT < FLOAT.
3. Permitir asignaciones al mismo tipo.
4. Permitir asignaciones hacia un tipo de mayor rango.
5. Rechazar asignaciones hacia tipos de menor rango.
6. TIPO_ERROR nunca debe ser compatible.
7. nombre_tipo debe devolver:
   "error", "char", "int" o "float".
8. Para valores desconocidos, devolver "desconocido".
9. No utilizar memoria dinámica.
10. No incluir main().
11. El código debe compilar con GCC utilizando:
    -std=c11 -Wall -Wextra -Werror.

Entrega el código completo y explica las reglas
utilizadas para determinar la compatibilidad.

También analiza la complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion06.h"

int compatible(Tipo destino, Tipo origen) {

    /* Rechazar tipos de error y valores no definidos */
    if (destino < TIPO_CHAR || destino > TIPO_FLOAT ||
        origen < TIPO_CHAR || origen > TIPO_FLOAT) {

        return 0;
    }

    /* Permitir conversiones hacia un tipo del mismo
       rango o de mayor rango */

    return destino >= origen;
}

const char *nombre_tipo(Tipo t) {

    switch (t) {

        case TIPO_ERROR:
            return "error";

        case TIPO_CHAR:
            return "char";

        case TIPO_INT:
            return "int";

        case TIPO_FLOAT:
            return "float";

        default:
            return "desconocido";
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Rechazo de asignaciones incompatibles

### Salida del harness
```text
=== AUDITORIA SESION 06 - Tabla de compatibilidad de tipos ===
Caso                               esperado:              obtenido:              resultado
[ 1] compatible(INT,INT)                esperado: 1            obtenido: 1            OK
[ 2] compatible(FLOAT,INT)              esperado: 1            obtenido: 1            OK
[ 3] compatible(INT,FLOAT)              esperado: 0            obtenido: 0            OK
[ 4] compatible(INT,ERROR)              esperado: 0            obtenido: 0            OK
[ 5] nombre_tipo(INT)                   esperado: int          obtenido: int          OK
[ 6] nombre_tipo(ERROR)                 esperado: error        obtenido: error        OK

RESULTADO: 6/6 pruebas superadas
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
Durante la revisión encontré un detalle que considero importante: el valor de TIPO_ERROR es cero. Esto puede causar un problema si alguien implementa la compatibilidad utilizando únicamente una comparación entre los números de los enumerados.

Por ejemplo, una condición como destino >= origen podría considerar válido compatible(TIPO_INT, TIPO_ERROR), porque el número de INT es mayor que cero.

Para evitar ese comportamiento, la implementación de IA verifica primero que ambos tipos sean válidos y que ninguno pertenezca a la categoría de error.

Otro aspecto que observé es que las seis pruebas no comprueban todas las combinaciones posibles. Por ejemplo, no incluyen directamente todas las asignaciones que involucran CHAR ni los valores desconocidos del enumerado.

Como mejora, sería conveniente ampliar el harness para probar esas combinaciones y comprobar que la función se comporte correctamente ante valores no definidos.
> 

---
**Firma digital:** [Usuario de GitHub] Luis Enrique Noh Poot

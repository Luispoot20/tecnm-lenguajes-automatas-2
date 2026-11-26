# Bitácora de Requerimientos y Auditoría IA — Sesión 12
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Atributos de Símbolos y Memoria  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Luis Enrique Noh Poot  
**Matrícula:** 24531400  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Estructura de registro de símbolos  
En esta práctica ya no trabajé directamente con operaciones matemáticas del árbol sintáctico. El problema fue organizar cómo quedarían ubicadas las variables dentro de un espacio de memoria. Para hacerlo tuve que distinguir dos valores: el cursor que indica dónde terminó el dato anterior y el offset que debe guardarse en el nuevo símbolo.

La parte que requiere más cuidado es la alineación. No siempre se puede colocar una variable justo en la posición donde terminó la anterior. Por ejemplo, después de reservar un char, el cursor queda en 1; pero un int debe comenzar en una posición múltiplo de 4. Por eso, antes de guardar el offset, se calcula cuánto espacio de relleno hace falta.

También consideré que no todos los datos recibidos son válidos. Un registro nulo, un cursor negativo o un nombre de tipo que no esté en la tabla deben rechazarse con -1.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Cálculo de desplazamientos y alineación |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como un ingeniero de software que trabaja en el diseño
interno de compiladores y distribución de memoria.

Implementa en C las funciones declaradas en api_sesion12.h:

int ancho_de_tipo(const char *tipo);
int asignar_offset(Simbolo *simbolo, int cursor);

Condiciones obligatorias:
1. Respetar exactamente las firmas y la estructura Simbolo.
2. Devolver estos tamaños: char=1, short=2, int=4,
   float=4, double=8 y puntero=8.
3. Devolver -1 ante un tipo desconocido o un tipo NULL.
4. Rechazar un registro NULL y un cursor negativo.
5. Alinear el offset al siguiente múltiplo del ancho del tipo.
6. Guardar el offset alineado dentro de simbolo->offset.
7. Devolver el cursor siguiente, es decir, offset + ancho.
8. No modificar el registro si la operación no es válida.
9. Evitar desbordamientos de enteros al calcular el padding
   y el cursor final.
10. No incluir main; el programa de pruebas proporciona main.
11. El código debe compilar con GCC usando
    -std=c11 -Wall -Wextra -Werror.

Entrega el código modular y explica el cálculo de alineación,
los casos inválidos y la complejidad.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion12.h"
#include <limits.h>
#include <string.h>

typedef struct {
    const char *nombre;
    int bytes;
} TipoInfo;

static const TipoInfo tipos[] = {
    {"char", 1},
    {"short", 2},
    {"int", 4},
    {"float", 4},
    {"double", 8},
    {"puntero", 8}
};

int ancho_de_tipo(const char *tipo) {
    size_t i;

    if (tipo == NULL) {
        return -1;
    }

    for (i = 0; i < sizeof(tipos) / sizeof(tipos[0]); i++) {
        if (strcmp(tipo, tipos[i].nombre) == 0) {
            return tipos[i].bytes;
        }
    }

    return -1;
}

int asignar_offset(Simbolo *simbolo, int cursor) {
    int bytes;
    int residuo;
    int offset_alineado;
    int padding;

    if (simbolo == NULL || cursor < 0) {
        return -1;
    }

    bytes = ancho_de_tipo(simbolo->tipo);
    if (bytes == -1) {
        return -1;
    }

    residuo = cursor % bytes;
    padding = (bytes - residuo) % bytes;

    if (cursor > INT_MAX - padding) {
        return -1;
    }

    offset_alineado = cursor + padding;
    if (offset_alineado > INT_MAX - bytes) {
        return -1;
    }

    simbolo->offset = offset_alineado;
    return offset_alineado + bytes;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Alineación de tipos y tamaños en bytes

### Salida del harness
```text
=== AUDITORIA SESION 12 - Registro de simbolo y offsets alineados ===
Caso                               esperado:              obtenido:              resultado
[ 1] ancho_de_tipo("int")               esperado: 4            obtenido: 4            OK
[ 2] ancho_de_tipo("double")            esperado: 8            obtenido: 8            OK
[ 3] int en cursor 0                    esperado: 4            obtenido: 4            OK
[ 4] char e int: offset de b            esperado: 4            obtenido: 4            OK
[ 5] double desde cursor 4              esperado: 8            obtenido: 8            OK
[ 6] registro NULL                      esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
Al revisar el alcance del harness noté que comprueba el tamaño de int y double, algunos escenarios de alineación y el registro NULL, pero no ejecuta directamente todos los rechazos descritos en el contrato. Por ejemplo, no tiene casos específicos para un tipo desconocido, un puntero tipo igual a NULL o un cursor negativo.

Por eso añadí esas validaciones antes de calcular el residuo o modificar el registro. También incorporé una comprobación de desbordamiento: si al sumar el padding o el tamaño se supera el límite de un int, la función devuelve -1 y no cambia el offset. Esto es importante porque una suma fuera de rango podría producir un cursor incorrecto.

La sesión me ayudó a entender que el compilador no puede colocar variables una detrás de otra sin considerar sus requisitos de alineación. A veces quedan bytes sin utilizar entre dos datos, pero ese espacio permite que cada variable comience en una dirección adecuada. También comprendí que asignar_offset() hace dos trabajos relacionados: registra dónde empieza el símbolo actual y calcula dónde podrá empezar el siguiente.
> 

---
**Firma digital:** Luis Enrique Noh Poot

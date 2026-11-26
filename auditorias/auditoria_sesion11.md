# Bitácora de Requerimientos y Auditoría IA — Sesión 11
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Arquitectura de la Tabla de Símbolos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Luis Enrique Noh Poot  
**Matrícula:** 24531400  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Tabla Hash con encadenamiento  
En esta sesión el trabajo consistió en organizar nombres de símbolos dentro de una tabla hash. La idea es que el compilador pueda localizar un identificador mediante una función de dispersión, en vez de revisar una lista completa desde el principio cada vez.

La práctica se dividió en tres partes: calcular el hash, insertar símbolos y comprobar qué sucede cuando dos nombres terminan en la misma cubeta. También había que impedir que un identificador se guardara dos veces.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Función de dispersión y colisiones O(1) |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como un ingeniero de software especializado en C
y en estructuras de datos para compiladores.

Implementa las funciones declaradas en api_sesion11.h:

unsigned int hash_djb2(const char *clave);

int insertar_simbolo(Simbolo *tabla[], unsigned int cubetas,
                     const char *nombre, int tipo, int offset);

Requisitos:
1. Implementa djb2 comenzando con hash = 5381.
2. Por cada byte, actualiza el hash multiplicando por 33
   y sumando el byte como unsigned char.
3. insertar_simbolo debe devolver:
   1 si agrega un símbolo nuevo,
   0 si el nombre ya existe,
   -1 si recibe argumentos inválidos o falla malloc.
4. Rechaza tabla NULL, nombre NULL y cubetas igual a cero.
5. Cada Simbolo tiene nombre[16], tipo, offset y sig.
6. No permitas nombres que no quepan en nombre[16],
   contando el terminador nulo.
7. Calcula la cubeta con hash_djb2(nombre) % cubetas.
8. Recorre la lista de esa cubeta para detectar duplicados.
9. No sobrescribas una entrada cuando exista una colisión:
   agrega el nuevo nodo al final usando sig.
10. Inicializa todos los campos del nodo.
11. No cambies las firmas, no agregues main y utiliza
    solamente C estándar.
12. El programa debe compilar con:
    gcc -std=c11 -Wall -Wextra -Werror

Explica el manejo de colisiones, los valores de retorno
y la complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion11.h"
#include <stdlib.h>
#include <string.h>

unsigned int hash_djb2(const char *clave) {
    unsigned int hash = 5381u;

    if (clave == NULL) {
        return hash;
    }

    while (*clave) {
        hash = ((hash << 5) + hash) + (unsigned char)*clave;
        ++clave;
    }

    return hash;
}

int insertar_simbolo(Simbolo *tabla[], unsigned int cubetas,
                     const char *nombre, int tipo, int offset) {
    size_t longitud;
    unsigned int indice;
    Simbolo **enlace;
    Simbolo *nuevo;

    if (tabla == NULL || nombre == NULL || cubetas == 0u) {
        return -1;
    }

    longitud = strlen(nombre);
    if (longitud >= sizeof(((Simbolo *)0)->nombre)) {
        return -1;
    }

    indice = hash_djb2(nombre) % cubetas;
    enlace = &tabla[indice];

    while (*enlace != NULL) {
        if (strcmp((*enlace)->nombre, nombre) == 0) {
            return 0;
        }
        enlace = &((*enlace)->sig);
    }

    nuevo = malloc(sizeof(*nuevo));
    if (nuevo == NULL) {
        return -1;
    }

    memcpy(nuevo->nombre, nombre, longitud + 1u);
    nuevo->tipo = tipo;
    nuevo->offset = offset;
    nuevo->sig = NULL;

    *enlace = nuevo;
    return 1;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Manejo de claves duplicadas y colisiones

### Salida del harness
```text
=== AUDITORIA SESION 11 - Tabla hash con encadenamiento ===
Caso                               esperado:              obtenido:              resultado
[ 1] hash_djb2("")                      esperado: 5381         obtenido: 5381         OK
[ 2] hash_djb2("abc")                   esperado: 193485963    obtenido: 193485963    OK
[ 3] insertar x dos veces               esperado: 0            obtenido: 0            OK
[ 4] dos claves, una cubeta             esperado: 2            obtenido: 2            OK
[ 5] offset de beta en cadena           esperado: 4            obtenido: 4            OK
[ 6] insertar en tabla NULL             esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
Al revisar la solución me fijé especialmente en el tamaño del campo nombre[16]. El harness prueba nombres cortos como x, alpha y beta, pero no incluye un identificador que exceda la capacidad de ese arreglo.

Ese caso es importante porque una copia directa sin validar la longitud podría escribir fuera del espacio reservado para el nombre. Para prevenirlo, antes de reservar e insertar el nodo se calcula strlen(nombre) y se rechaza la entrada cuando su longitud es igual o mayor que el tamaño del arreglo. De esta forma siempre queda espacio para el carácter terminador '\0'.

También comprobé que una tabla con una sola cubeta conserva los dos símbolos. Esto permite observar el encadenamiento: la segunda entrada queda enlazada a la primera y no la reemplaza.

La limitación que queda fuera del contrato es la liberación de todos los nodos al terminar el programa: el encabezado de esta sesión no declara una función destructora. En una aplicación completa sería conveniente añadir una operación para recorrer las cubetas y liberar cada nodo con free().
> 

---
**Firma digital:** Luis Enrique Noh Poot

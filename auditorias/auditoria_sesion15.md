# Bitácora de Requerimientos y Auditoría IA — Sesión 15
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Bloques Anidados y Ciclo de Vida  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Luis Enrique Noh Poot  
**Matrícula:** 24531400  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Operaciones Push y Pop de ámbito  
En esta práctica tuve que cuidar dos momentos distintos del ciclo de vida de un bloque. Al entrar a un bloque se necesita reservar un nodo nuevo y conectarlo con el ámbito que ya estaba activo. Al salir, en cambio, se debe recuperar la dirección del padre antes de liberar el nodo actual.

La parte delicada fue no tratar todos los ámbitos como si tuvieran la misma propiedad de memoria. La raíz se crea fuera de push_ambito(), por lo que no debe liberarse al ejecutar pop_ambito() sobre ella. En cambio, los ámbitos creados por push_ambito() sí se reservan con malloc() y deben liberarse al salir.

También tuve que contemplar entradas inválidas: no se puede crear un bloque si el padre o el nombre son NULL, y tampoco se debe intentar retirar un ámbito inexistente.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Liberación de memoria al cerrar bloque |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como un programador de C con experiencia en
estructuras enlazadas y administración de memoria.

Implementa estas funciones respetando exactamente
las declaraciones de api_sesion15.h:

Ambito *push_ambito(Ambito *actual, const char *nombre);
Ambito *pop_ambito(Ambito *actual);

Requisitos para push_ambito:
- Si actual o nombre son NULL, devolver NULL.
- Reservar un nuevo nodo Ambito con malloc.
- Si malloc falla, devolver NULL.
- Guardar el nombre recibido en el nuevo nodo.
- Enlazar el campo padre con actual.
- Devolver el nuevo nodo sin modificar el padre.

Requisitos para pop_ambito:
- Si actual es NULL, devolver NULL.
- Si actual no tiene padre, es la raíz: devolver NULL
  sin liberarla.
- Si tiene padre, guardar primero la dirección del padre.
- Liberar únicamente el nodo actual.
- Devolver el padre guardado.

No agregues main, no cambies las firmas y no modifiques
la estructura Ambito. El código debe compilar con:
gcc -std=c11 -Wall -Wextra -Werror

Entrega solamente las funciones y una explicación
breve de la propiedad de memoria.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion15.h"
#include <stdlib.h>

static int puede_crear_ambito(const Ambito *actual, const char *nombre) {
    return actual != NULL && nombre != NULL;
}

Ambito *push_ambito(Ambito *actual, const char *nombre) {
    if (!puede_crear_ambito(actual, nombre)) {
        return NULL;
    }

    Ambito *hijo = malloc(sizeof *hijo);
    if (hijo == NULL) {
        return NULL;
    }

    *hijo = (Ambito){
        .nombre = nombre,
        .padre = actual
    };

    return hijo;
}

Ambito *pop_ambito(Ambito *actual) {
    if (actual == NULL) {
        return NULL;
    }

    Ambito *padre = actual->padre;

    /* La raiz no fue reservada por push_ambito: no se libera. */
    if (padre == NULL) {
        return NULL;
    }

    actual->nombre = NULL;
    actual->padre = NULL;
    free(actual);

    return padre;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Aislamiento de variables fuera de bloque

### Salida del harness
```text
=== AUDITORIA SESION 15 - Push y pop de ambitos ===
Caso                               esperado:              obtenido:              resultado
[ 1] nombre del ambito nuevo            esperado: if           obtenido: if           OK
[ 2] profundidad con dos push           esperado: 3            obtenido: 3            OK
[ 3] pop devuelve el padre              esperado: 1            obtenido: 1            OK
[ 4] pop sobre la raiz                  esperado: 1            obtenido: 1            OK
[ 5] push con padre NULL                esperado: 1            obtenido: 1            OK
[ 6] profundidad tras pop               esperado: 2            obtenido: 2            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion15.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
Al revisar las dos soluciones identifiqué un detalle que el harness no mide directamente: el campo nombre es un puntero const char *, y las funciones guardan la dirección recibida, no una copia independiente del texto. Esto significa que la cadena original debe seguir existiendo mientras el ámbito la utilice. En los casos del laboratorio se emplean literales, por lo que su duración es suficiente.

También observé que el harness no fuerza un fallo de malloc(). Las dos implementaciones verifican si la reserva devuelve NULL, pero esa situación no queda demostrada por las seis pruebas oficiales.

La propiedad de memoria también debe respetarse al usar las funciones: pop_ambito() está pensado para retirar nodos que fueron creados dinámicamente mediante push_ambito(). La raíz del programa es externa y se conserva. Por eso no se debe llamar a pop_ambito() sobre un objeto local que no haya sido reservado con malloc() y que tenga un padre distinto de NULL.
> 

---
**Firma digital:** Luis Enrique Noh Poot

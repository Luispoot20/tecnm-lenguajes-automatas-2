# Bitácora de Requerimientos y Auditoría IA — Sesión 02
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Gestión en GitHub Web y Nodos Dinámicos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Luis Enrique Noh Poot  
**Matrícula:** 24531402  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Lista enlazada de tokens en Heap  
Al realizar el ejercicio manualmente, mi principal dificultad fue comprender cómo crear y conectar los nodos de una lista enlazada utilizando memoria dinámica. Tuve que aprender a reservar memoria con malloc(), recorrer la lista mediante punteros y agregar cada nuevo token al final sin perder la referencia del primer nodo.

También tuve que tener cuidado al liberar la memoria, porque primero es necesario guardar la dirección del siguiente nodo antes de utilizar free(). De lo contrario, se podría perder el acceso a los demás elementos de la lista.

Para resolverlo utilicé tres funciones: una para agregar tokens, otra para contarlos y una última para liberar toda la memoria utilizada.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Gestión de memoria dinámica C99/Rust |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como un ingeniero de software especializado en C
y desarrollo de compiladores.

Necesito implementar una lista enlazada de tokens
utilizando memoria dinámica en C.

Requisitos:

1. Crear la función agregar_token para insertar un
   nuevo token al final de la lista.
2. Utilizar malloc para reservar memoria.
3. Comprobar que la reserva de memoria sea correcta.
4. Manejar correctamente una lista vacía.
5. Evitar problemas cuando el lexema sea NULL.
6. Mantener el campo siguiente del último nodo en NULL.
7. Crear contar_tokens para contar los nodos.
8. Crear liberar_lista para liberar toda la memoria.
9. Evitar accesos a memoria después de liberar nodos.
10. Respetar exactamente estas firmas:

NodoToken *agregar_token(NodoToken *cabeza,
                         const char *lexema);

int contar_tokens(NodoToken *cabeza);

void liberar_lista(NodoToken *cabeza);

No utilizar main y entregar código modular,
compatible con el compilador GCC.
Explicar brevemente el funcionamiento y la complejidad.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion02.h"
#include <stdlib.h>
#include <string.h>

/* Agregar un token al final de la lista */
NodoToken *agregar_token(NodoToken *cabeza, const char *lexema) {
    if (lexema == NULL) {
        return cabeza;
    }

    NodoToken *nuevo = malloc(sizeof(NodoToken));

    if (nuevo == NULL) {
        return cabeza;
    }

    strncpy(nuevo->lexema, lexema, sizeof(nuevo->lexema) - 1);
    nuevo->lexema[sizeof(nuevo->lexema) - 1] = '\0';
    nuevo->siguiente = NULL;

    if (cabeza == NULL) {
        return nuevo;
    }

    NodoToken *actual = cabeza;

    while (actual->siguiente != NULL) {
        actual = actual->siguiente;
    }

    actual->siguiente = nuevo;

    return cabeza;
}

/* Contar los tokens de la lista */
int contar_tokens(NodoToken *cabeza) {
    int contador = 0;
    NodoToken *actual = cabeza;

    while (actual != NULL) {
        contador++;
        actual = actual->siguiente;
    }

    return contador;
}

/* Liberar la memoria de todos los nodos */
void liberar_lista(NodoToken *cabeza) {
    NodoToken *actual = cabeza;

    while (actual != NULL) {
        NodoToken *temporal = actual;
        actual = actual->siguiente;
        free(temporal);
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Prevención de Use-After-Free y cadenas nulas

### Salida del harness
```text
=== AUDITORIA SESION 02 - Lista enlazada de tokens en heap ===
Caso                               esperado:              obtenido:              resultado
[ 1] agregar_token(NULL,"if")           esperado: if           obtenido: if           OK
[ 2] contar_tokens(3 tokens)            esperado: 3            obtenido: 3            OK
[ 3] orden: segundo token               esperado: dos          obtenido: dos          OK
[ 4] ultimo->siguiente == NULL          esperado: 1            obtenido: 1            OK
[ 5] contar_tokens(NULL)                esperado: 0            obtenido: 0            OK
[ 6] liberar_lista(NULL)                esperado: sin crash    obtenido: sin crash    OK

RESULTADO: 6/6 pruebas superadas
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
Al revisar el código generado por la IA, observé que implementaba correctamente las funciones principales y también verificaba si el lexema recibido era NULL antes de crear un nuevo nodo.

Sin embargo, identifiqué que el laboratorio no comprueba directamente qué sucede cuando se intenta agregar un lexema NULL ni cuando la memoria dinámica no está disponible. También es importante revisar que los lexemas no excedan el tamaño del arreglo destinado a almacenarlos.

Para evitar problemas, la implementación utiliza strncpy() y coloca manualmente el carácter '\0' al final del arreglo. También comprueba si malloc() devuelve NULL antes de continuar.

Después de ejecutar las pruebas, comprobé que tanto mi implementación manual como la generada por la IA obtuvieron los mismos resultados: seis de seis pruebas superadas.

Esta actividad me ayudó a comprender mejor el funcionamiento de las listas enlazadas, el uso de punteros y la importancia de liberar correctamente la memoria dinámica.
> 

---
**Firma digital:** Luis Enrique Noh Poot

# Bitácora de Requerimientos y Auditoría IA — Sesión 13
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Manejo de Ámbitos (Scope Global/Local)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Luis Enrique Noh Poot  
**Matrícula:** 24531400  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Pila de tablas de símbolos (Stack Scopes)  
Lo que tuve que cuidar fue no mezclar las responsabilidades de las dos funciones. declarar() debe revisar únicamente el ámbito que recibe. No debe subir al padre para impedir una declaración, porque el lenguaje permite que un bloque interno vuelva a declarar un nombre y oculte al externo.

En cambio, buscar() sí debe recorrer desde el ámbito actual hacia sus padres. El primer ámbito que contenga el nombre es el que debe devolverse. Esto es lo que permite resolver el shadowing.

También consideré los argumentos inválidos, los nombres vacíos, las declaraciones repetidas y el límite de ocho nombres por ámbito.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Ocultamiento de variables (Shadowing) |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como un desarrollador de compiladores con experiencia
en C y tablas de símbolos.

Implementa las funciones siguientes respetando exactamente
api_sesion13.h:

int declarar(Ambito *ambito, const char *nombre);
const char *buscar(const Ambito *ambito, const char *nombre);

Reglas para declarar:
- Solo revisar el ámbito recibido; no revisar sus padres.
- Devolver 1 cuando el nombre se inserta.
- Devolver 0 si ya existe en ese mismo ámbito.
- Devolver -1 si el ámbito es NULL, el nombre es NULL
  o vacío, o el ámbito alcanzó AMBITO_MAX.
- No exceder el arreglo nombres.

Reglas para buscar:
- Comenzar en el ámbito recibido.
- Comparar los nombres locales del ámbito actual.
- Si no aparece, continuar por el campo padre.
- Devolver el campo nombre del ámbito más cercano
  donde se encontró el identificador.
- Devolver NULL cuando el nombre no existe o el
  argumento nombre es inválido.

No incluir main(). No cambiar las firmas ni la estructura.
El código debe compilar con:
gcc -std=c11 -Wall -Wextra -Werror
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion13.h"
#include <string.h>

static int existe_local(const Ambito *ambito, const char *nombre) {
    int posicion = 0;

    while (posicion < ambito->total) {
        const char *registrado = ambito->nombres[posicion];

        if (registrado != NULL && strcmp(registrado, nombre) == 0) {
            return 1;
        }

        posicion++;
    }

    return 0;
}

int declarar(Ambito *ambito, const char *nombre) {
    if (ambito == NULL || nombre == NULL || *nombre == '\0') {
        return -1;
    }

    if (ambito->total < 0 || ambito->total > AMBITO_MAX) {
        return -1;
    }

    /* La declaracion se limita a este nivel: no consulta al padre. */
    if (existe_local(ambito, nombre)) {
        return 0;
    }

    if (ambito->total >= AMBITO_MAX) {
        return -1;
    }

    ambito->nombres[ambito->total] = nombre;
    ++ambito->total;
    return 1;
}

const char *buscar(const Ambito *ambito, const char *nombre) {
    const Ambito *nivel = ambito;

    if (nombre == NULL || *nombre == '\0') {
        return NULL;
    }

    while (nivel != NULL) {
        if (nivel->total < 0 || nivel->total > AMBITO_MAX) {
            return NULL;
        }

        if (existe_local(nivel, nombre)) {
            return nivel->nombre;
        }

        nivel = nivel->padre;
    }

    return NULL;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Búsqueda jerárquica de identificadores

### Salida del harness
```text
=== AUDITORIA SESION 13 - Pila de ambitos y busqueda jerarquica ===
Caso                               esperado:              obtenido:              resultado
[ 1] declarar(&g, "x")                  esperado: 1            obtenido: 1            OK
[ 2] buscar en el mismo ambito          esperado: global       obtenido: global       OK
[ 3] buscar subiendo al padre           esperado: global       obtenido: global       OK
[ 4] shadowing en el hijo               esperado: bloque       obtenido: bloque       OK
[ 5] tres niveles hasta la raiz         esperado: raiz         obtenido: raiz         OK
[ 6] nombre no declarado                esperado: (nulo)       obtenido: (nulo)       OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion13.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
Al revisar la propuesta de IA identifiqué un detalle importante en el contrato: declarar y buscar no deben recorrer la jerarquía de la misma manera. Si declarar() consultara también al padre, no permitiría que un bloque interno definiera una variable con el mismo nombre que una variable global. Eso impediría el comportamiento de ocultamiento que precisamente se quería representar.

Por eso mantuve la declaración limitada al ámbito actual y reservé el recorrido por padre para buscar(). También agregué comprobaciones de capacidad y de argumentos vacíos o nulos. Las pruebas complementarias ayudaron a verificar esas condiciones que no aparecen entre los seis casos oficiales.

Hay una limitación de la estructura que conviene recordar: el arreglo almacena punteros const char *, no copias de los textos. Por eso, el texto entregado a declarar() debe seguir existiendo mientras el ámbito lo utilice. El laboratorio no incluye una prueba sobre la vida útil de esas cadenas; no afirmo que ese caso haya sido cubierto por el harness.
> 

---
**Firma digital:**  Luis Enrique Noh Poot

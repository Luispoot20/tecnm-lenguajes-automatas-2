# Bitácora de Requerimientos y Auditoría IA — Sesión 04
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Anatomía de un Árbol Sintáctico (AST)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Luis Enrique Noh Poot  
**Matrícula:** 24531402 
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Estructura de nodo binario (ASTNode)  
En esta parte de la actividad trabajé con una estructura diferente a las listas enlazadas de las sesiones anteriores. El objetivo fue construir un árbol donde cada elemento pudiera representar un número o una operación matemática.

Lo que más trabajo me costó fue entender cómo relacionar los nodos sin confundir el hijo izquierdo con el derecho. También tuve que revisar qué información debía guardar cada nodo, ya que no todos representan lo mismo.

Para organizar la estructura utilicé un campo para identificar el tipo de nodo, otro para guardar el valor numérico y un carácter para almacenar el operador. Además, agregué dos punteros para conectar los nodos.

Una vez construida la estructura, utilicé recursividad para recorrer el árbol y obtener el resultado de las operaciones.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Gramática E -> E + T | T con tipos de nodo |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Quiero implementar un árbol sintáctico abstracto (AST)
en lenguaje C para representar operaciones matemáticas.

Necesito que desarrolles estas tres funciones:

ASTNode *crear_numero(int v);
ASTNode *crear_binario(char op, ASTNode *izq, ASTNode *der);
int evaluar(ASTNode *n);

Considera las siguientes condiciones:

1. Los nodos deben reservarse con malloc.
2. Un nodo numérico debe guardar su valor.
3. Un nodo binario debe guardar el operador y sus dos hijos.
4. Los operadores permitidos son +, - y *.
5. No permitir operadores diferentes.
6. Comprobar que los hijos no sean NULL.
7. Verificar que malloc haya reservado memoria correctamente.
8. Inicializar todos los campos de cada nodo.
9. La evaluación debe realizarse mediante recursividad.
10. Si evaluar recibe NULL debe devolver -1.
11. Mantener las firmas originales de api_sesion04.h.
12. No incluir main ni modificar la estructura establecida.

El código debe ser compatible con GCC y evitar
advertencias durante la compilación.

Explica cómo se construye el árbol y cómo se obtiene
el resultado de una expresión matemática.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion04.h"
#include <stdlib.h>

ASTNode *crear_numero(int v) {
    ASTNode *n = malloc(sizeof(ASTNode));

    if (n == NULL) {
        return NULL;
    }

    n->tipo = NODO_NUM;
    n->valor = v;
    n->op = '\0';
    n->izq = NULL;
    n->der = NULL;

    return n;
}

ASTNode *crear_binario(char op, ASTNode *izq, ASTNode *der) {
    if (izq == NULL || der == NULL) {
        return NULL;
    }

    if (op != '+' && op != '-' && op != '*') {
        return NULL;
    }

    ASTNode *n = malloc(sizeof(ASTNode));

    if (n == NULL) {
        return NULL;
    }

    n->tipo = NODO_BINOP;
    n->valor = 0;
    n->op = op;
    n->izq = izq;
    n->der = der;

    return n;
}

int evaluar(ASTNode *n) {
    if (n == NULL) {
        return -1;
    }

    if (n->tipo == NODO_NUM) {
        return n->valor;
    }

    if (n->tipo != NODO_BINOP ||
        n->izq == NULL || n->der == NULL) {
        return -1;
    }

    int izq = evaluar(n->izq);
    int der = evaluar(n->der);

    switch (n->op) {
        case '+':
            return izq + der;
        case '-':
            return izq - der;
        case '*':
            return izq * der;
        default:
            return -1;
    }
}


```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Árbol vacío y evaluación de nodo raíz

### Salida del harness
```text
=== AUDITORIA SESION 04 - Nodo binario de AST ===
Caso                               esperado:              obtenido:              resultado
[ 1] evaluar(nodo 42)                   esperado: 42           obtenido: 42           OK
[ 2] evaluar(3+5*2)                     esperado: 13           obtenido: 13           OK
[ 3] hijo izquierdo de (* 5 2)          esperado: 5            obtenido: 5            OK
[ 4] operador guardado                  esperado: *            obtenido: *            OK
[ 5] evaluar(NULL)                      esperado: -1           obtenido: -1           OK
[ 6] evaluar(10-4)                      esperado: 6            obtenido: 6            OK

RESULTADO: 6/6 pruebas superadas
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
Al comparar las dos implementaciones encontré que prácticamente resolvían el ejercicio de la misma manera. Por esa razón, decidí revisar no solamente los resultados del programa, sino también la manera en que se administra la memoria.

Encontré un detalle importante: los nodos se crean mediante malloc(), pero el contrato de la actividad no contempla una función para destruir el árbol y liberar todos sus nodos. Aunque las pruebas terminan correctamente, esto significa que el laboratorio no demuestra que la memoria reservada se libere.

No agregué una cuarta función porque el ejercicio exige respetar exactamente las tres firmas establecidas en api_sesion04.h. Considero que una mejora para otra versión sería implementar una función que recorra el árbol en postorden y libere cada nodo con free().

También observé que el código comprueba los punteros nulos y restringe los operadores permitidos, lo que ayuda a evitar construcciones incorrectas.
> 

---
**Firma digital:** Luis Enrique Noh Poot

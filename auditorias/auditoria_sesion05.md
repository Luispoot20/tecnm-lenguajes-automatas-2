# Bitácora de Requerimientos y Auditoría IA — Sesión 05
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Recorridos del AST y Notación Polaca  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Luis Enrique Noh Poot  
**Matrícula:** 24531400  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Impresión en Postorden / RPN  
Durante esta actividad trabajé con un árbol sintáctico que ya no solamente debía almacenar operaciones, sino también recorrer sus elementos para convertirlos en una expresión que pudiera utilizar una máquina de pila.

Mi primera dificultad fue entender por qué los operadores tenían que escribirse después de sus operandos. Al principio estaba acostumbrado a representar las operaciones de manera normal, por ejemplo, 3 + 5. Sin embargo, en esta práctica tenía que obtener 3 5 +.

Para conseguirlo organicé el recorrido en tres pasos: primero visitar el hijo izquierdo, después el derecho y finalmente el nodo principal. También tuve que cuidar la escritura dentro del buffer, porque la función no debía imprimir directamente en pantalla ni sobrepasar el tamaño de memoria disponible.

Esta parte me ayudó a relacionar los árboles sintácticos con la manera en que se representan las operaciones dentro de un compilador.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Invariantes de recorrido recursivo |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como un programador especializado en C y
desarrollo de compiladores.

Necesito implementar la función postorden para
transformar un árbol AST en una expresión RPN.

Firma obligatoria:

void postorden(ASTNode *n, char *buf, int cap);

Condiciones:

- Recorrer primero el hijo izquierdo.
- Después recorrer el hijo derecho.
- Finalmente agregar el operador de la raíz.
- Cuando el nodo sea numérico, convertir su valor
  a texto y agregarlo al buffer.
- Separar los elementos utilizando un solo espacio.
- No dejar espacios al final.
- Si el árbol es NULL, producir una cadena vacía.
- Comprobar que buf no sea NULL y que cap sea mayor
  que cero.
- No escribir más de cap-1 caracteres.
- Asegurar que el buffer termine con '\0'.
- Utilizar recursividad.
- No utilizar printf ni crear una función main.
- Respetar las estructuras y firmas del encabezado
  api_sesion05.h.

Entrega una solución modular y explica los casos
en los que puede presentarse un problema de memoria
o de profundidad durante el recorrido.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion05.h"
#include <stdio.h>

static void agregar_token(char *buf, int cap, const char *token) {
    int usados = 0;

    while (usados < cap - 1 && buf[usados] != '\0') {
        usados++;
    }

    if (usados >= cap - 1) {
        return;
    }

    int i = 0;

    if (usados > 0) {
        buf[usados++] = ' ';
    }

    while (token[i] != '\0' && usados < cap - 1) {
        buf[usados++] = token[i++];
    }

    buf[usados] = '\0';
}

static void recorrer(ASTNode *n, char *buf, int cap) {
    if (n == NULL || cap <= 0) {
        return;
    }

    if (n->tipo == NODO_NUM) {
        char token[32];
        snprintf(token, sizeof(token), "%d", n->valor);
        agregar_token(buf, cap, token);
        return;
    }

    recorrer(n->izq, buf, cap);
    recorrer(n->der, buf, cap);

    if (n->tipo == NODO_BINOP) {
        char token[2] = {n->op, '\0'};
        agregar_token(buf, cap, token);
    }
}

void postorden(ASTNode *n, char *buf, int cap) {
    if (buf == NULL || cap <= 0) {
        return;
    }

    buf[0] = '\0';
    recorrer(n, buf, cap);
}


```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Validación contra desbordamiento de pila

### Salida del harness
```text
=== AUDITORIA SESION 05 - Recorrido postorden y RPN ===
Caso                               esperado:              obtenido:              resultado
[ 1] hoja 42                            esperado: 42           obtenido: 42           OK
[ 2] 3+(5*2)                            esperado: 3 5 2 * +    obtenido: 3 5 2 * +    OK
[ 3] arbol NULL                         esperado:              obtenido:              OK
[ 4] (1+2)*(3+4)
     esperado: 1 2 + 3 4 + *
     obtenido: 1 2 + 3 4 + *
     -> OK
[ 5] 8-3                                esperado: 8 3 -        obtenido: 8 3 -        OK

RESULTADO: 5/5 pruebas superadas
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
Al revisar el trabajo de la IA me llamó la atención que la función podía recorrer correctamente el árbol, pero dependía de la capacidad del buffer para almacenar todos los elementos.

Encontré un caso que no estaba contemplado en las pruebas: utilizar un buffer demasiado pequeño. Por ejemplo, cuando queda espacio únicamente para un separador, la función puede escribir ese espacio sin alcanzar a colocar el siguiente token. Esto podría provocar que el resultado terminara con un espacio innecesario.

La solución que identifiqué consiste en comprobar previamente si existe espacio suficiente para agregar el separador, el token completo y el terminador '\0'. De esa manera se evita dejar una expresión incompleta.

También observé que la implementación utiliza recursividad. Esto funciona correctamente con los árboles de las pruebas, pero un árbol demasiado profundo podría consumir mucha memoria de pila. Para comprobar ese comportamiento sería necesario realizar pruebas adicionales con árboles de mayor profundidad.

Estos dos escenarios no fueron evaluados por el harness proporcionado, por lo que los resultados de 5/5 corresponden únicamente a los casos que sí se ejecutaron.
> 

---
**Firma digital:** Luis Enrique Noh Poot

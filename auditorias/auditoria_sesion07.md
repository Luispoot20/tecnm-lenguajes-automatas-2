# Bitácora de Requerimientos y Auditoría IA — Sesión 07
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Evaluación de Expresiones Aritméticas  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Luis Enrique Noh Poot  
**Matrícula:** 24531400  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Evaluador recursivo de operaciones  
En este ejercicio trabajé con el árbol sintáctico de las sesiones anteriores, pero ahora el objetivo fue hacer que el programa pudiera resolver las operaciones que contiene.

Lo primero que intenté fue identificar cómo obtener el resultado de una operación utilizando sus dos hijos. Después comprendí que debía evaluar primero las ramas del árbol y utilizar sus resultados para ejecutar la operación que se encuentra en la raíz.

Una dificultad importante fue controlar las divisiones y los módulos entre cero. Si el programa realiza esas operaciones sin revisar el divisor, puede producir un error durante la ejecución.

Para evitarlo agregué una comprobación antes de realizar cada operación. También establecí dos códigos de error: el número 1 para división o módulo entre cero y el número 2 para nodos inválidos.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Manejo estricto de división entre cero |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como un ingeniero de software especializado
en programación de compiladores utilizando C.

Necesito desarrollar un evaluador recursivo de
expresiones aritméticas representadas mediante AST.

La interfaz obligatoria es:

int evaluar(ASTNode *n, int *error);

REQUISITOS:

1. Respetar exactamente la firma de la función.
2. Si el nodo es NODO_NUM, devolver su valor.
3. Si el nodo es NODO_BINOP, evaluar sus dos hijos.
4. Permitir las operaciones:
   - Suma (+)
   - Resta (-)
   - Multiplicación (*)
   - División (/)
   - Módulo (%)
5. Detectar división entre cero y establecer error=1.
6. Detectar módulo entre cero y establecer error=1.
7. Si el árbol o algún nodo es inválido,
   establecer error=2.
8. Nunca ejecutar división o módulo antes
   de comprobar el divisor.
9. Si un hijo genera un error, detener la evaluación
   y propagar ese error hacia los padres.
10. Comprobar los punteros antes de utilizarlos.
11. Inicializar el código de error al comenzar.
12. No utilizar memoria dinámica.
13. No incluir una función main.
14. Mantener compatibilidad con GCC y evitar
    advertencias con -Wall -Wextra -Werror.

Entrega el código completo y explica el recorrido
recursivo, los errores y la complejidad.
```

---

## 3. Código Devuelto por el Modelo
```c
#include <stddef.h>
#include "api_sesion07.h"

static int evaluar_nodo(ASTNode *n, int *error) {

    int valor_izq;
    int valor_der;

    if (n == NULL) {
        *error = 2;
        return 0;
    }

    switch (n->tipo) {

        case NODO_NUM:
            return n->valor;

        case NODO_BINOP:

            if (n->izq == NULL || n->der == NULL) {
                *error = 2;
                return 0;
            }

            valor_izq = evaluar_nodo(n->izq, error);

            if (*error != 0) {
                return 0;
            }

            valor_der = evaluar_nodo(n->der, error);

            if (*error != 0) {
                return 0;
            }

            switch (n->op) {

                case '+':
                    return valor_izq + valor_der;

                case '-':
                    return valor_izq - valor_der;

                case '*':
                    return valor_izq * valor_der;

                case '/':
                    if (valor_der == 0) {
                        *error = 1;
                        return 0;
                    }
                    return valor_izq / valor_der;

                case '%':
                    if (valor_der == 0) {
                        *error = 1;
                        return 0;
                    }
                    return valor_izq % valor_der;

                default:
                    *error = 2;
                    return 0;
            }

        default:
            *error = 2;
            return 0;
    }
}

int evaluar(ASTNode *n, int *error) {

    if (error == NULL) {
        return 0;
    }

    *error = 0;

    return evaluar_nodo(n, error);
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Expresiones complejas y división por 0

### Salida del harness
```text
=== AUDITORIA SESION 07 - Evaluador recursivo con deteccion de errores ===
Caso                               esperado:              obtenido:              resultado
[ 1] (2+3)*4                            esperado: 20           obtenido: 20           OK
[ 2] 7/0 -> error 1                     esperado: 1            obtenido: 1            OK
[ 3] 7%0 -> error 1                     esperado: 1            obtenido: 1            OK
[ 4] arbol NULL -> error 2              esperado: 2            obtenido: 2            OK
[ 5] 10-2-3                             esperado: 5            obtenido: 5            OK
[ 6] (8/2)+1                            esperado: 5            obtenido: 5            OK

RESULTADO: 6/6 pruebas superadas
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
Al revisar las implementaciones me di cuenta de que el harness evaluaba las operaciones principales, pero no incluía todos los posibles errores que podrían aparecer durante el recorrido.

Uno de los casos que consideré importante fue un nodo de operación que no tuviera alguno de sus dos hijos. Si el programa intenta acceder directamente a ese puntero, podría producir un fallo durante la ejecución.

Para evitarlo incorporé una comprobación que verifica que los dos hijos existan antes de continuar con la evaluación.

También revisé el comportamiento cuando una operación interna genera un error. La función no debe continuar calculando el nodo padre utilizando un resultado incorrecto. Por eso agregué comprobaciones después de evaluar cada hijo y hago que el error se propague hacia arriba.

Aunque estos dos escenarios adicionales no forman parte de los seis casos originales del harness, las implementaciones incluyen las protecciones correspondientes.

Otro aspecto que queda pendiente para una versión más completa es controlar posibles desbordamientos de enteros en operaciones como multiplicaciones de valores demasiado grandes. Este comportamiento no está contemplado en las pruebas proporcionadas.
> 

---
**Firma digital:** Luis Enrique Noh Poot

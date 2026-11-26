# Bitácora de Requerimientos y Auditoría IA — Sesión 10
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Hito 1: Mini-Evaluador de Expresiones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Luis Enrique Noh Poot  
**Matrícula:** 24531402  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Integración AST y Tipado Estático  
En esta sesión integré varias partes que había trabajado por separado. El programa ya no recibe un árbol construido previamente: ahora tiene que leer una expresión escrita como texto y calcular su resultado.

La dificultad principal fue organizar el análisis para que las operaciones se resolvieran en el orden correcto. Por ejemplo, en 2+3*4 no se debe sumar primero; la multiplicación tiene prioridad. Para manejarlo separé el trabajo en tres niveles:
- primaria() reconoce un número o una expresión encerrada entre paréntesis.
- termino() procesa las multiplicaciones y divisiones.
- expr() procesa las sumas y restas.
También tuve que controlar dos situaciones distintas. La división entre cero se reporta con el código 1, mientras que una expresión incompleta o con sintaxis incorrecta se reporta con el código 2. La función pública evaluar_expresion() prepara el analizador, inicia el recorrido y revisa que no queden caracteres sin procesar.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Contrato formal de Frontend Semántico |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
Actúa como un desarrollador especializado en compiladores y analizadores
sintácticos escritos en C.

Implementa la función:
long evaluar_expresion(const char *fuente, int *error);

Respeta exactamente la firma de api_sesion10.h. El analizador debe leer
expresiones con números enteros, suma (+), resta (-), multiplicación (*),
división (/), paréntesis y espacios.

Organiza la solución mediante descenso recursivo:
- primaria: reconoce un entero o una expresión entre paréntesis.
- termino: procesa multiplicación y división.
- expr: procesa suma y resta.

La multiplicación y la división deben tener prioridad sobre la suma y la
resta. Los paréntesis deben modificar el orden de evaluación.

Utiliza error=1 para división entre cero y error=2 para sintaxis inválida.
También valida una fuente NULL, detecta caracteres sobrantes y devuelve
cero cuando se presenta un error.

No agregues main(), no cambies la firma pública y evita advertencias
con gcc -std=c11 -Wall -Wextra -Werror.

Entrega código completo y modular, compatible con el harness proporcionado.
```text

```

---

## 3. Código Devuelto por el Modelo
/* ia_sesion10_c.c - Fase 2: propuesta de IA
 * Analizador descendente para expresiones aritmeticas.
 */
#include "api_sesion10.h"
#include <stddef.h>
#include <ctype.h>

typedef struct {
    const char *actual;
    int codigo_error;
} Cursor;

static void omitir_espacios(Cursor *c) {
    while (*c->actual == ' ') {
        ++c->actual;
    }
}

static long analizar_suma(Cursor *c);

static long analizar_primario(Cursor *c) {
    long numero = 0;

    omitir_espacios(c);

    if (c->codigo_error != 0) {
        return 0;
    }

    if (*c->actual == '(') {
        long interno;
        ++c->actual;
        interno = analizar_suma(c);
        omitir_espacios(c);

        if (c->codigo_error == 0) {
            if (*c->actual != ')') {
                c->codigo_error = 2;
            } else {
                ++c->actual;
            }
        }
        return interno;
    }

    if (!isdigit((unsigned char)*c->actual)) {
        c->codigo_error = 2;
        return 0;
    }

    do {
        numero = numero * 10L + (long)(*c->actual - '0');
        ++c->actual;
    } while (isdigit((unsigned char)*c->actual));

    return numero;
}

static long analizar_producto(Cursor *c) {
    long acumulado = analizar_primario(c);

    for (;;) {
        char signo;
        long operando;

        if (c->codigo_error != 0) {
            return 0;
        }

        omitir_espacios(c);
        signo = *c->actual;

        if (signo != '*' && signo != '/') {
            return acumulado;
        }

        ++c->actual;
        operando = analizar_primario(c);

        if (c->codigo_error != 0) {
            return 0;
        }

        if (signo == '/') {
            if (operando == 0) {
                c->codigo_error = 1;
                return 0;
            }
            acumulado /= operando;
        } else {
            acumulado *= operando;
        }
    }
}

static long analizar_suma(Cursor *c) {
    long acumulado = analizar_producto(c);

    for (;;) {
        char signo;
        long operando;

        if (c->codigo_error != 0) {
            return 0;
        }

        omitir_espacios(c);
        signo = *c->actual;

        if (signo != '+' && signo != '-') {
            return acumulado;
        }

        ++c->actual;
        operando = analizar_producto(c);

        if (c->codigo_error != 0) {
            return 0;
        }

        if (signo == '+') {
            acumulado += operando;
        } else {
            acumulado -= operando;
        }
    }
}

long evaluar_expresion(const char *fuente, int *error) {
    static int error_respaldo;
    Cursor c;
    long valor;

    if (error == NULL) {
        error = &error_respaldo;
    }
    *error = 0;

    if (fuente == NULL) {
        *error = 2;
        return 0;
    }

    c.actual = fuente;
    c.codigo_error = 0;

    valor = analizar_suma(&c);
    omitir_espacios(&c);

    if (c.codigo_error == 0 && *c.actual != '\0') {
        c.codigo_error = 2;
    }

    *error = c.codigo_error;
    return (c.codigo_error == 0) ? valor : 0;
}
```c

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Precedencia, paréntesis, división entre cero y sintaxis inválida

### Salida del harness
```text
=== AUDITORIA SESION 10 - Mini-evaluador de expresiones (Hito 1) ===
Caso                               esperado:              obtenido:              resultado
[ 1] "2+3*4"                            esperado: 14           obtenido: 14           OK
[ 2] "(2+3)*4"                          esperado: 20           obtenido: 20           OK
[ 3] "10/0" -> error 1                  esperado: 1            obtenido: 1            OK
[ 4] "2+" -> error 2                    esperado: 2            obtenido: 2            OK
[ 5] "  7 "                             esperado: 7            obtenido: 7            OK
[ 6] "5-10"                             esperado: -5           obtenido: -5           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion10.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
Al revisar la solución generada por la IA comprobé que los seis casos del laboratorio se resolvían correctamente. Sin embargo, el harness no prueba todas las formas de entrada que puede recibir el analizador.

Un caso adicional que identifiqué es el uso de tabulaciones. La función que omite espacios revisa el carácter ' '; por lo tanto, una entrada como \t7 no se procesa como el número 7, sino que se marca como sintaxis inválida. El enunciado menciona espacios, así que esto no afecta los seis casos solicitados, pero sería conveniente ampliar la función para reconocer otros espacios en blanco mediante isspace().

También conviene agregar pruebas para paréntesis sin cerrar, caracteres sobrantes y expresiones con varios niveles de agrupación. Estas entradas no aparecen en los seis casos oficiales, por lo que no afirmo que hayan sido verificadas por el harness.
> 

---
**Firma digital:** Luis Enrique Noh Poot

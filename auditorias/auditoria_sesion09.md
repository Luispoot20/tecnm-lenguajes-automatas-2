# Bitácora de Requerimientos y Auditoría IA — Sesión 09
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Reporte Formal de Errores Semánticos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Luis Enrique Noh Poot  
**Matrícula:** 24531400  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Diagnóstico con línea y columna  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Estructura estándar de errores GCC |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como un ingeniero de software con experiencia en C
 y en el desarrollo de compiladores.

Implementa exactamente esta función:

void reportar(const char *archivo, int linea, int columna,
              const char *mensaje, char *salida, int cap);

Requisitos:
1. Construir el diagnóstico con este formato exacto:
   archivo:linea:columna: error: mensaje
2. No agregar espacios extra ni punto final.
3. Escribir el resultado en el buffer recibido; no reservar
   memoria dinámica.
4. Nunca escribir más de cap-1 caracteres.
5. Terminar siempre el texto escrito con '\0' cuando
   salida no sea NULL y cap sea mayor que cero.
6. Si salida es NULL o cap no es positivo, no escribir.
7. Evitar pasar punteros NULL a los especificadores %s.
8. Utilizar snprintf para controlar la capacidad del buffer.
9. Respetar la firma declarada en api_sesion09.h.
10. No incluir main().
11. El código debe compilar con GCC usando
    -std=c11 -Wall -Wextra -Werror.

Entrega únicamente la implementación y una explicación
breve de cómo se controla el truncamiento.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion09.h"
#include <stdio.h>

void reportar(const char *archivo, int linea, int columna,
              const char *mensaje, char *salida, int cap) {
    int escritos;

    if (salida == NULL || cap < 1) {
        return;
    }

    escritos = snprintf(salida, (size_t)cap,
                        "%s:%d:%d: error: %s",
                        archivo != NULL ? archivo : "",
                        linea,
                        columna,
                        mensaje != NULL ? mensaje : "");

    /* Si la biblioteca informa un error, devolver una cadena vacia. */
    if (escritos < 0) {
        salida[0] = '\0';
    } else {
        /* Refuerza la terminacion incluso cuando el texto fue truncado. */
        salida[cap - 1] = '\0';
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Múltiples errores sin abortar parser

### Salida del harness
```text
=== AUDITORIA SESION 09 - Diagnostico con linea y columna ===
Caso                               esperado:              obtenido:              resultado
[ 1] main.c:14:5
     esperado: main.c:14:5: error: tipo incompatible
     obtenido: main.c:14:5: error: tipo incompatible
     -> OK
[ 2] a.c:1:1
     esperado: a.c:1:1: error: x
     obtenido: a.c:1:1: error: x
     -> OK
[ 3] linea 0
     esperado: x.c:0:0: error: y
     obtenido: x.c:0:0: error: y
     -> OK
[ 4] cap=10 trunca                      esperado: main.c:14    obtenido: main.c:14    OK

RESULTADO: 4/4 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion09.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
Al revisar el contrato y los casos del harness noté que la prueba del buffer de 10 posiciones es especialmente importante. Aunque el diagnóstico completo sea más largo, la función debe guardar únicamente los primeros nueve caracteres y reservar la última posición para \0. Por eso el resultado esperado es main.c:14 y no la cadena completa.

El harness no incluye pruebas para salida == NULL, cap == 0 ni para punteros nulos en archivo o mensaje. Para cubrir esos escenarios añadí comprobaciones en ambas implementaciones: si el buffer no puede utilizarse, la función regresa sin escribir; si el archivo o el mensaje son nulos, se sustituyen por cadenas vacías antes de llamar a snprintf.

Esos casos adicionales están protegidos por el código, pero no forman parte de los cuatro resultados del harness original. Por ello no los presento como pruebas aprobadas. Una ampliación útil del laboratorio sería agregar casos específicos para esas entradas y verificar también el comportamiento con cap == 1.

Complejidad: el formateo recorre los caracteres necesarios para producir el mensaje, por lo que su tiempo depende de la longitud del texto y está acotado por la capacidad del buffer. El espacio auxiliar es O(1), porque no se reserva memoria dinámica y se utiliza directamente el buffer recibido.
> 

---
**Firma digital:** Luis Enrique Noh Poot

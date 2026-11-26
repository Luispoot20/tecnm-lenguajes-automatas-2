# Bitácora de Requerimientos y Auditoría IA — Sesión 14
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Declaración vs Uso de Variables  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Luis Enrique Noh Poot  
**Matrícula:** 24531400  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Detección de variables no declaradas  
En esta práctica trabajé con una tabla de símbolos sencilla. La idea central fue separar dos acciones que parecen parecidas, pero tienen resultados distintos: declarar un nombre y comprobar si puede usarse.

Al implementar declarar() tuve que cuidar el orden de las comprobaciones. Primero revisé que la tabla y el nombre existieran y que el texto no estuviera vacío. Después recorrí los nombres que ya estaban guardados. Si encontraba uno igual, debía rechazar la nueva declaración sin aumentar el contador. Solo cuando no había coincidencia podía guardar el nombre y avanzar total.

Para usar() la regla es más directa: recorrer lo declarado y responder que sí únicamente cuando hay una coincidencia completa. La prueba con x y xy me hizo poner atención en no comparar solo los primeros caracteres.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Invariante de definición previa |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como un programador de C que implementa una tabla de símbolos.
Usa api_sesion14.h sin cambiar la estructura Tabla ni las firmas:
int declarar(Tabla *tabla, const char *nombre);
int usar(const Tabla *tabla, const char *nombre);

Reglas:
- declarar devuelve 1 si inserta, 0 si ya existe y -1 si la tabla,
  el nombre o la cadena son inválidos.
- usar devuelve 1 solo si el nombre completo ya está declarado;
  en cualquier otro caso devuelve 0.
- La comparación debe ser exacta: "x" y "xy" son distintos.
- No permitir que total rebase TABLA_MAX.
- No modificar total cuando se rechaza un duplicado.
- No incluir main y compilar con -std=c11 -Wall -Wextra -Werror.
Entrega únicamente las dos funciones y las auxiliares necesarias.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion14.h"
#include <string.h>

int declarar(Tabla *tabla, const char *nombre)
{
    int i;

    if (tabla == NULL || nombre == NULL || nombre[0] == '\0') {
        return -1;
    }

    if (tabla->total < 0 || tabla->total > TABLA_MAX) {
        return -1;
    }

    for (i = 0; i < tabla->total; ++i) {
        if (tabla->nombres[i] != NULL &&
            strcmp(tabla->nombres[i], nombre) == 0) {
            return 0;
        }
    }

    if (tabla->total >= TABLA_MAX) {
        return -1;
    }

    tabla->nombres[tabla->total] = nombre;
    ++tabla->total;
    return 1;
}

int usar(const Tabla *tabla, const char *nombre)
{
    int i;

    if (tabla == NULL || nombre == NULL || nombre[0] == '\0') {
        return 0;
    }

    if (tabla->total < 0 || tabla->total > TABLA_MAX) {
        return 0;
    }

    for (i = 0; i < tabla->total; ++i) {
        if (tabla->nombres[i] != NULL &&
            strcmp(tabla->nombres[i], nombre) == 0) {
            return 1;
        }
    }

    return 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Uso antes de declarar y redeclaración

### Salida del harness
```text
=== AUDITORIA SESION 14 - Declaracion previa y redeclaracion ===
Caso                               esperado:              obtenido:              resultado
[ 1] usar sin declarar                  esperado: 0            obtenido: 0            OK
[ 2] usar despues de declarar           esperado: 1            obtenido: 1            OK
[ 3] redeclarar el mismo nombre         esperado: 0            obtenido: 0            OK
[ 4] total tras redeclarar              esperado: 1            obtenido: 1            OK
[ 5] prefijo no es el mismo nombre      esperado: 0            obtenido: 0            OK
[ 6] declarar nombre vacio              esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion14.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
El harness oficial confirma los seis requisitos indicados, pero no prueba qué ocurre cuando la tabla alcanza su capacidad máxima ni cuando las funciones reciben punteros nulos. Por eso agregué una comprobación independiente para esos escenarios. Ambas versiones rechazaron la inserción cuando había 16 símbolos, conservaron el contador en 16 y respondieron correctamente a consultas con argumentos nulos.

Un aspecto de diseño que también debe tenerse presente es que Tabla guarda punteros const char *, no copias de las cadenas. Por lo tanto, los textos que se pasan a declarar() deben seguir existiendo mientras estén almacenados en la tabla. La interfaz de esta sesión no ofrece una función para copiar y liberar esos nombres; cambiar esa propiedad requeriría ampliar el contrato, no solo modificar estas dos funciones.

La versión artesanal compara cada carácter mediante una función auxiliar. La propuesta de IA usa strcmp(), que expresa directamente la comparación de cadenas completas. En ambas se evita aumentar total cuando el nombre ya existe.
> 

---
**Firma digital:** Luis Enrique Noh Poot

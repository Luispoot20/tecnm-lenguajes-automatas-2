# Bitácora de Requerimientos y Auditoría IA — Sesión 08
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Inferencia y Promoción de Tipos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** Luis Enrique Noh Poot  
**Matrícula:** 24531400  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Reglas de casteo implícito  
En esta actividad tuve que establecer cómo elegir un tipo común cuando una expresión combina dos tipos diferentes. El laboratorio define el orden CHAR < INT < FLOAT; por eso, al combinar dos operandos válidos, se debe conservar el tipo que tenga el rango más amplio.

La parte que más cuidado necesitó fue tratar TIPO_ERROR por separado. No debía considerarlo simplemente como el tipo de menor tamaño, porque representa una situación inválida y no un dato que pueda participar en una promoción normal.

También tuve que distinguir dos preguntas: promover(a, b) busca el tipo común más amplio para los operandos, mientras que hay_perdida(destino, origen) comprueba si el destino tiene menor rango que el origen. Para la primera versión utilicé condiciones explícitas, en vez de depender solamente de los números del enumerado.
> 

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Promoción entera y ensanchamiento |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Actúa como un programador especializado en C y en el
análisis semántico de compiladores.

Implementa las funciones declaradas en api_sesion08.h:

Tipo promover(Tipo a, Tipo b);
int hay_perdida(Tipo destino, Tipo origen);
const char *nombre_tipo(Tipo t);

El enumerado utiliza estos valores:
TIPO_ERROR = 0
TIPO_CHAR  = 1
TIPO_INT   = 2
TIPO_FLOAT = 3

Reglas:
1. La jerarquía es CHAR < INT < FLOAT.
2. promover(a,b) devuelve el tipo de mayor rango.
3. Si alguno de los argumentos de promover es TIPO_ERROR,
   debe devolver TIPO_ERROR.
4. hay_perdida devuelve 1 cuando el destino es más angosto
   que el origen; en otro caso devuelve 0.
5. TIPO_ERROR y los valores no definidos no deben tratarse
   como tipos válidos para comparar rangos.
6. nombre_tipo devuelve "error", "char", "int" o "float";
   para valores desconocidos debe devolver "desconocido".
7. Respeta exactamente las firmas del encabezado.
8. No agregues main ni modifiques api_sesion08.h.
9. El código debe compilar con:
   gcc -std=c11 -Wall -Wextra -Werror

Entrega las tres funciones completas y explica brevemente
las reglas que utiliza cada una.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion08.h"

Tipo promover(Tipo a, Tipo b) {
    if (a < TIPO_CHAR || a > TIPO_FLOAT ||
        b < TIPO_CHAR || b > TIPO_FLOAT) {
        return TIPO_ERROR;
    }

    return (a >= b) ? a : b;
}

int hay_perdida(Tipo destino, Tipo origen) {
    if (destino < TIPO_CHAR || destino > TIPO_FLOAT ||
        origen < TIPO_CHAR || origen > TIPO_FLOAT) {
        return 0;
    }

    return destino < origen;
}

const char *nombre_tipo(Tipo t) {
    switch (t) {
        case TIPO_ERROR:
            return "error";
        case TIPO_CHAR:
            return "char";
        case TIPO_INT:
            return "int";
        case TIPO_FLOAT:
            return "float";
        default:
            return "desconocido";
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Mezcla de tipos enteros y flotantes

### Salida del harness
```text
=== AUDITORIA SESION 08 - Reglas de promocion y perdida ===
Caso                               esperado:              obtenido:              resultado
[ 1] promover(CHAR,INT)                 esperado: int          obtenido: int          OK
[ 2] promover(INT,FLOAT)                esperado: float        obtenido: float        OK
[ 3] promover(FLOAT,FLOAT)              esperado: float        obtenido: float        OK
[ 4] promover(ERROR,INT)                esperado: error        obtenido: error        OK
[ 5] hay_perdida(CHAR,INT)              esperado: 1            obtenido: 1            OK
[ 6] hay_perdida(INT,CHAR)              esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion08.md
```

* [ ] Caso base y límites superados sin *Crash*.
* [ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
Al revisar la solución de IA, me fijé en que la comparación por números del enumerado solamente es correcta después de comprobar que los valores pertenecen a los tipos reconocidos. Si se compararan directamente sin esa validación, el valor TIPO_ERROR podría confundirse con un tipo válido de menor rango.

La propuesta de IA sí realiza esa validación. Por eso no tuve que cambiar su lógica para superar los seis casos oficiales. Sin embargo, el harness no prueba todas las combinaciones posibles: por ejemplo, no incluye todas las parejas con CHAR ni valores desconocidos del enumerado. Para ampliar la revisión, mantuve las comprobaciones de valores válidos e inválidos en ambas implementaciones.

En hay_perdida, el contrato no asigna una conversión válida a TIPO_ERROR; por esa razón, ambas versiones devuelven 0 cuando reciben un tipo que no se puede comparar. Así se evita clasificar el centinela como una conversión que pierde información.
> 

---
**Firma digital:** Luis Enrique Noh Poot

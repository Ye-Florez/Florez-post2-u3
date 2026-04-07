# Laboratorio: Ensamblado y Ejecución Paso a Paso
## Unidad 3 — Post-Contenido 2 | Arquitectura de Computadores

**Estudiante:** Florez  
**Repositorio:** florez-post2-u3  

## Proposito del Laboratorio
En este laboratorio se trabajó directamente con el depurador DEBUG de DOS,ensamblando dos 
programas en memoria y ejecutándolos instrucción por instrucción usando el comando T. 
El objetivo fue observar con exactitud cómo cambian los registros y las banderas del procesador 
después de cada operación, y entender cómo funciona un bucle controlado por la instrucción LOOP y
el registro CX.

## Checkpoint 1 — Tabla de Traza del Programa de Suma
El primer programa carga tres valores en los registros AX, BX y CX, los suma acumulando el resultado 
en AX, y termina con INT 20.

### Tabla de traza
| Instrucción  | AX   | BX   | CX   | IP siguiente | ZF | CF | SF |
|--------------|------|------|------|--------------|----|----|----|
| MOV AX, 000A | 000A | 0000 | 0000 | 0103         | NZ | NC | PL |
| MOV BX, 0005 | 000A | 0005 | 0000 | 0106         | NZ | NC | PL |
| MOV CX, 0003 | 000A | 0005 | 0003 | 0109         | NZ | NC | PL |
| ADD AX, BX   | 000F | 0005 | 0003 | 010B         | NZ | NC | PL |
| ADD AX, CX   | 0012 | 0005 | 0003 | 010D         | NZ | NC | PL |

### Observaciones 
La primera observación fue que las banderas no se alteran con las instrucciones MOV.
en cambio, ADD sí las actualiza de acuerdo con el resultado. El resultado es el mismo en ambas sumas.
ZF permaneció en NZ porque fue diferente de cero.
Tampoco hubo acarreo en ningún momento, así que CF permaneció en NC. Al finalizar las dos sumas, AX
vale 0x0012, que equivale a 18 en decimal, que es exactamente 10 + 5 + 3.

## Checkpoint 2 — Tabla de Traza del Programa con Bucle LOOP

### Tabla de traza

| # | Instrucción  | AX   | CX   | IP siguiente | ¿LOOP salta? |
|---|--------------|------|------|--------------|--------------|
| 1 | MOV AX, 0000 | 0000 | 0000 | 0103         | —            |
| 2 | MOV CX, 0004 | 0000 | 0004 | 0106         | —            |
| 3 | ADD AX, 0002 | 0002 | 0004 | 0109         | —            |
| 4 | LOOP 0106    | 0002 | 0003 | 0106         |    Sí        |
| 5 | ADD AX, 0002 | 0004 | 0003 | 0109         | —            |
| 6 | LOOP 0106    | 0004 | 0002 | 0106         |    Sí        |
| 7 | ADD AX, 0002 | 0006 | 0002 | 0109         | —            |
| 8 | LOOP 0106    | 0006 | 0001 | 0106         |    Sí        |
| 9 | ADD AX, 0002 | 0008 | 0001 | 0109         | —            |
|10 | LOOP 0106    | 0008 | 0000 | 010B         |    No        |
|11 | INT 20       | 0008 | 0000 | —            | —            |

### Observaciones
Algo que se puede evidenciar del LOOP es que en ningun momento se tuvo que escribir operaciones 
manuales, ni saltos condicionales, ya que la instruccion hace todo por si sola.
En cada iteración CX bajó de 4 hasta llegar a 0, momento en que el salto dejó
de ocurrir y el programa siguió hacia INT 20. El resultado final fue AX = 0x0008,
que es 2 × 4 = 8 en decimal

## Análisis del Código Máquina con D
Con el comando D CS:100 L0C se pudo ver directamente los bytes del programa
de bucle almacenados en memoria.

073F: 0100  B8 00 00   B9 04 00   05 02-00   E2 FB  CD  

Desglose de cada función
| Bytes      | Instrucción   | Tamaño  | Detalle                                     |
|------------|---------------|---------|---------------------------------------------|
| B8 00 00   | MOV AX, 0000  | 3 bytes | B8 = opcode, 00 00 = valor en little-endian |
| B9 04 00   | MOV CX, 0004  | 3 bytes | B9 = opcode, 04 00 = valor en little-endian |
| 05 02 00   | ADD AX, 0002  | 3 bytes | 05 = opcode, 02 00 = valor en little-endian |
| E2 FB      | LOOP 0106     | 2 bytes | E2 = opcode, FB = desplazamiento -5         |
| CD 20      | INT 20        | 2 bytes | CD = opcode, 20 = número de interrupción    |


## Conclusiones
Este laboratorio mostro que el comando T es una herramienta fundamentalpara entender exactamente qué hace el procesador en cada ciclo. 
Ver cómo AX va cambiando valor a valor, y cómo CX baja de 4 a 0 controlando el bucle, hace
que conceptos como registros, banderas y saltos relativos dejen de ser abstractos y se vuelvan completamente concretos.
La instrucción LOOP simplifica mucho la escritura de bucles contados y su codificación en solo 2 bytes demuestra la eficiencia del diseño del 8086.



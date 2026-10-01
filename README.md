# Práctica: Estados de un Proceso en Linux

**Módulo 3: Administración de recursos de un sistema computacional**
**Sesión de referencia:** 02 - Process Description and Control (Part 1)

## Objetivo

Que el alumno relacione el modelo teórico de estados de un proceso (New, Ready, Running, Blocked, Exit) y el Process Control Block (PCB) con la información real que expone un sistema operativo Linux, usando GitHub Codespaces como entorno común para todo el grupo.

## Conocimientos previos

- Modelo de dos estados y modelo de cinco estados de un proceso.
- Elementos del PCB (identificador, estado, prioridad, contador de programa, punteros de memoria, información de contabilidad, etc.).
- Concepto de colas Ready / Blocked y el rol del dispatcher.

## Requisitos

- Cuenta de GitHub con acceso a Codespaces.
- Un repositorio propio o proporcionado por el profesor donde crear el Codespace (no requiere contenido especial, solo sirve como entorno).
- No se necesita instalar nada en la máquina personal del alumno.

## Duración estimada

50 minutos (sesión de laboratorio dentro de clase).

## Instrucciones

### Parte 1: Preparar el entorno

1. Abre tu repositorio en GitHub y crea un Codespace (`Code` → `Codespaces` →
   `Create codespace on main`).
2. Espera a que cargue el entorno y abre una terminal (`` Ctrl+` ``).
3. Verifica que tienes herramientas básicas de monitoreo:
   ```bash
   ps --version
   which top
   ```

### Parte 2: Observar procesos en distintos estados

1. En una terminal, lanza un proceso **CPU-bound** en segundo plano:
   ```bash
   yes > /dev/null &
   ```
2. En la misma terminal, lanza un proceso que simule espera (**I/O/timer-bound**):
   ```bash
   sleep 120 &
   ```
3. Lista los procesos y observa la columna `STAT`:
   ```bash
   ps -eo pid,ppid,stat,pcpu,pmem,cmd | grep -E "yes|sleep"
   ```
4. Registra en tu reporte qué código de estado (`R`, `S`, `D`, `T`, `Z`, etc.) aparece para cada proceso y a qué estado del modelo de cinco estados corresponde cada uno.
5. Opcional: instala y usa `htop` para verlo de forma visual:
   ```bash
   sudo apt-get update && sudo apt-get install -y htop
   htop
   ```

## Referencia: códigos de estado (STAT) en Linux

La columna `STAT` de `ps` (o el campo `State` de `/proc/<PID>/status`) usa estos códigos, que puedes mapear directo al modelo de cinco estados visto en clase:

| Código | Nombre                        | Significado                                                                 | Estado del modelo de 5 estados |
|--------|-------------------------------|------------------------------------------------------------------------------|----------------------------------|
| `R`    | Running / Runnable            | Se ejecuta en el CPU o está listo esperando su turno del scheduler          | Running / Ready                  |
| `S`    | Sleeping (interrumpible)      | Espera un evento (señal, temporizador, teclado) y puede despertarse antes    | Blocked                          |
| `D`    | Uninterruptible sleep         | Espera I/O a disco o red; no puede interrumpirse hasta que termine la I/O    | Blocked (variante estricta)      |
| `T`    | Stopped                       | Pausado explícitamente (`SIGSTOP`, `Ctrl+Z`); no compite por CPU             | Similar a un estado suspendido   |
| `Z`    | Zombie                        | Ya terminó, pero el padre no ha leído su código de salida con `wait()`       | Exit (pendiente de liberar PCB)  |

Puedes ver también variantes como `+` (proceso en foreground de la terminal) o `<`/`N` (prioridad alta/baja, relacionada con `nice`), que no forman parte del modelo teórico pero pueden salir en la salida de `ps`.

### Parte 3: Inspeccionar el PCB real de un proceso

1. Obtén el PID del proceso `sleep` que lanzaste:
   ```bash
   pgrep -f "sleep 120"
   ```
2. Consulta su información de control como la mantiene el kernel:
   ```bash
   cat /proc/<PID>/status
   ```
3. Identifica en la salida al menos cuatro campos que correspondan a elementos del PCB visto en clase (por ejemplo: `Pid`, `State`, `VmRSS`,  `Threads`, `Uid`) y anota con cuál elemento teórico del PCB se relaciona cada uno.

### Parte 4: Terminar procesos y observar el estado Exit

1. Termina el proceso `sleep` antes de que expire su tiempo:
   ```bash
   kill <PID>
   ```
2. Intenta consultarlo de nuevo:
   ```bash
   ps -p <PID>
   cat /proc/<PID>/status
   ```
3. Explica en tu reporte qué observaste y cómo se relaciona con el estado
   "Exit" del modelo de cinco estados.

### Parte 5: Procesos padre e hijo (jerarquía)

1. Crea un archivo `fork_demo.py` con el siguiente contenido:
   ```python
   import os
   import time

   pid = os.fork()

   if pid == 0:
       print(f"Proceso hijo, PID={os.getpid()}, PPID={os.getppid()}")
       time.sleep(30)
   else:
       print(f"Proceso padre, PID={os.getpid()}, hijo creado con PID={pid}")
       os.wait()
   ```
2. Ejecútalo en segundo plano y observa la jerarquía mientras el hijo duerme:
   ```bash
   python3 fork_demo.py &
   ps --forest -eo pid,ppid,stat,cmd
   ```
3. Relaciona lo observado con el proceso de "Process Creation" visto en la presentación (slide 15).

## Entregable

Un reporte breve (PDF) que incluya:

1. Capturas de pantalla de los pasos 2, 3 y 4.



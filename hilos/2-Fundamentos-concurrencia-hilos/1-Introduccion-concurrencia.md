# **1. Introducción a la Concurrencia**

## **Diferencias entre Procesos e Hilos**

Ambos son unidades de ejecución, pero operan en niveles y
contextos diferentes:

| **Característica**   | **Proceso**                                               | **Hilo (Thread)**                                                                             |
| -------------------- | --------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| **Definición**       | Programa en ejecución con su propio espacio de memoria.   | Unidad de ejecución dentro de un proceso, comparte memoria con otros hilos del mismo proceso. |
| **Aislamiento**      | Total: Un proceso no accede a la memoria de otro.         | Parcial: Los hilos comparten memoria (variables, objetos).                                    |
| **Creación y costo** | Costoso en recursos (ej: `fork()` en sistemas Unix).      | Ligero: Menos consumo de CPU y memoria.                                                       |
| **Comunicación**     | Compleja: Usa mecanismos del SO (pipes, sockets, IPC).    | Directa: Acceso compartido a variables y objetos.                                             |
| **Ejecución**        | Gestionado por el SO.                                     | Gestionado por la JVM (en Java).                                                              |
| **Ejemplo en Java**  | Usando `ProcessBuilder` para ejecutar un comando externo. | Creando instancias de `Thread` o implementando `Runnable`.                                    |

**Ejemplo de procesos en Java**:

```java
ProcessBuilder processBuilder = new ProcessBuilder("notepad.exe");
Process process = processBuilder.start(); // Se ejecuta en un proceso separado del SO.
```

**Ejemplo de hilos en Java**:

```java
Runnable task = () -> System.out.println("Hilo ejecutándose: " + Thread.currentThread().getName());
Thread thread = new Thread(task);
thread.start(); // Hilo creado dentro de la JVM.
```

---

## **Ventajas de la Programación Multihilo**

1. **Mejor rendimiento en sistemas multicore**:

   - Ejecutar tareas en paralelo aprovechando múltiples
     núcleos del CPU.
   - Ejemplo: Procesar imágenes o cálculos matemáticos en
     paralelo.

2. **Responsividad en aplicaciones**:

   - Evitar bloqueos en interfaces de usuario (GUI) o
     servidores.
   - Ejemplo: Un servidor web que maneja múltiples
     solicitudes simultáneas.

3. **Uso eficiente de recursos**:

   - Hilos en espera (ej: I/O) no bloquean toda la
     aplicación.
   - Ejemplo: Una app que descarga archivos mientras
     responde a eventos del usuario.

4. **Escalabilidad**:
   - Manejar más tareas concurrentes sin aumentar
     procesos del SO.

---

## **Desafíos de la Programación Multihilo**

1. **Complejidad de sincronización**:

   - **Carreras de datos (Race Conditions)**:

     - Ejemplo: Dos hilos incrementan una variable
       `contador` al mismo tiempo.

     ```java
     public class Contador {
         private int valor = 0;

         public void incrementar() {
             valor++; // Operación no atómica.
         }
     }
     ```

     Si dos hilos llaman a `incrementar()`, el resultado
     final podría ser incorrecto (no 2).

   - **Solución**: Usar `synchronized` o clases atómicas
     (`AtomicInteger`).

2. **Interbloqueos (Deadlocks)**:

   - Dos o más hilos se bloquean mutuamente esperando
     recursos.
   - Ejemplo:
     ```java
     // Hilo 1: Bloquea recursoA y espera recursoB.
     // Hilo 2: Bloquea recursoB y espera recursoA.
     ```

3. **Problemas de visibilidad**:

   - Cambios en variables por un hilo pueden no verse
     inmediatamente en otros (por caching en CPU).
   - **Solución**: Usar `volatile` o sincronización.

4. **Dificultad de depuración**:

   - Errores no deterministas (ocurre solo bajo ciertas
     condiciones de ejecución).

5. **Sobrecarga por cambio de contexto**:
   - El SO debe alternar entre hilos, lo que consume
     tiempo de CPU.

---

## **Ejemplo de Rendimiento vs. Complejidad**

**Escenario**: Procesar 10,000 números (calcular el
cuadrado de cada uno).

- **Enfoque secuencial**:

  ```java
  for (int i = 0; i < 10000; i++) {
      Math.pow(i, 2);
  }
  ```

  - **Ventaja**: Simple.
  - **Desventaja**: No usa múltiples núcleos.

- **Enfoque multihilo**:
  ```java
  ExecutorService executor = Executors.newFixedThreadPool(4); // 4 hilos.
  for (int i = 0; i < 10000; i++) {
      final int num = i;
      executor.submit(() -> Math.pow(num, 2));
  }
  executor.shutdown();
  ```
  - **Ventaja**: Más rápido en CPUs multicore.
  - **Desventaja**: Riesgo de carreras de datos si hay
    recursos compartidos.

---

# **Resumen**

- **Procesos vs. Hilos**: Los procesos son
  independientes y costosos; los hilos son ligeros y
  comparten memoria.
- **Ventajas multihilo**: Rendimiento, responsividad, y
  eficiencia.
- **Desafíos**: Sincronización, deadlocks, y complejidad
  de depuración.

**Próximo paso**: Dominar cómo crear y gestionar hilos en
Java (Módulo 2). 🧠

# SO-SistemaImpresorasRed

Repositorio de la asignatura Sistemas Operativos para el proyecto **Sistema de Impresoras en Red**.

## Integrantes

| Nombre | Apellido |
|--------|---------------|
| Xavier | Santamaria |
| Matthews | Vega |

## Descripción de la situación

Simulación de un entorno donde múltiples usuarios envían documentos a un
conjunto limitado de impresoras conectadas a una red, gestionando el orden
y la disponibilidad de los dispositivos.

## Análisis

Las unidades de ejecución participantes son los procesos o hilos que reciben y gestionan las solicitudes de impresión, junto con los procesos o hilos que administran las impresoras. Los recursos compartidos principales son las impresoras disponibles y la cola de trabajos pendientes. Se requiere cooperación porque varias solicitudes pueden llegar al mismo tiempo; por ello, el sistema debe ordenar los trabajos, controlar la disponibilidad de las impresoras y evitar que dos unidades asignen simultáneamente un mismo recurso. La siguiente tabla compara cómo se implementaría cada modelo en esta situación.

**Tabla 1.** *Comparación de modelos en el Sistema de Impresoras en Red*

| Criterio | Memoria Compartida | Paso de Mensajes |
|---|---|---|
| **Ubicación del recurso** | La cola de trabajos y el estado de cada impresora (libre/ocupada) se guardan en una zona de memoria a la que acceden todos los procesos. | Un proceso servidor de impresión (spooler) es el único que guarda la cola y el estado de las impresoras. |
| **Envío de un documento** | El usuario escribe directamente su trabajo al final de la cola compartida. | El usuario envía un mensaje y el servidor lo agrega a su cola. |
| **Asignación de impresora** | Cada impresora libre revisa la cola compartida y toma el siguiente trabajo. | El servidor elige el siguiente trabajo y envía un mensaje a una impresora libre. La impresora avisa con un mensaje cuando termina. |
| **Control de conflictos** | Se necesitan mutex para proteger la cola y semáforos para contar las impresoras libres y los trabajos pendientes. Un error en la sincronización provoca condiciones de carrera. | Los conflictos se evitan por diseño, porque solo el servidor modifica la cola. Los mensajes se atienden uno por uno. |
| **Orden de atención** | Depende de que todos respeten correctamente el acceso a la cola (FIFO protegida). | El servidor garantiza el orden FIFO o por prioridad de forma centralizada. |
| **Adecuación a una red** | Poco natural, porque los usuarios están en equipos distintos que no comparten físicamente la memoria. | Natural, porque la comunicación en red ya funciona mediante mensajes. |
| **Ventajas** | Acceso muy rápido a los datos y menor costo de comunicación dentro de una misma máquina. | Mayor seguridad y orden, fácil de escalar (agregar usuarios o impresoras) y menos errores de sincronización. |
| **Desventajas** | Mayor complejidad para sincronizar, riesgo de interbloqueos y datos corruptos. Difícil de aplicar entre equipos distintos. | Más lento por el envío de mensajes. El servidor puede convertirse en un cuello de botella o punto único de falla. |

### Conclusión del análisis

Tras la comparación, el modelo más adecuado para este escenario es el de **Paso de Mensajes**. Los usuarios y las impresoras están distribuidos en una red y no comparten físicamente una misma memoria, por lo que comunicarse mediante mensajes refleja de forma directa el funcionamiento real del sistema. Además, centralizar la cola en un servidor de impresión garantiza que cada trabajo se asigne a una sola impresora, que se respete el orden de llegada y que ningún documento se pierda o se mezcle, sin depender de mecanismos de sincronización complejos. Aunque la Memoria Compartida ofrece mayor velocidad, sus ventajas pesan menos en este caso, porque el tiempo de impresión es mucho mayor que el de enviar un mensaje. En cambio, sus riesgos de condiciones de carrera e interbloqueos sí afectarían directamente la fiabilidad del sistema.
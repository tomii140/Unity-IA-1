# Estado del Proyecto y Deuda Técnica

> **Aviso de estado:** Este proyecto se encuentra **en desarrollo / en proceso**. La funcionalidad base opera en términos generales, pero existen puntos de arquitectura, rendimiento y diseño pendientes de refactorización y corrección. Este documento resume el estado actual y las mejoras planificadas para quien revise el repositorio.

---

## 1. Legibilidad y organización del código fuente y proyecto

* La estructura general es funcional, pero hay margen de mejora en modularidad y desacoplamiento.
* **Métodos `Start` y `Update`:** Concentran demasiada lógica. Se debe extraer comportamientos en métodos más pequeños y específicos para mejorar la legibilidad y el mantenimiento.
* **Reutilización y *God Scripts*:** Conviene dividir responsabilidades en componentes independientes en lugar de concentrar gran parte del comportamiento de un agente en una sola clase. Por ejemplo, separar la percepción de los lobos en un componente propio permitiría reutilizarla en otros agentes.
* **Gestión de referencias (`GameManager`):** Actualmente el `GameManager` centraliza referencias directas al cazador y a otros objetos de la escena. El objetivo es que cada componente obtenga o resuelva lo que necesita mediante sus propios mecanismos de búsqueda o comunicación.
* **Acoplamiento en listas compartidas:** El acceso directo a una lista compartida de lobos activos puede reemplazarse o complementarse con un sistema de eventos u otro mecanismo de comunicación que reduzca el acoplamiento entre sistemas.

---

## 2. Implementación y utilización del grafo

* El grafo cumple su función principal, pero la organización en escena puede mejorarse:
  * **Separación visual vs. lógica:** La representación visual del escenario y la lógica de nodos están demasiado acopladas; conviene separarlas para editar estética y estructura de forma independiente.
  * **Uso de *Prefabs*:** Los nodos están configurados de forma individual en escena en lugar de basarse en *Prefabs*. Convertirlos en *Prefabs* permitiría propagar cambios comunes desde un único punto en vez de editar nodo por nodo.

---

## 3. Implementación del algoritmo A*

* El algoritmo funciona en líneas generales, pero presenta dos puntos críticos a revisar:
  * **Actualización de prioridades en `PriorityQueue`:** Cuando se encuentra un camino de menor costo hacia un nodo ya presente en la cola, se actualizan su `G Cost` y `Parent`, pero no se reencola ni actualiza su prioridad (la estructura actual no permite modificar directamente la prioridad de un elemento ya insertado). Esto puede hacer que el nodo se expanda fuera de orden o pase al conjunto de cerrados con un costo no óptimo, arriesgando que el camino devuelto no sea el más corto.
  * **Estado de búsqueda en los nodos:** Almacenar valores como `G`, `H` u otros datos temporales de búsqueda directamente en los nodos es viable para una sola búsqueda secuencial, pero limita ejecuciones paralelas (por ejemplo, con *Jobs* o *Tasks*) o búsquedas repetidas entre *frames* sin colisiones de estado. Se planea aislar el estado por búsqueda.

---

## 4. Percepción, comportamiento e integración de los agentes

* **Percepción de los lobos:** Actualmente dependen de conocer al cazador a través del `GameManager`. Se busca migrar esto hacia mecanismos propios de búsqueda y percepción para desacoplar los agentes del manager global.
* **Velocidad del cazador:** `Max Speed` se sobrescribe continuamente desde `Update`, lo que impide ajustarlo de forma limpia en tiempo de ejecución desde el *Inspector* o mediante otros componentes. Debe evitarse esa sobrescritura constante.

---

## 5. Integración, performance y funcionamiento general

* El proyecto es funcional en su estado actual, pero las decisiones de acoplamiento vía `GameManager`, la resolución de referencias y el manejo de búsquedas limitan su escalabilidad, reutilización y mantenimiento a largo plazo.
* **Próximos pasos:** Refactorizar responsabilidades en componentes desacoplados, corregir la gestión de prioridades/estado en A* y limpiar el manejo de parámetros en *Update*.

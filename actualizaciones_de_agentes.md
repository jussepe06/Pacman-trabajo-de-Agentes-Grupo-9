# Actualizaciones de Agentes: Changelog y Mejoras de Arquitectura

Este documento describe la evolución técnica del proyecto de simulación de IA en Pac-Man. Se detallan los problemas presentes en el código base (commit inicial) y cómo las actualizaciones aplicadas transformaron el comportamiento del entorno y los agentes, basándose en la literatura de Inteligencia Artificial (Russell & Norvig).

---

## 1. Corrección del Paradigma de Agentes (Cazador vs Presa)
*   **Problema Inicial:** El código original de Claude había implementado a Pac-Man como el agente "inteligente" (otorgándole los modelos de Reflejo y Estado) y a los fantasmas como entidades sin lógica que vagaban al azar. Esto contradecía la evaluación clásica de algoritmos de persecución.
*   **Mejora Implementada:** Se invirtieron las arquitecturas. Los fantasmas pasaron a ser los "Agentes Evaluados" (las IAs) y Pac-Man se convirtió en su presa. La tabla de rendimiento ahora evalúa "quién atrapa más rápido a Pac-Man", cumpliendo con la rúbrica académica.

## 2. Resolución de Bucles Deterministas y Aleatoriedad
*   **Problema Inicial:** Los algoritmos calculaban la distancia menor matemáticamente de forma rígida. Al enfrentarse a varias rutas con el mismo coste, elegían siempre la misma. Esto causaba oscilaciones (fantasmas atrapados subiendo y bajando un pasillo eternamente) y resultados idénticos partida tras partida.
*   **Mejora Implementada:** Se introdujo *entropía controlada*. Cuando un agente detecta múltiples rutas óptimas (empates matemáticos de distancia), resuelve el empate eligiendo aleatoriamente. Esto eliminó los bucles infinitos y brindó un factor realista, haciendo que cada una de las simulaciones arroje resultados estadísticamente distintos.

## 3. Superposición y Física del Entorno
*   **Problema Inicial:** Los fantasmas se fusionaban en un solo bloque cuando seguían la misma ruta, ignorando su presencia mutua.
*   **Mejora Implementada:** Se alteró la función de construcción de `percepcion`. Los fantasmas ahora detectan a sus propios compañeros como obstáculos (como si fueran "paredes") en la celda inmediata, obligándolos a detenerse, rodear o buscar otra ruta, forzando una distribución más orgánica por el laberinto.

## 4. Expansión a 4 Arquitecturas Teóricas de IA
Se pasó de tener solo 2 modelos defectuosos, a implementar 4 modelos puros extraídos de la teoría de agentes reactivos:
*   **Blinky (Reflejo Simple):** Implementado con una función "golosa" (Greedy). Solo evalúa 1 casilla adelante priorizando la distancia Manhattan más corta inmediata a Pac-Man. Sin estado ni memoria.
*   **Pinky (Basado en Modelo):** Se le dotó de un `estadoInterno`. Guarda la posición pasada de Pac-Man y la actual para calcular un "vector de movimiento" (dx, dy). Usa esto para predecir dónde estará Pac-Man en 4 turnos (Predicción Futura) o navegar a su última posición conocida si lo pierde de vista (Memoria Pasada).
*   **Inky (Basado en Objetivos):** A diferencia de Blinky, Inky "planifica". Se le integró un algoritmo recursivo de Búsqueda en Anchura (BFS). En vez de ir en línea recta hacia una pared, Inky formula secuencias, buscando la ruta transitable real más corta en el laberinto, evitando encajonarse.
*   **Clyde (Basado en Utilidad):** Utiliza una función matemática compleja que balancea variables. Suma puntos para acercarse a Pac-Man y *resta masivamente puntos* si se acerca a otros fantasmas. Esto logra un comportamiento natural de **flanqueo** (si ve que los demás ya cubren un frente, él rodeará por el otro lado).

## 5. Parámetros de Observabilidad (Visión Parcial)
*   **Problema Inicial:** Los fantasmas eran omniscientes (siempre conocían las coordenadas de Pac-Man).
*   **Mejora Implementada:** Se añadió un selector de Visión. En "Visión Parcial", los fantasmas sufren "niebla de guerra" matemática (rango de 7 casillas). Esto permitió comprobar empíricamente que, al perder la señal, los agentes de reflejo vagan inútilmente, mientras que los basados en modelo usan su memoria para investigar la última ubicación registrada.

## 6. Evolución de Pac-Man a "Agente de Evasión Maximin"
*   **Problema Inicial:** Pac-Man era un simple bot recolector. Solo huía si el fantasma estaba exactamente a 1 casilla de distancia, lo que causaba que lo atraparan rápidamente.
*   **Mejora Implementada:** Se reconstruyó el `botPacman` utilizando una Función de Utilidad *Maximin*. Ahora, Pac-Man evalúa constantemente todos los caminos a su alcance simulando un paso a futuro. Elige la ruta que maximice la distancia mínima hacia el fantasma más amenazante. Este instinto puro de supervivencia disparó significativamente su tiempo de vida, poniendo a prueba a las IAs cazadoras.

## 7. Mejoras de UI y Analíticas de Prueba
*   **Mejoras:** Se separó la lógica de ticks de la lógica de animación interpolada en el *Canvas*, solucionando los bloqueos visuales ("tearing") en las paredes. Se añadió interactividad (tecla P para **Pausa**), un modal educativo de **Reglas**, panel de traza de decisiones en vivo, botón de aceleración algorítmica (**Fin Rápido**), y la capacidad de correr **lotes masivos de 80 simulaciones en fondo** para evaluar promedios y desviaciones estándar matemáticamente válidas.

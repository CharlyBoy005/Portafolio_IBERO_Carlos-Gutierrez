# Primer examen parcial

## Objetivo

Construir un juego Simón Dice de 4 colores en Raspberry Pi Pico 2.

- La secuencia crece +1 por ronda, de 1 hasta 15.
- La persona jugadora debe repetir la secuencia con 4 botones dentro de un tiempo límite por ronda.
- Tiempo límite por ronda (fase de entrada): TL = longitud + 5 segundos (p. ej., Ronda 7 → 12 s).
- Puntaje (0–15): mostrar la máxima ronda alcanzada en un display de 7 segmentos en hex (0–9, A, b, C, d, E, F).
- Aleatoriedad obligatoria: la secuencia debe ser impredecible en cada ejecución.

## Condiciones

1. **Encendido/Reset:** el 7 segmentos muestra “0” y queda en espera de Start (cualquier botón permite iniciar).
2. **Reproducción::** mostrar la secuencia actual (LEDs uno por uno con separación clara).
3. **Entrada:** al terminar la reproducción, la persona debe repetir la secuencia completa dentro de TL.
4. **Fallo (Game Over):** botón incorrecto, falta/extra de entradas o exceder TL.
5. **Progresión:** si acierta, puntaje = número de ronda, agrega 1 color aleatorio y avanza.
6. **Fin:** al fallar o completar la Ronda 15. Mostrar puntaje final en 7 segmentos (hex).
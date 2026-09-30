# AGENTS.md — PacMan MVP

Clon de Pac-Man vanilla JS (HTML + CSS + JS). Sin pasos de compilación,
sin package.json, sin tests, sin linter. Solo abre `src/index.html`
en un navegador.

## Cómo ejecutar / verificar

- Abre `src/index.html` en un navegador (Chrome/Firefox/Edge). No se
  necesita servidor de desarrollo.
- Canvas de 560x620; las teclas de flecha mueven a Pacman. Come todos
  los puntos para ganar; si un fantasma te atrapa, pierdes una vida
  (3 vidas).

## Estructura del proyecto

```
src/
  index.html          punto de entrada — carga los scripts en orden fijo
  css/style.css       estilo estilo arcade (sin framework)
  js/
    maze.js           layout del laberinto (28x31), posiciones iniciales, fila del túnel
    game.js           estado del juego, reglas de movimiento, colisiones, createGame/update
    render.js         dibujo en canvas (muros, puntos, Pacman, fantasmas, HUD)
    main.js           bucle (requestAnimationFrame), entrada de teclado, pantallas superpuestas
```

## Orden de carga de scripts (crítico)

`index.html` carga los scripts de forma secuencial. Los scripts
posteriores dependen de los globals expuestos por los anteriores.
**Mantén siempre este orden.**

```html
maze.js -> game.js -> render.js -> main.js
```

- `maze.js` expone `MAZE`, `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`
  (vía `window`).
- `game.js` lee esos globals; expone `createGame`, `update`, `DIRS`.
- `render.js` lee `game.grid` (la copia mutable por partida) y `DIRS`.
- `main.js` es el punto de entrada: crea el juego y ejecuta el bucle.

## Notas de arquitectura

- **Sin módulos / sin imports.** El estado se pasa mediante un grafo de
  dependencias global (`window`). Añadir `import` rompería el diseño.
- **El laberinto es inmutable.** `MAZE` (la cuadrícula prístina) nunca
  se muta. `createGame()` la copia a `game.grid`, que es lo único que
  cambia (se comen los puntos). Para nuevas funcionalidades, muta
  `game.grid`, nunca `MAZE`.
- **Sistema de coordenadas:** celda `(x, y)`, origen arriba-izquierda,
  x ∈ [0,27], y ∈ [0,30]. Tamaño de celda = 20px en render.
- **Túnel:** la fila 14 envuelve horizontalmente (los fantasmas y
  Pacman pueden salir por cualquier borde y reaparecer del lado opuesto).
- **IA de fantasmas:** el `hunter` (rojo) se mueve hacia Pacman; los
  demás son aleatorios. Los fantasmas nunca giran 180° salvo en un
  callejón sin salida.
- **Basado en frames:** `loop()` incrementa un contador global `frame`
  usado por la animación de la boca de Pacman (`render.js`).

## Flujo de trabajo de Spec-Driven Development

Este proyecto usa skills locales de OpenCode para desarrollo guiado
por especificaciones.

- `skill: spec` — diseña una spec (`carpeta specs/`).
- `skill: spec-impl` — implementa una spec **Aprobada** en una nueva rama.

Convenciones:
- Las specs viven en `specs/` con el nombre `NN-slug.md`.
- Los estados usan las etiquetas de `specs/template.md`:
  `Borrador | En revisión | Aprobado | Implementado | Obsoleto`.
- Cada paso del plan debe dejar el juego ejecutable (comitea tras cada uno).
- Las specs se crean/escriben en `specs/` — no crees archivos de specs
  en la raíz del repositorio.

## Directrices de edición

- Mantén `main.js` como el último `<script>` — es quien inicia el juego al cargar.
- Si añades un nuevo archivo JS, agrega su `<script>` en `index.html`
  **antes** de `main.js` y expone los globals que necesita.
- Las funciones de render esperan que `DIRS` (de `game.js`) esté en
  `window.DIRS` y las constantes del laberinto en `window`.
- Sin transpilación / sin Node. Cualquier `console.*` de depuración
  debe eliminarse antes de terminar (no hay compilación que los limpie).

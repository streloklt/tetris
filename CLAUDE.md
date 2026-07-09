# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Tetris implementado en JavaScript vanilla con HTML5 Canvas. Sin dependencias, sin build, sin `package.json`. Todo el proyecto son 3 archivos: `index.html`, `style.css`, `game.js`.

## Running the game

No hay build ni tests automatizados. Para probar cambios, abrir directamente o servir estáticamente:

```bash
open index.html          # macOS, abre en el navegador por defecto
python3 -m http.server 8000   # alternativa con servidor local
```

Verificar cambios manualmente jugando en el navegador (mover/rotar/soft drop/hard drop/pausa/game over/reinicio).

## Architecture (game.js)

Todo el estado y la lógica viven en `game.js` (~300 líneas), sin módulos ni clases: variables globales + funciones top-level operando sobre ellas.

- **Estado global**: `board, current, next, score, lines, level, paused, gameOver, lastTime, dropAccum, dropInterval, animId`.
- **Tablero**: matriz `ROWS × COLS` (20×10); cada celda es `0` (vacía) o índice de color 1–8 (`createBoard`).
- **Piezas**: matrices cuadradas en `PIECES`; `rotateCW` rota transponiendo + invirtiendo filas. Incluye la pieza "tuerca" (3×3 con hueco central en `0`), que deja un hueco real en el tablero al aterrizar: esa fila no se limpia hasta rellenarlo.
- **Colisiones**: `collide(shape, ox, oy)` — única fuente de verdad para saber si una posición/rotación es válida.
- **Wall kicks**: `tryRotate()` prueba desplazamientos ±1/±2 columnas si la rotación directa colisiona.
- **Ciclo de vida de la pieza**: `spawn()` → `merge()`/`lockPiece()` al aterrizar → `clearLines()` → siguiente `spawn()`. Si `spawn()` colisiona de inmediato, se dispara `endGame()`.
- **Game loop**: `loop(ts)` vía `requestAnimationFrame`, acumula `dt` en `dropAccum` y baja la pieza cuando supera `dropInterval`.
- **Puntuación/nivel**: `LINE_SCORES = [0,100,300,500,800]` × nivel actual; nivel sube cada 10 líneas; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
- **Render**: `draw()` dibuja grid + tablero + ghost piece (`ghostY()`, alpha 0.2) + pieza actual sobre `<canvas id="board">`; `drawNext()` dibuja la vista previa en `<canvas id="next-canvas">`.
- **Input**: listener de `keydown` mapea flechas/`X`/espacio/`P` a mover, rotar, soft drop, hard drop, pausa.

## Tunable constants

Al inicio de `game.js`: `COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`. Si se cambia `COLS`/`ROWS`/`BLOCK`, hay que ajustar también `width`/`height` de `<canvas id="board">` en `index.html` (`COLS × BLOCK` por `ROWS × BLOCK`).

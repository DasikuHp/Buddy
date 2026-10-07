# PROMPT 01 — Hito H0: harness + ajedrez contra Stockfish (local, Windows)

Copia todo lo que hay debajo de la línea y pégalo en Claude Code abierto en la carpeta `Buddy`.

---

Lee `goal.md` entero antes de hacer nada. Es el objetivo del proyecto; respeta la arquitectura de dos capas y el contrato de observación de la sección 3.2.

Estamos en Windows 11 nativo (portátil ROG Strix G18, i9 Ultra, GPU NVIDIA 8 GB). Python nativo de Windows, no WSL. Todo lo que no sea el juego debe poder correr después en Linux (Omarchy) sin cambios.

## Qué quiero en este hito

Un harness mínimo y el primer entorno: **ajedrez**. Al terminar debo poder:

1. Abrir un tablero en el navegador y jugar yo contra el agente.
2. Lanzar `python -m buddy.train.chess` y que el agente juegue N partidas contra Stockfish con nivel limitado, guarde checkpoints y escriba métricas (resultado, Elo aproximado, tiempo por jugada) en `runs/`.
3. Ver en logs qué observación ha pedido el agente y si el harness se la ha dado.

## Estructura que quiero

```
buddy/
  harness/        # bucle percibir-decidir-actuar, contrato de observación, registro de decisiones
  envs/chess/     # entorno de ajedrez: python-chess + Stockfish por UCI
  agents/         # agentes; el primero: una red pequeña (PyTorch) que elige entre jugadas legales
  brain/          # adaptador para Decider 2B (solo el esqueleto; no lo uses aún en el bucle)
  train/          # scripts de entrenamiento que corren solos y solo escriben métricas
  ui/             # tablero web mínimo (FastAPI + una página HTML con chessboard.js o similar)
runs/             # checkpoints y métricas (en .gitignore)
```

## Contrato de observación

Cada agente declara con un dataclass qué necesita: campos del estado, frecuencia y latencia máxima. El harness lo valida contra lo que el entorno ofrece. Si falta algo, falla al arrancar con un mensaje claro; no lo ignores. Registra cada decisión como JSONL: estado, opciones, elegida, probabilidad, resultado.

## Agente de ajedrez (primera versión)

- Entrada: tablero codificado (planos de piezas o FEN tokenizado, lo que sea más simple y rápido).
- Salida: puntuación para cada jugada legal; se elige por softmax con temperatura configurable.
- Entrenamiento en dos fases, ambas en `buddy/train/chess.py`:
  1. **Imitación:** generar posiciones jugando partidas Stockfish contra Stockfish y entrenar la red para predecir la jugada de Stockfish.
  2. **Partidas contra Stockfish** con `UCI_LimitStrength` y `UCI_Elo` configurables, subiendo el Elo del rival cuando el agente gana más del 55 %.
- Usa la GPU si está disponible; comprueba `torch.cuda.is_available()` y dilo en el log.

## Requisitos técnicos

- Python 3.11 o 3.12 con `uv` (si no está instalado, dime el comando y espera).
- Dependencias: `python-chess`, `torch` (build CUDA), `fastapi`, `uvicorn`, `numpy`.
- Stockfish 19: descárgalo de https://github.com/official-stockfish/Stockfish/releases/latest/download/stockfish-windows-x86-64-universal.zip, descomprímelo en `tools/stockfish/` (en .gitignore) y lee la ruta desde una variable de entorno `BUDDY_STOCKFISH` con ese valor por defecto.
- Un `README.md` corto con: instalar, lanzar la UI, lanzar el entrenamiento, dónde están las métricas.
- Tests mínimos con `pytest`: el contrato de observación falla cuando falta un campo; el entorno de ajedrez devuelve jugadas legales; una partida corta termina.

## Cómo trabajar

- Antes de escribir código, dime en 10 líneas cómo vas a hacerlo y qué dudas tienes. Espera mi OK.
- Commits pequeños con mensajes claros, en la rama `claude/vigilant-davinci-rn1hzn`. No abras PR.
- Si algo no funciona en Windows (por ejemplo, la build CUDA de torch), no lo disimules: dime el error exacto y la alternativa.
- No metas aún Decider, RLBot, RocketSim ni nada de Rocket League. Eso es el hito H1.

Cuando termines, dime qué comandos ejecuto yo para: (a) jugar contra el agente, (b) lanzar un entrenamiento corto de 5 minutos, (c) ver las métricas.

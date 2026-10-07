# PROMPT 02 — Hito H1: Rocket League con RLBot v5 (Epic, Windows)

Úsalo solo cuando el hito H0 (ajedrez) funcione. Antes de pegarlo en Claude Code, haz tú estos pasos a mano:

1. Instala RLBot v5 con el instalador de Windows: https://github.com/RLBot/launcher/releases/tag/installer
2. Abre el RLBot launcher una vez, deja que lance Rocket League desde Epic y comprueba que aparece un bot de ejemplo en una partida. Si no arranca el juego, dímelo: el soporte de Epic está documentado pero yo no lo he probado.
3. **Mallas de colisión para RocketSim (opcional en este hito, necesario para entrenar):** descarga la release de https://github.com/ZealanL/RLArenaCollisionDumper, abre Rocket League en **juego libre, sin conexión**, ejecuta el dumper y guarda la carpeta `collision-meshes/` fuera del repo. El dumper inyecta código en el proceso del juego; no lo ejecutes nunca con una partida online abierta.

Copia todo lo que hay debajo de la línea y pégalo en Claude Code abierto en la carpeta `Buddy`.

---

Lee `goal.md` y `PROMPT_01.md`. Ya existen el harness, el contrato de observación y el entorno de ajedrez. Ahora añadimos el segundo entorno: **Rocket League real** a través de RLBot v5. Rocket League es la versión de **Epic Games**, Windows 11 nativo, Python 3.12 nativo. RLBot v5 ya está instalado y he comprobado que lanza el juego.

## Qué quiero en este hito

1. Un bot RLBot v5 en `buddy/envs/rocket_league/` que use el **mismo harness** que el ajedrez: declara su contrato de observación (posición y velocidad del coche y la pelota, boost, estado del salto, a 120 Hz, latencia máxima 4 ms) y el harness la valida contra lo que entrega `GamePacket`.
2. Un agente de reflejos **programado a mano** (ir a la pelota, chutar hacia la portería) como primera política, para medir el bucle antes de entrenar nada.
3. Un script `python -m buddy.bench.rl_latency` que mida durante 60 s: ticks recibidos por segundo, tiempo desde que llega el `GamePacket` hasta que se envía el `ControllerState`, y percentiles p50, p95, p99. Resultado en `runs/` y resumen por consola.
4. Poder jugar yo contra el bot en una partida local (1 contra 1) lanzada desde un `match.toml` del repo.

## Restricciones

- Paquete `rlbot` de pip (v5, Python 3.11+). No uses nada de RLBot v4.
- No abras ninguna partida online; solo partidas locales que lanza RLBot.
- El bucle de juego no puede hacer E/S bloqueante ni escribir en disco por tick: acumula en memoria y vuelca al terminar.
- Si el harness no puede garantizar los 4 ms, que lo diga en el arranque con el número real, no que falle en silencio.
- No metas RocketSim ni rlgym todavía, salvo dejar el `pyproject` preparado para añadirlos. El entrenamiento es el siguiente prompt.

## Cómo trabajar

- Antes de escribir código, dime en 10 líneas cómo vas a hacerlo y qué dudas tienes. Espera mi OK.
- Commits pequeños en `claude/vigilant-davinci-rn1hzn`. No abras PR.
- Si algo del protocolo de RLBot v5 no está claro, consulta https://github.com/RLBot/python-interface y su wiki, y dime qué has supuesto.

Cuando termines, dime qué comandos ejecuto para: (a) medir la latencia, (b) jugar 1 contra 1 contra el bot, (c) ver las métricas.

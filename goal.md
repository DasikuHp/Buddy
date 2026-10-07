# GOAL — Agente de IA que actúa en una sandbox viva (juegos y tareas)

Fecha: 29 sep 2026

---

## 1. Mis mensajes (textuales, sin corregir)

### 1.1 A ChatGPT

> El documento que pegué empieza con la respuesta de ChatGPT, así que aquí solo están los dos mensajes míos que aparecen en él.

**Mensaje 1**

> busca el mejor dentro de lo que me cabria para compaginarlo con claude code o otra ia incluso para meterlo como compañero de juegos/quiero que vea mi pantalla como el modelo de nvidia aprenda a jugar y despues poder probarlo contra mi mismo ns si en un sandbox en mi propio ordenador pero me gustaria investiga bien y explica antes de empezar mi goal a ver si lo ahss entendido

**Mensaje 2**

> no se si ese es el stack que quiero que sea adaptable por eso habia pensado 1 que si darle directamente la memoria interna y el estado del motor.
> 2 entienda lo que pasa por eso un jev  o laya que son asequibles y con inferencia lo mas baja posible
> 3 que pueda enviar teclado, ratón  mando no tengo pero si le hes mas facil pues mando
> 4 aprenda pero tambien que pueda empezar desde 0 o incluso con un sanbox con neko para que pueda entrenar o incluso jugar contra mi
> 5 la memoria me hago cargo yo que ya tengo mi "Memoria asociativa semántico-episódica basada en grafos dinámicos y activación propagada (Semantic Associative Memory Networ "
> 6 y quiero en algo que pueda jugar por ejemplo a rocket league contra mi en una partida privada o al brawhala o a lo que yo quiera por eso el sanbox si hace falta solo
> 7 y mejor usar opencode para tener subagantes  analizar logs, modificar el entrenamiento, detectar fallos y ayudarte a evolucionarlo   y claude para programar el sistema
> investiga las mejores opciones Investigación avanzada

### 1.2 A Claude

**Mensaje 1**

> analiza la conversacion completa y investigando lo que he habaldo con chagpt dame una version realista de que podemos ahcer que hay hoy ne dia y si encuentras un eureka mejor que lo que hable con chagpt ademas dame un goal con todo lo que le he pedido a chatgpt con mis palabras exactas como inicio de output

**Mensaje 2**

> no llegaste a entenderlo quiero darle una sanbox que este vivo y que pueda actuar sobre juegos o tareas si quiero por eso lo de neko pero me intersa principalmente crear la idea que quiero decider 2b vl o 0.9 me serviria lo habia visto bien

**Mensaje 3**

> ahora que los has entendido solo dame un goal.md quiero que al principio pomga todos mis mensajes a ti y a chagpt despues lo que has entendido poniendo en duda que puede ser que no lo hayas entendido del todo y por ahora lo que haremos que es decider vl y neko o si hay una alteernativa de neko algo mejor mas rapido y mas exacto para lo que quiero

---

### 1.3 Respuestas a mis preguntas (29 sep 2026, textuales)

> **Hardware:** "ROG Strix G18 G815LP_G815LP, 8 vram, i9 Ultra, windows por ahora pero me gustaría que también poder correrlo en omarchy"
>
> **Primer hito:** "cualquier juego, había pensado rocket league (reflejos) / ajedrez (lógica) / y una búsqueda en internet"
>
> **Jugar contra mí:** "me gustaría poder conseguir la inmediatez o la menos latencia posible aunque tenga que usar el i9 ultra y la vram"
>
> **Reentrenar:** "reentrenamos nosotros, he pensado yo"
>
> **Reflejos / estado interno:** "sin problema, como necesite y si puede incluso ser consciente de que necesita eso mejor"
>
> **Quién entrena:** "puedo entrenarlo pero prefiero que lo entrenes tú en tu nube sin malgastar tokens"
>
> **Ajedrez:** "que peleé contra mí y que entrene contra Stockfish"
>
> **Orden:** "estoy instalando Rocket League así que sí"

---

## 2. Lo que he entendido (versión 2)

Una **sandbox viva** donde una IA actúa sobre juegos y tareas con la **menor latencia posible**, usando tu portátil (i9 Ultra, 8 GB de VRAM). Corre en **Windows ahora** y en **Omarchy después**. Tú juegas contra ella. Tú decides **qué** entrenar; el **cómo** y el cómputo de entrenamiento los pongo yo, en la nube, gastando pocos tokens (scripts que corren solos y solo devuelven métricas).

### Decisiones tomadas

1. **Un solo cerebro no vale.** Decider decide a ~1–5 por segundo (el 4B tarda 0,3–0,7 s por petición en CPU). Rocket League simula a 120 Hz (~8 ms). Se separan dos capas.
2. **Estado interno, no pantalla, en Rocket League.** RLBot v5 entrega el estado a 120 Hz. El agente **declara qué necesita** (ver 3.2) y el sistema se lo da. La pantalla queda como plan B.
3. **Ajedrez con dos modos:** rival que aprende (entrenado contra Stockfish) y jugar contra ti.
4. **Búsqueda en internet:** tarea para la capa lenta (Decider).
5. **Portabilidad:** Python y Docker para todo lo que no sea el juego. El juego corre nativo en cada sistema.

### Dudas que siguen abiertas

- **Nivel esperado en Rocket League.** Entrenar un bot decente por auto-juego cuesta cientos de millones a miles de millones de pasos. En un contenedor de 4 CPU sin GPU no llegaremos lejos. Ver 3.3.
- **"Consciente de lo que necesita".** Lo interpreto como un **contrato de observación**: el agente (o su capa lenta) pide qué campos del estado quiere y con qué frecuencia. No es metacognición. Corrígeme si querías otra cosa.
- **Decider con LoRA.** Cabe en 8 GB de VRAM en teoría. No lo he verificado en tu portátil.
- **Anti-cheat de Rocket League** en Linux/Proton para partidas online: no verificado. Para partidas privadas no debería importar.

---

## 3. Arquitectura y plan

### 3.1 Dos capas

| Capa | Función | Frecuencia | Implementación |
|---|---|---|---|
| **Reflejos** | Política pequeña que da los controles | 120 Hz, inferencia de menos de 1 ms | RL con auto-juego en RocketSim (RLGym 2 + rlgym-learn 2.0), jugada en el juego real con RLBot v5 |
| **Táctica / tareas** | Decider 2B: elige modo, objetivo, enlace, jugada candidata | ~1–5 Hz | GGUF o bf16 (~4 GB), LoRA con tus datos |
| **Memoria** | Tu SAMN, consultable, entra como hechos | — | Tuya |
| **Ingeniería** | OpenCode analiza logs y ajusta entrenamiento; Claude programa | Fuera del bucle | — |

### 3.2 Contrato de observación

Cada agente publica un pequeño manifiesto: campos del estado que necesita, frecuencia y latencia máxima. El harness lo cumple o avisa de que no puede. Así el mismo agente sirve con estado interno (Rocket League, ajedrez) o con árbol de accesibilidad del navegador (búsqueda), y pasa a captura de pantalla solo si no hay otra opción.

### 3.3 Dónde se entrena (decidido el 7 oct 2026: en local)

Se entrena en tu portátil (i9 Ultra, GPU de 8 GB). Es mejor máquina que el contenedor de la nube (4 CPU, sin GPU, efímero), así que la nube queda solo para programar y revisar logs.

- **Ajedrez:** Stockfish 19 como maestro. Primero imitación (destilar sus jugadas en una red pequeña), luego partidas contra Stockfish con nivel limitado. Corre bien en CPU.
- **Rocket League:** RocketSim necesita las mallas de colisión de tu instalación del juego. Las extraes tú con la herramienta de volcado de RocketSim; **no se suben al repo**. Con tu GPU se puede entrenar una política básica en horas; nivel alto son días o semanas.
- **Tokens:** los entrenamientos son scripts que corren solos con checkpoints y un resumen de métricas. Claude Code los escribe y los lanza; no mantiene el bucle a mano.
- **Windows primero:** RLBot obliga a Windows nativo (o Proton en Linux). Python nativo de Windows para todo; sin WSL en el bucle de juego, para no añadir latencia.

### 3.4 Hitos

1. **H0, ajedrez con Stockfish:** tablero en el navegador, tú contra el agente, agente contra Stockfish, métricas de Elo aproximado.
2. **H1, Rocket League en Windows:** bot mínimo con RLBot v5 leyendo el estado a 120 Hz, medir latencia real del bucle. Política inicial entrenada en la nube.
3. **H2, capa lenta:** Decider elige entre acciones (búsqueda web y modo de juego), con registro de decisiones para LoRA.
4. **H3, Omarchy:** repetir H0–H2 fuera de Windows.

### 3.5 Fuera de alcance por ahora

NitroGen y visión pura, Brawlhalla, RL de reflejos a nivel alto en la nube gratis, multijugador online.

---

## 4. Lo que no está verificado

- Rendimiento real de Decider 2B con LoRA en 8 GB de VRAM.
- Latencia extremo a extremo de RLBot v5 en tu portátil.
- Extracción de mallas de colisión en tu instalación de Rocket League y su uso en cloud.
- Rocket League en Omarchy con GPU híbrida (hay incidencias abiertas de gestión de energía).
- Que `decider-2b-vision` sirva para juegos.

## 5. Fuentes

- Decider: https://github.com/Mapika/decider
- RLBot v5, tick rate: https://wiki.rlbot.org/v5/botmaking/tick-rate/
- RLBot v5, sistemas operativos: https://wiki.rlbot.org/v5/framework/operating-system-support/
- rlgym-learn 2.0.0 y rocketsim 2.2.1: PyPI
- NitroGen: https://arxiv.org/pdf/2601.02427
- Omarchy, GPU híbrida: https://github.com/omacom/omarchy/issues/1776
- Neko: https://neko.m1k1o.net/
- Wolf / Games on Whales: https://games-on-whales.github.io/
- Selkies-GStreamer: https://github.com/selkies-project/selkies-gstreamer

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

## 2. Lo que he entendido (con dudas)

**Aviso:** ya me equivoqué una vez (tomé Neko por un sitio de entrenamiento y lo descarté por latencia), así que esto puede seguir estando incompleto o mal. Corrígelo antes de construir nada.

### Lo que entiendo

- Quieres una **sandbox viva**: un entorno aislado y persistente en tu ordenador donde una IA pueda **actuar** (ver, decidir, pulsar teclas/ratón, o mando virtual) sobre **juegos o tareas**, y donde tú puedas mirar, jugar contra ella o jugar con ella.
- Lo que más te interesa ahora es **crear la idea** y ponerla a funcionar, no optimizar un juego concreto.
- El **cerebro** sería un modelo de decisión tipo Jev/Laya, en concreto **Decider 2B con visión (VL)**, o el de ~0.9B. Es asequible y de inferencia baja, y no genera texto: elige entre opciones con probabilidades.
- Prefieres darle **el estado interno del juego** cuando se pueda, y **la pantalla** solo cuando no haya otra forma.
- La **memoria** es tuya (tu SAMN) y la resuelves tú: el agente solo debe poder consultarla.
- **OpenCode con subagentes** analizan logs, arreglan entrenamiento y detectan fallos; **Claude programa** el sistema. Ninguno juega dentro del bucle.
- A largo plazo: Rocket League en partida privada, Brawlhalla o "lo que yo quiera", y que el agente pueda **aprender**, incluso desde cero.

### Dónde puedo haberme equivocado (confírmame)

1. **"0.9"**: he supuesto que es Decider **0.8B**. ¿Es ese?
2. **"VL"**: he supuesto `decider-2b-vision`. Según su README, esa variante sigue con pesos de texto v5 y se está reeentrenando, así que puede ser inmadura. ¿Prefieres empezar con el 2B de texto y estado estructurado, y añadir visión después?
3. **Neko como pieza central**: ¿lo quieres sobre todo para **verlo y jugar tú dentro de la misma sesión**, o también como el canal por el que la IA percibe y actúa? Cambia mucho el diseño.
4. **Tareas**: ¿"tareas" son de navegador (formularios, clics), de escritorio en general, o de dentro de juegos?
5. **Rocket League y Brawlhalla en la sandbox**: los bots de Rocket League que vi se juegan en Windows, offline, y no he verificado que corran en un contenedor Linux. ¿La sandbox es un contenedor, o una ventana/VM en tu Windows?
6. **"Empezar desde 0"**: ¿es un agente sin experiencia previa (tabla rasa) o entrenar un modelo desde pesos aleatorios? Son cosas muy distintas.
7. **Aprender**: hoy no está en el paquete la fase de RL de Decider; lo que sí se puede es registrar partidas y reentrenar con datos propios. ¿Te vale como primera versión de "aprende"?

---

## 3. Lo que haremos por ahora

**Decisión provisional:** Decider (2B, con visión como experimento) + una sandbox viva. Neko es el candidato por defecto, pero comprobamos si hay una alternativa **más rápida y más exacta** antes de casarnos con él.

### Neko frente a alternativas

| Opción | Qué es | Lo bueno | Lo malo para tu caso |
|---|---|---|---|
| **Neko** | Escritorio/navegador en Docker con streaming WebRTC | Varios usuarios en la misma sesión; latencia menor de 300 ms; API REST y WebSocket para ratón/teclado; ya hay un MCP de la comunidad | Solo teclado y ratón; no está pensado para ser exacto ni de latencia de juego; el agente ve píxeles o capturas |
| **Playwright dentro del contenedor** | Control del navegador por CDP | Ve el **árbol de accesibilidad** con referencias estables a cada elemento, no píxeles: más rápido, más determinista y sin modelo de visión | Falla con canvas/WebGL, donde hay que caer a capturas. No sirve para juegos nativos |
| **Wolf (Games on Whales)** | Servidor Moonlight en Docker | Escritorios virtuales bajo demanda, baja latencia, GPU, teclado, ratón y mandos virtuales, multiusuario | Linux/Docker; no he verificado que sirva para tu portátil con Windows |
| **Selkies-GStreamer** | Escritorio Linux por WebRTC | 60 FPS a Full HD con GPU NVIDIA, pensado para investigación en contenedores | Necesita TURN dentro de Docker; también Linux |
| **Entorno propio (Gymnasium/Godot)** | El juego expone estado directo, sin vídeo | Lo más rápido y exacto posible; es lo que ya tienes en tus juegos de Godot | Solo sirve para juegos que controlas tú |

### Propuesta para empezar

- **Percepción exacta:** para tareas de navegador, el estado es el árbol de accesibilidad (Playwright), no píxeles. Encaja con cómo se entrenó Decider en navegador, donde los elementos llegaban como texto. Capturas solo como plan B.
- **Sandbox viva:** Neko sigue siendo la capa para **ver y compartir sesión contigo**. Que el agente lea y actúe por Playwright/CDP en vez de por WebRTC lo hace más rápido y exacto.
- **Juegos con GPU o mando:** probar en paralelo Wolf o Selkies solo cuando lleguemos a juegos que Neko no aguante.
- **Cerebro:** Decider 2B en GGUF Q4 (~1,3 GB). Decisiones a ~5 por segundo, suficiente para tareas y juegos por turnos, no para reflejos. El 0.8B solo si falta memoria; en las pruebas que vi rinde menos y apenas gana velocidad.
- **Memoria:** tu SAMN entra al estado como **hechos**, no como reglas largas en la pregunta, porque Decider no las sigue bien a este tamaño.
- **Aprender:** cada decisión se registra (estado, opciones, elegida, resultado). OpenCode arma datasets y Claude programa los adaptadores.

### Primer hito (medible)

Montar los tres harnesses con **la misma tarea de clics** y comparar:

- latencia del bucle completo (percibir, decidir, actuar);
- tasa de acierto de Decider en cada uno;
- esfuerzo para que tú juegues o intervengas en la misma sesión.

Con esos números decidimos si Neko se queda como núcleo o solo como visor.

### Fuera de alcance por ahora

Entrenar RL de reflejos, Rocket League, Brawlhalla real y NitroGen. Vuelven cuando la sandbox y el bucle con Decider funcionen.

---

## 4. Lo que no está verificado

- Que `decider-2b-vision` funcione bien en juegos o escritorio real: su propio README dice que los resultados en navegador son solo de tareas sintéticas de clic.
- Que Wolf o Selkies funcionen en tu portátil con Windows.
- Que Rocket League o Brawlhalla corran dentro de una sandbox en contenedor.
- Cuánto cabe en tus 8 GB de VRAM con Decider, la sandbox y un juego a la vez.

## 5. Fuentes

- Decider: https://github.com/Mapika/decider
- Neko: https://neko.m1k1o.net/ y su API https://neko.m1k1o.net/docs/v3/api
- Playwright MCP (snapshots): https://playwright.dev/mcp/snapshots
- Wolf / Games on Whales: https://games-on-whales.github.io/
- Selkies-GStreamer: https://github.com/selkies-project/selkies-gstreamer

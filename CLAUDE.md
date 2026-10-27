# CLAUDE.md

Guía para Claude Code al trabajar en este repositorio. Este archivo es la **especificación completa de diseño** del proyecto. No hace falta ningún documento externo.

---

Contenido (todas las referencias cruzadas de este archivo dicen "Sección N"):

| Sección | Título |
|---|---|
| 1 | Resumen del proyecto |
| 2 | Cómo trabajar |
| 3 | Decisiones confirmadas (fuente única de verdad) |
| 4 | Arquitectura mecánica (4.1 chasis, 4.2 dos niveles, 4.3 montaje, 4.4 dimensiones) |
| 5 | Alimentación (5.1 arquitectura … 5.5 batería) |
| 6 | ESP32 principal y mapa de pines |
| 7 | ESP32-CAM y cámara |
| 8 | UART entre los dos ESP32 |
| 9 | Sensores VL53L0X |
| 10 | LED de proximidad y LED de encendido |
| 11 | TB6612FNG y motores |
| 12 | Resumen de conexiones externas |
| 13 | Condensadores |
| 14 | Reglas de diseño de la PCB |
| 15 | Serigrafía |
| 16 | ERC y DRC |
| 17 | Verificación mecánica final |
| 18 | Etapas y orden de ejecución |
| 19 | Entregables y documentación |
| 20 | Puntos abiertos |
| 21 | Archivos del repositorio |

---

## Sección 1. Resumen del proyecto

**auxi** es un pequeño robot de fútbol, un proyecto universitario de primer semestre. Este repositorio contiene el diseño en KiCad de su **única PCB personalizada**.

**Para qué sirve el robot.** auxi juega fútbol en una competencia universitaria de robots. Una persona lo maneja con un control Xbox por Bluetooth Low Energy; su pieza delantera guía una pelota mientras la empuja. Las reglas de la competencia condicionan el diseño: el robot debe medir como máximo 150 mm y pesar como máximo 850 g, debe mostrar con tres LED de colores cuando algo está a menos de 10 cm de cada uno de sus tres sensores de distancia, y no debe interferir con otros equipos (por ejemplo con su video por Wi-Fi). También es un proyecto de aprendizaje: el equipo está en su primer semestre, así que se prefieren las soluciones simples, robustas y fáciles de armar sobre las ingeniosas.

**Qué contiene este repositorio.** La parte electrónica: una PCB personalizada que lleva los módulos controladores, el driver de motores, el regulador de voltaje, todos los conectores y los 4 LED. El chasis se imprime aparte (Sección 4.1). El firmware (ESP32 principal y ESP32-CAM) ya está escrito y vive en un **repositorio de GitHub separado**; usa el mapa de pines de la Sección 6.1, así que cualquier cambio en ese mapa debe hacerse en los dos repositorios.

- **Tracción:** 2 ruedas motrices (motores amarillos TT con caja reductora) más 1 rueda loca trasera de bola, cuya esfera es **metálica**.
- **Chasis:** impreso en 3D (ver Sección 4). Unos 144 mm de largo (límite 150 mm). El frente es una **pieza impresa en V (07) o curva (08) que guía la pelota**; es una pieza intercambiable atornillada a la base.
- **Arquitectura de dos niveles:**
  - **Nivel inferior (chasis):** base, bandeja, motores, rueda loca, batería (atrás, detrás de una compuerta), lastre y cableado.
  - **Nivel superior:** una **única PCB** sobre 4 columnas impresas (tornillos M3 × 25), a 10 mm de la bandeja y encima de las baterías y los motores.
- **La PCB lleva:**
  - Los módulos enchufables: ESP32-S3-DevKitC-1, ESP32-CAM y el módulo TB6612FNG.
  - El regulador MP1584.
  - Los componentes pasivos.
  - **Todos los conectores.**
  - **Los 4 LED, soldados directamente en la PCB**: los 3 LED de proximidad y el LED verde de encendido (ver Sección 10).
- **Todo lo demás está fuera de la placa** y se conecta a la PCB mediante conectores:
  - Los 3 sensores VL53L0X.
  - El interruptor (bornera de tornillo, en la torre del interruptor).
  - Los motores (borneras de tornillo).
  - La batería (bornera de tornillo).
  - La cámara (módulo estándar de 24 pines en el arco de la cámara, conectado a la ESP32-CAM por su flex de ~2 cm).
- **ESP32 principal:** controla el robot.
  - Recibe comandos de un **control Xbox por Bluetooth Low Energy**.
  - Lee los 3 sensores.
  - Maneja el TB6612FNG y los LED.
- **ESP32-CAM AI Thinker:**
  - Transmite video en vivo por Wi-Fi a un celular.
  - Muestra las distancias que recibe del ESP32 principal por UART.
  - Su cámara OV2640 va en una cuna impresa en el arco de la cámara, en el frente del chasis (ver Sección 7).
- **Sensores:** 3, montados en soportes impresos (04) en el frente: FRONT (0°), LEFT (+45°), RIGHT (−45°).
- **LED de proximidad:** 3, soldados en la PCB. Su comportamiento depende de los sensores y sigue las reglas de la universidad (Sección 10).

---

## Sección 2. Cómo trabajar

- **Usa las herramientas MCP de KiCad** (`mcp__kicad__*`) para crear, editar, verificar y organizar los archivos reales del proyecto. No edites a mano las expresiones S de `.kicad_sch` o `.kicad_pcb` salvo que el MCP no pueda hacer el trabajo.
- Trata esto como un proyecto real de ingeniería electrónica. **Nunca inventes** pines, dimensiones, footprints ni especificaciones eléctricas. Si un dato no está en este archivo ni en la documentación verificable del componente, **detente y pregunta** (por ejemplo, pide fotos o medidas con calibrador del módulo comprado).
- **La Sección 3 es la fuente única de verdad.** Si encuentras una contradicción, **no la resuelvas en silencio**: explícala y propón una solución.
- **El usuario es principiante.** Explica cada decisión en lenguaje sencillo, prefiere soluciones simples, robustas y fáciles de armar, y **no agregues funciones que no se pidieron**.
- **Idioma:** el usuario escribe en español, así que responde en español. La documentación del repositorio (`README.md` y este archivo) también está en español.
- **Archivos de bloqueo de KiCad:** los archivos `~*.lck` indican que KiCad tiene el proyecto abierto en la interfaz gráfica. Avisa al usuario antes de escribir en archivos que pueda estar editando.
- **Git:** haz commit solo cuando el usuario lo pida.

---

## Sección 3. Decisiones confirmadas (fuente única de verdad)

| Tema | Decisión |
|---|---|
| Número de PCB | **Exactamente una PCB personalizada.** Los módulos comerciales no cuentan como PCB adicionales. |
| Capas y piezas de la PCB | **1 capa de cobre (`B.Cu`, la cara inferior) con un relleno de GND en la misma capa.** `F.Cu` no tiene cobre. **Todos los componentes son THT** (sin SMD), montados en la cara superior y **soldados por abajo**. Sin agujeros metalizados, sin vías y sin puentes de cable. |
| Ubicación de la PCB | Nivel superior, sobre 4 columnas impresas con tornillos M3 × 25 (separadores impresos la mantienen a 10 mm de la bandeja), encima de las baterías y los motores. |
| Carcasa | **Ninguna** (decisión del usuario): el robot no tiene carcasa; la PCB queda visible sobre el chasis. Así los LED se ven (reglas de la universidad) y no hacen falta ventanas ni holguras de carcasa. |
| Tamaño de la PCB | **110 × 140 mm**, fijado por el usuario (antes era de 90 × 100 mm; el espacio extra es para el enrutado de una sola capa). No la agrandes sin aprobación. **El lado de 110 mm es el FRENTE del robot (entre las ruedas) y el lado de 140 mm va de adelante hacia atrás. Confirmado por el usuario con una foto del chasis.** |
| En la PCB | ESP32-S3-DevKitC-1, ESP32-CAM, módulo TB6612FNG, módulo MP1584, condensadores, resistencias, diodo y conectores. |
| Fuera de la placa (con un conector en la PCB) | 3 sensores VL53L0X, interruptor, 2 motores, batería y cámara OV2640. |
| Frente del chasis | Pieza delantera impresa en V (07) o curva (08) que guía la pelota (recomendada: la curva, de 10 mm de profundidad frente a un máximo de 12.9 mm). Va atornillada a la base sobre pivotes. |
| Batería | 2 × 18650 en serie (2S): 6.0 V mínimo, 7.4 V nominal, 8.4 V máximo. Atrás en el nivel inferior, detrás de una compuerta (14), para poder cargarla sin quitar la PCB. |
| Conector de la batería | **Bornera de tornillo de 2 pines y 5.08 mm** (del mismo tipo que los motores), soldada en los pads de `J_BAT1`, que conservan el footprint del XT30UPB-M (paso 5.0 mm, agujeros de 1.8 mm). BAT− en el pad izquierdo (marca `-`), BAT+ en el pad derecho (marca `+`). **Ya no hay enchufe con guía: la polaridad se revisa a ojo.** |
| Interruptor | Fuera de la placa, en la **torre del interruptor** (12) en un costado de la base: interruptor de palanca M6 (agujero de 6.2 mm), con la palanca hacia arriba. Va entre la batería y el regulador y corta **todo** el sistema. Bornera de tornillo de 2 pines `J_SWITCH1`, cable de 20 AWG. |
| ESP32 principal | **ESP32-S3-DevKitC-1 N16R8** (16 MB de flash, 8 MB de PSRAM octal, 44 pines en 2 filas de 22). Puede ser un clon: verifícalo con la documentación de Espressif. |
| Control | Control Xbox por Bluetooth Low Energy (el ESP32-S3 no tiene Bluetooth clásico). El modelo exacto y la biblioteca se eligen en el firmware. |
| Motores | 2 × motores amarillos TT con ruedas de 63 mm, atornillados a soportes de aluminio. El VM del TB6612FNG va después del interruptor; el firmware limita el PWM a 70 % como máximo. |
| Driver | Módulo comercial TB6612FNG, enchufado en la PCB con headers hembra. |
| Regulador de 5 V | Módulo ajustable MP1584EN mini, ajustado a 5.0 V. |
| 3.3 V | Del regulador de 3V3 de la placa del ESP32 principal. |
| Sensores | 3 × VL53L0X (módulo GY-530) a **3.3 V**, bus I2C compartido, un XSHUT independiente por sensor. |
| LED de proximidad | **LED de 5 mm soldados directamente en la PCB** (decisión del usuario), en los pares de agujeros `J_LED_*`. Los colores y el umbral siguen las reglas de la universidad: FRONT rojo (arriba), LEFT amarillo (en medio), RIGHT verde (abajo); encendidos a ≤ 10 cm. |
| LED de encendido | LED de 5 mm **verde, soldado directamente en la PCB** en `J_LED_POWER1` (decisión del usuario, reemplaza la idea del header externo + Dupont), con R1 = 1.5 kΩ. No lo controla ningún GPIO. |
| ESP32-CAM | AI Thinker, sin microSD, **desmontable** (se programa quitándola y usando la base ESP32-CAM-MB del usuario), sin header de programación. |
| Cámara | Módulo OV2640 estándar (24 pines, paso 0.5 mm, flex de ~2 cm) en una cuna impresa (16) del arco de la cámara (13), con la lente mirando al frente e inclinada 25° hacia abajo. Ver Sección 7. |
| UART ESP32 ↔ ESP32-CAM | Pistas directas en la PCB; sin conector. |
| Reglas de la PCB casera | Pistas ≥ 0.4 mm (se permiten 0.3 mm donde una pista pasa entre pads). Agujeros ≥ 0.3 mm. **Diámetro del pad = diámetro de la broca + 0.8 mm** en cada agujero con pad, para que quede cobre en el pad después de taladrar a mano. Ver Sección 14. |
| NO incluir | Monitoreo de la batería, header de programación, protección con MOSFET, cualquier otra función no pedida. |

---

## Sección 4. Arquitectura mecánica

> Para la etapa del esquemático, usa esta sección solo para elegir footprints y conectores que encajen en el armado final.

### Sección 4.1 Chasis (tal como se imprimió)

El chasis está impreso en 3D. Estos son sus datos. Los archivos de las piezas impresas viven en el proyecto del chasis, no en este repositorio; los números de pieza de abajo (01, 02, …) son los que se usan en ese proyecto.

- **Tamaño:** el robot mide **144.1 mm de largo** (límite 150 mm). La lente de la cámara sobresale solo 0.15 mm del frente.
- **Peso:** estimado en **762.5 g** con 3 placas de lastre y la bandeja estándar; objetivo 800 g; **límite reglamentario 850 g**. Pesa el robot terminado antes de agregar más placas.
- **Piezas impresas:** 01 base · 02 bandeja (cercha Warren, LITE o estándar) · 03 columnas ×4 · 04 soportes de sensor ×3 · 05 cunas de motor ×2 · 07 frente en V o 08 frente curvo · 09 suplementos de la rueda loca · 10 caja de lastre · 11 tapa inferior · 12 torre del interruptor · 13 arco de la cámara · 14 compuerta de la batería · 15 separadores de la PCB ×4 · 16 cuna de la cámara (25°; 15° y 35° opcionales). Opcionales: 06 abrazaderas, 17 soporte de motor impreso, 18 suplementos de motor.
- **Base (01):** el piso del robot. Lleva los pivotes de las cunas de los motores y del frente, la abertura del lastre y las columnas de la rueda loca.
- **Tracción:** dos motores TT atornillados a soportes de aluminio (2 × M3 × 30), fijados a las cunas 05 con 2 × M3 × 20 desde abajo. **Ruedas: 63 mm.**
- **Rueda loca:** rueda de bola W420 (bola de 15.8 mm, 20.0 mm de alto). La bola es una **esfera metálica** (decisión del usuario). Va sobre dos columnas de la base con suplementos de 2 mm. El portapilas queda encima, a Z = 24.
- **Batería:** dos celdas 18650 protegidas en un portapilas **atrás**. El portapilas sale por detrás quitando dos pasadores de filamento y la compuerta (14). Carga fuera del robot con un cargador balanceador 2S.
- **Lastre:** la caja 10 bajo la base, cerrada con la tapa 11 (2 tornillos). Al inicio, tres pletinas de acero de 37 mm de unos 47 g cada una.
- **Bandeja (02) y columnas (03):** cuatro columnas sostienen la bandeja; la bandeja une las columnas por arriba y sostiene los soportes de sensor. Los separadores impresos (15) mantienen la PCB **a 10 mm de la bandeja**, para que los cables pasen por debajo sin quedar aplastados contra las soldaduras.
- **Sensores:** tres soportes (04) en el frente, FRONT / LEFT / RIGHT. Cada GY-530 va atornillado con 2 × M3 × 10 (tuerca atrás). **Sus cables van soldados al módulo por detrás (sin conector en el sensor)**, salen por la ventana de la pared del soporte y van hacia atrás hasta los conectores JST-XH de la PCB. Nunca pasan delante del sensor ni de la cámara.
- **Arco de la cámara (13):** se apoya en las **dos esquinas delanteras de la PCB** y lleva la cuna de la cámara (16) y la placa de identificación del equipo. Cada pata recibe un M3 × 25 desde la barra superior que atraviesa el arco, la PCB y el separador hasta la tuerca de la columna 03.
- **Torre del interruptor (12):** torre hueca atornillada a la base con 2 × M3, en un costado del robot. El interruptor de palanca M6 queda dentro con la palanca hacia arriba. Los cables salen por una ventana del lado interior y entran directo a `J_SWITCH1` (unos 4 mm de cable pelado).
- **PCB:** 110 × 140 mm, en su posición real de KiCad. El diseño del chasis toma las alturas de las piezas de `auxi.kicad_pcb`, así que los modelos 3D de las piezas deben mantenerse correctos (ver Sección 21).

Coordenadas de la PCB: origen en el centro de la placa, X hacia la derecha, Y hacia adelante (el diseño del chasis usa la misma convención).

**Recorrido de los cables (planeado).** Los cables del nivel inferior suben por las aberturas de la bandeja al espacio de 10 mm, atraviesan la PCB por agujeros pasacables y llegan a su conector arriba.

| Cable | De → a | Tipo |
|---|---|---|
| Batería | portapilas → `J_BAT1` (bornera de tornillo), atrás a la derecha de la PCB | 18–20 AWG, **revisa la polaridad** |
| Motores | parte trasera de cada motor → borneras `J_MOTOR_*1` en el borde derecho | 20–22 AWG |
| Interruptor | torre del interruptor → `J_SWITCH1` | 20 AWG, nunca Dupont |
| Sensores | conectores JST-XH del borde delantero → los tres sensores (soldados por detrás) | 25 cm o menos, I2C a 100 kHz |
| Cámara | ESP32-CAM → cuna de la cámara en el arco | flex de unos 2 cm en una curva suave |

Prohibido: cables colgando sobre las ruedas y cables delante del sensor frontal o de la cámara. Amárralos con bridas cada 3–4 cm.

El diseño del chasis también propone tres agujeros pasacables en la PCB (mm desde el centro de la placa, X a la derecha, Y hacia adelante): sensores (X 38, Y 45, Ø 8 mm, 4.3 mm de cobre libre alrededor), potencia de batería y motores (X 28, Y −48, Ø 10 mm, 5.0 mm libres) y LED (X −41, Y −13, Ø 8 mm). **Ninguno se agregó: la PCB no cambió (la placa ya estaba fabricada).** El agujero de los LED ya no hace falta, porque los 4 LED están soldados en la placa. Los otros dos están en la Sección 20.

### Sección 4.2 Dos niveles (vista lateral)

```
   arco de la cámara (13) ┌── PCB única (nivel superior), 4 columnas impresas ──┐
                          │ ESP32-S3 · ESP32-CAM · TB6612 · MP1584 · LED        │
   ───────────────────────┴─────────┬── espacio de 10 mm (cables) ────┬──────────
                        bandeja (02) con la cercha Warren (soportes de sensor)
   ═════ baterías (atrás) · motores TT · lastre · cableado ═════════════
            base (01) · torre del interruptor (12) en un costado
```

**Por qué dos niveles:** el piso del chasis es pequeño. Los motores ocupan los costados, la caja de lastre y la batería ocupan el centro y la parte trasera, y la pieza delantera ocupa el frente. Con dos niveles, las baterías y los motores usan todo el piso y la PCB usa toda la zona superior. Además tiene estos beneficios:

- El peso de la batería queda abajo, lo que hace al robot más estable.
- Los cables suben distancias cortas hasta sus conectores.
- Los puertos USB, la ESP32-CAM y los conectores son accesibles desde arriba.

### Sección 4.3 Montaje, altura y revisión de espacio

Esta revisión es obligatoria antes de diseñar la PCB. No bloquea el esquemático.

- **Agujeros de montaje:** la PCB tiene 4 agujeros M3, uno cerca de cada esquina.
  - Cada agujero es de 3.2 mm, con una zona de exclusión de unos 6.5 mm de diámetro (sin cobre ni piezas), a al menos 4 mm del borde.
  - Los dos agujeros delanteros también sostienen el arco de la cámara (13). Cada agujero recibe un tornillo **M3 × 25** que atraviesa el arco (adelante) o la barra, la PCB y un separador impreso (15) hasta la tuerca de una columna (03).
- **Altura:** la PCB queda **a 10 mm de la bandeja** (separadores impresos), lo que la mantiene por encima de los motores, el portapilas y los cables.
  - En este diseño la bandeja impresa (plástico) queda entre las soldaduras de la PCB y los contactos metálicos del portapilas, que está atrás bajo la bandeja. Confirma en el robot real la separación de 5 mm a las partes metálicas; si no se cumple, agrega una lámina aislante.
- **Ruedas:** las ruedas son de **63 mm** de diámetro y quedan fuera de la PCB de 110 mm, a sus costados (no se espera contacto). Aun así, revisa el chasis real.
- **Frente y cámara:** la PCB no debe bloquear la entrada de la pelota ni la vista de la cámara. El arco de la cámara se apoya en las dos esquinas delanteras de la PCB y la cámara mira hacia adelante por encima del borde delantero.
- **Recorrido de cables:** ver Sección 4.1. Los cables suben por las aberturas de la bandeja al espacio de 10 mm y llegan a su conector (no hay carcasa). Los agujeros pasacables propuestos en el diseño del chasis **no** están en la PCB fabricada (Sección 20).
- **Tamaño: 110 × 140 mm, fijado por el usuario.**
  - **Orientación confirmada (foto del usuario):** el lado de 110 mm es el frente del robot y queda entre las dos ruedas; el lado de 140 mm va de adelante hacia atrás. El robot mide 144.1 mm de largo, así que la PCB de 140 mm casi ocupa todo el largo. Compruébala contra la pieza delantera y la rueda loca en el chasis real.
  - Comprueba que todos los módulos, conectores, agujeros M3 y zonas de exclusión de antena caben en ese contorno, con espacio para enrutar y para enchufar los cables a mano.
  - El ESP32-S3-DevKitC-1 mide unos 63–68 mm de largo. Su extremo USB queda a **1 mm** del borde trasero (inferior) (pedido del usuario: máximo 2 mm, lo más cerca posible), así que el conector USB-C sobresale del borde de la placa y nada lo bloquea. Mantén libre de piezas altas el espacio alrededor del extremo USB (el conector necesita unos 15 × 12 mm ahí).
  - Si algo no cabe, **no agrandes la placa**. Explica el problema y propón alternativas antes de continuar.
- Presenta una revisión en 2D (vistas superior y lateral) con medidas reales y **espera la aprobación**.
- Evalúa la distribución del peso: las ruedas motrices deben conservar tracción. Informa si es relevante.

### Sección 4.4 Dimensiones de referencia (verificar con calibrador)

Estos son valores de referencia para la distribución física. Los clones varían: confirma con el usuario antes de fijar posiciones.

| Componente | Dimensiones aprox. (mm) | Nota |
|---|---|---|
| ESP32-S3-DevKitC-1 | 62.74 × 25.40 oficial; clon del usuario ~68 × 25 | DXF oficial: los headers están a 1.27 mm de cada borde largo, por lo que **las filas quedan a 22.86 mm**; USB en el extremo del pin 22, antena en el extremo del pin 1 |
| ESP32-CAM AI Thinker | 26.9 × 39.8 (foto del vendedor) | Filas **a 22.86 mm** (valor común); pines en el extremo de la microSD, antena y FPC en el otro extremo (supuesto) |
| Módulo TB6612FNG | 20.5 × 20.4 (foto del vendedor, módulo rojo genérico) | Separación entre filas **supuesta de 17.78 mm (0.7")**; verificar |
| Módulo MP1584EN mini | 22.5 × 17.2 × 4 (foto del vendedor, tipo D-SUN) | 8 agujeros pasantes, 2 por terminal (paso 2.54 mm), ~19.9 mm entre lados (estimado) |
| Portapilas 2×18650 | ~75–78 × ~40 × ~18–22 | Nivel inferior |
| Motor TT (cuerpo) | 70 × 22 × 18; ~37 de ancho incluyendo ambos ejes | Nivel inferior, en un soporte de aluminio, más la rueda de 63 mm |
| VL53L0X GY-530 | ~25 × 11 | Fuera de la placa, en los soportes impresos (04) del frente |
| JST-XH de 5 pines vertical | ~14.9 × 5.75 × 7 | Uno por sensor |
| LED de 5 mm, THT | Ø5.0 (reborde Ø5.8) × 8.6 de alto | Los 4 LED soldados en la PCB (modelos 3D en `auxi.3dshapes/`) |
| Bornera de tornillo de 2 polos, 5.08 mm | ~10.2 × ~9.8 × ~10 | **4 en la placa:** motores ×2, interruptor y batería |
| Pads de la batería (footprint XT30UPB-M) | paso 5.0 mm, agujeros de 1.8 mm, pads de 3.0 mm | Sostienen la bornera de 5.08 mm de la batería (diferencia de paso de 0.08 mm: los agujeros de 1.8 mm la absorben) |
| Electrolítico de 470 µF | ~Ø10 × 13–20 | La altura depende del voltaje y la marca |
| Agujero M3 | Ø3.2 con zona de exclusión de ~Ø6.5 | 4 esquinas |

Los módulos enchufados en headers hembra quedan a cerca de 1 cm sobre la PCB. No hay carcasa, así que esto solo importa para lo que se monte encima de la PCB (comprueba que nada choque con los módulos).

---

## Sección 5. Alimentación

### Sección 5.1 Arquitectura

```
BAT 2S ─ J_BAT (bornera de tornillo) ─ J_SWITCH ═(cables)═ interruptor externo ═(cables)═ J_SWITCH ─ +VBAT_SW
                                                                        │
+VBAT_SW ─┬─ R1 1.5 kΩ ─ LED verde de encendido (soldado en J_LED_POWER) ─ GND
          ├─ condensador grande ─ TB6612 VM (motores)
          └─ MP1584 ─ +5V ─┬─ diodo Schottky ─ +5V_ESP ─ pin 5V del ESP32
                           └─ pin 5V de la ESP32-CAM
Pin 3V3 del ESP32 ─ +3V3 ─┬─ 3 × VL53L0X
                          ├─ TB6612 VCC (lógica)
                          └─ pull-ups de I2C (si hacen falta)
```

**Regla:** nada se conecta a la batería antes del interruptor. Con el interruptor apagado, el regulador, el driver de motores y todos los LED quedan sin alimentación.

### Sección 5.2 Motores y PWM

- Los motores TT suelen estar especificados para 3–6 V. **Verifícalo en la hoja de datos del vendedor.**
- VM se alimenta desde `+VBAT_SW`. El firmware limita el PWM a un máximo de 70 % (0.7 × 8.4 V ≈ 5.9 V promedio). Documenta esto como regla obligatoria del firmware.
- **No subas el voltaje de la batería.**
- **Corriente de bloqueo:** consigue la del motor real, o pide al usuario que la mida. Compárala con los límites del TB6612FNG en la hoja de datos de Toshiba. Las cifras habituales son 1.2 A continuos y 3.2 A pico por canal; verifícalas. Si el driver no alcanza, informa antes de continuar.

### Sección 5.3 Reguladores

- **MP1584:**
  - Verifica la distribución de pines y las dimensiones del módulo real; los clones difieren.
  - Calcula el presupuesto de corriente de 5 V con las hojas de datos: ESP32-S3 con Bluetooth, ESP32-CAM con Wi-Fi y cámara, sensores y LED.
  - Revisa el margen y la temperatura.
- **3V3 del ESP32:** comprueba en el esquemático oficial del ESP32-S3-DevKitC-1 que su regulador puede alimentar el ESP32, los 3 sensores, la lógica del TB6612 y los 3 LED con margen suficiente.
- **Diodo Schottky** (tipo SS14 / 1N5819, ≥1 A) entre `+5V` y el pin 5V del ESP32.
  - Evita que el USB de la computadora alimente el robot, y que el robot alimente el USB, mientras se programa.
  - Comprueba que el voltaje resultante (unos 4.5–4.7 V) alcanza para el regulador de la placa.
  - Si no alcanza, propón ajustar el MP1584 un poco más alto (por ejemplo 5.2 V) y comprueba que la ESP32-CAM lo tolera (su rango típico es 4.75–5.25 V).
  - **Implementado:** D1 = 1N5819 (DO-41). El pin 5V del ESP32 recibe unos 4.5–4.6 V, que aún alcanzan para su regulador de 3.3 V.

**Presupuesto de corriente (estimado con valores típicos de hojas de datos):**

| Carga | Riel | Pico estimado |
|---|---|---|
| ESP32-S3 con BLE | 3V3, desde `+5V_ESP` | ~350 mA |
| 3 × VL53L0X | 3V3 | ~60 mA |
| 3 LED + lógica del TB6612 | 3V3 | ~15 mA |
| ESP32-CAM con Wi-Fi y video | `+5V` | ~500 mA |
| **Total en `+5V`** | | **~1.0 A** |

- El MP1584 tiene mucho margen a ~1 A (el módulo está especificado hasta 3 A).
- Desde la batería, la electrónica consume ~0.75 A a 7.4 V, más los motores.

### Sección 5.4 Interruptor externo y LED de encendido

- **Ubicación:** el interruptor va **fuera de la PCB, en la torre del interruptor (12) en un costado de la base**, con la palanca hacia arriba para accionarlo sin levantar el robot.
- **Posición eléctrica:** va **entre la batería y el regulador**. Por la regla de la Sección 5.1, también corta VM y todos los LED.
- **J_SWITCH en la PCB:** 2 terminales, `SW_BAT` (desde la batería) y `SW_OUT` (hacia `+VBAT_SW`).
  - **Toda** la corriente del robot pasa por aquí, incluidos los picos de arranque de los motores.
  - Usa un conector con corriente nominal verificada de ≥ 3 A. **Implementado:** una bornera de tornillo de 2 pines y 5.08 mm de paso (del mismo tipo que los motores y la batería), que no necesita crimpado.
  - **No uses headers de 2.54 mm con cables Dupont para este camino.** Los Dupont aguantan unos 1–2 A y su contacto se afloja.
- **Pieza del interruptor:**
  - Corriente continua nominal ≥ 3 A DC y ≥ 12 V DC, **verificada en la hoja de datos**.
  - Estima la corriente pico del sistema y justifica la elección.
  - **Chasis impreso:** interruptor de palanca roscado M6 (agujero de 6.2 mm en la torre), cable de 20 AWG. Un interruptor deslizante o un mini balancín no cabrían en ese agujero.
  - Indica el calibre mínimo de cable.
- **J_LED_POWER en la PCB:** el par de agujeros (`LED+` / `LED−`) donde el LED de 5 mm **verde** va **soldado directamente** (decisión del usuario: sin header, sin cables Dupont, sin LED externo). `LED+` es el pad derecho, cuadrado (pata larga); `LED−` es el pad izquierdo (pata corta, lado plano). Está adelante a la derecha de la placa, junto a R1.
  - La resistencia limitadora está en la PCB. Dimensiónala para todo el rango de 6.0–8.4 V con unos 2–5 mA.
  - **Implementado:** R1 = 1.5 kΩ. Con un LED verde clásico (Vf ≈ 2.1 V) da unos 2.6 mA a 6.0 V y 4.2 mA a 8.4 V; con un verde brillante (Vf ≈ 3 V) unos 2.0 y 3.6 mA. Ambos son seguros. (Primero se dimensionó para un LED azul.)
- **Serigrafía:** `SWITCH` / `ON-OFF` en J_SWITCH; `POWER LED` y la polaridad en J_LED_POWER. (Ya está impresa en la placa: no cambió.)

### Sección 5.5 Batería

- No se agrega protección con MOSFET. **La batería antes era un enchufe XT30 con guía; ahora es una bornera de tornillo, así que nada impide una conexión invertida salvo la serigrafía `BAT+` / `BAT−` y los colores de los cables (rojo +, negro −).** Comprueba la polaridad con un multímetro antes de cerrar el interruptor. Una batería invertida alimentaría al revés el TB6612FNG y el MP1584.
- El BMS y la carga de las celdas quedan fuera del alcance de la PCB. En la documentación, advierte que las celdas deben estar protegidas, o cargarse fuera del robot con un cargador balanceador 2S.

---

## Sección 6. ESP32 principal: ESP32-S3-DevKitC-1 N16R8

- **Placa confirmada:** ESP32-S3-DevKitC-1 N16R8 (módulo ESP32-S3-WROOM-1, 16 MB de flash, 8 MB de PSRAM octal), 44 pines en 2 filas de 22.
- **Puede ser un clon comercial.** Antes de dibujar el footprint:
  - Compara la serigrafía de pines del clon con la distribución oficial de Espressif.
  - Mide el largo, el ancho, la separación entre filas y la posición de los conectores USB y de la antena.
  - Si algo no coincide, **detente y pide fotos**.
- **Montaje:** 2 × headers hembra de 1×22, para que la placa sea desmontable.
  - Al menos un conector USB-C debe ser accesible desde un borde. La placa suele tener dos: UART y USB nativo.
  - Los botones RESET y BOOT deben seguir accesibles.
  - La antena debe sobresalir del borde de la PCB, o tener debajo una zona libre de cobre.
- **Pines que NO deben usarse** (verifica con la documentación de Espressif):
  - **GPIO35, GPIO36, GPIO37:** en placas con PSRAM octal (como la N16R8) la memoria interna los usa, así que no están disponibles.
  - Pines de arranque: GPIO0, GPIO3, GPIO45, GPIO46.
  - USB nativo: GPIO19, GPIO20.
  - UART0 (consola y programación): GPIO43, GPIO44.
  - LED RGB de la placa: GPIO48 en la revisión v1.0 y GPIO38 en la v1.1. Comprueba cuál tiene el clon y evítalo.
- **Pines de señal candidatos a verificar:** GPIO1, 2, 4–18, 21, 39–42 y 47.
- **Señales requeridas (17 GPIO):**
  - TB6612: AIN1, AIN2, PWMA, BIN1, BIN2, PWMB, STBY (7 salidas).
  - XSHUT_FRONT, XSHUT_LEFT, XSHUT_RIGHT (3 salidas).
  - I2C: SDA, SCL.
  - UART hacia la ESP32-CAM: TX, RX.
  - LED_FRONT, LED_LEFT, LED_RIGHT (3 salidas).
- **Agrupación:** agrupa las señales por zona para simplificar el enrutado:
  - Las señales de motores cerca del TB6612.
  - I2C y XSHUT hacia los conectores de sensores.
  - Las señales de LED hacia sus conectores.
- **Bluetooth:** el ESP32-S3 solo tiene Bluetooth Low Energy (BLE 5), no Bluetooth clásico. Anótalo para el firmware: el control Xbox debe conectarse por BLE. No hace falta hardware extra.
- **Alimentación:** por su pin 5V, a través del diodo Schottky (Sección 5.3).

### Sección 6.1 Mapa de pines (revisado para la PCB de una capa, implementado en el esquemático)

En una sola capa de cobre dos pistas no pueden cruzarse. Los pines se **reasignaron** para que las señales salgan de los headers del ESP32 en el mismo orden que sus destinos. El cambio se hizo antes de escribir el firmware, así que no costó nada; el firmware (repositorio separado) ya usa el mapa de abajo. Esto **reemplaza** el mapa anterior (SDA GPIO8, SCL GPIO9, XSHUT 4/5/6, LED 10/11/12, motores en 1/2/42/41/40/39/21).

| Señal | GPIO | Pin del header | Razón |
|---|---|---|---|
| XSHUT_RIGHT | GPIO4 | J1-4 | Las líneas XSHUT son las más interiores del grupo que va al norte: la primera va al conector de sensor más al este (RIGHT) y la última al más al oeste (LEFT), así que ninguna línea se cruza con otra |
| XSHUT_FRONT | GPIO5 | J1-5 | |
| XSHUT_LEFT | GPIO6 | J1-6 | |
| SCL | GPIO7 | J1-7 | SCL y SDA giran hacia el este por encima de las líneas XSHUT, a la altura de los pines 4 y 3 de los conectores de sensores (primero SCL, luego SDA) |
| SDA | GPIO15 | J1-8 | |
| UART_CAM_TO_ESP | GPIO17 | J1-10 | **RX** del ESP32. El pin de UART superior va al pin superior de la cámara (IO14 = TX de la cámara), el inferior al pin inferior de la cámara, para que los dos carriles no se crucen |
| UART_ESP_TO_CAM | GPIO18 | J1-11 | **TX** del ESP32 (RX de la cámara, IO13) |
| LED_FRONT | GPIO8 | J1-12 | |
| LED_LEFT | GPIO9 | J1-15 | |
| LED_RIGHT | GPIO10 | J1-16 | |
| PWMB | GPIO1 | J3-4 | El TB6612FNG queda justo detrás de J3 con sus pines en el orden opuesto, por eso las señales se listan al revés |
| BIN2 | GPIO2 | J3-5 | |
| BIN1 | GPIO42 | J3-6 | |
| STBY | GPIO41 | J3-7 | R2 de 10 kΩ a GND mantiene los motores apagados mientras el ESP32 arranca |
| AIN1 | GPIO40 | J3-8 | |
| AIN2 | GPIO39 | J3-9 | |
| PWMA | GPIO21 | J3-18 | |

- **Nota para el firmware:** I2C no está en los pines por defecto de Arduino. Usa `Wire.begin(15, 7)` (SDA = GPIO15, SCL = GPIO7). La UART de la cámara tampoco está en los pines por defecto de `Serial1`: `Serial1.begin(115200, SERIAL_8N1, 17, 18)` (RX = GPIO17, TX = GPIO18). **TX y RX de la UART cambiaron de lugar respecto al mapa anterior.**
- **Sin usar a propósito:**
  - 0, 3, 45, 46 (arranque).
  - 19, 20 (USB).
  - 43, 44 (consola).
  - 35, 36, 37 (PSRAM octal).
  - 38, 48 (LED RGB).
  - 47.
- **Pines libres:** GPIO11, 12, 13, 14 y 16 (J1-9, 17, 18, 19, 20).

---

## Sección 7. ESP32-CAM y cámara externa

- **Módulo:** ESP32-CAM AI Thinker (unos 27 × 40.5 mm; verificar), en la misma PCB.
- **Montaje:** 2 × headers hembra de 1×8, para poder quitarla fácilmente y programarla en la base ESP32-CAM-MB.
  - Marca la orientación en la serigrafía (pin 5V y lado de la antena) para que no se pueda insertar al revés.
- **Pines de UART:** no se usa microSD.
  - UART hacia el ESP32 principal: **IO13 = RX de la cámara** (`UART_ESP_TO_CAM`) e **IO14 = TX de la cámara** (`UART_CAM_TO_ESP`).
  - No interfieren con la cámara, los pines de arranque (GPIO0, 2, 12, 15), el LED de flash (GPIO4) ni la PSRAM (GPIO16).
  - No uses GPIO1/GPIO3 (U0) para este enlace.
  - IO14 puede emitir unos pocos bytes basura al arrancar; el firmware debe descartarlos.
- **Pines sin conectar:**
  - Todos los demás pines quedan sin conectar.
  - Los pines 3V3 y VCC del módulo son **salidas** y no se conectan.
- **Alimentación:** por el pin 5V, con condensadores locales (Sección 13).
- **Cámara (chasis impreso):**
  - La cámara es un **módulo estándar OV2640 u OV3660** (24 pines, paso 0.5 mm) con su **flex normal de ~2 cm**. El flex largo del plan inicial **ya no hace falta**.
  - Va en la cuna impresa (16) bajo el arco de la cámara (13): cavidad de 9.2 mm, lente a través de un agujero de 7.6 mm, inclinada 25° hacia abajo (las cunas de 15° y 35° son opcionales). Sujétala con una gota de silicona caliente o cinta.
  - La ESP32-CAM está al **frente de la PCB, casi centrada** (14.5 mm a la izquierda del centro, con la cámara mirando al frente), con su conector FPC en el extremo delantero, bajo la cuna. El flex sube en una curva suave.
  - Mantén libre el recorrido del flex, con espacio para levantar a mano el seguro del conector. Aléjalo del TB6612, de las pistas de los motores y de los cables de los motores.
  - **Para programar la ESP32-CAM:** quita los dos tornillos M3 × 25 de las patas del arco y levanta el arco (solo quedan ~3 mm sobre el extremo delantero del módulo), luego abre el seguro del flex y saca el módulo.
  - Mantén libre el campo de visión: ningún cable debe pasar delante de la cámara.
  - Si el video llega a tener ruido, baja XCLK en el firmware (por ejemplo de 20 a 10 MHz).
  - Durante los partidos, el firmware debería poder apagar el video por Wi-Fi: puede cortar el enlace Bluetooth del control, y el reglamento prohíbe interferir con otros equipos.
- **Antenas:** zona libre de cobre bajo la antena de la ESP32-CAM. Pon las antenas del ESP32-S3 y de la ESP32-CAM en bordes distintos de la PCB, lo más lejos posible entre sí, porque ambas usan 2.4 GHz.

---

## Sección 8. UART entre el ESP32 y la ESP32-CAM

- Pistas directas en la PCB, sin conector.
- Los nombres de red siguen el sentido de la señal:
  - `UART_ESP_TO_CAM`: TX del ESP32 → RX de la cámara.
  - `UART_CAM_TO_ESP`: TX de la cámara → RX del ESP32.
- Ambos lados usan lógica de 3.3 V, así que no hace falta convertidor de nivel.
- Enruta la UART lejos de las pistas de los motores. Velocidad sugerida para el firmware: 115200 baudios.

---

## Sección 9. Sensores VL53L0X (fuera de la placa)

- **Módulos:** 3 módulos GY-530, atornillados a los soportes impresos (04) en el frente del chasis (2 × M3 × 10, tuerca atrás). **Sus cables van soldados al módulo por detrás, sin conector en el sensor.** Un conector en la cara del sensor entraría en su campo de visión.
  - Verifica los pines del módulo real (normalmente VIN, GND, SCL, SDA, GPIO1, XSHUT).
  - GPIO1 no se usa.
- **Alimentación:** **3.3 V**, para que los pull-ups de I2C del módulo nunca puedan poner 5 V en el ESP32.
- **Conectores:** un JST-XH de 5 pines por sensor: `SENSOR_FRONT`, `SENSOR_LEFT`, `SENSOR_RIGHT`.
  - Mismo orden en los tres: 1 3V3, 2 GND, 3 SDA, 4 SCL, 5 XSHUT.
  - En la PCB los tres van **uno al lado del otro a lo largo del borde delantero (superior), a la derecha de la cámara**, LEFT (oeste), FRONT, RIGHT (este), separados 10 mm, con los pines alejándose del borde (pin 1 = 3V3 más cerca del borde). Los buses 3V3, SDA y SCL son entonces tres pistas horizontales rectas que pasan por el pad del mismo número de cada conector, así que no pueden cruzarse. Cada pista XSHUT llega al pin 5 desde abajo.
- **Bus I2C:** un solo bus compartido (SDA en GPIO15, SCL en GPIO7, ver Sección 6.1), se recomiendan 100 kHz y cables de 25 cm o menos.
- **XSHUT:** RIGHT en GPIO4, FRONT en GPIO5, LEFT en GPIO6.
- **Pull-ups:**
  - Revisa los de los módulos (tres en paralelo).
  - **Decisión (del usuario):** sin pull-ups extra en la PCB. Los módulos VL53L0X GY-530 ya incluyen sus pull-ups, por eso se quitaron R3 y R4 del diseño.
- **Nivel de XSHUT:** comprueba a qué voltaje lleva XSHUT el pull-up del módulo, y que el ESP32 puede manejarlo con seguridad. Recomendado: "habilitar" un sensor poniendo su pin como entrada para que el pull-up del módulo lo suba, en vez de manejar un 1 lógico.
- **Secuencia de arranque del firmware:**
  1. Poner todos los pines XSHUT en bajo.
  2. Habilitar un sensor.
  3. Asignarle una dirección nueva (0x30, 0x31, 0x32; la de fábrica es 0x29).
  4. Repetir con el siguiente sensor.
- **Campo de visión:** cada VL53L0X ve un cono de unos 25°. Con sensores a 0° y ±45°, hay zonas ciegas entre los conos y en los costados puros. Esto se acepta; documéntalo.
- **Comportamiento esperado:** cuando la pelota está dentro de la pieza delantera, el sensor FRONT la detecta y se enciende el LED rojo. Documéntalo.

---

## Sección 10. LED de proximidad (soldados en la PCB, reglas de la universidad)

- Los 3 LED de proximidad los manejan **solo los sensores de proximidad**. Cada LED muestra la detección de su propio sensor.
- **Las reglas de la universidad fijan los colores, y no se pueden cambiar:**

| Sensor | LED | Color (reglas) | Conector en la PCB |
|---|---|---|---|
| FRONT | LED_FRONT | **Rojo** | J_LED_FRONT |
| RIGHT | LED_RIGHT | **Verde** | J_LED_RIGHT |
| LEFT | LED_LEFT | **Amarillo** | J_LED_LEFT |

- **Regla de activación:**
  - El LED está encendido cuando su sensor lee **≤ 100 mm (10 cm)**.
  - El LED está apagado cuando la lectura es > 100 mm, inválida o fuera de rango.
  - El firmware puede promediar varias lecturas para evitar parpadeo. No debe usar histéresis que mantenga un LED encendido por encima de 100 mm, porque rompería las reglas.
- **Ubicación (decisión del usuario):** los 3 LED son **LED de 5 mm soldados directamente en la PCB**, en el lado oeste entre el MP1584 y el ESP32-S3, de arriba abajo: FRONT (rojo), LEFT (amarillo), RIGHT (verde) (vista superior, frente del robot arriba). El robot **no tiene carcasa** (decisión del usuario), así que los LED quedan visibles en la placa, como exigen las reglas, y no hacen falta ventanas.
- **Conexión:** el footprint sigue siendo el par de pads de 2 pines y 2.54 mm `J_LED_FRONT1` / `J_LED_LEFT1` / `J_LED_RIGHT1`. Las patas del LED entran directo en esos agujeros (sin header, sin cables Dupont).
  - El **agujero derecho** (pad cuadrado, junto a la resistencia R3, R4 o R5) es el **ánodo (+, pata larga)**. El **agujero izquierdo** es el cátodo (−, pata corta, lado plano del LED).
  - Serigrafía con el nombre, el color y la polaridad (`+` en el ánodo).
- **Resistencias:** una por LED, en la PCB, dimensionada con la Vf de cada color para unos 2–5 mA a 3.3 V.
  - El valor actual es 470 Ω, que da unos 2.5–3 mA.
  - **R5 (verde) es provisional:** sirve para un verde clásico (Vf ≈ 2.1 V). Un verde brillante (Vf ≈ 3 V) necesita recalcularse.
- **Nota para la documentación:** un LED soldado al revés simplemente no enciende y no se daña. Desuéldalo y voltéalo.
- **LED de encendido:** es un LED distinto de estos tres. Es **verde** (usuario), está **soldado directamente en la PCB** en `J_LED_POWER1`, adelante a la derecha de la placa junto a R1 (Sección 5.4). Primero se planeó como un LED externo junto al interruptor: eso ya no es así. Como el LED de proximidad RIGHT también es verde, comprueba que no se puedan confundir (están en lados opuestos de la placa, y el LED de encendido tiene la etiqueta `POWER LED`); si las reglas exigen que los LED indicadores se distingan, cambia el color del LED de encendido.
- **Modelos 3D:** los 4 LED tienen modelos de 5 mm de colores en `auxi.3dshapes/` (rojo, amarillo, verde; 8.6 mm de alto) para que la vista 3D y el diseño del chasis vean la altura real. El footprint no cambió.

---

## Sección 11. TB6612FNG y motores

- **Módulo:** módulo comercial TB6612FNG (tipo SparkFun "Dual TB6612FNG 1A" o un clon), **enchufado en la PCB** con headers hembra.
  - Todas sus conexiones pasan por la PCB: alimentación, señales y salidas de motor.
  - Los motores nunca se conectan directamente al módulo.
- **Antes de dibujar el footprint:** verifica el orden exacto de pines del módulo comprado, porque los clones varían. Si no hay datos, detente y pide una foto. El esquemático usa actualmente el orden de SparkFun (pin 1 = PWMA, pin 4 = STBY).
- **Conexiones:**
  - VM ← `+VBAT_SW`.
  - VCC ← `+3V3`.
  - Todos los pines GND del módulo a GND.
  - AIN1, AIN2, PWMA, BIN1, BIN2, PWMB, STBY ← ESP32.
  - Salidas del canal A → MOTOR_LEFT; salidas del canal B → MOTOR_RIGHT.
- **Conectores de motor:** `MOTOR_LEFT` y `MOTOR_RIGHT` son borneras de tornillo de 2 pines (paso 5.08 mm) en el borde derecho (este) de la placa. Los cables suben desde los motores por la bandeja y el espacio de 10 mm (Sección 4.1).
  - Documenta que si un motor gira al revés, se intercambian sus cables o se invierte en el firmware.
- **Condensador del motor:** un cerámico de 100 nF soldado entre los terminales de cada motor, en el propio motor, no en la PCB.
- **Ruedas:** las ruedas son de **63 mm** (vienen con los motores TT). Los motores van atornillados a soportes de aluminio y los soportes a las cunas 05.

---

## Sección 12. Resumen de conexiones externas

Cada componente externo se conecta a la PCB mediante un conector etiquetado. Nada se suelda directamente a los módulos. **La única excepción son los 4 LED (3 de proximidad + el de encendido), que se sueldan directamente en los agujeros `J_LED_*` y `J_LED_POWER1` (decisión del usuario).** El robot tiene **4 borneras de tornillo** en total: 2 motores, el interruptor y la batería.

En el esquemático, las referencias terminan en un dígito (`J_BAT1`, `J_SENSOR_FRONT1`…) porque KiCad lo exige; de lo contrario las renumera en la etapa de PCB.

| Referencia | Pieza externa | Tipo de conector | Señales | Ubicación sugerida |
|---|---|---|---|---|
| J_BAT1 | Baterías 2S | **Bornera de tornillo de 2 polos, 5.08 mm** (igual que los motores), soldada en los pads del footprint XT30UPB-M (paso 5.0 mm) | 1 BAT− (pad izquierdo, marca `-`), 2 BAT+ (pad derecho, marca `+`), de acuerdo con las marcas `−`/`+` del footprint de KiCad | Borde trasero, con la entrada de cables hacia atrás (los cables de la batería vienen desde atrás) |
| J_SWITCH1 | Interruptor | Bornera de tornillo de 2 polos, 5.08 mm, ≥3 A | SW_BAT, SW_OUT | Cerca de J_BAT1 |
| J_LED_POWER1 | LED verde de encendido (5 mm, **soldado aquí**) | 2 agujeros, 2.54 mm | LED+ (agujero derecho, pad cuadrado), LED− | Adelante a la derecha de la placa, junto a R1 |
| J_MOTOR_LEFT1 | Motor izquierdo | Bornera de tornillo de 2 polos, 5.08 mm | M_L1, M_L2 | Borde derecho (este) de la placa, junto al TB6612 (ver Sección 18: las dos borneras de motor están en el mismo borde para que las salidas del driver no tengan que cruzar el ESP32). El cable del motor izquierdo simplemente es más largo |
| J_MOTOR_RIGHT1 | Motor derecho | Bornera de tornillo de 2 polos, 5.08 mm | M_R1, M_R2 | Borde derecho (este) |
| J_SENSOR_FRONT1 | Sensor frontal | JST-XH 5P vertical | 3V3, GND, SDA, SCL, XSHUT_FRONT | Borde delantero (superior), en medio |
| J_SENSOR_LEFT1 | Sensor izquierdo | JST-XH 5P vertical | 3V3, GND, SDA, SCL, XSHUT_LEFT | Borde delantero (superior), oeste |
| J_SENSOR_RIGHT1 | Sensor derecho | JST-XH 5P vertical | 3V3, GND, SDA, SCL, XSHUT_RIGHT | Borde delantero (superior), este |
| J_LED_FRONT1 | LED rojo (5 mm, **soldado aquí**) | 2 agujeros, 2.54 mm | LED+ (agujero derecho, pad cuadrado), LED− | Lado oeste, el de arriba de los 3 |
| J_LED_LEFT1 | LED amarillo (5 mm, **soldado aquí**) | 2 agujeros, 2.54 mm | LED+ (agujero derecho, pad cuadrado), LED− | Lado oeste, en medio |
| J_LED_RIGHT1 | LED verde (5 mm, **soldado aquí**) | 2 agujeros, 2.54 mm | LED+ (agujero derecho, pad cuadrado), LED− | Lado oeste, abajo |
| FPC de la ESP32-CAM | Cámara OV2640 | Flex de 24 pines (en el propio módulo), ~2 cm | Bus de la cámara | Hacia el frente, bajo la cuna de la cámara |

**Módulos enchufables en la PCB:**

| Módulo | Conexión |
|---|---|
| ESP32-S3-DevKitC-1 | 2 × headers hembra de 1×22 (`U_ESP_J1`, `U_ESP_J3`) |
| ESP32-CAM | 2 × headers hembra de 1×8 (`U_CAM1`, `U_CAM2`) |
| TB6612FNG | 2 × headers hembra de 1×8 (`U_TB1`, `U_TB2`) |
| MP1584 | 4 pines soldados a la PCB (`U_MP1584`); verificar con el módulo real |

**Reglas de conectores:**

- **Conectores de potencia** (batería, interruptor, motores): borneras de tornillo de 5.08 mm, dimensionadas para la corriente real. Las cuatro son la misma pieza.
- **LED:** los 4 LED (3 de proximidad + el de encendido) se sueldan directamente en sus agujeros. En ninguna parte de la placa se usan headers ni cables Dupont.
- **Sensores:** JST-XH polarizado, para que 3V3 y GND no se puedan intercambiar.
- **Colocación:** todos los conectores en los bordes, mirando hacia donde sube su cable, con espacio para enchufar y desenchufar a mano.

---

## Sección 13. Condensadores

| Ubicación | Valor | Voltaje mínimo |
|---|---|---|
| `+VBAT_SW`, junto a J_SWITCH | electrolítico de bajo ESR de 470 µF + 100 nF X7R | 16 V (preferible 25 V) |
| VM del TB6612, justo en el header | electrolítico de 220 µF + 100 nF | 16 V (C3 usa 220 µF 16 V: el riel es de 8.4 V como máximo, decisión del usuario) |
| Entrada del MP1584 | 22 µF + 100 nF | 16 V (preferible 25 V) |
| Salida del MP1584 (`+5V`) | electrolítico de 100 µF + cerámico de 10 µF | 10 V |
| Pin 5V del ESP32 (después del diodo) | 47–100 µF + 100 nF | 10 V |
| Pin 5V de la ESP32-CAM | electrolítico de 470 µF + 10 µF + 100 nF | 10 V |
| `+3V3` junto al VCC del TB6612 | 10 µF + 100 nF | 10 V |
| Cada conector de sensor | 10 µF + 100 nF | 10 V |
| Terminales de cada motor (en el motor) | cerámico de 100 nF | 50 V |

- Usa cerámicos X7R o X5R en 0805 o mayor, y considera la pérdida de capacidad con el voltaje DC.
- Coloca cada 100 nF lo más cerca posible del pin que desacopla.
- Comprueba que la altura de los electrolíticos no choque con nada montado encima de la PCB (no hay carcasa).

---

## Sección 14. Reglas de diseño de la PCB

- **Decisión (del usuario, la más reciente):** **una sola capa de cobre.** El cobre es `B.Cu` (la cara inferior). `F.Cu` **no tiene cobre**: un área de exclusión (`NO_COPPER_TOP`) cubre toda la cara superior, así que arriba solo se ven los pads de agujero pasante. Razón: la mayoría de los talleres locales no metalizan los agujeros, y el usuario no quiere metalizar vías a mano. Una sola capa no necesita ningún agujero metalizado.
  - Esto reemplaza las decisiones anteriores de 2 capas ("sin plano, solo pistas", luego "plano de GND en B.Cu y señales en F.Cu").
  - El archivo de KiCad sigue declarando dos capas de cobre, porque KiCad no puede conservar solo `B.Cu`. Traza o imprime **solo `B.Cu`** para el grabado.
- **El GND es un relleno en la misma capa (`B.Cu`).** Llena todas las áreas libres. Los pads se conectan a él con alivios térmicos (separación 0.3 mm, radios 0.6 mm), para poder soldarlos a mano. Donde una pista dejaría aislado un pad de GND del relleno, una pista corta de GND conecta ese pad (10 segmentos cortos de GND en la placa final).
- **Decisión (del usuario, confirmada):** **todos los componentes son de agujero pasante (THT)** para poder soldarlos fácilmente a mano. Sin piezas SMD. Los componentes van en la **cara superior**; sus patas se **sueldan por abajo**, en el lado del cobre.
- **Sin vías y sin puentes de cable.** El mapa de pines (Sección 6.1) y la colocación se eligieron para que ningún par de pistas se cruce.
- FR4 de 1.6 mm, cobre de 1 oz, **reglas de placa casera (del usuario):**
  - Pistas **≥ 0.4 mm**. Las pistas que pasan **entre pads** pueden ser de **0.3 mm**. KiCad rechaza cualquier valor menor a 0.3 mm (`auxi.kicad_pro`).
  - Separación de 0.2 mm entre objetos de cobre, y 0.5 mm del cobre al borde de la placa.
  - Agujero mínimo **0.3 mm**. Vía por defecto de pad 1.1 mm / agujero 0.3 mm (no se usa, pero se fija para que cualquier vía futura siga la regla).
  - **Diámetro del pad = diámetro de la broca + 0.8 mm** en cada agujero con pad (0.4 mm de cobre alrededor). Lo hace cumplir la regla DRC personalizada `pad_ring_0.4mm` en `auxi.kicad_dru`. Cuatro footprints estándar se copiaron en `auxi.pretty/` con pads más grandes (PinSocket 1×8, PinSocket 1×22, PinHeader 1×2, JST-XH B5B-XH-A); el esquemático apunta a las copias.
- **Anchos de pista usados** (verifica los de potencia con una calculadora IPC-2152 usando la corriente de bloqueo, no la nominal):
  - Señales: 0.4 mm, o 0.3 mm donde se aprietan entre pads o van en un haz apretado (55 segmentos, 377 mm en total).
  - `+3V3`: 0.3–0.4 mm (unos 100 mA).
  - `+5V` y `+5V_ESP`: 0.6 mm.
  - Batería, J_SWITCH, VM y motores: 1.2 mm.
- **Zonas:**
  - Zona de potencia: bornera de la batería, J_SWITCH, TB6612 y borneras de motor (este y sureste del robot); el MP1584 está en el lado oeste junto a sus cargas de 5 V.
  - Zona lógica y de radio: ESP32-S3 (atrás), ESP32-CAM y conectores de sensores (adelante).
- **Tierra y enrutado:**
  - Mantén las corrientes de los motores lejos de I2C, UART, la ESP32-CAM y las antenas.
  - Las pistas pueden pasar entre los pads de 2.54 mm (el espacio es de 0.74 mm), pero solo con 0.3 mm de ancho.
  - Las piezas pequeñas (condensadores, R2) van bajo el módulo TB6612 y el extremo USB del ESP32, donde hay espacio. **Los condensadores electrolíticos no deben ir bajo un módulo** (son más altos que los 8.5 mm de altura libre).
- Zonas de exclusión en los agujeros M3 y bajo ambas antenas (`ANT_ESP32`, `ANT_CAM`): sin pistas, vías ni relleno.
- **Lado inferior:** ahora es el lado del cobre. No tiene componentes. Solo soldaduras de agujero pasante, con la separación de la Sección 4.3 respecto a las baterías.

---

## Sección 15. Serigrafía

> **Etapa posterior (PCB).** No ejecutar ahora; tenlo en cuenta al elegir footprints.

Etiqueta claramente:

- ESP32-S3, ESP32-CAM, TB6612FNG, MP1584.
- BAT+ y BAT−.
- SWITCH / ON-OFF y POWER LED.
- MOTOR_LEFT y MOTOR_RIGHT.
- SENSOR_FRONT, SENSOR_LEFT y SENSOR_RIGHT.
- LED_FRONT (RED), LED_LEFT (YELLOW) y LED_RIGHT (GREEN).
- Pin 1 y polaridad en cada conector y condensador.
- Orientación de los módulos, y una nota "5.0 V" de ajuste junto al MP1584.
- Una flecha "FRONT" que indique el frente del robot.
- Nombre y versión del proyecto ("auxi v1").

No pongas texto sobre pads ni en zonas que queden cubiertas por los módulos una vez montados.

---

## Sección 16. ERC y DRC

- **ERC:**
  - Corrige los errores reales. Nunca ignores ni suprimas errores para obtener un resultado de apariencia limpia.
  - Documenta la razón de cualquier advertencia que quede.
  - Usa símbolos de alimentación y `PWR_FLAG` solo donde correspondan, y explica por qué.
  - `PWR_FLAG` actuales: `+VBAT_SW`, `GND` y `+5V_ESP`. Estas redes se alimentan solo a través de conectores o del diodo, que KiCad trata como pines pasivos.
- **DRC:** se ejecuta en la etapa de PCB.

---

## Sección 17. Verificación mecánica final

> **Etapa posterior (PCB).** No ejecutar ahora; tenlo en cuenta al elegir footprints.

- La PCB mide 110 × 140 mm con el lado de 110 mm al frente, entre las ruedas (foto del usuario); todavía hay que confirmarlo con el chasis real (ver Sección 20).
- La PCB queda a 10 mm de la bandeja (separadores impresos, tornillos M3 × 25), encima de las baterías, los motores y los cables, con la separación mínima a los contactos metálicos.
- La PCB no choca con las ruedas ni bloquea la pieza delantera ni la cámara.
- 4 agujeros M3 con zonas de exclusión.
- El USB y los botones del ESP32-S3 son accesibles.
- La ESP32-CAM es desmontable, con acceso al seguro del conector FPC.
- El recorrido del flex (~2 cm) de la cámara está libre y lejos de la zona de potencia, y el arco se puede levantar para sacar la ESP32-CAM.
- Todos los conectores externos son accesibles y miran hacia donde suben sus cables: batería, interruptor, LED de encendido, motores, sensores y LED.
- El chasis tiene un recorrido de cables planeado (Sección 4.1). Los agujeros pasacables del diseño del chasis siguen sin decidirse (Sección 20).
- No hay colisiones entre componentes, módulos, cables y el chasis.

---

## Sección 18. Etapas y orden de ejecución

### Etapa 1: esquemático (HECHA, pendiente de verificación)

1. Revisar esta especificación y resolver con el usuario los puntos abiertos marcados **necesarios para el esquemático** (Sección 20).
2. Presentar el mapa completo de pines del ESP32-S3 y la ESP32-CAM, el presupuesto de corriente y la selección del interruptor y del cable. **Esperar la aprobación.**
3. Crear el proyecto de KiCad `auxi` mediante el MCP.
4. Crear el esquemático, organizado en bloques con título claro:
   - Alimentación: batería, interruptor, LED de encendido, MP1584, diodo y condensadores.
   - ESP32-S3.
   - ESP32-CAM.
   - TB6612FNG y motores.
   - Sensores.
   - LED de proximidad.
5. Usar etiquetas de red con los nombres de este archivo (`+VBAT_SW`, `+5V`, `+5V_ESP`, `+3V3`, `UART_ESP_TO_CAM`, `UART_CAM_TO_ESP`, `SDA`, `SCL`, `XSHUT_FRONT`, …).
6. Asignar y verificar el footprint de cada símbolo.
7. Ejecutar el ERC y corregir.
8. Revisar de nuevo la alimentación, los conectores y las señales contra este archivo.
9. Generar la lista de materiales preliminar y la documentación del esquemático.
10. Presentar un resumen final con las advertencias restantes y luego **detenerse**.

**Estado actual:**

- El esquemático está completo y el ERC pasa con 0 errores y 0 advertencias.
- El mapa de pines está aprobado.
- El diseño es **preliminar** hasta que los puntos de la Sección 20 se cierren con los módulos reales.

### Etapa 2: PCB (HECHA como placa de una capa de 110 × 140 mm, pendiente de las comprobaciones con las piezas reales)

El usuario no tenía calibrador, así que aprobó usar dimensiones comunes de la web (Sección 4.4). La posición de cada módulo es **provisional** hasta que se compruebe con la pieza real.

**Historia.** La primera PCB era de 90 × 100 mm con un plano de GND en `B.Cu` y señales en `F.Cu` (2 capas, 75 vías). Como los talleres locales no metalizan agujeros, el usuario pidió una **placa de una sola capa con relleno de GND**, soldada por abajo, de **110 × 140 mm**, con las reglas de pistas y agujeros de la Sección 14. La placa se recolocó y se reenrutó desde cero. La versión antigua de 2 capas está en el historial de Git (antes de este cambio).

**Orientación de la placa (dada por el usuario, foto del chasis):**
- Contorno de la placa: **x 106.07–216.07 mm, y 47.5–187.5 mm** en coordenadas de KiCad (110 × 140 mm, esquinas de 2 mm de radio).
- El **lado de 110 mm es el FRENTE del robot** (queda entre las ruedas) y el lado de 140 mm va de adelante hacia atrás. En KiCad: **borde superior (y = 47.5) = FRENTE**, **borde inferior (y = 187.5) = PARTE TRASERA**, borde izquierdo (x = 106.07) = lado IZQUIERDO del robot, borde derecho (x = 216.07) = lado DERECHO del robot.
- La ESP32-CAM está al frente con la cámara mirando hacia adelante (casi centrada, ver abajo); el ESP32-S3 está atrás con su puerto USB en el borde trasero (pedido del usuario: máximo 2 mm del borde).

**Historia.** (1) 90 × 100 mm, 2 capas, 75 vías. (2) 110 × 140 mm de una capa, suponiendo que el lado de 110 mm iba de adelante hacia atrás, con el ESP32-S3 a 26 mm del borde. (3) Tras la foto del usuario y el pedido de "USB en el borde", la placa se **recolocó y reenrutó desde cero** en la orientación final, con la cámara en la esquina delantera izquierda y los sensores en medio del borde delantero. (4) A pedido del usuario se intercambiaron la cámara y los sensores: la cámara ahora está **casi centrada** al frente y los sensores a su derecha (esta distribución). Las versiones anteriores están en el historial de Git.

**Colocación (x, y del pin 1 en mm):**

| Pieza | Posición | Notas |
|---|---|---|
| ESP32-S3 | `U_ESP_J1` (149.64, 125.90), `U_ESP_J3` (172.50, 125.90) | Los pines van hacia el borde trasero. **El extremo del módulo (lado USB) queda a 1.0 mm del borde trasero** (extremo del módulo en y ≈ 186.5, borde en 187.5), así que el conector USB-C sobresale de la placa. Exclusión de antena `ANT_ESP32` x 148.1–174.1, y 116.2–125.2 |
| ESP32-CAM | `U_CAM1` (135.07, 86.30), `U_CAM2` (157.93, 86.30), girada 180° | Al frente, centro en x = 146.5 (14.5 mm a la izquierda del centro de la placa, 161.07). Pines en el extremo trasero del módulo, antena y flex en el borde delantero. Exclusión `ANT_CAM` x 132.5–160.5, y 48–60.5. El pin 1 (+5V) es el pin más al sur de la fila oeste; los pines de UART son los pines 4 y 6 de la misma fila |
| TB6612FNG | `U_TB1` (178.64, 148.76), `U_TB2` (196.42, 148.76), girado 180° | Justo junto a `J3`. Fila de control hacia el ESP32, fila de potencia hacia el borde este |
| MP1584 | centro (118.0, 160.0), girado 90° | Lado oeste. OUT+ hacia la cámara (norte), IN+ hacia atrás (sur) |
| Sensores | `J_SENSOR_LEFT1` (165.0, 53.0), `FRONT1` (175.0, 53.0), `RIGHT1` (185.0, 53.0), girados 270° | A lo largo del borde delantero, a la derecha de la cámara, con los pines hacia el sur |
| LED de proximidad | pares de pads en x = 133.5, y = 154.0 / 159.5 / 165.0 (FRONT rojo, LEFT amarillo, RIGHT verde), resistencias R3–R5 en x = 138–146 | Lado oeste, entre el MP1584 y el ESP32-S3. Los LED de 5 mm se sueldan directamente en los pads; `+` es el pad derecho (cuadrado) |
| Motores | `J_MOTOR_RIGHT1` (209.5, 115.76), `J_MOTOR_LEFT1` (209.5, 159.76) | Borde este, con la entrada de cables hacia el borde |
| Potencia | `J_SWITCH1` (209.5, 176.0), `J_BAT1` (185.64, 182.5), `J_LED_POWER1` (204.0, 53.0) | La bornera de la batería y la del interruptor son la misma pieza de 5.08 mm. Los condensadores grandes de VBAT C1, C3, C5 y los pequeños forman un grupo compacto en x ≈ 192–204, y ≈ 80–135 (no una columna larga: ver abajo) |
| Agujeros M3 | a 5 mm de cada esquina | |

- **Por qué esta disposición (planaridad de una sola capa).** Cada señal debe llegar a su destino sin cruzar a otra, así que los pines de los headers del ESP32 están en el mismo orden que los destinos alrededor del módulo:
  - **Columna oeste `J1`, de arriba abajo:** 3V3, los 3 XSHUT, SCL, SDA (van al norte, separados 1 mm, **entre la cámara y el ESP32**, luego al este por encima de la cámara hasta los sensores a lo largo del borde delantero), las 2 líneas de UART (al norte, dentro de las columnas de la cámara), las 3 líneas de LED (al suroeste, hacia los pares de agujeros de los LED) y el pin 5V (al sur).
  - **Columna este `J3`:** las 7 señales del TB6612, pistas horizontales rectas hasta el driver.
  - La pista de 5V de la cámara va desde el MP1584 al norte por el lado oeste hasta y = 140, luego al este hasta x = 138 y al norte hacia la cámara, **por fuera** (al oeste) de los dos carriles de UART (x = 140.5 y 142.5). Ambos entran a la cámara por su interior; los carriles de UART giran al oeste hacia los pines 4 y 6, y el carril de 5V hacia el pin 1.
  - El 3V3 para el pin VCC del TB6612 cruza el zócalo del ESP32 **bajo el módulo** y pasa entre los pads 9 y 10 de `J3` con 0.3 mm; la misma red sube luego al norte en x = 190.5 y va al oeste por el borde superior hasta el pin 1 de los conectores de sensores. Por eso los condensadores grandes y `J_LED_POWER1` se mantienen en un grupo compacto en x ≥ 192 y no en una columna larga: una columna larga de +VBAT_SW en el lado este encerraría el pin VCC.
  - La franja inferior bajo el ESP32 (unos 7 mm) lleva la pista +VBAT_SW (1.2 mm) desde la bornera de la batería hasta el MP1584; D1, C9 y C10 están al oeste de `J1`, junto al pin 5V.
  - **Por qué la cámara no está exactamente centrada:** una cámara en el centro exacto necesita que sus pistas de 5V y UART lleguen desde el sur, y las pistas de los sensores (que van al norte entre la cámara y el ESP32) tendrían que cruzarlas. En una sola capa eso es imposible sin un puente. Por eso la cámara se movió 14.5 mm a la izquierda del centro, lo que deja libre el espacio entre la cámara y el ESP32 para las pistas de los sensores.
- C11 (470 µF de la cámara) está al oeste de los carriles de la cámara, en el corredor bajo la cámara.
- Serigrafía: contornos de módulo para el ESP32-S3 (línea del extremo de la antena), la ESP32-CAM y el MP1584 (su propio footprint), contorno del TB6612 en `F.Fab`, y las etiquetas de la Sección 15 (`^ FRONT` arriba, `REAR (USB)` abajo). Los designadores de referencia quedan en `F.Fab`.

**Método de enrutado.** Las herramientas MCP de KiCad no estaban disponibles, así que el trabajo se hizo con scripts (API de Python de KiCad 10) y **Freerouting 2.1.0**:
1. La colocación se generó con un script (posiciones fijas de los módulos más una búsqueda voraz para las piezas pasivas).
2. Los carriles de UART de la cámara, el carril de 5V de la cámara, las cinco líneas de sensores, los buses de sensores y el largo camino del 3V3 se trazaron **a mano** (cables fijos) porque el autoenrutador no pudo encontrarlos solo; Freerouting enrutó todo lo demás **en una sola capa**, con GND enrutado como una red normal, separación de 0.2 mm y ancho base de pista de 0.3 mm.
3. Las pistas de GND que no hacían falta se eliminaron una por una, mientras el relleno siguiera llegando a todos los pads de GND (hasta quedar en 4 segmentos cortos de GND).
4. Toda pista que se pudo ensanchar a 0.4 mm sin romper la separación de 0.2 mm se ensanchó.
- Los scripts no son parte del repositorio. Repetir la distribución significa repetir este procedimiento en KiCad (o con Freerouting).

**Resultado del enrutado (placa final):**

| Elemento | Valor |
|---|---|
| Pistas | 167, todas en `B.Cu`, 0 vías, 0 puentes de cable |
| Anchos | 0.3 mm × 5 (37 mm), 0.4 mm × 64, 0.6 mm × 44, 0.8 mm × 4 (GND), 1.2 mm × 50 |
| Clases de red | `PWR_BAT` 1.2 mm (`+VBAT_SW`, `SW_BAT`, `M_*`), `PWR_LOW` 0.6 mm (`+5V`, `+5V_ESP`), `GND` 0.8 mm, `Default` 0.4 mm |
| Relleno | `GND_PLANE` en `B.Cu`, una zona, alivios térmicos, sin islas aisladas |
| Brocas | 0.8, 0.95, 1.0, 1.1, 1.3, 1.8 mm (agujeros de pads) y 4 × 3.2 mm (M3) |
| DRC (`kicad-cli pcb drc`, zonas rellenadas, reglas personalizadas) | **0 errores, 0 elementos sin conectar**. Queda una advertencia de serigrafía (el contorno del zócalo del ESP32 toca la marca "K" de D1) |
| ERC | 0 errores, 0 advertencias |
| Paridad con el esquemático | Las redes de la PCB coinciden con el esquemático pin por pin (173 pads, 0 discrepancias); el reporte de paridad de KiCad solo muestra advertencias del prefijo del nombre de red y de campos |

**Cosas que revisar a ojo antes de grabar (el DRC no las puede juzgar):**
- La pista `+3V3` hacia el pin VCC del TB6612 pasa **entre dos pads del header `J3` del ESP32** con 0.3 mm (y corre bajo el módulo): revísala por cortes al grabar. La pista +VBAT_SW (1.2 mm) corre por la franja trasera, cerca del borde de la placa (0.5 mm de separación).
- Las pistas de 0.3 mm (I2C, UART, señales de LED) van en haces apretados con espacios de 0.2 mm. Son las más propensas a romperse o ponerse en corto al grabar. Imprime la página del cobre, mírala a contraluz y considera una exposición un poco más larga.
- Pistas de I2C/UART frente a las pistas de los motores y las zonas de exclusión de antena (Sección 14). Las pistas de los motores están todas en el lado este de la placa.
- Cualquier pista de 0.3 mm que pase **entre pads de header** (los pads miden 1.8 mm de ancho con paso de 2.54 mm, así que cada espacio es de 0.74 mm y una pista de 0.3 mm conserva 0.22 mm de separación a cada lado). Mira las filas de headers del ESP32-S3 y del TB6612.
- Alturas de los condensadores (Sección 14), el espacio libre alrededor del extremo USB del ESP32, y que el chasis deje salir el cable USB por el borde trasero.

**Pendiente:** la revisión mecánica en 2D con medidas reales (Sección 4.3/Sección 17) y los puntos abiertos de la Sección 20.

### Etapa 3: chasis impreso y actualización del cableado (archivos actualizados; la PCB en sí NO cambió)

El chasis se imprimió en 3D (ver Sección 4.1) y cambiaron tres cosas en cómo se usa la PCB. **La placa que se fabricó es la misma: el cobre, los footprints, las posiciones y las pistas no se tocaron.** Solo cambiaron los modelos 3D, las descripciones de las piezas y los documentos.

1. **Los 4 LED se sueldan directamente en la PCB**: los 3 de proximidad (que ya lo estaban) y ahora también el **LED verde de encendido** (`J_LED_POWER1`, R1 = 1.5 kΩ). No hay header ni cable Dupont para ningún LED.
2. **La misma bornera de tornillo de 5.08 mm se usa para el interruptor y para la batería**, así que el robot tiene **4 borneras de tornillo** (motor izquierdo, motor derecho, interruptor, batería). La bornera de la batería se suelda en los pads de `J_BAT1`, que conservan el footprint XT30UPB-M (paso 5.0 mm, agujeros de 1.8 mm, pads de 3.0 mm): los pines de 5.08 mm de la bornera caben en los agujeros de 1.8 mm con 0.08 mm de diferencia.
3. **Chasis impreso** (Sección 4): 144.1 mm de largo, ruedas de 63 mm, rueda loca W420, batería atrás, caja de lastre, torre del interruptor, arco de la cámara con un flex estándar de ~2 cm, PCB a 10 mm de la bandeja sobre columnas impresas con tornillos M3 × 25.

Qué se cambió en los archivos de KiCad:

| Archivo | Cambio |
|---|---|
| `auxi.kicad_pcb` | **Solo modelos 3D.** `J_LED_FRONT1` / `J_LED_LEFT1` / `J_LED_RIGHT1` / `J_LED_POWER1`: el modelo del header de pines se reemplazó por un LED de 5 mm de color (`auxi.3dshapes/LED_D5.0mm_Red.wrl`, `_Yellow.wrl`, `_Green.wrl`; desplazamiento `0 −1.27 0` para que el LED quede entre los dos pads; 8.6 mm de alto). `J_BAT1`: el modelo del XT30 se reemplazó por el modelo de la bornera Phoenix MKDS de 5.08 mm (desplazamiento `−0.04 0 0` para repartir la diferencia de paso de 0.08 mm), y su valor ahora es `BAT 2S (bornera)`. |
| `auxi.kicad_sch`, `auxi.kicad_sym` | **Sin cambio eléctrico** (mismas redes y pines). La descripción de `Conn_LED` ahora dice que el LED se suelda en los agujeros, el valor de `J_BAT1` es `BAT 2S (bornera)` y las notas del bloque de alimentación dicen lo mismo. |
| `auxi.3dshapes/` | Nuevo: los tres modelos de LED (VRML, unidades de 0.1 pulgada, como espera KiCad). |
| `docs/` | Regenerados con KiCad 10.0.7: PDF del esquemático, BOM, los tres renders 2D y la página 1 del PDF a 1:1 (las páginas 2 y 3, el cobre, no cambian porque el cobre no cambió). Dos renders 3D nuevos muestran los nuevos modelos de LED y bornera. |

**Verificación hecha en la Etapa 3 (KiCad 10.0.7, `kicad-cli`):**

- ERC: **0 violaciones**.
- DRC (zonas rellenadas, reglas personalizadas, paridad con el esquemático): **0 errores, 0 elementos sin conectar, 1 advertencia de serigrafía** (la misma de antes). El resultado es idéntico al del commit anterior; la única diferencia de paridad es el nuevo texto `Description` de los 4 símbolos de LED.
- Render 3D: los 4 LED quedan entre sus dos pads, la bornera de la batería queda sobre los antiguos pads del XT30 con la entrada de cables hacia el borde trasero, y nada sale del contorno de la placa. El texto de serigrafía `BAT-` / `BAT+` sobre `J_BAT1` queda en parte tapado por el cuerpo de la bornera, pero las marcas `-` y `+` a cada lado de los pads siguen visibles.
- Renders 2D regenerados: `auxi_pcb_bottom.png` es idéntico píxel a píxel al anterior (el cobre no cambió).
- Nota sobre VRML para los modelos de LED: KiCad descarta en silencio un modelo que tenga un campo `ambientColor` (el campo VRML correcto es `ambientIntensity`).

**Limitación:** la copia en la biblioteca `auxi.pretty/PinHeader_1x02_P2.54mm_Vertical.kicad_mod` sigue apuntando al modelo 3D del header de pines, porque sus pads son los correctos para los LED y no se permitió cambiar el footprint. Los modelos de LED viven en el archivo de la PCB. Si alguien restablece esos 4 footprints desde la biblioteca ("Update footprint from library"), vuelve el modelo del header y hay que asignar de nuevo el modelo del LED.


### Etapas posteriores

- Verificación mecánica (Sección 17) una vez que se midan las piezas reales.

---

## Sección 19. Entregables y documentación

**Entregables de la Etapa 1:**

- Proyecto de KiCad `auxi` (`.kicad_pro`) con el esquemático completo (`.kicad_sch`) y un ERC que pasa.
- Footprints asignados y verificados. Bibliotecas personalizadas solo cuando hagan falta, y verificadas con la pieza real.
- Tablas del mapa de pines del ESP32-S3 y la ESP32-CAM.
- Lista de materiales preliminar, dividida en dos listas:
  - **Componentes de la PCB:** `docs/auxi_bom_pcb.csv`, exportada del esquemático.
  - **Componentes externos:** la tabla de abajo.

**Lista de materiales externos:**

| Cant. | Descripción |
|---|---|
| 1 | Interruptor ≥ 5 A DC y ≥ 12 V DC, **verificado en su hoja de datos** (**interruptor de palanca roscado M6**, agujero de 6.2 mm en la torre del interruptor). Confírmalo con la corriente de bloqueo medida. |
| 1 | LED de 5 mm **verde** (encendido, **soldado en la PCB** en `J_LED_POWER1`, con R1 = 1.5 kΩ) |
| 1 + 1 + 1 | LED de 5 mm: rojo, amarillo y verde (proximidad), **soldados directamente en la PCB** (ver la nota de R5 en la Sección 10) |
| 3 | Módulos VL53L0X GY-530 |
| 3 | Cables JST-XH de 5 pines, ≤ 25 cm (con carcasa hembra XHP-5) |
| 1 | Módulo de cámara OV2640 u OV3660 estándar (24 pines, paso 0.5 mm, flex de ~2 cm), pegado en la cuna (16) |
| 2 | Motores TT con caja reductora + 2 × ruedas de 63 mm + 2 soportes de aluminio para TT + 1 rueda loca de bola W420 |
| 2 | Condensadores cerámicos de 100 nF 50 V para los terminales de los motores |
| 1 | Portapilas 2×18650 (con cables sueltos de 18–20 AWG hacia la bornera de tornillo) + 2 celdas 18650 protegidas. No hace falta enchufe XT30 |
| — | Cable de 20 AWG (batería e interruptor) y de 22 AWG (motores) |
| 4 | Tornillos M3 × 25 para las columnas (las columnas impresas 03 y los separadores 15 reemplazan a los separadores comerciales). Las piezas impresas también necesitan sus propios tornillos y tuercas (ver Sección 4.1). |

**Documentación en lenguaje sencillo (`README.md` y este archivo):**

- Arquitectura, alimentación, pines, conectores externos (qué va dónde), sensores, LED que cumplen el reglamento, motores, TB6612FNG, UART, cámara externa y decisiones clave.

**Advertencias de armado (deben aparecer en `README.md`):**

1. Ajusta el MP1584 a 5.0 V con un multímetro **antes** de enchufar cualquier módulo.
2. Revisa la polaridad de la batería.
3. El camino del interruptor lleva toda la corriente del robot: usa el calibre de cable especificado (20 AWG) y la bornera de tornillo, nunca cables Dupont.
4. Quita la ESP32-CAM para programarla.
5. El PWM de los motores nunca debe superar 70 %.
6. Las celdas 18650 deben estar protegidas, o cargarse con un cargador balanceador 2S.

El proyecto **no** está terminado solo porque exista un archivo `.kicad_sch`: debe pasar el ERC y la revisión eléctrica.

---

## Sección 20. Puntos abiertos (detente y pregunta si siguen sin resolverse)

**Necesarios para finalizar el esquemático:**

1. Confirmar que el ESP32-S3-DevKitC-1 N16R8 comprado coincide con la distribución oficial de pines. Si es un clon, pedir fotos de ambas caras. Confirmar también la revisión de su LED RGB y la separación entre sus filas.
2. Distribución real de pines del TB6612FNG comprado (se supuso el orden de SparkFun), del MP1584 (el footprint `auxi:MP1584_Mini_PROVISIONAL` tiene pads estimados) y del GY-530 (nivel de XSHUT; los pull-ups ya están en el módulo, por eso se quitaron R3/R4).
3. Corriente de bloqueo real del motor TT, para validar el driver, el interruptor y los conectores de potencia.
4. Vf del LED verde, para fijar el valor final de R5.
5. Serigrafía de la ESP32-CAM, para confirmar el orden de pines usado.
6. ~~Qué pin del XT30UPB-M es `+`~~ **Resuelto:** pad 2 = `+` (lado plano), pad 1 = `−` (lado biselado). El esquemático se corrigió para coincidir.
7a. Revisión con calibrador de la separación entre filas (la distribución depende de esto, y los pads están fijados a ello): clon del ESP32-S3 (se supuso 22.86 mm), ESP32-CAM (se supuso 22.86 mm), módulo rojo TB6612 (se supuso 17.78 mm) y posiciones de los agujeros del MP1584 (estimadas desde una foto). Confirmar también qué extremo de la ESP32-CAM tiene los pines del header.

**Necesarios para la PCB (etapas posteriores):**

7. ~~¿El chasis de 150 × 150 mm incluye las ruedas?~~ **Resuelto:** el robot mide 144.1 mm de largo (límite 150 mm) con ruedas de 63 mm a los costados de la placa de 110 mm. El ancho no se conoce: falta confirmarlo con el robot real y las reglas.
8. ~~Orientación de la PCB de 110 × 140 mm~~ **Resuelto con la foto del usuario:** el lado de 110 mm es el frente (entre las ruedas). El robot mide 144.1 mm de largo, así que la placa de 140 mm casi lo ocupa: falta comprobar en el chasis real que libra la pieza delantera y la rueda loca, que el cable USB del borde trasero tiene salida, y que el arco se apoya en las dos esquinas delanteras como está dibujado.
9. Medidas reales del motor TT y del portapilas, y la altura disponible en el nivel inferior. (Ruedas: 63 mm.)
10. ~~Ubicación de los LED de proximidad~~ **Decidido por el usuario:** soldados directamente en la PCB (lado oeste). No hay carcasa, así que los LED se ven. El **LED de encendido ahora también está soldado en la PCB**, adelante a la derecha: falta comprobar solo que el LED verde de encendido no se confunda con el LED verde de proximidad.
11. ~~Longitud del flex de la cámara~~ **Resuelto:** un módulo estándar con su flex de ~2 cm, en la cuna del arco.
12. Altura de los condensadores electrolíticos que se comprarán (C1, C3, C11), y una pieza de perfil bajo de 6.3 × 7 mm para C9 (queda junto a la zona del conector USB del ESP32).
13. **Agujeros pasacables en la PCB** (propuesta del diseño del chasis, Sección 4.1): sensores (X 38, Y 45, Ø 8 mm) y potencia (X 28, Y −48, Ø 10 mm). **No** están en la placa fabricada. Decidir si los cables pasan por el borde de la placa o si esos dos agujeros se taladran a mano (y se agregan a KiCad, luego DRC). El agujero de los LED de esa propuesta ya no hace falta.
14. **Conexión invertida de la batería:** la batería ahora es una bornera de tornillo, así que no tiene guía mecánica. Confirmar el marcado de polaridad en la placa real (`-` en el pad izquierdo, `+` en el derecho) y marcar los cables.
15. **Paso de la bornera:** la bornera de la batería tiene pines de 5.08 mm en pads de 5.0 mm. Confirmar que entra sin forzarla (los agujeros de 1.8 mm deberían absorber los 0.08 mm). Confirmar también hacia qué lado mira su entrada de cable (el modelo 3D supone el borde trasero).
16. **Peso:** 762.5 g estimados, el límite es 850 g. Pesar el robot real.

---

## Sección 21. Archivos del repositorio

| Ruta | Notas |
|---|---|
| `auxi.kicad_pro` / `auxi.kicad_sch` / `auxi.kicad_pcb` | Proyecto, esquemático, PCB (una capa, 110 × 140 mm, enrutada) |
| `auxi.kicad_dru` | Reglas DRC personalizadas: anillo del pad ≥ 0.4 mm, agujero ≥ 0.3 mm |
| `auxi.kicad_sym`, `auxi.pretty/` | Bibliotecas del proyecto, cargadas con `sym-lib-table` y `fp-lib-table` mediante `${KIPRJMOD}` |
| `auxi.3dshapes/` | Modelos 3D de los LED de 5 mm (rojo, amarillo, verde; VRML), usados por la PCB mediante `${KIPRJMOD}`. Agregados en la Etapa 3 |
| `auxi.pretty/MP1584_Mini_PROVISIONAL.kicad_mod` | **Pads estimados.** Verificar con el módulo real antes de grabar |
| `auxi.pretty/PinSocket_1x08…`, `PinSocket_1x22…`, `PinHeader_1x02…`, `JST_XH_B5B-XH-A…` | Copias de los footprints estándar con pads más grandes (pad = broca + 0.8 mm). El esquemático usa estas |
| `docs/auxi_esquematico.pdf`, `docs/auxi_bom_pcb.csv` | Regenerar tras cambios en el esquemático |
| `docs/auxi_pcb_real_scale.pdf` | 3 páginas a 1:1 (carta): vista superior, cobre `B.Cu`, cobre `B.Cu` en espejo. Regenerar tras cambios en la PCB |
| `docs/auxi_pcb_top.png`, `docs/auxi_pcb_bottom.png`, `docs/auxi_pcb_placement.png` | Renders 2D. Regenerar tras cambios en la PCB |
| `docs/auxi_pcb_3d_top.png`, `docs/auxi_pcb_3d_iso.png` | Renders 3D (`kicad-cli pcb render`) que muestran los modelos 3D de las piezas |
| `auxi_drc_violations.json` | Último reporte DRC |
| `auxi.kicad_prl` | Ajustes locales de vista; ignorado por git |

**Representación de los módulos en el esquemático:**

- Cada fila de pines de un módulo es su propio símbolo con un footprint estándar de header hembra.
- No hay una separación oficial publicada entre filas y los clones varían, así que las filas se colocan en la etapa de PCB usando medidas con calibrador.
- Los **números** de pin coinciden con la posición física en el header, mientras que el dibujo del símbolo agrupa los pines por función.

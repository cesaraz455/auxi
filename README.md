# auxi — Placa controladora del robot de fútbol

**auxi** es un pequeño robot de fútbol: un proyecto universitario de primer semestre. Este repositorio contiene el diseño en KiCad de su única PCB personalizada.

> **Estado: Etapa 3 — actualización del chasis impreso y del cableado.** La placa es el mismo diseño de una sola capa que ya se fabricó (**no** cambió). Lo que cambió es cómo se usa: los 4 LED están soldados en la placa, la batería y el interruptor usan la misma bornera de tornillo que los motores, y el chasis se imprimió en 3D (ver la [Sección 5](#5-el-chasis-impreso)). El esquemático pasa el ERC sin errores ni advertencias, y la PCB pasa el DRC con **0 violaciones y 0 elementos sin conectar**. El diseño sigue siendo **preliminar** hasta que se midan los módulos comprados (ver la [Sección 16](#16-puntos-abiertos)).

---

## 1. Para qué sirve auxi

**auxi** juega fútbol en una competencia universitaria de robots. Una persona lo maneja con un control Xbox por Bluetooth Low Energy, y su pieza delantera guía una pelota mientras la empuja. Las reglas de la competencia condicionan el diseño: el robot debe medir como máximo 150 mm y pesar como máximo 850 g, tres LED de colores deben encenderse cuando algo esté a menos de 10 cm de cada uno de sus tres sensores de distancia, y no debe interferir con otros equipos (por ejemplo con su video por Wi-Fi).

También es un proyecto de aprendizaje de un equipo de primer semestre, así que se prefieren las soluciones simples, robustas y fáciles de armar. Este repositorio contiene la parte electrónica: **una PCB personalizada** que lleva los módulos controladores, el driver de motores, el regulador de voltaje, todos los conectores y los 4 LED. El chasis se imprime aparte (Sección 5). El firmware ya está escrito y vive en un **repositorio de GitHub separado**; usa el mapa de pines de la Sección 8, así que cualquier cambio en ese mapa debe hacerse en los dos repositorios.

**Contenido**

| Sección | Título |
|---|---|
| 1 | Para qué sirve auxi |
| 2 | El robot |
| 3 | Qué lleva la PCB |
| 4 | La placa: una capa de cobre, soldada por abajo |
| 5 | El chasis impreso |
| 6 | Por qué se movieron los pines |
| 7 | Arquitectura de alimentación |
| 8 | Mapa de pines (GPIO) |
| 9 | LED (reglas de la universidad y LED de encendido) |
| 10 | Reglas de fabricación de la placa casera |
| 11 | Armado de la placa |
| 12 | Estructura del repositorio |
| 13 | Abrir el proyecto |
| 14 | Advertencias de armado |
| 15 | Hoja de ruta |
| 16 | Puntos abiertos |

---

## 2. El robot

- **Tracción:** 2 motores amarillos TT con caja reductora y 1 rueda loca trasera de bola, cuya esfera es **metálica**.
- **Chasis:** impreso en 3D, de 144.1 mm de largo (límite 150 mm). El frente es una pieza impresa **en V o curva que guía la pelota**. Ver la [Sección 5](#5-el-chasis-impreso).
- **Dos niveles:**
  - **Nivel inferior (chasis):** base, motores con ruedas de 63 mm, rueda loca, batería (atrás, detrás de una compuerta), lastre y cableado.
  - **Nivel superior:** una **PCB de 110 × 140 mm** sobre cuatro columnas impresas (tornillos M3 × 25), a 10 mm de la bandeja, encima de las baterías y los motores.
- **MCU principal:** ESP32-S3-DevKitC-1 N16R8. Lee los sensores, maneja los motores y los LED, y recibe comandos de un **control Xbox por Bluetooth Low Energy**. El ESP32-S3 no tiene Bluetooth clásico.
- **Cámara:** ESP32-CAM AI Thinker. Transmite video en vivo por Wi-Fi a un celular y muestra las distancias que recibe del ESP32 principal por UART. La cámara OV2640 es un módulo estándar con su flex de ~2 cm, sujeta en una cuna impresa en el **arco de la cámara**, en el frente del chasis.
- **Sensores:** 3 sensores de tiempo de vuelo VL53L0X (módulos GY-530) en soportes impresos en el frente: FRONT (0°), LEFT (+45°) y RIGHT (−45°).
- **LED de proximidad:** 3 LED soldados en la placa, cuyos colores fija el reglamento de la universidad. Cada LED se enciende cuando su sensor lee **≤ 100 mm**.

## 3. Qué lleva la PCB

| En la PCB (módulos enchufables y piezas) | Fuera de la PCB (se conecta por un conector) |
|---|---|
| ESP32-S3-DevKitC-1 (2 headers hembra de 1×22) | Batería 2S de 18650 (**bornera de tornillo** `J_BAT1`) |
| ESP32-CAM (2 headers hembra de 1×8) | Interruptor de encendido (bornera de tornillo `J_SWITCH1`, en la torre del interruptor) |
| Módulo TB6612FNG (2 headers hembra de 1×8) | 2 motores TT (borneras de tornillo) |
| Módulo reductor MP1584EN mini (5.0 V) | 3 sensores VL53L0X (JST-XH de 5 pines) |
| Condensadores, resistencias, diodo Schottky | Cámara OV2640 (cable flex hacia la ESP32-CAM) |
| **4 LED de 5 mm, soldados directamente en la placa: 3 de proximidad + 1 verde de encendido** | |
| Todos los conectores (**4 borneras de tornillo idénticas**: 2 motores, interruptor, batería) | |

## 4. La placa: una capa de cobre, soldada por abajo

La placa está pensada para **grabado casero y taladrado a mano**, porque la mayoría de los talleres locales no metalizan los agujeros.

- **Una sola capa de cobre: `B.Cu` (la cara inferior).** Todas las pistas están abajo. El resto de la cara inferior es un **plano de GND** (relleno de cobre) que reemplaza las pistas de tierra.
- **Todas las piezas son de agujero pasante (THT).** Van en la **cara superior** (la cara sin cobre). Sus patas atraviesan los agujeros y se **sueldan por abajo**, en el lado del cobre. Ningún agujero necesita metalizado, porque no hay nada que conectar en la cara superior.
- **Sin vías y sin puentes de cable.** Las 167 pistas están todas en `B.Cu` y ninguna cruza a otra.
- **Tamaño: 110 × 140 mm**, 4 agujeros M3 a 5 mm de cada esquina, esquinas redondeadas de 2 mm.
- **Orientación (dada por el usuario, ver la foto del chasis):** el **lado de 110 mm es el FRENTE del robot** (queda entre las dos ruedas) y el lado de 140 mm va de adelante hacia atrás. En KiCad el **borde superior es el FRENTE del robot**, el inferior es la PARTE TRASERA, el izquierdo es el lado IZQUIERDO del robot y el derecho es el lado DERECHO. La ESP32-CAM está al frente (con la cámara mirando hacia adelante) y el ESP32-S3 está atrás.
- La capa `F.Cu` no tiene nada de cobre. Una zona de exclusión cubre toda la cara superior para que no pueda aparecer ninguna pista ni relleno por error. Arriba solo se ven los pads.

![Vista superior](docs/auxi_pcb_top.png)

![Vista inferior (lado del cobre)](docs/auxi_pcb_bottom.png)

Vista 3D (los cuatro LED y las cuatro borneras tal como están armados):

![Vista 3D](docs/auxi_pcb_3d_iso.png)

### 4.1 La distribución en un párrafo

Vista en KiCad (frente del robot = borde superior):

- **Frente (arriba):** la **ESP32-CAM** está al frente, casi centrada (su centro queda 14.5 mm a la izquierda del centro de la placa), con su antena y su conector del flex en el borde superior, para que la cámara mire hacia adelante. Los tres **conectores de sensores** van uno al lado del otro a lo largo del borde superior, a la derecha de la cámara (LEFT, FRONT, RIGHT de izquierda a derecha). Una cámara perfectamente centrada no es posible con una sola capa de cobre, porque las pistas de 5 V y de UART de la cámara tendrían que cruzar las pistas de los sensores.
- **Atrás (abajo):** el **ESP32-S3** queda en el centro del borde trasero con su **extremo USB a solo 1 mm del borde inferior** (pedido del usuario: máximo 2 mm). El conector USB-C sobresale del borde, así que el cable se enchufa sin ningún obstáculo.
- **Lado este:** el TB6612FNG está justo al lado de la fila de pines `J3` del ESP32-S3 (señales de control cortas), con las borneras de los motores en el borde derecho y los condensadores grandes, el LED verde de encendido (adelante a la derecha) y las borneras de batería e interruptor en el lado este y la esquina inferior derecha.
- **Lado oeste:** el MP1584, los 3 LED de proximidad (soldados directamente en la placa) y el condensador grande de la cámara.
- Las zonas libres entre los grupos son plano de GND.

## 5. El chasis impreso

El chasis está impreso en 3D y sus archivos viven en el proyecto del chasis, no en este repositorio. Esto es lo que importa para la placa:

| Elemento | Chasis impreso |
|---|---|
| Tamaño y peso | **144.1 mm de largo** (límite 150 mm). Unos **762.5 g** estimados con 3 placas de lastre y la bandeja estándar; objetivo 800 g, **límite reglamentario 850 g**. Hay que pesar el robot real |
| Tracción | 2 motores TT en soportes de aluminio, **ruedas de 63 mm**, rueda loca trasera de bola W420 con una **esfera metálica** |
| Batería | 2 celdas 18650 protegidas en un portapilas **atrás**, detrás de una compuerta, para cargarlas sin quitar la placa |
| Montaje de la PCB | 4 columnas impresas con tornillos **M3 × 25**; separadores impresos mantienen la placa **a 10 mm de la bandeja** para que los cables pasen por debajo |
| Cámara | El **arco de la cámara** se apoya en las dos esquinas delanteras de la PCB. Su cuna sostiene una cámara estándar de 24 pines (lente al frente, inclinada 25° hacia abajo) con su **flex de ~2 cm**, conectado a la ESP32-CAM que queda debajo. El flex largo del plan inicial ya no hace falta |
| Interruptor | Interruptor de palanca M6 (≥ 3 A, ≥ 12 V DC) en la **torre del interruptor**, en un costado, con la palanca hacia arriba. Sus cables de 20 AWG van directo a `J_SWITCH1` |
| Sensores | Los módulos GY-530 van atornillados a soportes impresos en el frente. Sus cables van **soldados al módulo por detrás** (sin conector en el sensor) y se enchufan en los conectores JST-XH del borde delantero de la placa |
| Frente | Pieza impresa en V o curva que guía la pelota (se recomienda la curva) |

**Cables.** Suben desde el nivel inferior por las aberturas de la bandeja hasta el espacio de 10 mm y llegan a su conector en la placa. Nunca dejes cables colgando sobre las ruedas ni delante del sensor frontal o de la cámara, y amárralos cada 3–4 cm.

| Cable | Va a | Calibre |
|---|---|---|
| Batería | bornera `J_BAT1` (atrás a la derecha) | 18–20 AWG. **Revisa la polaridad** |
| Motores | borneras `J_MOTOR_LEFT1` / `J_MOTOR_RIGHT1` (borde derecho) | 20–22 AWG |
| Interruptor | bornera `J_SWITCH1` | 20 AWG, nunca Dupont |
| Sensores | 3 × JST-XH en el borde delantero | 25 cm o menos, I2C a 100 kHz |

**Agujeros pasacables.** El diseño del chasis propone tres agujeros en la placa (sensores, potencia, LED). **No se agregaron: la placa que se fabricó no cambió.** El agujero de los LED ya no hace falta, porque los LED ahora están soldados en la placa. Ver la [Sección 16](#16-puntos-abiertos).

**Para programar la ESP32-CAM:** quita los dos tornillos M3 × 25 de las patas del arco y levanta el arco (solo quedan ~3 mm sobre el extremo delantero del módulo), abre el seguro del flex y saca el módulo.

## 6. Por qué se movieron los pines

Una sola capa no puede cruzar dos pistas. Para evitar cualquier cruce (y cualquier puente de cable), los GPIO se **reasignaron** para que las señales salgan de los headers del ESP32 en el mismo orden que sus destinos. Esto se hizo antes de escribir el firmware, así que no costó nada; el firmware (repositorio separado) ya usa el mapa. Ver la tabla en la [Sección 8](#8-mapa-de-pines-gpio).

El mapa cambió **una vez más** cuando se fijó la orientación de la placa (lado de 110 mm = frente): los pines de I2C y los de UART de la cámara pasaron al orden que necesita la nueva distribución, y los pines **TX y RX de la UART se intercambiaron** (`GPIO18` ahora es el pin de transmisión del ESP32). Revisa siempre la tabla de abajo.

## 7. Arquitectura de alimentación

```
BAT 2S (6.0–8.4 V) ─ bornera de tornillo ─ interruptor externo ─ +VBAT_SW
+VBAT_SW ─┬─ R1 1.5 kΩ ─ LED verde de encendido (soldado en la placa)
          ├─ condensadores grandes ─ TB6612 VM (motores)
          └─ MP1584 ─ +5V ─┬─ D1 Schottky ─ +5V_ESP ─ pin 5V del ESP32-S3
                           └─ pin 5V de la ESP32-CAM
Pin 3V3 del ESP32-S3 ─ +3V3 ─ sensores VL53L0X, lógica del TB6612 (los pull-ups de I2C están en los módulos GY-530)
```

- Nada se conecta a la batería antes del interruptor. Con el interruptor apagado, todo el robot está apagado.
- D1 evita que el puerto USB y el robot se alimenten entre sí mientras programas la placa.
- El firmware **debe** limitar el PWM de los motores a **70 %**, lo que da unos 5.9 V promedio con 8.4 V.

## 8. Mapa de pines (GPIO)

**Fila izquierda de pines (`U_ESP_J1`)**

| Señal | GPIO | Pin del header |
|---|---|---|
| XSHUT_RIGHT | GPIO4 | J1-4 |
| XSHUT_FRONT | GPIO5 | J1-5 |
| XSHUT_LEFT | GPIO6 | J1-6 |
| SCL | GPIO7 | J1-7 |
| SDA | GPIO15 | J1-8 |
| UART_CAM_TO_ESP (**RX** del ESP32) | GPIO17 | J1-10 |
| UART_ESP_TO_CAM (**TX** del ESP32) | GPIO18 | J1-11 |
| LED_FRONT | GPIO8 | J1-12 |
| LED_LEFT | GPIO9 | J1-15 |
| LED_RIGHT | GPIO10 | J1-16 |

**Fila derecha de pines (`U_ESP_J3`)**

| Señal | GPIO | Pin del header |
|---|---|---|
| PWMB | GPIO1 | J3-4 |
| BIN2 | GPIO2 | J3-5 |
| BIN1 | GPIO42 | J3-6 |
| STBY | GPIO41 | J3-7 |
| AIN1 | GPIO40 | J3-8 |
| AIN2 | GPIO39 | J3-9 |
| PWMA | GPIO21 | J3-18 |

- I2C **ya no** está en los pines por defecto de Arduino: llama a `Wire.begin(15, 7)` (SDA = GPIO15, SCL = GPIO7).
- La UART de la cámara tampoco está en los pines por defecto de `Serial1`: llama a `Serial1.begin(115200, SERIAL_8N1, 17, 18)` (RX = GPIO17, TX = GPIO18).
- Pines libres: GPIO11, 12, 13, 14 y 16.
- Se siguen evitando a propósito: pines de arranque (0, 3, 45, 46), USB (19, 20), consola (43, 44), PSRAM octal (35, 36, 37), LED RGB (38, 48) y 47.
- R2 (10 kΩ a GND) sigue manteniendo `STBY` en bajo mientras el ESP32 arranca, así que los motores quedan apagados.

## 9. LED (reglas de la universidad y LED de encendido)

| Sensor | Color del LED | Posición en la placa (vista superior, frente arriba) | Pads |
|---|---|---|---|
| FRONT | Rojo | arriba | `J_LED_FRONT1` |
| LEFT | Amarillo | en medio | `J_LED_LEFT1` |
| RIGHT | Verde | abajo | `J_LED_RIGHT1` |

Un LED está **encendido** cuando su sensor lee ≤ 100 mm. Está **apagado** cuando la lectura es mayor a 100 mm, inválida o fuera de rango. El firmware puede promediar lecturas, pero no debe usar histéresis que mantenga un LED encendido por encima de 100 mm.

**Decisión (del usuario): los tres LED de proximidad son LED de 5 mm soldados directamente en la placa**, en los pares de agujeros marcados `J_LED_FRONT1`, `J_LED_LEFT1` y `J_LED_RIGHT1` (sin header y sin cables Dupont). Están apilados en el lado oeste, entre el MP1584 y el ESP32-S3, de arriba abajo: FRONT (rojo), LEFT (amarillo), RIGHT (verde). En cada par de agujeros, el **agujero derecho** (el **pad cuadrado**, junto a la resistencia R3, R4 o R5) es el `+` del LED (ánodo, la **pata larga**) y el **agujero izquierdo** es el `−` (cátodo, la pata corta y el lado plano del LED). Un LED soldado al revés no enciende y no se daña. El robot **no tiene carcasa** (decisión del usuario): la placa queda visible sobre el chasis, así que los LED se ven como exige el reglamento de la universidad y no hacen falta ventanas ni agujeros. Suéldalos a una altura normal.

El **LED de encendido** es distinto. Es un LED **verde** de 5 mm **soldado directamente en la placa** también (decisión del usuario, Etapa 3: antes era un LED externo en un header de 2 pines con cables Dupont), en el par de agujeros `J_LED_POWER1`, adelante a la derecha de la placa, junto a su resistencia `R1` (1.5 kΩ). Misma regla de polaridad que los demás LED: el **agujero derecho, cuadrado**, es `+` (pata larga). `R1` da unos 2.6 mA a 6.0 V y 4.2 mA a 8.4 V con un LED verde clásico (2–3.6 mA con uno brillante). Como el LED de proximidad RIGHT también es verde, verifica que no se puedan confundir (están en lados opuestos de la placa, y el LED de encendido tiene la etiqueta `POWER LED`); si el reglamento exige que los LED indicadores se distingan, cambia el color del LED de encendido.

La vista 3D de la placa muestra los cuatro LED como modelos de 5 mm de colores (8.6 mm de alto), guardados en `auxi.3dshapes/`.

## 10. Reglas de fabricación de la placa casera

Estas reglas se pidieron para que las pistas sobrevivan al grabado y los pads conserven cobre después de taladrar a mano.

| Elemento | Regla | Cómo se hace cumplir |
|---|---|---|
| Ancho de pista | **≥ 0.4 mm**, salvo **0.3 mm** donde una pista pasa entre pads o por un haz apretado | Clases de red en `auxi.kicad_pro` y `auxi.kicad_dru`. KiCad rechaza cualquier valor menor a 0.3 mm |
| Pistas de potencia | `+5V`, `+5V_ESP`: 0.6 mm. Batería, interruptor, VM y motores: 1.2 mm | Clases de red |
| Separación | 0.2 mm entre objetos de cobre | DRC |
| Agujero mínimo | **0.3 mm** (la broca real más pequeña es de 0.8 mm) | Regla DRC `min_hole_0.3mm` |
| Tamaño del pad | **pad = broca + 0.8 mm** en cada agujero con pad (0.4 mm de cobre alrededor) | Regla DRC `pad_ring_0.4mm` |
| Vía | no se usa ninguna. La vía por defecto es pad de 1.1 mm / agujero de 0.3 mm (misma regla de 0.8 mm) | `auxi.kicad_pro` |
| Borde | 0.5 mm de separación del cobre al borde de la placa | DRC |

Resultado en la placa final: 167 pistas, de las cuales **162 miden 0.4 mm o más** y **5 miden 0.3 mm** (37 mm en total). Las pistas de 0.3 mm son las que se cuelan entre filas de pads o van en haces apretados (señales de I2C, UART y LED); el enrutador no pudo ensancharlas sin romper la separación de 0.2 mm.

### 10.1 Tabla de brocas

| Broca (mm) | Pad (mm) | Agujeros | Usada por |
|---|---|---|---|
| 0.80 | 1.6 | 48 | Resistencias y condensadores de hasta 8 mm |
| 0.95 | 1.75 × 1.95 | 15 | Conectores JST-XH de los sensores |
| 1.00 | 1.8 (2.0 para los dos condensadores de 470 µF) | 96 | Headers hembra (ESP32-S3, ESP32-CAM, TB6612FNG), agujeros de los LED (4 pares), pines del MP1584, condensadores de 470 µF |
| 1.10 | 2.2 | 2 | Diodo 1N5819 |
| 1.30 | 2.6 | 6 | Borneras de tornillo de 5.08 mm: motores ×2 e interruptor |
| 1.80 | 3.0 | 2 | Bornera de tornillo de la batería (los pads aún tienen el footprint del XT30, paso de 5.0 mm; los pines de 5.08 mm de la bornera caben en los agujeros de 1.8 mm) |
| 3.20 | sin pad | 4 | Agujeros de montaje M3 (sin metalizar) |

Los pines de los headers son cuadrados de 0.64 mm (0.9 mm en diagonal), así que un agujero de 1.0 mm basta. Revisa las patas de tus propias piezas antes de taladrar: si una pieza es más gruesa que el agujero, agranda ese agujero con la siguiente broca (el pad aún conserva al menos 0.3 mm de anillo).

## 11. Armado de la placa

1. Imprime la **página de B.Cu** de [`docs/auxi_pcb_real_scale.pdf`](docs/auxi_pcb_real_scale.pdf) a **escala 100 %** (sin "ajustar a la página"). La regla de 100 mm bajo la placa debe medir 100 mm.
   - La **página 2** es el cobre **visto desde arriba** (sin espejo). Para transferencia de tóner, apoya la cara impresa contra el cobre. El resultado se lee bien desde el lado del cobre.
   - La **página 3** es el mismo cobre **en espejo**, para métodos donde la hoja queda con la cara impresa hacia arriba.
   - El negro es cobre. Los puntitos blancos en los pads son guías de taladrado.
2. Graba, limpia y taladra (ver la tabla de brocas). Taladra desde el lado del cobre para que los puntos guía centren la broca.
3. Monta los componentes en la **cara superior** y suéldalos **por abajo**. Orden: primero las piezas bajas (resistencias, diodo, condensadores cerámicos), luego los condensadores electrolíticos, luego los zócalos y conectores.
4. La página 1 del PDF muestra dónde va cada pieza (vista superior con serigrafía y referencias).
5. Suelda **al final** los 4 pines del MP1584, después de ajustar su salida (ver las advertencias de abajo).

El **extremo USB** del ESP32-S3 está a solo **1 mm** del borde trasero de la placa, así que el conector sobresale del borde: mantén libre de piezas altas el área a su alrededor y asegúrate de que nada del chasis bloquee el cable atrás (el conector y el cable necesitan unos 15 × 12 mm). Usa un condensador de perfil bajo de **6.3 × 7 mm** para C9 (100 µF). Las pistas largas de `+3V3` y `+VBAT_SW` van por el lado este y la franja inferior de la placa, lejos de las pistas de I2C y UART.

## 12. Estructura del repositorio

| Ruta | Contenido |
|---|---|
| `auxi.kicad_pro` | Proyecto de KiCad, con las clases de red y las reglas de diseño |
| `auxi.kicad_dru` | Reglas DRC personalizadas (anillo del pad y tamaño de agujero) |
| `auxi.kicad_sch` | Esquemático |
| `auxi.kicad_pcb` | PCB de una capa (110 × 140 mm) |
| `auxi.kicad_sym` | Biblioteca de símbolos del proyecto (filas de pines de los módulos, etc.) |
| `auxi.pretty/` | Biblioteca de footprints del proyecto. Aquí se copiaron cuatro footprints estándar con pads más grandes (pad = broca + 0.8 mm). `MP1584_Mini_PROVISIONAL` tiene pads estimados |
| `auxi.3dshapes/` | Modelos 3D de los LED de 5 mm (rojo, amarillo, verde), usados por la PCB |
| `sym-lib-table`, `fp-lib-table` | Tablas de bibliotecas del proyecto |
| `auxi_drc_violations.json` | Último reporte DRC |
| `docs/auxi_esquematico.pdf` | Exportación del esquemático en PDF |
| `docs/auxi_pcb_real_scale.pdf` | Impresiones 1:1: vista superior, cobre, cobre en espejo |
| `docs/auxi_bom_pcb.csv` | Lista de materiales (BOM) de la PCB |
| `docs/auxi_pcb_top.png`, `docs/auxi_pcb_bottom.png`, `docs/auxi_pcb_placement.png` | Renders 2D de la placa |
| `docs/auxi_pcb_3d_top.png`, `docs/auxi_pcb_3d_iso.png` | Renders 3D de la placa |

La especificación completa del diseño (decisiones confirmadas, alimentación, pines, conectores, condensadores, reglas de la PCB y puntos abiertos) está en [`CLAUDE.md`](CLAUDE.md). Es la fuente de verdad de cada decisión de diseño.

## 13. Abrir el proyecto

1. Instala **KiCad 10**. Los archivos usan el formato de archivo de KiCad 10, así que las versiones anteriores no pueden abrirlos.
2. Abre `auxi.kicad_pro`. Las bibliotecas del proyecto (`auxi`) se cargan con los archivos locales `sym-lib-table` y `fp-lib-table` mediante `${KIPRJMOD}`.
3. El esquemático apunta a los cuatro footprints de conectores personalizados `auxi:` (pads más grandes), así que "Update PCB from Schematic" los conserva. No los cambies de nuevo por los estándar de KiCad.

## 14. Advertencias de armado

1. **Ajusta el MP1584 a 5.0 V con un multímetro ANTES de enchufar cualquier módulo.**
2. **Revisa la polaridad de la batería** (`BAT+` / `BAT−`) antes de conectarla. **La batería ahora es una bornera de tornillo simple, no un enchufe XT30 con guía, así que nada impide conectarla al revés.** Marca los cables (rojo +, negro −) y comprueba con un multímetro antes de cerrar el interruptor.
3. **El camino del interruptor lleva toda la corriente del robot.** Usa cable de 20 AWG y la bornera de tornillo. Nunca uses cables Dupont ahí. Lo mismo vale para la batería y los motores.
4. **Quita la ESP32-CAM para programarla**, usando la base programadora ESP32-CAM-MB.
5. **El PWM de los motores nunca debe superar 70 %.**
6. **Usa celdas 18650 protegidas**, o cárgalas fuera del robot con un cargador balanceador 2S. La PCB no tiene BMS ni circuito de carga.
7. **Revisa cada pista con una lupa** antes de soldar: con separaciones de 0.2 mm, un pequeño defecto de grabado puede poner dos pistas en corto. Revisa también que las pistas de 0.3 mm no estén cortadas.
8. **Suelda los 4 LED con la polaridad correcta**: la pata larga (`+`) va en el agujero derecho, cuadrado, de cada par. Un LED soldado al revés no enciende y no se daña.

## 15. Hoja de ruta

- [x] **Etapa 1:** esquemático, footprints y ERC
- [x] **Etapa 2:** distribución de la PCB de una capa de 110 × 140 mm, DRC con 0 violaciones
- [x] **Etapa 3:** chasis impreso, LED soldados en la placa, 4 borneras de tornillo (modelos 3D, textos del esquemático y documentos actualizados; la placa en sí no cambió)
- [ ] Cerrar los puntos abiertos con los módulos reales comprados
- [ ] Revisión de espacio en 2D (vistas superior y lateral) dentro del chasis, con medidas reales
- [x] Firmware (ESP32-S3 principal y ESP32-CAM), usando el mapa de pines de arriba. Vive en un repositorio de GitHub separado

## 16. Puntos abiertos

Estos necesitan las piezas reales que se compraron. Ver [`CLAUDE.md`](CLAUDE.md) (Sección 20) para los detalles.

1. Fotos y medidas de la placa ESP32-S3 si es un clon (separación entre filas, revisión del LED RGB).
2. Orden de pines del módulo TB6612FNG comprado (se supuso el orden de SparkFun). La distribución depende de eso.
3. Foto y medidas del módulo MP1584 (su footprint es provisional).
4. Fotos del GY-530: el nivel de voltaje de XSHUT (los módulos ya incluyen los pull-ups de I2C).
5. Corriente de bloqueo medida del motor TT, para validar el driver, el interruptor y el cableado.
6. Voltaje directo del LED verde, para fijar el valor final de R5.
7. Serigrafía de la ESP32-CAM, para confirmar el orden de pines y qué extremo tiene los pines del header.
8. La foto del chasis muestra que el **lado de 110 mm es el frente** (entre las ruedas) y el lado de 140 mm va de adelante hacia atrás. El robot mide 144.1 mm de largo, así que la placa de 140 mm casi lo ocupa todo. Falta comprobar en el chasis real: ¿la placa libra la pieza delantera y la rueda loca?, ¿el cable USB del borde trasero tiene salida?, ¿el arco de la cámara se apoya en las dos esquinas delanteras como está dibujado?
9. La ESP32-CAM está al frente de la placa, 14.5 mm a la izquierda del centro, y la cámara queda en el arco justo encima de su conector del flex (flex estándar de ~2 cm). Confirma en el robot real que el flex llega a la cuna sin tensión.
10. Altura de los condensadores que se comprarán (C1, C3 y C11 son los más altos).
11. **Agujeros pasacables en la placa** (sensores y potencia, propuestos por el diseño del chasis): decidir si los cables pasan por el borde de la placa o si los agujeros se taladran a mano (y se agregan a KiCad, luego DRC).
12. **Bornera de la batería:** tiene pines de 5.08 mm en los pads de 5.0 mm del antiguo footprint del XT30. Confirmar que entra sin forzarla y hacia qué lado mira la entrada de su cable.
13. **Peso:** 762.5 g estimados, el límite es 850 g.

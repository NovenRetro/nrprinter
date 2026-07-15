# NR Printer v1.0 — Game Boy Printer para ESP32 y SC03h

**NR Printer** convierte un **ESP32 DevKit** en una impresora compatible con **Game Boy Printer**, capaz de recibir imágenes desde **Game Boy Camera** o juegos compatibles, guardarlas en memoria interna y, opcionalmente, imprimirlas en una mini impresora térmica Bluetooth tipo **SC03h / iPrint**.

La Game Boy cree que está hablando con una Game Boy Printer original, pero en realidad el ESP32 actúa como intermediario: recibe los datos por el puerto Link, los guarda en LittleFS y los puede enviar por BLE a una impresora térmica china compatible.

---

## ✨ Características

- Compatible con **Game Boy**, **Game Boy Color** y **Game Boy Camera**
- Emula el protocolo de **Game Boy Printer**
- Recibe capturas por cable Link
- Guarda las imágenes en la memoria interna del ESP32 usando **LittleFS**
- Funciona aunque la impresora térmica esté apagada o no esté disponible
- Auto-print opcional hacia impresoras térmicas **SC03h / iPrint**
- Detección automática de impresoras compatibles por BLE
- Vinculación automática de impresora encontrada
- Permite olvidar la impresora vinculada desde la web
- Galería web local desde el propio ESP32
- Vista previa BMP de las capturas
- Descarga de capturas en BMP desde la web
- Reimpresión manual desde la galería web
- Borrado individual de capturas
- Borrado completo de galería
- Configuración de escala, oscuridad térmica, tramado de grises e inversión de colores
- Firmware instalable desde navegador mediante **ESP Web Tools**
- No requiere app externa ni servidor externo para funcionar

---

## 🧠 Cómo funciona

El flujo principal del proyecto es:

```text
Game Boy / Game Boy Camera
↓
Puerto Link
↓
Level shifter 5V ↔ 3.3V
↓
ESP32 DevKit
↓
Memoria interna LittleFS
↓
BLE
↓
Impresora térmica SC03h
```

El ESP32 se comporta como una Game Boy Printer real. Cuando la Game Boy envía una imagen, el firmware recibe los paquetes, valida el checksum, reconstruye la captura y la guarda como archivo interno `.GBP`.

Luego, según la configuración:

- si **Auto-print está OFF**, la imagen queda guardada solamente;
- si **Auto-print está ON** y la impresora está disponible, la imagen se imprime automáticamente;
- si **Auto-print está ON** pero la impresora no está disponible, la imagen se guarda igual y no se pierde.

---

## 🖨️ Compatibilidad de impresora térmica

Este firmware fue desarrollado y probado con una impresora térmica tipo:

```text
SC03h-D5D0
```

Usa el protocolo BLE de impresoras tipo iPrint / cat printer:

```text
Servicio BLE: AE30
TX Write: AE01
RX Notify: AE02
Comandos: 51 78
Ancho térmico: 384 px
```

### Importante

No es compatible con cualquier impresora térmica Bluetooth genérica.

Debe ser una impresora compatible con el protocolo:

```text
AE30 / AE01 / AE02
```

En la práctica, debería funcionar con impresoras del mismo modelo o familia que se anuncien como:

```text
SC03h
SC03H
SC03h-XXXX
```

---

## 🎮 Compatibilidad Game Boy

Probado con:

- Game Boy Camera
- Game Boy / Game Boy Color mediante cable Link compatible
- Impresión estándar del protocolo Game Boy Printer

El firmware responde a los comandos principales del protocolo:

| Comando | Función |
|---|---|
| `INIT` | Inicialización |
| `DATA` | Recepción de imagen |
| `PRINT` | Solicitud de impresión |
| `INQUIRY` | Consulta de estado |
| `BREAK` | Cancelación / interrupción |

---

## 🛠️ Hardware necesario

- ESP32 DevKit
- Cable Link de Game Boy / Game Boy Color
- Level shifter bidireccional 5V ↔ 3.3V
- Impresora térmica Bluetooth compatible tipo SC03h
- Cables Dupont o PCB propia
- Fuente USB para alimentar el ESP32

---

## 🔌 Esquema de conexiones

> El puerto Link de Game Boy trabaja a 5V y el ESP32 a 3.3V.  
> Por eso se recomienda usar un **level shifter** entre ambos.

| Game Boy Link | Función | ESP32 |
|---|---|---|
| Pin 1 | VCC 5V | HV del level shifter |
| Pin 2 | SOUT | GPIO 26 |
| Pin 3 | SIN | GPIO 25 |
| Pin 5 | CLK | GPIO 27 |
| Pin 6 | GND | GND común |

### Importante

- El pin VCC del cable Link se usa solo para alimentar el lado **HV** del level shifter.
- No alimentes el ESP32 desde el puerto Link de la Game Boy.
- El ESP32 debe alimentarse por USB o una fuente externa adecuada.
- El GND de Game Boy, level shifter y ESP32 debe ser común.

---

## 💾 Memoria interna y capturas

Las capturas se guardan en LittleFS con nombres internos del tipo:

```text
P000001.GBP
P000002.GBP
P000003.GBP
```

Cada captura estándar de Game Boy Camera ocupa aproximadamente:

```text
5760 bytes de imagen
5784 bytes con cabecera NR Printer
```

La galería muestra las fotos como:

```text
Foto 1 / 8
Foto 2 / 8
...
```

aunque internamente los archivos mantengan su ID único.

### Límite de capturas

El firmware usa un límite lógico de **100 capturas**.  
Si se supera el límite, se eliminan automáticamente las capturas más antiguas para dejar lugar a las nuevas.

---

## 🌐 Galería web local

El ESP32 puede levantar una red WiFi local llamada:

```text
NRPrinter
```

La red no requiere contraseña.

Para entrar a la galería:

```text
http://192.168.4.1
```

También funciona:

```text
http://192.168.4.1/nrprinter
```

Desde la web se puede:

- ver las capturas guardadas;
- descargar BMP;
- imprimir una captura en la SC03h;
- borrar una captura;
- borrar la galería completa;
- olvidar la impresora vinculada;
- cambiar configuración de impresión;
- volver a Modo Printer.

---

## ⚙️ Configuración disponible

Desde la web local se puede modificar:

### Oscuridad térmica

Ajusta la energía del cabezal térmico.

Valores incluidos:

| Modo | Valor |
|---|---|
| Clara | `0x25` |
| Normal | `0x30` |
| Oscura | `0x40` |
| Muy oscura | `0x50` |
| Extrema | `0x60` |

> La diferencia puede variar según el papel, batería/carga de la impresora y cantidad de zonas oscuras de la imagen.

### Escala

| Modo | Resultado |
|---|---|
| Original 1x | 160 px centrado |
| Doble 2x | 320 px centrado |
| Máximo SC03h | 384 px completo |

### Tramado de grises

Simula tonos de gris en una impresora térmica blanco/negro usando dithering.

### Invertir colores

Invierte la salida visual de la captura.

### Auto-print

Activa o desactiva la impresión automática al recibir una captura desde Game Boy.

---

## 🔵 Comportamiento del LED

El firmware usa el LED integrado del ESP32 como indicador de estado.

| Estado | LED |
|---|---|
| Modo Printer en espera | Apagado |
| Modo Web / Galería WiFi | Encendido fijo |
| Buscando o vinculando SC03h | Pulso lento |
| Imprimiendo en SC03h | Parpadeo controlado |
| Auto-print activado con botón BOOT | 4 parpadeos |
| Auto-print desactivado con botón BOOT | 2 parpadeos |
| Fin de impresión / reposo | Vuelve al estado correspondiente |

---

## 🔘 Uso del botón BOOT

El botón BOOT del ESP32 tiene dos funciones principales.

| Acción | Resultado |
|---|---|
| Pulsación corta | Activa/desactiva Auto-print |
| Mantener 2 segundos | Entra a Modo Web / Galería |

En Modo Web, el puerto Link queda pausado.  
Para volver a imprimir desde Game Boy, usá el botón web **Volver a Modo Printer** o mantené BOOT durante 2 segundos.

---

## 🚀 Instalación rápida

Puedes flashear el ESP32 directamente desde el navegador usando **ESP Web Tools**.

➡️ **Instalador web:**  
`https://novenretro.github.io/nr-printer/`

### Pasos

1. Conectá el ESP32 por USB
2. Abrí el instalador desde Google Chrome o Microsoft Edge
3. Hacé clic en **CONNECT**
4. Elegí el puerto correcto
5. Instalá el firmware
6. Esperá a que finalice el proceso
7. Reiniciá el ESP32 si fuera necesario

> Puede que sea necesario mantener presionado BOOT al iniciar la carga, dependiendo de tu placa ESP32.

---

## 📦 Archivos de firmware para WebTools

El instalador usa los archivos generados por Arduino IDE:

```text
NR_GBPrinter_SC03h_v1_0_2.ino.bootloader.bin
NR_GBPrinter_SC03h_v1_0_2.ino.partitions.bin
NR_GBPrinter_SC03h_v1_0_2.ino.bin
```

Offsets utilizados:

| Archivo | Offset |
|---|---|
| Bootloader | `0x1000` |
| Partitions | `0x8000` |
| Firmware | `0x10000` |

No son necesarios para el instalador web:

```text
.elf
.map
.merged.bin
```

---

## 🧪 Estado del proyecto

**NR Printer v1.0** fue validado con:

- múltiples impresiones consecutivas desde Game Boy Camera;
- Auto-print ON;
- Auto-print OFF;
- impresora SC03h apagada;
- impresora SC03h encendida;
- vinculación automática BLE;
- borrado de impresora vinculada;
- impresión manual desde galería web;
- borrado individual de capturas;
- borrado completo de galería;
- retorno estable de modo Web a modo Printer.

---

## ⚠️ Limitaciones conocidas

- No funciona con cualquier impresora térmica Bluetooth.
- Requiere una impresora compatible con protocolo SC03h / iPrint BLE `AE30 / AE01`.
- El modo Web pausa el puerto Link.
- Después de usar BLE, el firmware realiza un reinicio limpio para recuperar estabilidad total del Link Cable.
- El WiFi local del ESP32 no tiene internet; es solo para configuración y galería.
- OTA por internet no está implementado en v1.0.

---

## 🧩 Notas técnicas

La Game Boy Camera envía normalmente:

```text
9 paquetes DATA x 640 bytes = 5760 bytes
```

La imagen reconstruida corresponde a:

```text
160 x 144 px
```

El firmware convierte esa imagen al ancho de la impresora térmica:

```text
384 px
```

usando la escala configurada desde la web.

---

## 📺 Video del proyecto

Podés ver el proceso completo, pruebas, armado y explicación en el canal:

**YouTube:** [NovenRetro](https://youtube.com/@novenretro)

---

## 👨‍💻 Autor

Proyecto desarrollado por **@NovenRetro**  
Córdoba, Argentina — 2026

---

## 🚫 Uso comercial

Este proyecto se comparte para uso personal, educativo y experimental.

**Prohibida su comercialización en cualquier forma sin autorización de NovenRetro.**

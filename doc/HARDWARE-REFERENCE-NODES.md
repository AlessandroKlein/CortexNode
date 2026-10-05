# Hardware Reference Nodes

> **Tipo:** Arquitectura / Hardware de referencia
> **Estado:** Definición
> **Versión:** 1.0.0
> **Última actualización:** 2026-10-05

---

# 1. Objetivo

Este documento define arquitecturas de hardware de referencia para los distintos tipos de Nodes de la plataforma.

No pretende limitar el sistema a determinados ESP32 o componentes.

Su objetivo es establecer:

* qué familia de ESP32 resulta adecuada para cada función;
* qué periféricos son recomendables;
* qué interfaces utilizar;
* qué funciones puede cubrir cada Node;
* qué hardware debe preferirse;
* qué alternativas existen;
* cómo debe integrarse con la arquitectura distribuida.

La plataforma debe permitir agregar nuevas variantes sin modificar la arquitectura lógica.

---

# 2. Principio fundamental

> **El hardware se selecciona según la función que debe cumplir el Node.**

No debe utilizarse siempre el ESP32 más potente.

Ejemplo:

```text
Tecla + Ethernet
        ↓
ESP32-C3 + W5500
```

puede ser más conveniente que:

```text
ESP32-S3 + Ethernet
```

si el Node solamente necesita entradas digitales y comunicación.

---

# 3. Clasificación de Nodes

Se establecen las siguientes categorías:

```text
INPUT NODE
RELAY NODE
DIMMER NODE
SENSOR NODE
ENERGY NODE
MODBUS NODE
CAN NODE
ETHERNET NODE
DISPLAY NODE
CAMERA / AI NODE
GATEWAY NODE
ZONE CONTROLLER
CENTRAL
```

Un dispositivo puede pertenecer a más de una categoría.

---

# 4. Matriz general

| Función              | ESP32 recomendado | Interfaces principales       |
| -------------------- | ----------------- | ---------------------------- |
| Teclas / entradas    | ESP32-C3          | GPIO + W5500                 |
| Relays               | ESP32-WROOM-32    | GPIO + Ethernet PHY          |
| Dimmer AC            | ESP32-WROOM-32    | GPIO + Zero Cross + Ethernet |
| Sensores generales   | ESP32-C3 / WROOM  | GPIO/I2C/SPI                 |
| Modbus RTU           | ESP32-S3 / WROOM  | UART + RS485                 |
| CAN/CANopen          | ESP32-S3 / WROOM  | TWAI + transceiver           |
| Display táctil       | ESP32-S3          | SPI/RGB + I2C                |
| Cámara               | ESP32-S3          | Camera interface             |
| IA local             | ESP32-S3          | Camera + PSRAM               |
| Matter/Thread/Zigbee | ESP32-C6          | 802.15.4 + Wi-Fi             |
| Gateway              | ESP32-S3/C6       | Ethernet + buses             |
| Zone Controller      | ESP32-S3          | Ethernet/Wi-Fi               |
| Central              | ESP32-S3 + PSRAM  | Ethernet + storage           |

---

# 5. Node de teclas / entradas

## 5.1 Arquitectura recomendada

```text
ESP32-C3
   │
   ├── GPIO
   │    ├── tecla 1
   │    ├── tecla 2
   │    ├── tecla 3
   │    └── tecla N
   │
   └── SPI
        │
        ▼
      W5500
        │
        ▼
     Ethernet
```

---

# 6. ESP32-C3 + W5500

Este Node está pensado para:

* teclas;
* pulsadores;
* interruptores;
* entradas digitales;
* sensores simples;
* encoders;
* señales de estado.

El W5500 proporciona Ethernet mediante SPI.

### Ventajas

* bajo costo;
* bajo consumo;
* tamaño reducido;
* suficiente capacidad para entradas;
* Ethernet cableado;
* buena separación entre función y comunicación.

---

# 7. Aplicaciones

Ejemplo:

```text
Panel de pared

[ Luz ]
[ Escena ]
[ Persiana ]
[ Alarma ]
```

El ESP32-C3 puede detectar:

```text
BUTTON_PRESSED
BUTTON_RELEASED
BUTTON_LONG_PRESS
BUTTON_DOUBLE_PRESS
```

y publicar:

```text
input.button_01.pressed
```

El Node no necesita saber qué hará el sistema con ese evento.

---

# 8. Expansión de entradas

Cuando existan muchas teclas se pueden utilizar:

```text
MCP23017
MCP23S17
74HC165
```

según necesidades de hardware.

Ejemplo:

```text
ESP32-C3
   │
   ├── GPIO
   │
   └── I2C
        │
        ▼
     MCP23017
        │
   ┌────┼────┐
   ▼    ▼    ▼
 Keys Keys Keys
```

---

# 9. Ethernet mediante W5500

El W5500 se conecta mediante SPI.

Conceptualmente:

```text
ESP32-C3
│
├── SCK
├── MOSI
├── MISO
├── CS
├── INT
└── RST
       │
       ▼
     W5500
       │
       ▼
    RJ45
```

La asignación concreta de GPIO debe mantenerse configurable y validarse contra el módulo/placa utilizada.

---

# 10. Node de relays

## Arquitectura recomendada

```text
ESP32-WROOM-32
       │
       ├── GPIO
       │
       ▼
   Relay Driver
       │
       ▼
      Relay
```

Ethernet:

```text
ESP32-WROOM-32
       │
       ▼
LAN8720A / IP101
       │
       ▼
RJ45
```

---

# 11. ESP32-WROOM-32 + LAN8720A

Adecuado para:

* relays;
* contactores mediante driver;
* válvulas;
* bombas;
* iluminación;
* salidas digitales;
* sensores locales.

El Ethernet puede utilizar RMII.

Arquitectura:

```text
ESP32
 │
 ├── RMII
 │
 ▼
LAN8720A
 │
 ▼
Magnetics
 │
 ▼
RJ45
```

---

# 12. IP101 / IP101G

El IP101/IP101G puede utilizarse como alternativa de PHY Ethernet compatible con la arquitectura del ESP32.

La selección final debe realizarse según:

* disponibilidad;
* documentación;
* soporte del diseño;
* consumo;
* encapsulado;
* disponibilidad en PCB;
* disponibilidad en JLCPCB/PCB Assembly.

---

# 13. Relay Node recomendado

```text
ESP32-WROOM-32
│
├── Ethernet PHY
│
├── Relay 1
├── Relay 2
├── Relay 3
├── Relay 4
├── Relay 5
└── Relay 6
```

Para cargas de red eléctrica deben incorporarse:

* aislamiento;
* protección;
* separación de pistas;
* fusibles cuando corresponda;
* snubber/TVS según carga;
* driver adecuado;
* relay certificado para la carga.

---

# 14. Node de dimmer AC

## Arquitectura

```text
ESP32-WROOM-32
       │
       ├── Zero Cross
       │
       └── Gate / Trigger
              │
              ▼
          TRIAC Driver
              │
              ▼
             TRIAC
              │
              ▼
             Carga
```

Ethernet:

```text
ESP32
  │
  ▼
LAN8720A / IP101G
  │
  ▼
Ethernet
```

---

# 15. Zero Cross

El detector de cruce por cero informa al ESP32 cuándo la señal AC pasa por aproximadamente cero.

El firmware puede utilizar esta referencia para controlar el ángulo de disparo.

Conceptualmente:

```text
AC waveform

      /\
     /  \
----/----\----/----\----
   0      0    0

     ↑
 Zero Cross
```

---

# 16. Dimmer

El ESP32 calcula el retardo correspondiente al nivel deseado.

Ejemplo:

```text
0 %    → OFF
25 %   → baja potencia
50 %   → potencia media
75 %   → alta potencia
100 %  → máximo
```

La implementación concreta depende del tipo de carga.

No todas las cargas AC son compatibles con el mismo tipo de dimmer.

---

# 17. Precaución

El Node de dimmer trabaja con tensión de red.

Debe considerarse:

* aislamiento galvánico;
* optoacoplador;
* distancias de seguridad;
* creepage;
* clearance;
* fusible;
* protección contra transitorios;
* temperatura;
* encapsulado;
* puesta a tierra cuando corresponda.

El firmware no sustituye el diseño eléctrico de seguridad.

---

# 18. Node de sensores

Para sensores generales:

```text
ESP32-C3
ESP32-WROOM
ESP32-S3
```

según complejidad.

Interfaces:

```text
GPIO
ADC
I2C
SPI
UART
1-Wire
RS485
```

Ejemplo:

```text
ESP32
│
├── AHT30
├── BMP280
├── BH1750
└── DS18B20
```

---

# 19. Node Modbus RTU

## Arquitectura recomendada

```text
ESP32-S3
    │
   UART
    │
    ▼
RS485 Transceiver
    │
    ▼
A/B
    │
    ├── Sensor
    ├── Inverter
    ├── Energy Meter
    └── Industrial Device
```

---

# 20. ADM2483

El ADM2483 puede utilizarse como interfaz RS485 aislada.

Arquitectura:

```text
ESP32
 │
 ▼
UART
 │
 ▼
ADM2483
 │
 ▼
RS485
 │
 ▼
Dispositivo industrial
```

Es especialmente interesante cuando se necesita aislamiento entre lógica y bus.

---

# 21. TD501D485H

El TD501D485H puede utilizarse como alternativa de interfaz RS485 según disponibilidad y requisitos del diseño.

La elección del transceiver debe considerar:

* aislamiento;
* tensión;
* velocidad;
* protección ESD;
* protección contra sobretensiones;
* fail-safe;
* terminación;
* disponibilidad.

---

# 22. Modbus Node

Un Node puede funcionar como gateway:

```text
RS485
   │
   ▼
ESP32
   │
   ├── Modbus RTU
   │
   ▼
System Bus
   │
   ├── MQTT
   ├── REST
   └── Automation
```

Por ejemplo:

```text
Energy Meter
      │
    Modbus
      │
      ▼
ESP32
      │
      ▼
Power = 2.43 kW
```

El resto de la plataforma no necesita conocer Modbus.

---

# 23. Node CAN

## Arquitectura

```text
ESP32-S3
    │
   TWAI
    │
    ▼
CAN Transceiver
    │
    ▼
CANH / CANL
```

Transceiver de referencia:

```text
SN65HVD23x
```

La variante exacta debe seleccionarse según:

* alimentación;
* velocidad;
* protección;
* modo standby;
* disponibilidad.

---

# 24. CANopen

El Node CAN puede actuar como:

```text
CAN Node
CANopen Node
CAN Gateway
CAN Monitor
CAN Controller
```

Puede soportar conceptos como:

```text
PDO
SDO
Heartbeat
Emergency
Node ID
Object Dictionary
```

La aplicación no debe depender directamente de los registros del transceiver.

---

# 25. CAN Gateway

Ejemplo:

```text
CAN device
    │
    ▼
ESP32-S3
    │
    ├── CAN
    │
    ▼
System Bus
    │
    ├── Ethernet
    ├── MQTT
    └── REST
```

Esto permite integrar equipos industriales o marinos en la plataforma.

---

# 26. Node Ethernet industrial

Para un Node que necesite varias interfaces:

```text
ESP32-S3
│
├── Ethernet
├── RS485
├── CAN
├── I2C
├── SPI
└── GPIO
```

Este tipo de Node puede actuar como:

```text
Industrial Gateway
```

o:

```text
Zone Controller
```

---

# 27. ESP32-C6

El ESP32-C6 debe reservarse especialmente para Nodes que necesiten:

```text
Wi-Fi 6
802.15.4
Thread
Zigbee
Matter
```

Ejemplo:

```text
ESP32-C6
│
├── Wi-Fi
├── Thread
├── Zigbee
└── Matter
```

Puede actuar como:

* gateway;
* border router según arquitectura;
* dispositivo Matter;
* Node inalámbrico;
* puente de protocolos.

---

# 28. ESP32-S3

El ESP32-S3 debe utilizarse cuando se necesite mayor capacidad de procesamiento o memoria.

Aplicaciones:

```text
Display
Camera
AI
Touchscreen
Central
Zone Controller
Gateway
Advanced Sensor
```

Especialmente recomendado con:

```text
PSRAM
```

cuando se utilizan:

* imágenes;
* buffers grandes;
* interfaces gráficas;
* modelos de IA;
* grandes estructuras de datos.

---

# 29. Display Node

Arquitectura:

```text
ESP32-S3
│
├── Display
│
├── Touch Controller
│
├── Network
│
└── System Bus
```

Controladores de display posibles:

```text
ST7789
ILI9341
ILI9488
ST7796
GC9A01
ST7262
```

Controladores táctiles:

```text
CST816S
CST816D
CST820
FT6236
FT6336
GT911
XPT2046
```

La plataforma debe abstraer el hardware.

---

# 30. Display modular

El usuario debe poder configurar:

```text
Pantalla
│
├── Temperatura
├── Humedad
├── Luz
├── Energía
├── Seguridad
├── Música
└── Escenas
```

Los widgets deben ser módulos.

---

# 31. Camera / AI Node

Arquitectura:

```text
ESP32-S3
│
├── Camera
├── PSRAM
├── AI Processing
├── Network
└── Automation
```

Ejemplos:

```text
Person detection
Object detection
Presence
Image classification
Simple anomaly detection
```

El Node debería enviar eventos, no necesariamente vídeo continuo.

Ejemplo:

```text
person.detected
```

---

# 32. Energy Node

Para medición eléctrica pueden utilizarse:

```text
ZMPT101B
SCT-013
```

según la topología y circuito de medición.

Arquitectura:

```text
ESP32
│
├── Voltage
├── Current
├── Power
├── Energy
├── Frequency
└── Power Factor
```

Debe utilizarse un front-end adecuado para medición segura y calibrada.

---

# 33. Water Flow Node

Para medición de agua:

```text
ESP32
   │
   ▼
Hall Flow Sensor
   │
   ▼
Pulse Counter
   │
   ▼
Flow calculation
```

Variables:

```text
flow_rate
total_volume
pulse_count
```

Ejemplos:

```text
YF-series
DN-series
```

---

# 34. Node de seguridad

Puede incluir:

```text
PIR
Door Contact
Window Contact
Smoke
Water Leak
Vibration
Tamper
```

Ejemplo:

```text
ESP32-C3
│
├── PIR
├── Door
├── Window
├── Leak
└── Tamper
```

Los eventos deben publicarse como:

```text
motion.detected
door.opened
window.opened
water.detected
tamper.detected
```

---

# 35. Security Node

Debe funcionar incluso sin Central.

Ejemplo:

```text
PIR
 │
 ▼
Security Node
 │
 ├── local alarm
 ├── local siren
 └── event
```

El Central puede coordinar el sistema completo, pero la protección crítica debe permanecer local.

---

# 36. Zone Controller

Un Zone Controller es un ESP32 con mayor capacidad que coordina varios Nodes.

Puede ser:

```text
ESP32-S3
```

Preferentemente con:

```text
PSRAM
Ethernet
```

Funciones:

```text
Zone Automation
Gateway
State Aggregation
Local History
Failover
Protocol Translation
```

---

# 37. Ejemplo de Zone Controller

```text
                    ZONE CONTROLLER
                       ESP32-S3
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
       Node A           Node B           Node C
       Sensors          Relays           Display
```

Puede continuar funcionando aunque el Central esté desconectado.

---

# 38. Central

## Hardware recomendado

```text
ESP32-S3
+
PSRAM
+
Flash suficiente
+
Ethernet
+
microSD opcional
```

---

# 39. Central — funciones

```text
Web Portal
REST API
WebSocket
Device Registry
Discovery
Configuration
Users
Automation
History
Logs
OTA
Integration
Diagnostics
```

No debe ejecutar necesariamente todas las tareas de todos los Nodes.

---

# 40. Central Ethernet

Preferencia:

```text
ESP32-S3
    │
    ▼
Ethernet
    │
    ▼
LAN / Switch
```

Para un Central fijo se recomienda Ethernet siempre que sea posible.

Ventajas:

* estabilidad;
* menor interferencia;
* menor latencia;
* disponibilidad permanente;
* mejor infraestructura para instalaciones grandes.

---

# 41. microSD en Central

Puede utilizarse para:

```text
History
Logs
Backups
Configuration snapshots
Firmware packages
Large data
```

No se debe asumir que toda instalación necesita microSD.

---

# 42. Gateway multiprotocolo

Un ESP32-S3 puede actuar como:

```text
Ethernet
   │
   ├── RS485
   ├── CAN
   ├── Wi-Fi
   ├── BLE
   └── System Bus
```

Esto permite conectar sistemas externos.

---

# 43. Ejemplo de Gateway industrial

```text
                 Ethernet
                    │
                    ▼
               ESP32-S3
              /    |    \
             /     |     \
          RS485    CAN    GPIO
            │       │
         Modbus   CANopen
```

El resto del sistema solamente recibe:

```text
sensor.value
device.state
alarm.triggered
```

---

# 44. Arquitectura de hardware desacoplada

La plataforma debe separar:

```text
Logical Device
```

de:

```text
Physical Hardware
```

Ejemplo:

```text
Logical:
"Hall Light"
```

Puede estar implementado mediante:

```text
ESP32-WROOM + Relay
```

o:

```text
ESP32-S3 + MOSFET
```

o:

```text
CAN actuator
```

El software de automatización no debería cambiar.

---

# 45. Hardware Profiles

Cada Node debe declarar un perfil.

Ejemplo:

```json
{
  "profile": "relay_node",
  "hardware": {
    "mcu": "esp32-wroom-32",
    "ethernet": "lan8720a",
    "outputs": 8
  }
}
```

Otro:

```json
{
  "profile": "can_gateway",
  "hardware": {
    "mcu": "esp32-s3",
    "can": true,
    "ethernet": true
  }
}
```

---

# 46. Capabilities

El sistema debe descubrir capacidades.

Ejemplo:

```json
{
  "capabilities": [
    "digital_input",
    "digital_output",
    "pwm",
    "ethernet",
    "rs485"
  ]
}
```

La aplicación utiliza:

```text
capability
```

y no:

```text
GPIO number
```

---

# 47. Selección automática de hardware

El portal puede recomendar:

```text
¿Qué quieres construir?

[ Node de relays ]
[ Node de sensores ]
[ Node CAN ]
[ Node Modbus ]
[ Node de teclas ]
[ Node Display ]
[ Node cámara ]
```

Al seleccionar una función, el sistema puede mostrar:

```text
Hardware recomendado
```

por ejemplo:

```text
Relay Node

ESP32-WROOM-32
+
LAN8720A
+
Relay Driver
```

---

# 48. Tabla de referencia

| Node            | MCU              | Comunicación         | Función           |
| --------------- | ---------------- | -------------------- | ----------------- |
| Input Node      | ESP32-C3         | W5500                | Teclas / entradas |
| Relay Node      | ESP32-WROOM-32   | LAN8720A/IP101G      | Relays            |
| Dimmer Node     | ESP32-WROOM-32   | LAN8720A/IP101G      | Dimmer AC         |
| Sensor Node     | ESP32-C3         | Wi-Fi/Ethernet       | Sensores          |
| Modbus Node     | ESP32-S3         | RS485                | Modbus RTU        |
| CAN Node        | ESP32-S3         | CAN                  | CAN/CANopen       |
| Gateway         | ESP32-S3         | Ethernet + buses     | Protocolos        |
| Wireless Node   | ESP32-C6         | Thread/Zigbee/Matter | IoT               |
| Display Node    | ESP32-S3         | Ethernet/Wi-Fi       | UI                |
| AI Node         | ESP32-S3 + PSRAM | Ethernet/Wi-Fi       | Cámara/IA         |
| Zone Controller | ESP32-S3 + PSRAM | Ethernet             | Coordinación      |
| Central         | ESP32-S3 + PSRAM | Ethernet             | Administración    |

---

# 49. Selección por costo

No se debe utilizar siempre ESP32-S3.

Ejemplo:

```text
Función simple
     ↓
ESP32-C3
```

```text
GPIO + Ethernet
     ↓
ESP32-WROOM + PHY
```

```text
Ethernet SPI
     ↓
ESP32-C3 + W5500
```

```text
CAN / Modbus avanzado
     ↓
ESP32-S3
```

```text
Display / cámara / IA
     ↓
ESP32-S3 + PSRAM
```

```text
Matter / Thread / Zigbee
     ↓
ESP32-C6
```

---

# 50. Criterio de selección

La elección del MCU debe considerar:

```text
CPU
RAM
PSRAM
Flash
GPIO
ADC
PWM
UART
I2C
SPI
CAN/TWAI
Ethernet
Wi-Fi
802.15.4
Camera
Display
Power
Cost
Availability
```

No seleccionar únicamente por frecuencia de CPU.

---

# 51. Reglas para hardware de red

Ethernet puede implementarse mediante:

### MAC + PHY

```text
ESP32
+
LAN8720A
```

o:

```text
ESP32
+
IP101/IP101G
```

### Ethernet mediante SPI

```text
ESP32
+
W5500
```

La selección depende de:

* costo;
* cantidad de GPIO;
* rendimiento;
* PCB;
* disponibilidad;
* aislamiento;
* arquitectura del Node.

---

# 52. Ethernet nativo vs W5500

## Ethernet nativo

Preferible cuando:

* el Node necesita mayor rendimiento;
* existe suficiente disponibilidad de GPIO;
* se desea reducir dependencia de SPI;
* el dispositivo es fijo;
* se utiliza como Zone Controller o Central.

Ejemplo:

```text
ESP32-WROOM
+
LAN8720A
```

---

## W5500

Preferible cuando:

* el Node necesita Ethernet pero el MCU/placa no dispone de PHY integrado;
* el número de GPIO es limitado;
* el diseño necesita Ethernet mediante SPI;
* se busca un Node simple.

Ejemplo:

```text
ESP32-C3
+
W5500
```

---

# 53. RS485

RS485 debe considerarse una capa física.

Por encima puede existir:

```text
Modbus RTU
Protocolo propietario
DMX
```

No debe confundirse:

```text
RS485 ≠ Modbus
```

---

# 54. CAN

De forma similar:

```text
CAN PHY
   ↓
CAN protocol
   ↓
CANopen / protocolo propietario
```

No debe mezclarse el transceiver físico con la lógica de aplicación.

---

# 55. Reutilización de hardware

Un mismo diseño PCB puede admitir múltiples funciones.

Ejemplo:

```text
Universal IO Board
│
├── ESP32
├── Ethernet
├── RS485
├── CAN
├── GPIO
├── Relay outputs
└── Inputs
```

El portal determina qué funciones están activas.

Esto permite reducir:

* cantidad de PCBs;
* stock;
* mantenimiento;
* costos de producción.

---

# 56. Seguridad eléctrica

Los Nodes conectados a:

```text
110/220/240 VAC
```

deben diseñarse con separación adecuada entre:

```text
SELV / lógica
```

y:

```text
mains
```

Debe contemplarse:

* aislamiento;
* fusibles;
* protección contra sobretensión;
* creepage;
* clearance;
* temperatura;
* encapsulado;
* puesta a tierra;
* relays adecuados;
* optoacoplamiento cuando corresponda.

---

# 57. Regla de modularidad

Cada Node debe poder reemplazarse por otro equivalente.

Ejemplo:

```text
Relay Node v1
ESP32-WROOM + LAN8720A
```

puede reemplazarse por:

```text
Relay Node v2
ESP32-S3 + Ethernet
```

sin cambiar:

```text
Automation
Scenes
API
UI
Logical Device
```

---

# 58. Firmware común

Cuando sea posible, los Nodes deben utilizar un firmware común:

```text
Universal Automation Firmware
```

La configuración determina:

```text
Hardware
Capabilities
Resources
Modules
Services
```

Esto reduce el mantenimiento.

---

# 59. Profiles de firmware

El sistema puede utilizar perfiles:

```text
input_node
relay_node
dimmer_node
sensor_node
modbus_node
can_node
display_node
camera_node
gateway_node
zone_controller
central
```

Todos utilizan la misma base de software.

---

# 60. Evolución futura

El catálogo debe poder ampliarse con:

```text
ESP32-P4
Nuevos ESP32
MCUs adicionales
Ethernet PHY nuevos
CAN transceivers
RS485 transceivers
Displays
Sensores
Actuadores
```

La arquitectura lógica no debe depender de un modelo específico.

---

# 61. Ejemplo de instalación completa

```text
                         CENTRAL
                    ESP32-S3 + PSRAM
                            │
                  Ethernet / System Bus
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
   Zone Controller     CAN Gateway         Modbus Gateway
      ESP32-S3            ESP32-S3             ESP32-S3
        │                   │                   │
   ┌────┼────┐          CAN devices        RS485 devices
   │    │    │
   ▼    ▼    ▼
 C3    WROOM S3
Keys  Relays Display
 │      │      │
W5500  PHY   Touch
```

---

# 62. Principio definitivo

La plataforma no debe preguntarse:

> "¿Qué ESP32 tenemos?"

sino:

> "¿Qué función necesita realizar este Node y qué hardware es el más adecuado para hacerlo?"

La arquitectura debe permitir elegir posteriormente el MCU y periféricos sin modificar el modelo lógico del sistema.

---

# 63. Resumen

### ESP32-C3

Preferido para:

```text
Inputs
Buttons
Simple Sensors
Low-cost Nodes
W5500 Ethernet
```

### ESP32-WROOM-32

Preferido para:

```text
Relays
Dimmers
GPIO
Ethernet PHY
Industrial/simple Nodes
```

### ESP32-S3

Preferido para:

```text
Central
Zone Controller
CAN
Modbus
Gateway
Display
Camera
AI
Large applications
```

### ESP32-C6

Preferido para:

```text
Matter
Thread
Zigbee
Wi-Fi 6
Wireless Gateway
```

---

# 64. Principio maestro

> **El hardware es intercambiable; las capacidades y funciones son permanentes.**

La plataforma debe tratar cada dispositivo por sus:

```text
Capabilities
Resources
Services
```

y no por el modelo físico del ESP32.

De esta forma, un:

```text
ESP32-C3 + W5500
```

y un:

```text
ESP32-S3 + Ethernet
```

pueden ofrecer una interfaz lógica compatible aunque tengan capacidades físicas muy diferentes.

---

# 65. Relación con la arquitectura general

Este documento debe utilizarse conjuntamente con:

```text
ARCHITECTURE.md
SYSTEM-ARCHITECTURE.md
CENTRAL-ARCHITECTURE.md
LOAD-DISTRIBUTION.md
DEVICE-MODEL.md
COMMUNICATION.md
MODULE-DEVELOPMENT.md
FREERTOS-TASK-ARCHITECTURE.md
```

La cadena conceptual es:

```text
USER
 ↓
PORTAL
 ↓
LOGICAL FUNCTION
 ↓
DEVICE MODEL
 ↓
CAPABILITY
 ↓
HARDWARE PROFILE
 ↓
DRIVER
 ↓
FREERTOS SERVICE
 ↓
HARDWARE
```

Esto permite que el usuario configure el sistema sin necesidad de conocer ni modificar el firmware.

# HARDWARE-REFERENCE-NODES.md

> **Sistema:** Plataforma de Automatización Distribuida
> **Tipo:** Arquitectura de hardware / Nodos de referencia
> **Estado:** Definición
> **Versión:** 1.0.0
> **Última actualización:** 2026-10-06

---

# 1. Objetivo

Este documento define los **nodos de hardware de referencia** de la plataforma.

Los nodos de referencia son diseños recomendados que sirven como base para construir:

* controladores;
* sensores;
* actuadores;
* gateways;
* nodos de comunicación;
* interfaces de usuario;
* nodos de seguridad;
* nodos de energía;
* nodos de adquisición;
* nodos de procesamiento;
* nodos de inteligencia artificial;
* controladores de zona;
* Central.

No representan placas obligatorias.

La plataforma debe permitir crear hardware personalizado siempre que respete las interfaces y capacidades definidas por la arquitectura.

---

# 2. Principio fundamental

> **El nodo de referencia define una arquitectura recomendada, no una dependencia del sistema.**

Por lo tanto:

```text
Reference Node
      ↓
Build Profile
      ↓
Hardware Profile
      ↓
Drivers
      ↓
Modules
      ↓
Device Model
```

El resto del sistema no debe depender de una PCB concreta.

---

# 3. Objetivos de los nodos de referencia

Los diseños de referencia deben:

* reducir tiempo de desarrollo;
* proporcionar una base de hardware conocida;
* facilitar pruebas;
* establecer pinouts recomendados;
* definir interfaces;
* facilitar la fabricación;
* permitir reutilización;
* simplificar soporte;
* facilitar documentación;
* mantener compatibilidad entre versiones;
* permitir escalar la plataforma.

---

# 4. Categorías

Los nodos se dividen principalmente en:

```text
┌─────────────────────────────────────────┐
│ NODOS DE REFERENCIA                     │
├─────────────────────────────────────────┤
│ 1. Node Basic                           │
│ 2. Node Ethernet                        │
│ 3. Node RS485                           │
│ 4. Node CAN                             │
│ 5. Node IO                              │
│ 6. Node Sensor                          │
│ 7. Node Display                         │
│ 8. Node Camera / AI                     │
│ 9. Zone Controller                      │
│ 10. Central                             │
│ 11. Gateway                             │
│ 12. Energy Controller                   │
└─────────────────────────────────────────┘
```

---

# 5. Node Basic

El **Node Basic** es el nodo de menor complejidad.

Está pensado para:

* sensores;
* botones;
* relés;
* iluminación;
* pequeñas automatizaciones;
* actuadores simples.

## Hardware recomendado

```text
MCU:
ESP32-C3 / ESP32-C6 / ESP32-WROOM
```

Interfaces:

```text
Wi-Fi
Bluetooth según MCU
GPIO
ADC
PWM
I²C
SPI
UART
```

Arquitectura:

```text
ESP32
 │
 ├── GPIO
 ├── I²C
 ├── SPI
 ├── UART
 └── Wi-Fi
```

---

# 6. Node Basic — uso recomendado

Ejemplos:

```text
Interruptor inteligente
Sensor de temperatura
Control de iluminación
Control de ventilador
Control de válvula
PIR
Sensor de puerta
Sensor de humedad
```

No debe utilizarse como Central.

---

# 7. Node Basic — filosofía

Debe priorizar:

```text
bajo costo
bajo consumo
simplicidad
fiabilidad
autonomía
```

Debe evitar periféricos innecesarios.

---

# 8. Node Ethernet

El **Node Ethernet** está diseñado para instalaciones donde Ethernet es preferible a Wi-Fi.

Puede utilizar dos arquitecturas.

## Variante A — Ethernet nativo

```text
ESP32-WROOM
      │
      │ RMII
      ▼
LAN8720A
      │
      ▼
RJ45
```

## Variante B — Ethernet SPI

```text
ESP32-S3
      │
      │ SPI
      ▼
W5500
      │
      ▼
RJ45
```

Ambas deben proporcionar la misma abstracción de red al sistema.

---

# 9. Node Ethernet — LAN8720A

Configuración de referencia:

```ini
[env:node_ethernet_wroom]

build_flags =
    -D BOARD_ESP32_WROOM
    -D MCU_ESP32
    -D ETH_LAN8720
```

Hardware:

```text
ESP32-WROOM
LAN8720A
25 MHz Crystal
RJ45 with Magnetics
PHY reset
RMII
```

---

# 10. Node Ethernet — W5500

Configuración:

```ini
[env:node_ethernet_s3]

build_flags =
    -D BOARD_ESP32_S3
    -D MCU_ESP32_S3
    -D ETH_W5500
```

Hardware:

```text
ESP32-S3
    │
    ├── SPI
    ├── CS
    ├── INT
    └── RESET
          │
          ▼
        W5500
          │
          ▼
        RJ45
```

---

# 11. Ethernet común

Ambos nodos deben exponer:

```cpp
network.begin();
network.stop();
network.isConnected();
network.localIP();
network.macAddress();
network.linkSpeed();
```

La aplicación no debe distinguir:

```text
LAN8720A
```

de:

```text
W5500
```

salvo para diagnóstico.

---

# 12. Node RS485

El Node RS485 está diseñado para:

* Modbus RTU;
* sensores industriales;
* actuadores;
* medidores;
* dispositivos agrícolas;
* sistemas industriales;
* instalaciones largas.

Arquitectura:

```text
ESP32
  │
UART
  │
RS485 Transceiver
  │
A/B
  │
Bus RS485
```

---

# 13. RS485 — transceptores

Se pueden utilizar:

```text
SP3485
MAX3485
SN65HVD
ADM2483
aislados equivalentes
```

La selección concreta debe quedar en el Build Profile / Hardware Profile.

---

# 14. RS485 — aislamiento

Para instalaciones industriales o ambientes eléctricos agresivos, se recomienda:

```text
ESP32
   │
UART
   │
Aislador
   │
RS485 Transceiver
   │
Bus
```

El aislamiento puede incluir:

* aislamiento digital;
* alimentación aislada;
* protección contra sobretensiones;
* TVS;
* protección ESD.

---

# 15. Node CAN

El Node CAN está diseñado para:

* automatización distribuida;
* maquinaria;
* vehículos;
* aplicaciones marinas;
* controladores industriales;
* redes robustas.

Arquitectura:

```text
ESP32
 │
CAN Controller
 │
CAN Transceiver
 │
CANH / CANL
```

---

# 16. CAN nativo

Cuando el MCU dispone de controlador CAN/TWAI compatible:

```text
MCU
 │
TWAI / CAN Controller
 │
Transceiver
 │
CAN Bus
```

El transceiver puede ser:

```text
SN65HVD23x
SN65HVD25x
TJA1051
TJA1050
```

según la aplicación.

---

# 17. CAN externo

Cuando el MCU no disponga del controlador requerido:

```text
ESP32
 │
SPI
 │
CAN Controller externo
 │
CAN Transceiver
 │
CAN Bus
```

El nodo debe mantener la misma abstracción:

```cpp
can.begin();
can.send();
can.receive();
can.isConnected();
```

---

# 18. Node IO

El Node IO está pensado para grandes cantidades de entradas y salidas.

Puede utilizar:

```text
74HC595
74HC165
MCP23017
MCP23S17
PCF8574
TCA9555
```

Arquitectura:

```text
ESP32
  │
  ├── SPI
  │
  └── I²C
        │
        ▼
      Expander
        │
        ▼
   GPIO adicionales
```

---

# 19. 74HC595

Utilizar para:

* muchas salidas;
* LEDs;
* relés;
* señales digitales;
* actuadores simples.

Características arquitectónicas:

```text
8 outputs por chip
cascada
pocos GPIO del MCU
```

Ejemplo:

```text
ESP32
 │
SPI
 │
74HC595
 │
74HC595
 │
74HC595
 │
24 outputs
```

No se requiere un CS individual para cada chip.

---

# 20. 74HC165

Utilizar para:

* botones;
* interruptores;
* sensores digitales;
* entradas masivas.

Arquitectura:

```text
ESP32
 │
SPI / Shift Register Interface
 │
74HC165
 │
74HC165
 │
74HC165
 │
24 inputs
```

---

# 21. MCP23017

Utilizar cuando se necesiten GPIO completos y configurables.

Características:

```text
I²C
16 GPIO
direccionamiento
pull-up
interrupciones
entrada/salida
```

Arquitectura:

```text
ESP32
 │
I²C
 ├── MCP23017 0x20
 ├── MCP23017 0x21
 ├── MCP23017 0x22
 └── MCP23017 0x23
```

---

# 22. MCP23S17

Alternativa SPI:

```text
ESP32
 │
SPI
 ├── MCP23S17
 ├── MCP23S17
 └── MCP23S17
```

Cada dispositivo puede disponer de su selección mediante CS.

---

# 23. Node Sensor

El Node Sensor está optimizado para adquisición.

Puede incorporar:

```text
Temperatura
Humedad
Presión
Luz
CO₂
PM2.5
PIR
Presencia
Lluvia
Viento
Nivel
Flujo
pH
Conductividad
```

Arquitectura:

```text
ESP32
 │
 ├── I²C
 ├── SPI
 ├── ADC
 ├── UART
 └── RS485
```

---

# 24. Sensores locales vs industriales

## Locales

Ejemplos:

```text
AHT20
AHT30
BME280
DS18B20
BH1750
```

## Industriales

Ejemplos:

```text
RS485
Modbus RTU
4-20 mA
0-10 V
CAN
```

La aplicación debe ver ambos como:

```text
sensor.temperature
sensor.humidity
sensor.pressure
```

---

# 25. Node Sensor — autonomía

El nodo sensor debe poder continuar adquiriendo datos aunque Central esté fuera de línea.

Puede almacenar temporalmente:

```text
timestamp
value
quality
sequence
```

y sincronizar posteriormente.

---

# 26. Node Low Power

Para sensores alimentados por batería se recomienda un nodo específico.

Hardware:

```text
ESP32-C6 / ESP32-C3 / ESP32-S3 según necesidad
```

Debe priorizar:

```text
Deep Sleep
Light Sleep
RTC
Wake-up
GPIO interrupt
Timer
```

---

# 27. Wake-up

Ejemplos:

```text
RTC
GPIO
External Interrupt
Timer
Sensor interrupt
```

Arquitectura:

```text
Sleep
 ↓
Interrupt
 ↓
Wake
 ↓
Read Sensor
 ↓
Transmit
 ↓
Sleep
```

---

# 28. Node Display

Está diseñado para interfaces locales.

Puede incluir:

```text
ST7789
ILI9341
ILI9488
ST7796
GC9A01
```

y touch:

```text
GT911
CST816S
FT6236
XPT2046
```

---

# 29. Node Display — arquitectura

```text
ESP32-S3
 │
 ├── SPI
 │     └── Display
 │
 ├── I²C
 │     └── Touch
 │
 └── UI Framework
```

Para displays grandes o interfaces complejas se recomienda ESP32-S3 con PSRAM cuando sea compatible con la aplicación.

---

# 30. Node HMI

El Node HMI es una versión avanzada del Node Display.

Puede incluir:

```text
Display
Touch
Physical buttons
Encoder
Buzzer
Status LEDs
NFC/RFID
```

Uso:

```text
Control panel
Thermostat
Machine interface
Security panel
Home dashboard
```

---

# 31. Node Camera

El Node Camera está pensado para:

* captura de imágenes;
* vigilancia;
* presencia;
* inspección;
* lectura visual;
* análisis.

Hardware recomendado:

```text
ESP32-S3
PSRAM
Camera Interface
Wi-Fi/Ethernet
```

---

# 32. Node Camera AI

Es una evolución del Node Camera.

Arquitectura:

```text
Camera
   ↓
Frame Buffer
   ↓
Preprocessing
   ↓
AI Inference
   ↓
Result
   ↓
Entity / Event
```

Ejemplos:

```text
person.detected
vehicle.detected
object.detected
machine.anomaly
presence.detected
```

---

# 33. ESP32-S3 como nodo AI

El ESP32-S3 es una plataforma de referencia para:

* cámara;
* PSRAM;
* inferencia ligera;
* procesamiento local;
* interfaces gráficas.

Ejemplo:

```ini
[env:node_camera_ai]

build_flags =
    -D BOARD_ESP32_S3
    -D MCU_ESP32_S3
    -D HAS_PSRAM
    -D HAS_CAMERA
```

---

# 34. Node Energy

Diseñado para:

* corriente;
* tensión;
* potencia;
* energía;
* factor de potencia;
* consumo acumulado.

Puede utilizar:

```text
ADC
Shunt
CT
Energy Meter IC
Modbus
RS485
```

---

# 35. Seguridad eléctrica

Los nodos de energía deben mantener aislamiento apropiado entre:

```text
SELV / lógica
```

y:

```text
red eléctrica
```

cuando corresponda.

Los diseños conectados a red deben incorporar:

* fusibles;
* protección;
* aislamiento;
* distancias adecuadas;
* protección contra sobretensión;
* diseño de PCB apropiado;
* componentes certificados cuando corresponda.

---

# 36. Node Actuator

Diseñado para controlar:

```text
Relay
SSR
MOSFET
PWM
Motor
Valve
Fan
Pump
Lighting
```

Debe soportar:

```text
ON/OFF
PWM
speed
position
direction
```

según hardware.

---

# 37. Node Motor

Para motores puede utilizar:

```text
H-Bridge
MOSFET
Motor Driver
Servo Driver
Stepper Driver
```

El ESP32 debe controlar el driver y no necesariamente la potencia directamente.

Arquitectura:

```text
ESP32
 ↓
PWM / Direction
 ↓
Motor Driver
 ↓
Motor
```

---

# 38. Node Lighting

Debe soportar:

## Iluminación digital

```text
WS2812
SK6812
```

## Iluminación PWM

```text
MOSFET
```

## Iluminación AC

```text
Triac
SSR
Dimmer
Zero Cross
```

La interfaz lógica debe ser:

```text
light
```

independientemente del método físico.

---

# 39. Node Gateway

Un Gateway conecta redes o protocolos.

Ejemplo:

```text
CAN
 │
Gateway
 │
System Bus
 │
Ethernet
```

o:

```text
RS485
 │
Gateway
 │
Wi-Fi
```

o:

```text
Zigbee
 │
Gateway
 │
Ethernet
```

---

# 40. Gateway — función

Un Gateway puede:

* traducir protocolos;
* enrutar mensajes;
* descubrir dispositivos;
* aislar segmentos;
* almacenar temporalmente;
* realizar bridge.

Debe evitar transformar innecesariamente la lógica del Device Model.

---

# 41. Zone Controller

El Zone Controller coordina un área.

Ejemplos:

```text
Casa
Piso
Invernadero
Taller
Oficina
Barco
Sector industrial
```

Arquitectura:

```text
             Zone Controller
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
      Node        Node        Node
```

---

# 42. Hardware recomendado Zone Controller

Recomendación:

```text
ESP32-S3
```

con:

```text
PSRAM
Ethernet opcional
Wi-Fi
RS485
CAN
microSD opcional
```

No es obligatorio que todos estén presentes.

---

# 43. Funciones Zone Controller

Puede proporcionar:

* automatizaciones locales;
* cache de estados;
* coordinación;
* reglas;
* historial local;
* configuración;
* gateway;
* sincronización con Central.

Debe continuar funcionando si Central está desconectado.

---

# 44. Central

La Central es el nodo de mayor capacidad lógica.

Recomendación:

```text
ESP32-S3
```

con:

```text
PSRAM
Ethernet
Flash
microSD opcional
```

Para futuras plataformas de mayor capacidad:

```text
ESP32-P4
```

puede evaluarse como plataforma Central avanzada.

---

# 45. Central — funciones

La Central puede proporcionar:

```text
Web UI
REST API
WebSocket
User Management
Roles
Device Registry
Entity Registry
Automation Registry
Scene Registry
History
Discovery
Provisioning
Integrations
Firmware Management
Diagnostics
```

---

# 46. Central no es obligatorio

Principio fundamental:

> **El Central administra el sistema, pero no es dueño de la capacidad de funcionamiento del sistema.**

Por lo tanto:

```text
Central OFF
   ↓
Zone Controllers
   ↓
Nodes
   ↓
Critical automation continues
```

---

# 47. Jerarquía de nodos

La arquitectura general es:

```text
                         CENTRAL
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
        ZONE CONTROLLER  ZONE CONTROLLER  GATEWAY
             │              │
       ┌─────┼─────┐   ┌────┼────┐
       ▼     ▼     ▼   ▼    ▼    ▼
      Node  Node  Node Node Node Node
```

---

# 48. Nodos sin Zone Controller

Una instalación pequeña puede utilizar:

```text
Central
  │
  ├── Node
  ├── Node
  └── Node
```

No debe ser obligatorio implementar todos los niveles.

---

# 49. Nodos autónomos

También puede existir:

```text
Node
```

sin Central.

Ejemplo:

```text
Sensor
  ↓
Local automation
  ↓
Relay
```

Esto es importante para:

* instalaciones pequeñas;
* dispositivos aislados;
* sistemas industriales;
* sistemas marinos;
* instalaciones temporales.

---

# 50. Selección del MCU

La selección debe basarse en las necesidades.

## ESP32-WROOM

Recomendado para:

```text
GPIO
Wi-Fi
Bluetooth
Ethernet mediante PHY
automatización general
```

---

## ESP32-WROVER

Recomendado cuando se necesita:

```text
más memoria
PSRAM
procesamiento adicional
```

---

## ESP32-S3

Recomendado para:

```text
PSRAM
Cámara
AI ligera
Display
Touch
HMI
Ethernet W5500
```

---

## ESP32-C3

Recomendado para:

```text
nodos económicos
sensores
actuadores
bajo consumo
Wi-Fi
Bluetooth LE
```

---

## ESP32-C6

Recomendado para:

```text
Wi-Fi 6
Thread
Zigbee
Matter
nodos inalámbricos
gateways
```

---

## ESP32-C5

Recomendado para aplicaciones que necesiten sus capacidades inalámbricas específicas y que sean compatibles con el diseño del producto.

---

## ESP32-H2

Recomendado para:

```text
Thread
Zigbee
Matter
nodos de muy bajo consumo
```

normalmente acompañado por otro dispositivo cuando se necesite Wi-Fi/Ethernet.

---

## ESP32-P4

Recomendado para:

```text
alto rendimiento
HMI avanzada
cámara
AI más exigente
procesamiento local
Central avanzada
```

La conectividad de red debe resolverse mediante los periféricos/interfaces correspondientes al diseño concreto.

---

# 51. Tabla de selección

| Necesidad              | MCU recomendado                            |
| ---------------------- | ------------------------------------------ |
| Nodo básico            | ESP32-C3 / C6                              |
| Automatización general | ESP32-WROOM                                |
| Ethernet nativo        | ESP32-WROOM / plataformas compatibles      |
| Ethernet SPI           | ESP32-S3 / C3 / C6 / otras                 |
| RS485                  | Cualquiera compatible                      |
| CAN                    | MCU con controlador CAN/TWAI o externo     |
| Muchos GPIO            | WROOM/S3 + expansores                      |
| Cámara                 | ESP32-S3                                   |
| Cámara + AI            | ESP32-S3                                   |
| Display avanzado       | ESP32-S3                                   |
| PSRAM                  | ESP32-S3 / WROVER / P4 según diseño        |
| Thread                 | ESP32-C6 / H2                              |
| Zigbee                 | ESP32-C6 / H2                              |
| Matter                 | C6 / H2 / S3 según transporte y aplicación |
| Bajo consumo           | C3 / C6 / H2                               |
| Zone Controller        | ESP32-S3                                   |
| Central                | ESP32-S3                                   |
| Central avanzada       | ESP32-P4                                   |

---

# 52. Interfaces de comunicación

Los nodos deben utilizar interfaces estandarizadas.

```text
Ethernet
Wi-Fi
Thread
Zigbee
Matter
CAN
CAN-FD mediante controlador correspondiente
RS485
Modbus
UART
SPI
I²C
USB
```

---

# 53. Ethernet como backbone

Para instalaciones permanentes:

```text
Ethernet
```

debe considerarse una opción preferente cuando se necesite:

* baja latencia;
* alimentación separada;
* estabilidad;
* infraestructura fija;
* mayor disponibilidad.

---

# 54. Wi-Fi

Adecuado para:

```text
sensores
actuadores
dispositivos móviles
instalaciones existentes
```

No debe ser la única opción cuando una función crítica pueda utilizar una comunicación cableada.

---

# 55. CAN

Preferente para:

```text
maquinaria
vehículos
marino
instalaciones robustas
redes distribuidas
```

---

# 56. RS485

Preferente para:

```text
sensores industriales
Modbus
distancias largas
instrumentación
medidores
```

---

# 57. SPI

Principalmente para comunicación local con:

```text
W5500
Displays
SD
Expansores
ADC/DAC
controladores
```

---

# 58. I²C

Principalmente para:

```text
sensores
RTC
MCP23017
touch
EEPROM
displays/controladores
```

---

# 59. Alimentación

Los nodos deben diseñarse considerando:

```text
entrada
protección
regulación
consumo
picos
brownout
```

Opciones:

```text
5 V
12 V
24 V
PoE
USB
batería
```

---

# 60. Nodos industriales

Para ambientes industriales:

```text
24 VDC
```

debe considerarse una alimentación de referencia.

La placa puede incorporar:

```text
24 V → protección → buck → 5 V → 3.3 V
```

con protección adecuada.

---

# 61. PoE

Los nodos Ethernet pueden incorporar PoE.

Arquitectura:

```text
Ethernet
    │
PoE
    │
PD Controller
    │
Power Supply
    │
ESP32
```

Debe tratarse como una característica del Hardware Profile:

```text
HAS_POE
```

---

# 62. Watchdog

Los nodos deben utilizar watchdog cuando corresponda.

Especialmente:

```text
Zone Controller
Central
Gateway
Ethernet Node
CAN Node
RS485 Node
```

El watchdog no debe ocultar errores de software.

Debe utilizarse como mecanismo de recuperación.

---

# 63. RTC

Nodos que necesiten autonomía temporal pueden incorporar:

```text
RTC externo
```

Ejemplos:

```text
DS3231
PCF8563
```

o utilizar RTC interno cuando la precisión requerida lo permita.

---

# 64. Almacenamiento

Dependiendo del nodo:

```text
NVS
LittleFS
Flash
microSD
SDMMC
```

## NVS

Configuración pequeña.

## LittleFS

Archivos y configuración local.

## microSD

Historial, imágenes, logs y datasets grandes.

---

# 65. Conectores

Los nodos de referencia deben priorizar conectores mantenibles.

Ejemplos:

```text
Phoenix / terminal block
JST
Molex
Header
RJ45
USB
```

La selección depende del tipo de nodo.

---

# 66. LEDs de diagnóstico

Los nodos de referencia deberían incluir indicadores cuando sea posible.

Ejemplo:

```text
PWR
STATUS
NETWORK
ERROR
USER
```

Los significados deben ser uniformes.

---

# 67. LED STATUS

Ejemplo de estados:

```text
Apagado
    → sin alimentación / módulo detenido

Verde fijo
    → sistema operativo

Verde parpadeando
    → actividad

Amarillo
    → advertencia / configuración

Rojo
    → error crítico
```

Los patrones exactos deben definirse en la documentación de cada nodo.

---

# 68. Botón de servicio

Los nodos de referencia pueden incorporar:

```text
BOOT
RESET
USER
FACTORY
```

El botón de Factory Reset debe requerir una acción deliberada.

Ejemplo:

```text
Mantener 5-10 segundos
```

según el hardware concreto.

---

# 69. Seguridad física

Los nodos deben contemplar:

* protección ESD;
* protección de alimentación;
* watchdog;
* brownout;
* protección de entradas;
* protección de salidas;
* aislamiento cuando sea necesario.

---

# 70. Pinout

Cada nodo debe tener un pinout documentado.

Ejemplo:

```text
GPIO
│
├── GPIO0 → BOOT
├── GPIO2 → STATUS LED
├── GPIO4 → W5500 CS
├── GPIO5 → W5500 RESET
├── GPIO18 → SPI SCLK
├── GPIO19 → SPI MISO
├── GPIO23 → SPI MOSI
└── ...
```

Los pines deben tratarse como parte del Hardware Profile.

---

# 71. Pin Conflict

Nunca se debe asignar un pin sin comprobar:

```text
boot straps
flash
PSRAM
USB
Ethernet
SPI
I²C
UART
ADC
PWM
```

especialmente en:

```text
ESP32-S3
ESP32-C6
ESP32-P4
```

---

# 72. Recursos compartidos

Cuando un bus tenga varios dispositivos:

```text
SPI
 ├── W5500
 ├── Display
 └── SD
```

debe existir un administrador del bus.

Los nodos de referencia deben evitar diseños donde un periférico monopolice el bus sin necesidad.

---

# 73. Expansión

Los nodos deben permitir expansión.

Ejemplo:

```text
Node
 │
 ├── I²C
 ├── SPI
 ├── UART
 ├── GPIO
 └── Expansion Connector
```

Esto permite agregar posteriormente:

```text
sensor
expander
display
RS485
CAN
```

sin rediseñar completamente la placa.

---

# 74. Modularidad física

Siempre que sea posible:

```text
Main Board
   +
Communication Module
   +
I/O Module
   +
Power Module
```

Esto facilita:

* mantenimiento;
* reemplazo;
* variantes;
* fabricación;
* escalabilidad.

---

# 75. Hardware Profile

Cada nodo debe definir un Hardware Profile.

Ejemplo:

```json
{
  "profile": "NODE_ETHERNET_S3",
  "mcu": "ESP32-S3",
  "ethernet": "W5500",
  "psram": true,
  "interfaces": {
    "spi": 2,
    "i2c": 1,
    "uart": 2
  }
}
```

---

# 76. Build Profile

El Build Profile correspondiente podría ser:

```ini
[env:node_ethernet_s3]

build_flags =
    -D BOARD_NODE_ETHERNET_S3
    -D MCU_ESP32_S3
    -D ETH_W5500
    -D HAS_PSRAM
```

---

# 77. Relación Hardware Profile / Build Profile

```text
Build Profile
     ↓
Compilación
     ↓
Hardware Profile
     ↓
Runtime
```

El Build Profile determina la implementación.

El Hardware Profile describe el hardware disponible al sistema.

---

# 78. Nodos de referencia oficiales

La primera familia de referencia debería ser:

```text
REF-NODE-01  Basic
REF-NODE-02  Ethernet
REF-NODE-03  RS485
REF-NODE-04  CAN
REF-NODE-05  IO
REF-NODE-06  Sensor
REF-NODE-07  Display
REF-NODE-08  Camera AI
REF-NODE-09  Zone Controller
REF-NODE-10  Central
REF-NODE-11  Gateway
REF-NODE-12  Energy
```

---

# 79. REF-NODE-01 — Basic

```text
MCU:
ESP32-C3 / ESP32-C6

Connectivity:
Wi-Fi

Interfaces:
GPIO
I²C
SPI
UART
ADC

Uso:
sensores
relés
botones
actuadores
```

---

# 80. REF-NODE-02 — Ethernet

```text
MCU:
ESP32-WROOM

Ethernet:
LAN8720A

Interfaces:
RMII
I²C
SPI
UART
GPIO
```

Variante:

```text
MCU:
ESP32-S3

Ethernet:
W5500
```

---

# 81. REF-NODE-03 — RS485

```text
MCU:
ESP32-WROOM / S3 / C6

UART
RS485
TVS
Protección
Terminación configurable
```

---

# 82. REF-NODE-04 — CAN

```text
MCU:
ESP32 con CAN/TWAI
```

o:

```text
MCU
 ↓
SPI
 ↓
CAN Controller
```

con:

```text
CAN Transceiver
```

---

# 83. REF-NODE-05 — IO

```text
MCU:
ESP32-WROOM / S3

Expanders:
MCP23017
MCP23S17
74HC595
74HC165
```

Debe soportar configuración desde la UI.

---

# 84. REF-NODE-06 — Sensor

```text
MCU:
ESP32-C3 / C6

Sensores:
I²C
1-Wire
ADC
SPI
RS485 opcional
```

Optimizado para bajo consumo.

---

# 85. REF-NODE-07 — Display

```text
MCU:
ESP32-S3

Display:
SPI

Touch:
I²C / SPI

PSRAM:
recomendado
```

---

# 86. REF-NODE-08 — Camera AI

```text
MCU:
ESP32-S3 / P4

PSRAM:
Sí

Camera:
Sí

Storage:
microSD opcional

Network:
Wi-Fi / Ethernet W5500
```

---

# 87. REF-NODE-09 — Zone Controller

```text
MCU:
ESP32-S3

PSRAM:
Sí

Ethernet:
preferente

Wi-Fi:
Sí

RS485:
opcional

CAN:
opcional

microSD:
opcional
```

Funciones:

```text
automation
local state
gateway
cache
history
```

---

# 88. REF-NODE-10 — Central

```text
MCU:
ESP32-S3

PSRAM:
Sí

Ethernet:
preferente

microSD:
recomendado según historial

Wi-Fi:
Sí
```

Funciones:

```text
API
Web UI
Users
Roles
Entities
Automations
Scenes
Integrations
Discovery
```

---

# 89. REF-NODE-11 — Gateway

Puede utilizar:

```text
ESP32-C6
ESP32-S3
ESP32-P4
```

según protocolos.

Ejemplo:

```text
RS485
 +
Ethernet
```

o:

```text
CAN
 +
Ethernet
```

o:

```text
Thread
 +
Ethernet
```

---

# 90. REF-NODE-12 — Energy

```text
MCU:
ESP32-S3 / WROOM

Inputs:
Voltage
Current
Power
Energy

Communication:
RS485 / Modbus
Ethernet
Wi-Fi
```

Debe incluir aislamiento y protección apropiados cuando mida red eléctrica.

---

# 91. Recomendación de arquitectura de PCB

Una placa de referencia puede dividirse conceptualmente:

```text
┌──────────────────────────────────────┐
│              MCU                     │
│                                      │
│  Power      Communication            │
│    │             │                   │
│    │       ┌─────┴─────┐             │
│    │       │ Ethernet  │             │
│    │       │ RS485     │             │
│    │       │ CAN       │             │
│    │       └───────────┘             │
│    │                                  │
│    └─────── Power / Protection        │
│                                      │
│  I²C / SPI / GPIO / Expansion        │
└──────────────────────────────────────┘
```

---

# 92. Separación de alimentación

La alimentación debe separarse conceptualmente:

```text
Input Power
     ↓
Protection
     ↓
DC/DC
     ↓
5 V
     ↓
3.3 V
     ↓
MCU
```

Las cargas de potencia no deben contaminar directamente la alimentación del MCU.

---

# 93. Tierra

Cuando existan:

```text
relays
motors
PWM power
Ethernet
RS485
CAN
```

debe estudiarse cuidadosamente:

* retorno de corriente;
* planos de tierra;
* ruido;
* aislamiento;
* rutas de potencia.

---

# 94. Protección de comunicaciones

Para interfaces externas se recomienda evaluar:

```text
TVS
ESD protection
common mode protection
filtros
terminación
aislamiento
```

según protocolo.

---

# 95. Protección RS485

Referencia conceptual:

```text
Connector
   ↓
TVS
   ↓
Protection
   ↓
RS485 Transceiver
   ↓
UART
   ↓
ESP32
```

---

# 96. Protección CAN

Conceptualmente:

```text
CAN Connector
     ↓
ESD / TVS
     ↓
CAN Transceiver
     ↓
MCU
```

---

# 97. Protección Ethernet

Conceptualmente:

```text
RJ45
 ↓
Magnetics
 ↓
PHY / W5500
 ↓
MCU
```

En el diseño deben respetarse las recomendaciones del fabricante del PHY/controlador y del magnetics/RJ45 utilizado.

---

# 98. EMC / EMI

Los nodos destinados a instalaciones industriales deben diseñarse considerando:

```text
EMI
EMC
ESD
Surge
EFT
Grounding
Shielding
```

La implementación concreta depende del entorno y normativa aplicable.

---

# 99. Temperatura

Los nodos deben poder informar:

```text
MCU temperature
board temperature
external sensor temperature
```

cuando el hardware lo permita.

Los nodos instalados en ambientes extremos pueden requerir:

```text
ventilación
disipación
componentes adecuados
```

---

# 100. Identificación física

Cada nodo debe tener:

```text
Device ID
MAC
QR Code opcional
Serial Number
Hardware Revision
```

Ejemplo:

```text
NODE-000123
HW: 1.2
FW: 1.4.0
```

---

# 101. Hardware Revision

La PCB debe incorporar una revisión:

```text
REV A
REV B
REV C
```

El firmware debe poder conocerla.

Ejemplo:

```json
{
  "hardware": {
    "model": "NODE_ETHERNET",
    "revision": "B"
  }
}
```

---

# 102. Compatibilidad firmware/hardware

El firmware debe validar:

```text
MCU
Board
Hardware Revision
Build Profile
```

antes de utilizar hardware específico.

---

# 103. Test de fabricación

Los nodos de referencia deben permitir pruebas de fábrica.

Secuencia:

```text
Power
 ↓
MCU
 ↓
Flash
 ↓
GPIO
 ↓
I²C
 ↓
SPI
 ↓
UART
 ↓
Ethernet/Wi-Fi
 ↓
LED
 ↓
Button
 ↓
Final Test
```

---

# 104. Manufacturing Test Mode

Se recomienda un modo:

```text
FACTORY_TEST
```

que permita comprobar:

```text
GPIO
ADC
PWM
I²C
SPI
UART
Ethernet
CAN
RS485
Display
Touch
Storage
```

sin cargar la aplicación completa.

---

# 105. Diagnostic Mode

También debe existir:

```text
DIAGNOSTIC_MODE
```

para instalaciones.

Permite:

```text
test GPIO
test network
test sensors
test communication
test display
test storage
```

---

# 106. Modo mantenimiento

El sistema debe soportar:

```text
MAINTENANCE
```

En este modo:

* se pueden deshabilitar automatizaciones;
* se pueden probar actuadores;
* se pueden ejecutar diagnósticos;
* se puede cambiar hardware;
* se puede actualizar firmware.

Las funciones críticas de seguridad pueden permanecer activas.

---

# 107. Expansión futura

Los nodos deben dejar espacio para:

```text
CAN
RS485
Ethernet
W5500
SD
RTC
expansores
sensores
```

cuando sea viable.

No es necesario montar todos los componentes en todas las placas.

---

# 108. Filosofía de variantes

En lugar de fabricar:

```text
20 placas completamente diferentes
```

se recomienda mantener una familia:

```text
Base Board
   │
   ├── Basic
   ├── Ethernet
   ├── RS485
   ├── CAN
   ├── IO
   ├── Sensor
   └── HMI
```

con componentes opcionales.

---

# 109. Coste

La selección de hardware debe considerar:

```text
costo
disponibilidad
consumo
complejidad
mantenimiento
fabricación
stock
```

No siempre debe elegirse el MCU más potente.

---

# 110. Regla de selección

> **Utilizar el hardware mínimo que pueda cumplir la función con margen suficiente.**

Ejemplo:

```text
Sensor simple
→ ESP32-C3
```

No necesariamente:

```text
ESP32-P4
```

Mientras:

```text
Camera + AI + HMI
→ ESP32-S3 / P4
```

---

# 111. Nodos y escalabilidad

El sistema debe permitir pasar de:

```text
1 nodo
```

a:

```text
10 nodos
```

```text
50 nodos
```

```text
100+ nodos
```

sin cambiar el modelo lógico.

La diferencia debe estar principalmente en:

```text
topología
comunicación
Central
Zone Controllers
```

---

# 112. Ejemplo de instalación pequeña

```text
                    Router
                      │
                  Ethernet
                      │
                  Central
                  ESP32-S3
                      │
             ┌────────┼────────┐
             ▼        ▼        ▼
          Sensor    Relay     HMI
```

---

# 113. Ejemplo de instalación mediana

```text
                       Central
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
          Zone A       Zone B       Zone C
             │            │            │
          Nodes        Nodes        Nodes
```

---

# 114. Ejemplo industrial

```text
                     Central
                        │
                    Ethernet
                        │
                 Zone Controller
                        │
             ┌──────────┼──────────┐
             ▼          ▼          ▼
           CAN        RS485      Ethernet
             │          │          │
          Nodes       Nodes       Nodes
```

---

# 115. Ejemplo agrícola

```text
                 Central / Zone
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Sensor       RS485       Valve
       Node          Node        Node
          │           │           │
     Soil/Weather   Modbus      Pump
```

---

# 116. Ejemplo marino

```text
                    Central
                       │
                    CAN Bus
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      Engine Node   Energy Node   Safety
          │            │            │
        Sensors      Battery      Alarms
```

---

# 117. Ejemplo residencial

```text
                    Central
                       │
       ┌───────────────┼────────────────┐
       ▼               ▼                ▼
    Lighting         Security          HVAC
       │               │                │
     Nodes            Nodes            Nodes
```

---

# 118. Selección de comunicación por aplicación

| Aplicación    | Principal        | Alternativa |
| ------------- | ---------------- | ----------- |
| Sensor simple | Wi-Fi            | Thread      |
| Batería       | Thread / Zigbee  | Wi-Fi sleep |
| Iluminación   | Wi-Fi / Ethernet | Matter      |
| Industrial    | Ethernet / RS485 | CAN         |
| Maquinaria    | CAN              | Ethernet    |
| Marino        | CAN              | Ethernet    |
| Agricultura   | RS485            | CAN         |
| HMI           | Ethernet         | Wi-Fi       |
| Camera AI     | Ethernet/Wi-Fi   | —           |
| Central       | Ethernet         | Wi-Fi       |
| Gateway       | Ethernet         | Wi-Fi       |

---

# 119. Principio Local-First

Cada nodo debe poder ejecutar las funciones que sean razonables para su categoría.

Ejemplo:

```text
Sensor
    ↓
medición local
```

```text
Actuator
    ↓
control local
```

```text
Zone Controller
    ↓
automation local
```

```text
Central
    ↓
coordination
```

---

# 120. Principio de tolerancia a fallos

La arquitectura debe soportar:

```text
Internet OFF
Central OFF
Zone Controller OFF
Node OFF
Network segment OFF
```

sin producir un fallo global.

---

# 121. Estado de disponibilidad

Cada nodo debe informar:

```text
ONLINE
OFFLINE
DEGRADED
MAINTENANCE
UPDATING
ERROR
```

La Central debe distinguir:

```text
node unavailable
```

de:

```text
entity unavailable
```

---

# 122. Redundancia futura

Los nodos de referencia deben dejar abierta la posibilidad de:

```text
Central A
Central B
```

o:

```text
Zone Controller A
Zone Controller B
```

No es obligatorio para la primera versión.

---

# 123. Firmware común

Todos los nodos deben utilizar la misma arquitectura de firmware:

```text
Boot
 ↓
Hardware Detection
 ↓
Build Profile
 ↓
Configuration
 ↓
Module Registry
 ↓
Device Model
 ↓
System Bus
 ↓
Services
 ↓
Application
```

---

# 124. Boot

El bootloader debe:

* validar firmware;
* validar configuración cuando corresponda;
* iniciar watchdog;
* iniciar hardware mínimo;
* permitir recuperación;
* permitir OTA.

El boot no debe depender de Central.

---

# 125. Recovery

Ante una configuración inválida:

```text
Boot
 ↓
Configuration validation
 ↓
Invalid
 ↓
Safe Mode
 ↓
Web UI
 ↓
Repair
```

---

# 126. Safe Mode

El Safe Mode debe proporcionar:

```text
Network
Web UI
Diagnostics
Configuration
Firmware Update
Reset Configuration
```

y minimizar la activación de actuadores potencialmente peligrosos.

---

# 127. Seguridad de actuadores

Al arrancar:

```text
Relay
Motor
Valve
Dimmer
```

deben pasar a un estado seguro definido por el Hardware Profile.

Ejemplo:

```text
relay → OFF
motor → STOP
valve → CLOSED
```

cuando corresponda.

---

# 128. Estados seguros configurables

El estado seguro no debe ser universal.

Puede configurarse:

```text
safe_state
```

por recurso.

Ejemplo:

```json
{
  "resource": "relay.pump",
  "safe_state": false
}
```

---

# 129. Actualización OTA

Los nodos deben poder recibir:

```text
firmware
configuration
module updates
```

cuando corresponda.

La compatibilidad debe comprobar:

```text
MCU
Board
Hardware Revision
Build Profile
Firmware Version
```

---

# 130. Documentación del nodo

Cada nodo físico debe tener:

```text
README
SCHEMATIC
PINOUT
BOM
ASSEMBLY
TEST
FIRMWARE
BUILD PROFILE
```

---

# 131. Estructura recomendada del repositorio

```text
hardware/
│
├── reference-nodes/
│
│   ├── REF-NODE-01-BASIC/
│   │   ├── README.md
│   │   ├── schematic/
│   │   ├── pcb/
│   │   ├── bom/
│   │   └── test/
│   │
│   ├── REF-NODE-02-ETHERNET/
│   ├── REF-NODE-03-RS485/
│   ├── REF-NODE-04-CAN/
│   ├── REF-NODE-05-IO/
│   ├── REF-NODE-06-SENSOR/
│   ├── REF-NODE-07-DISPLAY/
│   ├── REF-NODE-08-CAMERA-AI/
│   ├── REF-NODE-09-ZONE/
│   ├── REF-NODE-10-CENTRAL/
│   ├── REF-NODE-11-GATEWAY/
│   └── REF-NODE-12-ENERGY/
```

---

# 132. BOM

Cada referencia física debe mantener:

```text
Reference
Part Number
Manufacturer
Package
Quantity
Alternative
Availability
```

Cuando sea posible, deben existir componentes alternativos.

---

# 133. Componentes equivalentes

La arquitectura no debe depender innecesariamente de un único fabricante.

Ejemplo:

```text
LAN8720A
IP101
```

pueden cumplir una función similar si el driver y hardware lo permiten.

Lo mismo:

```text
SP3485
MAX3485
```

para determinadas aplicaciones RS485.

---

# 134. Componentes críticos

Los componentes cuya sustitución cambie:

* pinout;
* protocolo;
* comportamiento eléctrico;
* driver;

deben considerarse variantes distintas del Hardware Profile.

---

# 135. Hardware Profile ID

Ejemplo:

```text
NODE_ETH_WROOM_LAN8720_REV_A
NODE_ETH_S3_W5500_REV_A
NODE_RS485_WROOM_REV_A
NODE_CAN_S3_REV_A
NODE_SENSOR_C6_REV_A
```

---

# 136. Compatibilidad de hardware

La compatibilidad se expresa:

```text
Hardware Profile
+
Build Profile
+
Firmware Version
```

Ejemplo:

```text
NODE_ETH_S3_W5500_REV_A
+
controller_s3
+
FW 1.2.x
```

---

# 137. No asumir un único pinout

Dos placas con:

```text
ESP32-S3
```

pueden tener:

```text
GPIO
SPI
PSRAM
USB
Display
```

en pines diferentes.

Por eso:

> **MCU ≠ Board ≠ Hardware Profile**

---

# 138. Regla para placas personalizadas

Toda placa personalizada debe definir:

```text
BOARD_*
```

Ejemplo:

```ini
-D BOARD_NODE_ETHERNET_S3
```

No debe utilizar simplemente:

```ini
-D BOARD_ESP32_S3
```

si el pinout y periféricos físicos son específicos de la placa.

---

# 139. Relación con BUILD-PROFILES.md

Este documento define:

```text
qué nodos físicos recomendamos
```

Mientras `BUILD-PROFILES.md` define:

```text
cómo el firmware selecciona las variantes durante la compilación
```

Ambos deben mantenerse sincronizados.

---

# 140. Relación con MODULE-DEVELOPMENT.md

Los nodos proporcionan:

```text
Hardware
Resources
Interfaces
Capabilities
```

Los módulos proporcionan:

```text
Functions
Entities
Services
Commands
Events
```

---

# 141. Relación con DEVICE-MODEL.md

La representación final debe ser:

```text
Reference Node
      ↓
Device
      ↓
Resources
      ↓
Capabilities
      ↓
Entities
```

Por ejemplo:

```text
REF-NODE-02
      ↓
device.garage_controller
      ↓
ethernet
gpio
temperature
relay
      ↓
switch.garage_light
sensor.garage.temperature
```

---

# 142. Relación con SYSTEM-BUS

Los nodos se comunican mediante el System Bus lógico.

Ejemplo:

```text
Sensor Node
    ↓
temperature.updated
    ↓
System Bus
    ↓
Zone Controller
    ↓
HVAC Module
```

El transporte puede ser:

```text
Wi-Fi
Ethernet
CAN
RS485
Thread
Matter
```

sin modificar el evento lógico.

---

# 143. Principio de transporte independiente

El mismo evento:

```text
sensor.temperature.updated
```

puede viajar por:

```text
Ethernet
Wi-Fi
CAN
RS485
Thread
```

El modelo lógico permanece igual.

---

# 144. Evolución de los nodos

La familia de referencia debe evolucionar por revisiones:

```text
REV A
 ↓
REV B
 ↓
REV C
```

pero manteniendo compatibilidad siempre que sea posible.

---

# 145. Compatibilidad hacia atrás

Cuando una revisión de hardware modifica:

```text
GPIO
periférico
driver
```

debe incrementarse el Hardware Revision.

El firmware debe poder identificarla.

---

# 146. Compatibilidad hacia adelante

Cuando sea posible:

```text
FW nuevo
```

debe continuar soportando:

```text
Hardware antiguo
```

mediante perfiles separados.

---

# 147. Principio de diseño

> **No diseñar una placa solamente para el firmware actual.**

Debe reservarse capacidad para:

* futuras interfaces;
* diagnóstico;
* expansión;
* revisiones;
* nuevas versiones de módulos.

---

# 148. Recomendación de primera generación

La primera familia física de referencia debería priorizar:

### Nodo económico

```text
ESP32-C3/C6
Wi-Fi
GPIO
I²C
SPI
```

### Nodo Ethernet

```text
ESP32-WROOM
LAN8720A
```

### Nodo Ethernet avanzado

```text
ESP32-S3
W5500
PSRAM
```

### Nodo industrial

```text
ESP32-S3/WROOM
RS485
CAN
Ethernet
```

### Nodo HMI

```text
ESP32-S3
PSRAM
Display
Touch
```

### Nodo AI

```text
ESP32-S3
PSRAM
Camera
Ethernet/Wi-Fi
```

### Zone Controller

```text
ESP32-S3
PSRAM
Ethernet
RS485
CAN
microSD opcional
```

### Central

```text
ESP32-S3
PSRAM
Ethernet
microSD opcional
```

---

# 149. Principio económico

No todos los nodos deben tener:

```text
Ethernet
CAN
RS485
PSRAM
Camera
Display
```

al mismo tiempo.

La plataforma debe favorecer:

```text
hardware mínimo necesario
+
módulos adecuados
+
expansión cuando sea necesaria
```

---

# 150. Principio final

La plataforma debe permitir construir:

```text
┌──────────────────────────────────────────┐
│                CENTRAL                   │
│             ESP32-S3 / P4               │
└────────────────────┬─────────────────────┘
                     │
              Ethernet / Wi-Fi
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
   Zone Controller  Gateway    Zone Controller
        │            │            │
   ┌────┼────┐       │       ┌────┼────┐
   ▼    ▼    ▼       ▼       ▼    ▼    ▼
 Sensor IO  HMI     RS485    CAN  IO   AI
 Node  Node Node    Node     Node Node Node
```

Todos estos nodos deben compartir:

```text
Device Model
Module Architecture
System Bus
API
Configuration
Security
Diagnostics
```

---

# 151. Regla de oro

> **Un nodo de referencia es una implementación física recomendada de una función del sistema, no una definición de la arquitectura lógica.**

El sistema debe seguir funcionando conceptualmente aunque se cambie:

```text
ESP32-WROOM
→ ESP32-S3

LAN8720
→ W5500

MCP23017
→ MCP23S17

RS485
→ CAN

Wi-Fi
→ Ethernet
```

siempre que exista una capacidad equivalente.

---

# 152. Principio arquitectónico definitivo

```text
                   HARDWARE
                      │
                      ▼
              REFERENCE NODE
                      │
                      ▼
              HARDWARE PROFILE
                      │
                      ▼
               BUILD PROFILE
                      │
                      ▼
                    DRIVER
                      │
                      ▼
                     HAL
                      │
                      ▼
                   MODULE
                      │
                      ▼
                  RESOURCE
                      │
                      ▼
                 CAPABILITY
                      │
                      ▼
                   ENTITY
                      │
                      ▼
                SYSTEM BUS
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       SERVICE    AUTOMATION    API
          │           │           │
          └───────────┼───────────┘
                      ▼
                 INTEGRATIONS
```

---

# 153. Conclusión

Los nodos de referencia proporcionan una familia de hardware coherente para desarrollar y validar la plataforma.

La estrategia oficial es:

```text
Hardware de referencia
        ↓
Hardware Profile
        ↓
Build Profile
        ↓
Driver / HAL
        ↓
Module
        ↓
Device Model
        ↓
System Bus
        ↓
Application
```

La plataforma no debe quedar atada a una única placa ni a un único ESP32.

Debe poder evolucionar desde un nodo simple de bajo costo hasta:

```text
nodos industriales
nodos Ethernet
nodos CAN
nodos RS485
nodos de sensores
nodos de energía
nodos HMI
nodos de cámara
nodos AI
gateways
Zone Controllers
Central
```

manteniendo la misma arquitectura lógica.

> **Los nodos pueden cambiar.**
>
> **Los microcontroladores pueden cambiar.**
>
> **Los buses físicos pueden cambiar.**
>
> **Los drivers pueden cambiar.**
>
> **Pero el modelo lógico de la plataforma debe permanecer estable.**

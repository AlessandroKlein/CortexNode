# Plataforma de Automatización Distribuida

> **Principio general:** Simple por fuera. Modular por dentro. Distribuida por diseño.

Sistema modular de automatización basado principalmente en microcontroladores **ESP32**, desarrollado con **PlatformIO**, orientado a viviendas, oficinas, edificios, comercios, laboratorios, instalaciones agroindustriales e instalaciones industriales ligeras.

El objetivo no es crear solamente un dispositivo IoT, sino desarrollar una **plataforma de automatización distribuida similar conceptualmente a un PLC modular**, donde múltiples dispositivos puedan comunicarse entre sí, compartir información, ejecutar automatizaciones y ser administrados desde una interfaz central.

El sistema debe permitir comenzar con:

```text
1 ESP32
1 habitación
1 sensor
1 relé
```

y evolucionar progresivamente hacia:

```text
100+ dispositivos
múltiples habitaciones
múltiples buses industriales
Ethernet
Wi-Fi
CAN/CANopen
Modbus RTU
Modbus TCP
Thread
Zigbee
Matter
pantallas táctiles
gateways
servidores centrales
APIs
automatizaciones complejas
analítica
IA local
```

sin tener que rediseñar completamente la arquitectura.

---

# 1. Descripción general

El proyecto busca desarrollar una plataforma capaz de controlar, medir, supervisar y automatizar elementos físicos mediante una red de dispositivos distribuidos.

Ejemplos:

* iluminación;
* dimmers;
* ventiladores;
* motores;
* persianas;
* cortinas;
* puertas;
* portones;
* cerraduras;
* alarmas;
* sensores ambientales;
* sensores meteorológicos;
* sensores de consumo eléctrico;
* sensores de agua;
* bombas;
* válvulas;
* calefacción;
* climatización;
* riego;
* inversores;
* baterías;
* sensores industriales;
* maquinaria;
* actuadores CAN;
* dispositivos Modbus;
* dispositivos Zigbee;
* dispositivos Matter;
* interfaces táctiles;
* paneles de control;
* gateways;
* dispositivos externos.

El sistema debe ser:

* modular;
* distribuido;
* escalable;
* configurable;
* interoperable;
* seguro;
* tolerante a fallos;
* mantenible;
* documentado;
* extensible;
* compatible con diferentes generaciones de ESP32.

---

# 2. Objetivo principal

Crear una plataforma que permita construir sistemas de automatización sin que cada proyecto tenga que comenzar desde cero.

La idea fundamental es:

```text
Hardware
    ↓
Driver
    ↓
Capability
    ↓
Device
    ↓
Service
    ↓
Automation
    ↓
User Interface
    ↓
API / Integraciones
```

El hardware no debe determinar cómo funciona el resto del sistema.

Por ejemplo:

```text
GPIO 23
```

no debería ser tratado por la aplicación simplemente como:

```text
GPIO23 = ventilador
```

sino como:

```text
GPIO23
    ↓
DigitalOutputDriver
    ↓
OutputChannel
    ↓
Capability: switch
    ↓
Device: Fan
    ↓
Service: Climate
    ↓
Automation
```

Esto permite cambiar el hardware sin modificar las automatizaciones.

---

# 3. Filosofía del proyecto

## 3.1 Principios

El proyecto debe seguir estos principios:

1. Simple por fuera.
2. Modular por dentro.
3. Separación estricta entre hardware y lógica.
4. Configuración desde interfaz cuando sea técnicamente posible.
5. No modificar código para tareas que puedan configurarse.
6. Una responsabilidad por módulo.
7. Evitar dependencias innecesarias.
8. Diseñar para crecer.
9. Diseñar para fallar de manera segura.
10. Diseñar para poder actualizarse.
11. Diseñar para poder desactivar funcionalidades.
12. Diseñar para poder agregar funcionalidades.
13. No asumir que existe conexión a Internet.
14. La automatización local debe continuar funcionando sin nube.
15. La pérdida de un dispositivo no debe inutilizar toda la instalación.
16. La comunicación debe estar desacoplada de la aplicación.
17. Las interfaces deben consumir capacidades, no hardware.
18. La configuración debe estar versionada.
19. Toda funcionalidad importante debe disponer de documentación.
20. Todo módulo nuevo debe poder integrarse siguiendo un contrato definido.

---

# 4. Concepto de PLC distribuido

El sistema estará inspirado conceptualmente en un PLC moderno, pero no pretende reemplazar automáticamente un PLC industrial certificado.

Un dispositivo puede actuar como:

* controlador;
* módulo de entradas;
* módulo de salidas;
* módulo de sensores;
* gateway;
* HMI;
* controlador de automatización;
* nodo de comunicación;
* dispositivo de medición;
* bridge entre protocolos.

Ejemplo:

```text
                 ┌─────────────────────────┐
                 │     CONTROL CENTRAL     │
                 │                         │
                 │ Automation Engine       │
                 │ Device Registry         │
                 │ User Interface          │
                 │ API                     │
                 └───────────┬─────────────┘
                             │
                    MQTT / Ethernet
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
          ▼                  ▼                  ▼
    ┌───────────┐      ┌───────────┐      ┌───────────┐
    │ Nodo Luz  │      │ Nodo HVAC │      │ Nodo Agua │
    │ ESP32     │      │ ESP32-S3  │      │ ESP32     │
    └───────────┘      └───────────┘      └───────────┘
          │                  │                  │
        Relays             Dimmer             Flow
        LEDs               Temp               Pump
        Buttons             Humidity           Valve
```

---

# 5. Arquitectura general

La arquitectura se dividirá en capas.

```text
┌─────────────────────────────────────────────┐
│                 USER LAYER                  │
│                                             │
│ Web UI │ Touch UI │ Mobile │ API │ Voice   │
└──────────────────────┬──────────────────────┘
                       │
┌──────────────────────▼──────────────────────┐
│              AUTOMATION LAYER               │
│                                             │
│ Rules │ Scenes │ Routines │ Schedules       │
│ Conditions │ Events │ Triggers │ Actions    │
└──────────────────────┬──────────────────────┘
                       │
┌──────────────────────▼──────────────────────┐
│                SERVICE LAYER                 │
│                                             │
│ Lighting │ Climate │ Security │ Energy      │
│ Water │ Weather │ Media │ Access │ etc.    │
└──────────────────────┬──────────────────────┘
                       │
┌──────────────────────▼──────────────────────┐
│              DEVICE MODEL                   │
│                                             │
│ Devices │ Entities │ Capabilities           │
│ States │ Commands │ Properties              │
└──────────────────────┬──────────────────────┘
                       │
┌──────────────────────▼──────────────────────┐
│             COMMUNICATION LAYER             │
│                                             │
│ MQTT │ HTTP │ WebSocket │ CAN │ Modbus     │
│ Matter │ Thread │ Zigbee │ BLE              │
└──────────────────────┬──────────────────────┘
                       │
┌──────────────────────▼──────────────────────┐
│             HARDWARE ABSTRACTION            │
│                                             │
│ GPIO │ ADC │ PWM │ I2C │ SPI │ UART        │
│ CAN │ Ethernet │ USB │ SD │ Display         │
└──────────────────────┬──────────────────────┘
                       │
┌──────────────────────▼──────────────────────┐
│                  HARDWARE                  │
│                                             │
│ ESP32 │ ESP32-S3 │ ESP32-C6 │ etc.         │
└─────────────────────────────────────────────┘
```

---

# 6. Plataformas ESP32

El proyecto no debe depender de un único modelo de ESP32.

## 6.1 ESP32 clásico

Uso recomendado:

* nodos simples;
* relés;
* sensores;
* entradas digitales;
* salidas digitales;
* Ethernet externa;
* Modbus;
* CAN mediante transceiver;
* controladores económicos.

Ejemplos:

```text
ESP32-WROOM-32
ESP32-WROOM-32E
ESP32-WROOM-32UE
```

---

# 6.2 ESP32-S3

Uso recomendado:

* HMI;
* pantallas;
* interfaces táctiles;
* cámaras;
* procesamiento de imágenes;
* IA ligera;
* reconocimiento de patrones;
* análisis local;
* nodos complejos;
* almacenamiento local;
* procesamiento de hábitos.

Puede utilizarse como:

```text
ESP32-S3
├── Wi-Fi
├── Bluetooth LE
├── Display
├── Touch
├── Camera
├── Sensors
├── AI/ML
└── Automation
```

---

# 6.3 ESP32-C6

Uso recomendado:

* Matter;
* Thread;
* Zigbee;
* dispositivos de bajo consumo;
* gateways;
* bridges;
* nodos inalámbricos;
* sensores distribuidos.

El ESP32-C6 incorpora IEEE 802.15.4 además de Wi-Fi y Bluetooth LE, por lo que es una plataforma especialmente importante para la arquitectura de interoperabilidad del proyecto.

---

# 6.4 Otros ESP32

La arquitectura deberá permitir incorporar posteriormente:

* ESP32-C3;
* ESP32-C5;
* ESP32-H2;
* ESP32-H;
* futuras familias ESP32.

El código de aplicación no debe depender directamente de las capacidades específicas del microcontrolador.

---

# 7. PlatformIO

Todo el firmware debe poder desarrollarse mediante:

```text
PlatformIO
```

La estructura debe permitir múltiples targets.

Ejemplo:

```ini
[env:esp32]
platform = espressif32
framework = arduino

[env:esp32-s3]
platform = espressif32
framework = arduino

[env:esp32-c6]
platform = espressif32
framework = arduino
```

Cuando una funcionalidad requiera ESP-IDF específico, deberá aislarse detrás de una interfaz.

---

# 8. Framework

La implementación inicial podrá utilizar:

```text
Arduino framework
```

sobre ESP32.

Cuando sea necesario utilizar funcionalidades avanzadas:

```text
ESP-IDF
FreeRTOS
```

podrán incorporarse mediante capas de abstracción.

El sistema debe aprovechar:

* FreeRTOS;
* tareas;
* colas;
* mutex;
* event groups;
* timers;
* watchdog;
* task notification;
* almacenamiento persistente.

En ESP32 con múltiples núcleos, las tareas críticas podrán distribuirse mediante task pinning cuando resulte conveniente.

---

# 9. Modelo de dispositivo

Todo dispositivo debe representarse mediante un modelo común.

Ejemplo:

```json
{
  "id": "livingroom-light-01",
  "name": "Luz Living",
  "type": "light",
  "room": "livingroom",
  "manufacturer": "Project",
  "model": "Relay4",
  "firmware": "1.0.0",
  "capabilities": [
    "switch",
    "brightness"
  ],
  "state": {
    "power": true,
    "brightness": 80
  }
}
```

---

# 10. Device ID

Cada dispositivo debe disponer de un identificador único.

Ejemplo:

```text
home.livingroom.light.01
```

o:

```text
ak-living-light-01
```

Se deberá evitar depender exclusivamente de:

* MAC;
* IP;
* GPIO;
* dirección CAN;
* dirección Modbus.

Estos valores pueden cambiar.

---

# 11. Rooms / zonas

Los dispositivos deben poder organizarse en:

```text
Site
 ├── Building
 │    ├── Floor
 │    │    ├── Room
 │    │    │    ├── Device
 │    │    │    └── Sensor
```

Ejemplo:

```text
Casa
├── Planta baja
│   ├── Living
│   ├── Cocina
│   ├── Baño
│   └── Garage
└── Planta alta
    ├── Dormitorio
    └── Oficina
```

Esto será fundamental para automatizaciones y pantallas.

---

# 12. Capabilities

Una capacidad describe qué puede hacer un dispositivo.

Ejemplos:

```text
switch
light
dimmer
temperature
humidity
pressure
motion
presence
door
window
cover
fan
lock
alarm
energy
voltage
current
power
water_flow
water_volume
air_quality
co2
ph
wind
rain
weather
media_player
display
touch
```

Un dispositivo puede tener varias capacidades.

Ejemplo:

```text
Ventilador
├── switch
├── speed
├── temperature
└── power
```

---

# 13. Separación entre dispositivo y capacidad

No debe asumirse:

```text
1 dispositivo = 1 función
```

Puede existir:

```text
ESP32-S3
├── temperature
├── humidity
├── display
├── touch
├── light
├── energy
└── media_player
```

Esto permitirá representar correctamente dispositivos complejos.

---

# 14. Hardware Abstraction Layer

El código de aplicación no debe manipular directamente:

```cpp
digitalWrite()
analogRead()
ledcWrite()
Wire.begin()
SPI.begin()
```

salvo dentro de drivers o HAL.

Ejemplo:

```text
Application
    ↓
OutputChannel
    ↓
GPIO Driver
    ↓
HAL
    ↓
ESP32
```

Esto facilita soportar distintas familias de ESP32.

---

# 15. Sistema de entradas

Debe soportar:

### Digitales

* botones;
* pulsadores;
* interruptores;
* sensores de movimiento;
* contactos magnéticos;
* finales de carrera.

### Analógicas

* 0-3.3 V;
* 0-5 V mediante adaptación;
* sensores resistivos;
* sensores industriales mediante ADC externo.

### Contadores

* pulsos;
* frecuencia;
* encoder;
* caudalímetros.

### Interrupciones

Debe existir un sistema común de:

```text
GPIO Interrupt
External Interrupt
Pulse Counter
Wake-up Source
```

---

# 16. Sistema de salidas

Debe soportar:

* GPIO;
* relés;
* MOSFET;
* PWM;
* dimmers;
* SSR;
* motores;
* servos;
* válvulas;
* actuadores;
* drivers externos.

---

# 17. PWM

Debe existir una abstracción:

```text
PwmChannel
```

que permita:

```text
0%
25%
50%
75%
100%
```

y configuración de:

```text
frecuencia
resolución
polaridad
canal
```

Debe contemplarse PWM por:

* periféricos internos;
* expansores;
* hardware externo;
* drivers dedicados.

---

# 18. Dimmers

El sistema debe diferenciar:

```text
DC PWM
AC Phase Control
AC Zero Crossing
Digital Dimming
```

Nunca se debe asumir que un PWM lógico puede conectarse directamente a una carga de red.

El módulo de dimmer será responsable de:

* frecuencia;
* sincronización;
* detección de cruce por cero;
* ángulo de disparo;
* límites;
* protección;
* estado seguro.

---

# 19. Relés

El sistema debe soportar:

```text
NO
NC
Active High
Active Low
```

y configurar:

```text
startup_state
safe_state
restore_state
```

Ejemplo:

```json
{
  "type": "relay",
  "safe_state": "off",
  "restore_state": "previous"
}
```

---

# 20. Expansores

Debe existir soporte para:

```text
MCP23017
MCP23S17
74HC595
74HCT595
PCA9685
```

y futuros dispositivos.

La configuración debe realizarse desde la interfaz.

Ejemplo:

```text
Expander:
MCP23017

Address:
0x20

Pin:
GPA0

Function:
Relay

Name:
Luz Living
```

---

# 21. Sensores

Los sensores deben seguir un modelo común.

```text
Sensor
├── Driver
├── Measurement
├── Unit
├── Calibration
├── Filtering
├── Update interval
└── Health
```

---

# 22. SEMA como módulo de sensores

La arquitectura de SEMA debe servir como referencia para el sistema de sensores.

Se deben contemplar:

* temperatura;
* humedad;
* presión;
* luminosidad;
* lluvia;
* viento;
* dirección del viento;
* radiación;
* calidad del aire;
* CO₂;
* partículas;
* suelo;
* pH;
* conductividad;
* etc.

Ejemplos:

```text
AHT10
AHT20
AHT30
DHT22
DS18B20
BMP280
BME280
BME680
SHT3x
```

El sistema debe permitir seleccionar el driver desde configuración cuando sea posible.

---

# 23. Medición eléctrica

Debe existir un módulo específico:

```text
Energy
```

con capacidades:

```text
voltage
current
power
energy
frequency
power_factor
```

Ejemplos de sensores:

```text
ZMPT101B
SCT-013
```

y sensores/medidores digitales futuros.

El sistema debe permitir:

```text
Voltage = 230 V
Current = 4.2 A
Power = 966 W
Energy = 3.4 kWh
```

---

# 24. Seguridad eléctrica

Las mediciones de red deben considerarse sistemas de riesgo.

El firmware nunca debe asumir que:

```text
230 V = seguro
```

Debe existir documentación específica de:

* aislamiento;
* fusibles;
* creepage;
* clearance;
* transformadores;
* CT;
* optoacopladores;
* relés;
* SSR;
* puesta a tierra;
* protección contra sobretensión.

El firmware no sustituye protecciones eléctricas físicas.

---

# 25. Agua

Debe existir una categoría:

```text
Water
```

con:

```text
flow
volume
pressure
temperature
level
leak
```

Se podrán utilizar:

```text
YF series
DN series
Hall-effect flow sensors
```

Ejemplo:

```json
{
  "flow": 12.4,
  "unit": "L/min",
  "total_volume": 843.2,
  "unit_total": "L"
}
```

El driver debe permitir configurar:

```text
pulses_per_liter
```

---

# 26. Modbus

Debe existir un módulo independiente:

```text
Modbus
```

con soporte para:

```text
Modbus RTU
Modbus TCP
```

y posteriormente:

```text
Modbus Security
```

cuando corresponda.

La arquitectura debe soportar:

```text
Master / Client
Slave / Server
Gateway
```

Ejemplo:

```text
ESP32
  │
  ├── Modbus RTU
  │
  ├── RS485
  │
  └── Sensor industrial
```

y:

```text
ESP32
  │
  └── Ethernet
       │
       └── Modbus TCP
             │
             └── Inverter
```

---

# 27. Configuración Modbus

Desde la web:

```text
Interface:
RS485-1

Baud:
9600

Parity:
None

Stop bits:
1

Device:
Address 12

Register:
40001

Data type:
Float32

Byte order:
ABCD

Scale:
0.1
```

Debe ser posible definir perfiles reutilizables.

---

# 28. CAN

Debe existir una abstracción:

```text
CAN Bus
```

independiente del protocolo de aplicación.

Debe soportar:

* CAN clásico;
* CAN-FD cuando el hardware lo permita;
* filtros;
* bitrate;
* frames estándar;
* frames extendidos;
* diagnóstico.

---

# 29. CANopen

Debe existir un módulo:

```text
CANopen
```

para dispositivos compatibles.

Debe contemplar:

```text
NMT
PDO
SDO
EMCY
Heartbeat
Node Guarding
Object Dictionary
LSS
```

Ejemplo:

```text
CAN Bus
 ├── Node 1
 │    └── Motor
 ├── Node 2
 │    └── Sensor
 └── Node 3
      └── Actuator
```

Esto será especialmente importante para integración de equipos industriales y aplicaciones marinas.

---

# 30. Comunicación interna

La comunicación entre nodos será dividida en capas.

## Nivel 1 — Transporte

```text
Ethernet
Wi-Fi
CAN
RS485
Thread
Zigbee
BLE
```

## Nivel 2 — Protocolos

```text
TCP/IP
UDP
MQTT
HTTP
WebSocket
CANopen
Modbus
Matter
Zigbee
```

## Nivel 3 — Modelo de datos

Todos los protocolos deben mapear hacia el mismo modelo:

```text
Device
Entity
Capability
Property
State
Command
Event
```

Esto es fundamental.

---

# 31. MQTT como bus lógico principal

MQTT será el protocolo recomendado para la comunicación distribuida sobre IP.

Arquitectura:

```text
Node A
   │
Node B ─── MQTT Broker ─── Node C
   │
Node D
```

Ventajas:

* publicación/suscripción;
* desacoplamiento;
* múltiples consumidores;
* eventos;
* escalabilidad;
* integración sencilla;
* funcionamiento local;
* integración con servidores externos.

---

# 32. MQTT Topics

Se recomienda una estructura:

```text
automation/<site>/<device>/<entity>/<property>
```

Ejemplo:

```text
automation/home/living/light01/state/power
```

Eventos:

```text
automation/home/living/light01/event
```

Comandos:

```text
automation/home/living/light01/command
```

Discovery:

```text
automation/home/living/light01/config
```

Estado:

```text
automation/home/living/light01/state
```

---

# 33. MQTT Discovery

Los dispositivos deben poder anunciarse automáticamente.

Ejemplo:

```json
{
  "device_id": "living-light-01",
  "name": "Luz Living",
  "room": "living",
  "capabilities": [
    "switch",
    "brightness"
  ],
  "firmware": "1.0.0"
}
```

Esto permitirá que un servidor central descubra nodos sin configuración manual.

---

# 34. Comunicación local sin servidor

El sistema no debe depender obligatoriamente de un servidor central.

Debe ser posible:

```text
ESP32
 ├── Web UI
 ├── Automation Engine
 ├── Sensors
 └── Outputs
```

y funcionar de forma autónoma.

---

# 35. Arquitectura con servidor central

En instalaciones grandes:

```text
                 ┌─────────────────┐
                 │ Central Server  │
                 │                 │
                 │ Database        │
                 │ API             │
                 │ Dashboard       │
                 │ Automation      │
                 └────────┬────────┘
                          │
                       MQTT
                          │
       ┌──────────────────┼──────────────────┐
       │                  │                  │
      ESP32              ESP32              ESP32
```

El servidor puede encargarse de:

* histórico;
* usuarios;
* dashboards;
* análisis;
* backups;
* actualizaciones;
* automatizaciones complejas.

Los nodos siguen pudiendo ejecutar automatizaciones críticas localmente.

---

# 36. Arquitectura híbrida

Será el modo recomendado.

```text
                 CENTRAL
                    │
              MQTT / API
                    │
        ┌───────────┼───────────┐
        │           │           │
      NODE        NODE        NODE
        │           │           │
      local       local       local
    automation  automation  automation
```

Si el servidor desaparece:

```text
Internet OFF
        ↓
Automatización local continúa
```

---

# 37. Web Interface

Cada dispositivo debe disponer, cuando tenga recursos suficientes, de una interfaz web local.

Funciones:

```text
Dashboard
Devices
Sensors
Outputs
Automation
Routines
Scenes
Users
Network
Communication
Hardware
Modules
Diagnostics
Logs
OTA
Backup
Restore
```

---

# 38. Usuarios

Debe existir autenticación.

Roles mínimos:

```text
Administrator
Installer
Operator
Viewer
```

Ejemplo:

```text
Administrator
 ├── configuración
 ├── usuarios
 ├── hardware
 ├── firmware
 └── automatizaciones

Operator
 ├── controlar
 └── visualizar

Viewer
 └── visualizar
```

---

# 39. Seguridad

Debe contemplarse:

* contraseñas con hash;
* sesiones;
* expiración;
* permisos;
* HTTPS cuando sea viable;
* tokens;
* API keys;
* MQTT authentication;
* TLS;
* certificados;
* secure boot;
* flash encryption;
* actualización firmada.

Nunca guardar contraseñas en texto plano.

---

# 40. API REST

La plataforma debe disponer de API.

Ejemplo:

```text
GET /api/v1/devices
GET /api/v1/devices/{id}
GET /api/v1/devices/{id}/state
POST /api/v1/devices/{id}/command
GET /api/v1/sensors
GET /api/v1/rooms
GET /api/v1/automations
POST /api/v1/automations
```

---

# 41. API WebSocket

Para datos en tiempo real:

```text
WebSocket
```

permitirá actualizar:

* temperaturas;
* estados;
* alarmas;
* consumo;
* sensores;
* dispositivos;
* música;
* pantallas.

Sin necesidad de realizar polling constante.

---

# 42. API externa

La API deberá diseñarse para que otros sistemas puedan consultar:

```text
temperatura
humedad
presión
estado de luces
consumo
agua
alarmas
puertas
ventanas
etc.
```

---

# 43. Integración Home Assistant

Debe existir una integración con:

```text
Home Assistant
```

mediante:

* MQTT;
* REST;
* WebSocket;
* discovery cuando sea posible.

El sistema no debe depender de Home Assistant.

Home Assistant será un consumidor externo.

---

# 44. Integración Homey Pro

Debe existir una estrategia de integración mediante:

```text
MQTT
REST API
Webhooks
```

cuando resulte compatible con las capacidades disponibles.

---

# 45. Matter

Matter será una de las principales capas de interoperabilidad.

El sistema debe permitir que determinadas capacidades puedan exponerse como dispositivos Matter.

Ejemplo:

```text
Sistema
   │
   └── Matter
        ├── Light
        ├── Switch
        ├── Fan
        ├── Thermostat
        ├── Sensor
        └── Cover
```

---

# 46. Thread

Thread podrá utilizarse como transporte para dispositivos compatibles con Matter.

Especialmente:

```text
ESP32-C6
ESP32-H2
```

La plataforma deberá tratar Thread como una capa de conectividad, no como el modelo principal de dispositivos.

---

# 47. Zigbee

Debe existir soporte para:

```text
Zigbee Device
Zigbee Coordinator
Zigbee Gateway
```

Un ESP32-C6 podrá utilizarse como parte de un gateway Zigbee cuando el diseño de hardware/software lo permita.

Ejemplo:

```text
Zigbee Sensors
      │
      ▼
ESP32-C6
      │
      ▼
Platform Device Model
      │
      ▼
MQTT / API / Automation
```

---

# 48. Matter Bridge

Debe contemplarse:

```text
Zigbee
   ↓
Gateway
   ↓
Device Model
   ↓
Matter
```

Esto permite que dispositivos que no son Matter puedan aparecer en ecosistemas compatibles.

---

# 49. Apple Home

La plataforma deberá contemplar integración con:

```text
Apple Home
```

preferentemente mediante Matter cuando el dispositivo/capacidad sea compatible.

---

# 50. Google Home

Debe contemplarse:

```text
Google Home
```

mediante Matter y mecanismos de integración compatibles.

---

# 51. Samsung SmartThings

Debe contemplarse:

```text
Samsung SmartThings
```

mediante:

* Matter;
* APIs;
* bridges;
* integraciones específicas cuando sean necesarias.

---

# 52. Pantallas táctiles

Las pantallas deben ser consideradas dispositivos del sistema.

Ejemplo:

```text
ESP32-S3
   │
   ├── Display
   ├── Touch
   ├── Wi-Fi
   └── Device Client
```

---

# 53. Controladores de pantalla

La arquitectura debe ser extensible para:

```text
ST7789
ST7789V
ILI9341
ILI9488
ST7796S
ST7796UI
GC9A01
SPD2010
CO5300
ST7262
EK9716
```

y otros controladores futuros.

---

# 54. Touch

Debe contemplarse:

```text
CST816S
CST816D
CST820
FT6336
FT6236
FT5x06
GT911
AXS15231
CHSC6x
XPT2046
```

El sistema debe separar:

```text
Touch Driver
```

de:

```text
UI Framework
```

---

# 55. Arquitectura de pantalla

Una pantalla no debe contener una interfaz fija en firmware.

Debe existir:

```text
Screen
 ├── Layout
 ├── Blocks
 ├── Widgets
 └── Actions
```

Ejemplo:

```text
Living Room
├── Temperature
├── Humidity
├── Lights
├── Curtains
├── Energy
└── Music
```

---

# 56. Bloques de interfaz

Cada bloque debe ser configurable.

Ejemplo:

```json
{
  "id": "temperature-card",
  "type": "sensor",
  "entity": "living.temperature",
  "position": 1,
  "enabled": true
}
```

---

# 57. Configuración desde la pantalla

Cuando el hardware lo permita, el usuario podrá:

```text
Agregar bloque
Eliminar bloque
Mover bloque
Cambiar tamaño
Cambiar entidad
Cambiar icono
Cambiar nombre
Configurar acción
```

---

# 58. Configuración desde Web

La misma configuración debe poder realizarse desde la interfaz web.

Esto permitirá:

```text
Web UI
   ↓
Screen Configuration
   ↓
ESP32-S3
   ↓
Touchscreen
```

---

# 59. Sincronización de interfaces

Una configuración modificada desde:

```text
Web
```

debe poder reflejarse en:

```text
Touchscreen
```

y viceversa.

---

# 60. Arquitectura de widgets

Ejemplos:

```text
TemperatureCard
HumidityCard
PressureCard
LightControl
DimmerControl
FanControl
CoverControl
EnergyCard
WaterFlowCard
WeatherCard
AlarmCard
CameraCard
MediaPlayer
SceneButton
AutomationButton
```

---

# 61. Rutinas

Una rutina representa una secuencia de acciones.

Ejemplo:

```text
"Buenas noches"
```

Acciones:

```text
Apagar luces
Cerrar cortinas
Apagar TV
Reducir climatización
Activar alarma
```

---

# 62. Escenas

Una escena define un estado deseado.

Ejemplo:

```text
Cine
```

```text
Living Light = 20%
Curtain = 100% closed
TV = ON
Ambient Light = ON
```

---

# 63. Automatizaciones

Una automatización tendrá:

```text
TRIGGER
    ↓
CONDITIONS
    ↓
ACTIONS
```

Ejemplo:

```text
Trigger:
Motion detected

Condition:
Time between 20:00 and 06:00

Action:
Turn light ON
```

---

# 64. Triggers

Tipos:

```text
State changed
Value changed
Threshold
Schedule
Timer
Button
Motion
Presence
Event
Webhook
MQTT
CAN
Modbus
Location
Sunrise
Sunset
```

---

# 65. Condiciones

Ejemplos:

```text
temperature > 28
humidity < 40
power > 2000W
door == closed
alarm == armed
time > 22:00
```

También deberán soportarse operadores lógicos:

```text
AND
OR
NOT
```

---

# 66. Acciones

Ejemplos:

```text
Turn ON
Turn OFF
Set brightness
Set speed
Move cover
Run scene
Run routine
Delay
Publish MQTT
Call API
Send notification
Activate alarm
Set variable
```

---

# 67. Motor de automatización

Debe existir un:

```text
Automation Engine
```

independiente de la interfaz.

La interfaz solamente crea/modifica reglas.

El motor ejecuta:

```text
Event
 ↓
Trigger matching
 ↓
Conditions
 ↓
Action queue
 ↓
Execution
 ↓
Result
```

---

# 68. Eventos

Todo cambio importante debe generar eventos.

Ejemplo:

```json
{
  "event": "state_changed",
  "device": "living-light-01",
  "entity": "power",
  "old": false,
  "new": true,
  "timestamp": 1791123456
}
```

---

# 69. Variables

Debe existir un sistema de variables.

Ejemplo:

```text
house.mode = "night"
living.temperature = 24.2
energy.today = 12.4
```

Las variables pueden utilizarse en automatizaciones.

---

# 70. Temporizadores

Debe existir soporte para:

```text
Delay
Timer
Countdown
Periodic timer
Schedule
Cron-like schedule
```

---

# 71. Calendario

El sistema debe poder ejecutar automatizaciones por:

* hora;
* día;
* día de semana;
* fecha;
* rango horario;
* amanecer;
* atardecer.

---

# 72. NTP

El sistema debe sincronizar la hora mediante:

```text
NTP
```

y soportar:

```text
Timezone
DST
UTC
Local time
```

La hora interna debe mantenerse preferentemente en UTC.

---

# 73. RTC

Para dispositivos donde sea necesario mantener hora sin conexión se podrá utilizar:

```text
RTC interno
RTC externo
```

Ejemplo:

```text
DS3231
```

---

# 74. Watchdog

Todo nodo deberá disponer de mecanismos de recuperación.

Debe existir:

```text
Hardware Watchdog
Task Watchdog
Communication Watchdog
Application Health Monitor
```

---

# 75. Health Monitoring

Cada dispositivo debe informar:

```text
uptime
free heap
CPU usage
temperature
Wi-Fi RSSI
network state
MQTT state
sensor errors
task health
watchdog state
firmware version
configuration version
```

---

# 76. Estado del dispositivo

Ejemplo:

```json
{
  "online": true,
  "uptime": 84322,
  "free_heap": 183420,
  "wifi_rssi": -54,
  "mqtt": true,
  "errors": 0
}
```

---

# 77. Diagnóstico

Debe existir una sección:

```text
Diagnostics
```

con:

```text
Logs
Errors
Warnings
Communication
Memory
CPU
Tasks
GPIO
Sensors
Network
Storage
```

---

# 78. Logging

Debe existir un logger común.

Niveles:

```text
TRACE
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

El nivel podrá configurarse por módulo.

---

# 79. Persistencia

El sistema debe separar:

```text
Configuration
State
History
Logs
Cache
```

No todo debe almacenarse en el mismo lugar.

---

# 80. Configuración

La configuración debe estar versionada.

Ejemplo:

```json
{
  "config_version": 3,
  "device": {},
  "network": {},
  "hardware": {},
  "modules": {},
  "automation": {}
}
```

---

# 81. Migraciones

Cuando cambie la estructura:

```text
Config v1
    ↓
Migration
    ↓
Config v2
```

Nunca se debe romper una instalación simplemente por actualizar firmware.

---

# 82. Backup

La interfaz debe permitir:

```text
Export configuration
Import configuration
Reset configuration
Factory reset
```

---

# 83. OTA

Debe existir actualización:

```text
OTA local
OTA por servidor
OTA desde repositorio
```

Debe verificarse:

```text
version
compatibility
checksum
signature
hardware
```

---

# 84. Versionado

Se utilizará:

```text
Semantic Versioning
```

Ejemplo:

```text
1.4.2
```

donde:

```text
MAJOR.MINOR.PATCH
```

---

# 85. Compatibilidad

Cada firmware debe indicar:

```json
{
  "firmware": "2.0.0",
  "api": "3",
  "config": "5",
  "hardware": [
    "esp32",
    "esp32-s3"
  ]
}
```

---

# 86. Module Registry

Debe existir un registro central de módulos.

```text
ModuleRegistry
├── Core
├── Network
├── MQTT
├── Web
├── API
├── Sensors
├── Outputs
├── Automation
├── Energy
├── Water
├── Modbus
├── CAN
├── CANopen
├── Matter
├── Zigbee
├── Thread
├── Display
├── Touch
├── Media
├── Security
└── Diagnostics
```

---

# 87. Manifest de módulo

Cada módulo deberá declarar:

```json
{
  "id": "modbus",
  "name": "Modbus",
  "version": "1.0.0",
  "api_version": 1,
  "enabled": true,
  "dependencies": [
    "core",
    "serial"
  ],
  "capabilities": [
    "modbus_rtu",
    "modbus_tcp"
  ]
}
```

---

# 88. Módulos opcionales

Un módulo no debe romper el sistema si no está instalado.

Ejemplo:

```text
CORE
 ├── Web
 ├── Devices
 └── Automation

Opcional
 ├── Modbus
 ├── CANopen
 ├── Matter
 ├── Zigbee
 ├── Display
 └── AI
```

---

# 89. Hardware Profiles

Debe existir un sistema de perfiles.

Ejemplo:

```text
ESP32 Basic
ESP32 Ethernet
ESP32-S3 Display
ESP32-C6 Wireless
ESP32 CAN
ESP32 Modbus
```

El usuario puede seleccionar el perfil desde la configuración.

---

# 90. Pin Manager

Uno de los componentes más importantes será:

```text
Pin Manager
```

Debe conocer:

```text
GPIO
ADC
PWM
I2C
SPI
UART
CAN
Interrupt
Touch
```

y evitar conflictos.

---

# 91. Detección de conflictos

Ejemplo:

```text
GPIO 21
 ├── I2C SDA
 └── Relay
```

Debe marcar:

```text
CONFLICT
```

antes de aplicar la configuración.

---

# 92. Hardware autodiscovery

Cuando sea posible, el sistema debe detectar:

```text
I2C devices
SPI devices
Modbus devices
CAN nodes
USB devices
```

y ofrecerlos al usuario.

---

# 93. I2C

Debe soportarse:

```text
multiple buses
configurable pins
addresses
scan
device detection
```

---

# 94. SPI

Debe soportarse:

```text
SPI bus
CS
DC
RST
MOSI
MISO
SCLK
```

con múltiples dispositivos.

---

# 95. UART

Debe soportarse:

```text
UART0
UART1
UART2
```

cuando el SoC lo permita.

Configuración:

```text
baud
parity
stop bits
data bits
flow control
```

---

# 96. Ethernet

Los dispositivos compatibles deberán soportar Ethernet.

Ejemplos de controladores:

```text
LAN8720
IP101
```

El Ethernet deberá integrarse en la misma capa de red que Wi-Fi.

---

# 97. Red

Debe soportar:

```text
DHCP
Static IP
DNS
mDNS
IPv4
IPv6 cuando sea viable
NTP
```

---

# 98. Wi-Fi

Debe soportar:

```text
Station
Access Point
Provisioning
Network scan
```

La configuración debe poder realizarse desde una interfaz.

---

# 99. Provisioning

Primer arranque:

```text
ESP32
 ↓
Access Point
 ↓
Configuración
 ↓
Seleccionar Wi-Fi
 ↓
Credenciales
 ↓
Aplicar
 ↓
Reinicio
```

---

# 100. Central Server Discovery

Los dispositivos deberán poder encontrar automáticamente el servidor mediante:

```text
mDNS
DNS-SD
MQTT discovery
configuración manual
```

---

# 101. Multi-site

La plataforma debe permitir múltiples instalaciones.

Ejemplo:

```text
Usuario
├── Casa
├── Oficina
├── Taller
└── Campo
```

Cada instalación posee sus propios:

```text
devices
rooms
automations
users
network
```

---

# 102. Multi-building

Dentro de una instalación:

```text
Site
├── Building A
├── Building B
└── Building C
```

---

# 103. Escalabilidad

La arquitectura debe funcionar en:

```text
1 nodo
10 nodos
50 nodos
100 nodos
500 nodos
```

sin modificar el modelo conceptual.

El límite práctico dependerá del broker, red, hardware y servidor.

---

# 104. Edge Computing

Las decisiones críticas deben poder ejecutarse en el nodo.

Ejemplo:

```text
Temperature > 30
    ↓
Fan ON
```

No debe requerir Internet.

---

# 105. Cloud opcional

La nube será opcional.

Arquitectura:

```text
Local
   │
   ├── Internet
   │
   └── Cloud
```

Nunca:

```text
Cloud required
```

para funciones básicas.

---

# 106. Inteligencia artificial

Los dispositivos más potentes, especialmente ESP32-S3, podrán utilizar algoritmos locales.

Ejemplos:

```text
Predicción de temperatura
Predicción de consumo
Detección de presencia
Reconocimiento de patrones
Predicción de ocupación
Aprendizaje de horarios
Detección de anomalías
```

---

# 107. Aprendizaje de hábitos

Ejemplo:

```text
19:30
Usuario enciende luz

19:32
Enciende TV

19:40
Cierra cortina
```

Después de suficiente información:

```text
19:30
→ sugerir rutina
```

El sistema no debe ejecutar automáticamente cambios importantes sin consentimiento del usuario.

---

# 108. Anomaly Detection

Ejemplos:

```text
Consumo normalmente:
500 W

Actual:
2800 W
```

Generar:

```text
Warning:
Unusual energy consumption
```

Otro ejemplo:

```text
Caudal esperado:
0 L/min

Caudal detectado:
7 L/min
```

Posible:

```text
Water leak
```

---

# 109. Media

Las pantallas podrán mostrar:

```text
Music
Album
Artist
Playback
Volume
```

si existe un proveedor o servidor compatible.

La plataforma solamente debe manejar la abstracción:

```text
MediaPlayer
```

---

# 110. Cámara

Los dispositivos con cámara podrán proporcionar:

```text
snapshot
stream
motion detection
object detection
```

sin que la plataforma dependa de una única implementación de IA.

---

# 111. Alarmas

Debe existir un módulo:

```text
Security
```

con:

```text
Alarm
Zone
Sensor
Event
Arming
Disarming
Delay
Siren
Notification
```

---

# 112. Zonas de seguridad

Ejemplo:

```text
Zone 1
Door

Zone 2
Window

Zone 3
Motion

Zone 4
Garage
```

---

# 113. Modos de alarma

```text
Disarmed
Armed Home
Armed Away
Night
Alarm
```

---

# 114. Notificaciones

Debe existir una abstracción:

```text
Notification Service
```

con proveedores intercambiables.

Ejemplos:

```text
Push
Email
Telegram
Webhook
MQTT
```

---

# 115. Sistema de permisos

Los permisos deben ser granulares.

Ejemplo:

```text
devices.read
devices.control
devices.configure

automation.read
automation.write

users.read
users.write

hardware.configure

firmware.update
```

---

# 116. API Tokens

Las integraciones externas podrán utilizar:

```text
API Token
```

con permisos limitados.

Ejemplo:

```text
Token Home Assistant

permissions:
    devices.read
    sensors.read
    devices.control
```

---

# 117. Auditoría

Toda acción administrativa importante debe poder registrarse:

```text
User
Action
Device
Timestamp
Result
```

Ejemplo:

```text
Administrator
changed relay configuration
2026-10-05 12:40
SUCCESS
```

---

# 118. Fail-safe

Cada actuador debe definir:

```text
startup state
communication loss state
watchdog state
emergency state
```

Ejemplo:

```text
Heater:
communication lost → OFF

Pump:
sensor failure → OFF

Ventilation:
temperature emergency → ON
```

---

# 119. Seguridad funcional

Este proyecto no debe considerarse inicialmente:

```text
Safety PLC
```

ni utilizarse para funciones donde sea necesaria certificación funcional sin incorporar hardware y procesos específicamente certificados.

Ejemplos de sistemas que requieren especial consideración:

* maquinaria peligrosa;
* paradas de emergencia;
* sistemas contra incendio;
* protección humana;
* instalaciones críticas;
* enclavamientos de seguridad.

Las protecciones críticas deberán existir físicamente y no depender exclusivamente del firmware.

---

# 120. Estructura del firmware

Propuesta:

```text
src/
├── core/
├── hal/
├── drivers/
├── devices/
├── capabilities/
├── services/
├── communication/
├── automation/
├── security/
├── storage/
├── web/
├── api/
├── ui/
├── modules/
├── diagnostics/
└── main.cpp
```

---

# 121. Core

Contendrá:

```text
System
EventBus
ModuleRegistry
DeviceRegistry
ConfigurationManager
StateManager
Logger
Scheduler
```

---

# 122. HAL

Contendrá:

```text
GPIO
ADC
PWM
I2C
SPI
UART
CAN
Ethernet
WiFi
Storage
Display
```

---

# 123. Drivers

Ejemplo:

```text
drivers/
├── sensors/
│   ├── aht20/
│   ├── bme280/
│   └── ds18b20/
├── displays/
├── touch/
├── energy/
├── flow/
└── actuators/
```

---

# 124. Services

Ejemplo:

```text
services/
├── lighting/
├── climate/
├── energy/
├── water/
├── weather/
├── security/
├── media/
└── access/
```

---

# 125. Communication

```text
communication/
├── mqtt/
├── http/
├── websocket/
├── modbus/
├── can/
├── canopen/
├── matter/
├── zigbee/
├── thread/
└── ble/
```

---

# 126. Modules

Cada módulo debe ser independiente.

Ejemplo:

```text
modules/
├── energy/
├── water/
├── weather/
├── automation/
├── display/
├── zigbee/
└── matter/
```

---

# 127. Module Interface

Conceptualmente:

```cpp
class IModule {
public:
    virtual bool begin() = 0;
    virtual void loop() = 0;
    virtual bool enable() = 0;
    virtual bool disable() = 0;
    virtual const ModuleManifest& manifest() = 0;
};
```

---

# 128. Device Interface

Conceptualmente:

```cpp
class IDevice {
public:
    virtual const char* id() = 0;
    virtual const DeviceInfo& info() = 0;
    virtual bool begin() = 0;
    virtual bool setState(...) = 0;
    virtual DeviceState getState() = 0;
};
```

---

# 129. Sensor Interface

```cpp
class ISensor {
public:
    virtual bool begin() = 0;
    virtual Measurement read() = 0;
    virtual SensorStatus status() = 0;
};
```

---

# 130. Actuator Interface

```cpp
class IActuator {
public:
    virtual bool begin() = 0;
    virtual bool setValue(...) = 0;
    virtual ActuatorState state() = 0;
};
```

---

# 131. Event Bus

El sistema debe disponer de un bus interno:

```text
EventBus
```

Ejemplo:

```text
Sensor
 ↓
EventBus
 ↓
Automation Engine
 ↓
Action
 ↓
Device
```

Esto evita dependencias directas.

---

# 132. Ejemplo de automatización completa

```text
Temperature Sensor
        ↓
temperature_changed
        ↓
EventBus
        ↓
Automation Engine
        ↓
Condition:
temperature > 28
        ↓
Action:
Fan ON
        ↓
Device
        ↓
Relay
```

---

# 133. Ejemplo de botón

```text
Touch Button
      ↓
Button Event
      ↓
Automation
      ↓
Scene
      ↓
Actions
```

Un único botón podría:

```text
Light ON
Curtain CLOSE
Fan ON
TV ON
```

---

# 134. Escenas reutilizables

Las escenas deben poder ser invocadas desde:

```text
Button
Web
Touchscreen
API
MQTT
Automation
Schedule
Voice Assistant
```

---

# 135. Plantillas de automatización

Debe existir la posibilidad de crear templates.

Ejemplo:

```text
Automatic Light

Trigger:
Motion

Condition:
Dark

Action:
Light ON

Timeout:
120 seconds
```

---

# 136. Importación/exportación

Las automatizaciones deben poder exportarse como JSON.

Ejemplo:

```json
{
  "name": "Night Light",
  "trigger": {
    "type": "motion"
  },
  "conditions": [
    {
      "type": "time_range",
      "from": "22:00",
      "to": "06:00"
    }
  ],
  "actions": [
    {
      "type": "turn_on",
      "device": "hall.light"
    }
  ]
}
```

---

# 137. Configuración completamente visual

Siempre que sea posible:

```text
Usuario
 ↓
Web
 ↓
Seleccionar dispositivo
 ↓
Seleccionar función
 ↓
Seleccionar acción
 ↓
Guardar
```

No:

```text
editar código
compilar
flashear
```

para tareas normales.

---

# 138. Instalación de módulos

El sistema deberá permitir incorporar módulos mediante:

```text
Firmware build
```

inicialmente.

Posteriormente podrá estudiarse:

```text
Module packages
```

si la memoria y arquitectura lo permiten.

La instalación dinámica de código no será un requisito inicial debido a limitaciones de memoria, seguridad y complejidad en sistemas embebidos.

---

# 139. Diseño modular de UI

Cada página estará formada por:

```text
Page
├── Header
├── Navigation
├── Sections
│   ├── Cards
│   ├── Widgets
│   ├── Tables
│   └── Charts
└── Actions
```

---

# 140. Design System

La interfaz debe seguir la filosofía:

```text
Simple por fuera.
Modular por dentro.
```

Debe priorizar:

* claridad;
* precisión;
* jerarquía;
* accesibilidad;
* consistencia;
* aspecto técnico/profesional.

Debe evitar:

* animaciones innecesarias;
* interfaces saturadas;
* decoración excesiva;
* información irrelevante.

---

# 141. Dashboard

Dashboard inicial:

```text
┌────────────────────────────────────────────┐
│ Sistema                         ● ONLINE    │
├────────────────────────────────────────────┤
│ Temperatura │ Humedad │ Energía │ Alarmas │
├────────────────────────────────────────────┤
│                                            │
│                 HABITACIONES                │
│                                            │
├────────────────────────────────────────────┤
│ Automatizaciones                           │
├────────────────────────────────────────────┤
│ Eventos                                    │
└────────────────────────────────────────────┘
```

---

# 142. Dashboards personalizados

El usuario podrá crear dashboards:

```text
Casa
Oficina
Energía
Clima
Seguridad
Agua
Producción
```

---

# 143. Históricos

El sistema deberá poder almacenar:

```text
Temperature
Humidity
Pressure
Power
Energy
Water
Events
```

El almacenamiento dependerá del dispositivo.

Para grandes cantidades:

```text
Servidor
Database
```

será la opción recomendada.

---

# 144. Retención

Debe configurarse:

```text
1 hora
1 día
7 días
30 días
1 año
```

según capacidad.

---

# 145. Métricas

Los sensores deben permitir:

```text
raw value
filtered value
calibrated value
timestamp
quality
```

Ejemplo:

```json
{
  "raw": 25.82,
  "value": 25.6,
  "unit": "°C",
  "quality": "good"
}
```

---

# 146. Calibración

Cada sensor puede disponer de:

```text
offset
gain
linear calibration
multi-point calibration
```

---

# 147. Filtrado

Opciones:

```text
Moving average
Median
EMA
Low-pass
Outlier rejection
```

El filtro deberá ser configurable.

---

# 148. Unidades

El sistema deberá utilizar un modelo interno normalizado.

Ejemplo:

```text
temperature → °C
pressure → Pa
energy → Wh
power → W
flow → L/min
```

La interfaz podrá convertir a:

```text
°F
bar
kWh
GPM
```

etc.

---

# 149. Internacionalización

Debe contemplarse:

```text
Español
English
Italiano
```

y otros idiomas posteriormente.

---

# 150. Configuración regional

Debe soportar:

```text
timezone
locale
units
decimal separator
date format
time format
```

---

# 151. Arquitectura de red recomendada

Para una instalación pequeña:

```text
Wi-Fi
```

Para instalaciones más grandes:

```text
Ethernet + Wi-Fi
```

Para campo:

```text
CAN / RS485
```

Para dispositivos domésticos inalámbricos:

```text
Thread / Zigbee / Matter
```

---

# 152. Arquitectura completa

```text
                         INTERNET
                             │
                    ┌────────▼────────┐
                    │ External Cloud  │
                    └────────┬────────┘
                             │
                      ┌──────▼──────┐
                      │ Central     │
                      │ Server      │
                      │ MQTT/API    │
                      └──────┬──────┘
                             │
               ┌─────────────┼─────────────┐
               │             │             │
            Ethernet        Wi-Fi        Thread
               │             │             │
       ┌───────┼──────┐      │       ┌─────┴─────┐
       │       │      │      │       │           │
     ESP32   ESP32   Gateway C6    Matter     Sensors
       │       │
     CAN     RS485
       │       │
   CANopen   Modbus
       │       │
 Industrial Devices
```

---

# 153. Gateways

Un dispositivo podrá actuar como gateway.

Ejemplo:

```text
ESP32-C6 Gateway

Zigbee
   ↓
Device Model
   ↓
MQTT
   ↓
Network
```

Otro:

```text
ESP32 Ethernet

CANopen
   ↓
Device Model
   ↓
Modbus TCP
```

---

# 154. Bridges

Debe existir una abstracción:

```text
Bridge
```

para transformar:

```text
Protocol A
     ↓
Device Model
     ↓
Protocol B
```

Ejemplos:

```text
Zigbee → MQTT
CANopen → MQTT
Modbus → MQTT
MQTT → Matter
```

---

# 155. Independencia de protocolo

Una automatización nunca debe depender directamente de:

```text
MQTT topic
CAN ID
Modbus register
GPIO
```

Debe depender de:

```text
device
entity
capability
state
```

Esto es uno de los principios arquitectónicos más importantes.

---

# 156. Ejemplo

Incorrecto:

```text
IF GPIO23 == HIGH
THEN GPIO18 = LOW
```

Correcto:

```text
IF living.motion == detected
THEN living.light = ON
```

El sistema decide cómo implementar esa acción.

---

# 157. Redundancia

Para sistemas importantes podrá existir:

```text
Primary Controller
Secondary Controller
```

y mecanismos de recuperación.

No será requisito inicial, pero la arquitectura no debe impedirlo.

---

# 158. Offline-first

Las funciones fundamentales deberán funcionar sin:

```text
Internet
Cloud
Mobile App
Central Server
```

si el nodo posee los recursos necesarios.

---

# 159. Estado deseado vs estado real

Todo actuador debe poder distinguir:

```text
Desired State
Actual State
```

Ejemplo:

```text
Desired:
ON

Actual:
OFF
```

Esto permite detectar:

* fallo;
* comunicación perdida;
* relé defectuoso;
* protección activada.

---

# 160. Acknowledgement

Los comandos importantes podrán requerir:

```text
Command
 ↓
Device
 ↓
ACK
 ↓
State confirmation
```

---

# 161. Command ID

Los comandos deberán tener identificadores.

```json
{
  "command_id": "a83f22",
  "device": "pump01",
  "action": "start"
}
```

Esto permite detectar duplicados.

---

# 162. Timeouts

Cada comunicación deberá tener:

```text
timeout
retry
backoff
max retries
```

cuando corresponda.

---

# 163. Versionado de API

Todas las APIs deben versionarse.

```text
/api/v1/
```

Una nueva versión no debe romper inmediatamente clientes anteriores.

---

# 164. Compatibilidad hacia atrás

El sistema debe intentar mantener:

```text
API
Config
MQTT schema
Device model
```

compatibles entre versiones.

---

# 165. Esquema de datos

Se recomienda utilizar JSON para:

```text
configuration
API
MQTT
discovery
automation
events
```

y formatos binarios solamente cuando sean necesarios por rendimiento.

---

# 166. JSON Schema

Los mensajes importantes deberán disponer de schemas.

Ejemplo:

```text
schemas/
├── device.json
├── state.json
├── command.json
├── event.json
├── automation.json
└── module.json
```

---

# 167. Documentación

El proyecto deberá documentar:

```text
Architecture
Hardware
Firmware
Communication
API
UI
Modules
Sensors
Drivers
Automation
Security
Deployment
Development
Troubleshooting
```

---

# 168. Estructura de documentación

```text
docs/
├── architecture/
├── hardware/
├── firmware/
├── communication/
├── api/
├── ui/
├── modules/
├── sensors/
├── actuators/
├── automation/
├── security/
├── deployment/
├── development/
├── troubleshooting/
└── decisions/
```

---

# 169. Dudas y decisiones

Debe existir:

```text
docs/DUDAS-Y-DECISIONES.md
```

Cada decisión importante deberá registrarse.

Formato:

```markdown
## DEC-001 — MQTT como bus lógico

Estado: Aceptado

Problema:
...

Decisión:
...

Motivo:
...

Alternativas:
...

Consecuencias:
...
```

---

# 170. ADR

Se recomienda utilizar Architecture Decision Records.

Ejemplos:

```text
ADR-001 Device Model
ADR-002 MQTT
ADR-003 Modbus
ADR-004 CANopen
ADR-005 Matter
ADR-006 Storage
ADR-007 Authentication
ADR-008 UI Architecture
```

---

# 171. Nuevo módulo

Para crear un nuevo módulo:

```text
1. Definir objetivo
2. Definir capabilities
3. Definir interfaces
4. Definir dependencias
5. Crear manifest
6. Implementar driver
7. Implementar servicio
8. Implementar API
9. Implementar UI
10. Agregar documentación
11. Agregar tests
12. Agregar ejemplo
```

---

# 172. Ejemplo de nuevo sensor

Supongamos:

```text
Sensor XYZ
```

Debe implementarse:

```text
XYZDriver
    ↓
Sensor Entity
    ↓
temperature
humidity
```

No modificar:

```text
Automation Engine
Dashboard
MQTT
API
```

excepto para registrar sus capacidades.

---

# 173. Ejemplo de nuevo actuador

```text
New Motor
    ↓
Motor Driver
    ↓
Motor Entity
    ↓
Capabilities:
    start
    stop
    speed
    direction
```

Automatización:

```text
garage.motor.speed = 50%
```

sin conocer el hardware.

---

# 174. Tests

Debe existir una estrategia de pruebas.

### Unit tests

Para:

```text
Drivers
Parsers
Automation
Config
Protocol
```

### Integration tests

Para:

```text
MQTT
Modbus
CAN
API
```

### Hardware tests

Para:

```text
GPIO
Sensors
Displays
Relays
```

---

# 175. Simulación

Debe existir un modo:

```text
SIMULATION
```

que permita ejecutar el núcleo sin hardware físico.

Ejemplo:

```text
Simulated Temperature = 25°C
```

y comprobar:

```text
Automation → Fan ON
```

---

# 176. Mock devices

Debe poder existir:

```text
MockSensor
MockLight
MockSwitch
MockEnergyMeter
```

Esto facilita desarrollo y pruebas.

---

# 177. CI/CD

El repositorio debería utilizar:

```text
Build
Unit tests
Static analysis
Documentation validation
Firmware compilation
```

para cada target.

---

# 178. Targets mínimos

Inicialmente:

```text
ESP32-WROOM-32
ESP32-S3
ESP32-C6
```

Posteriormente:

```text
ESP32-C3
ESP32-C5
ESP32-H2
```

---

# 179. Perfiles de producto

La misma plataforma podrá generar diferentes productos.

### Basic Node

```text
ESP32
GPIO
Wi-Fi
MQTT
Web
```

### Ethernet Node

```text
ESP32
Ethernet
MQTT
Modbus
```

### HMI Node

```text
ESP32-S3
Display
Touch
Wi-Fi
MQTT
```

### Wireless Gateway

```text
ESP32-C6
Wi-Fi
Thread
Zigbee
Matter
```

### Industrial Node

```text
ESP32
Ethernet
RS485
CAN
CANopen
Modbus
```

---

# 180. Ejemplo residencial

```text
Casa
│
├── Living
│   ├── ESP32-S3 Touch
│   ├── Light
│   ├── Fan
│   ├── Curtain
│   └── Temperature
│
├── Cocina
│   ├── Light
│   ├── Energy
│   └── Water
│
├── Garage
│   ├── Door
│   ├── Alarm
│   └── Camera
│
└── Central
    └── MQTT
```

---

# 181. Ejemplo industrial

```text
Industrial Network
│
├── PLC-like Controller
│
├── CANopen
│   ├── Motor
│   ├── Sensor
│   └── Actuator
│
├── Modbus RTU
│   ├── Temperature
│   ├── Pressure
│   └── Inverter
│
└── Ethernet
    └── SCADA/API
```

---

# 182. Ejemplo marino

```text
ESP32
│
├── CAN
│
├── CANopen
│
├── Sensors
│
├── Engine Data
│
├── Energy
│
└── Display
```

La plataforma deberá permitir integrar dispositivos comerciales mediante drivers específicos sin modificar el núcleo.

---

# 183. Ejemplo de automatización residencial

```text
Motion detected
AND
Lux < 100

→ Light ON

After 120 seconds

→ Light OFF
```

---

# 184. Ejemplo energético

```text
Power > 4000 W

AND

House mode == normal

→ Disable water heater

→ Notification

→ Log event
```

---

# 185. Ejemplo climático

```text
Temperature > 28°C

→ Fan 50%

Temperature > 30°C

→ Fan 80%

Temperature > 32°C

→ Fan 100%
```

---

# 186. Ejemplo de agua

```text
Leak sensor = ON

→ Close valve

→ Turn pump OFF

→ Alarm ON

→ Notification
```

---

# 187. Ejemplo de pantalla

Pantalla del dormitorio:

```text
┌─────────────────────────────┐
│ Dormitorio                  │
├─────────────────────────────┤
│ 22.4 °C      54 %           │
│ Temperatura  Humedad         │
├─────────────────────────────┤
│       💡 LUZ                │
│          ON                 │
├─────────────────────────────┤
│ Cortina       60 %          │
├─────────────────────────────┤
│        🌙 NOCHE             │
└─────────────────────────────┘
```

Todos estos elementos deben ser bloques configurables.

---

# 188. API como fuente única de verdad

La UI web y la pantalla no deben tener lógica independiente.

Ambas deben consumir:

```text
Device Model
API
Events
```

Así:

```text
Web UI
   │
   ├── API
   │
Touch UI
   │
   └── API
```

ambas muestran el mismo estado.

---

# 189. Arquitectura de software completa

```text
                    ┌───────────────────────┐
                    │       CLIENTS         │
                    │                       │
                    │ Web │ Touch │ Mobile │
                    └───────────┬───────────┘
                                │
                             REST/WS
                                │
                    ┌───────────▼───────────┐
                    │          API          │
                    └───────────┬───────────┘
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                 │
       ┌──────▼──────┐   ┌──────▼──────┐   ┌──────▼──────┐
       │ Automation  │   │  Services   │   │   Security  │
       └──────┬──────┘   └──────┬──────┘   └─────────────┘
              │                 │
              └────────┬────────┘
                       │
                ┌──────▼──────┐
                │ Device Model│
                └──────┬──────┘
                       │
              ┌────────▼────────┐
              │ Communication   │
              └────────┬────────┘
                       │
       ┌───────────────┼────────────────┐
       │               │                │
     MQTT           Modbus          CANopen
       │               │                │
     Wi-Fi         RS485/TCP          CAN
       │
    Ethernet
```

---

# 190. Roadmap

## Fase 0 — Arquitectura

* definir Device Model;
* definir Capability Model;
* definir Event Model;
* definir Command Model;
* definir configuración;
* definir Module Registry;
* definir ADR.

---

## Fase 1 — Core

Implementar:

```text
Core
HAL
Configuration
Logging
EventBus
DeviceRegistry
ModuleRegistry
```

---

## Fase 2 — Hardware

Implementar:

```text
GPIO
ADC
PWM
I2C
SPI
UART
```

---

## Fase 3 — Networking

Implementar:

```text
Wi-Fi
Ethernet
mDNS
NTP
HTTP
WebSocket
```

---

## Fase 4 — MQTT

Implementar:

```text
MQTT client
Discovery
State
Commands
Events
```

---

## Fase 5 — Web UI

Implementar:

```text
Login
Dashboard
Devices
Sensors
Outputs
Configuration
Diagnostics
```

---

## Fase 6 — Automation

Implementar:

```text
Triggers
Conditions
Actions
Scenes
Routines
Schedules
```

---

## Fase 7 — Sensores

Implementar inicialmente:

```text
AHT20
BME280
DS18B20
Digital input
Analog input
Pulse counter
```

---

## Fase 8 — Actuators

Implementar:

```text
Relay
PWM
Dimmer abstraction
Fan
Cover
```

---

## Fase 9 — Energy

Implementar:

```text
Voltage
Current
Power
Energy
```

---

## Fase 10 — Water

Implementar:

```text
Flow
Volume
Leak
Level
```

---

## Fase 11 — Modbus

Implementar:

```text
Modbus RTU
Modbus TCP
Profiles
Discovery
```

---

## Fase 12 — CAN

Implementar:

```text
CAN
CANopen
Node management
PDO
SDO
EMCY
```

---

## Fase 13 — Touchscreen

Implementar:

```text
Display abstraction
Touch abstraction
Widget system
Screen configuration
```

---

## Fase 14 — ESP32-S3

Agregar:

```text
Camera
AI
Advanced HMI
Local analytics
```

---

## Fase 15 — ESP32-C6

Agregar:

```text
Thread
Zigbee
Matter
Gateway
Bridge
```

---

## Fase 16 — Ecosistema

Integrar:

```text
Home Assistant
Homey Pro
Apple Home
Google Home
Samsung SmartThings
```

---

# 191. Prioridad de desarrollo

La prioridad debe ser:

```text
1. Core
2. Device Model
3. Event Bus
4. Configuration
5. Network
6. MQTT
7. Web
8. Automation
9. Drivers
10. Modbus
11. CANopen
12. Display
13. Matter/Zigbee/Thread
14. AI
```

No se debe comenzar por Matter, IA o una pantalla compleja antes de estabilizar el modelo de dispositivo.

---

# 192. Principio fundamental de desarrollo

Cada nuevo hardware deberá responder:

```text
¿Qué es?
¿Qué capacidades tiene?
¿Cómo se comunica?
¿Qué datos expone?
¿Qué comandos acepta?
¿Qué configuración necesita?
¿Qué dependencias tiene?
¿Cómo se diagnostica?
¿Cómo se actualiza?
¿Cómo se integra con la UI?
¿Cómo se integra con API?
¿Cómo se integra con automatizaciones?
```

Si estas preguntas pueden responderse mediante las interfaces existentes, el diseño es correcto.

---

# 193. Criterio para aceptar un módulo

Un módulo nuevo debe:

* tener responsabilidad definida;
* tener manifest;
* declarar dependencias;
* utilizar interfaces existentes;
* no modificar módulos no relacionados;
* incluir configuración;
* incluir diagnóstico;
* incluir documentación;
* incluir ejemplo;
* tener pruebas;
* poder deshabilitarse.

---

# 194. Criterio para aceptar un nuevo driver

Un driver debe:

* implementar una interfaz estándar;
* ocultar detalles del hardware;
* exponer capacidades;
* proporcionar estado;
* proporcionar errores;
* proporcionar configuración;
* soportar diagnóstico;
* evitar lógica de negocio.

---

# 195. Criterio para aceptar una nueva integración

Una integración externa debe:

* ser opcional;
* utilizar la API o Device Model;
* no modificar la lógica central;
* tener autenticación;
* disponer de configuración;
* disponer de logs;
* poder deshabilitarse.

---

# 196. Principio de evolución

El sistema debe poder evolucionar:

```text
ESP32
 ↓
ESP32-S3
 ↓
ESP32-C6
 ↓
Gateway
 ↓
Central Server
 ↓
Multi-site
```

sin cambiar el concepto fundamental de:

```text
Device
Entity
Capability
Event
Command
Automation
```

---

# 197. Definición final de la plataforma

La plataforma puede resumirse como:

```text
                    AUTOMATION PLATFORM
                            │
        ┌───────────────────┼────────────────────┐
        │                   │                    │
     DEVICES            SERVICES           AUTOMATION
        │                   │                    │
        ├── Sensors         ├── Climate          ├── Rules
        ├── Outputs         ├── Energy           ├── Scenes
        ├── Displays        ├── Water            ├── Routines
        ├── Gateways        ├── Security         ├── Schedules
        └── Controllers     └── Weather          └── Events
        │
        └───────────────────┬────────────────────┘
                            │
                       DEVICE MODEL
                            │
        ┌───────────────────┼────────────────────┐
        │                   │                    │
      MQTT                Modbus              CANopen
        │                   │                    │
    Ethernet              RS485                 CAN
        │
   Wi-Fi / IP
        │
 ┌──────┼───────────┐
 │      │           │
Matter Thread     Zigbee
 │
ESP32-C6 / ESP32-H2
```

---

# 198. Resultado esperado

El resultado final debe ser una plataforma donde crear un nuevo sistema de automatización sea principalmente una cuestión de configuración.

Por ejemplo:

```text
Crear habitación
       ↓
Agregar ESP32
       ↓
Configurar hardware
       ↓
Agregar sensor
       ↓
Agregar relay
       ↓
Asignar capacidades
       ↓
Crear automatización
       ↓
Crear pantalla
       ↓
Agregar widgets
       ↓
Publicar
```

sin tener que desarrollar nuevamente toda la aplicación.

---

# 199. Objetivo a largo plazo

El objetivo final es disponer de una plataforma que permita construir:

```text
Smart Home
Smart Office
Smart Building
Smart Workshop
Smart Farm
Weather Station
Energy Monitor
Industrial Controller
Marine Controller
Security System
HVAC Controller
Water Management
```

utilizando la misma base tecnológica.

---

# 200. Regla de oro

> **El hardware cambia. Los protocolos cambian. Los sensores cambian. Los fabricantes cambian. La plataforma no debería tener que cambiar por cada uno de ellos.**

Por eso:

```text
Hardware
   ↓
Driver
   ↓
Capability
   ↓
Device
   ↓
Service
   ↓
Automation
   ↓
Interface
```

debe mantenerse como la arquitectura fundamental del proyecto.

---

# 201. Estado inicial del proyecto

```text
Estado: Planificación

Arquitectura: Definida conceptualmente

Framework:
PlatformIO

Plataforma principal:
ESP32

Targets iniciales:
ESP32-WROOM
ESP32-S3
ESP32-C6

Comunicación principal:
MQTT / TCP-IP

Field buses:
Modbus
CAN/CANopen

Wireless ecosystems:
Matter
Thread
Zigbee
BLE

Interfaces:
Web
REST API
WebSocket
Touchscreen

Automation:
Rules
Scenes
Routines
Schedules

Security:
Authentication
Authorization
TLS
OTA security

Extensibilidad:
Module Registry
Device Registry
Capability Model

Documentación:
README
DESIGN-SYSTEM
ADR
DUDAS-Y-DECISIONES
API documentation
Module documentation
Hardware documentation
```

---

# 202. Próximo paso

Antes de comenzar a implementar sensores, relés o pantallas, se recomienda crear primero los siguientes documentos:

```text
docs/
├── ARCHITECTURE.md
├── DEVICE-MODEL.md
├── CAPABILITY-MODEL.md
├── EVENT-MODEL.md
├── COMMAND-MODEL.md
├── COMMUNICATION.md
├── MQTT-SPECIFICATION.md
├── API.md
├── AUTOMATION-ENGINE.md
├── MODULE-DEVELOPMENT.md
├── DRIVER-DEVELOPMENT.md
├── HARDWARE-PROFILES.md
├── SECURITY.md
├── UI-ARCHITECTURE.md
├── DISPLAY-ARCHITECTURE.md
├── ADR/
│   ├── ADR-001-device-model.md
│   ├── ADR-002-mqtt.md
│   ├── ADR-003-modbus.md
│   ├── ADR-004-canopen.md
│   ├── ADR-005-matter.md
│   └── ...
└── DUDAS-Y-DECISIONES.md
```

La implementación debe comenzar por el **núcleo abstracto**, no por un sensor concreto.

El primer objetivo técnico debería ser conseguir:

```text
ESP32
  ↓
Device Model
  ↓
Event Bus
  ↓
MQTT
  ↓
Web/API
  ↓
Automation
```

y poder demostrar una automatización sencilla:

```text
Botón
 ↓
Evento
 ↓
Automation Engine
 ↓
Comando
 ↓
Relé
 ↓
Estado
 ↓
MQTT
 ↓
Web
```

Una vez que esto funcione correctamente, los sensores, pantallas, Modbus, CANopen, Matter, Zigbee, Thread, energía, agua e IA podrán incorporarse como módulos sobre una arquitectura estable.

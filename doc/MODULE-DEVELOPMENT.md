# MODULE-DEVELOPMENT.md

> **Sistema:** Plataforma de Automatización Distribuida
> **Tipo:** Convención de desarrollo
> **Estado:** Definición
> **Versión:** 1.0.0
> **Última actualización:** 2026-10-06

---

# 1. Objetivo

Este documento define cómo deben diseñarse, desarrollarse, integrarse, configurarse y mantenerse los módulos de la plataforma.

La plataforma está diseñada para crecer mediante módulos independientes que puedan incorporarse sin modificar la arquitectura principal.

Un módulo puede representar:

* un sensor;
* un actuador;
* un driver;
* una interfaz de comunicación;
* un expansor de GPIO;
* un display;
* una pantalla táctil;
* una cámara;
* un sistema de almacenamiento;
* una función;
* una integración;
* una automatización;
* un servicio;
* una función de energía;
* una función de seguridad;
* una función industrial;
* una función agrícola;
* una función marina;
* una funcionalidad de inteligencia artificial.

La arquitectura debe permitir:

```text
Agregar módulo
      ↓
Declarar dependencias
      ↓
Detectar compatibilidad
      ↓
Inicializar
      ↓
Registrar capacidades
      ↓
Crear recursos / entidades
      ↓
Exponer configuración
      ↓
Integrar con el sistema
```

sin modificar innecesariamente el resto de la plataforma.

---

# 2. Principio fundamental

> **Un módulo debe aportar una capacidad al sistema sin convertirse en una dependencia innecesaria del resto de la plataforma.**

La arquitectura debe seguir:

```text
Hardware
    ↓
Build Profile
    ↓
HAL
    ↓
Driver
    ↓
Module
    ↓
Capability
    ↓
Entity
    ↓
Function
    ↓
Automation
```

El módulo no debe asumir que existe un hardware específico salvo cuando esa sea precisamente su función.

---

# 3. Qué es un módulo

Un módulo es una unidad funcional autocontenida que proporciona una o varias capacidades al sistema.

Ejemplos:

```text
Temperature Sensor Module
Ethernet Module
CAN Module
RS485 Module
MCP23017 Module
74HC595 Module
Display Module
Touch Module
Camera Module
Energy Meter Module
Irrigation Module
Lighting Module
Alarm Module
AI Camera Module
Weather Module
MQTT Integration Module
Matter Integration Module
```

---

# 4. Tipos de módulos

La plataforma debe distinguir diferentes categorías.

## 4.1 Hardware Module

Representa hardware físico.

Ejemplos:

```text
MCP23017
MCP23S17
W5500
LAN8720A
DS18B20
BME280
GT911
ST7789
```

---

## 4.2 Driver Module

Permite controlar un dispositivo físico.

Ejemplo:

```text
W5500Driver
MCP23017Driver
ST7789Driver
GT911Driver
```

Normalmente pertenece a la capa de hardware.

---

## 4.3 Service Module

Proporciona una función reutilizable.

Ejemplos:

```text
NetworkService
TimeService
StorageService
WatchdogService
NotificationService
EnergyService
```

---

## 4.4 Functional Module

Implementa una función del sistema.

Ejemplos:

```text
Lighting
Irrigation
HVAC
Security
Garage
Door Control
Energy Management
Water Management
```

---

## 4.5 Integration Module

Conecta el sistema con otro sistema.

Ejemplos:

```text
MQTT
Matter
Home Assistant
Homey
SmartThings
Apple Home
Google Home
Alexa
REST
WebSocket
```

---

## 4.6 UI Module

Agrega componentes a la interfaz web.

Ejemplos:

```text
Temperature Card
Energy Graph
Camera Viewer
Network Status
Ethernet Configuration
Automation Editor
Device Diagnostics
```

---

## 4.7 AI Module

Módulos que utilizan procesamiento local o externo.

Ejemplos:

```text
Object Detection
Presence Detection
Image Classification
Anomaly Detection
Predictive Maintenance
Consumption Prediction
Behavior Learning
```

---

# 5. Arquitectura de un módulo

Un módulo debe seguir una estructura similar a:

```text
Module
│
├── Metadata
├── Lifecycle
├── Configuration
├── Dependencies
├── Capabilities
├── Resources
├── Entities
├── Commands
├── Events
├── State
├── Diagnostics
├── API
└── UI
```

No todos los módulos necesitan todos los componentes.

---

# 6. Separación de responsabilidades

Un módulo no debe mezclar indiscriminadamente:

```text
Hardware
Configuración
UI
API
Automatización
Persistencia
Comunicación
```

Debe existir separación.

Arquitectura recomendada:

```text
Module
│
├── Core
│
├── Hardware Adapter
│
├── Service
│
├── Configuration
│
├── API
│
├── Events
└── UI
```

---

# 7. Dependencias

Cada módulo debe declarar sus dependencias.

Ejemplo:

```json
{
  "module": "camera_ai",
  "version": "1.0.0",
  "requires": {
    "camera": true,
    "psram": true
  }
}
```

Otro:

```json
{
  "module": "ethernet_w5500",
  "version": "1.0.0",
  "requires": {
    "spi": true
  }
}
```

---

# 8. Dependencias de hardware

Un módulo puede requerir:

```text
GPIO
SPI
I2C
UART
CAN
PSRAM
Flash
SD
Camera Interface
Ethernet
Wi-Fi
```

Pero debe declarar estos requisitos de manera explícita.

Ejemplo:

```json
{
  "requirements": {
    "bus": "SPI",
    "cs": true
  }
}
```

---

# 9. Dependencias de software

También pueden existir dependencias de software:

```text
NetworkService
StorageService
TimeService
SecurityService
SystemBus
DeviceModel
```

Ejemplo:

```text
Camera AI
    ↓
CameraService
    ↓
PSRAM
    ↓
AI Runtime
```

---

# 10. Build Profile y módulos

Los Build Profiles determinan qué módulos pueden compilarse.

Ejemplo:

```ini
[env:esp32_s3]
board = esp32-s3-devkitc-1

build_flags =
    -D BOARD_ESP32_S3
    -D MCU_ESP32_S3
    -D HAS_PSRAM
    -D HAS_CAMERA
```

El módulo puede comprobar:

```cpp
#if defined(HAS_CAMERA) && defined(HAS_PSRAM)

    // Compilar Camera AI

#endif
```

Pero la lógica del módulo debe permanecer independiente del modelo concreto de MCU siempre que sea posible.

---

# 11. Módulos opcionales

Los módulos opcionales no deben aumentar innecesariamente el firmware de todos los dispositivos.

Ejemplo:

```text
ESP32-WROOM
    ├── Core
    ├── Network
    ├── Sensors
    └── Automation

ESP32-S3
    ├── Core
    ├── Network
    ├── Camera
    ├── AI
    ├── Display
    └── Automation
```

La compilación puede excluir componentes que no sean necesarios.

---

# 12. Módulos runtime

No todo debe decidirse durante la compilación.

Un módulo puede existir físicamente pero estar deshabilitado:

```text
MCP23017
    ↓
Hardware detectado
    ↓
Module disponible
    ↓
Usuario lo deshabilita
    ↓
Module disabled
```

Por lo tanto:

```text
Build Profile
```

define:

> ¿Qué hardware/software puede existir?

Mientras que:

```text
Runtime Configuration
```

define:

> ¿Qué módulos están activos?

---

# 13. Estados de un módulo

Todo módulo debe tener un estado.

Estados recomendados:

```text
UNINITIALIZED
INITIALIZING
READY
RUNNING
DISABLED
DEGRADED
ERROR
UNAVAILABLE
STOPPING
STOPPED
```

Ejemplo:

```text
Camera Module
      ↓
INITIALIZING
      ↓
READY
      ↓
RUNNING
```

Si la cámara falla:

```text
RUNNING
   ↓
ERROR
   ↓
DEGRADED / UNAVAILABLE
```

---

# 14. Lifecycle

Los módulos deben seguir un ciclo de vida uniforme.

```text
Created
   ↓
Loaded
   ↓
Dependencies Checked
   ↓
Initialized
   ↓
Configured
   ↓
Started
   ↓
Running
   ↓
Stopping
   ↓
Stopped
```

---

# 15. Inicialización

La inicialización debe ser independiente del orden arbitrario de archivos.

Ejemplo:

```cpp
module.init();
```

debe:

* comprobar dependencias;
* validar configuración;
* reservar recursos;
* inicializar hardware;
* registrar capabilities;
* preparar entidades.

No debería comenzar inmediatamente tareas complejas si todavía no se han inicializado sus dependencias.

---

# 16. Dependencias entre módulos

Ejemplo:

```text
Camera AI
    │
    ├── CameraService
    ├── PSRAM
    └── StorageService
```

El sistema debe comprobar:

```text
¿CameraService disponible?
¿PSRAM disponible?
¿Storage disponible?
```

antes de activar el módulo.

---

# 17. Dependencias obligatorias y opcionales

Ejemplo:

```json
{
  "requires": [
    {
      "module": "camera",
      "mandatory": true
    },
    {
      "module": "storage",
      "mandatory": false
    }
  ]
}
```

Si falla una dependencia obligatoria:

```text
Module = UNAVAILABLE
```

Si falla una dependencia opcional:

```text
Module = DEGRADED
```

---

# 18. Registro de módulos

El sistema debe mantener un registro:

```text
ModuleRegistry
```

Ejemplo:

```cpp
ModuleRegistry.registerModule(&ethernetModule);
ModuleRegistry.registerModule(&sensorModule);
ModuleRegistry.registerModule(&automationModule);
```

El registro debe permitir:

```text
buscar
activar
desactivar
reiniciar
diagnosticar
consultar estado
```

---

# 19. Identidad de módulo

Cada módulo debe tener:

```text
module_id
name
version
type
vendor
description
```

Ejemplo:

```json
{
  "module_id": "ethernet.w5500",
  "name": "W5500 Ethernet",
  "version": "1.0.0",
  "type": "hardware"
}
```

---

# 20. Versionado

Los módulos deben utilizar versionado semántico:

```text
MAJOR.MINOR.PATCH
```

Ejemplo:

```text
1.0.0
1.1.0
1.1.1
2.0.0
```

### MAJOR

Cambios incompatibles.

### MINOR

Nuevas capacidades compatibles.

### PATCH

Correcciones.

---

# 21. Compatibilidad de módulos

Un módulo debe declarar:

```text
minimum_platform_version
minimum_api_version
minimum_module_version
```

Ejemplo:

```json
{
  "compatibility": {
    "platform": ">=1.0.0",
    "api": ">=1.0.0",
    "device_model": ">=1.0.0"
  }
}
```

---

# 22. Resources

Un módulo puede registrar recursos.

Ejemplo:

```text
MCP23017
    ↓
16 GPIO
```

El módulo puede crear:

```text
resource.mcp23017.gpio0
resource.mcp23017.gpio1
...
resource.mcp23017.gpio15
```

---

# 23. Capabilities

Los recursos se convierten en capabilities.

Ejemplo:

```text
GPIO
 ↓
Output
 ↓
on_off
```

Otro:

```text
ADC
 ↓
Analog Input
 ↓
voltage
```

Otro:

```text
Sensor
 ↓
Temperature
 ↓
temperature
```

---

# 24. Entities

Las capabilities pueden generar entidades.

Ejemplo:

```text
MCP23017 GPIO0
      ↓
Relay
      ↓
switch.pump
```

El módulo no debe crear IDs arbitrarios en cada arranque.

Los IDs deben ser estables.

---

# 25. Configuración del módulo

Cada módulo debe definir su configuración.

Ejemplo:

```json
{
  "enabled": true,
  "name": "Ethernet Principal",
  "interface": "w5500",
  "spi_bus": 2,
  "cs_pin": 5,
  "reset_pin": 4
}
```

La configuración debe almacenarse mediante el sistema de configuración del proyecto.

---

# 26. Configuración física

Cuando corresponda, el módulo puede definir:

```text
GPIO
SPI
I2C
UART
CS
INT
RESET
ADDRESS
```

Ejemplo MCP23017:

```json
{
  "bus": "i2c0",
  "address": "0x20",
  "interrupt": 16
}
```

---

# 27. Configuración lógica

Debe separarse:

```text
Configuración física
```

de:

```text
Configuración lógica
```

Ejemplo:

```text
Físico:
MCP23017 GPIO3

Lógico:
switch.pool_pump
```

Esto permite cambiar el hardware sin romper las automatizaciones.

---

# 28. Configuración mediante Web UI

Los módulos deben poder proporcionar metadatos para construir automáticamente su interfaz.

Ejemplo:

```json
{
  "field": "address",
  "type": "select",
  "label": "Dirección I²C",
  "options": [
    "0x20",
    "0x21",
    "0x22",
    "0x23"
  ]
}
```

La UI puede generar:

```text
┌─────────────────────────────┐
│ MCP23017                    │
│                             │
│ Dirección I²C               │
│ [ 0x20 ▼ ]                  │
│                             │
│ [✓] Habilitado              │
└─────────────────────────────┘
```

---

# 29. UI modular

Cada módulo puede registrar bloques de interfaz.

Ejemplo:

```text
Module
 ├── Settings Card
 ├── Status Card
 ├── Diagnostics Card
 └── Control Card
```

Estos bloques deben ser reutilizables.

---

# 30. No acoplar UI y lógica

Incorrecto:

```cpp
if (buttonPressed)
{
    relayOn();
}
```

dentro del código HTML.

Correcto:

```text
UI
 ↓
API
 ↓
Command
 ↓
Module
 ↓
Entity
 ↓
Hardware
```

---

# 31. API de módulo

Cuando corresponda, el módulo puede exponer endpoints.

Ejemplo:

```text
/api/v1/modules
/api/v1/modules/{module_id}
/api/v1/modules/{module_id}/config
/api/v1/modules/{module_id}/diagnostics
```

Pero los módulos funcionales deben preferir las entidades y comandos estándar cuando sea posible.

---

# 32. Commands

Los módulos pueden registrar comandos.

Ejemplo:

```text
camera.capture
camera.restart
display.clear
relay.test
ethernet.reconnect
```

El comando debe pasar por el sistema de autorización.

---

# 33. Events

Los módulos pueden generar eventos.

Ejemplos:

```text
camera.frame
sensor.updated
ethernet.connected
ethernet.disconnected
button.pressed
mcp23017.interrupt
```

Los eventos deben utilizar el modelo de eventos del sistema.

---

# 34. State

Los módulos pueden publicar estado.

Ejemplo:

```json
{
  "module": "ethernet.w5500",
  "state": {
    "connected": true,
    "link_speed": 100,
    "ip": "192.168.1.50"
  }
}
```

---

# 35. Diagnostics

Todo módulo debería proporcionar diagnóstico.

Ejemplo:

```json
{
  "status": "ok",
  "uptime": 3600,
  "errors": 0,
  "warnings": 1
}
```

Puede incluir:

```text
temperatura
errores
reinicios
timeouts
paquetes
estado de bus
estado de hardware
memoria
CPU
latencia
```

---

# 36. Health

Los módulos críticos deben implementar health checks.

Estados:

```text
HEALTHY
DEGRADED
UNHEALTHY
UNKNOWN
```

Ejemplo:

```text
Ethernet
    ↓
Link DOWN
    ↓
DEGRADED
```

No necesariamente debe detenerse todo el sistema.

---

# 37. Fallos aislados

Un fallo de módulo no debe detener el sistema completo.

Ejemplo:

```text
Camera AI
   ↓
ERROR
```

No debería provocar:

```text
Central
   ↓
RESTART
```

salvo que exista una condición crítica explícitamente definida.

---

# 38. Watchdog

Los módulos no deben bloquear indefinidamente.

Las operaciones largas deben:

* utilizar timeout;
* ceder CPU;
* permitir cancelación;
* informar errores.

Ejemplo:

```text
W5500 timeout
    ↓
Driver detects timeout
    ↓
Module DEGRADED
    ↓
Reconnect
```

---

# 39. FreeRTOS

Los módulos que necesiten tareas propias deben respetar la arquitectura FreeRTOS del proyecto.

No crear tareas indiscriminadamente.

Antes:

```cpp
xTaskCreate(...)
```

debe evaluarse:

```text
¿Necesita una tarea propia?
¿Puede utilizar una tarea existente?
¿Necesita Core 0?
¿Necesita Core 1?
¿Puede ejecutarse por eventos?
¿Puede utilizar timer?
```

---

# 40. Event-driven

Siempre que sea posible, se debe preferir:

```text
Event
 ↓
Callback / Queue
 ↓
Module
```

en lugar de:

```text
while(true)
{
    revisar();
}
```

Esto reduce:

* consumo de CPU;
* latencia innecesaria;
* cantidad de tareas;
* consumo de memoria.

---

# 41. Periodicidad

Cuando un módulo necesite ejecución periódica:

```text
Timer
Task
Scheduler
```

debe utilizar la infraestructura existente.

Ejemplo:

```text
TemperatureSensor
    ↓
cada 5 s
    ↓
read()
    ↓
publish state
```

No crear un task independiente si el Scheduler ya proporciona esa funcionalidad.

---

# 42. Prioridades

Los módulos deben utilizar prioridades razonables.

Jerarquía conceptual:

```text
Safety
   ↓
Real-time Control
   ↓
Communication
   ↓
Sensors
   ↓
Automation
   ↓
UI
   ↓
Diagnostics
   ↓
Background
```

---

# 43. Memoria

Los módulos deben minimizar:

* `malloc` frecuente;
* `new/delete` repetitivos;
* buffers gigantes;
* copias innecesarias;
* Strings dinámicos innecesarios.

Especialmente en:

```text
ESP32-WROOM
ESP32-C3
ESP32-C6
```

Los módulos que utilicen mucha memoria deben declarar requisitos.

---

# 44. PSRAM

Los módulos que necesiten PSRAM deben declararlo.

Ejemplo:

```text
Camera AI
    ↓
requires HAS_PSRAM
```

Si no existe:

```text
Module = UNAVAILABLE
```

o utilizar un modo reducido.

---

# 45. Modos degradados

Un módulo puede ofrecer diferentes niveles de funcionamiento.

Ejemplo:

```text
Camera AI

FULL:
    detección + almacenamiento + inferencia

DEGRADED:
    detección básica

MINIMAL:
    captura solamente
```

Esto permite aprovechar hardware diferente.

---

# 46. Ejemplo: módulo de cámara

```text
Camera Module
│
├── Camera Driver
├── Frame Buffer
├── Capture Service
├── Entity
├── Events
├── Diagnostics
└── UI
```

Si existe AI:

```text
Camera
  ↓
Frame
  ↓
AI Module
  ↓
Inference
  ↓
Entity / Event
```

---

# 47. Ejemplo: módulo Ethernet

```text
Ethernet Module
│
├── HAL
├── Driver
├── Configuration
├── State
├── Events
├── Diagnostics
└── API
```

Puede utilizar:

```text
LAN8720
```

o:

```text
W5500
```

pero exponer la misma interfaz:

```cpp
network.begin();
network.stop();
network.isConnected();
network.localIP();
```

---

# 48. Ejemplo: módulo MCP23017

```text
MCP23017
   ↓
I2C Driver
   ↓
GPIO Resources
   ↓
Capabilities
   ↓
Entities
```

El módulo debe permitir:

```text
GPIO0 → Relay
GPIO1 → Relay
GPIO2 → Button
GPIO3 → Sensor
```

sin que las automatizaciones conozcan que existe un MCP23017.

---

# 49. Ejemplo: módulo 74HC595

```text
74HC595
   ↓
SPI / Shift Register Driver
   ↓
8 / 16 / 24 / ... outputs
   ↓
Capabilities
   ↓
Entities
```

La cantidad de chips puede configurarse:

```text
1 → 8 outputs
2 → 16 outputs
3 → 24 outputs
```

El módulo debe ocultar la cadena física.

---

# 50. Módulos de integración

Las integraciones también son módulos.

Ejemplo:

```text
Matter Module
MQTT Module
Home Assistant Module
REST Module
```

Deben consumir el Device Model.

No deben acceder directamente a:

```text
GPIO
SPI
I²C
UART
```

La arquitectura correcta es:

```text
Hardware
   ↓
Device Model
   ↓
Integration
   ↓
External Ecosystem
```

---

# 51. Módulos y System Bus

Los módulos deben comunicarse mediante el System Bus cuando sea apropiado.

Ejemplo:

```text
Temperature Module
        ↓
temperature.updated
        ↓
System Bus
        ↓
HVAC Module
```

No:

```text
Temperature Module
        ↓
llamar directamente
        ↓
HVAC Module
```

Esto reduce el acoplamiento.

---

# 52. Comunicación síncrona vs eventos

Utilizar llamada directa cuando:

```text
resultado inmediato
```

Ejemplo:

```cpp
sensor.read();
```

Utilizar evento cuando:

```text
evento asíncrono
```

Ejemplo:

```text
door.opened
```

Utilizar command cuando:

```text
solicitud de acción
```

Ejemplo:

```text
cover.open
```

---

# 53. Persistencia

Los módulos no deberían escribir directamente en NVS/LittleFS sin pasar por el sistema de configuración.

Correcto:

```text
Module
 ↓
ConfigManager
 ↓
Storage
```

Esto permite:

* versionado;
* migraciones;
* backup;
* restauración;
* validación.

---

# 54. Migraciones

Cuando cambia la configuración de un módulo:

```text
v1
 ↓
Migration
 ↓
v2
```

Ejemplo:

```json
{
  "config_version": 2
}
```

El módulo debe poder migrar configuraciones antiguas cuando sea posible.

---

# 55. Configuración deseada y aplicada

Debe distinguirse:

```text
desired configuration
```

de:

```text
applied configuration
```

Ejemplo:

```text
Usuario configura W5500
       ↓
Desired Config
       ↓
Driver aplica configuración
       ↓
Applied Config
```

Si falla:

```text
Desired ≠ Applied
```

El sistema debe informar el error.

---

# 56. Seguridad

Los módulos deben respetar:

```text
Authentication
Authorization
Permissions
```

Un módulo no debe permitir:

```text
reset
firmware update
factory reset
hardware configuration
```

sin los permisos adecuados.

---

# 57. Permisos

Ejemplo:

```text
module.view
module.configure
module.control
module.diagnostics
module.admin
```

Los módulos críticos pueden requerir:

```text
module.admin
```

---

# 58. Descubrimiento

Los módulos deben poder informar:

```text
qué son
qué versión tienen
qué necesitan
qué proporcionan
qué estado tienen
```

Ejemplo:

```json
{
  "module_id": "io.mcp23017",
  "version": "1.0.0",
  "status": "running",
  "capabilities": [
    "gpio.input",
    "gpio.output",
    "gpio.interrupt"
  ]
}
```

---

# 59. Auto Discovery

Cuando sea técnicamente posible:

```text
Bus
 ↓
Device detected
 ↓
Driver identified
 ↓
Module created
 ↓
Capabilities registered
```

Ejemplo:

```text
I²C scan
   ↓
0x20
   ↓
MCP23017
   ↓
16 GPIO
```

Pero el sistema no debe asumir que todo dispositivo descubierto automáticamente es seguro para activar.

---

# 60. Provisioning

Un módulo puede requerir configuración inicial.

Ejemplo:

```text
Nuevo MCP23017
      ↓
Detected
      ↓
Needs configuration
      ↓
Web UI
      ↓
User assigns GPIOs
      ↓
Module Active
```

---

# 61. Hot Configuration

Cuando sea seguro, algunos parámetros pueden modificarse sin reiniciar.

Ejemplo:

```text
nombre
intervalo de sensor
umbrales
entidad
zona
```

Otros pueden requerir reinicio:

```text
SPI bus
GPIO crítico
Ethernet driver
memoria
hardware mode
```

El módulo debe declarar qué parámetros requieren restart.

---

# 62. Hot Plug

La plataforma puede soportar dispositivos conectados/desconectados cuando el hardware lo permita.

Ejemplo:

```text
RS485 Device
   ↓
Disconnected
   ↓
UNAVAILABLE
   ↓
Reconnected
   ↓
AVAILABLE
```

No debe destruir inmediatamente toda la configuración lógica.

---

# 63. Entidades virtuales

Un módulo puede crear entidades sin hardware físico.

Ejemplo:

```text
Energy Calculation Module
       ↓
energy.total
```

o:

```text
Presence Fusion Module
       ↓
binary_sensor.presence
```

---

# 64. Módulos compuestos

Un módulo puede estar formado por otros módulos.

Ejemplo:

```text
HVAC Module
│
├── Temperature
├── Humidity
├── Fan
├── Relay
└── Control Algorithm
```

El módulo compuesto debe utilizar interfaces estándar.

---

# 65. Módulos de automatización

Un módulo puede implementar una función completa:

```text
Irrigation Module
```

Puede utilizar:

```text
soil moisture
rain
water flow
valve
pump
schedule
```

pero debe trabajar mediante:

```text
Entities
Capabilities
Commands
Events
```

no directamente con GPIO.

---

# 66. Módulos de seguridad

Los módulos de seguridad tienen prioridad elevada.

Ejemplos:

```text
Alarm
Fire
Water Leak
Intrusion
Emergency Stop
```

Deben seguir:

```text
Local-first
Fail-safe
Priority
Redundancy when required
```

Nunca deben depender exclusivamente de:

```text
Internet
Cloud
Central
```

para una función crítica local.

---

# 67. Módulos críticos

Un módulo puede declarar:

```json
{
  "critical": true
}
```

Esto permite aplicar políticas especiales:

* mayor prioridad;
* watchdog;
* recuperación automática;
* persistencia local;
* ejecución sin Central;
* diagnóstico prioritario.

---

# 68. Módulos no críticos

Ejemplos:

```text
Weather API
Cloud Sync
Analytics
Remote Logging
```

Si fallan:

```text
System continues
```

---

# 69. Local-first

Todo módulo debe definir qué ocurre cuando:

```text
Internet = OFF
Central = OFF
Zone Controller = OFF
```

Ejemplo:

```text
Lighting Module
    ↓
Continúa localmente

Cloud Integration
    ↓
Unavailable
```

---

# 70. Autonomía

La prioridad es:

```text
DEVICE
   >
ZONE
   >
CENTRAL
   >
INTERNET
   >
CLOUD
```

Un módulo debe ejecutar su función en el nivel más bajo posible.

---

# 71. Comunicación con Central

Los módulos no deben depender de que Central esté disponible para iniciar.

Incorrecto:

```text
Boot
 ↓
esperar Central
 ↓
obtener configuración
 ↓
iniciar módulo
```

Correcto:

```text
Boot
 ↓
Load local configuration
 ↓
Initialize module
 ↓
Run
 ↓
Synchronize with Central
```

---

# 72. Sincronización

Cuando Central vuelva:

```text
Node
 ↓
Discovery
 ↓
State Sync
 ↓
Configuration Reconciliation
 ↓
Central Updated
```

El módulo debe continuar funcionando durante el proceso.

---

# 73. API y módulos

La API puede consultar:

```text
/api/v1/modules
```

y:

```text
/api/v1/modules/{module_id}
```

Pero las operaciones funcionales deben utilizar:

```text
entities
commands
events
```

cuando sea posible.

---

# 74. Documentación de módulo

Cada módulo debe tener documentación.

Estructura:

```text
modules/
└── module-name/
    ├── README.md
    ├── CONFIGURATION.md
    ├── HARDWARE.md
    └── API.md
```

---

# 75. README de módulo

Debe incluir:

```text
Nombre
Descripción
Versión
Autor
Licencia
Dependencias
Hardware soportado
Build Profiles
Configuración
Entidades
Eventos
Comandos
API
Limitaciones
Diagnóstico
Ejemplos
```

---

# 76. Testing

Todo módulo debe tener pruebas.

Como mínimo:

```text
Build test
Initialization test
Configuration test
Error test
Recovery test
API test
```

Cuando sea posible:

```text
Hardware test
Integration test
Stress test
```

---

# 77. Test de compilación

Cada módulo debe compilar en los perfiles compatibles.

Ejemplo:

```text
ESP32-WROOM
ESP32-S3
ESP32-C6
```

cuando corresponda.

No debe exigirse que un módulo compile para hardware que explícitamente no soporta.

---

# 78. Matriz de compatibilidad

Ejemplo:

| Módulo           |    WROOM | S3 |       C3 |       C6 | P4 |
| ---------------- | -------: | -: | -------: | -------: | -: |
| Ethernet LAN8720 |        ✓ |  — |        — |        — | ✓* |
| Ethernet W5500   |        ✓ |  ✓ |        ✓ |        ✓ |  ✓ |
| Camera           |        ✓ |  ✓ | limitado | limitado |  ✓ |
| AI               | limitado |  ✓ | limitado | limitado |  ✓ |
| MCP23017         |        ✓ |  ✓ |        ✓ |        ✓ |  ✓ |
| ST7789           |        ✓ |  ✓ |        ✓ |        ✓ |  ✓ |

`✓*` debe validarse para el hardware concreto.

---

# 79. Recursos compartidos

Los módulos no deben asumir que un periférico es exclusivamente suyo.

Ejemplo:

```text
SPI Bus
 ├── W5500
 ├── Display
 └── SD
```

Debe existir:

```text
BusManager
```

para coordinar el acceso.

---

# 80. I²C

Mismo principio:

```text
I²C
 ├── MCP23017
 ├── BME280
 ├── RTC
 └── Touch Controller
```

Los módulos deben utilizar:

```text
I2CBusManager
```

en lugar de controlar directamente el bus global.

---

# 81. SPI

Ejemplo:

```text
SPI
 ├── W5500
 ├── Display
 ├── SD
 └── MCP23S17
```

Cada dispositivo debe declarar:

```text
bus
CS
frequency
mode
```

y utilizar el administrador correspondiente.

---

# 82. GPIO

Los módulos no deben reservar GPIO sin registrar el recurso.

Debe existir:

```text
ResourceManager
```

que permita saber:

```text
GPIO 5
 ↓
ocupado por W5500 CS
```

y evitar:

```text
GPIO 5
 ↓
también utilizado por Relay
```

---

# 83. Resource Conflict

Si existe conflicto:

```text
W5500 CS → GPIO5
Relay → GPIO5
```

el sistema debe detectar:

```text
RESOURCE_CONFLICT
```

antes de activar ambos módulos.

---

# 84. Reserva de recursos

El proceso recomendado:

```text
Module
 ↓
Request Resource
 ↓
ResourceManager
 ↓
Available?
 ├── YES → Reserve
 └── NO → Reject
```

---

# 85. Liberación

Cuando un módulo se deshabilita:

```text
Module stop
 ↓
Release resources
 ↓
Resources available
```

Esto permite reutilización cuando sea seguro.

---

# 86. Prioridad de recursos

Algunos recursos pueden ser críticos.

Ejemplo:

```text
Emergency Stop GPIO
```

no debería poder ser reasignado por un módulo normal.

---

# 87. Namespaces

Los módulos deben utilizar nombres consistentes.

Ejemplos:

```text
ethernet.w5500
ethernet.lan8720
sensor.ds18b20
sensor.bme280
io.mcp23017
display.st7789
touch.gt911
camera.ov2640
```

---

# 88. Entidades generadas

Ejemplo:

```text
sensor.office.temperature
sensor.office.humidity
switch.pump
light.living
cover.garage
binary_sensor.front_door
```

El módulo puede proporcionar el origen:

```json
{
  "entity_id": "sensor.office.temperature",
  "source": "sensor.bme280"
}
```

---

# 89. No acoplar Entity ID al hardware

Incorrecto:

```text
mcp23017_gpio3_relay
```

Correcto:

```text
switch.pool_pump
```

El hardware puede cambiar posteriormente.

---

# 90. Reasignación de hardware

Ejemplo:

```text
Antes:
MCP23017 GPIO3
   ↓
switch.pool_pump
```

Después:

```text
ESP32 GPIO18
   ↓
switch.pool_pump
```

La automatización permanece:

```text
switch.pool_pump
```

---

# 91. Eventos de módulo

Formato conceptual:

```json
{
  "event_id": "01...",
  "type": "module.state_changed",
  "source": "ethernet.w5500",
  "timestamp": "2026-10-06T12:00:00Z",
  "data": {
    "state": "connected"
  }
}
```

---

# 92. Telemetría

Los módulos pueden publicar:

```text
metrics
```

Ejemplos:

```text
cpu_usage
memory_free
temperature
packet_count
error_count
read_latency
write_latency
```

No toda telemetría debe publicarse permanentemente.

Debe existir una política configurable.

---

# 93. Logging

Los módulos deben utilizar el sistema centralizado de logging.

Niveles:

```text
TRACE
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

No deben utilizar `Serial.println()` indiscriminadamente.

Preferir:

```cpp
LOG_INFO("Ethernet connected");
```

---

# 94. Errores

Los módulos deben utilizar errores estructurados.

Ejemplo:

```text
MODULE_NOT_FOUND
DEPENDENCY_MISSING
RESOURCE_CONFLICT
INVALID_CONFIGURATION
INITIALIZATION_FAILED
TIMEOUT
HARDWARE_ERROR
COMMUNICATION_ERROR
UNSUPPORTED
```

---

# 95. Recuperación

Cuando sea posible:

```text
Error
 ↓
Retry
 ↓
Backoff
 ↓
Reinitialize
 ↓
Recover
```

No utilizar loops de retry infinitos sin control.

---

# 96. Backoff

Ejemplo:

```text
1 s
2 s
4 s
8 s
16 s
30 s
```

con límite máximo.

---

# 97. Persistencia de errores

Los errores críticos pueden almacenarse:

```text
Last Error
Error Counter
Last Recovery
```

para diagnóstico después de reinicios.

---

# 98. Factory Reset

Un módulo debe poder restaurar su configuración cuando corresponda.

Debe distinguirse:

```text
Reset module
```

de:

```text
Factory reset entire device
```

---

# 99. Actualización

Los módulos pueden actualizarse como parte del firmware.

En versiones futuras podrían existir módulos dinámicos, pero:

> **No se debe asumir que el ESP32 puede cargar código arbitrario dinámicamente.**

La estrategia principal es:

```text
Firmware Build
    ↓
Build Profile
    ↓
Included Modules
```

---

# 100. Seguridad de módulos

Los módulos deben evitar:

* acceso directo no autorizado;
* comandos sin autenticación;
* cambios de hardware sin permisos;
* exposición innecesaria de información;
* credenciales en logs;
* secretos en firmware.

---

# 101. Secretos

Los módulos que utilicen:

```text
API keys
passwords
tokens
certificates
```

deben utilizar el sistema seguro de almacenamiento definido por la plataforma.

Nunca:

```cpp
const char* password = "123456";
```

en código.

---

# 102. Módulos de terceros

Los módulos de terceros deben declarar:

```text
author
license
version
dependencies
source
security considerations
```

y pasar controles de compatibilidad antes de incorporarse.

---

# 103. Licencias

Cada módulo debe respetar la licencia de las dependencias que utiliza.

Debe existir:

```text
THIRD-PARTY-NOTICES
```

cuando corresponda.

---

# 104. Estructura recomendada

Una implementación puede organizarse así:

```text
src/
└── modules/
    ├── ethernet/
    │   ├── EthernetModule.hpp
    │   ├── EthernetModule.cpp
    │   ├── EthernetConfig.hpp
    │   └── EthernetDiagnostics.cpp
    │
    ├── sensors/
    │   ├── TemperatureModule.hpp
    │   └── TemperatureModule.cpp
    │
    ├── expanders/
    │   ├── MCP23017Module.hpp
    │   └── MCP23017Module.cpp
    │
    ├── display/
    │   └── DisplayModule.cpp
    │
    └── camera/
        └── CameraModule.cpp
```

---

# 105. Driver vs Module

Es importante no confundirlos.

## Driver

Habla con el hardware.

```text
W5500Driver
MCP23017Driver
ST7789Driver
```

## Module

Utiliza el driver y lo integra al sistema.

```text
EthernetModule
IOExpanderModule
DisplayModule
```

Arquitectura:

```text
Application
     ↓
Module
     ↓
Service
     ↓
Driver
     ↓
Hardware
```

---

# 106. Ejemplo completo: W5500

```text
W5500
 ↓
W5500Driver
 ↓
EthernetHAL
 ↓
EthernetModule
 ↓
NetworkService
 ↓
System Bus
 ↓
API / Central / Integrations
```

---

# 107. Ejemplo completo: MCP23017

```text
MCP23017
 ↓
I2C Driver
 ↓
GPIO HAL
 ↓
IOExpanderModule
 ↓
ResourceManager
 ↓
Capability
 ↓
Entity
 ↓
Automation
```

---

# 108. Ejemplo completo: sensor

```text
BME280
 ↓
BME280Driver
 ↓
SensorHAL
 ↓
Temperature/Humidity/Pressure Module
 ↓
Capabilities
 ↓
Entities
 ↓
History
 ↓
Automation / API
```

---

# 109. Ejemplo completo: cámara + AI

```text
Camera
 ↓
CameraDriver
 ↓
CameraModule
 ↓
Frame
 ↓
AI Module
 ↓
Inference
 ↓
Entity/Event
 ↓
Automation
```

---

# 110. Reutilización

Los módulos deben diseñarse para funcionar en diferentes dispositivos.

Ejemplo:

```text
TemperatureModule
```

puede utilizarse en:

```text
Invernadero
SEMA
Casa
Oficina
Industria
Servidor central
```

sin modificar su lógica principal.

---

# 111. Configuración específica del proyecto

El módulo debe evitar conocer:

```text
Invernadero
SEMA
Casa
Oficina
```

Debe conocer conceptos genéricos:

```text
Zone
Entity
Sensor
Capability
Automation
```

---

# 112. Módulos compartidos

Los módulos genéricos deben poder reutilizarse entre proyectos.

Ejemplo:

```text
Temperature
Humidity
Pressure
Network
Ethernet
RS485
CAN
MCP23017
Display
Touch
Storage
Watchdog
NTP
```

Esto permite construir posteriormente una librería común de plataforma.

---

# 113. Módulos específicos

Los módulos que contienen lógica específica pueden pertenecer a un dominio.

Ejemplo:

```text
Agriculture.Irrigation
Agriculture.Greenhouse
Marine.Bilge
Industrial.Machine
Home.Security
Energy.LoadManagement
```

Pero deben utilizar interfaces comunes.

---

# 114. Dominio vs plataforma

La plataforma proporciona:

```text
Device Model
System Bus
API
Configuration
Security
Storage
Networking
Automation Engine
```

Los módulos de dominio proporcionan:

```text
Irrigation
HVAC
Lighting
Security
Energy
Marine
Agriculture
```

---

# 115. Principio de extensibilidad

Agregar una nueva funcionalidad debería parecerse a:

```text
Nuevo módulo
      ↓
Declare dependencies
      ↓
Implement interfaces
      ↓
Register
      ↓
Expose capabilities
      ↓
Expose entities
      ↓
Add UI
      ↓
Add tests
```

y no:

```text
Modificar 30 partes del firmware
```

---

# 116. Checklist de desarrollo

Antes de considerar terminado un módulo:

## Arquitectura

* [ ] Tiene responsabilidad claramente definida.
* [ ] No duplica otro módulo.
* [ ] Está desacoplado del hardware cuando corresponde.
* [ ] Utiliza HAL/Driver correctamente.

## Hardware

* [ ] Declara requisitos.
* [ ] Declara recursos.
* [ ] Detecta conflictos.
* [ ] Respeta Build Profiles.

## Configuración

* [ ] Tiene configuración versionada.
* [ ] Puede validarse.
* [ ] Soporta migraciones si es necesario.
* [ ] Diferencia desired/applied configuration.

## Device Model

* [ ] Registra capabilities.
* [ ] Registra resources.
* [ ] Crea entities estables.
* [ ] Genera events cuando corresponde.

## Comunicación

* [ ] Utiliza System Bus cuando corresponde.
* [ ] No depende innecesariamente de Central.
* [ ] Funciona sin Internet cuando debe hacerlo.

## FreeRTOS

* [ ] No crea tareas innecesarias.
* [ ] Utiliza scheduler/timers existentes.
* [ ] Tiene timeouts.
* [ ] No bloquea tareas críticas.

## Seguridad

* [ ] Respeta permisos.
* [ ] No expone secretos.
* [ ] Valida comandos.

## UI

* [ ] Tiene configuración web si corresponde.
* [ ] Utiliza componentes modulares.
* [ ] Oculta opciones incompatibles.

## API

* [ ] Respeta `/api/v1`.
* [ ] Utiliza modelos estándar.
* [ ] Implementa errores estructurados.

## Diagnóstico

* [ ] Tiene health state.
* [ ] Registra errores.
* [ ] Proporciona diagnóstico.
* [ ] Puede recuperarse cuando sea posible.

## Testing

* [ ] Compila en perfiles compatibles.
* [ ] Tiene pruebas de configuración.
* [ ] Tiene pruebas de error.
* [ ] Tiene pruebas de recuperación.

---

# 117. Flujo oficial de desarrollo

El desarrollo de un nuevo módulo debe seguir:

```text
1. Definir propósito
       ↓
2. Definir capabilities
       ↓
3. Definir entities
       ↓
4. Definir resources
       ↓
5. Definir dependencias
       ↓
6. Definir Build Profiles compatibles
       ↓
7. Implementar Driver/HAL si corresponde
       ↓
8. Implementar Module
       ↓
9. Implementar configuración
       ↓
10. Registrar entities
       ↓
11. Registrar events/commands
       ↓
12. Implementar API
       ↓
13. Implementar UI
       ↓
14. Implementar diagnostics
       ↓
15. Implementar tests
       ↓
16. Actualizar documentación
```

---

# 118. Regla de oro

> **Un módulo debe poder reemplazar su implementación física sin obligar a cambiar la lógica que lo consume.**

Ejemplo:

```text
DS18B20
   ↓
Temperature Module
```

puede reemplazarse por:

```text
BME280
   ↓
Temperature Module
```

y la aplicación continúa utilizando:

```text
sensor.room.temperature
```

---

# 119. Regla de independencia

La dependencia ideal es:

```text
Application
    ↓
Entity / Capability
    ↓
Module
    ↓
HAL
    ↓
Driver
    ↓
Hardware
```

Nunca:

```text
Application
    ↓
GPIO17
```

ni:

```text
Automation
    ↓
MCP23017 GPIO3
```

ni:

```text
API
    ↓
Modbus Register 40001
```

El hardware debe permanecer oculto detrás de las abstracciones.

---

# 120. Principio final

La plataforma debe poder evolucionar de:

```text
ESP32 + algunos sensores
```

a:

```text
ESP32-WROOM
ESP32-S3
ESP32-C3
ESP32-C5
ESP32-C6
ESP32-H2
ESP32-P4
```

con:

```text
Ethernet
Wi-Fi
Thread
Zigbee
Matter
CAN
RS485
Modbus
I²C
SPI
USB
PSRAM
Cámara
AI
Displays
Touch
Expansores
Sensores
Actuadores
```

sin modificar la arquitectura lógica central.

La arquitectura definitiva es:

```text
                    BUILD PROFILE
                         │
                         ▼
                     HARDWARE
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
            ┌────────────┼────────────┐
            ▼            ▼            ▼
        RESOURCE     CAPABILITY     STATE
            │            │
            └──────┬─────┘
                   ▼
                 ENTITY
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
       COMMAND   EVENT    HISTORY
          │        │
          └────────┼────────┘
                   ▼
              SYSTEM BUS
                   │
       ┌───────────┼───────────┐
       ▼           ▼           ▼
   AUTOMATION    CENTRAL      API
       │                       │
       ▼                       ▼
   FUNCTIONS              INTEGRATIONS
```

> **Los módulos son las unidades de extensión de la plataforma.**
>
> **Los Build Profiles determinan qué puede compilarse.**
>
> **Los Drivers conocen el hardware.**
>
> **Los módulos integran ese hardware al Device Model.**
>
> **Las Entities representan capacidades independientemente del hardware.**
>
> **El System Bus permite que los módulos se comuniquen sin acoplarse entre sí.**
>
> **La aplicación debe trabajar con capacidades, entidades, comandos y eventos, no con GPIO, chips o buses físicos.**

---

# 121. Principio arquitectónico definitivo

> **Agregar hardware debe significar agregar o adaptar un Driver.**
>
> **Agregar una función debe significar agregar un Module.**
>
> **Agregar una capacidad debe significar agregar una Capability.**
>
> **Agregar una integración debe significar agregar un Integration Module.**
>
> **Agregar una interfaz debe significar agregar componentes UI.**
>
> **Ninguno de estos cambios debería obligar a modificar innecesariamente el núcleo de la plataforma.**

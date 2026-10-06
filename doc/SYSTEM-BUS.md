# SYSTEM-BUS.md

# System Bus — Bus de Comunicación del Sistema

> **Tipo:** Arquitectura / Convención
> **Estado:** Planificación
> **Versión:** 1.0.0
> **Fecha:** 2026-10-06

---

## 1. Propósito

El **System Bus** es la capa lógica de comunicación interna de la plataforma de automatización distribuida.

Su objetivo es permitir que:

* módulos;
* dispositivos;
* nodos;
* controladores de zona;
* Central;
* sensores;
* actuadores;
* servicios;
* automatizaciones;
* escenas;
* integraciones;

puedan comunicarse entre sí **sin depender directamente del medio físico utilizado para transportar los mensajes**.

La aplicación no debe necesitar saber si un mensaje viaja mediante:

* memoria interna;
* FreeRTOS Queue;
* Wi-Fi;
* Ethernet;
* CAN;
* RS485;
* Modbus RTU;
* Thread;
* Matter;
* otro protocolo futuro.

La aplicación trabaja con una abstracción común:

```text
Aplicación
    ↓
System Bus API
    ↓
System Bus Core
    ↓
Transport Adapter
    ↓
Transporte físico/protocolo
```

---

# 2. Objetivo arquitectónico

El principio fundamental es:

> **La aplicación publica intenciones y consume eventos; el transporte es una implementación intercambiable.**

Por lo tanto:

```text
                  SYSTEM BUS
                      │
        ┌─────────────┼─────────────┐
        │             │             │
      Local         Network       Field Bus
        │             │             │
   FreeRTOS       Ethernet/WiFi   CAN/RS485
        │             │             │
        └─────────────┼─────────────┘
                      │
                  Device Model
```

El cambio de transporte no debe obligar a modificar la lógica de negocio.

---

# 3. Relación con la arquitectura general

El System Bus conecta las diferentes capas de la plataforma:

```text
┌──────────────────────────────────────────────┐
│                  Aplicación                  │
│                                              │
│ Escenas / Automatizaciones / Funciones       │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│               Device Model                   │
│                                              │
│ Entity / Capability / State / Command/Event  │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│                SYSTEM BUS                    │
│                                              │
│ Routing / QoS / ACK / Retry / Correlation    │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│             Transport Adapter               │
├────────┬────────┬────────┬────────┬──────────┤
│ Local  │ Wi-Fi  │Ethernet│  CAN   │  RS485  │
└────────┴────────┴────────┴────────┴──────────┘
```

---

# 4. Principios fundamentales

El System Bus deberá cumplir los siguientes principios.

## 4.1 Independencia del transporte

La aplicación nunca debe depender directamente de:

```text
GPIO
CAN ID
Modbus Register
IP address
MAC address
SPI CS
I²C address
UART
```

Estos pertenecen a las capas inferiores.

---

## 4.2 Identidad lógica

La comunicación debe utilizar identificadores lógicos:

```text
site_id
zone_id
device_id
entity_id
function_id
group_id
```

Por ejemplo:

```text
light.living_room
sensor.living_room.temperature
switch.pool.pump
cover.garage
```

No:

```text
GPIO23
CAN_ID=0x125
MODBUS_REGISTER=40001
```

---

## 4.3 Local-first

Una comunicación local no debe depender de Internet.

```text
Internet
   ↓
Cloud
   ↓
Central
   ↓
Zone
   ↓
Node
   ↓
Device
```

Las funciones críticas deberán ejecutarse en el nivel más bajo posible.

---

## 4.4 Autonomía distribuida

Se mantiene la jerarquía:

```text
DEVICE AUTONOMY
       >
ZONE AUTONOMY
       >
CENTRAL
       >
INTERNET
       >
CLOUD
```

El System Bus debe permitir que un nodo continúe funcionando aunque:

* Central esté apagada;
* Internet no exista;
* otro nodo no responda;
* una integración externa esté desconectada.

---

## 4.5 Mensajes explícitos

Toda comunicación deberá tener una semántica definida.

Tipos principales:

```text
COMMAND
EVENT
STATE
TELEMETRY
DISCOVERY
RESPONSE
ACK
ERROR
```

---

## 4.6 Idempotencia

Los comandos que puedan repetirse deberán diseñarse para evitar efectos duplicados.

Ejemplo:

```text
command_id = 01J...
idempotency_key = abc123
```

Si el mismo comando llega dos veces debido a un retry, el dispositivo deberá poder determinar que se trata de la misma operación.

---

# 5. Arquitectura del System Bus

La implementación recomendada es:

```text
┌───────────────────────────────┐
│          Application          │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│       System Bus API          │
│                               │
│ publish()                     │
│ subscribe()                  │
│ request()                     │
│ sendCommand()                 │
│ emitEvent()                   │
│ publishState()                │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│        System Bus Core        │
│                               │
│ Routing                       │
│ Filtering                     │
│ QoS                           │
│ Retry                         │
│ Timeout                       │
│ Deduplication                 │
│ Correlation                   │
│ Security                      │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│      Transport Manager        │
└───────────────┬───────────────┘
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
     Local    Network   Field
              Bus       Bus
```

---

# 6. Capas

## 6.1 System Bus API

Es la interfaz utilizada por módulos y servicios.

Ejemplo conceptual:

```cpp
bus.publish(event);
bus.send(command);
bus.subscribe(filter, callback);
bus.request(request);
```

El módulo no debe conocer el transporte.

---

## 6.2 System Bus Core

Gestiona:

* routing;
* prioridades;
* colas;
* timeout;
* retry;
* ACK;
* correlación;
* deduplicación;
* suscripciones;
* filtrado;
* seguridad;
* versionado;
* métricas.

---

## 6.3 Transport Manager

Selecciona el transporte apropiado.

Ejemplo:

```text
Destination
     ↓
Transport Resolver
     ↓
┌───────────────┐
│ Ethernet      │
│ Wi-Fi         │
│ CAN           │
│ RS485         │
│ Thread        │
│ Matter        │
│ Local         │
└───────────────┘
```

---

# 7. Transport Adapters

Cada transporte se implementará mediante un adaptador.

Ejemplo:

```text
TransportAdapter
│
├── LocalTransport
├── WifiTransport
├── EthernetTransport
├── CanTransport
├── Rs485Transport
├── ThreadTransport
├── MatterTransport
└── CustomTransport
```

La interfaz conceptual puede ser:

```cpp
class TransportAdapter {
public:
    virtual bool begin() = 0;

    virtual bool send(
        const BusMessage& message
    ) = 0;

    virtual bool receive(
        BusMessage& message
    ) = 0;

    virtual bool available() = 0;

    virtual bool connected() = 0;

    virtual void update() = 0;
};
```

La implementación real podrá adaptarse al modelo FreeRTOS utilizado.

---

# 8. Transporte local

Dentro de un mismo ESP32 no es necesario utilizar una red.

Los módulos pueden comunicarse mediante:

```text
FreeRTOS Queue
Event Group
Task Notification
Ring Buffer
Shared State
```

Por ejemplo:

```text
SensorTask
    ↓
System Bus
    ↓
AutomationTask
```

Esto permite utilizar exactamente la misma arquitectura lógica para comunicación interna y externa.

---

# 9. Comunicación entre dispositivos

Cuando el destino está en otro dispositivo:

```text
Node A
  │
  ▼
System Bus
  │
  ▼
Transport Adapter
  │
  ▼
Ethernet / Wi-Fi / CAN / RS485
  │
  ▼
Transport Adapter
  │
  ▼
System Bus
  │
  ▼
Node B
```

El mensaje mantiene su identidad lógica.

---

# 10. Tipos de mensajes

## 10.1 COMMAND

Representa una intención de modificar algo.

Ejemplo:

```json
{
  "message_type": "command",
  "entity_id": "light.living_room",
  "command": "turn_on"
}
```

Otros ejemplos:

```text
set_brightness
set_temperature
set_position
open
close
lock
unlock
start
stop
set_speed
set_mode
```

---

# 11. EVENT

Representa algo que ocurrió.

Ejemplo:

```json
{
  "message_type": "event",
  "event_type": "motion_detected",
  "entity_id": "binary_sensor.hall_motion"
}
```

Los eventos no representan necesariamente el estado actual.

Por ejemplo:

```text
EVENT:
motion_detected
```

no significa:

```text
motion = true
```

La diferencia debe mantenerse.

---

# 12. STATE

Representa el estado conocido de una entidad.

Ejemplo:

```json
{
  "message_type": "state",
  "entity_id": "light.living_room",
  "state": {
    "power": true,
    "brightness": 80
  }
}
```

El estado debe incluir, cuando corresponda:

```text
timestamp
source
quality
confidence
version
```

---

# 13. TELEMETRY

La telemetría transporta mediciones periódicas.

Ejemplo:

```json
{
  "message_type": "telemetry",
  "entity_id": "sensor.greenhouse.temperature",
  "value": 24.7,
  "unit": "°C"
}
```

Puede utilizarse para:

* temperatura;
* humedad;
* presión;
* corriente;
* tensión;
* potencia;
* energía;
* flujo;
* velocidad;
* posición;
* calidad del aire;
* etc.

---

# 14. DISCOVERY

Se utiliza para descubrir dispositivos, recursos, capacidades y entidades.

Ejemplo:

```text
Central
   ↓
DISCOVERY_REQUEST
   ↓
Node
   ↓
DISCOVERY_RESPONSE
```

La respuesta puede informar:

```text
device_id
hardware_profile
firmware_version
resources
capabilities
entities
supported_transports
```

---

# 15. ACK / RESPONSE

Los mensajes que requieren confirmación pueden utilizar:

```text
REQUEST
   ↓
ACK
   ↓
RESPONSE
```

Por ejemplo:

```text
Central
   │
   │ COMMAND
   ▼
Node
   │
   │ ACK
   ▼
Central
   │
   │ RESPONSE
   ▼
Central
```

El ACK confirma recepción.

El RESPONSE confirma el resultado de la operación.

No deben confundirse.

---

# 16. Envelope del mensaje

Todos los mensajes deberán utilizar una envoltura común.

Modelo conceptual:

```json
{
  "message_id": "01J...",
  "message_type": "command",
  "schema_version": "1.0",
  "timestamp": "2026-10-06T12:00:00Z",

  "source": {
    "site_id": "site-001",
    "zone_id": "zone-living",
    "device_id": "node-001"
  },

  "destination": {
    "device_id": "node-005",
    "entity_id": "light.living_room"
  },

  "request_id": "01J...",
  "correlation_id": "01J...",

  "priority": "normal",
  "qos": "reliable",
  "ttl": 10,

  "payload": {}
}
```

---

# 17. Campos principales

## `message_id`

Identificador único del mensaje.

Debe permitir:

* deduplicación;
* trazabilidad;
* diagnóstico;
* correlación.

---

## `message_type`

Ejemplos:

```text
command
event
state
telemetry
discovery
response
ack
error
```

---

## `schema_version`

Versión del formato del mensaje.

Ejemplo:

```text
1.0
1.1
2.0
```

---

## `timestamp`

Marca temporal de creación del mensaje.

Cuando el dispositivo no tenga hora válida deberá poder utilizar:

```text
monotonic timestamp
sequence number
boot_id
```

hasta que NTP o una fuente de tiempo válida esté disponible.

---

## `source`

Identifica el origen lógico.

---

## `destination`

Identifica el destino.

Puede apuntar a:

```text
device
zone
group
entity
function
central
broadcast
```

---

## `request_id`

Identifica una solicitud lógica.

---

## `correlation_id`

Permite asociar múltiples mensajes con una misma operación.

Ejemplo:

```text
COMMAND
   ↓
ACK
   ↓
STATE
   ↓
EVENT
```

Todos pueden compartir:

```text
correlation_id
```

---

# 18. TTL

Los mensajes distribuidos deberán poder incluir:

```text
ttl
```

para evitar propagación infinita.

Ejemplo:

```text
TTL = 8
```

Cada salto puede reducirlo:

```text
8 → 7 → 6 → 5 ...
```

Cuando llegue a:

```text
0
```

el mensaje deberá descartarse.

---

# 19. Prioridades

Se recomienda utilizar prioridades lógicas.

```text
CRITICAL
HIGH
NORMAL
LOW
BACKGROUND
```

Ejemplo:

| Prioridad  | Ejemplo        |
| ---------- | -------------- |
| CRITICAL   | emergencia     |
| HIGH       | seguridad      |
| NORMAL     | automatización |
| LOW        | telemetría     |
| BACKGROUND | diagnóstico    |

Las prioridades deben influir en las colas y planificación, no necesariamente en el protocolo físico.

---

# 20. QoS

El System Bus deberá soportar diferentes niveles de confiabilidad.

Propuesta:

```text
BEST_EFFORT
AT_LEAST_ONCE
RELIABLE
```

## BEST_EFFORT

Adecuado para:

* telemetría;
* valores periódicos;
* información no crítica.

## AT_LEAST_ONCE

Adecuado para:

* eventos;
* comandos donde exista idempotencia.

## RELIABLE

Adecuado para:

* configuración;
* provisioning;
* operaciones críticas.

La disponibilidad real dependerá del transporte.

---

# 21. Routing

El System Bus deberá soportar diferentes destinos.

## Unicast

```text
Node A → Node B
```

## Multicast

```text
Node A → Grupo
```

## Broadcast

```text
Node A → Todos
```

## Zonecast

```text
Central → Zone 1
```

## Entity routing

```text
light.living_room
```

El router resolverá qué dispositivo contiene dicha entidad.

---

# 22. Direccionamiento lógico

La aplicación puede trabajar con:

```text
site_id
zone_id
device_id
group_id
entity_id
```

Ejemplo:

```text
site: home-001
zone: living-room
device: node-024
entity: light.living_room
```

La resolución hacia:

```text
IP
MAC
CAN ID
RS485 address
Modbus address
Thread address
Matter node
```

queda a cargo de las capas inferiores.

---

# 23. Separación entre dirección lógica y física

Esta separación es fundamental.

```text
ENTITY
   ↓
DEVICE
   ↓
ROUTING
   ↓
TRANSPORT
   ↓
PHYSICAL ADDRESS
```

Nunca:

```text
ENTITY
   ↓
GPIO
```

directamente.

---

# 24. Command Lifecycle

Todo comando distribuido deberá poder seguir un ciclo de vida.

```text
REQUESTED
    ↓
AUTHORIZED
    ↓
ACCEPTED
    ↓
QUEUED
    ↓
EXECUTING
    ↓
EXECUTED
```

También puede finalizar como:

```text
REJECTED
FAILED
TIMEOUT
CANCELLED
```

Ejemplo:

```text
POST command
      ↓
command_id
      ↓
ACCEPTED
      ↓
Node receives
      ↓
EXECUTING
      ↓
ACTUAL STATE
      ↓
EXECUTED
```

---

# 25. Command vs State

Un comando expresa:

> “Haz esto.”

El estado expresa:

> “Esto es lo que está ocurriendo.”

Ejemplo:

```text
COMMAND
light.living_room → turn_on
```

No implica necesariamente:

```text
STATE
power = true
```

hasta que el dispositivo confirme el resultado.

Esto permite distinguir:

```text
desired state
actual state
```

---

# 26. Desired State

Los actuadores podrán utilizar:

```json
{
  "desired": {
    "power": true,
    "brightness": 75
  },
  "actual": {
    "power": false,
    "brightness": 0
  }
}
```

Esto permite detectar:

```text
desired != actual
```

y determinar que existe una condición pendiente o un fallo.

---

# 27. Events

Los eventos representan cambios o hechos.

Ejemplos:

```text
motion_detected
door_opened
button_pressed
device_connected
device_disconnected
alarm_triggered
temperature_threshold_exceeded
configuration_changed
```

Un evento puede disparar:

```text
automation
scene
notification
logging
integration
```

---

# 28. Event-driven architecture

El sistema debe favorecer comunicación orientada a eventos.

Ejemplo:

```text
PIR
 ↓
motion_detected
 ↓
System Bus
 ↓
Automation
 ↓
COMMAND
 ↓
Light
```

En lugar de:

```text
PIR → llamar directamente a Light
```

Esto reduce el acoplamiento.

---

# 29. Suscripciones

Los módulos pueden suscribirse a:

```text
message_type
entity_id
event_type
zone_id
group_id
capability
priority
```

Ejemplo:

```text
subscribe(
    event_type = "motion_detected",
    zone = "hall"
)
```

---

# 30. Filtros

Los filtros permiten reducir tráfico.

Ejemplo:

```text
zone = living
message_type = event
event_type = motion_detected
```

También:

```text
entity = sensor.temperature
```

---

# 31. Store-and-forward

Los nodos podrán almacenar temporalmente mensajes cuando el transporte esté desconectado.

Ejemplo:

```text
Node
 ↓
No connection
 ↓
Local queue
 ↓
Connection restored
 ↓
Transmit pending messages
```

No todos los mensajes deben almacenarse.

Debe definirse por:

```text
QoS
priority
message_type
TTL
```

---

# 32. Offline behavior

Si Central está desconectada:

```text
Node
 ↓
Local System Bus
 ↓
Local automation
 ↓
Actuator
```

debe continuar funcionando.

La pérdida de Central no debe bloquear:

* iluminación local;
* climatización crítica;
* seguridad;
* sensores;
* actuadores;
* automatizaciones locales.

---

# 33. Central como coordinador

Central puede recibir:

```text
state
event
telemetry
diagnostics
```

y enviar:

```text
commands
configuration
scene activation
automation configuration
```

Pero no debe ser una dependencia obligatoria para cada operación local.

---

# 34. Comunicación entre Zone Controllers

Ejemplo:

```text
Zone A
   │
   │
   ▼
Central
   │
   ▼
Zone B
```

También deberá ser posible, cuando la arquitectura lo permita:

```text
Zone A
   │
   └──────► Zone B
```

sin depender obligatoriamente de Central.

Esto permite implementar automatizaciones distribuidas.

---

# 35. Transportes soportados

El diseño debe contemplar:

| Transporte | Uso                      |
| ---------- | ------------------------ |
| Local      | comunicación interna     |
| Wi-Fi      | nodos inalámbricos       |
| Ethernet   | backbone                 |
| CAN        | automatización robusta   |
| RS485      | campo/industrial         |
| Modbus RTU | dispositivos compatibles |
| Thread     | dispositivos IoT         |
| Matter     | interoperabilidad        |
| Futuro     | nuevos transportes       |

---

# 36. Ethernet

Ethernet puede utilizar:

```text
ESP32 + LAN8720A/IP101
```

o:

```text
ESP32-S3 + W5500
```

La aplicación deberá ver ambos simplemente como:

```text
EthernetTransport
```

No debe existir lógica de negocio diferente por el PHY/controlador Ethernet.

---

# 37. Wi-Fi

Wi-Fi se utilizará principalmente para:

* nodos inalámbricos;
* configuración;
* API;
* WebSocket;
* comunicación distribuida;
* integración.

El System Bus no debe asumir que Wi-Fi siempre está disponible.

---

# 38. CAN

CAN puede utilizarse para:

* automatización distribuida;
* industria ligera;
* vehículos;
* aplicaciones marinas;
* largas distancias dentro de una instalación;
* comunicación robusta entre controladores.

El System Bus deberá abstraer el CAN ID.

Ejemplo:

```text
Entity
   ↓
System Bus
   ↓
CAN Transport
   ↓
CAN ID
```

---

# 39. RS485

RS485 es una capa física.

El protocolo superior puede ser:

```text
Modbus RTU
```

u otro protocolo.

Por lo tanto:

```text
RS485 != Modbus
```

El System Bus deberá mantener esta separación.

Ejemplo:

```text
System Bus
    ↓
RS485 Transport
    ↓
Modbus Adapter
    ↓
RS485
```

---

# 40. Thread y Matter

Thread y Matter podrán utilizarse como transportes o integraciones según el contexto.

Debe mantenerse la separación:

```text
System Bus
    ↓
Matter Adapter
    ↓
Matter
```

La plataforma no debe convertir internamente todas sus entidades en entidades Matter.

Matter es una representación externa/interop.

---

# 41. Integraciones

El System Bus también puede servir como fuente interna para:

```text
MQTT
REST API
WebSocket
Matter
Home Assistant
SmartThings
Homey
Apple Home
Google Home
Alexa
```

Arquitectura:

```text
                 Device Model
                      │
                 System Bus
                      │
        ┌─────────────┼──────────────┐
        │             │              │
       API           MQTT          Matter
        │             │              │
     External      External       External
```

---

# 42. MQTT

MQTT no debe ser el System Bus interno obligatorio.

Puede funcionar como:

```text
System Bus
    ↓
MQTT Adapter
    ↓
MQTT Broker
```

Esto permite utilizar MQTT para integraciones externas sin acoplar toda la arquitectura a él.

---

# 43. Seguridad

Los mensajes distribuidos deberán poder autenticarse.

Dependiendo del transporte se podrá utilizar:

```text
TLS
DTLS
CAN authentication
application-level authentication
Matter security
device certificates
keys
tokens
```

La seguridad deberá dividirse en:

```text
Authentication
Authorization
Integrity
Confidentiality
Replay Protection
```

---

# 44. Autorización

No todos los nodos podrán ejecutar todos los comandos.

Ejemplo:

```text
User
 ↓
Central
 ↓
Authorization
 ↓
System Bus
 ↓
Node
```

Un nodo también puede aplicar reglas locales.

Por ejemplo:

```text
Usuario → abrir válvula industrial
```

puede ser rechazado aunque el mensaje haya sido autenticado.

---

# 45. Protección contra replay

Los mensajes sensibles deberán utilizar mecanismos como:

```text
message_id
timestamp
nonce
sequence
expiration
```

para impedir reutilizar mensajes antiguos.

---

# 46. Deduplicación

Cada receptor deberá poder detectar mensajes duplicados cuando el QoS lo requiera.

Ejemplo:

```text
message_id = ABC

receive ABC
execute

receive ABC again
detect duplicate
do not execute twice
```

El período de retención de IDs dependerá de:

```text
RAM
Flash
QoS
criticidad
```

---

# 47. Fragmentación

El System Bus no deberá asumir un tamaño de paquete ilimitado.

Los transportes tienen diferentes límites.

Por ejemplo:

```text
CAN
RS485
UDP
TCP
Matter
Wi-Fi
```

pueden tener diferentes restricciones.

El System Bus deberá proporcionar:

```text
message size
fragmentation
reassembly
maximum payload
```

cuando el transporte lo requiera.

Los mensajes grandes deberán evitarse siempre que sea posible.

---

# 48. Configuración

La configuración del System Bus deberá poder definir:

```text
transport
priority
QoS
timeouts
retry
routing
subscriptions
security
buffer sizes
```

Los valores críticos deberán estar protegidos contra configuraciones incompatibles.

---

# 49. Hardware Profile

El System Bus puede depender de las capacidades declaradas por el Hardware Profile.

Ejemplo:

```text
NODE_ETH_WROOM_LAN8720_REV_A
```

puede ofrecer:

```text
Ethernet
Wi-Fi
CAN
UART
I2C
SPI
```

Mientras:

```text
NODE_BASIC_C3_REV_A
```

puede ofrecer:

```text
Wi-Fi
UART
I2C
SPI
```

La aplicación utilizará únicamente los transportes disponibles.

---

# 50. Build Profile

El Build Profile determina qué implementación se compila.

Ejemplo:

```ini
build_flags =
    -D BOARD_ESP32_WROOM
    -D ETH_LAN8720
```

o:

```ini
build_flags =
    -D BOARD_ESP32_S3
    -D ETH_W5500
```

El System Bus no deberá tener que duplicar la lógica de aplicación para cada board.

---

# 51. FreeRTOS

El System Bus deberá integrarse naturalmente con FreeRTOS.

Arquitectura conceptual:

```text
Task Sensor
     │
     ▼
Bus Queue
     │
     ▼
System Bus Task
     │
     ├── Local subscribers
     ├── Network transport
     └── Storage
```

Puede utilizar:

```text
Queue
RingBuffer
EventGroup
TaskNotification
Mutex
Semaphore
```

según la necesidad.

---

# 52. Tasks recomendadas

Una implementación puede separar:

```text
BusCoreTask
TransportTask
RoutingTask
PersistenceTask
DiagnosticsTask
```

Sin embargo, no se deberá crear una tarea por cada pequeña función sin justificación.

En ESP32 con recursos limitados se deberá priorizar:

```text
menos tareas
colas correctamente dimensionadas
event-driven
bloqueos mínimos
```

---

# 53. Task Pinning

En plataformas con múltiples núcleos podrá utilizarse:

```text
Core 0
├── Wi-Fi / networking
├── System services
└── Communication

Core 1
├── Automation
├── Sensors
└── Application
```

Pero el pinning no debe formar parte de la API lógica del System Bus.

Es una decisión de implementación.

---

# 54. Backpressure

El System Bus deberá controlar situaciones donde un productor genere mensajes más rápido que un consumidor.

Ejemplo:

```text
Sensor
 ↓
1000 msg/s
 ↓
Queue
 ↓
Consumer
 ↓
100 msg/s
```

Esto puede producir:

```text
queue overflow
RAM exhaustion
latency
```

Se deberán definir políticas:

```text
DROP_OLDEST
DROP_NEWEST
COALESCE
BLOCK
REJECT
PRIORITIZE
```

---

# 55. Coalescing

Para estados de alta frecuencia puede ser preferible mantener únicamente el valor más reciente.

Ejemplo:

```text
temperature:
24.1
24.2
24.3
24.4
24.5
```

En lugar de enviar todos los valores, puede enviarse:

```text
24.5
```

si los valores intermedios no son relevantes.

Esto es especialmente útil para:

* sensores;
* telemetría;
* posición;
* intensidad;
* temperatura;
* nivel.

---

# 56. Rate limiting

Los mensajes deberán poder limitarse por:

```text
device
entity
message_type
transport
integration
```

Esto evita que un sensor o módulo defectuoso sature el sistema.

---

# 57. Health y diagnóstico

El System Bus deberá generar métricas como:

```text
messages_sent
messages_received
messages_failed
messages_dropped
queue_usage
retry_count
timeouts
latency
transport_errors
duplicate_messages
```

Estas métricas estarán disponibles para:

```text
Central
Web UI
API
Diagnostics
Logs
```

---

# 58. Latencia

Deberán registrarse, cuando sea posible:

```text
created_at
sent_at
received_at
processed_at
completed_at
```

Esto permite calcular:

```text
transport latency
queue latency
processing latency
end-to-end latency
```

---

# 59. Trazabilidad

Una operación distribuida deberá poder seguirse mediante:

```text
request_id
correlation_id
message_id
command_id
```

Ejemplo:

```text
User
 ↓
API
 ↓ request_id
Central
 ↓ correlation_id
System Bus
 ↓ command_id
Node
 ↓
Actuator
 ↓
STATE
```

Esto simplifica enormemente el diagnóstico.

---

# 60. Error handling

Los errores deben tener formato uniforme.

Ejemplo:

```json
{
  "message_type": "error",
  "code": "ENTITY_UNAVAILABLE",
  "message": "Entity is currently unavailable",
  "request_id": "01J...",
  "correlation_id": "01J..."
}
```

Códigos posibles:

```text
INVALID_MESSAGE
INVALID_SCHEMA
UNAUTHORIZED
FORBIDDEN
UNKNOWN_ENTITY
ENTITY_UNAVAILABLE
DEVICE_OFFLINE
TRANSPORT_ERROR
TIMEOUT
QUEUE_FULL
INVALID_COMMAND
COMMAND_REJECTED
EXECUTION_FAILED
```

---

# 61. Discovery

Al conectarse un nodo:

```text
Node boot
   ↓
Network available
   ↓
System Bus discovery
   ↓
Announce
   ↓
Central / Zone Controller
   ↓
Request metadata
   ↓
Response
   ↓
Register
```

El nodo debe poder funcionar aunque el proceso de discovery no se complete.

---

# 62. Device Announce

Ejemplo conceptual:

```json
{
  "message_type": "discovery",
  "operation": "announce",
  "device_id": "node-001",
  "hardware_profile": "NODE_ETH_WROOM_LAN8720_REV_A",
  "firmware": "1.4.0"
}
```

---

# 63. State Synchronization

Cuando un dispositivo se reconecta:

```text
Node reconnect
     ↓
Discovery
     ↓
State synchronization
     ↓
Configuration reconciliation
     ↓
Normal operation
```

No se debe asumir que Central conoce necesariamente el último estado real.

El nodo es autoridad sobre sus recursos físicos.

---

# 64. Authority

La autoridad depende del tipo de dato.

Ejemplo:

```text
GPIO output actual
        ↓
Node
```

```text
Zone automation
        ↓
Zone Controller
```

```text
Global user configuration
        ↓
Central
```

```text
External weather data
        ↓
External provider
```

La autoridad deberá quedar definida en el modelo de datos.

---

# 65. Conflictos

Puede ocurrir:

```text
Central desired = ON
Node actual = OFF
```

o:

```text
Central configuration version = 10
Node configuration version = 12
```

El sistema deberá comparar:

```text
version
timestamp
authority
source
priority
```

antes de sobrescribir información.

---

# 66. Versionado

El System Bus tendrá su propio versionado.

Ejemplo:

```text
Bus Protocol Version
1.0
```

y los mensajes:

```text
schema_version = 1.0
```

El cambio de versión deberá ser compatible siempre que sea posible.

---

# 67. Compatibilidad

Se recomienda utilizar:

```text
backward compatibility
forward compatibility
capability negotiation
```

Un dispositivo puede anunciar:

```text
supported_schema_versions
supported_message_types
supported_qos
supported_transports
```

---

# 68. Capability Negotiation

Ejemplo:

```json
{
  "supported": {
    "commands": [
      "turn_on",
      "turn_off",
      "set_brightness"
    ],
    "qos": [
      "best_effort",
      "reliable"
    ]
  }
}
```

Esto evita asumir capacidades inexistentes.

---

# 69. Comunicación con API

La API externa y el System Bus deben utilizar el mismo modelo conceptual.

```text
REST API
   ↓
Command
   ↓
System Bus
   ↓
Node
```

No:

```text
REST API
   ↓
GPIO directamente
```

---

# 70. Comunicación con WebSocket

WebSocket puede utilizarse para transmitir:

```text
state updates
events
diagnostics
command status
```

Ejemplo:

```text
Browser
   │
   │ WebSocket
   ▼
Central
   │
   │ System Bus
   ▼
Node
```

---

# 71. Comunicación con MQTT

MQTT puede mapearse:

```text
System Bus Event
      ↓
MQTT Adapter
      ↓
topic
```

y:

```text
MQTT message
      ↓
MQTT Adapter
      ↓
System Bus Command
```

---

# 72. Automatizaciones

Las automatizaciones deben utilizar el System Bus.

Ejemplo:

```text
EVENT
motion_detected
       ↓
Condition
time > 22:00
       ↓
COMMAND
light.hall = 20%
```

No deberán existir conexiones directas rígidas entre módulos.

---

# 73. Escenas

Una escena puede generar múltiples comandos:

```text
Scene: Night
   │
   ├── Light 1 → OFF
   ├── Light 2 → 20%
   ├── Thermostat → 21°C
   └── Cover → CLOSE
```

Todos pasan por el System Bus.

---

# 74. Modos del sistema

El System Bus puede transportar cambios de modo:

```text
HOUSE.MODE = NORMAL
HOUSE.MODE = SLEEP
HOUSE.MODE = AWAY
HOUSE.MODE = VACATION
HOUSE.MODE = MAINTENANCE
HOUSE.MODE = EMERGENCY
```

Las automatizaciones pueden reaccionar ante estos eventos.

---

# 75. Ejemplo de modo SLEEP

```text
HOUSE.MODE = SLEEP
```

Puede producir:

```text
Bedroom PIR
    ↓
EVENT
    ↓
Log only
```

Mientras:

```text
Hall PIR
    ↓
EVENT
    ↓
Automation
    ↓
Light hall = 15%
```

Y:

```text
Exterior PIR
    ↓
EVENT
    ↓
Security automation
    ↓
Alarm
```

El System Bus permite que todos utilicen la misma infraestructura.

---

# 76. Seguridad crítica

Las funciones críticas deben tener prioridad sobre mensajes normales.

Ejemplo:

```text
CRITICAL
Emergency stop
     ↓
System Bus
     ↓
Actuator
```

No deberá quedar detrás de una cola llena de:

```text
telemetry
logs
diagnostics
```

---

# 77. Fallo del System Bus

El System Bus es infraestructura crítica.

Por ello, los módulos críticos deberán tener una estrategia de degradación.

Ejemplo:

```text
System Bus unavailable
        ↓
Local safety logic
        ↓
Safe state
```

Nunca deberá asumirse que una automatización de seguridad puede depender exclusivamente de una cola de comunicación no disponible.

---

# 78. Fail-safe

Los actuadores deberán definir:

```text
safe_state
startup_state
communication_loss_state
```

Ejemplo:

```text
Pump:
communication loss → OFF
```

o:

```text
Ventilation:
communication loss → 50%
```

según la aplicación.

Esto deberá configurarse según el dispositivo y la criticidad.

---

# 79. Watchdog

El System Bus podrá integrarse con:

```text
Task Watchdog
Hardware Watchdog
Transport Watchdog
Node Watchdog
```

Un bloqueo de comunicación no debe bloquear indefinidamente el sistema.

---

# 80. Persistencia

No todos los mensajes deben persistirse.

Se recomienda:

| Mensaje       | Persistencia     |
| ------------- | ---------------- |
| Telemetry     | opcional         |
| Event         | según criticidad |
| Command       | según QoS        |
| Configuration | sí               |
| Discovery     | no               |
| Diagnostics   | opcional         |
| Safety Event  | sí               |

---

# 81. Prioridad de almacenamiento

Cuando el almacenamiento sea limitado:

```text
Safety
   >
Configuration
   >
Important Events
   >
State
   >
Telemetry
   >
Diagnostics
```

---

# 82. Ejemplo completo

Supongamos:

```text
binary_sensor.front_door
```

detecta apertura.

Flujo:

```text
Sensor
   ↓
EVENT
door_opened
   ↓
System Bus
   ↓
Automation
   ↓
Check:
HOUSE.MODE == AWAY
   ↓
COMMAND
alarm.house → trigger
   ↓
System Bus
   ↓
Alarm Node
   ↓
STATE
alarm = triggered
   ↓
Event
alarm_triggered
   ↓
Central
   ↓
Notification
```

Ningún módulo necesita conocer el GPIO del sensor ni el relé de la alarma.

---

# 83. Ejemplo con diferentes transportes

```text
Door Sensor
ESP32-C3
   │
   │ Wi-Fi
   ▼
Central
   │
   │ Ethernet
   ▼
Zone Controller
   │
   │ CAN
   ▼
Alarm Node
```

La lógica sigue siendo:

```text
EVENT
   ↓
COMMAND
   ↓
STATE
```

El transporte es transparente.

---

# 84. Ejemplo RS485

```text
Industrial Sensor
      │
      │ Modbus RTU
      ▼
RS485 Node
      │
      │ System Bus
      ▼
Zone Controller
      │
      ▼
Automation
```

La automatización no necesita conocer:

```text
slave_id
register
baudrate
parity
```

Esos datos pertenecen al driver/adapter.

---

# 85. Ejemplo CAN

```text
Temperature Node
      │
      │ CAN
      ▼
Zone Controller
      │
      ▼
System Bus
      │
      ▼
Climate Function
```

La aplicación utiliza:

```text
sensor.zone.temperature
```

no:

```text
CAN_ID 0x123
```

---

# 86. Estructura recomendada del código

```text
src/
├── bus/
│   ├── SystemBus.hpp
│   ├── SystemBus.cpp
│   ├── BusMessage.hpp
│   ├── BusRouter.hpp
│   ├── BusRouter.cpp
│   ├── BusSubscription.hpp
│   ├── BusQoS.hpp
│   ├── BusPriority.hpp
│   ├── BusSecurity.hpp
│   └── transports/
│       ├── LocalTransport.hpp
│       ├── WifiTransport.hpp
│       ├── EthernetTransport.hpp
│       ├── CanTransport.hpp
│       ├── Rs485Transport.hpp
│       ├── ThreadTransport.hpp
│       └── MatterTransport.hpp
```

---

# 87. Separación del código

La aplicación:

```cpp
bus.send(command);
```

El transporte:

```cpp
transport.send(message);
```

El driver:

```cpp
ethernet.write(...);
```

La separación debe mantenerse.

---

# 88. Ejemplo de interfaz

Conceptualmente:

```cpp
class SystemBus {
public:

    bool begin();

    bool publish(
        const BusMessage& message
    );

    bool subscribe(
        const BusSubscription& subscription,
        BusCallback callback
    );

    bool request(
        const BusMessage& request,
        BusResponse& response
    );

    bool sendCommand(
        const Command& command
    );

    bool emitEvent(
        const Event& event
    );

    bool publishState(
        const EntityState& state
    );

    void update();
};
```

La interfaz definitiva podrá modificarse durante la implementación.

---

# 89. No bloquear la aplicación

Las operaciones de comunicación distribuida deberían ser preferentemente asíncronas.

Evitar:

```cpp
sendCommand();
waitForever();
```

Preferir:

```cpp
sendCommand();
```

y luego:

```text
COMMAND_STATUS
```

o:

```text
EVENT
STATE
```

---

# 90. Timeouts

Todo intercambio que espere respuesta deberá tener timeout.

Ejemplo:

```text
request timeout = 2 s
```

Después:

```text
retry
```

hasta:

```text
max_retries
```

Finalmente:

```text
TIMEOUT
```

---

# 91. Retries

Los retries dependerán del tipo de mensaje.

No se recomienda repetir indefinidamente.

Ejemplo:

```text
retry = 3
backoff = exponential
```

Conceptualmente:

```text
100 ms
200 ms
400 ms
```

---

# 92. Circuit Breaker

Para un nodo persistentemente inaccesible puede utilizarse:

```text
CLOSED
   ↓
FAILURES
   ↓
OPEN
   ↓
WAIT
   ↓
HALF_OPEN
   ↓
CLOSED
```

Esto evita inundar la red con retries.

---

# 93. Comunicación con dispositivos externos

El System Bus también puede servir para integrar:

```text
Modbus devices
CAN devices
MQTT devices
Matter devices
REST devices
```

mediante gateways/adapters.

---

# 94. Gateway

Un Gateway puede traducir:

```text
Protocol A
   ↓
Gateway
   ↓
System Bus
   ↓
Protocol B
```

Ejemplo:

```text
Modbus RTU
   ↓
RS485 Gateway
   ↓
System Bus
   ↓
MQTT
```

---

# 95. Regla para Gateways

Un Gateway no deberá mezclar innecesariamente:

```text
protocol conversion
application logic
automation logic
```

Su función principal es adaptar comunicaciones.

---

# 96. Topologías

El System Bus debe soportar:

## Estrella

```text
        Central
       /   |   \
     Node Node Node
```

## Árbol

```text
Central
   |
Zone A
 /   \
N1   N2
```

## Distribuida

```text
Node A ─ Node B
  │        │
Node C ─ Node D
```

## Híbrida

```text
Ethernet
   │
Zone
 ┌─┴─────┐
CAN    RS485
 │       │
Nodes   Sensors
```

---

# 97. Topología física vs lógica

La topología física no debe definir la lógica de la aplicación.

Por ejemplo:

```text
light.living_room
```

debe seguir siendo la misma entidad aunque cambie de:

```text
Wi-Fi
```

a:

```text
Ethernet
```

o:

```text
CAN
```

---

# 98. Configuración dinámica

El System Bus debe poder detectar cambios de configuración.

Ejemplo:

```text
Entity
light.garage
```

puede pasar de:

```text
Node A
```

a:

```text
Node B
```

sin cambiar necesariamente:

```text
entity_id
```

si la identidad lógica sigue representando la misma función.

---

# 99. Persistencia de identidad

La identidad lógica debe ser estable.

No debe depender de:

```text
IP
MAC
GPIO
CAN ID
Modbus address
```

Esto es especialmente importante cuando se reemplaza hardware.

---

# 100. Reemplazo de nodos

Ejemplo:

```text
Node A
   ↓
light.garage
```

Node A falla.

Se instala:

```text
Node B
```

y se reasigna:

```text
light.garage
```

La automatización:

```text
IF motion.garage
THEN light.garage ON
```

puede permanecer intacta.

---

# 101. System Bus y Device Model

La relación oficial es:

```text
Hardware
    ↓
Resource
    ↓
Capability
    ↓
Entity
    ↓
Device Model
    ↓
System Bus
    ↓
Communication
```

El System Bus no reemplaza al Device Model.

El Device Model define:

> qué existe.

El System Bus define:

> cómo se comunica.

---

# 102. System Bus y API

La API define:

> cómo clientes externos interactúan con el sistema.

El System Bus define:

> cómo componentes internos y nodos se comunican.

Ambos utilizan el mismo modelo lógico.

---

# 103. System Bus y MQTT

MQTT define:

> un mecanismo/protocolo de mensajería.

El System Bus define:

> la semántica interna de comunicación de la plataforma.

Por lo tanto:

```text
MQTT ≠ System Bus
```

MQTT puede ser un transporte/adaptador.

---

# 104. System Bus y CAN

CAN define un protocolo de comunicación de bajo nivel.

El System Bus proporciona:

```text
routing lógico
entity addressing
commands
events
state
QoS
correlation
```

Por lo tanto:

```text
CAN ≠ System Bus
```

---

# 105. System Bus y RS485

RS485 es principalmente una capa física.

Modbus RTU es un protocolo que puede funcionar sobre RS485.

El System Bus se sitúa por encima:

```text
System Bus
    ↓
Modbus Adapter
    ↓
RS485
```

---

# 106. Registro de transportes

Cada nodo deberá poder anunciar:

```json
{
  "transports": [
    {
      "type": "ethernet",
      "status": "connected"
    },
    {
      "type": "wifi",
      "status": "connected"
    },
    {
      "type": "can",
      "status": "available"
    }
  ]
}
```

---

# 107. Selección de ruta

Cuando existan varios caminos:

```text
Node A
 ├── Wi-Fi ────── Central
 └── Ethernet ─── Central
```

el sistema podrá seleccionar según:

```text
availability
priority
latency
reliability
cost
configuration
```

Por ejemplo:

```text
Ethernet > Wi-Fi
```

para el backbone cuando ambos estén disponibles.

---

# 108. Redundancia

En instalaciones avanzadas podrá existir:

```text
Primary Transport
Secondary Transport
```

Ejemplo:

```text
Ethernet
   ↓
Primary

Wi-Fi
   ↓
Backup
```

El cambio deberá ser transparente para la aplicación.

---

# 109. Failover

Ejemplo:

```text
Ethernet DOWN
      ↓
Transport Manager
      ↓
Wi-Fi
      ↓
System Bus continues
```

Las funciones no deberían tener que reiniciarse.

---

# 110. Heartbeat

Los nodos podrán utilizar:

```text
heartbeat
```

para indicar disponibilidad.

Ejemplo:

```text
Node → heartbeat → Central
```

La ausencia de heartbeat durante un intervalo configurable puede producir:

```text
DEVICE_UNAVAILABLE
```

---

# 111. Last Will / Offline Event

Los transportes que lo soporten podrán informar desconexión.

Pero la plataforma no debe depender exclusivamente de un mecanismo específico.

También deberá existir:

```text
timeout
heartbeat
health check
```

---

# 112. Calidad de información

Los mensajes de estado podrán incluir:

```text
quality:
  good
  uncertain
  stale
  invalid
  unavailable
```

Esto es especialmente importante en sensores.

---

# 113. Provenance

El System Bus deberá poder transportar información de origen.

Ejemplo:

```json
{
  "source": "sensor.living.temperature",
  "origin_device": "node-003",
  "origin_module": "AHT20"
}
```

Esto permite saber de dónde procede un dato.

---

# 114. Confianza

Los sensores avanzados podrán proporcionar:

```text
confidence
```

Ejemplo:

```text
temperature = 24.3
confidence = 0.98
```

Esto puede ser utilizado posteriormente por:

* IA;
* predicción;
* detección de anomalías;
* automatizaciones.

---

# 115. IA

La futura capa de IA no debe comunicarse directamente con GPIO.

Debe utilizar:

```text
Entities
States
Events
Commands
```

Ejemplo:

```text
AI
 ↓
predicts:
temperature will rise
 ↓
System Bus
 ↓
Automation
 ↓
COMMAND
 ↓
Ventilation
```

---

# 116. Cámara

Una cámara puede publicar:

```text
EVENT:
person_detected
```

o:

```text
ENTITY:
camera.front
```

La aplicación no necesita conocer:

```text
camera driver
DMA
frame buffer
GPIO
```

---

# 117. Energy

Un medidor energético puede publicar:

```text
sensor.house.power
sensor.house.energy
sensor.house.voltage
sensor.house.current
```

Las automatizaciones consumen esas entidades.

Ejemplo:

```text
power > 5000 W
      ↓
EVENT
      ↓
Automation
      ↓
disable non-critical loads
```

---

# 118. Water

El mismo modelo puede utilizar:

```text
water.flow
water.pressure
water.level
water.consumption
```

Esto permite utilizar la misma plataforma en:

* viviendas;
* agricultura;
* riego;
* industria;
* instalaciones marinas.

---

# 119. Industrial

Para aplicaciones industriales se podrán representar:

```text
motor.speed
pump.state
valve.position
pressure
temperature
flow
alarm
emergency_stop
```

El System Bus debe seguir siendo independiente de si el dato proviene de:

```text
CAN
Modbus
Ethernet
GPIO
analog input
```

---

# 120. Marine

La misma arquitectura puede utilizar:

```text
engine.rpm
tank.level
bilge.water
battery.voltage
battery.current
navigation.position
alarm.engine
```

Los transportes pueden variar sin modificar el modelo lógico.

---

# 121. Testing

El System Bus debe poder probarse sin hardware.

Debe existir un:

```text
MockTransport
```

Ejemplo:

```text
Test
 ↓
Mock System Bus
 ↓
Fake Node
 ↓
Expected Event
```

Esto permite ejecutar pruebas en:

```text
PC
CI/CD
PlatformIO
```

sin hardware físico.

---

# 122. Tests mínimos

Deberán existir pruebas para:

* creación de mensajes;
* validación;
* routing;
* subscriptions;
* filtros;
* prioridades;
* QoS;
* retry;
* timeout;
* deduplicación;
* correlation;
* TTL;
* serialization;
* deserialization;
* state synchronization;
* transport failover;
* queue overflow;
* security;
* version compatibility.

---

# 123. Conformance Tests

Cada nuevo Transport Adapter deberá cumplir una interfaz común.

Ejemplo:

```text
LocalTransport
WiFiTransport
EthernetTransport
CANTransport
RS485Transport
```

deberán superar el mismo conjunto de pruebas conceptuales.

Esto garantiza que cambiar de transporte no cambie la semántica.

---

# 124. Observabilidad

El System Bus deberá poder inspeccionarse desde la interfaz web.

Ejemplo:

```text
System → Communication → System Bus
```

Mostrar:

```text
Messages/s
Queue usage
Connected transports
Latency
Errors
Retries
Dropped messages
Nodes online
Nodes offline
```

---

# 125. Modo diagnóstico

Un modo técnico podrá mostrar:

```text
Message ID
Source
Destination
Transport
QoS
Priority
Timestamp
Latency
Payload size
Status
```

Este modo debe estar restringido a usuarios con permisos adecuados.

---

# 126. Logs

Los logs deberán poder diferenciar:

```text
BUS
TRANSPORT
ROUTING
SECURITY
COMMAND
EVENT
STATE
ERROR
```

Ejemplo:

```text
[BUS] command accepted
[ROUTING] destination=node-004
[TRANSPORT] ethernet
[ACK] received
[COMMAND] executed
```

---

# 127. Evitar exceso de logs

No se deberá registrar cada telemetría de alta frecuencia en niveles normales.

Los logs detallados podrán activarse mediante:

```text
DEBUG
TRACE
```

durante diagnóstico.

---

# 128. Persistencia de eventos críticos

Eventos críticos deberán poder almacenarse localmente.

Ejemplo:

```text
alarm_triggered
emergency_stop
over_temperature
water_leak
fire_detected
```

Esto permite recuperar información después de una desconexión.

---

# 129. Prioridad de funcionamiento

La arquitectura debe seguir:

```text
1. Seguridad
2. Control local
3. Automatización local
4. Automatización de zona
5. Coordinación Central
6. Integraciones externas
7. Cloud
```

El System Bus debe respetar esta prioridad.

---

# 130. Regla de diseño

Nunca implementar una función crítica de esta forma:

```text
Sensor
 ↓
Internet
 ↓
Cloud
 ↓
Central
 ↓
Command
 ↓
Actuator
```

cuando pueda implementarse:

```text
Sensor
 ↓
Local System Bus
 ↓
Automation
 ↓
Actuator
```

---

# 131. Evolución futura

El System Bus deberá permitir agregar:

```text
Matter
Thread
Zigbee
CANopen
Ethernet/IP
OPC UA Gateway
BACnet Gateway
KNX Gateway
LoRaWAN Gateway
```

sin rediseñar el Device Model.

La incorporación de un nuevo transporte debe consistir principalmente en:

```text
nuevo adapter
+
routing
+
capability negotiation
+
tests
```

---

# 132. Regla de desacoplamiento

La aplicación nunca deberá hacer:

```cpp
if (transport == WIFI) {
    ...
}

if (transport == CAN) {
    ...
}

if (transport == MODBUS) {
    ...
}
```

para resolver lógica de negocio.

Debe hacer:

```cpp
bus.send(command);
```

y permitir que el System Bus resuelva el transporte.

---

# 133. Excepción: características específicas del transporte

Puede existir código específico cuando una característica realmente pertenezca al transporte.

Ejemplo:

```text
CAN bitrate
Ethernet link speed
Wi-Fi RSSI
RS485 baudrate
```

Pero deberá quedar en:

```text
Transport Adapter
Diagnostics
Hardware Configuration
```

y no en:

```text
Automation Logic
```

---

# 134. Estructura conceptual completa

```text
┌─────────────────────────────────────────────┐
│                  USER/API                   │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────┐
│              DEVICE MODEL                   │
│                                             │
│ Entity / State / Command / Event            │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────┐
│                SYSTEM BUS                   │
│                                             │
│ Routing                                     │
│ QoS                                         │
│ Priority                                    │
│ Retry                                       │
│ Timeout                                     │
│ Correlation                                 │
│ Deduplication                               │
│ Security                                    │
└──────────────────────┬──────────────────────┘
                       │
             ┌─────────┼─────────┐
             │         │         │
             ▼         ▼         ▼
          Local     Network    Field
             │         │         │
          FreeRTOS   Ethernet   CAN
                    Wi-Fi       RS485
                    Thread      Modbus
                    Matter
```

---

# 135. Flujo completo de un comando

```text
Usuario
   ↓
Web UI
   ↓
REST API
   ↓
Authorization
   ↓
Command
   ↓
System Bus
   ↓
Routing
   ↓
Transport Selection
   ↓
Ethernet/Wi-Fi/CAN/RS485
   ↓
Destination Node
   ↓
System Bus
   ↓
Module
   ↓
Hardware Driver
   ↓
Actuator
   ↓
Actual State
   ↓
STATE EVENT
   ↓
System Bus
   ↓
Central / API / UI
```

---

# 136. Regla de oro

La arquitectura completa debe respetar:

```text
Hardware
   ↓
Resource
   ↓
Capability
   ↓
Entity
   ↓
Device Model
   ↓
System Bus
   ↓
Transport
```

Nunca al revés.

---

# 137. Principios definitivos

El System Bus debe cumplir:

1. **No depender de un transporte específico.**
2. **Utilizar identidades lógicas.**
3. **Separar comandos, estados y eventos.**
4. **Soportar comunicación local y distribuida.**
5. **Permitir autonomía sin Central.**
6. **Permitir múltiples transportes simultáneamente.**
7. **Soportar QoS y prioridades.**
8. **Soportar timeout y retry.**
9. **Soportar deduplicación.**
10. **Soportar correlation IDs.**
11. **Permitir discovery.**
12. **Permitir state synchronization.**
13. **Integrarse con FreeRTOS.**
14. **Ser testeable sin hardware.**
15. **Permitir nuevos transportes sin modificar la aplicación.**
16. **Mantener separación entre hardware y lógica.**
17. **Permitir diagnóstico y observabilidad.**
18. **Priorizar seguridad y autonomía local.**
19. **Mantener compatibilidad entre versiones.**
20. **No convertir MQTT, CAN, RS485 o Ethernet en dependencia arquitectónica.**

---

# 138. Principio final

> **El System Bus es la columna vertebral lógica de comunicación de la plataforma.**

El hardware puede cambiar.

El MCU puede cambiar.

El transporte puede cambiar.

La topología puede cambiar.

Un nodo puede ser reemplazado.

Ethernet puede cambiar por Wi-Fi.

Wi-Fi puede cambiar por CAN.

RS485 puede cambiar por Ethernet.

Un dispositivo puede pasar de un ESP32-WROOM a un ESP32-S3.

Nada de esto debería obligar a modificar la lógica de las automatizaciones.

La aplicación debe seguir viendo:

```text
Entities
Capabilities
States
Commands
Events
Functions
Scenes
Automations
```

y el System Bus debe encargarse de convertir esa comunicación lógica en el mecanismo físico apropiado.

> **La aplicación publica intenciones y consume eventos; el transporte es una implementación intercambiable.**

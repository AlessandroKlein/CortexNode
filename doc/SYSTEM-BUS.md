# SYSTEM-BUS.md

# System Bus — Distributed Automation Platform

> **Tipo:** Especificación de arquitectura y comunicación
> **Estado:** Diseño
> **Versión:** 1.0.0
> **Última actualización:** 2026-10-05
> **Objetivo:** Definir el bus lógico de comunicación interno de la plataforma, independiente del medio físico de transporte.

---

# 1. Objetivo

El `System Bus` es la capa lógica que permite que todos los componentes de la plataforma intercambien:

* comandos;
* estados;
* eventos;
* descubrimiento;
* configuración;
* diagnósticos;
* sincronización;
* información de disponibilidad;
* información de salud;
* mensajes de control.

El System Bus **no es un protocolo físico**.

No debe depender directamente de:

* Wi-Fi;
* Ethernet;
* MQTT;
* CAN;
* CANopen;
* RS485;
* Modbus;
* Zigbee;
* Thread;
* Matter;
* Bluetooth.

La arquitectura debe ser:

```text
                    SYSTEM BUS
                        │
        ┌───────────────┼────────────────┐
        │               │                │
      Wi-Fi          Ethernet          CAN
        │               │                │
      MQTT           TCP/UDP         CAN/CANopen
        │               │                │
      RS485        WebSocket        Modbus
```

La lógica superior no debe saber qué transporte se está utilizando.

---

# 2. Principio fundamental

> **La aplicación define qué quiere comunicar. El System Bus define cómo se entrega. El transporte físico define cómo viaja.**

Ejemplo:

```text
Automation
    ↓
Command
    ↓
System Bus
    ↓
Transport Adapter
    ↓
Ethernet
    ↓
Node
```

La automatización nunca debería hacer:

```text
sendMQTT(...)
```

ni:

```text
sendCAN(...)
```

ni:

```text
writeModbusRegister(...)
```

directamente.

Debe hacer:

```text
bus.publish(command)
```

---

# 3. Arquitectura por capas

```text
┌──────────────────────────────────────────────┐
│              APPLICATION LAYER              │
│                                              │
│ Automations / Scenes / Functions / UI / API │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│                 DATA MODEL                   │
│                                              │
│ Entity / State / Command / Event             │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│                 SYSTEM BUS                   │
│                                              │
│ Routing / Priority / ACK / Retry / QoS       │
│ Discovery / Synchronization / Correlation    │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│              TRANSPORT LAYER                 │
│                                              │
│ Ethernet / Wi-Fi / CAN / RS485 / MQTT       │
│ WebSocket / Thread / Matter / etc.          │
└──────────────────────────────────────────────┘
```

---

# 4. System Bus vs Transport

Esta separación es obligatoria.

## System Bus

Se ocupa de:

* mensajes;
* direccionamiento lógico;
* prioridades;
* correlación;
* ACK;
* reintentos;
* deduplicación;
* expiración;
* sincronización;
* discovery;
* seguridad lógica;
* routing.

## Transport

Se ocupa de:

* transmitir bytes;
* conexión física;
* framing;
* paquetes;
* checksum;
* enlace;
* reconexión;
* MTU;
* velocidad;
* características específicas del medio.

---

# 5. Transport Adapters

Cada transporte se implementará mediante un adaptador.

```text
System Bus
    │
    ├── EthernetAdapter
    ├── WiFiAdapter
    ├── CANAdapter
    ├── RS485Adapter
    ├── MQTTAdapter
    ├── WebSocketAdapter
    ├── MatterAdapter
    └── FutureAdapter
```

Cada adaptador debe implementar una interfaz común.

Conceptualmente:

```cpp
class TransportAdapter {
public:
    virtual bool send(const BusMessage& message);
    virtual bool receive(BusMessage& message);
    virtual bool isAvailable();
    virtual void process();
};
```

La implementación real podrá variar según firmware.

---

# 6. Bus Message

Todos los mensajes utilizarán un envelope común.

Ejemplo:

```json
{
  "message_id": "msg_01JXYZ",
  "message_type": "command",
  "schema_version": "1.0.0",

  "timestamp": "2026-10-05T15:30:00Z",

  "source": {
    "node_id": "node.living"
  },

  "destination": {
    "type": "entity",
    "id": "light.living.main"
  },

  "priority": "normal",

  "request_id": "req_01JXYZ",
  "correlation_id": "corr_01JXYZ",

  "payload": {}
}
```

---

# 7. Message ID

Cada mensaje debe tener:

```text
message_id
```

Debe ser globalmente único dentro del ámbito de la plataforma.

Ejemplo:

```text
msg_01JXYZ...
```

El `message_id` permite:

* deduplicación;
* trazabilidad;
* debugging;
* auditoría;
* reintentos.

---

# 8. Message Types

Tipos iniciales:

```text
command
command_result
event
state
state_snapshot
discovery
discovery_response
configuration
configuration_result
health
diagnostic
ack
nack
sync
sync_response
heartbeat
presence
```

---

# 9. Command Message

Ejemplo:

```json
{
  "message_type": "command",

  "destination": {
    "type": "entity",
    "id": "light.living.main"
  },

  "payload": {
    "action": "turn_on",
    "parameters": {
      "brightness": 80
    }
  }
}
```

El comando se ejecuta en el destino correspondiente.

---

# 10. Event Message

Ejemplo:

```json
{
  "message_type": "event",

  "destination": {
    "type": "broadcast"
  },

  "payload": {
    "event_id": "evt_01JXYZ",
    "type": "motion.detected",
    "entity_id": "binary_sensor.hall.motion"
  }
}
```

Los eventos normalmente no requieren una respuesta.

---

# 11. State Message

Un State Message informa de un estado actual.

Ejemplo:

```json
{
  "message_type": "state",

  "destination": {
    "type": "broadcast"
  },

  "payload": {
    "entity_id": "sensor.living.temperature",
    "value": 23.4,
    "unit": "°C",
    "quality": "good"
  }
}
```

---

# 12. State Snapshot

Un `state_snapshot` contiene múltiples estados.

Se utiliza principalmente para:

* sincronización;
* recuperación;
* reconexión;
* Central;
* Zone Controller.

Ejemplo:

```json
{
  "message_type": "state_snapshot",

  "payload": {
    "entities": [
      {
        "entity_id": "light.living.main",
        "state": {
          "on": true
        }
      },
      {
        "entity_id": "sensor.living.temperature",
        "state": {
          "value": 23.4,
          "unit": "°C"
        }
      }
    ]
  }
}
```

---

# 13. ACK

Un ACK confirma recepción o aceptación de un mensaje.

Importante:

> ACK no significa necesariamente que la operación haya terminado.

Ejemplo:

```json
{
  "message_type": "ack",
  "payload": {
    "message_id": "msg_01JXYZ",
    "status": "accepted"
  }
}
```

---

# 14. NACK

Un NACK indica rechazo.

Ejemplo:

```json
{
  "message_type": "nack",
  "payload": {
    "message_id": "msg_01JXYZ",
    "error": {
      "code": "INVALID_COMMAND",
      "message": "Unsupported action"
    }
  }
}
```

---

# 15. ACK vs Command Result

Debe distinguirse:

```text
ACK
    =
"Recibí / acepté el mensaje"

Command Result
    =
"El comando terminó"
```

Ejemplo:

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
   │
   │ ... ejecución ...
   │
   │ COMMAND_RESULT
   ▼
Central
```

---

# 16. Correlation

Los mensajes relacionados deben utilizar:

```text
request_id
correlation_id
```

Ejemplo:

```text
User request
     │
     └── request_id
           │
           ├── command
           ├── ack
           ├── command_result
           └── events
```

Esto permite reconstruir una operación completa.

---

# 17. Request ID

`request_id` identifica la solicitud original.

Ejemplo:

```text
req_01JXYZ
```

Puede atravesar múltiples capas.

---

# 18. Correlation ID

`correlation_id` agrupa una secuencia relacionada.

Ejemplo:

```text
corr.living.motion.123
```

Puede representar:

```text
Motion
 ↓
Automation
 ↓
Light command
 ↓
Notification
```

---

# 19. Addressing

El direccionamiento debe ser lógico.

Tipos de destino:

```text
node
device
entity
zone
group
function
central
broadcast
multicast
```

Ejemplo:

```json
{
  "destination": {
    "type": "zone",
    "id": "zone.living"
  }
}
```

---

# 20. Addressing Examples

### Node

```text
node.kitchen.controller
```

### Device

```text
device.kitchen.relay01
```

### Entity

```text
light.kitchen.main
```

### Zone

```text
zone.kitchen
```

### Group

```text
group.downstairs_lights
```

### Broadcast

```text
broadcast
```

---

# 21. Routing

El System Bus debe poder determinar la ruta adecuada.

Ejemplo:

```text
Central
   ↓
Zone: Ground Floor
   ↓
Node: Living Controller
   ↓
Entity: light.living.main
```

El origen no necesita conocer necesariamente:

```text
IP
MAC
GPIO
MQTT topic
CAN ID
```

---

# 22. Local Routing

Un Node puede resolver localmente un comando.

Ejemplo:

```text
Automation
    ↓
System Bus
    ↓
Local Entity
```

No debe enviarse innecesariamente al Central.

---

# 23. Zone Routing

Si una entidad pertenece a una zona:

```text
Zone Controller
```

puede actuar como router local.

Ejemplo:

```text
Central
   ↓
Zone Controller
   ├── Node A
   ├── Node B
   └── Node C
```

---

# 24. Central Routing

El Central puede actuar como router lógico cuando:

* no existe ruta local;
* se necesita coordinación global;
* el destino pertenece a otra zona;
* una integración externa genera la orden.

Pero:

> El Central no debe ser un requisito para la comunicación local.

---

# 25. Broadcast

Broadcast permite enviar información a todos los nodos compatibles.

Ejemplos:

```text
system announcement
time synchronization
emergency
discovery
firmware availability
```

No debe utilizarse indiscriminadamente.

---

# 26. Multicast

Multicast permite dirigirse a un conjunto lógico.

Ejemplo:

```text
group.all_lights
```

o:

```text
zone.ground_floor
```

---

# 27. Priority

Los mensajes pueden tener:

```text
emergency
critical
high
normal
low
background
```

Prioridad:

```text
EMERGENCY
    ↓
CRITICAL
    ↓
HIGH
    ↓
NORMAL
    ↓
LOW
    ↓
BACKGROUND
```

---

# 28. Priority Rules

Un mensaje de prioridad baja nunca debe bloquear indefinidamente uno crítico.

Ejemplo:

```text
Firmware telemetry
      ↓
LOW

Door alarm
      ↓
CRITICAL
```

El sistema debe procesar primero el segundo.

---

# 29. Safety Override

Las órdenes relacionadas con seguridad pueden interrumpir otras operaciones.

Ejemplo:

```text
normal:
motor = running

emergency:
motor = stop
```

La orden de emergencia debe poder superar la cola normal.

---

# 30. Message TTL

Los mensajes pueden tener TTL.

Ejemplo:

```json
{
  "ttl_ms": 5000
}
```

Si el mensaje expira antes de ejecutarse:

```text
TIMEOUT / EXPIRED
```

Esto es especialmente importante para:

* movimiento;
* iluminación temporal;
* comandos de UI;
* presencia;
* acciones dependientes del tiempo.

---

# 31. Retry

Los mensajes pueden reintentarse.

Parámetros:

```text
max_retries
retry_interval
backoff
```

Ejemplo:

```text
Retry 1 → 100 ms
Retry 2 → 250 ms
Retry 3 → 500 ms
Retry 4 → 1 s
```

Se recomienda exponential backoff con límites.

---

# 32. Retry Policy

No todos los mensajes deben reintentarse.

### Sí

* configuración;
* comandos idempotentes;
* sincronización;
* discovery.

### Condicional

* eventos;
* estados;
* notificaciones.

### Generalmente no

* acciones temporales ya expiradas;
* comandos peligrosos no idempotentes;
* comandos de emergencia ya ejecutados.

---

# 33. Deduplication

Los nodos deben detectar mensajes repetidos mediante:

```text
message_id
command_id
request_id
```

Ejemplo:

```text
COMMAND
message_id = A

retry

COMMAND
message_id = B
command_id = C
```

Si `command_id = C` ya fue ejecutado, el nodo puede devolver el resultado almacenado.

---

# 34. Idempotency

Los comandos deben indicar si son idempotentes cuando sea relevante.

Ejemplo:

```text
turn_on
```

normalmente puede tratarse como idempotente.

Mientras:

```text
toggle
```

no lo es.

Por eso:

```text
turn_on
turn_off
```

son preferibles a:

```text
toggle
```

en comunicaciones distribuidas críticas.

---

# 35. QoS

El System Bus puede definir niveles conceptuales:

```text
QoS 0
Best effort

QoS 1
At least once

QoS 2
Exactly once lógico
```

La implementación exacta dependerá del transporte.

Importante:

> “Exactly once” debe entenderse como semántica lógica, no necesariamente como garantía física del transporte.

---

# 36. Reliability Classes

Además de QoS, se pueden definir clases:

```text
best_effort
reliable
critical
```

Ejemplo:

```text
temperature telemetry
→ best_effort

configuration
→ reliable

emergency stop
→ critical
```

---

# 37. Message Persistence

No todos los mensajes deben persistirse.

### Persistentes

* configuración;
* comandos críticos pendientes;
* eventos importantes;
* estado deseado.

### No persistentes

* telemetry de alta frecuencia;
* heartbeats;
* estados redundantes;
* métricas temporales.

---

# 38. Offline Queue

Los nodos pueden mantener una cola local.

Ejemplo:

```text
Central offline
       ↓
Node continues
       ↓
Local automation
       ↓
Events queued
       ↓
Central returns
       ↓
Synchronization
```

La cola debe tener:

```text
maximum size
priority
expiration
persistence policy
```

---

# 39. Queue Overflow

Si la cola se llena:

```text
critical
    >
high
    >
normal
    >
low
    >
background
```

Los mensajes de menor prioridad pueden descartarse primero.

Nunca se deben descartar silenciosamente mensajes críticos.

---

# 40. Heartbeat

Los nodos pueden enviar:

```text
heartbeat
```

Ejemplo:

```json
{
  "message_type": "heartbeat",
  "source": {
    "node_id": "node.living"
  },
  "payload": {
    "uptime": 123456,
    "load": 31,
    "free_heap": 182000
  }
}
```

---

# 41. Presence

El heartbeat no necesariamente representa disponibilidad funcional.

Por eso puede existir:

```text
presence
```

Estados:

```text
online
offline
degraded
maintenance
unknown
```

---

# 42. Health

Health proporciona información más detallada.

Ejemplo:

```json
{
  "message_type": "health",
  "payload": {
    "status": "degraded",
    "cpu": 80,
    "memory": 92,
    "network": "good",
    "storage": "warning"
  }
}
```

---

# 43. Discovery

Discovery permite que un nodo anuncie sus capacidades.

Flujo:

```text
Node boot
   ↓
Identity
   ↓
Discovery
   ↓
Capabilities
   ↓
Resources
   ↓
Entities
   ↓
Ready
```

---

# 44. Discovery Request

Ejemplo:

```json
{
  "message_type": "discovery",
  "payload": {
    "action": "request"
  }
}
```

---

# 45. Discovery Response

Ejemplo:

```json
{
  "message_type": "discovery_response",
  "payload": {
    "device_id": "device.node01",
    "firmware": {
      "name": "AutomationNode",
      "version": "1.2.0"
    },
    "resources": [],
    "entities": []
  }
}
```

---

# 46. Discovery Rules

Discovery debe ser:

* seguro;
* autenticado cuando corresponda;
* repetible;
* idempotente;
* compatible con nodos offline;
* independiente del transporte.

Un nodo no debe depender del Central para conocer sus propias funciones.

---

# 47. Provisioning

Discovery no significa necesariamente autorización.

```text
Discovery
    ↓
Unknown Device
    ↓
Authentication
    ↓
Authorization
    ↓
Provisioning
    ↓
Operational
```

Esto evita que cualquier dispositivo pueda incorporarse automáticamente a una instalación protegida.

---

# 48. Synchronization

La sincronización tiene varias dimensiones:

```text
Configuration
State
Events
Time
Capabilities
Firmware
Permissions
```

Cada una debe poder sincronizarse independientemente.

---

# 49. Configuration Synchronization

Ejemplo:

```text
Central config version = 20
Node config version    = 18
```

El Node informa:

```text
CONFIGURATION_OUTDATED
```

El Central puede enviar la versión 20.

---

# 50. Configuration Reconciliation

El objetivo no es simplemente enviar configuración.

Debe existir:

```text
Desired configuration
        ↓
Applied configuration
        ↓
Verification
        ↓
Reconciliation
```

Si la aplicación falla:

```text
desired ≠ applied
```

el sistema debe conservar esa información.

---

# 51. State Synchronization

Cuando un nodo reconecta:

```text
Node
 ↓
Current state
 ↓
Central compares
 ↓
Differences
 ↓
Reconciliation
```

El estado físico real tiene prioridad sobre una suposición obsoleta.

---

# 52. Event Synchronization

Los eventos pueden sincronizarse mediante:

```text
event_id
timestamp
sequence
```

El nodo puede indicar:

```text
last_event_id
```

o:

```text
last_sequence
```

El Central puede solicitar:

```text
events since sequence 1024
```

---

# 53. Sequence Numbers

Los nodos pueden utilizar:

```text
sequence_number
```

por stream.

Ejemplo:

```text
1001
1002
1003
1004
```

Si Central recibe:

```text
1001
1002
1004
```

detecta:

```text
missing = 1003
```

---

# 54. Time Synchronization

El sistema debe soportar:

```text
NTP
```

y sincronización interna.

Jerarquía recomendada:

```text
Internet NTP
      ↓
Central
      ↓
Zone Controller
      ↓
Nodes
```

Pero un nodo debe poder operar con su propio reloj si pierde conexión.

---

# 55. Monotonic Time

Para:

* timeout;
* retry;
* TTL;
* scheduling interno;

se debe utilizar preferentemente un reloj monotónico.

No depender exclusivamente de:

```text
wall clock
```

porque puede cambiar después de una sincronización NTP.

---

# 56. Security

El System Bus debe contemplar:

```text
Authentication
Authorization
Integrity
Confidentiality
Replay protection
Message freshness
```

---

# 57. Device Authentication

Cada dispositivo debe poseer una identidad.

Ejemplo conceptual:

```text
device_id
+
credential
+
certificate/key
```

Nunca se debe autenticar únicamente por:

```text
MAC address
IP
hostname
```

---

# 58. Message Integrity

Los mensajes críticos deben poder verificar integridad.

Opciones:

```text
TLS
DTLS
MAC
digital signature
transport-level security
```

La implementación dependerá del transporte.

---

# 59. Encryption

El bus no debe asumir que todos los transportes son seguros.

Ejemplos:

```text
Ethernet
→ TLS cuando corresponda

Wi-Fi
→ TLS / secure transport

MQTT
→ MQTT over TLS

RS485
→ cifrado/autenticación a nivel superior si es necesario

CAN
→ seguridad de aplicación cuando corresponda
```

---

# 60. Replay Protection

Los mensajes críticos deben protegerse contra repetición.

Se pueden utilizar:

```text
timestamp
nonce
sequence
message_id
expiration
```

Ejemplo:

```text
command
+
nonce
+
timestamp
+
signature
```

---

# 61. Permissions

El System Bus debe respetar las políticas de autorización.

Ejemplo:

```text
User
 ↓
Permission
 ↓
Command
 ↓
System Bus
 ↓
Target
```

Un nodo no debe ejecutar automáticamente cualquier comando recibido por el bus.

---

# 62. Local Authority

Un Node puede rechazar un comando del Central si:

* viola una condición de seguridad;
* está fuera de rango;
* el dispositivo está en mantenimiento;
* el comando requiere permisos superiores;
* el comando está expirado;
* la automatización local tiene prioridad.

---

# 63. Local-First Principle

Regla fundamental:

> **La pérdida del Central no debe detener las funciones locales críticas.**

Ejemplo:

```text
Central OFFLINE

Security Node
    ↓
continues detecting doors

Lighting Node
    ↓
continues local automations

Climate Node
    ↓
continues climate control
```

---

# 64. Autonomy Hierarchy

La arquitectura utiliza:

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

La capa superior coordina, pero no debe quitar capacidad esencial a las capas inferiores.

---

# 65. Central Failure

Si Central falla:

```text
Central
   X
```

debe continuar:

```text
Node → Node
Node → Zone
Local Automation
Safety
Critical control
```

Puede perderse temporalmente:

```text
Global UI
Global history
External integrations
Central orchestration
```

pero no necesariamente:

```text
security
lighting
climate
local control
```

---

# 66. Zone Controller Failure

Si un Zone Controller falla:

```text
Zone Controller
       X
```

los nodos deben conservar:

```text
local configuration
local entities
critical automations
safety functions
```

Cuando vuelva:

```text
Discovery
↓
Sync
↓
Reconciliation
```

---

# 67. Node Failure

Un fallo de un Node debe afectar únicamente las funciones dependientes de él.

Ejemplo:

```text
Node Kitchen OFFLINE

Kitchen light
Kitchen temperature
Kitchen relay

affected

Living room
Garage
Security

unaffected
```

---

# 68. Transport Failure

El System Bus debe detectar la pérdida de un transporte.

Ejemplo:

```text
Ethernet unavailable
        ↓
Wi-Fi available
        ↓
same logical bus
```

Si existen múltiples transportes compatibles, el sistema puede cambiar de ruta.

---

# 69. Multi-Transport

Un Node puede disponer de:

```text
Wi-Fi
Ethernet
CAN
RS485
```

El modelo lógico permanece:

```text
Node
 ↓
System Bus
```

y el router selecciona:

```text
best transport
```

según:

* disponibilidad;
* latencia;
* prioridad;
* coste;
* confiabilidad;
* alcance.

---

# 70. Transport Priority

Ejemplo:

```text
Ethernet
    >
Wi-Fi
    >
Thread
    >
Mesh
```

Pero la prioridad debe ser configurable.

En una instalación industrial:

```text
CAN
    >
RS485
    >
Wi-Fi
```

podría ser más apropiado.

---

# 71. Routing Table

Un nodo puede mantener información como:

```json
{
  "routes": [
    {
      "destination": "zone.living",
      "transport": "ethernet",
      "cost": 10
    },
    {
      "destination": "zone.garage",
      "transport": "can",
      "cost": 20
    }
  ]
}
```

La implementación final podrá utilizar estructuras más compactas.

---

# 72. Bus Topology

La plataforma debe permitir:

```text
Star
Tree
Mesh
Point-to-point
Bus
Hybrid
```

Ejemplo:

```text
              Central
             /       \
          Zone A    Zone B
         /   \      /   \
      Node  Node  Node  Node
```

No debe existir dependencia arquitectónica de una única topología.

---

# 73. Physical Bus vs Logical Bus

Ejemplo:

```text
LOGICAL

light.living.main
       ↓
System Bus


PHYSICAL

Ethernet
192.168.x.x
```

Otro caso:

```text
LOGICAL

light.living.main
       ↓
System Bus


PHYSICAL

CAN
ID 0x123
```

La Entity es la misma.

---

# 74. CAN

CAN puede utilizarse como transporte de bajo nivel.

El System Bus debe mapear:

```text
Bus Message
    ↓
CAN frame(s)
```

Debe contemplar:

* fragmentación;
* reensamblado;
* prioridad;
* CAN ID;
* timeout;
* CRC proporcionado por CAN;
* autenticación a nivel superior cuando corresponda.

---

# 75. RS485

RS485 es un medio físico.

No debe confundirse con:

```text
Modbus
```

La arquitectura puede soportar:

```text
System Bus
    ↓
RS485 Adapter
    ↓
Modbus
```

pero también otros protocolos.

---

# 76. Modbus

Modbus debe considerarse un protocolo/adaptador externo.

Ejemplo:

```text
System Bus
    ↓
Modbus Adapter
    ↓
RS485
    ↓
Industrial Sensor
```

El registro Modbus no debe aparecer en el modelo lógico de la Entity.

---

# 77. MQTT

MQTT puede actuar como:

```text
Transport Adapter
```

o:

```text
Integration Transport
```

Ejemplo:

```text
System Bus
    ↓
MQTT Adapter
    ↓
Broker
```

El modelo interno no debe depender de MQTT.

---

# 78. MQTT Topic Mapping

Ejemplo:

```text
Entity:
light.living.main
```

puede mapearse a:

```text
automation/site/home/entity/light/living/main
```

Pero el topic es una implementación del adaptador.

Cambiar el topic no cambia:

```text
entity_id
```

---

# 79. WebSocket

WebSocket es especialmente útil para:

* Web UI;
* streaming de estados;
* eventos;
* comandos;
* dashboards.

Arquitectura:

```text
Browser
   ↓
WebSocket
   ↓
API Layer
   ↓
System Bus
```

El navegador no debe hablar directamente con GPIO.

---

# 80. Matter

Matter se considera una integración/protocolo externo.

Arquitectura:

```text
Matter
   ↓
Matter Adapter
   ↓
Entity Model
   ↓
System Bus
```

La entidad interna permanece independiente de:

```text
endpoint
cluster
attribute
```

---

# 81. External Systems

Sistemas como:

* Home Assistant;
* Homey;
* SmartThings;
* Apple Home;
* Google Home;
* Alexa;
* aplicaciones propias;

deben conectarse mediante adaptadores.

```text
External Ecosystem
       ↓
Integration Adapter
       ↓
System Bus
       ↓
Entity Model
```

---

# 82. Loop Prevention

Las integraciones pueden producir loops.

Ejemplo:

```text
Home Assistant
      ↓
Command
      ↓
System
      ↓
State
      ↓
Home Assistant
      ↓
Command
      ↓
...
```

El sistema debe utilizar:

```text
source
origin
correlation_id
message_id
```

para detectar loops.

---

# 83. Event Propagation

Un evento puede propagarse:

```text
Sensor Node
    ↓
Zone
    ↓
Central
    ↓
Integration
```

Pero no necesariamente debe propagarse a todas las capas.

La política debe depender de:

```text
event type
priority
scope
subscriptions
```

---

# 84. Subscriptions

Los consumidores podrán suscribirse a:

```text
entity
domain
zone
group
event type
state changes
device
```

Ejemplo:

```text
subscribe:
zone.living
```

o:

```text
subscribe:
binary_sensor.*
```

---

# 85. Subscription Example

```json
{
  "message_type": "subscription",
  "payload": {
    "filters": [
      {
        "zone_id": "zone.living",
        "domain": "light"
      }
    ]
  }
}
```

---

# 86. Event Filtering

Los filtros pueden incluir:

```text
entity_id
domain
zone_id
device_id
event_type
priority
tag
```

Esto evita enviar información innecesaria a dispositivos pequeños.

---

# 87. Backpressure

Un consumidor lento no debe bloquear todo el bus.

Ejemplo:

```text
High frequency sensor
        ↓
100 messages/s

Slow consumer
        ↓
10 messages/s
```

El bus debe poder:

* limitar;
* agrupar;
* descartar telemetry no crítica;
* reducir frecuencia;
* aplicar backpressure.

---

# 88. State Coalescing

Para estados que cambian rápidamente:

```text
20
21
22
23
24
25
```

puede ser innecesario transmitir cada valor.

El bus puede enviar:

```text
25
```

si los estados intermedios no son importantes.

Esto no debe aplicarse a eventos críticos.

---

# 89. High Frequency Telemetry

Sensores como:

* acelerómetros;
* energía;
* temperatura rápida;
* corriente;
* audio;
* cámaras;

pueden generar mucho tráfico.

Se recomienda separar:

```text
control plane
```

de:

```text
telemetry plane
```

---

# 90. Control Plane

Contiene:

```text
commands
configuration
discovery
authentication
synchronization
critical events
```

Debe tener alta confiabilidad.

---

# 91. Telemetry Plane

Contiene:

```text
sensor samples
diagnostics
metrics
high-frequency data
```

Puede utilizar:

```text
sampling
aggregation
batching
compression
coalescing
```

---

# 92. Data Plane

En futuras implementaciones puede distinguirse además:

```text
Control Plane
Telemetry Plane
Media/Data Plane
```

Por ejemplo:

```text
Camera
    ↓
Media/Data Plane
```

No debe saturar:

```text
Control Plane
```

---

# 93. Large Payloads

El System Bus no debe transportar indefinidamente grandes blobs.

Para:

* imágenes;
* vídeo;
* firmware;
* archivos;
* logs grandes;

se recomienda utilizar:

```text
reference
+
metadata
```

Ejemplo:

```json
{
  "message_type": "event",
  "payload": {
    "type": "camera.snapshot",
    "resource": {
      "id": "media_123",
      "location": "/storage/snapshot/123.jpg"
    }
  }
}
```

---

# 94. Fragmentation

Si un transporte tiene un MTU pequeño:

```text
Bus Message
     ↓
Fragment
     ↓
Transport
     ↓
Reassemble
     ↓
Bus Message
```

La fragmentación debe pertenecer al adaptador cuando sea posible.

La aplicación no debería preocuparse por ella.

---

# 95. Compression

La compresión puede utilizarse para:

* snapshots;
* configuration;
* logs;
* telemetry agregada.

No se debe comprimir indiscriminadamente mensajes pequeños.

---

# 96. Message Expiration

Un mensaje puede contener:

```text
created_at
expires_at
ttl
```

Ejemplo:

```json
{
  "created_at": "2026-10-05T15:30:00Z",
  "ttl_ms": 3000
}
```

Un comando de movimiento puede quedar inválido después de 3 segundos.

---

# 97. Critical Commands

Los comandos críticos deben contener:

```text
priority
authorization
expiration
correlation_id
source
safety constraints
```

Ejemplo:

```text
EMERGENCY STOP
```

No debe depender de una cola de baja prioridad.

---

# 98. Emergency Communication

La plataforma debe permitir un canal lógico de emergencia.

Ejemplo:

```text
Emergency event
      ↓
System Bus
      ↓
Critical priority
      ↓
Local nodes
      ↓
Actuators
```

El mecanismo físico concreto dependerá de la aplicación.

Para funciones donde una comunicación digital no sea suficiente como medida de seguridad, deberán utilizarse circuitos de seguridad independientes.

---

# 99. Watchdog

El System Bus puede supervisar:

```text
message processing
transport health
queue health
consumer health
```

Un watchdog debe poder detectar:

```text
bus blocked
queue overflow
transport dead
consumer unresponsive
```

Pero no debe reiniciar innecesariamente un sistema que continúa funcionando correctamente.

---

# 100. FreeRTOS Integration

En ESP32 el bus deberá integrarse con FreeRTOS.

Arquitectura conceptual:

```text
┌─────────────────────────────┐
│ Application Tasks           │
└──────────────┬──────────────┘
               │
          Bus API
               │
               ▼
┌─────────────────────────────┐
│ System Bus Task             │
└──────────────┬──────────────┘
               │
        Message Queues
               │
       ┌───────┼────────┐
       ▼       ▼        ▼
    Ethernet  CAN      RS485
```

Las tareas de aplicación no deberían escribir directamente en sockets o buses físicos.

---

# 101. Task Isolation

Cada transporte puede poseer su propia tarea o mecanismo de procesamiento.

Ejemplo:

```text
Bus Core
Ethernet Task
CAN Task
RS485 Task
MQTT Task
WebSocket Task
```

El diseño final dependerá del dispositivo.

---

# 102. Queue Architecture

Ejemplo:

```text
Application
     ↓
Outgoing Queue
     ↓
System Bus
     ↓
Transport Queue
     ↓
Transport
```

Recepción:

```text
Transport
     ↓
Incoming Queue
     ↓
System Bus
     ↓
Routing
     ↓
Application
```

---

# 103. Priority Queues

Puede utilizarse:

```text
Emergency Queue
Critical Queue
Normal Queue
Background Queue
```

o una cola única con prioridad.

La elección depende de RAM y rendimiento.

---

# 104. Memory Ownership

Los mensajes deben tener una política clara de ownership.

Nunca se debe permitir que:

```text
Task A
```

libere memoria mientras:

```text
Task B
```

todavía utiliza el mensaje.

Se recomienda definir claramente:

```text
allocate
own
transfer
process
release
```

---

# 105. Zero-Copy

En dispositivos con pocos recursos puede utilizarse zero-copy cuando sea beneficioso.

Especialmente:

```text
Ethernet
Wi-Fi
CAN
large telemetry
```

Pero la complejidad no debe comprometer la estabilidad.

---

# 106. Buffer Limits

Cada transporte debe definir:

```text
max_message_size
rx_buffer
tx_buffer
queue_size
fragment_size
```

Los límites deben poder conocerse mediante capabilities del transporte.

---

# 107. Bus Health

El System Bus debe exponer métricas como:

```text
messages_sent
messages_received
messages_failed
messages_retried
messages_dropped
queue_depth
latency
timeouts
transport_errors
```

---

# 108. Latency

El sistema debe poder medir:

```text
message latency
command latency
transport latency
execution latency
```

Ejemplo:

```text
UI
 ↓ 5 ms
Central
 ↓ 8 ms
Zone
 ↓ 3 ms
Node
 ↓ 2 ms
Actuator

Total ≈ 18 ms
```

---

# 109. Distributed Tracing

Cada operación importante debe poder rastrearse:

```text
request_id
correlation_id
message_id
source
destination
timestamp
```

Esto permite generar:

```text
Trace:
req_123

Web
 ↓
Central
 ↓
Zone
 ↓
Node
 ↓
Entity
```

---

# 110. Diagnostics Mode

El System Bus debe poder disponer de un modo diagnóstico.

Permite inspeccionar:

```text
messages
routes
latency
errors
retries
queues
subscriptions
```

Pero debe existir protección para evitar que el diagnóstico sature el sistema.

---

# 111. Logging

Los logs deben poder clasificarse:

```text
error
warning
info
debug
trace
```

`trace` debe estar desactivado normalmente en dispositivos pequeños.

---

# 112. Security Logging

Los eventos de seguridad deben registrarse independientemente de los logs normales.

Ejemplos:

```text
authentication failed
authorization denied
unknown device
invalid signature
replay detected
credential revoked
```

---

# 113. Protocol Independence

La aplicación debe poder ejecutar:

```text
light.living.main → ON
```

sin saber si finalmente se utiliza:

```text
Ethernet
Wi-Fi
CAN
RS485
MQTT
Matter
Thread
```

Esta es una de las propiedades arquitectónicas más importantes del sistema.

---

# 114. Example — Ethernet

```text
Application
     ↓
System Bus
     ↓
Ethernet Adapter
     ↓
TCP/TLS
     ↓
Remote Node
```

---

# 115. Example — CAN

```text
Application
     ↓
System Bus
     ↓
CAN Adapter
     ↓
CAN Frame
     ↓
Remote Node
```

---

# 116. Example — RS485 / Modbus

```text
Application
     ↓
System Bus
     ↓
Modbus Adapter
     ↓
RS485
     ↓
Industrial Device
```

---

# 117. Example — MQTT

```text
Application
     ↓
System Bus
     ↓
MQTT Adapter
     ↓
Broker
     ↓
Remote Node
```

---

# 118. Example — Matter

```text
External Matter Controller
          ↓
Matter Adapter
          ↓
Entity Model
          ↓
System Bus
          ↓
Device
```

---

# 119. Example — Local Automation

```text
PIR
 ↓
binary_sensor.hall.motion
 ↓
Event
 ↓
Automation Engine
 ↓
Command
 ↓
light.hall
```

El Central puede no participar.

---

# 120. Example — Central Automation

```text
Sensor
   ↓
Node
   ↓
Zone
   ↓
Central
   ↓
Automation
   ↓
Command
   ↓
Zone
   ↓
Node
   ↓
Actuator
```

---

# 121. Example — External Integration

```text
Home Assistant
       ↓
Integration Adapter
       ↓
System Bus
       ↓
Entity
       ↓
Node
       ↓
Actuator
```

---

# 122. Failure Scenario — Internet

```text
Internet
   X
```

Debe continuar:

```text
Node
 ↓
Zone
 ↓
Central
```

si la red local sigue disponible.

Las integraciones cloud pueden quedar offline.

---

# 123. Failure Scenario — Central

```text
Central
   X
```

Debe continuar:

```text
Node
 ↓
Local Automation
 ↓
Actuator
```

y:

```text
Node ↔ Node
```

cuando exista conectividad directa.

---

# 124. Failure Scenario — Wi-Fi

Si Wi-Fi falla:

```text
Wi-Fi
  X
```

y existe Ethernet:

```text
System Bus
     ↓
Ethernet
```

La aplicación no debe necesitar cambios.

---

# 125. Failure Scenario — Node

```text
Node A
   X
```

Los demás nodos continúan funcionando.

El sistema debe generar:

```text
device.unavailable
```

para las entidades afectadas.

---

# 126. Failure Scenario — Transport

Si un transporte falla:

```text
Transport A
     X
```

el adaptador informa:

```text
transport unavailable
```

y el router puede:

```text
retry
failover
queue
```

según política.

---

# 127. System Bus API

La API interna conceptual debe ser pequeña.

Ejemplo:

```cpp
bus.publish(message);
bus.send(message);
bus.request(message);
bus.subscribe(filter, callback);
bus.unsubscribe(subscription);
bus.isAvailable(destination);
```

La implementación puede cambiar.

---

# 128. `publish`

Para mensajes sin respuesta directa:

```text
event
state
telemetry
```

---

# 129. `send`

Para mensajes dirigidos:

```text
command
configuration
```

---

# 130. `request`

Para operaciones request/response:

```text
discovery
health
configuration query
state query
```

---

# 131. `subscribe`

Para escuchar:

```text
events
state changes
telemetry
```

---

# 132. Local Bus

Cada dispositivo puede disponer de un bus local.

```text
Sensor
 ↓
Local Bus
 ↓
Entity
```

No todo necesita salir a la red.

---

# 133. Node Bus

Un Node puede contener:

```text
Hardware
 ↓
Resources
 ↓
Entities
 ↓
Local System Bus
```

---

# 134. Zone Bus

Un Zone Controller coordina:

```text
Node A
Node B
Node C
```

mediante el mismo modelo.

---

# 135. Central Bus

Central puede proporcionar:

```text
Global routing
Global subscriptions
Global automations
Integration bridge
API
```

pero sigue utilizando el mismo bus lógico.

---

# 136. Bus Federation

Múltiples dominios de bus pueden conectarse.

Ejemplo:

```text
Site A
   │
   └── Central A
          │
       Federation
          │
   Central B
   │
Site B
```

Esto permite futuras instalaciones multi-site.

---

# 137. Multi-Site

El `site_id` debe formar parte del direccionamiento lógico.

Ejemplo:

```text
site.home
site.office
site.greenhouse
site.boat
```

Una aplicación podría acceder a:

```text
site.office.zone.serverroom
```

sin cambiar el modelo.

---

# 138. Bus Namespace

Se recomienda un namespace lógico:

```text
site
zone
device
entity
```

Ejemplo:

```text
site.home
zone.home.living
device.home.living.controller
light.home.living.main
```

La sintaxis final de IDs podrá simplificarse, pero debe existir una jerarquía inequívoca.

---

# 139. System Bus vs API

No son lo mismo.

### API

Pensada para:

* usuarios;
* aplicaciones;
* integraciones;
* clientes externos.

### System Bus

Pensado para:

* comunicación interna;
* nodos;
* Central;
* automatizaciones;
* sincronización.

Arquitectura:

```text
External App
     ↓
API
     ↓
System Bus
     ↓
Node
```

---

# 140. System Bus vs Database

Tampoco son lo mismo.

```text
System Bus
→ comunicación en tiempo real

Database
→ persistencia
```

Un evento puede:

```text
System Bus
    ├── Automation
    ├── Integration
    └── Database
```

---

# 141. System Bus vs MQTT

MQTT es un transporte/protocolo.

System Bus es una abstracción arquitectónica.

Por lo tanto:

```text
System Bus ≠ MQTT
```

La plataforma puede funcionar:

```text
sin MQTT
```

---

# 142. System Bus vs Matter

De igual manera:

```text
System Bus ≠ Matter
```

Matter es una integración/protocolo externo.

---

# 143. Message Lifecycle

Todo mensaje debería seguir conceptualmente:

```text
CREATE
  ↓
VALIDATE
  ↓
AUTHORIZE
  ↓
ROUTE
  ↓
TRANSMIT
  ↓
RECEIVE
  ↓
DEDUPLICATE
  ↓
PROCESS
  ↓
ACK / RESULT
  ↓
TRACE
```

---

# 144. Invalid Message

Si un mensaje no cumple el schema:

```text
RECEIVE
   ↓
VALIDATE
   ↓
INVALID
   ↓
REJECT
```

Nunca debe ejecutarse parcialmente.

---

# 145. Unauthorized Message

```text
RECEIVE
   ↓
VALIDATE
   ↓
AUTHORIZATION
   ↓
DENIED
```

Debe generar:

```text
UNAUTHORIZED
```

o:

```text
FORBIDDEN
```

según el caso.

---

# 146. Unknown Destination

Si el destino no existe:

```text
DESTINATION_NOT_FOUND
```

El mensaje no debe retransmitirse indefinidamente.

---

# 147. Transport Unavailable

Si el destino existe pero no existe una ruta:

```text
NO_ROUTE
```

El bus puede:

```text
queue
retry
fail
```

según la política.

---

# 148. Message Expired

Si:

```text
now > expires_at
```

el mensaje debe descartarse.

Debe producir:

```text
MESSAGE_EXPIRED
```

cuando la semántica requiera informar el fallo.

---

# 149. Queue Policy

Cada cola deberá definir:

```text
capacity
priority
retention
drop policy
persistence
```

No debe existir una única cola ilimitada.

---

# 150. Rate Limiting

El bus debe limitar fuentes que generen demasiado tráfico.

Ejemplo:

```text
sensor → 1000 msg/s
```

puede limitarse a:

```text
100 msg/s
```

cuando no sea crítico.

Esto protege:

* RAM;
* CPU;
* Wi-Fi;
* Ethernet;
* Central;
* almacenamiento.

---

# 151. Security Rate Limiting

También debe limitar:

```text
authentication attempts
discovery requests
command requests
API-to-bus requests
```

para reducir abuso.

---

# 152. Bus Capacity

Cada dispositivo debe conocer sus límites aproximados:

```text
max_messages_per_second
max_payload_size
queue_capacity
```

Los dispositivos pequeños pueden anunciar límites inferiores.

---

# 153. Graceful Degradation

Cuando el bus está saturado:

```text
critical control
    ↓
continues

high priority
    ↓
continues

normal
    ↓
reduced

telemetry
    ↓
throttled
```

Nunca debe sacrificarse una función crítica para mantener telemetry.

---

# 154. Bus Configuration

La configuración debe poder definir:

```text
transport priority
retry policy
queue sizes
message limits
subscriptions
routing
security
```

Pero los valores peligrosos deben estar protegidos.

---

# 155. No-Code Configuration

El usuario normal debe poder configurar desde Web UI:

```text
Transportes
Nodes
Zones
Routes
Subscriptions
Integrations
Priorities
```

sin modificar firmware.

---

# 156. Expert Mode

Opciones avanzadas pueden existir para:

* CAN ID;
* RS485;
* Modbus;
* MTU;
* QoS;
* retry;
* queue;
* routing.

Pero deben permanecer separadas del modo normal.

---

# 157. Configuration Example

```text
System Bus
────────────────────────────

Transportes

☑ Ethernet
☑ Wi-Fi
☐ CAN
☐ RS485
☑ MQTT

Prioridad

Ethernet     [1]
Wi-Fi        [2]
CAN          [1]

Colas

Critical    32
Normal      128
Low         64
```

---

# 158. Compatibility

Los mensajes deben indicar:

```text
schema_version
protocol_version
capabilities
```

Ejemplo:

```json
{
  "protocol_version": "1.0",
  "schema_version": "1.0.0"
}
```

---

# 159. Protocol Version

Debe diferenciarse:

```text
protocol_version
```

de:

```text
schema_version
```

### Protocol

Cómo funciona el System Bus.

### Schema

Qué estructura tienen los datos.

---

# 160. Negotiation

Al conectarse dos nodos:

```text
Node A
  ↓
HELLO
  ↓
Node B
  ↓
CAPABILITIES
  ↓
VERSION NEGOTIATION
  ↓
READY
```

---

# 161. HELLO Message

Ejemplo:

```json
{
  "message_type": "hello",
  "protocol_version": "1.0",
  "device_id": "device.node01",
  "capabilities": [
    "command",
    "event",
    "state",
    "discovery",
    "sync"
  ]
}
```

---

# 162. READY State

Un nodo sólo debe comenzar operaciones distribuidas después de:

```text
Identity
↓
Authentication
↓
Protocol negotiation
↓
Schema compatibility
↓
Ready
```

Sin embargo, sus funciones locales críticas pueden iniciar antes.

---

# 163. Boot Order

Principio:

```text
BOOT
 ↓
Hardware
 ↓
Local Configuration
 ↓
Safety
 ↓
Local Automation
 ↓
System Bus
 ↓
Network
 ↓
Discovery
 ↓
Synchronization
 ↓
Central
```

Nunca:

```text
BOOT
 ↓
WAIT FOR CENTRAL
 ↓
START
```

---

# 164. Safe Boot

En caso de configuración corrupta:

```text
Stored config
      X
```

el nodo debe:

```text
fallback configuration
      ↓
safe state
      ↓
local operation
```

---

# 165. Bus Recovery

Después de un fallo:

```text
Transport failure
      ↓
Reconnect
      ↓
HELLO
      ↓
Authentication
      ↓
Sync
      ↓
Resume
```

No se debe asumir que el estado anterior sigue siendo válido.

---

# 166. State Resynchronization

Al reconectar:

```text
Current local state
        +
Known remote state
        ↓
Compare
        ↓
Resolve
```

La política depende de la entidad.

Ejemplo:

```text
Sensor:
local actual state → authoritative

Actuator:
actual + desired → reconcile
```

---

# 167. Configuration Reconciliation

Si:

```text
Central desired = v20
Node applied = v18
```

el sistema debe actualizar.

Pero si:

```text
Node reports hardware incompatibility
```

no debe aplicar ciegamente.

Debe generar:

```text
CONFIGURATION_FAILED
```

---

# 168. Hardware Capability Mismatch

Ejemplo:

Central solicita:

```text
brightness
```

pero el Node sólo soporta:

```text
on_off
```

Debe responder:

```text
CAPABILITY_NOT_SUPPORTED
```

No debe intentar emular silenciosamente una capacidad inexistente.

---

# 169. Command Validation

Antes de ejecutar:

```text
Schema
 ↓
Capability
 ↓
Availability
 ↓
Permission
 ↓
Safety
 ↓
Execute
```

---

# 170. Event Ordering

No siempre se puede garantizar orden global en un sistema distribuido.

Por eso debe utilizarse:

```text
timestamp
sequence
source
```

cuando sea necesario.

Nunca debe suponerse que dos mensajes recibidos en diferente transporte están globalmente ordenados sólo por su orden de llegada.

---

# 171. Exactly Once

No debe diseñarse la plataforma suponiendo que todos los transportes proporcionan exactamente una entrega.

Debe utilizarse:

```text
at-least-once delivery
+
deduplication
+
idempotency
```

cuando la operación lo requiera.

---

# 172. At-Most-Once

Para telemetry no crítica:

```text
send once
drop if unavailable
```

puede ser suficiente.

---

# 173. At-Least-Once

Para configuración:

```text
send
retry
deduplicate
```

es preferible.

---

# 174. Critical Delivery

Para comandos críticos:

```text
send
ACK
execute
RESULT
verify
```

y, cuando sea necesario, una verificación adicional del estado real.

---

# 175. Command Verification

Ejemplo:

```text
Command:
turn_on light

ACK
 ↓
Executed
 ↓
State:
ON
```

Si:

```text
actual_state = OFF
```

el sistema debe marcar:

```text
command_result = failed
```

o:

```text
state mismatch
```

---

# 176. Distributed Transactions

No se recomienda crear transacciones distribuidas complejas para operaciones normales.

Para escenas:

```text
Scene
 ↓
Commands
```

cada comando puede tener:

```text
correlation_id
```

pero la plataforma debe aceptar ejecución parcial cuando no sea posible garantizar atomicidad.

---

# 177. Scene Failure

Ejemplo:

```text
Scene Movie

Light A → success
Light B → success
Curtain → failed
```

El sistema debe conservar:

```text
partial success
```

y generar diagnóstico.

---

# 178. Atomic Operations

Sólo operaciones especialmente diseñadas podrán requerir atomicidad.

Ejemplo:

```text
Safety interlock
```

Estas operaciones deben estar explícitamente definidas y no asumirse por defecto.

---

# 179. System Bus Events

Eventos internos recomendados:

```text
node.online
node.offline
node.degraded

device.available
device.unavailable

entity.created
entity.updated
entity.removed
entity.state_changed

command.requested
command.accepted
command.executed
command.failed
command.timeout

configuration.updated
configuration.applied
configuration.failed

transport.connected
transport.disconnected

security.authentication_failed
security.authorization_denied
```

---

# 180. Reserved Names

Se deben reservar namespaces:

```text
system.*
node.*
device.*
entity.*
command.*
configuration.*
security.*
transport.*
diagnostic.*
```

Esto evita conflictos futuros.

---

# 181. Custom Events

Los módulos pueden definir:

```text
custom.*
```

o un namespace propio:

```text
greenhouse.*
marine.*
industrial.*
agriculture.*
```

Ejemplo:

```text
greenhouse.irrigation.started
```

---

# 182. Module Independence

Un módulo no debe modificar directamente otro módulo.

Debe comunicarse mediante:

```text
System Bus
+
Data Model
```

Ejemplo:

```text
Security Module
       ↓
event
       ↓
Lighting Module
```

---

# 183. Plugin Architecture

Los plugins pueden:

```text
publish events
subscribe events
send commands
register entities
register capabilities
```

pero deben utilizar las mismas reglas del bus.

---

# 184. Third-Party Extensions

Una aplicación externa puede:

```text
GET state
SUBSCRIBE event
SEND command
```

a través de la API.

La API puede convertir:

```text
REST
 ↓
System Bus
```

---

# 185. System Bus as Internal Contract

El System Bus se convierte así en el contrato común entre:

```text
Firmware
Central
API
Web UI
Automation Engine
Integrations
Plugins
```

Esto reduce acoplamiento.

---

# 186. Golden Architecture

La arquitectura final:

```text
                         ┌───────────────┐
                         │   WEB / APP   │
                         └───────┬───────┘
                                 │
                                API
                                 │
                         ┌───────▼───────┐
                         │    CENTRAL    │
                         │    ESP32-S3   │
                         └───────┬───────┘
                                 │
                          SYSTEM BUS
                                 │
               ┌─────────────────┼─────────────────┐
               │                 │                 │
            Ethernet           Wi-Fi             CAN
               │                 │                 │
            Zone A             Zone B            Zone C
               │                 │                 │
          ┌────┴────┐       ┌────┴────┐       ┌────┴────┐
          │         │       │         │       │         │
        Node      Node    Node      Node    Node      Node
          │         │       │         │       │         │
        Sensor    Relay   Sensor    Light   Motor     Sensor
```

---

# 187. Golden Rule

> **Ninguna aplicación debe depender directamente del transporte físico.**

La aplicación debe pensar:

```text
"enciende la luz del living"
```

y no:

```text
"activa GPIO 12 mediante MQTT".
```

---

# 188. Architecture Rule

> **El System Bus transporta intención y estado; los adaptadores traducen esa intención al medio físico correspondiente.**

---

# 189. Local-First Rule

> **El bus debe facilitar la coordinación distribuida, pero nunca convertir al Central en un punto único de fallo para las funciones críticas.**

---

# 190. Final Design Rule

La arquitectura completa queda:

```text
┌──────────────────────────────┐
│         USER / APP           │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│             API              │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│        AUTOMATION             │
│        FUNCTIONS              │
│        SCENES                 │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│         DATA MODEL            │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│         SYSTEM BUS            │
│                              │
│ Routing                      │
│ Priority                     │
│ ACK/NACK                     │
│ Retry                        │
│ Deduplication                │
│ Discovery                    │
│ Synchronization              │
│ Security                     │
│ Failover                     │
└──────────────┬───────────────┘
               ↓
┌──────────────────────────────┐
│      TRANSPORT ADAPTERS       │
├────────┬────────┬────────────┤
│Ethernet│ Wi-Fi  │ CAN / RS485│
│ MQTT   │ WS     │ Matter     │
└────────┴────────┴────────────┘
               ↓
┌──────────────────────────────┐
│        DEVICES / NODES        │
└──────────────────────────────┘
```

---

# 191. Estado del documento

```text
Estado: Diseño arquitectónico base

Definido:

- System Bus lógico
- separación Bus / Transport
- Message Envelope
- Message Types
- Command
- Event
- State
- ACK / NACK
- Request ID
- Correlation ID
- Addressing
- Routing
- Broadcast
- Multicast
- Priority
- TTL
- Retry
- Deduplication
- Idempotency
- QoS
- Offline Queue
- Discovery
- Provisioning
- Synchronization
- State Reconciliation
- Configuration Reconciliation
- Heartbeat
- Presence
- Health
- Security
- Authentication
- Authorization
- Replay Protection
- Transport Failover
- Multi-Transport
- FreeRTOS integration
- Queue architecture
- Backpressure
- Rate limiting
- Telemetry
- Control Plane
- Diagnostics
- Distributed tracing
- Local-first operation
- Central failure handling
- Zone failure handling
- Node failure handling

Pendiente:

- especificación binaria
- estructura definitiva de BusMessage
- protocolo HELLO
- protocolo DISCOVERY
- protocolo SYNC
- ACK/NACK formal
- routing protocol
- tablas de códigos de error
- políticas QoS definitivas
- seguridad criptográfica concreta
- adaptación CAN
- adaptación RS485
- adaptación Ethernet
- adaptación Wi-Fi
- adaptación MQTT
- integración Matter
- pruebas de carga
- pruebas de pérdida de paquetes
- pruebas de recuperación
```

---

# 192. Próximos documentos

Con:

```text
DATA-MODEL.md
        ↓
DATA-SCHEMAS.md
        ↓
SYSTEM-BUS.md
```

la siguiente capa recomendada es:

```text
API-SPECIFICATION.md
```

que definirá cómo aplicaciones, Web UI, móvil e integraciones externas interactúan con:

```text
Entities
States
Commands
Events
Devices
Zones
Groups
Scenes
Automations
```

Después:

```text
DISCOVERY-PROVISIONING.md
```

para definir cómo un ESP32 nuevo entra al sistema, se autentica, descubre sus recursos/capabilities, recibe configuración y pasa de `unknown` a `provisioned`.

Finalmente:

```text
EVENT-MODEL.md
CONFIGURATION-MODEL.md
DATABASE-STORAGE.md
AUTOMATION-ENGINE.md
TESTING-VALIDATION.md
```

---

# 193. Principio arquitectónico final

> **El System Bus es la columna vertebral lógica de la plataforma.**

> **No importa si un mensaje viaja por Ethernet, Wi-Fi, CAN, RS485, MQTT, Thread, Matter o cualquier transporte futuro: para la aplicación debe seguir siendo el mismo Command, Event, State o Configuration Message.**

> **La plataforma debe poder cambiar su infraestructura de comunicación sin tener que reescribir sus automatizaciones, entidades, funciones o integraciones.**

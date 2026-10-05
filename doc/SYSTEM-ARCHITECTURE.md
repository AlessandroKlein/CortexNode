# SYSTEM-ARCHITECTURE.md

# Arquitectura General del Sistema

> **Tipo:** Especificación de arquitectura / Visión del sistema
> **Estado:** Planificación / Diseño
> **Versión:** 1.0.0
> **Proyecto:** Plataforma Distribuida de Automatización
> **Base:** ESP32 + PlatformIO
> **Objetivo:** Automatización distribuida, escalable, modular y autónoma

---

# 1. Propósito

Este documento define la visión general de una plataforma de automatización distribuida capaz de funcionar como un sistema similar conceptualmente a un **PLC distribuido**, pero orientado a una arquitectura moderna, modular, conectada y escalable.

El sistema debe poder utilizarse en:

* Viviendas.
* Oficinas.
* Edificios.
* Comercios.
* Hoteles.
* Talleres.
* Pequeñas industrias.
* Invernaderos.
* Instalaciones agrícolas.
* Sistemas energéticos.
* Sistemas de seguridad.
* Sistemas marinos.
* Automatización de máquinas.
* Instalaciones híbridas.

La plataforma no debe estar diseñada alrededor de un único tipo de instalación.

Debe existir un **núcleo común** sobre el cual puedan construirse diferentes aplicaciones.

---

# 2. Idea principal

La plataforma debe funcionar como una **red distribuida de dispositivos inteligentes**.

No se debe pensar:

```text
ESP32 → sensor
ESP32 → relay
ESP32 → pantalla
```

sino:

```text
                  SISTEMA
                     │
          ┌──────────┴──────────┐
          │                     │
       ZONAS                  SERVICIOS
          │                     │
    ┌─────┼─────┐        ┌──────┼──────┐
    │     │     │        │      │      │
  Nodo  Nodo  Nodo     Alarmas Energía API
    │     │     │
 Sensores Actuadores IO
```

Cada dispositivo forma parte de un sistema común.

Los dispositivos deben:

* Descubrirse.
* Identificarse.
* Comunicar su capacidad.
* Publicar estados.
* Recibir comandos.
* Ejecutar acciones localmente.
* Generar eventos.
* Participar en automatizaciones.
* Detectar fallos.
* Informar su estado.
* Poder funcionar sin depender permanentemente del servidor central.

---

# 3. Principio fundamental: Local First

La plataforma debe seguir el principio:

> **El servidor central puede coordinar el sistema, pero no debe ser necesario para que las funciones críticas funcionen.**

Por ejemplo:

```text
Sensor PIR
   │
   ▼
Nodo de seguridad
   │
   ├── Detecta movimiento
   │
   ├── Evalúa modo actual
   │
   ├── Activa alarma
   │
   └── Enciende iluminación
```

Esto debe poder ocurrir aunque:

```text
Servidor central = OFFLINE
Internet = OFFLINE
Cloud = OFFLINE
```

El servidor puede agregar:

* Historial.
* Estadísticas.
* Interfaces.
* Configuración.
* Integraciones.
* Inteligencia avanzada.
* Backup.
* Supervisión.

Pero no debe convertirse en un punto único de fallo.

---

# 4. Sistema distribuido

Cada nodo debe tener cierta capacidad de decisión.

La inteligencia puede distribuirse en diferentes niveles:

```text
NIVEL 1
Dispositivo
│
├── lectura
├── filtrado
├── seguridad
└── acciones inmediatas

NIVEL 2
Zona / Gateway
│
├── coordinación
├── automatizaciones
└── reglas locales

NIVEL 3
Servidor central
│
├── administración
├── histórico
├── análisis
├── configuración
└── coordinación global

NIVEL 4
Servicios externos
│
├── Home Assistant
├── Homey
├── Matter
├── Cloud
└── APIs externas
```

No todas las instalaciones necesitan utilizar todos los niveles.

---

# 5. Modelo conceptual

La arquitectura debe separar claramente los siguientes conceptos:

```text
DISPOSITIVO
      │
      ▼
RECURSO
      │
      ▼
ZONA
      │
      ▼
GRUPO
      │
      ▼
FUNCIÓN
      │
      ▼
ESCENA
      │
      ▼
AUTOMATIZACIÓN
      │
      ▼
SISTEMA
```

Esto es fundamental para evitar que la lógica dependa directamente del hardware.

---

# 6. Dispositivo

Un dispositivo representa una unidad física.

Ejemplos:

```text
ESP32-WROOM
ESP32-S3
ESP32-C6
ESP32-C5
Gateway Ethernet
Pantalla táctil
Nodo CAN
Nodo RS485
```

Un dispositivo puede contener muchos recursos.

Por ejemplo:

```text
ESP32-S3
│
├── Sensor temperatura
├── Sensor humedad
├── PIR
├── Relé 1
├── Relé 2
├── PWM
├── Pantalla
├── Cámara
└── Wi-Fi
```

---

# 7. Recursos

Un recurso representa una capacidad lógica.

Ejemplos:

```text
temperature
humidity
motion
light
relay
switch
power
energy
water_flow
pressure
door
window
camera
display
speaker
```

El sistema debe trabajar principalmente con recursos y no con GPIO.

Por ejemplo:

Incorrecto:

```text
GPIO 14 = ON
```

Correcto:

```text
LIVING_ROOM.LIGHT_MAIN = ON
```

El sistema se encarga de traducir:

```text
LIGHT_MAIN
      ↓
Recurso lógico
      ↓
Dispositivo
      ↓
GPIO 14
```

---

# 8. Zonas

Una zona es una agrupación lógica de recursos.

Ejemplos:

```text
Casa
├── Entrada
├── Living
├── Cocina
├── Dormitorio 1
├── Dormitorio 2
├── Baño
├── Garage
└── Exterior
```

Una zona no tiene que corresponder a un dispositivo físico.

Por ejemplo:

```text
Dormitorio 1
│
├── ESP32 #01
│   ├── PIR
│   └── Luz
│
├── ESP32 #02
│   └── Temperatura
│
└── ESP32-S3
    └── Pantalla
```

Todos pueden pertenecer a la misma zona.

---

# 9. Grupos

Los grupos permiten controlar recursos independientemente de su ubicación física.

Ejemplo:

```text
GRUPO: Todas las luces

├── Living
├── Cocina
├── Dormitorio 1
├── Dormitorio 2
├── Exterior
└── Garage
```

Otro ejemplo:

```text
GRUPO: Iluminación exterior

├── Entrada
├── Garage
├── Patio
└── Jardín
```

Esto permite crear acciones como:

```text
Apagar todas las luces
```

sin importar qué dispositivos físicos las controlan.

---

# 10. Funciones

Una función representa una capacidad del sistema.

Ejemplos:

```text
Iluminación
Seguridad
Climatización
Ventilación
Riego
Energía
Control de acceso
Audio
Video
Medición
Alarmas
```

Una función puede utilizar múltiples dispositivos.

Ejemplo:

```text
FUNCIÓN: Seguridad exterior

├── PIR exterior 1
├── PIR exterior 2
├── Sensor puerta
├── Sensor ventana
├── Sirena
├── Cámara
└── Iluminación exterior
```

---

# 11. Sectores de seguridad

La seguridad debe estar basada en sectores lógicos.

Ejemplo:

```text
SEGURIDAD

Exterior
├── Patio
├── Jardín
├── Garage
└── Entrada

Interior
├── Living
├── Cocina
└── Pasillo

Dormitorios
├── Dormitorio 1
├── Dormitorio 2
└── Dormitorio 3
```

Cada sector puede tener diferentes comportamientos.

---

# 12. Modos del sistema

El sistema debe tener estados globales o perfiles.

Ejemplos:

```text
DISARMED
ARMED
NIGHT
AWAY
VACATION
MAINTENANCE
EMERGENCY
```

También pueden existir modos personalizados.

Por ejemplo:

```text
HOME.DAY
HOME.NIGHT
HOME.SLEEP
HOME.AWAY
HOME.VACATION
```

---

# 13. Ejemplo: modo dormir

Este ejemplo es fundamental para la arquitectura.

Supongamos que existen:

```text
PIR Dormitorio
PIR Pasillo
PIR Exterior
Sensor ventana
```

Durante el día:

```text
PIR Dormitorio
    ↓
Movimiento
    ↓
Encender luz dormitorio
```

Pero durante el modo:

```text
SYSTEM.MODE = SLEEP
```

el comportamiento cambia.

### Dormitorio

```text
PIR
 ↓
Movimiento
 ↓
Registrar evento
 ↓
NO encender luz principal
```

### Pasillo

```text
PIR
 ↓
Movimiento
 ↓
Encender luz al 15 %
 ↓
Apagar después de X segundos
```

### Exterior

```text
PIR
 ↓
Movimiento
 ↓
Activar iluminación exterior
 ↓
Registrar evento
 ↓
Evaluar alarma
```

### Ventana

```text
Ventana abierta
      ↓
Modo SLEEP
      ↓
Sector armado
      ↓
ALARMA
```

La lógica no debe estar programada directamente dentro del PIR.

El PIR solamente debe informar:

```text
motion.detected
```

El sistema decide qué hacer.

---

# 14. Mallas de sensores

Los dispositivos de seguridad deben poder formar una red lógica.

No necesariamente una única malla física.

Debe existir una:

> **Malla lógica de seguridad**

que pueda utilizar diferentes tecnologías físicas.

Por ejemplo:

```text
                 SECURITY NETWORK
                        │
       ┌────────────────┼────────────────┐
       │                │                │
     Wi-Fi            Thread            CAN
       │                │                │
    ESP32             ESP32-C6         ESP32
       │                │                │
      PIR              PIR             Door
```

Para el sistema todos pueden aparecer como:

```text
SECURITY.SENSOR.*
```

La capa superior no debería preocuparse por el transporte.

---

# 15. Redundancia

La plataforma debe permitir que diferentes sensores cubran una misma zona.

Ejemplo:

```text
Sector: Garage

PIR 1
PIR 2
Sensor puerta
Cámara
```

Si uno falla:

```text
PIR 1 = OFFLINE
```

los demás continúan funcionando.

El sistema puede generar:

```text
WARNING:
Garage motion sensor offline
```

sin desactivar automáticamente toda la seguridad.

---

# 16. Correlación de sensores

El sistema puede combinar diferentes eventos.

Ejemplo:

```text
PIR detecta movimiento
+
Puerta abierta
+
Cámara detecta persona
```

puede producir:

```text
SECURITY.CONFIRMED_INTRUSION
```

Mientras que:

```text
PIR detecta movimiento
```

solo puede producir:

```text
SECURITY.MOTION_DETECTED
```

Esto permite reducir falsos positivos.

---

# 17. Motor de reglas

Las reglas deben seguir el modelo:

```text
TRIGGER
   ↓
CONDITIONS
   ↓
ACTIONS
```

Ejemplo:

```text
TRIGGER:
Motion detected

CONDITIONS:
Mode = NIGHT
Zone = Hallway

ACTION:
Light = 15 %
```

Otro:

```text
TRIGGER:
Window opened

CONDITIONS:
Security mode = AWAY

ACTIONS:
Alarm = ON
Light exterior = ON
Notification = SEND
Camera = RECORD
```

---

# 18. Automatizaciones distribuidas

Una automatización puede ejecutarse:

### Localmente

```text
PIR → ESP32 → Relay
```

### En una zona

```text
PIR 1
PIR 2
Door
   ↓
Zone Controller
   ↓
Alarm
```

### En servidor

```text
Multiple zones
      ↓
Central Automation Engine
      ↓
Global actions
```

La ubicación de ejecución debe ser configurable.

---

# 19. Prioridad de automatizaciones

Las automatizaciones deben tener prioridades.

Ejemplo:

```text
PRIORIDAD 0
Emergencia

PRIORIDAD 1
Seguridad

PRIORIDAD 2
Protección de equipos

PRIORIDAD 3
Automatizaciones normales

PRIORIDAD 4
Confort

PRIORIDAD 5
Estadísticas
```

Por ejemplo:

```text
Control de temperatura
```

no debe impedir:

```text
Alarma de incendio
```

---

# 20. Escenas

Una escena representa un estado deseado.

Ejemplo:

## Salir de casa

```text
Lights = OFF
HVAC = ECO
Security = ARMED
Garage = CLOSED
Water valves = SAFE
```

## Llegar a casa

```text
Security = DISARMED
Entrance light = ON
Hallway light = ON
HVAC = COMFORT
```

## Dormir

```text
Security = NIGHT
Main lights = OFF
Hallway = 15 %
Exterior = Armed
Bedroom PIR = Monitoring
```

---

# 21. Rutinas

Las rutinas permiten ejecutar secuencias.

Ejemplo:

```text
Rutina: Preparar casa para dormir

1. Apagar luces principales
2. Esperar 2 segundos
3. Cerrar persianas
4. Esperar 5 segundos
5. Activar sector exterior
6. Activar sector interior
7. Cambiar modo a SLEEP
```

Una rutina puede incluir:

* Delays.
* Condiciones.
* Repeticiones.
* Branches.
* Timeouts.
* Reintentos.
* Acciones paralelas.
* Acciones secuenciales.

---

# 22. Bus lógico del sistema

La plataforma debe disponer de un **System Bus lógico**.

La aplicación no debería saber si un mensaje viaja mediante:

```text
Wi-Fi
Ethernet
CAN
RS485
Thread
Zigbee
Matter
BLE
```

Por ejemplo:

```text
publishEvent(
    "living_room.motion.detected"
)
```

La capa de comunicación determina cómo transportar ese evento.

Esto permite cambiar hardware sin reescribir la aplicación.

---

# 23. Ejemplo de abstracción

Una automatización:

```text
SI
    living_room.motion == true

Y
    system.mode == NIGHT

ENTONCES
    hallway.light = 15%
```

No debería contener:

```text
GPIO 23
GPIO 14
I2C address 0x20
CAN node 4
MQTT topic...
```

Esos detalles pertenecen a capas inferiores.

---

# 24. Descubrimiento automático

Cuando se conecta un dispositivo:

```text
Device joins network
       ↓
DISCOVERY
       ↓
Identity
       ↓
Capabilities
       ↓
Resources
       ↓
Services
       ↓
Configuration
       ↓
Available
```

Ejemplo:

```json
{
  "device_id": "esp32s3_001",
  "model": "ESP32-S3",
  "capabilities": [
    "temperature",
    "humidity",
    "motion",
    "display",
    "camera"
  ]
}
```

---

# 25. Configuración sin recompilar

El usuario debe poder modificar desde la interfaz:

* GPIO.
* Sensores.
* Actuadores.
* Zonas.
* Grupos.
* Nombres.
* Unidades.
* Direcciones Modbus.
* CAN.
* I2C.
* Pantallas.
* Widgets.
* Automatizaciones.
* Escenas.
* Modos.
* Permisos.

No debería ser necesario modificar:

```text
.cpp
.h
platformio.ini
```

para una configuración normal.

---

# 26. Hardware desacoplado

Ejemplo:

```text
Luz Living
```

puede estar conectada a:

```text
ESP32 GPIO
```

o:

```text
MCP23017
```

o:

```text
RS485 relay
```

o:

```text
CAN actuator
```

o:

```text
Zigbee actuator
```

La aplicación debe seguir viendo:

```text
living_room.light
```

---

# 27. Pantallas táctiles

Las pantallas deben ser nodos del sistema.

Ejemplo:

```text
ESP32-S3
│
├── Touch
├── Display
└── Network
```

Una pantalla puede mostrar:

```text
Temperatura
Humedad
Luces
Persianas
Seguridad
Música
Escenas
Cámaras
Energía
```

Los elementos mostrados deben ser configurables.

---

# 28. UI modular

La interfaz debe utilizar bloques.

Ejemplo:

```text
PANTALLA LIVING

┌──────────────────────┐
│ Temperatura          │
├──────────────────────┤
│ Humedad              │
├──────────────────────┤
│ Luces                │
├──────────────────────┤
│ Música               │
├──────────────────────┤
│ Seguridad            │
└──────────────────────┘
```

Los bloques deben poder:

* Instalarse.
* Eliminarse.
* Habilitarse.
* Deshabilitarse.
* Reordenarse.
* Configurarse.
* Compartirse entre pantallas.

---

# 29. Energía

La plataforma debe tratar la energía como un recurso universal.

Modelo:

```text
Voltage
Current
Power
Energy
Frequency
Power Factor
```

Ejemplo:

```text
Casa
│
├── General
├── Cocina
├── HVAC
├── Iluminación
├── Garage
└── Solar
```

Esto permite construir:

```text
Power monitoring
Load management
Energy alarms
Consumption history
Solar management
```

---

# 30. Agua

El agua también debe ser un recurso universal.

Ejemplo:

```text
Flow
Pressure
Temperature
Total volume
Valve state
Leak detection
```

Puede utilizar:

* YF.
* DN.
* Sensores industriales.
* Modbus.
* Pulsos.
* Analógicos.

---

# 31. Modbus

Modbus debe ser tratado como un protocolo de integración.

Soportar:

```text
Modbus RTU
Modbus TCP
```

Ejemplos:

* Inversores.
* Medidores.
* Variadores.
* Sensores industriales.
* Controladores.
* Equipos HVAC.

Los registros deben poder mapearse a recursos lógicos.

Ejemplo:

```text
Modbus Register 40001
        ↓
Energy.Voltage
```

---

# 32. CAN / CANopen

CAN debe permitir integrar dispositivos industriales y comerciales.

Ejemplo:

```text
CAN Bus
│
├── Node 1
├── Node 2
├── Node 3
└── Gateway
```

CANopen debe contemplar:

* Node ID.
* Object Dictionary.
* PDO.
* SDO.
* Heartbeat.
* Emergency.
* NMT.

Los dispositivos CAN deben aparecer en el sistema mediante el mismo modelo lógico.

---

# 33. Integraciones externas

La plataforma debe funcionar como sistema independiente pero poder integrarse con:

```text
Home Assistant
Homey Pro
Apple Home
Google Home
Samsung SmartThings
MQTT
Matter
REST API
WebSocket
```

No se debe diseñar el núcleo alrededor de una sola plataforma externa.

---

# 34. API

El sistema debe exponer una API.

Ejemplo conceptual:

```text
GET /api/devices
GET /api/zones
GET /api/resources
GET /api/scenes
GET /api/automations
GET /api/events
GET /api/system
```

Y comandos:

```text
POST /api/devices/{id}/command
POST /api/scenes/{id}/activate
POST /api/automations/{id}/enable
```

---

# 35. Eventos

Los eventos son fundamentales.

Ejemplos:

```text
motion.detected
door.opened
door.closed
temperature.changed
power.overload
water.leak
alarm.triggered
device.offline
device.online
button.pressed
```

Los eventos deben poder ser consumidos por:

* Automatizaciones.
* UI.
* Logs.
* APIs.
* Integraciones.
* Otros dispositivos.

---

# 36. Estados

Cada recurso debe tener un estado.

Ejemplo:

```text
ONLINE
OFFLINE
UNKNOWN
ERROR
DISABLED
MAINTENANCE
```

Además de su valor:

```text
light = ON
temperature = 23.4
door = CLOSED
```

---

# 37. Heartbeat

Los nodos deben poder anunciar periódicamente:

```text
I'm alive
```

Ejemplo:

```text
Device
 ↓
Heartbeat
 ↓
Gateway
 ↓
Central
```

Si deja de responder:

```text
ONLINE
 ↓
TIMEOUT
 ↓
OFFLINE
```

Esto permite detectar fallos.

---

# 38. Store and Forward

Los dispositivos importantes deben poder almacenar eventos temporalmente.

Ejemplo:

```text
Internet OFFLINE

Sensor
 ↓
Evento
 ↓
Memoria local

Internet ONLINE
 ↓
Sincronización
```

Esto es especialmente importante para:

* Energía.
* Seguridad.
* Alarmas.
* Meteorología.
* Producción.
* Históricos.

---

# 39. Seguridad distribuida

La seguridad no debe depender únicamente del servidor.

Cada nodo debe poder validar:

* Mensajes.
* Origen.
* Destino.
* Permisos.
* Integridad.
* Autenticación.

Los dispositivos críticos deben poder continuar protegiendo la instalación aunque el servidor esté desconectado.

---

# 40. Roles y permisos

El sistema debe soportar diferentes usuarios.

Ejemplo:

```text
ADMIN
INSTALLER
OPERATOR
USER
GUEST
API
SERVICE
```

Ejemplo:

```text
USER
├── controlar luces
├── ver sensores
└── activar escenas

INSTALLER
├── configuración hardware
├── redes
├── módulos
└── diagnóstico

ADMIN
├── todo
├── usuarios
└── seguridad
```

---

# 41. Inteligencia artificial

La IA debe ser una capa opcional.

No debe ser necesaria para las funciones básicas.

Puede utilizarse para:

* Detección de personas.
* Reconocimiento de objetos.
* Predicción de consumo.
* Predicción meteorológica.
* Detección de anomalías.
* Aprendizaje de hábitos.
* Optimización energética.
* Mantenimiento predictivo.

Ejemplo:

```text
ESP32-S3
   │
   ├── Cámara
   ├── Sensores
   └── IA local
          ↓
       Evento
          ↓
     System Bus
```

---

# 42. Aprendizaje de hábitos

El sistema podría aprender comportamientos.

Ejemplo:

```text
Todos los días:

18:30 → luz living
19:00 → TV
23:00 → luces OFF
23:15 → modo SLEEP
```

El sistema puede detectar patrones y sugerir:

```text
¿Quieres crear una automatización
basada en este comportamiento?
```

La creación automática debe ser opcional.

---

# 43. Inteligencia híbrida

La IA puede ejecutarse en:

```text
ESP32-S3
        ↓
Gateway
        ↓
Servidor local
        ↓
Cloud
```

dependiendo de los recursos disponibles.

Por ejemplo:

```text
Detección simple
→ ESP32-S3

Modelo pesado
→ servidor local

Análisis avanzado
→ cloud
```

---

# 44. Escalabilidad

La plataforma debe funcionar desde:

### Instalación pequeña

```text
1 ESP32
5 sensores
3 relés
```

hasta:

### Instalación grande

```text
Servidor
│
├── Gateway Ethernet
├── Gateway CAN
├── Gateway RS485
├── Gateway Thread
├── Gateway Zigbee
│
├── Zona 1
│   ├── 20 nodos
│
├── Zona 2
│   ├── 30 nodos
│
└── Zona 3
    └── 50 nodos
```

La arquitectura lógica no debe cambiar.

---

# 45. Jerarquía de red

Para instalaciones grandes se recomienda:

```text
                 CORE
                  │
        ┌─────────┼─────────┐
        │         │         │
     Gateway   Gateway   Gateway
        │         │         │
      Zona A    Zona B    Zona C
        │         │         │
      Nodes     Nodes     Nodes
```

Esto permite:

* Aislamiento.
* Escalabilidad.
* Diagnóstico.
* Redundancia.
* Distribución de carga.

---

# 46. Gateways

Un gateway traduce entre tecnologías.

Ejemplos:

```text
CAN → Ethernet
RS485 → Ethernet
Zigbee → MQTT
Thread → IP
Matter → System Bus
```

Pero el sistema superior no debe depender de la tecnología.

---

# 47. Tolerancia a fallos

El sistema debe asumir que los dispositivos fallarán.

Posibles fallos:

```text
Sensor offline
Nodo offline
Gateway offline
Wi-Fi offline
Ethernet offline
Internet offline
Servidor offline
Cloud offline
```

El sistema debe definir qué sucede en cada caso.

---

# 48. Degradación controlada

Ejemplo:

```text
Servidor OFFLINE
        ↓
Automatizaciones locales
        ↓
CONTINÚAN
```

Si además falla un gateway:

```text
Gateway OFFLINE
        ↓
Nodos locales
        ↓
Funciones críticas continúan
```

El objetivo es evitar:

```text
un fallo → caída completa del sistema
```

---

# 49. Prioridad de comunicaciones

No todos los mensajes son iguales.

### Críticos

```text
ALARM
EMERGENCY
SAFETY
```

### Importantes

```text
COMMAND
STATE
DEVICE_OFFLINE
```

### Normales

```text
SENSOR_UPDATE
```

### Baja prioridad

```text
STATISTICS
LOG
HISTORY
```

Esto permite gestionar redes grandes.

---

# 50. Configuración de instalación

La instalación completa debe poder representarse como datos.

Conceptualmente:

```text
SITE
│
├── DEVICES
├── ZONES
├── GROUPS
├── RESOURCES
├── USERS
├── SECURITY
├── SCENES
├── AUTOMATIONS
└── INTEGRATIONS
```

Esto permite realizar:

```text
Export
Import
Backup
Restore
Clone
Migration
```

---

# 51. Plantillas

Las instalaciones comunes deberían poder utilizar plantillas.

Ejemplo:

```text
Casa estándar
```

incluye:

```text
Entrada
Living
Cocina
2 dormitorios
Baño
Exterior
Garage
```

Con automatizaciones predeterminadas.

Esto reduce el tiempo de instalación.

---

# 52. Sistema de plugins

La plataforma debe permitir agregar módulos sin modificar el núcleo.

Ejemplo:

```text
modules/
│
├── temperature/
├── humidity/
├── pir/
├── relay/
├── energy/
├── water/
├── modbus/
├── canopen/
├── camera/
├── display/
├── security/
└── ai/
```

Cada módulo debe tener:

```text
Manifest
Configuration
Driver
Logical model
Events
Commands
UI
Documentation
Tests
```

---

# 53. Compatibilidad de hardware

La plataforma debe soportar diferentes familias de ESP32.

Por ejemplo:

```text
ESP32-WROOM
ESP32-S3
ESP32-C6
ESP32-C5
```

Cada familia puede utilizarse donde resulte más conveniente.

### ESP32-WROOM

Ideal para:

* Ethernet.
* GPIO.
* RS485.
* CAN.
* Sensores.
* Relés.
* Nodos económicos.

### ESP32-C6

Ideal para:

* Matter.
* Thread.
* Zigbee.
* Wi-Fi 6.
* Gateways inalámbricos.

### ESP32-S3

Ideal para:

* Pantallas.
* Cámaras.
* IA.
* Audio.
* Interfaces avanzadas.
* Procesamiento local.

---

# 54. PlatformIO

Todo el ecosistema debe ser desarrollable mediante PlatformIO.

Debe existir una arquitectura común para:

```text
Core
HAL
Device Model
Communication
Modules
UI
Storage
Security
Automation
Diagnostics
```

El hardware específico debe quedar aislado.

---

# 55. Separación de capas

La estructura conceptual debe ser:

```text
┌─────────────────────────────┐
│          UI / API           │
├─────────────────────────────┤
│    Automation / Scenes      │
├─────────────────────────────┤
│        Services             │
├─────────────────────────────┤
│       Device Model          │
├─────────────────────────────┤
│      System Bus             │
├─────────────────────────────┤
│      Communication          │
├─────────────────────────────┤
│          HAL                │
├─────────────────────────────┤
│         Hardware            │
└─────────────────────────────┘
```

Una capa superior no debería depender directamente de una inferior.

---

# 56. Regla de oro

Una automatización debería poder ejecutarse sin saber:

* Qué ESP32 se utiliza.
* Qué GPIO se utiliza.
* Qué protocolo físico se utiliza.
* Qué sensor específico existe.
* Qué fabricante fabricó el sensor.

Debe saber únicamente:

```text
qué recurso necesita
qué condición debe cumplirse
qué acción debe realizar
```

---

# 57. Ejemplo completo

Supongamos:

```text
Dormitorio
```

Tiene:

```text
ESP32-S3
├── PIR
├── temperatura
├── humedad
└── pantalla

ESP32-WROOM
└── luz

ESP32-C6
└── sensor ventana
```

Todos pertenecen a:

```text
ZONE = BEDROOM_01
```

Y la zona pertenece a:

```text
SECURITY_SECTOR = BEDROOMS
```

Durante el día:

```text
PIR
 ↓
Motion
 ↓
Light ON
```

Durante la noche:

```text
SYSTEM.MODE = SLEEP
```

el mismo evento produce:

```text
PIR
 ↓
Motion
 ↓
Bedroom rule
 ↓
NO main light
```

Mientras:

```text
Hallway PIR
 ↓
Motion
 ↓
Hallway light 15%
```

Y:

```text
Window sensor
 ↓
Window opened
 ↓
Bedrooms sector
 ↓
Night mode
 ↓
Alarm evaluation
```

No se modificó ningún firmware.

Solamente cambió:

```text
SYSTEM.MODE
```

---

# 58. Sistema orientado a eventos

La arquitectura debe priorizar eventos sobre polling constante cuando sea posible.

Ejemplo:

```text
Sensor
 ↓
EVENT
 ↓
System Bus
 ↓
Subscribers
```

En lugar de:

```text
Server
 ↓
¿Cambió el sensor?
 ↓
¿Cambió?
 ↓
¿Cambió?
```

Esto reduce:

* Tráfico.
* CPU.
* Latencia.
* Consumo energético.

---

# 59. Tiempo

El sistema debe disponer de una referencia temporal común.

Debe soportar:

```text
NTP
RTC
Unix timestamp
Timezone
DST cuando corresponda
```

Los eventos deben poder registrar:

```text
timestamp
source
sequence
```

---

# 60. Diagnóstico

Cada nodo debe poder informar:

```text
CPU
RAM
Flash
Uptime
Temperature
Network
Signal
Errors
Restart reason
Watchdog
Modules
Sensors
```

Esto permitirá crear una pantalla:

```text
SYSTEM HEALTH
```

---

# 61. Actualizaciones OTA

La plataforma debe soportar:

```text
Firmware version
Hardware compatibility
Module compatibility
Configuration migration
Rollback
```

Idealmente:

```text
New firmware
      ↓
Compatibility check
      ↓
Download
      ↓
Verify
      ↓
Install
      ↓
Health check
      ↓
Rollback if failed
```

---

# 62. Versionado

Todos los elementos importantes deberían tener versión:

```text
Firmware
Protocol
Device Model
Module
Configuration
API
Schema
```

Esto permite evolucionar el sistema sin romper instalaciones existentes.

---

# 63. Migración de configuración

Si cambia el modelo de configuración:

```text
v1
 ↓
Migration
 ↓
v2
```

no debería ser necesario configurar nuevamente toda la instalación.

---

# 64. Seguridad desde el diseño

La seguridad no debe agregarse al final.

Debe contemplarse desde el comienzo:

```text
Authentication
Authorization
Encryption
Integrity
Secure OTA
Secrets management
Audit logs
Network isolation
Role-based access
```

---

# 65. Filosofía de diseño

La plataforma debe seguir estos principios:

### 1. Local First

Las funciones críticas deben funcionar localmente.

### 2. Distributed by Design

La inteligencia puede distribuirse.

### 3. Hardware Agnostic

La lógica no debe depender del hardware.

### 4. Modular

Las funcionalidades deben poder instalarse y eliminarse.

### 5. Configurable

El usuario debe poder configurar sin recompilar.

### 6. Observable

Todo nodo debe poder informar su estado.

### 7. Secure by Design

La seguridad debe formar parte del núcleo.

### 8. Fail Safe

Los fallos deben producir una degradación controlada.

### 9. Scalable

La misma arquitectura debe servir para una casa y una instalación grande.

### 10. Open Integration

Debe ser posible integrarse con sistemas externos.

---

# 66. Evolución futura

La arquitectura debe permitir incorporar posteriormente:

```text
Machine Learning
Computer Vision
Voice Control
Digital Twin
Predictive Maintenance
Energy Optimization
Industrial Automation
Building Management
Agriculture
Marine Automation
Robotics
```

sin rediseñar el núcleo.

---

# 67. Relación con SEMA

SEMA puede convertirse posteriormente en una aplicación especializada dentro de esta plataforma.

Por ejemplo:

```text
Universal Automation Platform
             │
             ├── SEMA
             │    ├── Weather
             │    ├── Environment
             │    └── Meteorological API
             │
             ├── Invernadero
             │    ├── Climate
             │    ├── Irrigation
             │    └── Lighting
             │
             ├── Security
             │
             ├── Energy
             │
             └── Building Automation
```

SEMA no tendría que reinventar:

* Comunicación.
* Usuarios.
* OTA.
* Device Model.
* API.
* Web UI.
* Discovery.
* Storage.
* Seguridad.

Utilizaría el núcleo común.

---

# 68. Relación con Invernadero

El proyecto Invernadero puede evolucionar de la misma manera.

Por ejemplo:

```text
Universal Platform
        │
        └── Agriculture
              │
              └── Greenhouse
                    ├── Temperature
                    ├── Humidity
                    ├── CO2
                    ├── pH
                    ├── EC
                    ├── Irrigation
                    ├── Lighting
                    └── Ventilation
```

Esto permite reutilizar componentes entre proyectos.

---

# 69. Arquitectura final conceptual

La visión completa puede resumirse:

```text
                         PLATFORM
                             │
              ┌──────────────┼──────────────┐
              │              │              │
           SECURITY        ENERGY       ENVIRONMENT
              │              │              │
              └──────────────┼──────────────┘
                             │
                       SYSTEM BUS
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
     Ethernet              Wi-Fi                CAN
        │                    │                    │
      RS485               Thread              Modbus
        │                    │                    │
     Zigbee               Matter                BLE
        │                    │                    │
        └────────────────────┼────────────────────┘
                             │
                       DEVICE MODEL
                             │
              ┌──────────────┼──────────────┐
              │              │              │
            SENSORS       ACTUATORS       UI
              │              │              │
              └──────────────┼──────────────┘
                             │
                         HARDWARE
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
       ESP32-WROOM        ESP32-C6          ESP32-S3
          │                  │                  │
       Ethernet          Matter/Thread       Camera/AI
       RS485             Zigbee              Display
       CAN               Wi-Fi 6             Touch
```

---

# 70. Objetivo final

El objetivo no es crear simplemente una colección de firmware para ESP32.

El objetivo es crear una:

> **Plataforma distribuida de automatización programable, modular, escalable y autónoma.**

Una instalación debería poder construirse combinando:

```text
Nodos
+
Sensores
+
Actuadores
+
Gateways
+
Zonas
+
Funciones
+
Escenas
+
Automatizaciones
+
Interfaces
+
Integraciones
```

sin modificar el núcleo del sistema.

---

# 71. Regla arquitectónica definitiva

Toda nueva funcionalidad debe responder primero:

```text
¿Es un recurso?
¿Es un módulo?
¿Es un servicio?
¿Es una función?
¿Es una automatización?
¿Es un protocolo?
¿Es una integración?
```

y posteriormente:

```text
¿Cómo se implementa físicamente?
```

Nunca al revés.

La plataforma debe diseñarse primero desde la lógica del sistema y posteriormente mapear esa lógica al hardware.

---

# 72. Documentos relacionados

Esta especificación debe utilizarse conjuntamente con:

```text
README.md

docs/
├── SYSTEM-ARCHITECTURE.md
├── ARCHITECTURE.md
├── DEVICE-MODEL.md
├── COMMUNICATION.md
├── MODULE-DEVELOPMENT.md
├── SECURITY.md
├── AUTOMATION.md
├── API.md
├── UI-ARCHITECTURE.md
└── ADR/
```

Estos documentos deberán evolucionar conjuntamente a medida que avance el proyecto.

---

# 73. Próxima etapa de diseño

Antes de comenzar a desarrollar módulos concretos, se recomienda definir:

1. **System Bus Specification**
2. **Device/Resource JSON Schema**
3. **Event Specification**
4. **Command Specification**
5. **Automation DSL**
6. **Zone/Group/Sector Model**
7. **Security Model**
8. **Discovery Protocol**
9. **Node Identity**
10. **Configuration Schema**
11. **API Specification**
12. **MQTT Topic Convention**
13. **State Machine**
14. **Error/Diagnostic Model**
15. **Firmware Update Model**
16. **Module Manifest Specification**
17. **UI Widget Specification**
18. **Persistence Model**
19. **Network Topology**
20. **Failover Strategy**

Estos elementos constituyen la siguiente capa de especificación necesaria antes de implementar el núcleo definitivo.

---

# 74. Visión resumida

La plataforma debe permitir construir sistemas donde:

```text
UN SENSOR
    ↓
puede generar un EVENTO

UN EVENTO
    ↓
puede activar una AUTOMATIZACIÓN

UNA AUTOMATIZACIÓN
    ↓
puede controlar múltiples DISPOSITIVOS

LOS DISPOSITIVOS
    ↓
pueden estar en diferentes ZONAS

LAS ZONAS
    ↓
pueden pertenecer a diferentes SECTORES

LOS SECTORES
    ↓
pueden cambiar su comportamiento según el MODO

EL MODO
    ↓
puede modificar todo el comportamiento de la instalación

EL SERVIDOR
    ↓
puede coordinar y administrar

PERO LOS NODOS
    ↓
siguen siendo capaces de funcionar de manera autónoma.
```

Esta separación constituye el fundamento de toda la plataforma.

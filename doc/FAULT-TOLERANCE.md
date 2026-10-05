# FAULT-TOLERANCE.md

# Tolerancia a Fallos y Funcionamiento Autónomo

> **Tipo:** Especificación de arquitectura
> **Estado:** Planificación / Diseño
> **Versión:** 1.0.0
> **Objetivo:** Garantizar el funcionamiento de la automatización ante pérdida de Internet, servidor central, gateway, red o dispositivos individuales.

---

# 1. Objetivo

La plataforma debe diseñarse bajo un principio fundamental:

> **Ningún componente externo debe convertirse en un punto único de fallo para las funciones críticas de automatización.**

Por lo tanto:

```text
Internet OFFLINE
        ≠
Casa OFFLINE
```

y:

```text
Servidor central OFFLINE
        ≠
Automatización OFFLINE
```

La instalación debe continuar funcionando con el mayor nivel de autonomía posible.

---

# 2. Principio fundamental

La arquitectura debe cumplir:

> **Internet es opcional. El servidor central es opcional (Explicación completa de Servidor Central en `docs/CENTRAL-ARCHITECTURE.md`). La automatización local es obligatoria.**

Esto significa que:

### Internet

Puede desaparecer completamente.

Deben continuar funcionando:

* Luces.
* Interruptores.
* Sensores.
* Alarmas.
* Climatización.
* Persianas.
* Riego.
* Automatizaciones.
* Escenas.
* Rutinas.
* Seguridad.
* Control energético local.

Lo que puede dejar de funcionar son las funciones que realmente dependen de Internet.

Por ejemplo:

* Cloud.
* Servicios meteorológicos externos.
* Notificaciones externas.
* Integraciones remotas.
* Acceso desde fuera de la vivienda.
* Servicios de terceros.

---

# 3. Separación entre Internet y red local

La plataforma debe distinguir claramente:

```text
INTERNET
   │
   │
   ▼
┌──────────────────────┐
│ Servicios externos   │
│ Cloud                │
│ APIs                 │
│ Integraciones        │
└──────────────────────┘


RED LOCAL
   │
   ├── Ethernet
   ├── Wi-Fi
   ├── CAN
   ├── RS485
   ├── Thread
   ├── Zigbee
   └── Matter
```

La pérdida de Internet no debería afectar la red local.

Por ejemplo:

```text
Internet
   X
   │
   │
Router ─────── LAN ─────── ESP32
                       ├── Sensor
                       ├── Relay
                       └── Display
```

El ESP32 debe poder continuar funcionando.

---

# 4. Modos de funcionamiento

La plataforma debe definir diferentes niveles de funcionamiento.

```text
MODE 0
NORMAL

MODE 1
INTERNET OFFLINE

MODE 2
CENTRAL OFFLINE

MODE 3
GATEWAY OFFLINE

MODE 4
NETWORK PARTITION

MODE 5
ISOLATED ZONE

MODE 6
EMERGENCY / FAIL-SAFE
```

---

# 5. MODE 0 — Funcionamiento normal

En condiciones normales:

```text
Internet
   │
   ▼
Router
   │
   ▼
Central
   │
   ├── Zona 1
   ├── Zona 2
   ├── Zona 3
   └── Exterior
```

Todos los servicios están disponibles.

El servidor central puede proporcionar:

* Configuración.
* Historial.
* Dashboard.
* Automatizaciones globales.
* Usuarios.
* Integraciones.
* Backup.
* OTA.
* Estadísticas.
* IA.
* Acceso remoto.

---

# 6. MODE 1 — Internet offline

Si desaparece Internet:

```text
Internet
   X
   │
Router
   │
   ▼
LAN
   │
   ├── Central
   ├── ESP32
   ├── ESP32-C6
   ├── ESP32-S3
   └── Gateways
```

La red local continúa funcionando.

Deben continuar:

```text
Luces
Sensores
Alarmas
Climatización
Persianas
Energía
Agua
Automatizaciones
Escenas
Rutinas
```

No deben funcionar únicamente las características que dependan realmente de Internet.

---

# 7. MODE 2 — Servidor central offline

Este caso es más importante.

Supongamos:

```text
                 CENTRAL
                    X
                    │
       ┌────────────┼────────────┐
       │            │            │
      ESP1         ESP2         ESP3
```

La caída del servidor central **no debe apagar la instalación**.

Los nodos deben continuar ejecutando:

* Automatizaciones locales.
* Reglas.
* Seguridad.
* Escenas locales.
* Control de iluminación.
* Sensores.
* Actuadores.

---

# 8. Autonomía por niveles

La plataforma debe utilizar tres niveles de autonomía.

```text
NIVEL 1
DEVICE AUTONOMY

NIVEL 2
ZONE AUTONOMY

NIVEL 3
SITE AUTONOMY
```

---

# 9. Nivel 1 — Autonomía del dispositivo

Cada nodo debe poder ejecutar determinadas funciones por sí mismo.

Ejemplo:

```text
ESP32
│
├── PIR
└── Relay
```

Regla:

```text
PIR = MOTION
        ↓
Relay = ON
```

Esto debe funcionar aunque:

```text
Internet = OFF
Central = OFF
Gateway = OFF
```

siempre que el dispositivo pueda comunicarse con sus propios recursos.

---

# 10. Nivel 2 — Autonomía de zona

Cuando una automatización necesita varios dispositivos, debe poder ejecutarse dentro de una zona.

Ejemplo:

```text
ZONA: LIVING

ESP32 #1
├── PIR

ESP32 #2
├── Luz

ESP32 #3
├── Sensor temperatura
```

Una regla:

```text
PIR detecta movimiento
        ↓
Light ON
```

puede ejecutarse mediante comunicación local entre los nodos.

No debe ser obligatorio:

```text
ESP32 → Internet → Cloud → Central → ESP32
```

Debe ser posible:

```text
ESP32 → LAN → ESP32
```

o:

```text
ESP32 → CAN → ESP32
```

o:

```text
ESP32 → RS485 → ESP32
```

---

# 11. Nivel 3 — Autonomía de instalación

La instalación completa debe poder continuar funcionando aunque el servidor central falle.

Ejemplo:

```text
                 CENTRAL
                    X

       ┌────────────┼────────────┐
       │            │            │
    Living       Cocina       Dormitorios
       │            │            │
      ESP          ESP          ESP
```

Los nodos deben mantener:

* Reglas locales.
* Automatizaciones distribuidas.
* Estados.
* Modos de seguridad.
* Escenas críticas.
* Horarios.
* Configuración necesaria.

---

# 12. No depender de un único nodo principal

El sistema debe evitar una arquitectura como:

```text
TODOS LOS NODOS
       │
       ▼
CENTRAL
       │
       ▼
DECISIÓN
       │
       ▼
ACCIÓN
```

porque:

```text
CENTRAL OFFLINE
      ↓
TODO OFFLINE
```

Esto debe estar prohibido arquitectónicamente.

---

# 13. Arquitectura distribuida

La arquitectura recomendada es:

```text
             CENTRAL
                │
        ┌───────┼────────┐
        │       │        │
      Zona A  Zona B   Zona C
        │       │        │
      Nodes   Nodes    Nodes
```

Cada zona debe poder seguir funcionando independientemente.

---

# 14. Redundancia de funciones

Las funciones críticas deben poder tener más de un responsable.

Ejemplo:

```text
SECURITY CONTROLLER

Primary:
ESP32 #01

Backup:
ESP32 #05
```

Si:

```text
ESP32 #01 OFFLINE
```

el sistema puede utilizar:

```text
ESP32 #05
```

como controlador secundario.

---

# 15. Elección dinámica del controlador

Cuando una función requiere coordinación, el sistema puede elegir un nodo responsable.

Ejemplo:

```text
Security Zone
       │
       ├── Node A
       ├── Node B
       └── Node C
```

Uno puede actuar como:

```text
LEADER
```

y los demás como:

```text
FOLLOWERS
```

Si el líder desaparece:

```text
LEADER OFFLINE
       ↓
Election
       ↓
Node B
       ↓
NEW LEADER
```

La automatización continúa.

---

# 16. Leader Election

La elección de líder debe utilizarse solamente donde sea realmente necesaria.

No todos los dispositivos necesitan un líder.

Puede utilizarse para:

* Coordinación de zonas.
* Seguridad.
* Sincronización.
* Gateways redundantes.
* Automatizaciones distribuidas.
* Gestión de estados globales.

No debe ser necesaria para:

```text
PIR → Luz
Botón → Relay
Sensor → Ventilador
```

Estas funciones deben ser locales.

---

# 17. Heartbeat

Los nodos críticos deben emitir:

```text
HEARTBEAT
```

periódicamente.

Ejemplo:

```text
Node A
  │
  ├── heartbeat
  ├── heartbeat
  ├── heartbeat
  X
```

Después de un timeout:

```text
Node A = OFFLINE
```

La red puede iniciar:

```text
Failover
```

cuando corresponda.

---

# 18. Detección de fallo

El sistema debe diferenciar:

```text
OFFLINE
```

de:

```text
NETWORK_ERROR
```

y:

```text
DEVICE_ERROR
```

y:

```text
RESOURCE_ERROR
```

Por ejemplo:

```text
Central offline
```

no significa necesariamente:

```text
ESP32 offline
```

---

# 19. Red local sin Internet

El sistema debe poder crear o mantener una red local incluso sin salida a Internet.

Ejemplo:

```text
             ROUTER
          INTERNET X
              │
              │ LAN
              │
      ┌───────┼────────┐
      │       │        │
     ESP1    ESP2     ESP3
```

La red LAN continúa funcionando.

---

# 20. Red local independiente

Para instalaciones críticas se recomienda que la automatización no dependa exclusivamente del router doméstico.

Puede existir:

```text
AUTOMATION NETWORK
```

independiente.

Ejemplo:

```text
                 Internet
                    X
                    │
                Home Router
                    │
                 Firewall
                    │
          ┌─────────┴─────────┐
          │                   │
      Automation LAN       User LAN
          │
     ┌────┼────┐
     │    │    │
   ESP1 ESP2 ESP3
```

Esto aumenta la resiliencia.

---

# 21. Comunicación alternativa

Los nodos críticos deberían poder utilizar más de un medio cuando sea necesario.

Ejemplo:

```text
Primary:
Ethernet

Secondary:
CAN

Tertiary:
Wi-Fi
```

Si Ethernet falla:

```text
Ethernet X
   ↓
CAN
   ↓
Comunicación continúa
```

No todos los dispositivos necesitan múltiples interfaces.

Debe reservarse para nodos críticos.

---

# 22. Ejemplo: salida de la vivienda

Una situación importante es:

```text
Internet OFFLINE
```

pero el usuario quiere salir de la casa.

Debe poder:

* Activar alarma.
* Cerrar puertas.
* Cerrar persianas.
* Apagar luces.
* Cambiar a modo AWAY.
* Verificar ventanas.
* Verificar puertas.

sin Internet.

Por ejemplo:

```text
Botón "SALIR"
       ↓
Scene: LEAVE_HOME
       ↓
Security = AWAY
       ↓
Lights = OFF
       ↓
HVAC = ECO
       ↓
Garage = CLOSED
       ↓
Windows = CHECK
```

Todo debe ejecutarse localmente.

---

# 23. Acceso local durante pérdida de Internet

Si el usuario necesita configurar el sistema mientras Internet está caída, debe poder conectarse localmente.

Opciones:

```text
http://automation.local
```

o:

```text
IP local
```

o mediante:

```text
Pantalla táctil
```

o:

```text
Aplicación móvil en LAN
```

---

# 24. Portal local de emergencia

La plataforma debería contemplar un:

```text
LOCAL EMERGENCY UI
```

que permita realizar operaciones básicas aunque el sistema esté aislado.

Por ejemplo:

```text
Estado general
Luces
Alarmas
Puertas
Ventanas
Temperatura
Modo
Escenas
Diagnóstico
```

---

# 25. Pantallas autónomas

Las pantallas táctiles no deben depender obligatoriamente del servidor central.

Una pantalla puede almacenar:

* UI básica.
* Estado de la zona.
* Controles principales.
* Escenas.
* Información crítica.

Por ejemplo:

```text
Pantalla Living
      │
      ├── Luces
      ├── Temperatura
      ├── Seguridad
      └── Escenas
```

debe seguir funcionando aunque:

```text
Central = OFFLINE
```

---

# 26. Escenas críticas

Las escenas importantes deben almacenarse localmente.

Ejemplos:

```text
LEAVE_HOME
ARRIVE_HOME
SLEEP
EMERGENCY
ALL_LIGHTS_OFF
PANIC
```

Cada nodo o zona crítica debe disponer de las escenas necesarias.

---

# 27. Automatizaciones críticas

Las automatizaciones críticas deben tener una copia local.

Ejemplo:

```text
AUTOMATION:

IF
window.opened

AND
security.mode == AWAY

THEN
alarm = ON
```

Debe estar almacenada:

```text
Central
+
Zona
```

o:

```text
Central
+
Nodo crítico
```

según la importancia.

---

# 28. Clasificación de automatizaciones

No todas las automatizaciones requieren el mismo nivel de redundancia.

### Nivel CRITICAL

Ejemplos:

* Incendio.
* Intrusión.
* Fuga de agua.
* Sobretemperatura peligrosa.
* Emergencia.

Debe existir localmente.

### Nivel IMPORTANT

Ejemplos:

* Climatización.
* Persianas.
* Iluminación principal.
* Gestión energética.

Debe funcionar sin Internet y preferentemente sin central.

### Nivel NORMAL

Ejemplos:

* Luces automáticas.
* Música.
* Escenas.

Debe funcionar localmente cuando sea posible.

### Nivel OPTIONAL

Ejemplos:

* Estadísticas.
* IA cloud.
* Integraciones externas.

Puede dejar de funcionar durante una desconexión.

---

# 29. Matriz de dependencia

Cada automatización debe declarar sus dependencias.

Ejemplo:

```text
Automation:
Exterior Alarm

Required:
PIR
Door sensor
Local controller

Internet:
NOT REQUIRED

Central:
NOT REQUIRED
```

Otro ejemplo:

```text
Automation:
Weather Forecast Optimization

Required:
Internet
Weather API

Central:
RECOMMENDED
```

Esto permite determinar qué puede sobrevivir a cada fallo.

---

# 30. Regla de independencia

Una función crítica nunca debe depender de:

```text
Internet
Cloud
Third-party API
Central server
```

si puede realizarse localmente.

---

# 31. Datos locales

Cada nodo debe almacenar localmente la información necesaria para continuar funcionando.

Por ejemplo:

```text
Device configuration
Local rules
Critical scenes
Security mode
Schedules
Calibration
Network configuration
Fallback behavior
```

---

# 32. Persistencia distribuida

Los datos importantes pueden existir en varios niveles:

```text
Central
   +
Zone Controller
   +
Critical Node
```

Ejemplo:

```text
Security configuration

Central copy
Zone copy
Local backup
```

---

# 33. Configuración versionada

Cuando una configuración se distribuye:

```text
CONFIG VERSION 125
```

los nodos deben conocer qué versión tienen.

Ejemplo:

```text
Central = v125
Zone A = v125
Node A = v125
Node B = v124
```

El sistema puede detectar:

```text
Node B requires synchronization
```

---

# 34. Funcionamiento con configuración antigua

Si se pierde la comunicación con el servidor mientras se actualiza una configuración:

```text
Node = v124
Central = v125
```

el nodo debe continuar utilizando:

```text
v124
```

hasta recibir correctamente:

```text
v125
```

Nunca debe quedar sin configuración válida.

---

# 35. Atomic Configuration Update

Las configuraciones deben actualizarse de forma atómica.

Proceso:

```text
Receive configuration
       ↓
Validate
       ↓
Store temporary
       ↓
Verify
       ↓
Activate
```

Si algo falla:

```text
Rollback
```

---

# 36. Store and Forward

Cuando la comunicación con el central desaparece:

```text
Node
 ↓
Generate event
 ↓
Store locally
```

Cuando vuelve:

```text
Connection restored
       ↓
Synchronize
       ↓
Send pending events
```

Esto evita perder información.

---

# 37. Sincronización posterior

Al recuperar la comunicación:

```text
CENTRAL ONLINE
      ↓
Discovery
      ↓
Heartbeat
      ↓
Configuration comparison
      ↓
State synchronization
      ↓
Pending events
      ↓
History synchronization
```

---

# 38. Evitar conflictos

Si un dispositivo cambió mientras estaba desconectado:

```text
Central:
Light = OFF

Node:
Light = ON
```

el sistema debe tener reglas claras de prioridad.

Debe existir:

```text
Timestamp
Source
Priority
Sequence
Version
```

para resolver conflictos.

---

# 39. Estado de modo global

Los modos globales deben tener mecanismos de fallback.

Ejemplo:

```text
SYSTEM.MODE = SLEEP
```

El modo debe poder mantenerse localmente aunque el central esté offline.

Cada zona puede conocer:

```text
Current mode
Last known mode
Mode timestamp
Mode source
```

---

# 40. Seguridad durante desconexión

Si se pierde Internet:

```text
Security = continúa funcionando
```

Si se pierde el central:

```text
Security = continúa funcionando
```

Si se pierde un nodo:

```text
Otros sensores continúan
```

Si se pierde un sensor:

```text
Sistema registra fallo
```

---

# 41. Fail-safe

Cada recurso crítico debe definir un estado seguro.

Ejemplo:

```text
Valve
→ CLOSED

Heater
→ OFF

Motor
→ STOP

Door lock
→ según política de seguridad

Alarm
→ ACTIVE

Ventilation
→ según condición
```

El estado seguro debe configurarse según el tipo de dispositivo.

No existe un único estado seguro universal.

---

# 42. Ejemplo: fuga de agua

Supongamos:

```text
Sensor de fuga
```

detecta:

```text
WATER_LEAK
```

La acción crítica puede ser:

```text
Sensor
 ↓
Local controller
 ↓
Water valve CLOSE
```

No:

```text
Sensor
 ↓
Internet
 ↓
Cloud
 ↓
Central
 ↓
Valve
```

La segunda arquitectura es incorrecta para una función crítica.

---

# 43. Ejemplo: alarma

```text
PIR
 ↓
Local security node
 ↓
Alarm evaluation
 ↓
Siren
```

Después:

```text
Event
 ↓
Central
 ↓
Notification
```

La notificación es secundaria.

La alarma local es primaria.

---

# 44. Ejemplo: incendio

```text
Smoke sensor
       ↓
LOCAL
       ↓
Alarm
       ↓
Lights ON
       ↓
HVAC OFF
       ↓
Unlock/lock according to configured safety policy
```

Luego:

```text
Central
 ↓
Log
 ↓
Notification
```

Internet no debe ser necesaria para activar la respuesta local.

---

# 45. Redundancia por habitación

Como mínimo, una habitación crítica puede tener autonomía local.

Ejemplo:

```text
BEDROOM
│
├── PIR
├── Light
├── Temperature
├── Window
└── Touchscreen
```

Si el central falla:

```text
Bedroom
   ↓
Local automation
   ↓
CONTINUES
```

---

# 46. Redundancia por zona

Para instalaciones grandes:

```text
BUILDING
│
├── Zone A
│   ├── Controller
│   └── Nodes
│
├── Zone B
│   ├── Controller
│   └── Nodes
│
└── Zone C
    ├── Controller
    └── Nodes
```

Cada zona debe ser capaz de operar de forma independiente durante una falla del sistema superior.

---

# 47. Recuperación del nodo central

Cuando el central vuelve:

```text
Central boot
   ↓
Network discovery
   ↓
Node discovery
   ↓
Heartbeat
   ↓
State synchronization
   ↓
Configuration validation
   ↓
History synchronization
   ↓
Normal operation
```

No debe reiniciar innecesariamente los nodos.

---

# 48. No sobrescribir estados locales incorrectamente

Ejemplo:

Durante la caída:

```text
Light = ON
```

Luego vuelve el central y tenía:

```text
Light = OFF
```

El sistema no debe simplemente ejecutar:

```text
Central → OFF
```

sin evaluar:

* Timestamp.
* Origen.
* Prioridad.
* Estado actual.
* Automatización activa.
* Modo del sistema.

---

# 49. Recuperación gradual

La recuperación debe ser gradual:

```text
1. Comunicación
2. Autenticación
3. Estado
4. Configuración
5. Eventos
6. Historial
7. Servicios externos
```

No debe bloquear las funciones locales esperando la recuperación completa.

---

# 50. Internet vuelve después del central

Si:

```text
Central ONLINE
Internet OFFLINE
```

el sistema debe permanecer en:

```text
LOCAL MODE
```

y simplemente esperar a que Internet vuelva.

---

# 51. Internet vuelve

Cuando vuelve Internet:

```text
Internet ONLINE
      ↓
External integrations
      ↓
Cloud synchronization
      ↓
Notifications
      ↓
Weather services
      ↓
Remote access
```

Las funciones locales no necesitan reiniciarse.

---

# 52. Integraciones de terceros

Las integraciones externas se consideran:

```text
OPTIONAL SERVICES
```

Ejemplos:

```text
Home Assistant
Homey
SmartThings
Apple Home
Google Home
Cloud
Weather APIs
MQTT brokers externos
```

Su caída no debe detener la automatización interna.

---

# 53. Clasificación de servicios

### CORE

No puede fallar por Internet:

```text
Sensors
Actuators
Local automation
Safety
Security
Scenes
Schedules
```

### LOCAL SERVICES

Pueden funcionar sin Internet:

```text
Web UI
REST API
WebSocket
Local dashboard
Local database
```

### EXTERNAL

Pueden dejar de funcionar:

```text
Cloud
Remote access
External notifications
Third-party APIs
External weather
```

---

# 54. Acceso remoto

El acceso desde fuera de la casa depende normalmente de Internet.

Por lo tanto:

```text
Internet OFF
      ↓
Remote access OFF
```

Esto es aceptable.

No debe interpretarse como una falla de la automatización.

---

# 55. Control desde el exterior

Cuando el usuario está fuera de la vivienda y no existe Internet:

```text
Usuario
   X
Internet
   X
Casa
```

No podrá controlar remotamente el sistema.

Sin embargo:

```text
Casa
 ↓
continúa funcionando
```

con sus automatizaciones locales.

---

# 56. Comunicación interna alternativa

Para evitar que la red Wi-Fi sea el único medio, los dispositivos críticos pueden utilizar:

```text
Ethernet
CAN
RS485
Thread
```

según el caso.

La arquitectura debe permitir múltiples transportes.

---

# 57. Nodo central como coordinador, no como cerebro único

El nodo central debe entenderse como:

```text
COORDINATOR
```

y no:

```text
SINGLE POINT OF FAILURE
```

Debe proporcionar:

* Administración.
* Visualización.
* Configuración.
* Históricos.
* Coordinación.
* Backup.
* Integraciones.

Pero las funciones críticas deben distribuirse.

---

# 58. Central redundante

En instalaciones importantes puede existir:

```text
CENTRAL A
CENTRAL B
```

Ejemplo:

```text
          ┌─────────────┐
          │ CENTRAL A   │
          └──────┬──────┘
                 │
              Primary
                 │
          ┌──────┴──────┐
          │ CENTRAL B   │
          └─────────────┘
              Backup
```

Si A falla:

```text
A OFFLINE
   ↓
B ACTIVE
```

---

# 59. Central activo/pasivo

Para instalaciones grandes:

```text
Primary Central
      │
      │ heartbeat
      ▼
Backup Central
```

El backup puede mantener:

* Configuración.
* Estado.
* Base de datos.
* Automatizaciones.
* Usuarios.
* Integraciones.

---

# 60. Central activo/activo

En sistemas de mayor complejidad puede existir:

```text
Central A
Central B
```

ambos activos.

Los dispositivos pueden conectarse a ambos.

Esto proporciona mayor disponibilidad, pero aumenta considerablemente la complejidad.

Debe considerarse una característica avanzada y no una obligación para instalaciones pequeñas.

---

# 61. Autonomía configurable

Cada módulo debe declarar:

```text
requires_central
requires_internet
supports_offline
supports_local_execution
criticality
fallback_behavior
```

Ejemplo conceptual:

```json
{
  "module": "security_alarm",
  "requires_central": false,
  "requires_internet": false,
  "supports_offline": true,
  "criticality": "critical"
}
```

---

# 62. Matriz de funcionamiento

La plataforma debería disponer de una matriz como:

| Función                | Internet |           Central | Funcionamiento local |
| ---------------------- | -------: | ----------------: | -------------------: |
| Interruptor → luz      |       No |                No |                   Sí |
| PIR → luz              |       No |                No |                   Sí |
| Alarma                 |       No |                No |                   Sí |
| Sensor temperatura     |       No |                No |                   Sí |
| Climatización          |       No |                No |                   Sí |
| Escena SLEEP           |       No |                No |                   Sí |
| Seguridad              |       No |                No |                   Sí |
| Web local              |       No |               No* |                   Sí |
| Historial local        |       No |               No* |                   Sí |
| Home Assistant externo |       Sí |    Normalmente sí |                   No |
| Weather API            |       Sí | No necesariamente |                   No |
| Notificación cloud     |       Sí |    Normalmente sí |                   No |
| Acceso remoto          |       Sí |    Normalmente sí |                   No |

`*` Siempre que exista un nodo local que proporcione el servicio.

---

# 63. Objetivo de disponibilidad

Para funciones críticas:

```text
Availability ≈ 100 %
```

independientemente de:

```text
Internet
Cloud
Central
```

siempre que los dispositivos físicos necesarios estén disponibles.

---

# 64. Regla de dependencia mínima

Cada función debe depender del menor número posible de componentes.

Incorrecto:

```text
PIR
 ↓
Wi-Fi
 ↓
Router
 ↓
Central
 ↓
MQTT
 ↓
Automation Engine
 ↓
Wi-Fi
 ↓
Relay
```

Correcto:

```text
PIR
 ↓
Local Event
 ↓
Automation
 ↓
Relay
```

La segunda arquitectura tiene menos puntos de fallo.

---

# 65. Principio de proximidad

Cuanto más crítica sea una función, más cerca debe estar la lógica de decisión del recurso físico.

Ejemplo:

```text
Seguridad crítica
        ↓
Zona
        ↓
Nodo
```

No:

```text
Seguridad crítica
        ↓
Cloud
```

---

# 66. Principio de supervivencia

Cada nodo crítico debe poder responder:

> "¿Qué hago si todos los demás desaparecen?"

Debe existir un comportamiento definido.

Ejemplo:

```text
No communication
        ↓
Continue local rules
        ↓
Use last valid configuration
        ↓
Enter safe mode when required
```

---

# 67. Principio de último estado válido

Si se pierde la comunicación con el central:

```text
Use last known valid configuration
```

No:

```text
Erase configuration
```

ni:

```text
Disable automation
```

---

# 68. Principio de no bloqueo

Ningún nodo debe quedarse bloqueado indefinidamente esperando:

```text
Central
Internet
MQTT
Gateway
API
```

Debe existir:

```text
Timeout
Retry
Fallback
Local execution
```

---

# 69. Watchdog

Todos los nodos críticos deben utilizar watchdog.

El watchdog debe detectar:

```text
Task stuck
Deadlock
Communication lock
Unexpected state
```

y reiniciar el componente cuando corresponda.

Después del reinicio:

```text
Boot
 ↓
Load last valid configuration
 ↓
Initialize hardware
 ↓
Restore local automation
 ↓
Continue operation
```

---

# 70. Reinicio seguro

Un reinicio no debería eliminar:

* Configuración.
* Escenas.
* Automatizaciones.
* Calibraciones.
* Seguridad.

Debe utilizarse almacenamiento persistente adecuado.

---

# 71. Prueba de fallos

La plataforma debe incluir pruebas explícitas:

```text
TEST 01
Internet OFF

TEST 02
Central OFF

TEST 03
Router OFF

TEST 04
Gateway OFF

TEST 05
Wi-Fi OFF

TEST 06
Ethernet OFF

TEST 07
Node OFF

TEST 08
Sensor OFF

TEST 09
Power cycle

TEST 10
Central recovery
```

---

# 72. Criterio de aceptación

Una instalación no debe considerarse correctamente diseñada hasta comprobar que:

```text
Internet OFF
```

no detiene:

```text
Automatizaciones críticas
```

y:

```text
Central OFF
```

no detiene:

```text
Automatizaciones locales
```

---

# 73. Objetivo final

La arquitectura final debe comportarse conceptualmente así:

```text
                    INTERNET
                       X
                       │
                    CENTRAL
                       X
                       │
          ┌────────────┼────────────┐
          │            │            │
        ZONA A       ZONA B       ZONA C
          │            │            │
       ┌──┼──┐      ┌──┼──┐      ┌──┼──┐
       │  │  │      │  │  │      │  │  │
      N1 N2 N3     N4 N5 N6     N7 N8 N9
```

Aunque:

```text
Internet = OFF
Central = OFF
```

debe continuar:

```text
ZONA A → funcionando
ZONA B → funcionando
ZONA C → funcionando
```

siempre que los nodos físicos necesarios estén disponibles.

---

# 74. Arquitectura de resiliencia recomendada

La arquitectura final recomendada es:

```text
                         CLOUD
                           │
                           │ opcional
                           ▼
                     INTERNET
                           │
                           │ opcional
                           ▼
                 ┌──────────────────┐
                 │ CENTRAL / SERVER │
                 └────────┬─────────┘
                          │
                    coordinación
                          │
       ┌──────────────────┼──────────────────┐
       │                  │                  │
       ▼                  ▼                  ▼
   ZONE A              ZONE B              ZONE C
       │                  │                  │
   ┌───┼───┐          ┌───┼───┐          ┌───┼───┐
   │   │   │          │   │   │          │   │   │
  N1  N2  N3         N4  N5  N6         N7  N8  N9
   │   │   │          │   │   │          │   │   │
   └───┴───┘          └───┴───┘          └───┴───┘

        LOCAL           LOCAL             LOCAL
     AUTOMATION      AUTOMATION        AUTOMATION
```

El central mejora el sistema.

No lo hace posible.

---

# 75. Regla arquitectónica definitiva

Debe considerarse una violación de arquitectura diseñar una función crítica de esta forma:

```text
Sensor
 ↓
Internet
 ↓
Cloud
 ↓
Central
 ↓
Internet
 ↓
Actuator
```

cuando la función podría ejecutarse localmente.

La arquitectura correcta es:

```text
Sensor
 ↓
Local / Zone Controller
 ↓
Actuator
```

y posteriormente:

```text
                    ┌── Central
                    ├── History
                    ├── Cloud
                    └── Notifications
```

como servicios complementarios.

---

# 76. Relación con otros documentos

Este documento debe integrarse con:

```text
docs/
├── SYSTEM-ARCHITECTURE.md
├── ARCHITECTURE.md
├── DEVICE-MODEL.md
├── COMMUNICATION.md
├── MODULE-DEVELOPMENT.md
└── FAULT-TOLERANCE.md
```

### SYSTEM-ARCHITECTURE.md

Define:

* Concepto general.
* Zonas.
* Recursos.
* Funciones.
* Escenas.
* Automatizaciones.
* Arquitectura distribuida.

### ARCHITECTURE.md

Define:

* Capas.
* Componentes.
* Servicios.
* Estructura técnica.

### DEVICE-MODEL.md

Define:

* Dispositivos.
* Recursos.
* Estados.
* Capacidades.

### COMMUNICATION.md

Define:

* Transporte.
* Mensajes.
* Eventos.
* Heartbeat.
* Discovery.
* Gateways.

### FAULT-TOLERANCE.md

Define:

* Qué sucede ante fallos.
* Autonomía.
* Failover.
* Recuperación.
* Redundancia.
* Funcionamiento offline.

---

# 77. Regla para futuros módulos

Todo nuevo módulo debe declarar explícitamente:

```text
Offline capable?
Local execution?
Requires central?
Requires Internet?
Criticality?
Fallback?
Recovery?
Persistent configuration?
```

Esto debe impedir que accidentalmente se cree un módulo que dependa del servidor central para una función que debería ser autónoma.

---

# 78. Conclusión

La plataforma debe diseñarse como un sistema:

```text
DISTRIBUIDO
+
AUTÓNOMO
+
LOCAL FIRST
+
FAULT TOLERANT
+
MODULAR
+
ESCALABLE
```

El servidor central debe proporcionar inteligencia y administración global.

Internet debe proporcionar servicios externos.

Pero los dispositivos deben continuar controlando la instalación.

La pérdida de Internet debe significar:

```text
"Se perdieron servicios externos"
```

y no:

```text
"Se perdió la automatización"
```

La pérdida del servidor central debe significar:

```text
"Se perdió la administración central"
```

y no:

```text
"Se apagó la casa"
```

La pérdida de un nodo debe significar:

```text
"Se perdió una parte del sistema"
```

y no:

```text
"Se perdió todo el sistema"
```

Este principio debe considerarse uno de los requisitos fundamentales de la plataforma.

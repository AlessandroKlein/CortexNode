# FreeRTOS Task Architecture

> **Tipo:** Arquitectura / Convención
> **Estado:** Definición
> **Versión:** 1.0.0
> **Última actualización:** 2026-10-05

---

## 1. Objetivo

Este documento define cómo debe utilizarse **FreeRTOS** en todos los dispositivos de la plataforma.

El objetivo es unificar criterios para que los distintos Nodes, Zone Controllers y Centrals mantengan una arquitectura coherente independientemente del modelo de ESP32 utilizado.

La plataforma no debe permitir que cada módulo cree tareas FreeRTOS arbitrariamente.

Debe existir una arquitectura común para:

* tareas;
* prioridades;
* núcleos;
* colas;
* eventos;
* mutex;
* semáforos;
* timers;
* watchdog;
* comunicación;
* adquisición de sensores;
* control de actuadores;
* automatizaciones;
* diagnóstico;
* almacenamiento;
* red.

---

# 2. Principio fundamental

> **Una tarea FreeRTOS representa una responsabilidad lógica, no necesariamente un dispositivo físico.**

No se debe crear una tarea simplemente porque existe un sensor, relay o módulo.

Incorrecto:

```text
Sensor temperatura
    ↓
TaskTemperature

Sensor humedad
    ↓
TaskHumidity

Sensor presión
    ↓
TaskPressure

Relay
    ↓
TaskRelay
```

Esto puede generar una cantidad innecesaria de tareas.

Preferentemente:

```text
┌──────────────────────────────┐
│ Sensor Acquisition Task      │
│                              │
│ ├── temperatura              │
│ ├── humedad                  │
│ └── presión                  │
└──────────────┬───────────────┘
               │
               ▼
          Event Bus
```

La creación de tareas debe justificarse por:

* tiempo de ejecución;
* periodicidad;
* prioridad;
* aislamiento;
* bloqueo;
* hardware;
* latencia;
* criticidad.

---

# 3. Arquitectura general

Cada firmware debe organizarse conceptualmente en:

```text
┌───────────────────────────────────────────┐
│                  APP                      │
│                                           │
│ Automatizaciones / Escenas / Servicios    │
└─────────────────────┬─────────────────────┘
                      │
┌─────────────────────▼─────────────────────┐
│             SYSTEM SERVICES               │
│                                           │
│ State / Event / Command / Config          │
└─────────────────────┬─────────────────────┘
                      │
┌─────────────────────▼─────────────────────┐
│              RTOS SERVICES                │
│                                           │
│ Tasks / Queues / Timers / Mutex           │
└─────────────────────┬─────────────────────┘
                      │
┌─────────────────────▼─────────────────────┐
│              HARDWARE                     │
│                                           │
│ GPIO / I2C / SPI / UART / CAN / ADC      │
└───────────────────────────────────────────┘
```

La aplicación no debe acceder directamente a mecanismos internos de FreeRTOS cuando exista una abstracción del sistema.

---

# 4. Principio de separación

Las tareas deben dividirse según su responsabilidad.

Categorías recomendadas:

```text
SYSTEM
NETWORK
COMMUNICATION
SENSOR
ACTUATOR
AUTOMATION
STORAGE
UI
DIAGNOSTIC
WATCHDOG
```

No todos los dispositivos necesitan todas las categorías.

---

# 5. Tareas estándar

Cuando corresponda, un Node podrá disponer de:

```text
SystemTask
NetworkTask
CommunicationTask
SensorTask
ActuatorTask
AutomationTask
StorageTask
UITask
DiagnosticTask
WatchdogTask
```

No significa que todas deban existir.

La arquitectura debe crear únicamente las necesarias.

---

# 6. SystemTask

Responsabilidad:

* inicialización;
* coordinación del arranque;
* estado general;
* supervisión de servicios;
* cambio de modo;
* gestión de errores;
* apagado controlado.

Ejemplo:

```text
SystemTask
    │
    ├── initialize
    ├── load configuration
    ├── start services
    ├── verify hardware
    └── enter RUNNING
```

No debe ejecutar tareas periódicas pesadas.

---

# 7. NetworkTask

Gestiona, cuando corresponda:

* Wi-Fi;
* Ethernet;
* DHCP;
* IP;
* DNS;
* mDNS;
* conexión/desconexión;
* estado de red.

No debe ejecutar directamente las automatizaciones.

Ejemplo:

```text
NetworkTask
     │
     ├── Ethernet
     ├── Wi-Fi
     ├── DHCP
     └── connectivity state
```

La aplicación recibe eventos:

```text
NETWORK_CONNECTED
NETWORK_DISCONNECTED
NETWORK_IP_CHANGED
```

---

# 8. CommunicationTask

Gestiona la comunicación lógica del sistema.

Puede trabajar con:

* MQTT;
* REST;
* WebSocket;
* CAN;
* RS485;
* System Bus;
* protocolos propietarios.

Debe separar:

```text
Transport
```

de:

```text
Application Message
```

Ejemplo:

```text
EVENT
motion.detected
```

no debe depender de si fue recibido por:

```text
Wi-Fi
Ethernet
CAN
RS485
```

---

# 9. SensorTask

Responsabilidad:

* adquisición;
* lectura;
* filtrado;
* validación;
* conversión de unidades;
* detección de cambios;
* publicación de estados.

Ejemplo:

```text
SensorTask
     │
     ├── AHT30
     ├── DS18B20
     ├── BMP280
     └── ADC
          │
          ▼
      Sensor State
```

Debe evitar publicar datos innecesariamente.

Se recomienda configurar:

```text
sample_interval
publish_interval
change_threshold
```

Ejemplo:

```text
Temperatura:
leer cada 1 s
publicar cada 5 s
o si cambia > 0.2 °C
```

---

# 10. ActuatorTask

Gestiona:

* relays;
* PWM;
* dimmers;
* motores;
* válvulas;
* salidas digitales;
* actuadores analógicos.

Debe recibir comandos mediante:

```text
Queue
Event
Command Bus
```

No se recomienda que múltiples tareas escriban directamente sobre el mismo GPIO.

Ejemplo:

```text
AutomationTask
      │
      ▼
Command Queue
      │
      ▼
ActuatorTask
      │
      ▼
GPIO
```

---

# 11. AutomationTask

Ejecuta:

* reglas;
* escenas;
* rutinas;
* condiciones;
* temporizadores lógicos;
* acciones.

Ejemplo:

```text
EVENT:
motion.detected

CONDITION:
HOUSE.MODE == NIGHT

ACTION:
LIGHT = 15 %
```

La AutomationTask no debería controlar directamente el hardware.

Debe generar:

```text
Command
```

Ejemplo:

```text
SET_OUTPUT
device=light_01
value=15
```

---

# 12. StorageTask

Gestiona operaciones potencialmente lentas:

* NVS;
* LittleFS;
* SPIFFS cuando corresponda;
* SD;
* registros;
* configuración;
* historial.

No se debe realizar escritura frecuente directamente desde tareas críticas.

Incorrecto:

```text
SensorTask
   │
   ├── leer sensor
   ├── guardar en Flash
   └── continuar
```

Preferentemente:

```text
SensorTask
   │
   ▼
Queue
   │
   ▼
StorageTask
   │
   ▼
Flash / SD
```

---

# 13. UITask

Cuando el dispositivo tenga:

* pantalla;
* touchscreen;
* botones;
* encoder;
* interfaz local.

La UI debe estar separada de la lógica de automatización.

Ejemplo:

```text
UITask
  │
  ├── Display
  ├── Touch
  ├── Buttons
  └── Encoder
```

Una pulsación genera:

```text
INPUT_EVENT
```

La automatización decide qué hacer.

---

# 14. DiagnosticTask

Responsabilidad:

* CPU;
* RAM;
* PSRAM;
* stack;
* temperatura del chip cuando esté disponible;
* errores;
* buses;
* dispositivos;
* latencia;
* paquetes;
* watchdog;
* uptime.

Debe permitir detectar:

```text
Task stack overflow
Heap fragmentation
Queue overflow
Communication timeout
Bus error
Device offline
```

---

# 15. WatchdogTask

Todo Node crítico debe disponer de supervisión.

Debe verificar que las tareas críticas continúan funcionando.

No se debe utilizar el watchdog como mecanismo normal de sincronización.

El watchdog existe para detectar:

```text
deadlock
infinite loop
task starvation
hardware lockup
unexpected blocking
```

---

# 16. Tareas periódicas vs event-driven

La plataforma debe favorecer tareas **event-driven** cuando sea posible.

En lugar de:

```text
while(true)
{
    checkSensor();
    checkButton();
    checkNetwork();
    checkAutomation();
    delay(10);
}
```

preferentemente:

```text
Event
 ↓
Queue
 ↓
Task
 ↓
Process
```

Esto reduce:

* CPU;
* polling;
* consumo;
* latencia innecesaria.

---

# 17. Uso de `vTaskDelay`

Las tareas periódicas deben utilizar mecanismos apropiados de FreeRTOS.

Ejemplo conceptual:

```cpp
while (true) {
    readSensors();
    vTaskDelayUntil(&lastWakeTime, period);
}
```

Para tareas que esperan eventos:

```cpp
xQueueReceive(queue, &event, portMAX_DELAY);
```

No se recomienda utilizar:

```cpp
delay()
```

como mecanismo principal de coordinación de tareas.

---

# 18. Colas

Las colas deben utilizarse para pasar datos entre tareas.

Ejemplo:

```text
SensorTask
     │
     ▼
SensorQueue
     │
     ▼
AutomationTask
```

Y:

```text
AutomationTask
     │
     ▼
CommandQueue
     │
     ▼
ActuatorTask
```

Las colas deben tener tamaño suficiente para evitar pérdida de eventos.

---

# 19. Event Groups

Los Event Groups se utilizarán para estados colectivos.

Ejemplo:

```text
NETWORK_CONNECTED
TIME_SYNCHRONIZED
CONFIG_LOADED
SYSTEM_READY
OTA_ACTIVE
```

Ejemplo conceptual:

```text
SYSTEM_READY =
CONFIG_LOADED
+
NETWORK_CONNECTED
+
HARDWARE_OK
```

---

# 20. Mutex

Los mutex deben utilizarse para proteger recursos compartidos.

Ejemplos:

```text
I2C bus
SPI bus
Shared configuration
Shared state
Display
SD card
```

No deben utilizarse como sustituto de una arquitectura correcta de comunicación.

---

# 21. Semáforos

Los semáforos pueden utilizarse para:

* sincronización;
* interrupciones;
* disponibilidad de hardware;
* eventos externos.

Ejemplo:

```text
GPIO Interrupt
      │
      ▼
Binary Semaphore
      │
      ▼
Task
```

La ISR debe hacer el mínimo trabajo posible.

---

# 22. ISR — Interrupt Service Routine

Las ISR deben ser extremadamente pequeñas.

No realizar dentro de una ISR:

* acceso complejo a buses;
* escritura en Flash;
* llamadas de red;
* procesamiento pesado;
* asignación dinámica de memoria;
* lógica de automatización.

Preferentemente:

```text
ISR
 │
 └── notify/semaphore/queue
          │
          ▼
       FreeRTOS Task
```

---

# 23. Task Notifications

Cuando solamente sea necesario notificar a una tarea, se recomienda considerar Task Notifications antes de crear una cola.

Ejemplo:

```text
GPIO interrupt
      │
      ▼
Task Notification
      │
      ▼
InputTask
```

Es más eficiente que utilizar mecanismos más pesados cuando solo existe un consumidor.

---

# 24. Prioridades

Las prioridades deben utilizarse con moderación.

Propuesta conceptual:

```text
Priority 5+
    Seguridad / tiempo crítico

Priority 4
    Comunicación crítica

Priority 3
    Control / sensores

Priority 2
    Automatización normal

Priority 1
    Servicios generales

Priority 0
    Background
```

Los valores concretos pueden variar según el firmware y el modelo de ESP32.

Lo importante es mantener la jerarquía.

---

# 25. Regla de prioridades

No debe utilizarse una prioridad alta simplemente porque una tarea sea importante para el desarrollador.

Una prioridad alta significa:

> Esta tarea debe ejecutarse antes que otras cuando existe competencia por CPU.

Por lo tanto:

```text
Safety > Control > Communication > UI > Logging
```

como regla general.

---

# 26. Core Affinity

Los ESP32 con múltiples núcleos pueden utilizar:

```text
Core 0
Core 1
```

pero no debe dividirse el sistema arbitrariamente.

La asignación debe responder a una razón técnica.

Ejemplo:

```text
Core 0
├── Networking
├── System
└── Communication

Core 1
├── Sensors
├── Actuators
└── Automation
```

Esto es solamente un ejemplo.

La distribución real dependerá del SoC y del framework.

---

# 27. No asumir dual-core

El código debe ser compatible conceptualmente con:

```text
ESP32 single-core
ESP32 dual-core
```

No debe asumir que siempre existen dos núcleos.

Por eso:

```text
Task Architecture
```

debe ser independiente de:

```text
Core Affinity
```

---

# 28. SMP

Cuando el SoC y framework lo permitan, podrá utilizarse SMP.

La aplicación no debería depender de que una tarea específica esté permanentemente en un núcleo salvo que exista una razón técnica.

La prioridad debe ser:

```text
Correctness
>
Determinism
>
Resource usage
>
Optimization
```

---

# 29. Bloqueos

No se deben realizar operaciones potencialmente largas dentro de tareas críticas.

Ejemplos:

```text
HTTP request
DNS
MQTT reconnect
Flash write
SD write
Modbus timeout
CAN recovery
```

Deben ejecutarse de forma controlada.

---

# 30. Timeouts

Toda comunicación debe tener timeout.

Nunca:

```text
wait forever
```

salvo mecanismos cuyo bloqueo indefinido sea deliberado y seguro, como una tarea esperando eventos.

Ejemplo:

```text
Modbus timeout = 100 ms
Network timeout = configurable
Sensor timeout = configurable
```

---

# 31. Comunicación entre tareas

Las tareas no deberían modificar directamente estructuras globales sin protección.

Preferentemente:

```text
Event
Queue
Notification
State Manager
```

Ejemplo:

```text
SensorTask
      │
      ▼
State Manager
      │
      ├── Automation
      ├── API
      └── Display
```

---

# 32. Estado del dispositivo

Se recomienda mantener un estado centralizado:

```text
DeviceState
```

con información como:

```text
online
network
time_synced
alarm_state
system_mode
errors
resources
```

Las tareas modifican el estado mediante mecanismos controlados.

---

# 33. Estados de operación

Todos los dispositivos deberían contemplar:

```text
BOOTING
INITIALIZING
CONFIGURING
RUNNING
DEGRADED
MAINTENANCE
OTA
ERROR
SAFE_MODE
```

---

# 34. Modo degradado

Si falla un servicio no crítico:

```text
MQTT OFF
```

no debería provocar:

```text
SYSTEM OFF
```

Ejemplo:

```text
MQTT ❌
WebSocket ❌
History ❌

Local automation ✓
Sensors ✓
Actuators ✓
Safety ✓
```

---

# 35. Fallo de una tarea

Una tarea defectuosa no debería provocar necesariamente el reinicio de todo el dispositivo.

La arquitectura debe permitir:

```text
Task failure
     ↓
Detect
     ↓
Restart task
     ↓
Recover
```

Si no es posible:

```text
Safe state
     ↓
Watchdog
     ↓
Restart device
```

---

# 36. Stack de cada tarea

Cada tarea debe definir explícitamente una cantidad adecuada de stack.

No utilizar valores enormes "por seguridad" sin justificación.

Debe monitorizarse:

```text
uxTaskGetStackHighWaterMark()
```

durante desarrollo y diagnóstico.

---

# 37. Memoria dinámica

Debe minimizarse la asignación dinámica frecuente dentro de tareas de tiempo crítico.

Evitar:

```text
malloc/free
String allocation
JSON creation
```

repetidamente dentro de loops rápidos.

Preferir:

```text
buffers
object pools
static allocation
reutilización
```

cuando sea necesario.

---

# 38. JSON

El procesamiento JSON debe mantenerse fuera de tareas de control crítico cuando sea posible.

Ejemplo:

```text
NetworkTask
      │
      ▼
JSON Parser
      │
      ▼
Command
      │
      ▼
Automation / Actuator
```

No:

```text
ISR
 ↓
JSON
 ↓
GPIO
```

---

# 39. Regla para crear una nueva tarea

Antes de crear una tarea se debe responder:

```text
1. ¿Tiene una responsabilidad independiente?
2. ¿Tiene una periodicidad diferente?
3. ¿Necesita otra prioridad?
4. ¿Puede bloquearse?
5. ¿Necesita aislamiento?
6. ¿Existe una razón de tiempo real?
7. ¿No puede resolverse mediante una tarea existente?
```

Si la respuesta es mayoritariamente "no":

> No crear una nueva tarea.

---

# 40. Ejemplo incorrecto

```text
TemperatureTask
HumidityTask
PressureTask
LightTask
RelayTask
FanTask
MQTTTask
WebTask
JsonTask
ConfigTask
LogTask
DisplayTask
ButtonTask
```

Un sistema pequeño puede terminar con demasiadas tareas.

---

# 41. Ejemplo recomendado

```text
SystemTask
NetworkTask
CommunicationTask
SensorTask
ActuatorTask
AutomationTask
UITask
StorageTask
DiagnosticTask
```

Y solamente incluir las necesarias.

---

# 42. Arquitectura para un Node simple

Ejemplo:

```text
ESP32
│
├── SystemTask
├── NetworkTask
├── SensorTask
├── ActuatorTask
├── AutomationTask
└── DiagnosticTask
```

---

# 43. Arquitectura para Node con Ethernet + RS485

```text
ESP32
│
├── SystemTask
├── EthernetTask
├── CommunicationTask
├── ModbusTask
├── SensorTask
├── ActuatorTask
├── AutomationTask
└── DiagnosticTask
```

---

# 44. Arquitectura para Display

```text
ESP32-S3
│
├── SystemTask
├── NetworkTask
├── CommunicationTask
├── UITask
├── TouchTask
├── AutomationTask
├── StorageTask
└── DiagnosticTask
```

Touch y UI pueden combinarse si no existe una razón para separarlos.

---

# 45. Arquitectura para cámara/IA

```text
ESP32-S3
│
├── SystemTask
├── CameraTask
├── AIProcessingTask
├── NetworkTask
├── CommunicationTask
├── AutomationTask
└── DiagnosticTask
```

El procesamiento de imagen debe estar desacoplado de la red.

---

# 46. Arquitectura para Central

El Central puede requerir más servicios:

```text
ESP32-S3
│
├── SystemTask
├── NetworkTask
├── CommunicationTask
├── DiscoveryTask
├── AutomationTask
├── API Task
├── WebSocket Task
├── StorageTask
├── HistoryTask
├── OTA Task
├── IntegrationTask
├── DiagnosticTask
└── WatchdogTask
```

Sin embargo, esto no significa necesariamente que cada elemento deba ser una tarea independiente.

Servicios con poca carga pueden compartir tareas.

---

# 47. Regla especial para el Central

El Central debe evitar convertirse en un sistema monolítico.

Debe estar dividido en:

```text
Core
Services
Modules
Tasks
Drivers
```

El hecho de que el Central administre todo el sistema no significa que deba ejecutar todas las operaciones.

---

# 48. Eventos frente a polling

Preferir:

```text
EVENT-DRIVEN
```

cuando exista un evento físico o lógico.

Ejemplos:

```text
Button pressed
Door opened
Motion detected
CAN message received
Modbus response received
Ethernet connected
```

Utilizar polling cuando:

* el sensor no dispone de interrupción;
* la lectura requiere periodicidad;
* el hardware lo exige;
* resulte más eficiente.

---

# 49. Timer Services

Los timers deben utilizarse para:

* retardos;
* expiración;
* acciones temporizadas;
* periodicidad ligera.

No utilizar un timer para ejecutar grandes cantidades de procesamiento.

Preferentemente:

```text
Timer
 ↓
Event
 ↓
Task
```

---

# 50. Watchdog y Health Manager

Se recomienda un servicio:

```text
HealthManager
```

que supervise:

```text
Task heartbeat
Network
Sensors
Actuators
Communication
Memory
Storage
```

Ejemplo:

```text
SensorTask
   │
   └── heartbeat

AutomationTask
   │
   └── heartbeat

CommunicationTask
   │
   └── heartbeat
```

HealthManager verifica que estén activas.

---

# 51. Registro de errores

Los errores deben ser estructurados.

Ejemplo:

```json
{
  "module": "modbus",
  "code": "TIMEOUT",
  "severity": "WARNING",
  "timestamp": 123456
}
```

Esto permite que el portal web muestre información comprensible.

---

# 52. Métricas de ejecución

Cada dispositivo debería poder reportar:

```text
CPU usage
Free heap
Minimum heap
PSRAM
Task stack watermark
Queue usage
Uptime
Reset reason
Watchdog status
Network latency
```

Esto permite diagnóstico sin conectar un programador.

---

# 53. Configuración desde el portal

La arquitectura FreeRTOS no debe quedar expuesta al usuario común.

El usuario verá:

```text
Rendimiento
✓ Normal
```

Mientras el instalador puede ver:

```text
Tasks: 8
CPU: 27 %
Free heap: 142 KB
PSRAM: 3.2 MB
```

Y el desarrollador:

```text
Task
Priority
Core
Stack
Runtime
State
```

---

# 54. FreeRTOS no debe definir la lógica de negocio

FreeRTOS es un mecanismo de ejecución.

No debe convertirse en la arquitectura de la aplicación.

La separación correcta es:

```text
Business Logic
      ↓
Services
      ↓
RTOS abstraction
      ↓
FreeRTOS
      ↓
Hardware
```

Esto permite mantener la plataforma portable.

---

# 55. Regla de oro

> **Crear tareas por responsabilidades y necesidades de ejecución, no por cantidad de módulos o dispositivos físicos.**

---

# 56. Arquitectura de referencia

```text
                     SYSTEM
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       NETWORK      COMMUNICATION  HEALTH
          │            │
          │            ├──── MQTT
          │            ├──── CAN
          │            ├──── Modbus
          │            └──── System Bus
          │
          └────────────┐
                       ▼
                    SENSOR
                       │
                       ▼
                    STATE
                       │
                       ▼
                  AUTOMATION
                       │
                       ▼
                   COMMAND
                       │
                       ▼
                   ACTUATOR
                       │
                       ▼
                   HARDWARE
```

---

# 57. Objetivo final

Todos los dispositivos de la plataforma deben utilizar una filosofía común:

```text
Hardware
   ↓
Driver
   ↓
Service
   ↓
State/Event
   ↓
Automation
   ↓
Command
   ↓
Actuator
```

FreeRTOS debe proporcionar la ejecución concurrente y determinista necesaria para implementar esta arquitectura, pero no debe convertirse en una dependencia conceptual de cada módulo.

---

# 58. Resumen de reglas obligatorias

1. No crear tareas sin justificación.
2. No crear una tarea por cada sensor.
3. No crear una tarea por cada relay.
4. Preferir arquitectura event-driven.
5. Utilizar queues para comunicación entre tareas.
6. Utilizar notifications cuando sean suficientes.
7. Utilizar mutex para recursos compartidos.
8. Mantener ISR extremadamente pequeñas.
9. Evitar bloqueos largos.
10. Toda comunicación debe tener timeout.
11. Separar adquisición de sensores de automatización.
12. Separar automatización de actuadores.
13. Separar almacenamiento de tareas críticas.
14. No asumir dual-core.
15. Utilizar Core Affinity solamente cuando exista una razón.
16. Supervisar stack y memoria.
17. Utilizar watchdog como mecanismo de recuperación.
18. Permitir modo degradado.
19. Mantener configuración crítica local.
20. Mantener la lógica de negocio independiente de FreeRTOS.

---

# 59. Principio definitivo

> **FreeRTOS debe organizar la ejecución del sistema, no definir cómo funciona el sistema.**

La arquitectura debe permitir que un mismo servicio pueda ejecutarse en diferentes ESP32 y diferentes configuraciones de hardware sin modificar su lógica fundamental.

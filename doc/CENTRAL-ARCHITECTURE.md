# CENTRAL-ARCHITECTURE.md

# Arquitectura del Controlador Central

> **Tipo:** Especificación de arquitectura
> **Estado:** Planificación / Diseño
> **Versión:** 1.0.0
> **Objetivo:** Definir el concepto de controlador central basado completamente en ESP32.

---

# 1. Objetivo

La plataforma debe poder funcionar sin necesidad de:

* PC.
* Raspberry Pi.
* NAS.
* Servidor Linux.
* Servidor Windows.
* Cloud.
* Internet.

El sistema completo debe poder construirse utilizando únicamente dispositivos basados en ESP32.

El controlador central será un **ESP32 con funciones avanzadas de coordinación, configuración, almacenamiento y administración**.

---

# 2. Definición de "Central"

En esta plataforma:

> **Central es un rol lógico, no necesariamente un tipo de hardware.**

Un ESP32 puede actuar como:

```text
SENSOR NODE
ACTUATOR NODE
DISPLAY NODE
GATEWAY
ZONE CONTROLLER
CENTRAL CONTROLLER
```

dependiendo de su configuración y capacidades.

Por lo tanto:

```text
Hardware
   ↓
ESP32
   ↓
Role
   ↓
Capabilities
```

---

# 3. Central basado en ESP32

El controlador central será preferentemente un:

```text
ESP32-S3
```

con recursos suficientes para ejecutar:

* Web Server.
* REST API.
* WebSocket.
* Device Registry.
* Discovery.
* Automation Engine.
* Scene Engine.
* User Management.
* Configuration Management.
* Local Database.
* Event Bus.
* MQTT Broker opcional.
* OTA Management.
* Diagnostics.
* Network Management.
* Backup.
* Local UI.
* Security.
* Time/NTP.
* Logging.

---

# 4. Hardware recomendado

La implementación inicial debería priorizar:

```text
ESP32-S3
+
PSRAM
+
Flash suficiente
+
Ethernet preferentemente
+
microSD opcional
```

La microSD no debe ser obligatoria para las funciones básicas.

Puede utilizarse para:

* Históricos.
* Logs.
* Backups.
* Configuraciones.
* Firmware.
* Archivos de UI.
* Datos de mayor volumen.

---

# 5. Por qué ESP32-S3

El ESP32-S3 es especialmente interesante como Central debido a:

* Mayor capacidad de procesamiento que los nodos simples.
* Dual Core.
* FreeRTOS.
* PSRAM en determinados módulos.
* Wi-Fi.
* Bluetooth LE.
* USB en determinados diseños.
* Capacidad suficiente para servidor web.
* Capacidad suficiente para APIs.
* Procesamiento local.
* Posibilidad de utilizar Ethernet mediante hardware externo.
* Capacidad para tareas relacionadas con IA/visión en otros escenarios.

No se debe asumir que todos los ESP32-S3 tienen las mismas prestaciones.

El sistema debe detectar las capacidades reales del hardware.

---

# 6. Central no significa "cerebro obligatorio"

El Central debe coordinar el sistema.

No debe ser necesario para ejecutar todas las acciones.

Incorrecto:

```text
Sensor
  ↓
Central
  ↓
Decision
  ↓
Actuator
```

para absolutamente todo.

Correcto:

```text
                     CENTRAL
                        │
                 configuration
                 coordination
                 history
                 management
                        │
                        │
        ┌───────────────┼───────────────┐
        │               │               │
      ZONE A          ZONE B          ZONE C
        │               │               │
     local            local           local
   automation       automation      automation
```

---

# 7. Funciones del Central

El Central debe encargarse principalmente de:

## Administración

* Usuarios.
* Roles.
* Permisos.
* Dispositivos.
* Zonas.
* Grupos.
* Sectores.
* Escenas.
* Automatizaciones.
* Integraciones.

## Configuración

* Hardware.
* Red.
* Protocolos.
* Módulos.
* Sensores.
* Actuadores.
* Pantallas.

## Supervisión

* Estado de nodos.
* Estado de red.
* Alarmas.
* Diagnóstico.
* Recursos.
* Firmware.

## Coordinación

* Automatizaciones globales.
* Escenas globales.
* Modos.
* Sincronización.
* Discovery.

## Persistencia

* Configuración.
* Historial.
* Eventos.
* Logs.
* Usuarios.

---

# 8. Funciones que NO deben depender exclusivamente del Central

No deben depender exclusivamente del Central:

* Encendido de luces críticas.
* Apagado de equipos peligrosos.
* Alarmas.
* Detección de fugas.
* Control de temperatura crítico.
* Automatizaciones básicas.
* Interruptores.
* Botones.
* Seguridad.
* Actuaciones locales.

Estas funciones deben poder ejecutarse localmente.

---

# 9. Arquitectura general

La arquitectura recomendada es:

```text
                         INTERNET
                            │
                         OPCIONAL
                            │
                    ┌───────▼───────┐
                    │ CENTRAL ESP32 │
                    │     S3        │
                    └───────┬───────┘
                            │
              ┌─────────────┼─────────────┐
              │             │             │
              ▼             ▼             ▼
           ZONA A        ZONA B        ZONA C
              │             │             │
         ┌────┼────┐   ┌────┼────┐   ┌────┼────┐
         │    │    │   │    │    │   │    │    │
        ESP  ESP  ESP ESP  ESP  ESP ESP  ESP  ESP
```

---

# 10. Arquitectura completamente ESP32

No se debe requerir obligatoriamente:

```text
Raspberry Pi
PC
Mini PC
NAS
Cloud
```

El sistema completo puede ser:

```text
ESP32 Central
     │
ESP32 Nodes
     │
ESP32-C6
     │
ESP32-S3
     │
ESP32-WROOM
```

---

# 11. Roles de los diferentes ESP32

La plataforma debe aprovechar las características de cada familia.

## ESP32-WROOM

Ideal para:

* Sensores.
* Relés.
* GPIO.
* Ethernet.
* RS485.
* CAN.
* Nodos simples.
* Automatización local.

## ESP32-C6

Ideal para:

* Matter.
* Thread.
* Zigbee.
* Wi-Fi 6.
* Gateways.
* Redes inalámbricas.
* Integraciones.

## ESP32-S3

Ideal para:

* Central.
* Pantallas.
* Cámaras.
* IA.
* Interfaces avanzadas.
* Procesamiento.
* Gateways avanzados.

---

# 12. Central con Ethernet

Para una instalación permanente se recomienda que el Central utilice Ethernet cuando sea posible.

Ejemplo:

```text
                ROUTER / SWITCH
                       │
                    Ethernet
                       │
                CENTRAL ESP32
                       │
          ┌────────────┼────────────┐
          │            │            │
        Wi-Fi         CAN         RS485
          │            │            │
       ESP32-C6       Nodes        Nodes
```

Esto evita que el dispositivo que administra toda la instalación dependa exclusivamente de Wi-Fi.

---

# 13. Central inalámbrico

También debe ser posible utilizar:

```text
ESP32-S3 Central
       │
      Wi-Fi
       │
     Router
```

para instalaciones pequeñas.

Sin embargo, Ethernet debe considerarse preferible para instalaciones permanentes y críticas.

---

# 14. Central como servidor web

El ESP32 Central debe alojar la interfaz web.

Por ejemplo:

```text
http://automation.local
```

La interfaz debe permitir:

```text
Dashboard
Devices
Zones
Groups
Security
Scenes
Automations
Energy
Water
Environment
Network
Users
Integrations
Diagnostics
Firmware
```

---

# 15. API local

El Central debe proporcionar una API REST local.

Ejemplo:

```text
GET /api/system
GET /api/devices
GET /api/resources
GET /api/zones
GET /api/groups
GET /api/scenes
GET /api/automations
GET /api/events
GET /api/security
```

Y comandos:

```text
POST /api/devices/{id}/command
POST /api/scenes/{id}/activate
POST /api/automations/{id}/enable
POST /api/security/mode
```

---

# 16. WebSocket

La interfaz debe poder utilizar WebSocket para información en tiempo real.

Ejemplo:

```text
ESP32 Node
   │
   │ EVENT
   ▼
Central
   │
   │ WebSocket
   ▼
Browser
```

Esto permite actualizar:

* Sensores.
* Luces.
* Alarmas.
* Estado de dispositivos.
* Diagnóstico.

sin realizar polling constante.

---

# 17. Base de datos

El Central debe disponer de almacenamiento persistente.

La arquitectura debe abstraer el almacenamiento:

```text
Storage API
    │
    ├── NVS
    ├── LittleFS
    ├── SPIFFS si fuera necesario
    └── SD
```

No se debe acoplar toda la aplicación a un único sistema de archivos.

---

# 18. Datos que deben persistir

Como mínimo:

```text
Device registry
Configuration
Users
Permissions
Zones
Groups
Scenes
Automations
Security modes
Schedules
Calibration
System settings
```

Los datos de mayor volumen pueden almacenarse opcionalmente en:

```text
microSD
```

---

# 19. Históricos

El sistema debe diferenciar:

### Configuración

Debe ser altamente persistente.

### Estado actual

Debe mantenerse localmente.

### Eventos

Puede almacenarse temporalmente y sincronizarse.

### Histórico

Puede almacenarse en:

```text
microSD
```

cuando el volumen sea elevado.

---

# 20. Central como MQTT Broker

El ESP32 Central puede incorporar opcionalmente un MQTT broker local.

Ejemplo:

```text
ESP32 Node
     │
     ▼
MQTT Broker
     │
     ├── Central
     ├── Dashboard
     ├── Automation
     └── Integrations
```

Esto permitiría mantener MQTT completamente dentro de la red local.

Sin Internet.

---

# 21. MQTT no debe ser obligatorio para el núcleo

El sistema debe diferenciar:

```text
System Bus
```

de:

```text
MQTT
```

MQTT puede ser una implementación de transporte o integración.

La lógica del sistema no debe depender de MQTT.

---

# 22. System Bus

El Central debe proporcionar o participar en un:

```text
SYSTEM BUS
```

para:

* Events.
* Commands.
* States.
* Discovery.
* Configuration.
* Heartbeats.
* Alarms.

Los nodos pueden comunicarse:

```text
Node
 ↓
System Bus
 ↓
Central
```

o directamente:

```text
Node A
 ↓
System Bus
 ↓
Node B
```

---

# 23. Comunicación directa

El Central no debe ser obligatorio como intermediario para todas las comunicaciones.

Ejemplo:

```text
PIR
 │
 └──────────────► Light
```

El Central puede observar:

```text
PIR
 ↓
Central
```

pero no necesita intervenir para ejecutar:

```text
PIR → Light
```

---

# 24. Automatizaciones locales

Cada nodo debe poder ejecutar automatizaciones simples.

Ejemplo:

```text
IF PIR == ON
THEN LIGHT == ON
```

Esto puede ejecutarse completamente dentro del ESP32.

---

# 25. Automatizaciones de zona

Cuando una automatización involucra varios nodos:

```text
PIR Bedroom
      │
      ▼
Zone Controller
      │
      ▼
Light Bedroom
```

Puede ejecutarse mediante un controlador de zona.

---

# 26. Automatizaciones globales

Las automatizaciones que involucran toda la instalación pueden ejecutarse en el Central.

Ejemplo:

```text
Scene:
LEAVE_HOME

Actions:

All lights OFF
Security AWAY
HVAC ECO
Garage CLOSED
Water SAFE
```

Sin embargo, cada acción crítica debe tener fallback local.

---

# 27. Copia distribuida de automatizaciones

Una automatización global importante puede tener:

```text
Central copy
+
Zone copy
```

o incluso:

```text
Central copy
+
Zone copy
+
Node fallback
```

Esto depende de su criticidad.

---

# 28. Ejemplo: modo dormir

El Central puede establecer:

```text
SYSTEM.MODE = SLEEP
```

Las zonas reciben:

```text
SLEEP
```

Pero cada zona puede continuar utilizando el modo aunque el Central desaparezca.

Ejemplo:

```text
Central
  X

Bedroom
  ↓
Last valid mode = SLEEP
  ↓
Local rules continue
```

---

# 29. Ejemplo: alarma

La alarma no debe depender del Central.

Arquitectura:

```text
PIR
 ↓
Security Zone Controller
 ↓
Alarm
```

El Central solamente agrega:

```text
Logging
UI
Notifications
History
```

---

# 30. Ejemplo: iluminación

Una luz puede funcionar:

```text
Button
 ↓
Local ESP32
 ↓
Relay
```

aunque:

```text
Central OFF
Internet OFF
```

---

# 31. Ejemplo: climatización

```text
Temperature
      ↓
Local controller
      ↓
HVAC
```

El Central puede modificar:

```text
Setpoint
Mode
Schedule
```

pero si desaparece:

```text
Local controller
      ↓
continúa
```

con la última configuración válida.

---

# 32. Central redundante

Para instalaciones grandes se puede utilizar más de un Central.

Ejemplo:

```text
              CENTRAL A
              ESP32-S3
                  │
                  │
              CENTRAL B
              ESP32-S3
```

Uno puede actuar como:

```text
PRIMARY
```

y otro:

```text
BACKUP
```

---

# 33. Central A + Central B

La configuración puede mantenerse sincronizada:

```text
Central A
   │
   ├── Config
   ├── Users
   ├── Automations
   └── History
          │
          ▼
      Central B
```

Si A falla:

```text
Central A
    X

Central B
    ↓
ACTIVE
```

---

# 34. Limitación de Central redundante

No debe implementarse desde el principio si aumenta demasiado la complejidad.

La arquitectura debe soportarlo, pero la primera versión puede utilizar:

```text
1 Central
+
autonomía distribuida
```

Esto ya evita que el Central sea un punto único de fallo.

---

# 35. Failover

Cuando el Central desaparece:

```text
Central OFFLINE
      ↓
Nodes detect timeout
      ↓
Local automation continues
      ↓
Zone controllers continue
      ↓
Critical functions continue
```

No debe existir un:

```text
GLOBAL SYSTEM OFF
```

por la simple pérdida del Central.

---

# 36. Recuperación del Central

Cuando vuelve:

```text
Central BOOT
     ↓
Network
     ↓
Discovery
     ↓
Authentication
     ↓
State synchronization
     ↓
Configuration comparison
     ↓
Pending events
     ↓
Normal operation
```

Los nodos no deberían reiniciarse innecesariamente.

---

# 37. Central offline durante una actualización

Si el Central está actualizándose:

```text
Central
   ↓
OTA
   ↓
REBOOT
```

las zonas deben seguir funcionando.

Esto requiere que:

```text
Local automation
```

no dependa de:

```text
Central automation
```

para funciones críticas.

---

# 38. Central no debe bloquear nodos

Nunca se debe implementar:

```text
Node boot
 ↓
Wait for Central
 ↓
Central unavailable
 ↓
Node unusable
```

Debe ser:

```text
Node boot
 ↓
Load local configuration
 ↓
Start local services
 ↓
Try Central
 ↓
Central available?
 ├── YES → synchronize
 └── NO  → continue locally
```

---

# 39. Provisionamiento inicial

Cuando se instala un nodo:

```text
New ESP32
    ↓
Provisioning
    ↓
Network
    ↓
Discovery
    ↓
Central
    ↓
Assign role
    ↓
Assign zone
    ↓
Configure resources
```

Una vez configurado, el nodo debe almacenar la configuración necesaria localmente.

---

# 40. Pérdida del Central durante el funcionamiento

Ejemplo:

```text
08:00 Central ONLINE

08:10 Central OFFLINE

08:11 PIR detects motion

08:11 Local automation executes

08:12 Light turns ON

08:30 Central returns

08:31 Event synchronization
```

El evento puede quedar almacenado localmente durante la desconexión.

---

# 41. Central como coordinador de estados

El Central puede mantener una visión global:

```text
SYSTEM
│
├── Mode
├── Security
├── Energy
├── Climate
├── Water
└── Devices
```

Pero los nodos deben mantener una copia local de los estados necesarios.

---

# 42. Último estado válido

Si un nodo pierde comunicación:

```text
Last valid configuration
```

debe continuar activa.

Ejemplo:

```text
Temperature target = 24 °C
```

Si Central desaparece:

```text
Target = 24 °C
```

continúa siendo válido.

---

# 43. Prioridades

La arquitectura debe definir:

```text
LOCAL SAFETY
     >
LOCAL AUTOMATION
     >
ZONE AUTOMATION
     >
CENTRAL AUTOMATION
     >
EXTERNAL INTEGRATIONS
```

Esto evita que una orden externa sobrescriba una condición de seguridad.

---

# 44. Integraciones externas

Las integraciones como:

```text
Home Assistant
Homey Pro
Apple Home
Google Home
SmartThings
Matter Cloud
Weather APIs
Cloud services
```

son opcionales.

Su desconexión no debe detener:

```text
ESP32
Local automation
Security
Zones
Scenes
```

---

# 45. Internet como servicio externo

Internet debe considerarse:

```text
SERVICE
```

no:

```text
CORE DEPENDENCY
```

Puede proporcionar:

* Remote access.
* Cloud.
* Notifications.
* Weather.
* Updates.
* External integrations.
* Remote monitoring.

---

# 46. Acceso remoto

Cuando Internet está disponible:

```text
User
 ↓
Internet
 ↓
VPN / secure access
 ↓
Central
```

Cuando Internet desaparece:

```text
Remote access
      X
```

pero:

```text
Local system
      ↓
CONTINUES
```

---

# 47. Red local

El sistema debe continuar siendo administrable mediante:

```text
Ethernet
Wi-Fi
Local touchscreen
Local API
```

aunque Internet esté desconectado.

---

# 48. Modo de emergencia

Si existe una falla grave:

```text
Central OFF
+
Network problems
```

los nodos deben pasar a un estado definido por su configuración.

Por ejemplo:

```text
Security → ACTIVE
Heating → SAFE
Water → CLOSED
Critical loads → OFF
Emergency lighting → ON
```

---

# 49. Health Monitoring

El Central debe supervisar:

```text
CPU
RAM
Flash
Network
Heartbeat
Temperature
Uptime
Errors
Watchdog
Firmware
Configuration
```

de cada nodo.

---

# 50. Distribución de carga

El Central no debe ejecutar obligatoriamente todas las tareas.

Puede delegar:

```text
Security → Security Zone
Climate → Climate Zone
Energy → Energy Zone
Lighting → Local Nodes
```

Esto permite escalar.

---

# 51. Arquitectura jerárquica

La arquitectura final puede ser:

```text
                        CENTRAL
                       ESP32-S3
                           │
          ┌────────────────┼────────────────┐
          │                │                │
      SECURITY          ENERGY          CLIMATE
      CONTROLLER        CONTROLLER       CONTROLLER
          │                │                │
       ESP32s            ESP32s           ESP32s
```

Cada controlador secundario también puede ser un ESP32.

---

# 52. Zone Controller

Un Zone Controller es un ESP32 con una responsabilidad superior a un nodo normal.

Puede:

* Coordinar varios nodos.
* Ejecutar automatizaciones de zona.
* Mantener estados.
* Mantener escenas.
* Mantener reglas.
* Realizar failover.
* Actuar como gateway.

No necesita ser un hardware diferente.

---

# 53. Central vs Zone Controller

### Central

Administra:

```text
Toda la instalación
```

### Zone Controller

Administra:

```text
Una zona
```

### Node

Administra:

```text
Sus propios recursos
```

---

# 54. Ejemplo completo

```text
                     CENTRAL
                   ESP32-S3
                       │
       ┌───────────────┼────────────────┐
       │               │                │
    SECURITY          HOUSE           GARDEN
    CONTROLLER      CONTROLLER       CONTROLLER
       │               │                │
    ┌──┼──┐         ┌──┼──┐         ┌──┼──┐
    │  │  │         │  │  │         │  │  │
   N1 N2 N3        N4 N5 N6        N7 N8 N9
```

---

# 55. Si falla el Central

```text
                     CENTRAL
                        X

       ┌───────────────┼────────────────┐
       │               │                │
    SECURITY          HOUSE           GARDEN
    CONTROLLER      CONTROLLER       CONTROLLER
       │               │                │
    ┌──┼──┐         ┌──┼──┐         ┌──┼──┐
    │  │  │         │  │  │         │  │  │
   N1 N2 N3        N4 N5 N6        N7 N8 N9
```

Resultado:

```text
Security → continúa
House → continúa
Garden → continúa
Local automation → continúa
```

Se pierde:

```text
Global administration
Global dashboard
Central history
External integrations
```

hasta que el Central vuelva o exista un Central secundario.

---

# 56. Si falla un Zone Controller

Ejemplo:

```text
HOUSE CONTROLLER
      X
```

Los nodos pueden:

```text
continuar localmente
```

y, si la función lo permite:

```text
otro Zone Controller
```

puede asumir la coordinación.

---

# 57. Si falla un nodo

Ejemplo:

```text
N5 OFFLINE
```

solo deben verse afectadas las funciones dependientes de:

```text
N5
```

Los demás nodos continúan.

---

# 58. Diseño sin punto único de fallo

El objetivo no es eliminar absolutamente todos los puntos de fallo.

Eso aumentaría demasiado la complejidad y el coste.

El objetivo es eliminar los puntos de fallo que puedan provocar:

> **la pérdida completa de la instalación.**

---

# 59. Coste

Una de las ventajas principales de esta arquitectura es que no requiere:

```text
PC 24/7
Raspberry Pi
Servidor dedicado
Cloud obligatorio
Licencias de servidor
```

Un ESP32-S3 puede funcionar continuamente con un consumo muy bajo comparado con un ordenador.

---

# 60. Mantenimiento

El mantenimiento también se simplifica:

```text
Central ESP32
+
Nodes ESP32
```

Todos pueden compartir:

* Framework.
* OTA.
* Logs.
* API.
* Device Model.
* Configuración.
* Herramientas.
* Sistema de diagnóstico.

---

# 61. Limitaciones

El hecho de utilizar ESP32 para todo implica aceptar determinadas limitaciones.

El Central no debe pretender reemplazar completamente a un servidor empresarial.

Las principales limitaciones son:

* RAM limitada.
* Flash limitada.
* Almacenamiento limitado.
* CPU limitada.
* Base de datos limitada.
* Número limitado de conexiones simultáneas.
* Capacidad limitada para históricos masivos.
* Capacidad limitada para IA avanzada.
* Capacidad limitada para interfaces web extremadamente grandes.

Por eso la arquitectura debe ser eficiente.

---

# 62. Solución a las limitaciones

Cuando la instalación crezca:

```text
1 Central
```

puede convertirse en:

```text
2 Centrals
```

y posteriormente:

```text
Central
+
Zone Controllers
+
Distributed Nodes
```

sin cambiar el modelo lógico.

---

# 63. Escalabilidad

### Instalación pequeña

```text
1 ESP32-S3 Central
+
5-20 ESP32 nodes
```

### Instalación media

```text
1 ESP32-S3 Central
+
3-10 Zone Controllers
+
decenas de nodes
```

### Instalación grande

```text
2+ ESP32-S3 Centrals
+
Zone Controllers
+
Gateways
+
muchos nodes
```

Los números reales dependerán del protocolo, tráfico, automatizaciones y hardware.

---

# 64. Central múltiple

El sistema debe soportar múltiples Centrals lógicos.

Por ejemplo:

```text
SITE
│
├── CENTRAL A
├── CENTRAL B
│
├── ZONE 1
├── ZONE 2
├── ZONE 3
└── ZONE 4
```

Los Centrals pueden actuar como:

```text
Primary
Backup
Load Balancer
Regional Controller
```

según la implementación futura.

---

# 65. Distribución geográfica

La misma arquitectura puede utilizarse para:

```text
Casa
```

o:

```text
Edificio
```

o:

```text
Campo
```

o:

```text
Invernadero
```

o:

```text
Barco
```

donde cada sector puede tener su propio controlador.

---

# 66. Central y SEMA

SEMA puede utilizar el Central ESP32 para:

```text
Weather dashboard
Historical data
API
External services
Device management
```

pero los sensores meteorológicos pueden seguir funcionando localmente.

---

# 67. Central e Invernadero

El Invernadero puede utilizar:

```text
Central ESP32
```

para coordinar:

* Clima.
* Riego.
* Iluminación.
* Ventilación.
* pH.
* EC.
* CO₂.

Pero cada controlador local debe poder proteger la instalación.

Ejemplo:

```text
Temperature too high
        ↓
Local ventilation
```

sin esperar al Central.

---

# 68. Arquitectura recomendada para el proyecto

La arquitectura base debe quedar:

```text
                    ESP32 CENTRAL
                         │
              ┌──────────┼──────────┐
              │          │          │
           SERVICES   AUTOMATION   API
              │          │          │
              └──────────┼──────────┘
                         │
                     SYSTEM BUS
                         │
          ┌──────────────┼──────────────┐
          │              │              │
      ZONE CTRL       ZONE CTRL      ZONE CTRL
          │              │              │
       ESP32s          ESP32s         ESP32s
          │              │              │
      Sensors         Sensors        Sensors
      Actuators       Actuators      Actuators
```

---

# 69. Regla fundamental

El Central debe mejorar el sistema:

```text
Central ON
    ↓
más funciones
```

pero no debe definir si el sistema puede funcionar:

```text
Central OFF
    ↓
sistema continúa
```

---

# 70. Regla de diseño

Toda funcionalidad nueva debe responder:

### ¿Puede ejecutarse localmente?

Si sí:

```text
Implementar localmente.
```

### ¿Necesita coordinación?

Entonces:

```text
Zone Controller
```

### ¿Necesita visión global?

Entonces:

```text
Central
```

### ¿Necesita Internet?

Entonces:

```text
External Service
```

Nunca al revés.

---

# 71. Jerarquía definitiva

La plataforma debe utilizar la siguiente jerarquía:

```text
EXTERNAL / CLOUD
       │
       ▼
CENTRAL
       │
       ▼
ZONE CONTROLLER
       │
       ▼
DEVICE
       │
       ▼
RESOURCE
```

Pero la ejecución puede ocurrir en sentido contrario:

```text
RESOURCE
   ↓
DEVICE
   ↓
ZONE
   ↓
CENTRAL
   ↓
EXTERNAL
```

dependiendo de la función.

---

# 72. Principio de autonomía

La prioridad de diseño debe ser:

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

Cuanto más crítica sea una función, más abajo debe ejecutarse.

---

# 73. Arquitectura final

La plataforma se define como:

> **Una plataforma de automatización distribuida donde todos los componentes pueden estar construidos sobre ESP32, existiendo un controlador central opcional basado preferentemente en ESP32-S3, controladores de zona y nodos especializados.**

El Central proporciona:

```text
Administración
Configuración
API
Web
Histórico
Usuarios
Automatizaciones globales
Discovery
Integraciones
Coordinación
```

Los Zone Controllers proporcionan:

```text
Automatización regional
Estados locales
Coordinación
Failover
Gateway
```

Los Nodes proporcionan:

```text
Sensores
Actuadores
IO
Automatización local
Seguridad
```

Y todos pueden continuar funcionando con distintos niveles de autonomía.

---

# 74. Objetivo final

La instalación ideal debe funcionar así:

```text
                         INTERNET
                            X
                            │
                     CENTRAL ESP32
                            X
                            │
             ┌──────────────┼──────────────┐
             │              │              │
           ZONA A         ZONA B         ZONA C
             │              │              │
          ESP32s          ESP32s          ESP32s
             │              │              │
          LOCAL           LOCAL           LOCAL
        AUTOMATION      AUTOMATION      AUTOMATION
```

Aunque Internet y el Central estén fuera de servicio:

```text
ZONA A → funciona
ZONA B → funciona
ZONA C → funciona
```

y cuando el Central vuelve:

```text
Central
   ↓
Synchronize
   ↓
Recover
   ↓
Continue
```

sin necesidad de reiniciar toda la instalación.

---

# 75. Decisión arquitectónica

A partir de esta especificación:

> **El proyecto no requerirá un servidor externo para funcionar.**

El hardware base del ecosistema será:

```text
ESP32
```

El controlador central preferido será:

```text
ESP32-S3 + PSRAM
```

Los controladores de zona podrán utilizar:

```text
ESP32-WROOM
ESP32-S3
ESP32-C6
```

según sus necesidades.

Los servidores externos, PCs, Raspberry Pi, NAS o servicios Cloud podrán utilizarse posteriormente como **extensiones opcionales**, pero nunca serán requisitos fundamentales del sistema.

---

# 76. Relación con otros documentos

Este documento debe considerarse complementario a:

```text
docs/
├── SYSTEM-ARCHITECTURE.md
├── ARCHITECTURE.md
├── DEVICE-MODEL.md
├── COMMUNICATION.md
├── MODULE-DEVELOPMENT.md
├── FAULT-TOLERANCE.md
└── CENTRAL-ARCHITECTURE.md
```

`CENTRAL-ARCHITECTURE.md` define específicamente:

* Qué es el Central.
* Qué hardware puede utilizar.
* Qué responsabilidades tiene.
* Qué no debe hacer.
* Cómo se relaciona con los Zone Controllers.
* Cómo se relaciona con los nodos.
* Cómo se comporta cuando falla.
* Cómo escalar a múltiples Centrals.

---

# 77. Decisión definitiva

La arquitectura debe seguir este principio:

> **El Central administra el sistema, pero no es dueño de la capacidad de funcionamiento del sistema.**

Por lo tanto:

```text
Central = Coordinador
Zone Controller = Coordinador regional
Node = Ejecutor local
Resource = Capacidad física
Internet = Servicio externo
Cloud = Integración opcional
```

La pérdida de cualquiera de los niveles superiores debe provocar únicamente la pérdida de las capacidades que realmente dependan de ellos.

Nunca debe provocar innecesariamente la pérdida completa de la automatización.

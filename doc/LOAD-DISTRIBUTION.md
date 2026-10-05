# Distribución de Carga y Arquitectura de Ejecución

> **Tipo:** Arquitectura
> **Estado:** Definición
> **Versión:** 1.0.0
> **Última actualización:** 2026-10-05

---

## 1. Objetivo

El sistema está diseñado para funcionar como una **plataforma distribuida de automatización**, no como un único controlador que deba procesar absolutamente todo.

El objetivo es que una instalación pueda crecer desde unos pocos dispositivos hasta instalaciones grandes sin que el usuario tenga que:

* modificar código fuente;
* recompilar firmware;
* decidir manualmente qué ESP32 ejecuta cada tarea;
* conocer qué protocolo utiliza cada dispositivo;
* configurar manualmente la distribución de memoria o CPU;
* preocuparse por qué dispositivo debe actuar como servidor;
* reemplazar toda la instalación porque un único dispositivo dejó de funcionar.

La plataforma debe encargarse automáticamente de distribuir las funciones disponibles entre los dispositivos.

### Principio fundamental

> **El usuario configura qué quiere que haga el sistema. La plataforma decide dónde y cómo ejecutar esas funciones.**

---

# 2. Decisión arquitectónica fundamental

No se intentará convertir al **ESP32-S3 Central** en un "servidor gigante" que haga absolutamente todo.

El Central no debe convertirse en:

```text
             TODO EL SISTEMA
                  │
                  ▼
          ESP32-S3 CENTRAL
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
    Sensores   Automat.   Actuadores
       │          │          │
       └──────────┴──────────┘
                  │
             Internet
```

Este diseño genera un **punto único de fallo**, limita la cantidad de dispositivos que pueden conectarse y hace que el rendimiento del sistema dependa excesivamente de un único ESP32.

En su lugar:

```text
                    INTERNET
                       │
                 ┌─────┴─────┐
                 │  CENTRAL   │
                 │ ESP32-S3   │
                 └─────┬─────┘
                       │
             ┌─────────┼─────────┐
             │         │         │
             ▼         ▼         ▼
          ZONA A     ZONA B    ZONA C
             │         │         │
          ┌──┴──┐   ┌──┴──┐   ┌──┴──┐
          ▼     ▼   ▼     ▼   ▼     ▼
        Node  Node Node  Node Node  Node
```

Cada dispositivo realiza las tareas para las que resulta más apropiado.

---

# 3. Principio "distribuir, no centralizar"

La arquitectura utilizará una estrategia de distribución jerárquica:

```text
Nivel 0
RECURSO
│
├── GPIO
├── Sensor
├── Relé
├── PWM
├── Display
├── Cámara
├── Medidor de energía
├── RS485
├── CAN
└── etc.

Nivel 1
NODE ESP32
│
├── Lectura
├── Control local
├── Protección
├── Automatización local
└── Comunicación

Nivel 2
ZONE CONTROLLER
│
├── Coordinación de zona
├── Automatizaciones de zona
├── Estado de zona
├── Gateway
└── Failover

Nivel 3
CENTRAL ESP32-S3
│
├── Administración
├── Configuración
├── Web UI
├── API
├── Usuarios
├── Discovery
├── Automatizaciones globales
├── Historial
└── Integraciones

Nivel 4
EXTERNO
│
├── Home Assistant
├── Homey
├── Matter
├── SmartThings
├── Google Home
├── Apple Home
└── Servicios externos
```

La responsabilidad debe colocarse en el nivel más bajo posible.

---

# 4. Regla de prioridad de ejecución

La regla general será:

```text
RECURSO LOCAL
      ↓
NODE
      ↓
ZONE CONTROLLER
      ↓
CENTRAL
      ↓
INTERNET / CLOUD
```

O expresado como prioridad:

```text
AUTONOMÍA LOCAL
    >
AUTOMATIZACIÓN DE ZONA
    >
AUTOMATIZACIÓN CENTRAL
    >
INTEGRACIONES EXTERNAS
```

Cuanto más crítica sea una función, más cerca del hardware debe ejecutarse.

---

# 5. ¿Qué debe hacer el Central?

El Central debe ser principalmente el **administrador y coordinador del sistema**, no el encargado de ejecutar cada operación individual.

Entre sus responsabilidades estarán:

### 5.1 Administración

* usuarios;
* contraseñas;
* permisos;
* roles;
* dispositivos;
* zonas;
* grupos;
* módulos;
* configuraciones;
* versiones;
* actualizaciones;
* diagnósticos.

### 5.2 Portal web

El Central alojará la interfaz principal del sistema.

El usuario podrá acceder desde:

```text
http://central.local
```

o mediante el nombre/IP configurado.

Desde allí podrá administrar toda la instalación.

### 5.3 Discovery

El Central detectará:

* nuevos ESP32;
* sensores;
* actuadores;
* gateways;
* displays;
* interfaces;
* módulos;
* capacidades.

Por ejemplo:

```text
Nuevo dispositivo encontrado

Nombre:
ESP32-S3-001

Modelo:
ESP32-S3

Capacidades:

✓ Wi-Fi
✓ Ethernet
✓ 8 GPIO
✓ I2C
✓ SPI
✓ UART
✓ PWM
✓ Cámara

[Agregar dispositivo]
```

El usuario no necesita modificar código.

---

# 6. El usuario no debería configurar "hardware", sino "funciones"

Esta es una de las decisiones más importantes de UX.

El sistema no debería obligar al usuario a pensar:

```text
GPIO 17
GPIO 18
GPIO 21
I2C bus 0
UART2
MCP23017 dirección 0x20
```

Eso debe quedar oculto para la mayoría de los usuarios.

En cambio debería mostrar:

```text
Habitación

┌───────────────────────────────┐
│ Luz principal                 │
│ Relé 1                        │
│                               │
│ [✓] Encendida                │
└───────────────────────────────┘

┌───────────────────────────────┐
│ Temperatura                   │
│ 23.4 °C                       │
│ Sensor: AHT30                 │
└───────────────────────────────┘

┌───────────────────────────────┐
│ Movimiento                    │
│ Sensor PIR                    │
│ Estado: Sin movimiento        │
└───────────────────────────────┘
```

La plataforma internamente mantiene la relación:

```text
Luz principal
      │
      ▼
Device: ESP32-XXXX
      │
      ▼
GPIO17
      │
      ▼
Relay Module
```

El usuario trabaja con **recursos lógicos**, no con pines.

---

# 7. Configuración mediante portal interactivo

Todo el sistema debe poder configurarse desde una interfaz web.

El flujo recomendado es:

```text
Instalar dispositivo
       │
       ▼
Encender
       │
       ▼
Discovery
       │
       ▼
Detectar hardware
       │
       ▼
Detectar capacidades
       │
       ▼
Usuario asigna función
       │
       ▼
Selecciona zona
       │
       ▼
Configura comportamiento
       │
       ▼
Guardar
       │
       ▼
Sistema genera configuración
       │
       ▼
ESP32 comienza a funcionar
```

No debería ser necesario:

```text
Modificar código
      ✗
Editar #define
      ✗
Cambiar GPIO en firmware
      ✗
Compilar
      ✗
Subir firmware
      ✗
```

---

# 8. El sistema debe separar configuración y firmware

Los dispositivos deben tener un firmware genérico.

Por ejemplo:

```text
ESP32 Universal Automation Firmware
```

El firmware conoce:

* GPIO;
* ADC;
* PWM;
* I2C;
* SPI;
* UART;
* RS485;
* CAN;
* Ethernet;
* Wi-Fi;
* sensores compatibles;
* actuadores compatibles;
* módulos disponibles.

Pero no necesita conocer previamente la instalación concreta.

La configuración determina qué función cumple cada recurso.

Ejemplo:

```json
{
  "device": "ESP32-001",
  "resources": [
    {
      "id": "gpio17",
      "function": "relay",
      "name": "Luz principal"
    },
    {
      "id": "i2c_0",
      "function": "aht30",
      "name": "Temperatura habitación"
    }
  ]
}
```

El firmware permanece igual.

La configuración cambia.

---

# 9. Distribución automática de carga

La plataforma debe mantener información sobre las capacidades y utilización de cada dispositivo.

Por ejemplo:

```text
ESP32-S3 CENTRAL

CPU:        42 %
RAM:        61 %
Flash:      48 %
Storage:    37 %
Web:        Activo
API:        Activo
History:    Activo
MQTT:       Activo
Automations: 14
```

Y:

```text
ZONE CONTROLLER 01

CPU:        21 %
RAM:        34 %
Automations: 8
Devices:     17
```

Y:

```text
NODE 07

CPU:        8 %
RAM:        18 %
Sensors:    4
Actuators:  2
```

Esta información permite determinar dónde es conveniente ejecutar una función.

---

# 10. Clasificación de tareas

Las tareas se dividirán en varias categorías.

## 10.1 Tareas críticas locales

Ejemplos:

* protección térmica;
* corte por sobrecorriente;
* control de ventilación;
* límites de temperatura;
* control de bombas;
* protección de motores;
* detección de estados peligrosos;
* temporizaciones críticas.

Estas funciones deben ejecutarse en el dispositivo local siempre que sea posible.

Ejemplo:

```text
Sensor temperatura
        │
        ▼
ESP32 local
        │
        ├── T > 80 °C
        │
        ▼
Apagar salida
```

No:

```text
Sensor
  │
  ▼
Central
  │
  ▼
Internet
  │
  ▼
Servidor
  │
  ▼
ESP32
```

---

# 11. Automatizaciones locales

Una automatización simple debería ejecutarse directamente en el dispositivo.

Ejemplo:

```text
SI
temperatura > 30 °C

ENTONCES
activar ventilador
```

El flujo será:

```text
Sensor
  │
  ▼
Node ESP32
  │
  ├── condición
  │
  ▼
Relay
```

Ventajas:

* latencia mínima;
* funcionamiento sin Central;
* funcionamiento sin Internet;
* menor tráfico;
* menor carga;
* mayor confiabilidad.

---

# 12. Automatizaciones de zona

Cuando una automatización involucra varios dispositivos de una misma zona, puede ejecutarse en un Zone Controller.

Ejemplo:

```text
Habitación

PIR
 │
 ├── Luz
 ├── Ventilador
 ├── Persiana
 └── Display
```

El Zone Controller puede coordinar:

```text
PIR detecta movimiento
        │
        ▼
Zone Controller
        │
        ├── Luz 30 %
        ├── Display ON
        ├── Ventilador 20 %
        └── Registrar evento
```

Si el Central está apagado, la zona continúa funcionando.

---

# 13. Automatizaciones globales

Las automatizaciones que involucren toda la instalación pueden ser administradas por el Central.

Ejemplo:

```text
ESCENA "SALIR DE CASA"

→ apagar luces
→ cerrar persianas
→ apagar climatización
→ activar alarma
→ cerrar válvula de agua
→ verificar puertas
→ enviar estado
```

El Central puede coordinar esta escena.

Pero los dispositivos no deben depender permanentemente de él para ejecutar las acciones críticas.

La escena puede distribuirse:

```text
Central
 │
 ├── Zona A → apagar luces
 │
 ├── Zona B → cerrar persianas
 │
 ├── Zona C → activar alarma
 │
 └── Nodo → cerrar válvula
```

Cada dispositivo ejecuta su parte.

---

# 14. Automatización compilada/distribuida

Una característica futura recomendada es que el sistema pueda transformar automáticamente una automatización global en pequeñas tareas locales.

Por ejemplo, el usuario crea:

```text
SI
hay movimiento en pasillo

Y
la casa está en modo NOCHE

ENTONCES
encender luz del pasillo al 15 %
durante 60 segundos
```

El usuario no sabe dónde se ejecutará.

El sistema puede determinar:

```text
PIR → Node A

Estado CASA.MODE → Zone Controller

Luz → Node B
```

Y distribuir:

```text
Node A
  │
  └── evento motion.detected
            │
            ▼
Zone Controller
  │
  ├── verificar NIGHT
  │
  ▼
Node B
  │
  └── PWM 15 %
```

El usuario únicamente ve:

```text
Movimiento → Modo Noche → Luz 15 % → 60 s
```

---

# 15. El Central como orquestador

El Central puede actuar como **orquestador**.

Su trabajo será determinar:

* qué dispositivos existen;
* qué capacidades tienen;
* qué recursos están disponibles;
* qué automatizaciones existen;
* qué zonas existen;
* qué dispositivos están disponibles;
* qué tareas pueden ejecutarse localmente;
* qué tareas necesitan coordinación;
* qué dispositivos están sobrecargados;
* qué rutas de comunicación están disponibles.

Pero no necesariamente ejecutará cada instrucción.

---

# 16. Principio de "ejecución más cercana al recurso"

Cuando sea posible, una función debe ejecutarse cerca del hardware que la necesita.

Ejemplo:

```text
                    CENTRAL
                       │
                  configuración
                       │
             ┌─────────┴─────────┐
             │                   │
          ZONA A              ZONA B
             │                   │
          NODE A              NODE B
             │                   │
          Sensor              Relay
```

Si Node A puede resolver una automatización sin ayuda:

```text
Sensor → Node A → Actuador
```

No debe utilizarse:

```text
Sensor → Node A → Central → Node B → Actuador
```

si no es necesario.

---

# 17. El Central no debe convertirse en cuello de botella

Un diseño donde todo pase por el Central puede provocar:

* saturación de CPU;
* saturación de RAM;
* demasiados mensajes;
* latencia;
* pérdida de eventos;
* mayor consumo;
* mayor complejidad;
* punto único de fallo.

Por eso:

```text
Central ≠ Router obligatorio de todas las operaciones
```

El sistema debe permitir:

```text
Node ↔ Node
Node ↔ Zone Controller
Zone Controller ↔ Central
Node ↔ Central
```

según las necesidades.

---

# 18. Comunicación directa

Cuando dos dispositivos necesiten comunicarse directamente, pueden hacerlo mediante el System Bus.

Por ejemplo:

```text
ESP32-Sensor
      │
      │ motion.detected
      ▼
ESP32-Luz
      │
      ▼
PWM 20 %
```

El Central puede registrar la automatización, configurarla y supervisarla, pero no necesita estar en medio de cada evento.

---

# 19. El System Bus debe ocultar el transporte

La aplicación no debería preocuparse por si los dispositivos utilizan:

```text
Ethernet
Wi-Fi
RS485
CAN
Thread
Zigbee
Matter
```

La aplicación trabaja con:

```text
EVENT
COMMAND
STATE
ALARM
DISCOVERY
CONFIGURATION
HEARTBEAT
DIAGNOSTIC
```

Por ejemplo:

```text
motion.detected
```

puede viajar mediante:

```text
Wi-Fi
Ethernet
CAN
RS485
Thread
```

sin modificar la automatización.

---

# 20. ¿Cómo decide el sistema dónde ejecutar una tarea?

Cada dispositivo debe publicar una descripción de sus capacidades.

Ejemplo:

```json
{
  "device": "zone-controller-01",
  "resources": {
    "cpu": 4,
    "ram": 8388608,
    "psram": 8388608,
    "storage": 16777216,
    "ethernet": true,
    "wifi": true,
    "camera": false
  },
  "services": [
    "automation",
    "gateway",
    "mqtt",
    "modbus",
    "can"
  ]
}
```

La plataforma puede utilizar estos datos para decidir.

---

# 21. Motor de asignación de tareas

Se recomienda crear un componente:

```text
Task Placement Manager
```

o:

```text
Execution Manager
```

Su función será determinar dónde ejecutar cada servicio.

Debe considerar:

### Capacidad

```text
CPU
RAM
PSRAM
Flash
Storage
```

### Conectividad

```text
Ethernet
Wi-Fi
RS485
CAN
Thread
Zigbee
```

### Hardware

```text
GPIO
ADC
PWM
I2C
SPI
UART
Camera
Display
```

### Estado

```text
Online
Offline
Busy
Maintenance
Error
```

### Prioridad

```text
Critical
High
Normal
Low
Background
```

### Ubicación

```text
Casa
Planta baja
Cocina
Habitación
Exterior
Invernadero
```

### Dependencias

Una tarea puede requerir estar cerca de:

```text
sensor
actuator
gateway
bus
display
camera
```

---

# 22. Ejemplo de decisión automática

El usuario configura:

```text
Analizar cámara
+ detectar movimiento
+ registrar evento
```

La plataforma encuentra:

```text
Node A
ESP32-WROOM
CPU disponible: 70 %
Cámara: NO

Node B
ESP32-S3
PSRAM: 8 MB
Cámara: SÍ
CPU disponible: 55 %
```

La plataforma asigna:

```text
Análisis de cámara
        ↓
Node B
```

El usuario nunca tuvo que seleccionar Node B manualmente.

---

# 23. Servicios que deben permanecer en el Central

Hay servicios que naturalmente pertenecen al Central.

Por ejemplo:

```text
Web Portal
User Management
Global Configuration
Device Registry
Discovery
System Database
Firmware Registry
Global Dashboard
Global Logs
Global API
System Backup
```

Pero incluso estos servicios deberían diseñarse modularmente.

---

# 24. Servicios que pueden distribuirse

Otros servicios pueden ejecutarse en diferentes dispositivos.

Ejemplos:

```text
Automation Engine
MQTT Broker
Modbus Gateway
CAN Gateway
History Collector
Notification Service
Rule Engine
Energy Aggregation
Environmental Processing
Camera Processing
```

La plataforma debe permitir decidir automáticamente o manualmente dónde se ejecutan.

---

# 25. Configuración automática para usuarios normales

La opción predeterminada será:

```text
Modo:
AUTOMÁTICO
```

El sistema decidirá la distribución.

El usuario avanzado podrá habilitar:

```text
Modo:
MANUAL / AVANZADO
```

para seleccionar:

```text
Ejecutar en:
- Local
- Zone Controller
- Central
- Automático
```

Pero esta opción no debe ser necesaria para el usuario común.

---

# 26. Niveles de configuración

Se recomienda ofrecer tres niveles.

## Nivel 1 — Usuario

Interfaz extremadamente sencilla:

```text
¿Qué quieres hacer?

[ Automatización ]
[ Escena ]
[ Horario ]
[ Sensor ]
[ Dispositivo ]
```

Sin información técnica.

---

## Nivel 2 — Instalador

Permite:

* asignar dispositivos;
* configurar zonas;
* seleccionar sensores;
* configurar buses;
* administrar módulos;
* realizar diagnósticos;
* configurar redes.

---

## Nivel 3 — Desarrollador

Permite:

* GPIO;
* buses;
* módulos;
* protocolos;
* logs;
* recursos;
* tareas;
* prioridades;
* memoria;
* CPU;
* debugging.

El usuario normal nunca necesita acceder a este nivel.

---

# 27. Portal de instalación inicial

Un dispositivo nuevo debería poder instalarse mediante un asistente.

### Paso 1

```text
Bienvenido

Vamos a configurar tu sistema.
```

### Paso 2

```text
Crear instalación

Nombre:
Casa Klein
```

### Paso 3

```text
Agregar Central

ESP32-S3
✓ Detectado
```

### Paso 4

```text
Red

Wi-Fi
Ethernet

✓ Conectado
```

### Paso 5

```text
Buscar dispositivos

Encontrados: 8
```

### Paso 6

```text
Organizar

Casa
├── Exterior
├── Living
├── Cocina
├── Habitación
└── Garaje
```

### Paso 7

```text
Configurar funciones

¿Para qué quieres utilizar este dispositivo?

[ Sensores ]
[ Iluminación ]
[ Climatización ]
[ Seguridad ]
[ Energía ]
[ Agua ]
[ Display ]
[ Gateway ]
```

### Paso 8

```text
Listo

Tu instalación está funcionando.
```

---

# 28. Configuración visual de automatizaciones

Las automatizaciones deberían poder crearse visualmente.

Ejemplo:

```text
┌──────────────┐
│ Movimiento   │
└──────┬───────┘
       ↓
┌──────────────┐
│ Modo = Noche │
└──────┬───────┘
       ↓
┌──────────────┐
│ Luz = 15 %   │
└──────┬───────┘
       ↓
┌──────────────┐
│ Esperar 60 s │
└──────┬───────┘
       ↓
┌──────────────┐
│ Luz = OFF    │
└──────────────┘
```

El sistema transforma este flujo en una representación ejecutable distribuida.

---

# 29. El usuario debe pensar en "qué", no en "dónde"

La diferencia fundamental será:

### Sistema tradicional

```text
¿En qué ESP32 ejecuto esto?
¿En qué GPIO está?
¿Quién tiene el sensor?
¿Dónde está el relay?
¿Tengo que modificar el firmware?
```

### Esta plataforma

```text
¿Qué quiero que ocurra?
```

El sistema se ocupa del resto.

---

# 30. Fallos y redistribución

La distribución de carga también debe utilizarse para tolerancia a fallos.

Ejemplo:

```text
Central
  │
  ├── Zone Controller A
  ├── Zone Controller B
  └── Zone Controller C
```

Si A falla:

```text
Zone Controller A
       X
       │
       ▼
Nodes locales
       │
       ├── continúan funciones locales
       └── ejecutan automatizaciones críticas
```

Si existe capacidad disponible en B:

```text
Central
  │
  ├── Zone A X
  └── Zone B
        │
        └── puede asumir determinadas funciones
```

La redistribución debe realizarse únicamente cuando sea segura.

No se debe mover automáticamente una tarea crítica si hacerlo puede provocar una condición insegura.

---

# 31. Principio de persistencia local

Cada Node debe conservar localmente:

* configuración necesaria para funcionar;
* automatizaciones críticas;
* límites de seguridad;
* estados importantes;
* parámetros de hardware;
* última configuración válida.

Por lo tanto:

```text
Central OFF
     ↓
Node continúa
```

---

# 32. Qué ocurre si Internet desaparece

No debe afectar las funciones locales.

```text
INTERNET
   X
   │
   ▼
CENTRAL
   │
   ├── Web local       ✓
   ├── Automatización  ✓
   ├── Sensores        ✓
   ├── Actuadores      ✓
   ├── Historial local ✓
   └── API local       ✓
```

Solamente deberían fallar funciones que realmente dependan de Internet:

```text
Cloud
Notificaciones externas
Servicios meteorológicos externos
Acceso remoto
Integraciones cloud
```

---

# 33. Qué ocurre si el Central falla

El sistema no debe detenerse.

```text
CENTRAL
   X

ZONE A
  ✓

ZONE B
  ✓

ZONE C
  ✓

NODE A
  ✓

NODE B
  ✓

NODE C
  ✓
```

Las funciones locales continúan.

Cuando el Central vuelve:

```text
Central vuelve
      │
      ▼
Discovery
      │
      ▼
Sincronización
      │
      ▼
Reconciliación
      │
      ▼
Sistema normal
```

---

# 34. No almacenar toda la información exclusivamente en el Central

Para reducir el impacto de un fallo, los Nodes pueden almacenar:

```text
Configuración crítica
Últimos estados
Automatizaciones locales
Eventos importantes
Contadores
Alarmas
```

El Central puede almacenar:

```text
Historial global
Configuración completa
Usuarios
Logs
Backups
Estadísticas
```

---

# 35. Distribución de almacenamiento

No toda la información tiene la misma importancia.

### Node

```text
Configuración crítica
Estados
Eventos recientes
```

### Zone Controller

```text
Estados de zona
Automatizaciones
Eventos de zona
Cache
```

### Central

```text
Configuración global
Historial
Usuarios
Logs
Backups
Inventario
```

### Cloud

```text
Solo información que el usuario decida enviar
```

---

# 36. Distribución de procesamiento

La misma filosofía se aplica al procesamiento.

```text
LOCAL
│
├── lectura de sensores
├── filtrado básico
├── control
├── seguridad
└── automatización crítica

ZONA
│
├── coordinación
├── reglas de zona
├── agregación
└── gateways

CENTRAL
│
├── administración
├── interfaz
├── configuración
├── análisis global
├── historial
└── automatización global

EXTERNO
│
├── servicios cloud
├── IA pesada
├── almacenamiento externo
└── integraciones
```

---

# 37. IA y procesamiento avanzado

El ESP32-S3 puede utilizarse para:

* reconocimiento simple;
* detección de presencia;
* procesamiento de imágenes;
* clasificación básica;
* análisis de sensores;
* predicciones pequeñas;
* detección de anomalías.

No se debe asumir que todo modelo de IA debe ejecutarse en el Central.

Por ejemplo:

```text
Cámara
   │
   ▼
ESP32-S3 local
   │
   └── detección de persona
             │
             ▼
        evento/person.detected
```

El sistema puede utilizar ese resultado sin transportar todo el vídeo hacia el Central.

---

# 38. IA pesada como servicio opcional

Para tareas que excedan la capacidad del ESP32:

```text
ESP32-S3
   │
   └── procesamiento preliminar
             │
             ▼
       Servidor opcional
             │
             ▼
        resultado
```

Esto permite que el sistema continúe funcionando incluso sin ese servidor, aunque determinadas funciones avanzadas puedan quedar deshabilitadas.

---

# 39. Central como plataforma extensible

El Central debe funcionar mediante servicios/módulos.

Por ejemplo:

```text
CENTRAL
│
├── Core
├── Web
├── API
├── Discovery
├── Automation
├── Users
├── Storage
├── History
├── MQTT
├── Matter
├── Home Assistant
├── Energy
├── Security
└── Diagnostics
```

Cada módulo debe poder:

```text
habilitar
deshabilitar
configurar
actualizar
diagnosticar
```

Esto evita tener un firmware monolítico.

---

# 40. Instalación pequeña

Una casa pequeña podría tener:

```text
1 Central ESP32-S3
2 Nodes ESP32
```

El Central puede hacer:

```text
Web
API
Discovery
Automations
History
```

Los Nodes:

```text
Sensores
Relays
Luces
```

No se necesitan Zone Controllers dedicados.

---

# 41. Instalación mediana

```text
1 Central
3 Zone Controllers
15 Nodes
```

El Central administra.

Los Zone Controllers coordinan:

```text
Planta baja
Planta alta
Exterior
```

---

# 42. Instalación grande

```text
                 CENTRAL
                    │
       ┌────────────┼────────────┐
       │            │            │
     Zona A       Zona B       Zona C
       │            │            │
    20 nodes      30 nodes      15 nodes
```

Incluso puede existir:

```text
Central A
Central B
```

para instalaciones que requieran mayor disponibilidad.

---

# 43. No todas las instalaciones necesitan la misma arquitectura

El sistema debe adaptarse automáticamente.

### Instalación pequeña

```text
Node ↔ Central
```

### Instalación mediana

```text
Node ↔ Zone Controller ↔ Central
```

### Instalación grande

```text
Node
  ↕
Zone Controller
  ↕
Central A/B
  ↕
Integraciones
```

El usuario no debería tener que rediseñar el software.

---

# 44. El sistema debe medir su propia capacidad

El Central debe mostrar un panel de salud:

```text
SISTEMA

Dispositivos:       47
Online:             46
Offline:             1

CPU Central:        38 %
RAM:                52 %
Storage:            41 %

Automatizaciones:   127
Eventos/min:        342

Estado:
✓ Normal
```

Y advertir:

```text
⚠ Zone Controller 02
CPU > 85 %

Recomendación:
Redistribuir servicios.
```

---

# 45. Distribución automática vs manual

Por defecto:

```text
Distribución:
AUTOMÁTICA
```

La plataforma decide.

Pero debe existir:

```text
Distribución:
AVANZADA
```

para instaladores y desarrolladores.

Ejemplo:

```text
Automatización:
Control bomba piscina

Ejecución:
[ Automático ▼ ]

Preferencia:
[ Local ]

Fallback:
[ Zone Controller ]

Prioridad:
[ Critical ]
```

---

# 46. Reglas para decidir dónde ejecutar

El sistema debe priorizar:

### 1. Seguridad

¿Es una función crítica?

Si sí:

```text
Local
```

---

### 2. Latencia

¿Necesita respuesta inmediata?

Si sí:

```text
Local / Zone
```

---

### 3. Dependencia física

¿Está directamente asociada a un hardware?

Si sí:

```text
Node correspondiente
```

---

### 4. Carga

¿El dispositivo está saturado?

Buscar alternativa compatible.

---

### 5. Conectividad

¿Existe un enlace disponible?

Si no:

```text
Ejecutar localmente
```

---

### 6. Recursos

¿Necesita PSRAM, cámara, almacenamiento, etc.?

Buscar dispositivo compatible.

---

### 7. Disponibilidad

¿El dispositivo está online?

Si no:

```text
Fallback
```

cuando sea seguro.

---

# 47. Prioridades de tareas

Cada tarea tendrá una prioridad lógica.

```text
CRITICAL
HIGH
NORMAL
LOW
BACKGROUND
```

Ejemplo:

```text
Corte por sobretemperatura
→ CRITICAL

Control de bomba
→ HIGH

Automatización de iluminación
→ NORMAL

Historial
→ LOW

Sincronización estadística
→ BACKGROUND
```

Una tarea `CRITICAL` nunca debe quedar esperando a una tarea `BACKGROUND`.

---

# 48. Protección contra sobrecarga

El sistema debe evitar asignar tareas cuando:

```text
CPU demasiado alta
RAM insuficiente
PSRAM insuficiente
almacenamiento lleno
bus saturado
conectividad inestable
```

Debe poder informar:

```text
No se puede instalar este servicio en ESP32-04.

Motivo:
PSRAM insuficiente.

Dispositivos compatibles:
ESP32-S3-02
ESP32-S3-03
```

---

# 49. Instalación de módulos sin tocar código

Los módulos deben ser instalables desde el portal cuando sea técnicamente posible.

Ejemplo:

```text
Módulos disponibles

✓ Energy Meter
✓ Modbus
✓ CAN
✓ Water Flow
✓ Weather
✓ Security
✓ Camera AI
✓ MQTT
✓ Matter
```

El usuario puede:

```text
[Instalar]
[Activar]
[Configurar]
[Desactivar]
```

El firmware base y el sistema de módulos deben diseñarse para permitir esta evolución.

---

# 50. Compatibilidad de hardware

El sistema debe detectar automáticamente:

```text
ESP32-WROOM
ESP32-WROOM-32E
ESP32-WROOM-32UE
ESP32-S3
ESP32-C6
ESP32-C5
```

y determinar sus capacidades.

Ejemplo:

```text
ESP32-S3

✓ PSRAM
✓ Cámara
✓ USB
✓ Wi-Fi
✓ GPIO
```

Mientras:

```text
ESP32-C6

✓ Wi-Fi 6
✓ Thread
✓ Zigbee
✓ Matter
```

El sistema utilizará estas capacidades para decidir dónde instalar determinados servicios.

---

# 51. Ejemplo completo

Supongamos una vivienda:

```text
Casa
│
├── Living
├── Cocina
├── Habitación
├── Baño
├── Garaje
└── Exterior
```

Hardware:

```text
Central
ESP32-S3

Living
ESP32-S3

Cocina
ESP32-WROOM

Habitación
ESP32

Garaje
ESP32-C6

Exterior
ESP32
```

El usuario configura:

```text
Movimiento en Living
→ encender luz

Temperatura Cocina > 28 °C
→ ventilador

Puerta Garaje abierta
→ aviso

Movimiento Exterior
→ seguridad
```

El sistema puede distribuir:

```text
Living
PIR → luz
LOCAL

Cocina
Temperatura → ventilador
LOCAL

Garaje
Puerta → seguridad
ZONE / LOCAL

Exterior
PIR → alarma
LOCAL
```

El Central solamente coordina y administra.

---

# 52. Ejemplo de modo noche

El usuario configura:

```text
Modo NOCHE
```

Y crea:

```text
Dormitorio:
PIR → registrar solamente

Pasillo:
PIR → luz 15 %

Exterior:
PIR → alarma

Puerta:
Abrir → alarma
```

El sistema no necesita que el Central procese cada evento.

Cada zona conoce el estado:

```text
HOUSE.MODE = NIGHT
```

y aplica sus reglas.

---

# 53. El Central distribuye "políticas", no necesariamente eventos

Esta distinción es importante.

En lugar de:

```text
Sensor
 ↓
Central
 ↓
Central decide
 ↓
Actuador
```

preferentemente:

```text
Central
 ↓
Distribuye configuración/reglas
 ↓
Node
 ↓
Ejecuta localmente
```

Por ejemplo:

```text
Central:
"Cuando detectes movimiento y MODE=NIGHT,
enciende tu salida al 15 % durante 60 s."
```

El Node puede ejecutar esa regla sin depender del Central.

---

# 54. Sincronización de configuración

El Central mantiene la configuración global.

Cada Node mantiene una copia de la configuración que necesita.

Se recomienda utilizar:

```text
configuration_version
```

Ejemplo:

```text
Central:
version = 105

Node:
version = 104
```

El Central detecta:

```text
Node desactualizado
```

y sincroniza.

---

# 55. Reconciliación

Cuando un dispositivo vuelve después de estar desconectado:

```text
Node OFFLINE
      ↓
Node vuelve
      ↓
Handshake
      ↓
Comparar versiones
      ↓
Comparar estados
      ↓
Sincronizar
      ↓
ONLINE
```

No se debe sobrescribir ciegamente una configuración válida.

Debe existir una política de reconciliación.

---

# 56. Actualizaciones OTA

Las actualizaciones también deben ser distribuidas cuidadosamente.

El Central puede actuar como administrador OTA:

```text
Nueva versión
      │
      ▼
Compatibilidad
      │
      ▼
Descargar
      │
      ▼
Validar
      │
      ▼
Actualizar
      │
      ▼
Verificar
```

No se debe actualizar toda una instalación crítica simultáneamente.

Se recomienda:

```text
1 dispositivo
      ↓
prueba
      ↓
verificación
      ↓
siguiente grupo
```

---

# 57. Seguridad

La distribución no debe permitir que cualquier dispositivo ejecute servicios arbitrarios.

Cada dispositivo debe autenticarse.

El Central debe verificar:

```text
Device ID
Firmware
Capabilities
Certificate / Key
Permissions
Compatibility
```

Y los servicios deben tener permisos.

Ejemplo:

```text
Servicio:
Security

Puede:
✓ Leer PIR
✓ Leer puertas
✓ Activar alarma

No puede:
✗ Modificar firmware
✗ Crear usuarios
✗ Cambiar credenciales
```

---

# 58. La experiencia del usuario es una capa independiente

Internamente el sistema puede ser extremadamente complejo.

La interfaz no debe reflejar esa complejidad.

Internamente:

```text
Device
Resource
Service
Task
Node
Zone
Bus
Protocol
Capability
Priority
Fallback
```

Externamente:

```text
Casa
Habitación
Luz
Temperatura
Seguridad
Automatización
Escena
```

Esta separación es fundamental para conseguir una plataforma profesional.

---

# 59. Principio "Zero Code"

El objetivo para el usuario final será:

> **Zero Code Configuration**

El usuario no debe necesitar:

```text
Arduino IDE
PlatformIO
Git
C++
Python
JSON manual
SSH
Terminal
```

para utilizar el sistema.

Todo lo necesario debe estar disponible desde:

```text
Portal Web
```

---

# 60. Configuración avanzada sin perder simplicidad

La plataforma sí debe permitir acceso técnico.

Por ejemplo:

```text
Usuario
   ↓
Configuración sencilla

Instalador
   ↓
Configuración avanzada

Desarrollador
   ↓
Diagnóstico técnico
```

Esto permite utilizar la misma plataforma para:

* hogares;
* oficinas;
* talleres;
* comercios;
* agricultura;
* industria ligera;
* embarcaciones;
* laboratorios;
* instalaciones técnicas.

---

# 61. Portal como "sistema operativo" de la plataforma

El portal web debe considerarse una parte fundamental del sistema.

No será simplemente una página de configuración.

Debe permitir:

```text
Dashboard
Dispositivos
Zonas
Sensores
Actuadores
Automatizaciones
Escenas
Rutinas
Seguridad
Energía
Agua
Climatización
Red
Usuarios
Módulos
Integraciones
Historial
Diagnóstico
OTA
Backups
```

---

# 62. Dashboard adaptativo

El dashboard debe construirse mediante módulos.

Ejemplo:

```text
┌─────────────────┐
│ Temperatura     │
│ 23.5 °C         │
└─────────────────┘

┌─────────────────┐
│ Humedad         │
│ 54 %            │
└─────────────────┘

┌─────────────────┐
│ Energía         │
│ 1.24 kW         │
└─────────────────┘

┌─────────────────┐
│ Seguridad       │
│ ARMADA          │
└─────────────────┘
```

Los bloques pueden:

```text
Agregar
Eliminar
Mover
Redimensionar
Configurar
Ocultar
```

Esto coincide con el principio de módulos instalables y habilitables/deshabilitables.

---

# 63. El sistema debe ocultar la complejidad

Un buen ejemplo de la filosofía:

### Lo que ve el usuario

```text
Agregar sensor

Tipo:
[ Temperatura ▼ ]

Ubicación:
[ Cocina ▼ ]

Nombre:
[ Temperatura cocina ]

Guardar
```

### Lo que sucede internamente

```text
Discovery
 ↓
Capability matching
 ↓
Driver selection
 ↓
Resource assignment
 ↓
Configuration generation
 ↓
Task placement
 ↓
Persistence
 ↓
Event registration
 ↓
UI generation
```

El usuario no necesita conocer nada de esto.

---

# 64. Administración de recursos

Cada dispositivo tendrá un inventario lógico:

```text
CPU
RAM
PSRAM
Flash
Storage
GPIO
ADC
PWM
I2C
SPI
UART
CAN
Ethernet
Wi-Fi
BLE
Thread
Zigbee
Camera
Display
```

El sistema utilizará este inventario para seleccionar la implementación adecuada.

---

# 65. No todo debe distribuirse automáticamente

La automatización de la distribución debe tener límites.

Una función crítica puede exigir:

```text
EXECUTION_POLICY = LOCAL_ONLY
```

Otra:

```text
EXECUTION_POLICY = ZONE
```

Otra:

```text
EXECUTION_POLICY = CENTRAL_ALLOWED
```

Y otra:

```text
EXECUTION_POLICY = ANY_COMPATIBLE
```

Esto permite mantener control sobre funciones sensibles.

---

# 66. Políticas de ejecución

Se propone:

```text
LOCAL_ONLY
```

Debe ejecutarse localmente.

```text
LOCAL_PREFERRED
```

Preferentemente local, pero admite fallback.

```text
ZONE_PREFERRED
```

Preferentemente en Zone Controller.

```text
CENTRAL
```

Debe ejecutarse en Central.

```text
DISTRIBUTED
```

Puede repartirse.

```text
EXTERNAL
```

Puede utilizar un servidor externo.

---

# 67. Ejemplo de política

Protección de temperatura:

```text
LOCAL_ONLY
```

Control de iluminación de una habitación:

```text
LOCAL_PREFERRED
```

Automatización de una planta:

```text
ZONE_PREFERRED
```

Dashboard:

```text
CENTRAL
```

Procesamiento de cámara:

```text
DISTRIBUTED
```

IA avanzada:

```text
EXTERNAL
```

---

# 68. Arquitectura final de distribución

La arquitectura conceptual queda:

```text
                         ┌──────────────────────┐
                         │       INTERNET       │
                         │      / CLOUD         │
                         └──────────┬───────────┘
                                    │
                           opcional │
                                    │
                    ┌───────────────▼───────────────┐
                    │       CENTRAL ESP32-S3        │
                    │                               │
                    │ Web / API / Config / Users   │
                    │ Discovery / History / Global │
                    │ Automation / Integrations    │
                    └───────────────┬───────────────┘
                                    │
                     ┌──────────────┼──────────────┐
                     │              │              │
                     ▼              ▼              ▼
                ┌─────────┐    ┌─────────┐    ┌─────────┐
                │  ZONE A │    │  ZONE B │    │  ZONE C │
                │ CTRL    │    │ CTRL    │    │ CTRL    │
                └────┬────┘    └────┬────┘    └────┬────┘
                     │              │              │
              ┌──────┼──────┐ ┌─────┼─────┐ ┌─────┼─────┐
              ▼      ▼      ▼ ▼     ▼     ▼ ▼     ▼     ▼
             NODE   NODE   NODE NODE NODE NODE NODE NODE NODE
              │      │      │    │     │     │    │     │
           Sensors Act.   I/O  Sensors Act.  etc.
```

---

# 69. Regla fundamental de resiliencia

El sistema seguirá este principio:

```text
Si puedo hacerlo localmente:
        hazlo localmente.

Si necesito coordinación:
        hazlo en la zona.

Si necesito visión global:
        usa el Central.

Si necesito un servicio externo:
        utiliza Internet/Cloud.

Pero nunca hagas que una función crítica dependa
de un nivel superior si puede evitarse.
```

---

# 70. Regla fundamental de UX

El sistema seguirá también esta regla:

```text
COMPLEJIDAD INTERNA
        ↓
   ┌───────────┐
   │ Plataforma│
   └───────────┘
        ↓
SIMPLICIDAD EXTERNA
        ↓
      Usuario
```

El usuario debe poder instalar y configurar una instalación completa sin conocer:

* FreeRTOS;
* tareas;
* núcleos;
* GPIO;
* buses;
* protocolos;
* MQTT;
* CAN;
* Modbus;
* memoria;
* PSRAM;
* arquitectura de red.

Los usuarios avanzados podrán acceder a estas opciones, pero no serán obligatorias.

---

# 71. Objetivo final

La plataforma debe conseguir que el usuario pueda pensar:

> "Quiero que cuando ocurra X, suceda Y."

y no:

> "Necesito programar el ESP32 número 7 para que lea GPIO 21 y envíe un MQTT al ESP32 número 3."

La primera frase representa el objetivo del producto.

La segunda representa un detalle interno de implementación.

---

# 72. Principio arquitectónico definitivo

La arquitectura queda resumida en:

```text
                    USUARIO
                       │
                       ▼
               PORTAL INTERACTIVO
                       │
                       ▼
                CONFIGURACIÓN
                       │
                       ▼
              MOTOR DE ORQUESTACIÓN
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
           LOCAL      ZONA     CENTRAL
             │         │         │
             └─────────┼─────────┘
                       ▼
                  SYSTEM BUS
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
           SENSOR   ACTUADOR   GATEWAY
```

La plataforma se encarga de transformar una configuración realizada por el usuario en una arquitectura distribuida de ejecución.

---

# 73. Resumen de la decisión

### No se hará

```text
ESP32-S3 Central = servidor que hace absolutamente todo
```

### Sí se hará

```text
ESP32-S3 Central = administrador + orquestador + portal
```

junto con:

```text
ESP32 Nodes = ejecución local
```

```text
Zone Controllers = coordinación regional
```

```text
System Bus = comunicación abstracta
```

```text
Portal Web = configuración sin código
```

```text
Execution Manager = distribución automática
```

```text
Local Persistence = autonomía
```

```text
Fallback = tolerancia a fallos
```

---

# 74. Principio maestro de la plataforma

> **El Central administra el sistema, pero no es dueño de la capacidad de funcionamiento del sistema.**

Y, desde el punto de vista del usuario:

> **El usuario configura la intención; la plataforma resuelve la implementación.**

Esto permite mantener simultáneamente:

* simplicidad de uso;
* escalabilidad;
* autonomía;
* tolerancia a fallos;
* bajo costo;
* mantenimiento sencillo;
* modularidad;
* compatibilidad con múltiples ESP32;
* incorporación de nuevos dispositivos;
* incorporación de nuevos protocolos;
* integración con sistemas externos;
* posibilidad de utilizar IA;
* crecimiento desde una vivienda pequeña hasta instalaciones grandes.

---

# 75. Relación con otros documentos

Este documento debe considerarse relacionado directamente con:

```text
docs/
├── ARCHITECTURE.md
├── SYSTEM-ARCHITECTURE.md
├── CENTRAL-ARCHITECTURE.md
├── DEVICE-MODEL.md
├── COMMUNICATION.md
├── MODULE-DEVELOPMENT.md
└── LOAD-DISTRIBUTION.md
```

### Dependencias conceptuales

```text
ARCHITECTURE
      │
      ├── SYSTEM-ARCHITECTURE
      │
      ├── CENTRAL-ARCHITECTURE
      │
      ├── LOAD-DISTRIBUTION
      │
      ├── DEVICE-MODEL
      │
      ├── COMMUNICATION
      │
      └── MODULE-DEVELOPMENT
```

Todos deben mantener la misma filosofía:

> **Distribución de responsabilidades, autonomía local, configuración sin código y complejidad técnica oculta al usuario final.**

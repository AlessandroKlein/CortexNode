# Module Development — Guía para Desarrollo de Módulos

> **Proyecto:** Plataforma de Automatización Distribuida
> **Versión:** 1.0.0
> **Objetivo:** Definir cómo crear nuevos módulos sin romper la arquitectura.

---

# 1. Filosofía

Un módulo debe poder agregarse al sistema sin modificar innecesariamente el núcleo.

La arquitectura debe favorecer:

```text
Plugin / Module Architecture
```

---

# 2. Tipos de módulos

Se contemplan:

```text
Sensor
Actuator
Protocol
Communication
Display
UI
Automation
Security
Energy
Gateway
Storage
Service
```

---

# 3. Ejemplo

Un nuevo sensor de CO₂ no debería requerir modificar:

```text
Alarm
Lighting
Automation
API
Web UI
```

Debe implementar la interfaz estándar:

```text
Sensor
```

---

# 4. Estructura

Ejemplo:

```text
modules/
└── co2_sensor/
    ├── co2_sensor.cpp
    ├── co2_sensor.h
    ├── module.json
    ├── README.md
    └── tests/
```

---

# 5. Manifest

Cada módulo debe declarar sus propiedades.

Ejemplo conceptual:

```json
{
  "id": "sensor.co2",
  "name": "CO2 Sensor",
  "version": "1.0.0",
  "category": "sensor",
  "protocol": "i2c"
}
```

---

# 6. Inicialización

Todos los módulos deben tener una fase de:

```text
construct
initialize
configure
start
stop
```

---

# 7. Ciclo de vida

```text
DISCOVERED
    ↓
INITIALIZING
    ↓
READY
    ↓
RUNNING
    ↓
STOPPING
    ↓
STOPPED
```

En caso de error:

```text
ERROR
```

---

# 8. Configuración

Los módulos deben exponer una configuración declarativa.

Ejemplo:

```json
{
  "address": 118,
  "sample_rate": 10,
  "enabled": true
}
```

La interfaz web puede generar automáticamente formularios a partir de esa definición.

---

# 9. GPIO

Un módulo nunca debe asumir un GPIO fijo salvo que el hardware lo requiera.

Debe utilizar:

```text
hardware abstraction layer
```

Ejemplo:

```text
Sensor
 ↓
GPIO abstraction
 ↓
ESP32 GPIO
```

---

# 10. I2C

Los módulos I2C deben poder configurar:

* dirección;
* bus;
* velocidad;
* interrupción cuando corresponda.

---

# 11. SPI

Igualmente:

* bus;
* CS;
* frecuencia;
* modo SPI.

---

# 12. UART / RS485

Debe existir una abstracción para:

* UART;
* RS485;
* baudrate;
* parity;
* stop bits;
* dirección.

---

# 13. Modbus

Los módulos Modbus deben declarar:

```text
slave_id
registers
function_codes
datatype
scaling
unit
```

---

# 14. CAN

Los módulos CAN deben declarar:

```text
CAN interface
bitrate
node ID
PDO
SDO
```

---

# 15. Sensor

Todo sensor debe proporcionar:

```text
read()
getValue()
getUnit()
getStatus()
```

Y preferentemente eventos:

```text
onChange()
onThreshold()
onError()
```

---

# 16. Actuador

Debe proporcionar:

```text
set()
get()
enable()
disable()
```

Ejemplo:

```text
setBrightness(50)
```

---

# 17. Eventos

Los módulos pueden publicar:

```text
event.publish()
```

Ejemplo:

```text
sensor.temperature.changed
```

---

# 18. Automatización

Un módulo nunca debe implementar directamente lógica específica de una casa.

Incorrecto:

```text
if temperature > 30:
    turn_on_fan()
```

Correcto:

```text
publish temperature
```

y dejar la automatización al motor de reglas.

Esto mantiene los módulos reutilizables.

---

# 19. Interfaz web

Los módulos pueden declarar componentes UI.

Ejemplo:

```text
configuration_schema
status_schema
dashboard_widget
```

Así un módulo nuevo puede incorporar automáticamente:

* configuración;
* estado;
* widget;
* documentación.

---

# 20. Pantallas

Los módulos pueden declarar widgets.

Ejemplo:

```json
{
  "widget": "temperature",
  "source": "sensor.temperature"
}
```

La pantalla decide dónde mostrarlo.

---

# 21. Compatibilidad

Cada módulo debe especificar:

```text
minimum firmware
maximum firmware
supported hardware
required libraries
required protocols
```

---

# 22. Tests

Todo módulo debería incluir pruebas.

Ejemplo:

```text
tests/
├── test_init
├── test_read
├── test_error
└── test_configuration
```

Cuando sea posible deben existir:

* pruebas unitarias;
* pruebas hardware;
* pruebas de integración.

---

# 23. Documentación

Todo módulo debe contener:

```text
README.md
```

con:

1. Descripción.
2. Hardware.
3. Alimentación.
4. Conexiones.
5. GPIO.
6. Protocolos.
7. Configuración.
8. API.
9. Eventos.
10. Limitaciones.
11. Seguridad.
12. Ejemplos.
13. Troubleshooting.

---

# 24. Ejemplo de módulo de sensor

```text
sensor.temperature.ds18b20
```

Puede utilizar:

```text
GPIO configurable
```

y publicar:

```text
temperature.value
temperature.status
```

La plataforma no necesita conocer que físicamente es un DS18B20.

---

# 25. Módulos intercambiables

Ejemplo:

```text
AHT10
AHT20
AHT30
DHT22
```

pueden proporcionar:

```text
temperature
humidity
```

Por lo tanto:

```text
Automation
```

trabaja con:

```text
temperature
humidity
```

y no con el modelo físico.

---

# 26. Módulos de energía

Ejemplo:

```text
energy.zmpt101b
energy.sct013
energy.modbus
```

Todos pueden exponer:

```text
voltage
current
power
energy
```

---

# 27. Módulos de agua

Ejemplo:

```text
water.flow.yf
water.flow.dn
water.flow.modbus
```

Todos pueden exponer:

```text
flow_rate
volume
```

---

# 28. Módulos de seguridad

Ejemplo:

```text
security.pir
security.door
security.window
security.smoke
security.water_leak
```

Todos pueden generar eventos.

---

# 29. Módulos de comunicación

Ejemplo:

```text
transport.wifi
transport.ethernet
transport.zigbee
transport.thread
transport.can
transport.rs485
```

---

# 30. Módulos gateway

Un gateway debe traducir:

```text
Protocol A
     ↓
Device Model
     ↓
Protocol B
```

No debe copiar la lógica de aplicación.

---

# 31. Versionado

Los módulos utilizarán Semantic Versioning:

```text
MAJOR.MINOR.PATCH
```

Ejemplo:

```text
2.1.3
```

---

# 32. Compatibilidad hacia atrás

Los cambios incompatibles deben aumentar:

```text
MAJOR
```

Las nuevas funcionalidades:

```text
MINOR
```

Correcciones:

```text
PATCH
```

---

# 33. Registro de módulos

La plataforma debe mantener un registro:

```text
Module Registry
```

con:

```text
module_id
version
status
dependencies
hardware
```

---

# 34. Habilitar / deshabilitar

Todo módulo debe poder ser:

```text
enabled
disabled
```

desde la configuración.

Esto es especialmente importante para usuarios no técnicos.

---

# 35. Fallos

Un módulo defectuoso no debería detener todo el sistema.

Ejemplo:

```text
Sensor CO2 falla

       ↓

CO2 module ERROR

       ↓

resto del sistema continúa funcionando
```

---

# 36. Recursos

Los módulos deben informar cuando consumen:

* RAM;
* Flash;
* CPU;
* timers;
* GPIO;
* buses.

Esto permitirá detectar conflictos.

---

# 37. Reglas para desarrolladores

Un módulo:

### DEBE

* ser independiente;
* documentarse;
* validar configuración;
* informar errores;
* respetar interfaces;
* evitar bloquear el sistema;
* utilizar abstracciones.

### NO DEBE

* modificar directamente módulos ajenos;
* bloquear el loop principal;
* asumir que existe Wi-Fi;
* asumir que existe servidor;
* asumir GPIO fijo;
* almacenar configuración crítica sin el sistema de configuración;
* ejecutar automatizaciones específicas de usuario.

---

# 38. Flujo para crear un módulo

```text
1. Definir necesidad
        ↓
2. Definir modelo lógico
        ↓
3. Definir interfaz
        ↓
4. Crear módulo
        ↓
5. Implementar driver
        ↓
6. Implementar configuración
        ↓
7. Implementar eventos
        ↓
8. Implementar UI
        ↓
9. Implementar tests
        ↓
10. Documentar
        ↓
11. Registrar módulo
        ↓
12. Integrar
```

---

# 39. Objetivo final

Crear un ecosistema donde agregar:

```text
nuevo sensor
nuevo actuador
nuevo protocolo
nuevo display
nuevo gateway
```

sea una extensión del sistema y no una modificación del sistema completo.

La plataforma debe crecer mediante módulos.

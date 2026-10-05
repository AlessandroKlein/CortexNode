# Device Model — Modelo Universal de Dispositivos

> **Proyecto:** Plataforma de Automatización Distribuida
> **Versión:** 1.0.0

---

# 1. Objetivo

Definir una representación universal de cualquier dispositivo conectado al sistema.

Un dispositivo puede contener:

* sensores;
* actuadores;
* entradas;
* salidas;
* interfaces;
* servicios;
* capacidades;
* protocolos.

El sistema no debe depender del modelo físico.

---

# 2. Identidad

Cada dispositivo debe tener un identificador único.

Ejemplo:

```text
device_id:
    8A7F-23B1-91D2
```

Además:

```text
name:
    Control Dormitorio 1

manufacturer:
    Plataforma

model:
    ESP32-S3-Display

firmware:
    1.4.0
```

---

# 3. Estructura conceptual

```text
DEVICE
│
├── Identity
├── Hardware
├── Connectivity
├── Capabilities
├── Services
├── Sensors
├── Actuators
├── Configuration
├── State
└── Diagnostics
```

---

# 4. Capabilities

Un dispositivo debe anunciar sus capacidades.

Ejemplo:

```json
{
  "capabilities": [
    "digital_input",
    "digital_output",
    "analog_input",
    "temperature",
    "humidity",
    "ethernet",
    "wifi",
    "mqtt"
  ]
}
```

Esto permite que el servidor sepa qué puede hacer el dispositivo sin conocer su firmware internamente.

---

# 5. Recursos

Cada elemento físico debe convertirse en un recurso lógico.

Ejemplo:

```text
Device
 ├── temperature_1
 ├── humidity_1
 ├── relay_1
 ├── relay_2
 └── button_1
```

---

# 6. Sensores

Un sensor debe definir:

```text
id
type
unit
value
precision
status
timestamp
```

Ejemplo:

```json
{
  "id": "temperature_1",
  "type": "temperature",
  "unit": "°C",
  "value": 23.6,
  "status": "online"
}
```

---

# 7. Actuadores

Los actuadores deben definir:

* estado;
* comandos;
* límites;
* feedback.

Ejemplo:

```text
curtain_1
    position: 75%
    target: 100%
```

---

# 8. Entradas digitales

Ejemplos:

* pulsadores;
* sensores magnéticos;
* PIR;
* finales de carrera;
* alarmas.

Deben soportar eventos:

```text
pressed
released
rising
falling
changed
```

---

# 9. Salidas digitales

Ejemplos:

* relés;
* contactores;
* LEDs;
* sirenas;
* válvulas.

---

# 10. Salidas PWM

Debe existir un modelo genérico:

```text
pwm_1
frequency
duty_cycle
enabled
```

Puede utilizarse para:

* iluminación;
* ventiladores;
* motores;
* actuadores DC.

---

# 11. Dimmers AC

El modelo lógico debe ser:

```text
dimmer_1

power
brightness
min
max
```

El hardware podrá utilizar:

* TRIAC;
* MOSFET;
* SSR;
* drivers especializados.

El software no debe depender de la tecnología física.

---

# 12. Energía

Debe existir un modelo estándar:

```text
voltage
current
power
energy
frequency
power_factor
```

Ejemplo:

```json
{
  "voltage": 231.2,
  "current": 1.82,
  "power": 401.7,
  "energy": 12.4,
  "frequency": 50.0
}
```

Puede integrarse con:

* ZMPT101B;
* SCT-013;
* medidores Modbus;
* sensores comerciales.

---

# 13. Agua

Modelo:

```text
flow_rate
total_volume
pressure
temperature
```

Para caudalímetros:

```text
pulse_count
pulse_frequency
flow_rate
```

Compatible conceptualmente con sensores:

* YF;
* DN;
* Hall effect;
* sensores industriales.

---

# 14. Ambiente

Debe reutilizar el modelo definido para SEMA.

Ejemplos:

```text
temperature
humidity
pressure
dew_point
air_quality
co2
light
uv
rain
wind_speed
wind_direction
```

Esto permite que SEMA se convierta en un conjunto de módulos dentro de la plataforma general.

---

# 15. Modbus

Un dispositivo Modbus debe exponerse como recursos normales.

El resto del sistema no necesita saber que el sensor utiliza Modbus.

```text
Modbus sensor
      ↓
Gateway
      ↓
Device Model
      ↓
API
```

---

# 16. CAN / CANopen

La misma filosofía se aplica a CAN.

```text
CAN device
    ↓
CAN driver
    ↓
CANopen layer
    ↓
Device abstraction
```

Esto permite integrar equipos comerciales sin contaminar el resto de la aplicación.

---

# 17. Pantallas

Una pantalla debe ser considerada un dispositivo.

Debe publicar:

```text
display
touch
resolution
orientation
widgets
```

Ejemplo:

```text
Display
 ├── temperature_widget
 ├── humidity_widget
 ├── light_widget
 ├── alarm_widget
 └── media_widget
```

---

# 18. Widgets

Los widgets deben tener identidad.

Ejemplo:

```text
widget_id
type
position
size
configuration
permissions
```

Esto permitirá construir interfaces dinámicas.

---

# 19. Zonas

Cada dispositivo puede estar asociado a una zona.

```json
{
  "device": "8A7F-23B1",
  "zone": "bedroom_1"
}
```

---

# 20. Relaciones

El sistema debe permitir relaciones lógicas.

Ejemplo:

```text
PIR dormitorio
       ↓
     Zona
       ↓
Modo dormir
       ↓
Regla
       ↓
Luz pasillo
```

---

# 21. Estados

Los recursos deben diferenciar:

```text
online
offline
unknown
error
disabled
maintenance
```

---

# 22. Eventos

Los dispositivos deben poder generar eventos.

Ejemplos:

```text
motion.detected
door.opened
temperature.changed
water.flow.started
power.overload
device.offline
alarm.triggered
```

---

# 23. Comandos

Los comandos deben ser genéricos.

Ejemplo:

```json
{
  "device": "light_1",
  "command": "set",
  "parameters": {
    "state": true,
    "brightness": 75
  }
}
```

---

# 24. Descubrimiento

Cuando aparece un nuevo dispositivo:

```text
DISCOVERY
    ↓
IDENTIFICATION
    ↓
CAPABILITIES
    ↓
CONFIGURATION
    ↓
REGISTRATION
    ↓
ONLINE
```

El usuario no debería tener que programarlo manualmente.

---

# 25. Compatibilidad futura

El modelo debe permitir incorporar nuevos tipos sin romper los existentes.

Por ejemplo:

```text
sensor.temperature
sensor.humidity
sensor.pressure
sensor.co2
sensor.pm25
sensor.energy
sensor.water
sensor.custom
```

Los tipos desconocidos pueden registrarse como:

```text
custom
```

---

# 26. Regla fundamental

El modelo lógico nunca debe depender directamente de:

* GPIO;
* I2C;
* SPI;
* UART;
* CAN;
* Modbus;
* Wi-Fi;
* Ethernet.

Esas son implementaciones físicas.

La aplicación debe trabajar con:

```text
Sensor
Actuator
Event
Command
State
Service
```

y no con GPIO directamente.

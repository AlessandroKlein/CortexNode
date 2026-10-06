# Communication — Sistema de Comunicaciones

> **Proyecto:** Plataforma de Automatización Distribuida
> **Versión:** 1.0.0

---

# 1. Objetivo

Definir cómo se comunican todos los dispositivos de la plataforma.

El sistema debe permitir utilizar diferentes medios físicos sin modificar la lógica de aplicación.

---

# 2. Arquitectura de comunicaciones

Se utilizará una arquitectura por capas.

```text
Aplicación
     ↓
Device Model
     ↓
Messaging
     ↓
Transport
     ↓
Physical Layer
```

---

# 3. Medios físicos

La plataforma debe poder utilizar:

### Ethernet

Para instalaciones grandes y críticas.

### Wi-Fi

Para instalaciones residenciales y nodos flexibles.

### RS485

Para instalaciones cableadas e industriales.

### CAN

Para aplicaciones industriales, automotrices y marinas.

### Zigbee

Para dispositivos inalámbricos de bajo consumo.

### Thread

Para redes IP mesh.

### Matter

Para interoperabilidad.

### Bluetooth LE

Para configuración y dispositivos específicos.

---

# 4. Comunicación interna

La comunicación interna debe utilizar mensajes estandarizados.

Ejemplo:

```text
EVENT
COMMAND
STATE
DISCOVERY
CONFIGURATION
HEARTBEAT
ALARM
DIAGNOSTIC
```

---

# 5. Eventos

Ejemplo:

```json
{
  "type": "event",
  "event": "motion.detected",
  "source": "pir_01",
  "zone": "living",
  "timestamp": 1780000000
}
```

---

# 6. Comandos

Ejemplo:

```json
{
  "type": "command",
  "target": "light_01",
  "command": "set_state",
  "value": true
}
```

---

# 7. Estado

Los dispositivos deben poder publicar periódicamente su estado.

Ejemplo:

```text
device.online
device.offline
light.state
temperature.value
alarm.state
```

---

# 8. MQTT

MQTT será uno de los protocolos principales para integración y sistemas distribuidos.

Ejemplo de topics:

```text
automation/{site}/{zone}/{device}/state
automation/{site}/{zone}/{device}/event
automation/{site}/{zone}/{device}/command
automation/{site}/{zone}/{device}/availability
```

---

# 9. MQTT Discovery

El sistema debe poder anunciar dispositivos automáticamente.

Esto facilitará integración con plataformas como Home Assistant.

---

# 10. REST API

REST se utilizará principalmente para:

* configuración;
* administración;
* integración;
* consulta de información;
* diagnóstico.

Ejemplo:

```text
GET /api/v1/devices
GET /api/v1/zones
GET /api/v1/events
POST /api/v1/commands
```

---

# 11. WebSocket

WebSocket se utilizará para:

* dashboards;
* pantallas;
* datos en tiempo real;
* eventos;
* alarmas.

Esto evita realizar polling continuo.

---

# 12. Comunicación local

Cuando varios dispositivos están en la misma red, pueden comunicarse directamente.

Ejemplo:

```text
PIR
 │
 ├──→ Luz
 ├──→ Pantalla
 └──→ Controlador
```

No es obligatorio pasar por el servidor central.

---

# 13. Mesh

Para redes inalámbricas se debe permitir una topología mesh.

Ejemplo:

```text
             Nodo A
            /      \
        Nodo B     Nodo C
          |          |
        Nodo D     Nodo E
```

Esto resulta especialmente útil para:

* alarmas;
* sensores;
* edificios;
* instalaciones grandes.

---

# 14. Red de alarma

La alarma debe aprovechar la comunicación distribuida.

Ejemplo:

```text
                    CENTRAL
                       │
             ┌─────────┴─────────┐
             │                   │
        Sector exterior      Sector interior
             │                   │
       ┌─────┼─────┐       ┌─────┼─────┐
       │     │     │       │     │     │
      PIR   Puerta Ventana PIR  PIR  PIR
```

Cada nodo puede continuar detectando eventos incluso si pierde comunicación temporalmente.

---

# 15. Modos de seguridad

Los dispositivos deben poder recibir perfiles.

Ejemplo:

```text
DISARMED
ARMED
NIGHT
AWAY
VACATION
MAINTENANCE
```

Cada perfil determina cómo se interpretan los eventos.

---

# 16. Comunicación crítica

Los mensajes críticos deben tener:

* prioridad;
* confirmación;
* timestamp;
* origen;
* destino;
* número de secuencia.

Ejemplo:

```text
ALARM
PRIORITY = CRITICAL
ACK_REQUIRED = true
```

---

# 17. Comunicación normal

Para datos periódicos:

```text
temperature
humidity
pressure
energy
flow
```

no es necesario utilizar la misma prioridad que una alarma.

Esto permite reducir tráfico.

---

# 18. Heartbeat

Cada nodo debe enviar periódicamente:

```text
heartbeat
```

El servidor puede detectar:

```text
ONLINE
OFFLINE
UNSTABLE
```

---

# 19. Store and Forward

Los nodos importantes pueden almacenar eventos cuando la red está caída.

Cuando vuelve la conexión:

```text
Offline
   ↓
Eventos almacenados
   ↓
Network restored
   ↓
Synchronize
```

---

# 20. Sincronización temporal

El sistema utilizará:

* NTP;
* timestamp Unix;
* zona horaria configurada.

Los dispositivos deberán conservar información suficiente para trabajar temporalmente sin NTP.

---

# 21. Descubrimiento

Debe existir un mecanismo de discovery.

El dispositivo debe anunciar:

```text
device_id
model
firmware
ip
capabilities
services
protocols
```

---

# 22. Gateways

Un gateway permite conectar redes diferentes.

Ejemplo:

```text
Zigbee
   ↓
ESP32-C6 Gateway
   ↓
Device Model
   ↓
MQTT / Ethernet
```

Otro:

```text
CANopen
   ↓
ESP32 Gateway
   ↓
MQTT
```

Otro:

```text
Modbus RTU
   ↓
ESP32
   ↓
REST / MQTT
```

---

# 23. CANopen

CANopen será considerado una capa especializada sobre CAN.

Debe permitir:

* PDO;
* SDO;
* heartbeat;
* node guarding;
* emergency messages;
* object dictionary.

La aplicación debe recibir recursos abstractos.

---

# 24. Seguridad de comunicaciones

Se deben considerar:

* TLS;
* autenticación;
* certificados;
* claves;
* tokens;
* ACL;
* aislamiento de dispositivos.

---

# 25. Redes grandes

Para instalaciones grandes:

```text
                 CORE
                  │
          ┌───────┴────────┐
          │                │
      Gateway A        Gateway B
          │                │
      ┌───┴───┐        ┌───┴───┐
      │       │        │       │
    Zona A  Zona B   Zona C  Zona D
```

Esto evita que todos los dispositivos deban comunicarse directamente con todos.

---

# 26. Principio de interoperabilidad

La aplicación nunca debe asumir:

```text
Wi-Fi = comunicación
```

Debe asumir:

```text
Transport
```

El mismo dispositivo podría pasar de:

```text
Wi-Fi
```

a:

```text
Ethernet
```

sin modificar la lógica de automatización.

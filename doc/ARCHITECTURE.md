# Architecture — Plataforma de Automatización Distribuida

> **Proyecto:** Plataforma modular de automatización basada en ESP32
> **Estado:** Diseño / Arquitectura
> **Versión:** 1.0.0
> **Plataforma:** PlatformIO + ESP-IDF / Arduino según módulo
> **Objetivo:** Automatización residencial, comercial e industrial ligera

---

## 1. Objetivo

El proyecto tiene como objetivo crear una plataforma de automatización distribuida, modular y escalable, basada principalmente en microcontroladores ESP32.

La plataforma debe permitir controlar y supervisar:

* Iluminación.
* Dimmers.
* Ventiladores.
* Motores.
* Cortinas.
* Persianas.
* Puertas.
* Cerraduras.
* Bombas.
* Válvulas.
* Sistemas de riego.
* Alarmas.
* Sensores ambientales.
* Sensores eléctricos.
* Sensores de agua.
* Sensores industriales.
* Equipos mediante Modbus.
* Equipos mediante CAN/CANopen.
* Dispositivos Zigbee.
* Dispositivos Matter/Thread.
* Pantallas táctiles.
* Sistemas multimedia.
* Automatizaciones y escenas.
* Rutinas complejas.

El sistema debe funcionar tanto en:

* viviendas;
* departamentos;
* oficinas;
* comercios;
* hoteles;
* edificios;
* talleres;
* pequeñas industrias;
* instalaciones agrícolas;
* embarcaciones;
* instalaciones industriales especializadas.

---

# 2. Concepto general

El sistema se basa en una arquitectura de **PLC distribuido**.

En lugar de tener un único controlador conectado físicamente a todos los sensores y actuadores:

```text
                   ┌───────────────────┐
                   │   SERVIDOR        │
                   │   CENTRAL         │
                   └─────────┬─────────┘
                             │
                     RED PRINCIPAL
                             │
       ┌─────────────────────┼─────────────────────┐
       │                     │                     │
 ┌─────▼─────┐         ┌─────▼─────┐         ┌─────▼─────┐
 │ Nodo      │         │ Nodo      │         │ Nodo      │
 │ Habitación│         │ Cocina    │         │ Exterior  │
 └─────┬─────┘         └─────┬─────┘         └─────┬─────┘
       │                     │                     │
 sensores/actuadores    sensores/actuadores    sensores/actuadores
```

Cada nodo posee inteligencia propia.

La caída del servidor central **no debe inutilizar la instalación**.

Por ejemplo:

```text
Sensor puerta
      │
      ▼
Nodo alarma
      │
      ├── Sirena
      ├── Luces
      ├── Notificación
      └── Cámara
```

Estas acciones pueden ejecutarse localmente aunque el servidor central esté desconectado.

---

# 3. Principios arquitectónicos

## 3.1 Distribución

Cada dispositivo debe poder realizar tareas localmente.

## 3.2 Modularidad

Hardware y software deben estar divididos en módulos.

## 3.3 Descubrimiento automático

Los dispositivos deben poder anunciar:

* identidad;
* modelo;
* versión;
* capacidades;
* entradas;
* salidas;
* sensores;
* protocolos;
* servicios disponibles.

## 3.4 Configuración sin recompilar

La configuración normal debe realizarse mediante interfaz web.

No se debe exigir modificar código fuente para:

* cambiar GPIO;
* modificar sensores;
* cambiar nombres;
* crear automatizaciones;
* configurar horarios;
* cambiar sectores;
* asignar funciones.

## 3.5 Funcionamiento autónomo

Los nodos críticos deben poder continuar funcionando sin servidor.

## 3.6 Seguridad

Todo acceso debe utilizar autenticación y autorización.

## 3.7 Escalabilidad

La arquitectura debe permitir pasar de:

```text
1 dispositivo
```

a:

```text
10 dispositivos
```

```text
100 dispositivos
```

o instalaciones mucho mayores.

---

# 4. Capas del sistema

La arquitectura se divide en capas.

```text
┌───────────────────────────────────────────┐
│              INTERFAZ USUARIO             │
├───────────────────────────────────────────┤
│       AUTOMATIZACIÓN / ESCENAS / REGLAS   │
├───────────────────────────────────────────┤
│              SERVICIOS DEL SISTEMA        │
├───────────────────────────────────────────┤
│            MODELO DE DISPOSITIVOS         │
├───────────────────────────────────────────┤
│             COMUNICACIÓN                   │
├───────────────────────────────────────────┤
│              HARDWARE                      │
└───────────────────────────────────────────┘
```

---

# 5. Hardware

La plataforma no debe depender de un único ESP32.

Se deben soportar diferentes familias según las necesidades.

## ESP32-WROOM / ESP32 clásico

Uso recomendado:

* nodos Ethernet;
* entradas/salidas;
* relés;
* sensores;
* Modbus;
* CAN;
* automatización básica.

Puede utilizar Ethernet mediante:

* LAN8720A;
* IP101;
* otros PHY compatibles.

---

## ESP32-C6

Uso recomendado:

* Matter;
* Thread;
* Zigbee;
* Wi-Fi 6;
* nodos de comunicación;
* gateways.

Puede funcionar como:

```text
ESP32-C6
   │
   ├── Wi-Fi
   ├── Thread
   ├── Zigbee
   └── Matter
```

También puede actuar como gateway entre tecnologías.

---

## ESP32-S3

Uso recomendado:

* interfaces gráficas;
* pantallas táctiles;
* cámaras;
* reconocimiento;
* procesamiento local;
* algoritmos de predicción;
* aprendizaje de hábitos;
* interfaces multimedia.

La memoria adicional puede utilizarse para:

* modelos pequeños;
* buffers;
* historiales;
* imágenes;
* interfaces gráficas;
* almacenamiento temporal.

---

# 6. Tipos de nodos

El sistema no debe definir un único tipo de dispositivo.

Debe existir una clasificación lógica.

## Sensor Node

Nodo especializado en sensores.

## Actuator Node

Nodo especializado en actuadores.

## IO Node

Entradas y salidas digitales/analógicas.

## Energy Node

Medición eléctrica.

## Environmental Node

Temperatura, humedad, presión, calidad de aire, etc.

## Security Node

Alarmas y seguridad.

## Display Node

Pantallas.

## Gateway Node

Conversión entre protocolos.

## Controller Node

Ejecuta automatizaciones.

## Central Server

Servidor principal.

---

# 7. Habitación / Zona / Sector

Una de las características fundamentales será el concepto de **zona lógica**.

Una zona puede representar:

* dormitorio;
* cocina;
* baño;
* living;
* garaje;
* jardín;
* oficina;
* depósito;
* barco;
* sector industrial.

Un dispositivo puede pertenecer a una o varias zonas lógicas.

Ejemplo:

```text
Casa
│
├── Planta baja
│   ├── Living
│   ├── Cocina
│   └── Garaje
│
├── Planta alta
│   ├── Dormitorio 1
│   ├── Dormitorio 2
│   └── Baño
│
└── Exterior
    ├── Jardín
    └── Entrada
```

---

# 8. Grupos

Los grupos permiten controlar varios dispositivos simultáneamente.

Ejemplo:

```text
Grupo: Luces Dormitorios

├── Luz dormitorio 1
├── Luz dormitorio 2
├── Luz pasillo
└── Luz baño
```

Una única orden puede controlar todo el grupo.

---

# 9. Sectores de seguridad

Los sectores son especialmente importantes para alarmas.

Ejemplo:

```text
ALARMA
│
├── Sector Exterior
│   ├── Sensor jardín
│   ├── Sensor portón
│   └── Sensor patio
│
├── Sector Planta Baja
│   ├── Puerta principal
│   ├── Ventana cocina
│   └── Movimiento living
│
└── Sector Dormitorios
    ├── Movimiento dormitorio 1
    ├── Movimiento dormitorio 2
    └── Ventanas
```

Cada sector puede tener estados independientes:

* desarmado;
* armado;
* armado parcial;
* armado nocturno;
* armado perimetral;
* modo vacaciones;
* modo mantenimiento.

---

# 10. Modo dormir

El modo dormir es un ejemplo de automatización distribuida.

Al activarlo:

```text
Modo Dormir
│
├── Dormitorios
│   ├── sensores activos
│   └── sensores de seguridad activos
│
├── Living
│   └── sensores de movimiento desactivados
│
├── Exterior
│   └── sensores activos
│
└── Iluminación
    └── apagado
```

No se debe simplemente "apagar sensores".

El sistema debe permitir modificar qué eventos genera cada sensor.

Ejemplo:

```text
Sensor movimiento dormitorio

Modo Normal:
    movimiento → encender luz

Modo Dormir:
    movimiento → registrar evento
    movimiento → NO encender luz

Modo Alarma:
    movimiento → disparar alarma
```

Esto permite mantener sensores físicamente activos pero cambiar su comportamiento lógico.

---

# 11. Motor de automatización

El sistema debe incluir un motor de reglas.

Una regla tendrá:

```text
TRIGGER
   ↓
CONDICIONES
   ↓
ACCIONES
```

Ejemplo:

```text
SI
    movimiento = true

Y
    hora > 22:00

Y
    modo = "Dormir"

ENTONCES
    encender luz pasillo
    intensidad = 15%
```

---

# 12. Escenas

Una escena es un conjunto de estados predefinidos.

Ejemplo:

## Escena "Salir de casa"

```text
Luces → OFF
Climatización → OFF
Persianas → cerrar
Alarma → ARMADA
Puerta → bloquear
```

## Escena "Llegar"

```text
Puerta → desbloquear
Luces entrada → 60%
Alarma → desarmar
```

---

# 13. Rutinas

Las rutinas permiten encadenar acciones.

Ejemplo:

```text
Botón "Buenas noches"

1. Apagar luces
2. Cerrar persianas
3. Apagar climatización
4. Activar alarma nocturna
5. Apagar equipos
6. Activar sensores perimetrales
```

Las rutinas pueden contener:

* acciones;
* retrasos;
* condiciones;
* repeticiones;
* ramas;
* temporizadores;
* eventos externos.

---

# 14. Ejecución distribuida

Las automatizaciones importantes deben poder ejecutarse en:

### Local

Dentro del dispositivo.

### Regional

Entre varios dispositivos de una zona.

### Central

Desde el servidor.

Ejemplo:

```text
Sensor puerta
     │
     ▼
Nodo local
     │
     ├── Sirena local
     └── Luz local
     
     ↓

Servidor
     │
     ├── Notificación
     ├── Cámara
     └── Registro
```

La automatización crítica debe priorizar ejecución local.

---

# 15. Interfaz web

La interfaz web debe ser completamente configurable.

Debe permitir:

* agregar dispositivos;
* eliminar dispositivos;
* descubrir nodos;
* configurar sensores;
* configurar GPIO;
* crear habitaciones;
* crear grupos;
* crear sectores;
* crear escenas;
* crear rutinas;
* crear reglas;
* visualizar sensores;
* actualizar firmware;
* gestionar usuarios;
* configurar permisos;
* revisar logs.

---

# 16. Interfaz táctil

Los nodos ESP32-S3 con pantalla deben poder utilizar la misma arquitectura.

La interfaz debe utilizar widgets modulares.

Ejemplo:

```text
┌────────────────────────────┐
│ Dormitorio                 │
├────────────────────────────┤
│ 🌡 23.4 °C                 │
│ 💧 51 %                    │
│                            │
│ 💡 Luz          [ ON ]     │
│                            │
│ 🪟 Cortina      [ 75% ]    │
│                            │
│ 🎵 Música                  │
│     ▶  ───────────         │
└────────────────────────────┘
```

Los bloques deben poder agregarse, eliminarse, reorganizarse y configurarse.

La configuración puede realizarse:

* desde la propia pantalla;
* desde la web;
* desde una aplicación futura.

---

# 17. API

Toda la información debe estar disponible mediante API.

Ejemplo:

```text
GET /api/v1/devices
GET /api/v1/devices/{id}
GET /api/v1/zones
GET /api/v1/sensors
GET /api/v1/events
GET /api/v1/automations
```

Para acciones:

```text
POST /api/v1/devices/{id}/commands
```

La API debe utilizar JSON.

---

# 18. Integraciones externas

Se debe diseñar una capa de integración.

Objetivos:

* Home Assistant;
* Homey Pro;
* Apple Home;
* Google Home;
* Samsung SmartThings;
* MQTT;
* Matter;
* REST;
* WebSocket;
* otros sistemas.

La integración debe estar separada del núcleo.

---

# 19. Persistencia

El sistema debe diferenciar:

### Configuración

Información que debe sobrevivir reinicios.

### Estado

Información actual.

### Historial

Datos históricos.

### Logs

Eventos del sistema.

Los nodos pequeños pueden utilizar:

* NVS;
* LittleFS.

El servidor central puede utilizar una base de datos.

---

# 20. Seguridad

Se deben contemplar:

* usuarios;
* roles;
* contraseñas;
* tokens;
* API keys;
* permisos;
* TLS cuando corresponda;
* actualización segura;
* control de acceso por dispositivo.

Roles posibles:

```text
Administrador
Instalador
Operador
Usuario
Invitado
```

---

# 21. Actualizaciones

Debe existir OTA.

El sistema debe soportar:

```text
Firmware
Configuración
Módulos
Interfaces
```

Las actualizaciones deben comprobar:

* versión;
* compatibilidad;
* integridad;
* arquitectura;
* rollback cuando sea posible.

---

# 22. Watchdog

Cada nodo debe implementar watchdog.

Se deben supervisar:

* tareas;
* comunicaciones;
* sensores críticos;
* memoria;
* alimentación cuando sea posible.

---

# 23. Diagnóstico

Cada dispositivo debe poder informar:

* uptime;
* memoria;
* CPU;
* temperatura interna cuando esté disponible;
* RSSI;
* estado de red;
* errores;
* reinicios;
* watchdog;
* sensores desconectados;
* firmware.

---

# 24. Principio fundamental

Ningún módulo debe asumir que existe un servidor.

El sistema debe funcionar:

```text
sin servidor
```

y mejorar sus capacidades cuando existe:

```text
servidor central
```

Esto permite crear instalaciones pequeñas y luego ampliarlas sin reemplazar los dispositivos.

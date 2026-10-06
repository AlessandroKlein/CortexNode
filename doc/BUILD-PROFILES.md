# BUILD-PROFILES.md

> **Sistema:** Plataforma de Automatización Distribuida
> **Tipo:** Convención de arquitectura y compilación
> **Estado:** Definición
> **Versión:** 1.0.0
> **Última actualización:** 2026-10-06

---

# 1. Objetivo

Este documento define la estrategia oficial del proyecto para soportar diferentes microcontroladores, placas, variantes de hardware y capacidades mediante **un único código fuente**, utilizando:

* PlatformIO
* `platformio.ini`
* `build_flags`
* Directivas de preprocesador C/C++
* `#if defined(...)`
* `#elif defined(...)`
* `#else`
* `#endif`

El objetivo es que las diferencias entre plataformas de hardware sean resueltas durante la compilación sin necesidad de mantener diferentes versiones del firmware.

La aplicación, el modelo de dispositivos, las entidades, automatizaciones, API, System Bus y demás componentes deben permanecer lo más independientes posible del hardware físico.

---

# 2. Principio fundamental

El proyecto debe seguir el siguiente principio:

> **Un único código fuente, múltiples perfiles de hardware.**

Por ejemplo:

```text
                    MISMO PROYECTO
                         │
             ┌───────────┴───────────┐
             │                       │
      ESP32-WROOM-32             ESP32-S3
             │                       │
        LAN8720A                 W5500
        Ethernet                 Ethernet
             │                       │
             └───────────┬───────────┘
                         │
                  MISMA APLICACIÓN
```

El código de aplicación no debería necesitar saber si el dispositivo utiliza:

* Ethernet nativo;
* W5500 por SPI;
* Wi-Fi;
* CAN;
* RS485;
* un expansor MCP23017;
* un 74HC595;
* PSRAM;
* cámara;
* pantalla;
* almacenamiento externo.

Estas diferencias pertenecen principalmente a las capas de hardware y drivers.

---

# 3. Problema que resuelve

Sin esta arquitectura, sería habitual terminar con proyectos independientes:

```text
firmware_esp32_wroom/
firmware_esp32_s3/
firmware_esp32_c6/
firmware_esp32_p4/
firmware_w5500/
firmware_lan8720/
```

Esto provoca:

* código duplicado;
* correcciones que deben hacerse varias veces;
* divergencia entre versiones;
* mayor cantidad de errores;
* dificultad para mantener compatibilidad;
* dificultad para agregar nuevas placas.

La arquitectura propuesta permite:

```text
src/
├── application/
├── core/
├── device/
├── hardware/
├── drivers/
├── services/
├── communication/
├── integrations/
└── ui/

                 ↓

       BUILD PROFILE

                 ↓

┌───────────────┬───────────────┬───────────────┐
│ ESP32 WROOM   │ ESP32 S3      │ ESP32 C6      │
│ LAN8720A      │ W5500         │ Wi-Fi/Thread  │
└───────────────┴───────────────┴───────────────┘
```

---

# 4. Alcance

Esta convención no se limita a Ethernet.

Debe utilizarse para cualquier diferencia de hardware que pueda conocerse durante la compilación.

Ejemplos:

* modelo de microcontrolador;
* placa;
* CPU;
* cantidad de núcleos;
* arquitectura;
* Ethernet;
* Wi-Fi;
* Bluetooth;
* Thread;
* Zigbee;
* Matter;
* CAN;
* RS485;
* UART;
* SPI;
* I²C;
* USB;
* GPIO;
* ADC;
* DAC;
* PWM;
* RMT;
* LEDC;
* MCPWM;
* PSRAM;
* Flash;
* microSD;
* cámara;
* pantalla;
* touch;
* expansores;
* sensores;
* actuadores;
* watchdog;
* almacenamiento;
* capacidades de procesamiento;
* aceleradores;
* periféricos específicos.

---

# 5. Jerarquía de configuración

La selección de hardware debe seguir esta jerarquía:

```text
platformio.ini
      │
      ▼
BUILD FLAGS
      │
      ▼
BUILD PROFILE
      │
      ▼
HARDWARE CONFIGURATION
      │
      ▼
DRIVERS
      │
      ▼
SERVICES
      │
      ▼
APPLICATION
```

La aplicación nunca debería depender directamente de:

```cpp
#if defined(BOARD_ESP32_WROOM)
```

salvo en situaciones excepcionales.

Preferentemente:

```text
BUILD PROFILE
      ↓
Hardware abstraction
      ↓
Driver
      ↓
Service
      ↓
Application
```

---

# 6. Build Profiles

Cada combinación de hardware debe representarse mediante un **Build Profile**.

Ejemplos:

```text
BOARD_ESP32_WROOM
BOARD_ESP32_WROVER
BOARD_ESP32_S3
BOARD_ESP32_C3
BOARD_ESP32_C5
BOARD_ESP32_C6
BOARD_ESP32_H2
BOARD_ESP32_P4
```

También pueden existir perfiles de placa:

```text
BOARD_WT32_ETH01
BOARD_ESP32_ETH_KIT
BOARD_CUSTOM_WROOM
BOARD_CUSTOM_S3
```

Y perfiles específicos de hardware:

```text
ETH_LAN8720
ETH_W5500

CAN_NATIVE
CAN_EXTERNAL

RS485_UART
RS485_EXTERNAL

STORAGE_NONE
STORAGE_SD
STORAGE_SDMMC

DISPLAY_NONE
DISPLAY_ST7789
DISPLAY_ILI9341
```

---

# 7. Separar MCU, placa y periféricos

No se debe asumir que:

```text
ESP32-WROOM = una única configuración de hardware
```

Por ejemplo, pueden existir:

```text
ESP32-WROOM + LAN8720A
ESP32-WROOM + W5500
ESP32-WROOM + CAN
ESP32-WROOM + RS485
```

Por lo tanto deben diferenciarse tres conceptos:

```text
MCU
 ↓
BOARD
 ↓
HARDWARE FEATURES
```

Ejemplo:

```text
ESP32-WROOM-32
       ↓
CUSTOM_ETH_BOARD
       ↓
LAN8720A
       ↓
Ethernet
```

Otro:

```text
ESP32-S3
       ↓
CUSTOM_CONTROLLER_S3
       ↓
W5500
       ↓
Ethernet
```

---

# 8. platformio.ini

El archivo `platformio.ini` debe utilizar entornos separados para cada Build Profile.

Ejemplo:

```ini
[platformio]
default_envs = esp32_wroom

[env]
platform = espressif32
framework = arduino

monitor_speed = 115200

lib_deps =
    bblanchon/ArduinoJson

[env:esp32_wroom]
board = esp32dev

build_flags =
    -D BOARD_ESP32_WROOM
    -D MCU_ESP32
    -D ETH_LAN8720

[env:esp32_s3]
board = esp32-s3-devkitc-1

build_flags =
    -D BOARD_ESP32_S3
    -D MCU_ESP32_S3
    -D ETH_W5500

[env:esp32_c6]
board = esp32-c6-devkitc-1

build_flags =
    -D BOARD_ESP32_C6
    -D MCU_ESP32_C6

[env:esp32_p4]
board = esp32-p4

build_flags =
    -D BOARD_ESP32_P4
    -D MCU_ESP32_P4
```

---

# 9. Regla importante: un perfil debe describir una configuración válida

No se deben agregar flags arbitrariamente.

Incorrecto:

```ini
build_flags =
    -D BOARD_ESP32_S3
    -D ETH_LAN8720
    -D ETH_W5500
```

Esto genera una configuración ambigua.

Correcto:

```ini
build_flags =
    -D BOARD_ESP32_S3
    -D ETH_W5500
```

O:

```ini
build_flags =
    -D BOARD_ESP32_WROOM
    -D ETH_LAN8720
```

Cada combinación debe representar una configuración física real y soportada.

---

# 10. Convención de nombres

Se utilizarán nombres en mayúsculas.

## 10.1 Microcontroladores

```text
MCU_ESP32
MCU_ESP32_S2
MCU_ESP32_S3
MCU_ESP32_C2
MCU_ESP32_C3
MCU_ESP32_C5
MCU_ESP32_C6
MCU_ESP32_H2
MCU_ESP32_P4
```

## 10.2 Placas

```text
BOARD_ESP32_WROOM
BOARD_ESP32_WROVER
BOARD_ESP32_S3
BOARD_WT32_ETH01
BOARD_CUSTOM_WROOM
BOARD_CUSTOM_S3
```

## 10.3 Interfaces

```text
ETH_LAN8720
ETH_W5500

CAN_NATIVE
CAN_EXTERNAL

RS485_UART
RS485_TRANSCEIVER

WIFI
BLE
THREAD
ZIGBEE
MATTER
```

## 10.4 Memoria

```text
HAS_PSRAM
HAS_EXTERNAL_FLASH
HAS_SD
HAS_SDMMC
```

## 10.5 Periféricos

```text
HAS_CAMERA
HAS_DISPLAY
HAS_TOUCH
HAS_RTC
HAS_ADC
HAS_DAC
```

---

# 11. Uso de `#if defined`

La selección de implementación debe realizarse mediante el preprocesador.

Ejemplo:

```cpp
#if defined(ETH_LAN8720)

    // Implementación LAN8720A

#elif defined(ETH_W5500)

    // Implementación W5500

#else

    // Sin Ethernet

#endif
```

Esto permite mantener un único archivo fuente.

---

# 12. Ejemplo Ethernet

## 12.1 Definición

En `platformio.ini`:

```ini
[env:esp32_wroom]
build_flags =
    -D BOARD_ESP32_WROOM
    -D ETH_LAN8720

[env:esp32_s3]
build_flags =
    -D BOARD_ESP32_S3
    -D ETH_W5500
```

## 12.2 Driver

```cpp
#if defined(ETH_LAN8720)

#include "EthernetLAN8720.hpp"

#elif defined(ETH_W5500)

#include "EthernetW5500.hpp"

#endif
```

La aplicación no debería realizar:

```cpp
#if defined(ETH_LAN8720)
```

La aplicación debería utilizar:

```cpp
EthernetManager.begin();
EthernetManager.isConnected();
EthernetManager.localIP();
```

La implementación concreta queda aislada.

---

# 13. Ethernet nativo vs W5500

## ESP32 con MAC Ethernet integrada

Familias que pueden disponer de MAC Ethernet integrada:

```text
ESP32-WROOM-32
ESP32-WROVER
ESP32-D0WDQ6
ESP32-D0WD
ESP32-S0WD
ESP32-P4
```

En estas plataformas puede utilizarse un PHY externo, por ejemplo:

```text
LAN8720A
IP101
IP101GR
LAN8710
```

Arquitectura:

```text
ESP32 MAC
    │
RMII
    │
LAN8720A / IP101 / etc.
    │
RJ45 + Magnetics
    │
Ethernet
```

---

# 14. Ethernet mediante SPI

Plataformas sin MAC Ethernet integrada pueden utilizar un controlador Ethernet externo mediante SPI.

Ejemplo:

```text
ESP32-S3
   │
 SPI
   │
 W5500
   │
 RJ45 + Magnetics
   │
 Ethernet
```

Configuración:

```ini
build_flags =
    -D BOARD_ESP32_S3
    -D ETH_W5500
```

El código común no debería necesitar conocer esta diferencia.

---

# 15. Regla general para cualquier periférico

La misma metodología debe utilizarse para todos los periféricos.

Ejemplo:

```cpp
#if defined(CAN_NATIVE)

    // CAN interno del MCU

#elif defined(CAN_EXTERNAL)

    // Controlador CAN externo

#endif
```

Otro:

```cpp
#if defined(RS485_UART)

    // UART + transceptor RS485

#elif defined(RS485_NATIVE_CONTROLLER)

    // Controlador específico

#endif
```

Otro:

```cpp
#if defined(DISPLAY_ST7789)

#elif defined(DISPLAY_ILI9341)

#elif defined(DISPLAY_ILI9488)

#endif
```

---

# 16. Sensores

Los sensores también deben poder seleccionarse mediante Build Profile cuando la selección sea una característica fija del hardware.

Ejemplo:

```ini
build_flags =
    -D SENSOR_TEMPERATURE_DS18B20
```

Código:

```cpp
#if defined(SENSOR_TEMPERATURE_DS18B20)

#include "DS18B20Driver.hpp"

#elif defined(SENSOR_TEMPERATURE_AHT20)

#include "AHT20Driver.hpp"

#elif defined(SENSOR_TEMPERATURE_BME280)

#include "BME280Driver.hpp"

#endif
```

Sin embargo, esto no debe impedir que el sistema permita configurar sensores dinámicamente desde la web.

---

# 17. Build Time vs Runtime

Es fundamental diferenciar:

## Build Time

Características determinadas durante la compilación:

```text
MCU
arquitectura
driver
librerías
periféricos físicos
PSRAM
Ethernet
CAN
tipo de display
tipo de controlador
```

Se resuelven mediante:

```text
platformio.ini
+
build_flags
+
#if defined(...)
```

## Runtime

Características configurables por el usuario:

```text
nombre del dispositivo
IP
Wi-Fi
zona
sensores habilitados
GPIO utilizado
entidad
nombre de entidad
automatizaciones
escenas
horarios
umbrales
integraciones
credenciales
```

Se almacenan mediante:

```text
NVS
Preferences
LittleFS
SPIFFS
SD
configuración central
```

---

# 18. Regla fundamental

No utilizar `build_flags` para una configuración que debería poder cambiarse desde la web.

Incorrecto:

```ini
-D TEMPERATURE_MIN=20
-D TEMPERATURE_MAX=25
```

Correcto:

```text
Configuración web
       ↓
Config Manager
       ↓
NVS
       ↓
Runtime
```

Los `build_flags` describen **qué hardware existe**.

La configuración almacenada describe **cómo quiere utilizarlo el usuario**.

---

# 19. Ejemplo combinado

Supongamos:

```text
ESP32-S3
W5500
PSRAM
Cámara
ST7789
GT911
MCP23017
```

El entorno podría ser:

```ini
[env:controller_s3]
board = esp32-s3-devkitc-1

build_flags =
    -D BOARD_ESP32_S3
    -D MCU_ESP32_S3

    -D ETH_W5500

    -D HAS_PSRAM
    -D HAS_CAMERA

    -D DISPLAY_ST7789
    -D TOUCH_GT911

    -D IO_EXPANDER_MCP23017
```

---

# 20. Ejemplo ESP32-WROOM

```ini
[env:controller_wroom]
board = esp32dev

build_flags =
    -D BOARD_ESP32_WROOM
    -D MCU_ESP32

    -D ETH_LAN8720

    -D CAN_NATIVE

    -D RS485_UART

    -D IO_EXPANDER_MCP23017
```

El mismo firmware puede compilarse para ambos perfiles.

---

# 21. Compatibilidad de características

No todas las combinaciones son válidas.

Por ejemplo:

```text
ESP32-WROOM
    └── CAN_NATIVE
```

puede ser válido.

Pero una característica que dependa de hardware inexistente no debe compilarse silenciosamente.

Se deben generar errores claros.

Ejemplo:

```cpp
#if defined(ETH_LAN8720) && !defined(MCU_ESP32)

#error "ETH_LAN8720 requiere un MCU con MAC Ethernet compatible"
#endif
```

---

# 22. Validación de perfiles

Los perfiles deben validarse durante la compilación.

Ejemplo:

```cpp
#if defined(ETH_LAN8720) && defined(ETH_W5500)

#error "No se puede seleccionar ETH_LAN8720 y ETH_W5500 simultáneamente"
#endif
```

Otro:

```cpp
#if defined(DISPLAY_ST7789) && defined(DISPLAY_ILI9341)

#error "Solo se puede seleccionar un controlador de display"
#endif
```

Otro:

```cpp
#if defined(TOUCH_GT911) && defined(TOUCH_XPT2046)

#error "Solo se puede seleccionar un controlador táctil"
#endif
```

---

# 23. Características opcionales

Cuando una característica simplemente puede estar presente o ausente, utilizar:

```text
HAS_*
```

Ejemplo:

```ini
-D HAS_PSRAM
-D HAS_CAMERA
-D HAS_SD
```

Código:

```cpp
#if defined(HAS_PSRAM)

    initializePSRAM();

#endif
```

---

# 24. Características exclusivas

Cuando solamente puede existir una implementación, utilizar una familia de defines:

```text
ETH_LAN8720
ETH_W5500
```

```text
DISPLAY_ST7789
DISPLAY_ILI9341
DISPLAY_ILI9488
```

```text
TOUCH_GT911
TOUCH_CST816S
TOUCH_XPT2046
```

---

# 25. Evitar macros excesivamente específicas

No crear macros como:

```text
ESP32_S3_W5500_ST7789_GT911_MCP23017_CAMERA
```

Esto genera una explosión combinatoria.

Incorrecto:

```text
BOARD_A
BOARD_B
BOARD_C
BOARD_D
...
```

si cada combinación representa una pequeña diferencia.

Preferir:

```text
BOARD_ESP32_S3

ETH_W5500
DISPLAY_ST7789
TOUCH_GT911
IO_EXPANDER_MCP23017
HAS_CAMERA
HAS_PSRAM
```

Así cada característica puede reutilizarse.

---

# 26. Hardware Capability Matrix

El proyecto debería mantener una matriz de compatibilidad.

Ejemplo:

| Característica |    WROOM | WROVER |       S2 | S3 |       C2 |       C3 |       C5 |       C6 | H2 |             P4 |
| -------------- | -------: | -----: | -------: | -: | -------: | -------: | -------: | -------: | -: | -------------: |
| Wi-Fi          |        ✓ |      ✓ |        ✓ |  ✓ |        ✓ |        ✓ |        ✓ |        ✓ |  — | según variante |
| Bluetooth      |        ✓ |      ✓ |        ✓ |  ✓ |        — |        ✓ |        ✓ |        ✓ |  ✓ | según variante |
| Ethernet MAC   |       ✓* |     ✓* |        — |  — |        — |        — |        — |        — |  — |              ✓ |
| SPI            |        ✓ |      ✓ |        ✓ |  ✓ |        ✓ |        ✓ |        ✓ |        ✓ |  ✓ |              ✓ |
| I²C            |        ✓ |      ✓ |        ✓ |  ✓ |        ✓ |        ✓ |        ✓ |        ✓ |  ✓ |              ✓ |
| UART           |        ✓ |      ✓ |        ✓ |  ✓ |        ✓ |        ✓ |        ✓ |        ✓ |  ✓ |              ✓ |
| PSRAM          | variante |      ✓ | variante |  ✓ |        — | variante | variante | variante |  — |              ✓ |
| W5500          |        ✓ |      ✓ |        ✓ |  ✓ |        ✓ |        ✓ |        ✓ |        ✓ |  ✓ |              ✓ |
| Cámara         |        ✓ |      ✓ |        ✓ |  ✓ | limitada | limitada | variante | variante |  — |              ✓ |

`*` La disponibilidad de Ethernet MAC debe comprobarse para la variante exacta del SoC/módulo y su diseño de placa.

Esta tabla es una referencia arquitectónica y no sustituye la validación de cada variante concreta.

---

# 27. ESP32-WROOM

Ejemplo de perfil:

```ini
[env:esp32_wroom]
board = esp32dev

build_flags =
    -D BOARD_ESP32_WROOM
    -D MCU_ESP32
    -D ETH_LAN8720
```

Implementación:

```cpp
#if defined(MCU_ESP32)

    // Funcionalidad específica del ESP32 clásico

#endif
```

Ethernet:

```cpp
#if defined(ETH_LAN8720)

    // MAC + RMII + PHY

#endif
```

---

# 28. ESP32-S3

Ejemplo:

```ini
[env:esp32_s3]
board = esp32-s3-devkitc-1

build_flags =
    -D BOARD_ESP32_S3
    -D MCU_ESP32_S3
    -D ETH_W5500
    -D HAS_PSRAM
```

Ethernet:

```cpp
#if defined(ETH_W5500)

    // SPI + W5500

#endif
```

---

# 29. ESP32-C6

Ejemplo:

```ini
[env:esp32_c6]
board = esp32-c6-devkitc-1

build_flags =
    -D BOARD_ESP32_C6
    -D MCU_ESP32_C6
```

Puede posteriormente incorporar:

```text
Wi-Fi
Thread
Zigbee
Matter
W5500
RS485
CAN mediante controlador externo
```

sin necesidad de crear un proyecto independiente.

---

# 30. ESP32-P4

El ESP32-P4 debe considerarse como una plataforma de alto rendimiento dentro del mismo ecosistema.

Ejemplo conceptual:

```ini
[env:esp32_p4]
board = esp32-p4

build_flags =
    -D BOARD_ESP32_P4
    -D MCU_ESP32_P4
    -D HAS_PSRAM
    -D HAS_CAMERA
```

Las capacidades concretas deben validarse contra el SoC y placa utilizados.

---

# 31. Abstracción de hardware

La estructura recomendada es:

```text
src/
│
├── application/
│
├── core/
│
├── device/
│
├── services/
│
├── communication/
│
├── integrations/
│
├── drivers/
│   ├── ethernet/
│   │   ├── EthernetLAN8720.cpp
│   │   ├── EthernetLAN8720.hpp
│   │   ├── EthernetW5500.cpp
│   │   └── EthernetW5500.hpp
│   │
│   ├── can/
│   ├── rs485/
│   ├── displays/
│   ├── touch/
│   ├── sensors/
│   └── expanders/
│
└── hardware/
    ├── HardwareProfile.cpp
    ├── HardwareProfile.hpp
    └── PinManager.cpp
```

---

# 32. Hardware Profile

Debe existir una abstracción capaz de informar qué capacidades tiene el dispositivo.

Ejemplo conceptual:

```cpp
struct HardwareCapabilities
{
    bool ethernet;
    bool wifi;
    bool bluetooth;
    bool can;
    bool rs485;
    bool psram;
    bool camera;
    bool display;
    bool touch;
};
```

El sistema puede utilizar esta información para:

* habilitar servicios;
* mostrar opciones en la web;
* evitar configuraciones inválidas;
* seleccionar drivers;
* realizar diagnóstico;
* generar información de dispositivo.

---

# 33. Compilación condicional vs configuración dinámica

La regla general es:

```text
¿Depende del silicio o hardware físico?
            │
           Sí
            ↓
     BUILD PROFILE
```

Ejemplo:

```text
¿Existe PSRAM?
¿Existe MAC Ethernet?
¿Usa W5500?
¿Qué controlador de display tiene?
¿Qué driver CAN utiliza?
```

Mientras que:

```text
¿Está habilitado?
¿En qué GPIO?
¿Qué nombre tiene?
¿A qué zona pertenece?
¿Con qué entidad está asociado?
```

debe resolverse mediante configuración runtime.

---

# 34. Ejemplo completo

Supongamos un dispositivo:

```text
ESP32-S3
W5500
PSRAM
MCP23017
ST7789
GT911
RS485
Cámara
```

`platformio.ini`:

```ini
[env:controller_s3]
board = esp32-s3-devkitc-1

build_flags =
    -D BOARD_ESP32_S3
    -D MCU_ESP32_S3

    -D ETH_W5500

    -D HAS_PSRAM
    -D HAS_CAMERA

    -D IO_EXPANDER_MCP23017

    -D DISPLAY_ST7789
    -D TOUCH_GT911

    -D RS485_UART
```

La aplicación puede seguir utilizando:

```cpp
network.begin();
display.begin();
touch.begin();
io.begin();
rs485.begin();
camera.begin();
```

sin saber qué hardware concreto hay debajo.

---

# 35. Librerías

Las librerías necesarias también pueden depender del Build Profile.

Ejemplo:

```ini
[env:esp32_wroom]
lib_deps =
    ...

build_flags =
    -D BOARD_ESP32_WROOM
    -D ETH_LAN8720
```

Mientras:

```ini
[env:esp32_s3]
lib_deps =
    ...

build_flags =
    -D BOARD_ESP32_S3
    -D ETH_W5500
```

Cuando una librería solamente es necesaria para una plataforma concreta, debe evitarse incluirla globalmente.

---

# 36. No duplicar lógica de aplicación

Incorrecto:

```cpp
#if defined(BOARD_ESP32_WROOM)

    ejecutarAutomatizacionWroom();

#elif defined(BOARD_ESP32_S3)

    ejecutarAutomatizacionS3();

#endif
```

Correcto:

```cpp
ejecutarAutomatizacion();
```

La automatización no debería saber qué microcontrolador la ejecuta.

La diferencia debe resolverse en:

```text
Hardware
↓
Driver
↓
Service
```

---

# 37. Ejemplo de sensor

Incorrecto:

```cpp
#if defined(BOARD_ESP32_WROOM)
    readDS18B20();

#elif defined(BOARD_ESP32_S3)
    readAHT20();

#endif
```

Correcto:

```cpp
temperatureSensor.read();
```

La implementación puede ser:

```text
TemperatureSensor
       │
       ├── DS18B20Driver
       ├── AHT20Driver
       └── BME280Driver
```

---

# 38. Ejemplo de expansores

La misma regla se aplica a:

```text
74HC595
74HC165
MCP23017
MCP23S17
PCF8574
TCA9555
```

Por ejemplo:

```cpp
#if defined(IO_EXPANDER_74HC595)

#elif defined(IO_EXPANDER_74HC165)

#elif defined(IO_EXPANDER_MCP23017)

#elif defined(IO_EXPANDER_MCP23S17)

#endif
```

La aplicación utiliza:

```cpp
io.setOutput(channel, true);
```

o:

```cpp
bool state = io.getInput(channel);
```

sin conocer el circuito físico.

---

# 39. GPIO

Los GPIO deben ser una propiedad del perfil de hardware o de la configuración de placa.

Ejemplo:

```cpp
#if defined(BOARD_ESP32_WROOM)

constexpr int STATUS_LED_PIN = 2;

#elif defined(BOARD_ESP32_S3)

constexpr int STATUS_LED_PIN = 48;

#endif
```

Sin embargo, cuando sea posible, debe utilizarse una tabla de configuración:

```cpp
struct PinDefinition
{
    int pin;
    const char* name;
};
```

De esta forma se reduce el uso de `#if` repartido por todo el proyecto.

---

# 40. Regla de concentración de `#if`

Los bloques:

```cpp
#if defined(...)
```

deben estar concentrados principalmente en:

```text
drivers/
hardware/
hal/
platform/
```

y no distribuidos por toda la aplicación.

Idealmente:

```text
application/
    ↓
services/
    ↓
HAL
    ↓
drivers/
    ↓
hardware
```

---

# 41. Hardware Abstraction Layer

Se recomienda una capa HAL:

```text
Application
      ↓
Service
      ↓
HAL
      ↓
Driver
      ↓
Hardware
```

Ejemplo:

```text
Application
      ↓
NetworkService
      ↓
NetworkHAL
      ↓
EthernetLAN8720 / EthernetW5500
      ↓
Hardware
```

Esto permite que la aplicación sea independiente del hardware.

---

# 42. Registro de capacidades

Durante el arranque, el dispositivo debe registrar sus capacidades.

Ejemplo conceptual:

```json
{
  "mcu": "ESP32-S3",
  "board": "controller_s3",
  "capabilities": {
    "wifi": true,
    "ethernet": true,
    "ethernet_driver": "w5500",
    "psram": true,
    "camera": true,
    "display": true,
    "touch": true,
    "rs485": true,
    "can": false
  }
}
```

Esto puede ser utilizado por:

* Central;
* API;
* web UI;
* diagnóstico;
* discovery;
* provisioning;
* actualización OTA;
* compatibilidad de módulos.

---

# 43. Compatibilidad de módulos

Los módulos del sistema pueden declarar requisitos.

Ejemplo:

```json
{
  "module": "camera_ai",
  "requirements": {
    "camera": true,
    "psram": true
  }
}
```

Otro:

```json
{
  "module": "ethernet",
  "requirements": {
    "ethernet": true
  }
}
```

El sistema puede comprobar automáticamente:

```text
Módulo
   ↓
Requisitos
   ↓
HardwareCapabilities
   ↓
Compatible / No compatible
```

---

# 44. Web UI

La interfaz web debe mostrar solamente las funciones compatibles con el hardware.

Ejemplo:

```text
Hardware
────────────────────────

MCU
ESP32-S3

PSRAM
✓ Disponible

Ethernet
✓ W5500

Cámara
✓ Disponible

CAN
✗ No disponible
```

Esto evita presentar configuraciones imposibles al usuario.

---

# 45. Configuración protegida

Las opciones de hardware deben estar protegidas.

Por ejemplo:

```text
Configuración
 ├── General
 ├── Red
 ├── Entidades
 ├── Automatizaciones
 │
 └── Hardware avanzado
```

`Hardware avanzado` puede requerir permisos de administrador/técnico.

---

# 46. Configuración física vs lógica

El sistema debe separar:

```text
HARDWARE
    ↓
GPIO / SPI / I²C / UART / CAN
    ↓
RESOURCE
    ↓
CAPABILITY
    ↓
ENTITY
    ↓
FUNCTION
    ↓
AUTOMATION
```

Por ejemplo:

```text
GPIO 17
   ↓
MCP23017
   ↓
Output
   ↓
Relay
   ↓
switch.pump
   ↓
Irrigation
   ↓
Automation
```

La automatización nunca debería depender directamente de GPIO 17.

---

# 47. OTA y Build Profiles

La actualización OTA debe comprobar que el firmware corresponde al hardware.

Un firmware para:

```text
ESP32-S3 + W5500
```

no debe instalarse accidentalmente en:

```text
ESP32-WROOM + LAN8720
```

El firmware debe incluir información como:

```json
{
  "firmware": "1.4.0",
  "mcu": "ESP32-S3",
  "board": "controller_s3",
  "profile": "ETH_W5500"
}
```

El sistema OTA debe validar compatibilidad antes de instalar.

---

# 48. Versionado del Build Profile

Los perfiles deben tener versión.

Ejemplo:

```cpp
#define HARDWARE_PROFILE_VERSION 3
```

O mediante:

```ini
-D HARDWARE_PROFILE_VERSION=3
```

Esto permite detectar cambios incompatibles en:

* pinout;
* periféricos;
* drivers;
* configuración;
* hardware soportado.

---

# 49. Errores de configuración

Los errores deben ser detectados lo antes posible.

Ejemplo:

```cpp
#if defined(ETH_LAN8720) && defined(ETH_W5500)

#error "Configuración Ethernet inválida: LAN8720 y W5500 no pueden estar activos simultáneamente."

#endif
```

Otro:

```cpp
#if defined(HAS_CAMERA) && !defined(HAS_PSRAM)

#warning "La cámara está habilitada sin PSRAM. Verifique que el hardware y la aplicación lo soporten."

#endif
```

Cuando una característica sea obligatoria:

```cpp
#error
```

Cuando sea una advertencia:

```cpp
#warning
```

---

# 50. Detección automática del MCU

El compilador puede proporcionar macros propias del SDK/toolchain.

Estas pueden utilizarse cuando sea necesario:

```cpp
#ifdef CONFIG_IDF_TARGET_ESP32
#endif

#ifdef CONFIG_IDF_TARGET_ESP32S3
#endif

#ifdef CONFIG_IDF_TARGET_ESP32C6
#endif
```

Sin embargo, el proyecto debe preferir sus propios Build Profiles:

```cpp
#if defined(MCU_ESP32_S3)
```

porque estos representan la arquitectura del proyecto y no dependen directamente de macros internas del framework.

Las macros nativas pueden utilizarse como validación.

Ejemplo:

```cpp
#if defined(MCU_ESP32_S3) && !defined(CONFIG_IDF_TARGET_ESP32S3)

#error "El Build Profile declara ESP32-S3 pero el target real no coincide."
#endif
```

---

# 51. No utilizar detección de hardware como arquitectura principal

No se recomienda:

```cpp
if (chipModel == ESP32_S3)
{
    ...
}
```

para decidir qué driver compilar.

La detección runtime puede utilizarse para diagnóstico:

```text
¿Qué hardware tengo?
```

pero la selección de implementación debe hacerse preferentemente durante compilación.

---

# 52. Ventajas de la compilación condicional

Esta arquitectura permite:

### Un único código

```text
src/
```

### Diferentes plataformas

```text
ESP32
ESP32-S3
ESP32-C6
ESP32-P4
...
```

### Diferentes interfaces

```text
LAN8720
W5500
Wi-Fi
CAN
RS485
...
```

### Diferentes periféricos

```text
MCP23017
MCP23S17
74HC595
74HC165
...
```

sin duplicar:

```text
application/
core/
services/
API/
System Bus/
Data Model/
automations/
```

---

# 53. Regla de oro

Debe cumplirse:

> **El hardware puede cambiar; el modelo lógico no debería cambiar.**

Por ejemplo:

```text
ESP32-WROOM + LAN8720
```

y:

```text
ESP32-S3 + W5500
```

deben poder proporcionar la misma entidad:

```text
binary_sensor.garage_ethernet
```

cuando corresponda.

La aplicación no debe preocuparse por cómo se implementa físicamente.

---

# 54. Arquitectura final

La arquitectura completa queda:

```text
                    PLATFORMIO
                        │
                        ▼
                 BUILD PROFILE
                        │
           ┌────────────┴────────────┐
           │                         │
        MCU/BOARD                FEATURES
           │                         │
           ├── ESP32                 ├── Ethernet
           ├── ESP32-S3              ├── CAN
           ├── ESP32-C6              ├── RS485
           ├── ESP32-C5              ├── PSRAM
           └── ESP32-P4              ├── Camera
                                     ├── Display
                                     └── Expanders
                        │
                        ▼
                HARDWARE / HAL
                        │
                        ▼
                     DRIVERS
                        │
                        ▼
                    SERVICES
                        │
                        ▼
                  DEVICE MODEL
                        │
                        ▼
                 SYSTEM BUS
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       ENTITIES      FUNCTIONS    AUTOMATIONS
          │             │             │
          └─────────────┼─────────────┘
                        ▼
                    API / UI
                        │
                        ▼
                  INTEGRATIONS
```

---

# 55. Ejemplo final completo

## platformio.ini

```ini
[platformio]
default_envs = controller_wroom

[env]
platform = espressif32
framework = arduino
monitor_speed = 115200

[env:controller_wroom]
board = esp32dev

build_flags =
    -D BOARD_ESP32_WROOM
    -D MCU_ESP32

    -D ETH_LAN8720

    -D CAN_NATIVE
    -D RS485_UART

    -D IO_EXPANDER_MCP23017


[env:controller_s3]
board = esp32-s3-devkitc-1

build_flags =
    -D BOARD_ESP32_S3
    -D MCU_ESP32_S3

    -D ETH_W5500

    -D HAS_PSRAM
    -D HAS_CAMERA

    -D RS485_UART

    -D IO_EXPANDER_MCP23S17

    -D DISPLAY_ST7789
    -D TOUCH_GT911


[env:controller_c6]
board = esp32-c6-devkitc-1

build_flags =
    -D BOARD_ESP32_C6
    -D MCU_ESP32_C6

    -D HAS_PSRAM

    -D WIRELESS_THREAD
    -D WIRELESS_ZIGBEE
```

---

# 56. Ejemplo de selección de drivers

```cpp
// EthernetManager.cpp

#if defined(ETH_LAN8720)

#include "EthernetLAN8720.hpp"

#elif defined(ETH_W5500)

#include "EthernetW5500.hpp"

#else

#include "EthernetNone.hpp"

#endif
```

```cpp
// DisplayManager.cpp

#if defined(DISPLAY_ST7789)

#include "ST7789Driver.hpp"

#elif defined(DISPLAY_ILI9341)

#include "ILI9341Driver.hpp"

#elif defined(DISPLAY_ILI9488)

#include "ILI9488Driver.hpp"

#else

#include "DisplayNone.hpp"

#endif
```

```cpp
// IOExpanderManager.cpp

#if defined(IO_EXPANDER_74HC595)

#include "74HC595Driver.hpp"

#elif defined(IO_EXPANDER_74HC165)

#include "74HC165Driver.hpp"

#elif defined(IO_EXPANDER_MCP23017)

#include "MCP23017Driver.hpp"

#elif defined(IO_EXPANDER_MCP23S17)

#include "MCP23S17Driver.hpp"

#endif
```

---

# 57. Ejemplo de código de aplicación

La aplicación no debería tener:

```cpp
#if defined(BOARD_ESP32_WROOM)
```

ni:

```cpp
#if defined(BOARD_ESP32_S3)
```

ni:

```cpp
#if defined(ETH_W5500)
```

En su lugar:

```cpp
void AutomationService::process()
{
    if (garageDoor.isOpen())
    {
        lightGarage.turnOn();
    }
}
```

La aplicación utiliza entidades y servicios.

Los servicios se encargan del hardware.

---

# 58. Regla de separación

## Build Profile

Define:

> **Qué hardware tengo.**

## Runtime Configuration

Define:

> **Cómo quiero utilizarlo.**

## Device Model

Define:

> **Qué representa el hardware dentro del sistema.**

## System Bus

Define:

> **Cómo se comunican los componentes.**

## Application

Define:

> **Qué debe hacer el sistema.**

Esta separación es obligatoria para mantener la escalabilidad.

---

# 59. Nuevas plataformas

Agregar una nueva plataforma no debe requerir modificar toda la aplicación.

Por ejemplo, para agregar:

```text
ESP32-C5
```

debería bastar con:

```ini
[env:controller_c5]
board = ...
build_flags =
    -D BOARD_ESP32_C5
    -D MCU_ESP32_C5
```

y posteriormente agregar únicamente los drivers específicos que sean necesarios.

---

# 60. Nuevos drivers

Agregar un nuevo hardware tampoco debe requerir modificar la lógica de aplicación.

Ejemplo:

```text
Nuevo Ethernet PHY
```

se incorpora:

```text
drivers/ethernet/
    EthernetNuevoPHY.cpp
    EthernetNuevoPHY.hpp
```

y:

```ini
-D ETH_NUEVO_PHY
```

La interfaz:

```cpp
network.begin();
network.isConnected();
network.localIP();
```

permanece igual.

---

# 61. Compatibilidad hacia el futuro

Esta arquitectura debe considerarse extensible a:

```text
ESP32
ESP32-S2
ESP32-S3
ESP32-C2
ESP32-C3
ESP32-C5
ESP32-C6
ESP32-H2
ESP32-P4
```

y a futuras variantes.

También permite incorporar posteriormente:

```text
LAN8720
IP101
IP101GR
W5500

CAN
CAN-FD mediante controlador externo

RS485
Modbus RTU

I²C
SPI

74HC595
74HC165
MCP23017
MCP23S17
PCF8574
TCA9555

ST7789
ILI9341
ILI9488
GT911
CST816S

Cámara
PSRAM
microSD
RTC
```

sin modificar la arquitectura superior.

---

# 62. Relación con la configuración modular

Los módulos del proyecto deben declarar sus dependencias.

Ejemplo:

```text
Módulo:
Camera AI

Necesita:
    HAS_CAMERA
    HAS_PSRAM
```

Otro:

```text
Módulo:
Ethernet

Necesita:
    ETH_LAN8720
    OR
    ETH_W5500
```

Otro:

```text
Módulo:
Touchscreen

Necesita:
    DISPLAY_*
    TOUCH_*
```

El sistema debe poder determinar:

```text
Hardware
    ↓
Capabilities
    ↓
Compatible modules
```

---

# 63. Principio para desarrolladores

Antes de agregar:

```cpp
#if defined(...)
```

debe hacerse la siguiente pregunta:

> **¿Esta diferencia pertenece realmente al hardware o estoy utilizando una macro para ocultar un problema de arquitectura?**

Si pertenece al hardware:

```text
HAL / Driver
```

Si pertenece a configuración:

```text
Runtime Configuration
```

Si pertenece a lógica:

```text
Application / Service
```

---

# 64. Anti-patrones

No hacer:

```cpp
#if defined(BOARD_ESP32_WROOM)
    ...
#elif defined(BOARD_ESP32_S3)
    ...
#elif defined(BOARD_ESP32_C6)
    ...
#endif
```

en cientos de archivos.

No hacer:

```text
src_wroom/
src_s3/
src_c6/
```

No duplicar:

```text
API
Device Model
System Bus
Automations
Scenes
Services
```

por plataforma.

No utilizar GPIO directamente desde las automatizaciones.

No hacer depender el funcionamiento crítico del Central.

No utilizar MQTT como dependencia obligatoria del hardware.

No utilizar Internet como dependencia de las funciones locales.

---

# 65. Regla para funciones críticas

Las funciones críticas deben continuar funcionando independientemente del Build Profile.

Por ejemplo:

```text
sensor
 ↓
entity
 ↓
automation
 ↓
actuator
```

debe poder funcionar:

```text
ESP32-WROOM
ESP32-S3
ESP32-C6
```

siempre que los recursos necesarios existan.

La plataforma concreta debe ser transparente para la automatización.

---

# 66. Principio de diseño definitivo

El proyecto debe seguir esta filosofía:

```text
             HARDWARE
                 │
                 ▼
          BUILD PROFILE
                 │
                 ▼
               HAL
                 │
                 ▼
             DRIVERS
                 │
                 ▼
             SERVICES
                 │
                 ▼
          DEVICE MODEL
                 │
                 ▼
           SYSTEM BUS
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
     ENTITY   FUNCTION  EVENT
        │        │        │
        └────────┼────────┘
                 ▼
            AUTOMATION
                 │
                 ▼
                API
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
       WEB      MQTT     MATTER
```

La aplicación debe mantenerse independiente del hardware siempre que sea técnicamente posible.

---

# 67. Resumen

La plataforma utilizará **Build Profiles definidos mediante `platformio.ini` y `build_flags`** para seleccionar las características físicas y de compilación.

Ejemplo:

```ini
[env:esp32_wroom]

build_flags =
    -D BOARD_ESP32_WROOM
    -D MCU_ESP32
    -D ETH_LAN8720
```

y:

```ini
[env:esp32_s3]

build_flags =
    -D BOARD_ESP32_S3
    -D MCU_ESP32_S3
    -D ETH_W5500
```

Los drivers utilizarán:

```cpp
#if defined(ETH_LAN8720)

#elif defined(ETH_W5500)

#endif
```

Pero la aplicación utilizará una abstracción común:

```cpp
network.begin();
```

La misma filosofía se aplicará a:

```text
Ethernet
Wi-Fi
CAN
RS485
SPI
I²C
GPIO
PWM
ADC
DAC
USB
PSRAM
Flash
SD
Cámara
Display
Touch
Sensores
Actuadores
Expansores
Watchdog
RTC
Comunicación
Integraciones
```

El resultado debe ser:

> **Un solo proyecto, un único modelo lógico y una única aplicación, capaz de compilarse para múltiples plataformas y configuraciones de hardware mediante Build Profiles.**

---

# 68. Regla de oro del proyecto

> **El firmware se adapta al hardware durante la compilación; el usuario configura el comportamiento durante la ejecución.**

Y, como principio arquitectónico complementario:

> **Las diferencias físicas deben quedar aisladas en Hardware, HAL y Drivers. El Device Model, System Bus, Services, Automations, API e Integrations deben permanecer independientes del microcontrolador siempre que sea posible.**

---

# 69. Checklist para agregar una nueva plataforma

Antes de considerar soportada una nueva plataforma:

* [ ] Crear Build Profile en `platformio.ini`.
* [ ] Definir `BOARD_*`.
* [ ] Definir `MCU_*`.
* [ ] Identificar periféricos disponibles.
* [ ] Definir `HAS_*` cuando corresponda.
* [ ] Definir drivers específicos.
* [ ] Validar incompatibilidades mediante `#error`.
* [ ] Implementar HAL cuando sea necesario.
* [ ] Evitar modificar la aplicación.
* [ ] Registrar `HardwareCapabilities`.
* [ ] Validar API.
* [ ] Validar Web UI.
* [ ] Validar configuración runtime.
* [ ] Validar OTA.
* [ ] Validar watchdog.
* [ ] Validar comunicación con Central.
* [ ] Validar funcionamiento sin Central.
* [ ] Validar funcionamiento sin Internet.
* [ ] Documentar pinout.
* [ ] Documentar limitaciones.
* [ ] Crear pruebas de hardware.
* [ ] Actualizar matriz de compatibilidad.

---

# 70. Checklist para agregar un nuevo periférico

* [ ] Crear macro `FEATURE_*`, `HAS_*` o `DRIVER_*`.
* [ ] Crear driver.
* [ ] Crear HAL si corresponde.
* [ ] No modificar la lógica de aplicación innecesariamente.
* [ ] Implementar detección/diagnóstico.
* [ ] Registrar capability.
* [ ] Añadir configuración Web si corresponde.
* [ ] Añadir validación.
* [ ] Añadir documentación.
* [ ] Añadir pruebas.
* [ ] Verificar compatibilidad con Central.
* [ ] Verificar API.
* [ ] Verificar comportamiento offline.

---

# 71. Conclusión

Esta metodología permite que la plataforma evolucione desde unos pocos dispositivos ESP32 hasta una familia completa de controladores distribuidos sin convertir el código en una colección de variantes específicas.

La diferencia entre:

```text
ESP32-WROOM + LAN8720A
```

y:

```text
ESP32-S3 + W5500
```

debe quedar principalmente en:

```text
Build Profile
      ↓
HAL
      ↓
Driver
```

y no en:

```text
Application
Device Model
System Bus
Automations
API
Integrations
```

De esta forma, agregar hardware nuevo no implica crear un nuevo proyecto: **se agrega un nuevo perfil y, cuando sea necesario, nuevos drivers detrás de las mismas interfaces.**

> **Hardware diferente no significa firmware diferente.**
>
> **Significa un Build Profile diferente sobre la misma arquitectura.**

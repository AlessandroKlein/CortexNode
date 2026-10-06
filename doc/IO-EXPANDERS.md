# IO Expanders — Expansores de entradas y salidas

> **Tipo:** Arquitectura / Hardware / Convención
> **Estado:** Definición
> **Versión:** 1.0.0
> **Fecha:** 2026-10-06
> **Proyecto:** Plataforma de Automatización Distribuida
> **Familias principales:** 74HC595 / 74HC165 / MCP23017 / MCP23S17

---

# 1. Propósito

Este documento define cómo la plataforma debe utilizar y abstraer los **expansores de entradas y salidas digitales**.

El objetivo es permitir que un dispositivo ESP32 pueda aumentar considerablemente la cantidad de entradas y salidas disponibles sin quedar limitado por la cantidad de GPIO físicos del microcontrolador.

La plataforma debe soportar principalmente:

```text
74HC595
74HC165
MCP23017
MCP23S17
```

y debe estar preparada para incorporar posteriormente otros dispositivos equivalentes.

---

# 2. Motivación

Los microcontroladores ESP32 disponen de una cantidad limitada de GPIO utilizables.

En una aplicación compleja pueden ser necesarios simultáneamente:

```text
Relés
Pulsadores
Interruptores
LED
Sensores digitales
Finales de carrera
Motores
Válvulas
Indicadores
Displays
Teclados
Entradas de alarma
Salidas auxiliares
```

Por lo tanto, el hardware debe poder ampliar las E/S sin consumir un GPIO del ESP32 por cada señal.

---

# 3. Familias soportadas

La plataforma distingue dos arquitecturas principales:

```text
74HC595 / 74HC165
        │
        └── Shift Registers

MCP23017 / MCP23S17
        │
        └── GPIO Expanders
```

No deben tratarse como dispositivos equivalentes internamente aunque ambos permitan ampliar GPIO.

---

# 4. Resumen

| Dispositivo | Función  | Interfaz |       GPIO | Cascada de datos | Selección      |
| ----------- | -------- | -------- | ---------: | ---------------- | -------------- |
| 74HC595     | Salidas  | Serial   |  8 salidas | Sí               | No             |
| 74HC165     | Entradas | Serial   | 8 entradas | Sí               | No             |
| MCP23017    | E/S      | I²C      |         16 | No directa       | Dirección I²C  |
| MCP23S17    | E/S      | SPI      |         16 | No directa       | CS + dirección |

La diferencia arquitectónica fundamental es:

```text
74HC595 / 74HC165
→ cadena de registros

MCP23017
→ bus I²C con direccionamiento

MCP23S17
→ bus SPI compartido con Chip Select
```

---

# 5. Concepto de cascada

Una cadena de shift registers permite conectar varios dispositivos en serie.

Por ejemplo:

```text
ESP32
  │
  ├── DATA
  ├── CLOCK
  └── LATCH
       │
       ▼
   74HC595 #1
       │
       ▼
   74HC595 #2
       │
       ▼
   74HC595 #3
       │
       ▼
   74HC595 #4
```

Los datos atraviesan todos los registros.

Esto permite controlar:

```text
4 × 8 = 32 salidas
```

utilizando esencialmente las mismas líneas de control.

---

# 6. Cascada del 74HC595

El 74HC595 dispone de:

```text
SER
SRCLK
RCLK
SRCLR
OE
Q0...Q7
Q7'
```

La salida serial:

```text
Q7'
```

permite conectar el registro con el siguiente.

Conceptualmente:

```text
ESP32
 │
 │ DATA
 ▼
┌────────────┐
│ 74HC595 #1 │
└─────┬──────┘
      │ Q7'
      ▼
┌────────────┐
│ 74HC595 #2 │
└─────┬──────┘
      │ Q7'
      ▼
┌────────────┐
│ 74HC595 #3 │
└────────────┘
```

Todos comparten:

```text
CLOCK
LATCH
```

---

# 7. ¿El 74HC595 necesita Chip Select?

No.

El 74HC595 **no utiliza una línea CS/Chip Select equivalente a SPI**.

La selección temporal del registro se realiza mediante:

```text
CLOCK
+
LATCH
```

El ESP32 puede desplazar todos los bits necesarios y posteriormente activar el latch.

Por ejemplo:

```text
32 bits enviados
        ↓
CLOCK
        ↓
LATCH
        ↓
32 salidas actualizadas
```

Por este motivo es especialmente interesante para ampliar salidas digitales con muy pocos GPIO del ESP32.

---

# 8. 74HC595 — Arquitectura

Cada 74HC595 proporciona:

```text
8 salidas digitales
```

pero internamente utiliza:

```text
Shift Register
+
Storage Register
```

Esto permite separar:

```text
desplazamiento de datos
```

de:

```text
actualización de salidas
```

---

# 9. Ventaja del LATCH

Supongamos:

```text
Salida 1 = ON
Salida 2 = OFF
Salida 3 = ON
...
```

Los nuevos valores pueden desplazarse al registro sin modificar inmediatamente las salidas.

Finalmente:

```text
LATCH
```

actualiza todas las salidas.

Esto evita que las salidas cambien parcialmente durante la transmisión.

---

# 10. Ejemplo de 4 × 74HC595

```text
ESP32

GPIO DATA
GPIO CLOCK
GPIO LATCH
      │
      ▼
┌──────────┐
│ 595 #1   │
│ Q0-Q7    │
└────┬─────┘
     │
     ▼
┌──────────┐
│ 595 #2   │
│ Q0-Q7    │
└────┬─────┘
     │
     ▼
┌──────────┐
│ 595 #3   │
│ Q0-Q7    │
└────┬─────┘
     │
     ▼
┌──────────┐
│ 595 #4   │
│ Q0-Q7    │
└──────────┘
```

Resultado:

```text
32 salidas
```

utilizando:

```text
3 GPIO principales
```

más las líneas auxiliares opcionales:

```text
OE
SRCLR
```

---

# 11. OE del 74HC595

`OE` significa:

```text
Output Enable
```

Permite habilitar o deshabilitar las salidas.

Normalmente:

```text
OE = LOW
```

habilita las salidas.

```text
OE = HIGH
```

pone las salidas en alta impedancia.

---

# 12. SRCLR del 74HC595

`SRCLR` permite limpiar el registro de desplazamiento.

Normalmente se mantiene en el estado inactivo mediante hardware.

La plataforma debe contemplar esta señal cuando sea necesario realizar:

* reset;
* inicialización segura;
* apagado de salidas;
* recuperación de errores.

---

# 13. 74HC165

El 74HC165 funciona conceptualmente en sentido inverso.

Es un:

```text
Parallel-In
Serial-Out
```

Mientras el 74HC595 es:

```text
Serial-In
Parallel-Out
```

Por lo tanto:

```text
74HC595
ESP32 → chip → salidas

74HC165
entradas → chip → ESP32
```

---

# 14. Entradas del 74HC165

Cada 74HC165 proporciona:

```text
8 entradas paralelas
```

Estas entradas pueden ser cargadas simultáneamente y posteriormente desplazadas hacia el ESP32.

---

# 15. Cascada del 74HC165

También puede utilizarse una cadena:

```text
Entradas
   │
   ▼
┌──────────┐
│ 165 #1   │
└────┬─────┘
     │ serial
     ▼
┌──────────┐
│ 165 #2   │
└────┬─────┘
     │ serial
     ▼
┌──────────┐
│ 165 #3   │
└──────────┘
     │
     ▼
   ESP32
```

Resultado:

```text
3 × 8 = 24 entradas
```

---

# 16. ¿El 74HC165 necesita Chip Select?

No utiliza un `CS` equivalente al Chip Select tradicional de SPI.

Dispone de señales para:

* carga paralela;
* reloj;
* desplazamiento serial.

Por lo tanto, la cadena se controla mediante sus señales de control y no mediante una dirección individual para cada chip.

---

# 17. Señal PL del 74HC165

La señal de carga paralela permite copiar las entradas físicas al registro.

Conceptualmente:

```text
Entradas físicas
       ↓
Parallel Load
       ↓
Shift Register
       ↓
Serial Data
       ↓
ESP32
```

Esto permite capturar el estado de las entradas y posteriormente leerlas mediante desplazamiento.

---

# 18. Combinación 595 + 165

Es perfectamente válido utilizar ambos tipos en el mismo sistema.

Ejemplo:

```text
ESP32
 │
 ├─────────────── 74HC595 chain
 │                    │
 │                    └── Salidas
 │
 └─────────────── 74HC165 chain
                      │
                      └── Entradas
```

Esto es especialmente útil para:

```text
Paneles
Teclados
Controladores de relés
Máquinas
Tableros
Automatización
```

---

# 19. Arquitectura combinada

Ejemplo:

```text
ESP32
 │
 ├── DATA_OUT
 ├── DATA_IN
 ├── CLOCK
 │
 ├── LATCH_595
 │
 └── LOAD_165
```

Puede existir:

```text
64 salidas
+
64 entradas
```

con una cantidad muy reducida de GPIO.

---

# 20. Limitación importante

Los 74HC595 y 74HC165 son **registros digitales**, no GPIO expanders inteligentes.

No proporcionan por sí mismos:

* configuración individual de entrada/salida;
* pull-up interno equivalente al de un GPIO;
* interrupciones;
* dirección I²C;
* dirección SPI;
* lectura individual sin desplazar el registro;
* gestión de estado independiente.

La lógica debe implementarse en el microcontrolador.

---

# 21. MCP23017

El MCP23017 es un expansor GPIO de:

```text
16 GPIO
```

mediante:

```text
I²C
```

Puede configurar cada pin como:

```text
INPUT
OUTPUT
```

y dispone de funciones adicionales para GPIO.

---

# 22. Organización del MCP23017

Los 16 GPIO se organizan como:

```text
GPIOA
 ├── GPA0
 ├── GPA1
 ├── GPA2
 ├── GPA3
 ├── GPA4
 ├── GPA5
 ├── GPA6
 └── GPA7

GPIOB
 ├── GPB0
 ├── GPB1
 ├── GPB2
 ├── GPB3
 ├── GPB4
 ├── GPB5
 ├── GPB6
 └── GPB7
```

---

# 23. MCP23017 y direccionamiento

El MCP23017 utiliza líneas de dirección:

```text
A0
A1
A2
```

Esto permite configurar diferentes direcciones I²C.

Por lo tanto, varios MCP23017 pueden coexistir en el mismo bus siempre que no tengan la misma dirección.

Conceptualmente:

```text
ESP32
 │
 ├── SDA ──────────────┬──────────┬──────────┐
 │                     │          │          │
 └── SCL ──────────────┼──────────┼──────────┤
                       │          │          │
                    MCP23017   MCP23017   MCP23017
                     Addr 0     Addr 1     Addr 2
```

---

# 24. ¿El MCP23017 tiene cascada?

No posee una cascada de desplazamiento equivalente al:

```text
Q7' → SER
```

del 74HC595.

Los MCP23017 no se conectan uno detrás de otro formando una cadena de bits.

En cambio:

```text
todos comparten SDA
todos comparten SCL
cada uno tiene una dirección
```

---

# 25. Ejemplo con varios MCP23017

```text
ESP32
 │
 ├── SDA ─────────┬──────────┬──────────┬──────────┐
 │                │          │          │          │
 └── SCL ─────────┼──────────┼──────────┼──────────┤
                  │          │          │          │
              MCP23017   MCP23017   MCP23017   MCP23017
               0x20       0x21       0x22       0x23
```

Cada dispositivo posee:

```text
16 GPIO
```

Por lo tanto:

```text
4 × 16 = 64 GPIO
```

---

# 26. Ventaja del MCP23017

A diferencia del 74HC595/165, el MCP23017 proporciona características propias de un expansor GPIO.

Puede gestionar:

* entradas;
* salidas;
* pull-ups;
* interrupciones;
* lectura/escritura de registros;
* configuración individual.

Por eso es más apropiado cuando el firmware necesita tratar los GPIO expandidos como GPIO relativamente independientes.

---

# 27. Interrupciones del MCP23017

El MCP23017 dispone de salidas de interrupción:

```text
INTA
INTB
```

Estas pueden utilizarse para informar al ESP32 de cambios en las entradas.

Ejemplo:

```text
Pulsador
   ↓
MCP23017
   ↓
INT
   ↓
ESP32
```

Esto evita tener que consultar continuamente todos los GPIO.

---

# 28. MCP23S17

El MCP23S17 es conceptualmente similar al MCP23017, pero utiliza:

```text
SPI
```

en lugar de:

```text
I²C
```

También proporciona:

```text
16 GPIO
```

---

# 29. MCP23S17 y Chip Select

El MCP23S17 utiliza:

```text
CS
```

para seleccionar el dispositivo en el bus SPI.

Ejemplo:

```text
ESP32
 │
 ├── MOSI ─────────┬─────────┬─────────┐
 ├── MISO ─────────┼─────────┼─────────┤
 ├── SCLK ─────────┼─────────┼─────────┤
 │                 │         │         │
 ├── CS1 ──────────┤         │         │
 ├── CS2 ──────────┼─────────┤         │
 └── CS3 ──────────┼─────────┼─────────┤
                   │         │         │
                MCP23S17  MCP23S17  MCP23S17
```

---

# 30. ¿El MCP23S17 tiene cascada?

No tiene una cascada de desplazamiento como:

```text
74HC595 → 74HC595 → 74HC595
```

Los dispositivos comparten:

```text
MOSI
MISO
SCLK
```

y se seleccionan mediante:

```text
CS
```

---

# 31. Dirección del MCP23S17

El MCP23S17 también dispone de configuración de dirección mediante:

```text
A0
A1
A2
```

Por lo tanto, el diseño puede combinar:

```text
CS
+
Hardware Address
```

según la configuración del dispositivo y del bus.

---

# 32. Comparación arquitectónica

## Shift Register

```text id="k9b8mz"
ESP32
 │
 ▼
595
 │
 ▼
595
 │
 ▼
595
```

## I²C GPIO Expander

```text id="c1y3te"
ESP32
 │
 ├──── 23017 @ address 1
 ├──── 23017 @ address 2
 ├──── 23017 @ address 3
 └──── 23017 @ address 4
```

## SPI GPIO Expander

```text id="5e9a4m"
ESP32
 │
 ├──── CS1 → 23S17
 ├──── CS2 → 23S17
 ├──── CS3 → 23S17
 └──── CS4 → 23S17
```

---

# 33. Diferencia fundamental

La arquitectura puede resumirse como:

```text
74HC595
→ cadena de bits
```

```text
74HC165
→ cadena de bits
```

```text
MCP23017
→ dispositivos direccionables en I²C
```

```text
MCP23S17
→ dispositivos seleccionables en SPI
```

---

# 34. ¿Cuál consume menos GPIO del ESP32?

## 74HC595

Mínimo conceptual:

```text
DATA
CLOCK
LATCH
```

Para N chips:

```text
3 GPIO
```

sin contar señales opcionales.

---

## 74HC165

Mínimo conceptual:

```text
DATA
CLOCK
LOAD
```

Para N chips:

```text
3 GPIO
```

aproximadamente.

---

## MCP23017

```text
SDA
SCL
```

Total:

```text
2 GPIO
```

independientemente de varios dispositivos en el mismo bus.

---

## MCP23S17

Normalmente:

```text
MOSI
MISO
SCLK
```

más:

```text
CS
```

por dispositivo, salvo arquitecturas especiales de selección.

---

# 35. Comparación práctica

| Característica               |   74HC595 |   74HC165 |  MCP23017 |  MCP23S17 |
| ---------------------------- | --------: | --------: | --------: | --------: |
| Entradas                     |        No |        Sí |        Sí |        Sí |
| Salidas                      |        Sí |        No |        Sí |        Sí |
| GPIO                         |         8 |         8 |        16 |        16 |
| Interfaz                     |    Serial |    Serial |       I²C |       SPI |
| Cascada directa              |        Sí |        Sí |        No |        No |
| Dirección                    |        No |        No |        Sí |        Sí |
| Chip Select                  |        No |        No |        No |        Sí |
| Pull-up configurable         |        No |        No |        Sí |        Sí |
| Interrupciones               |        No |        No |        Sí |        Sí |
| Lectura individual           |        No |        No |        Sí |        Sí |
| Escritura individual         |        No |        No |        Sí |        Sí |
| Configuración por GPIO       |        No |        No |        Sí |        Sí |
| Ideal para muchas salidas    | Excelente |         — | Muy bueno | Muy bueno |
| Ideal para muchas entradas   |         — | Excelente | Muy bueno | Muy bueno |
| Ideal para GPIO inteligentes |        No |        No | Excelente | Excelente |

---

# 36. ¿Qué solución debe utilizar la plataforma?

No debe existir un único expansor obligatorio.

El sistema debe soportar diferentes tipos de expansores según la necesidad.

---

# 37. Uso recomendado del 74HC595

Preferir:

```text
74HC595
```

para:

* relés;
* LEDs;
* indicadores;
* salidas digitales simples;
* paneles;
* grandes cantidades de salidas;
* aplicaciones donde la latencia no sea crítica;
* matrices de salidas.

---

# 38. Uso recomendado del 74HC165

Preferir:

```text
74HC165
```

para:

* pulsadores;
* interruptores;
* finales de carrera;
* contactos;
* entradas digitales;
* paneles;
* teclados;
* sensores digitales simples.

---

# 39. Uso recomendado del MCP23017

Preferir:

```text
MCP23017
```

cuando se necesite:

* entradas y salidas en el mismo dispositivo;
* interrupciones;
* pull-ups;
* control individual;
* configuración dinámica;
* integración sencilla mediante I²C.

---

# 40. Uso recomendado del MCP23S17

Preferir:

```text
MCP23S17
```

cuando:

* se necesita mayor velocidad que I²C;
* ya existe un bus SPI;
* existen muchos dispositivos SPI;
* se desea reducir el tiempo de actualización;
* se dispone de líneas CS.

---

# 41. Rendimiento

Los 74HC595/165 son extremadamente sencillos.

El costo computacional consiste principalmente en:

```text
desplazar bits
```

Mientras que los MCP requieren:

```text
transacciones de bus
+
registros
+
direccionamiento
```

Por lo tanto, no debe asumirse que:

```text
más funciones
=
mejor solución
```

La elección debe depender del caso de uso.

---

# 42. Actualización de salidas

Con un 74HC595:

```text
Modificar estado local
       ↓
Preparar buffer
       ↓
Shift out
       ↓
Latch
       ↓
Actualizar salidas
```

Esto permite actualizar un conjunto completo de salidas de forma determinista.

---

# 43. Actualización MCP23017

Con MCP23017:

```text
Modificar GPIO
       ↓
Generar transacción I²C
       ↓
Escribir registro
       ↓
GPIO actualizado
```

Se puede modificar un GPIO individual sin tener que transmitir necesariamente todo el estado de los demás.

---

# 44. Lectura 74HC165

```text
LOAD
 ↓
Capturar entradas
 ↓
CLOCK
 ↓
SHIFT
 ↓
ESP32 recibe bits
```

---

# 45. Lectura MCP23017

```text
ESP32
 ↓
I²C
 ↓
Registro GPIO
 ↓
Valor de entrada
```

---

# 46. Buffer interno de la plataforma

La plataforma debe abstraer ambos sistemas mediante un modelo común.

Ejemplo:

```text
IO Expander
│
├── channel 0
├── channel 1
├── channel 2
├── ...
└── channel N
```

El firmware no debería obligar a la aplicación superior a conocer si el canal pertenece a:

```text
74HC595
74HC165
MCP23017
MCP23S17
```

---

# 47. Modelo lógico

Ejemplo:

```json id="3k1i7x"
{
  "id": "ioexpander_01",
  "type": "74hc595",
  "channels": 8
}
```

Otro:

```json id="1z50ez"
{
  "id": "ioexpander_02",
  "type": "mcp23017",
  "channels": 16
}
```

La aplicación utiliza:

```text
channel
```

en lugar de:

```text
GPIO físico
```

---

# 48. Canal lógico

Cada canal debe poder tener:

```text
channel_id
direction
function
state
inverted
active_level
entity_id
```

Ejemplo:

```json id="d7tqzt"
{
  "channel_id": 5,
  "direction": "output",
  "function": "relay",
  "inverted": false,
  "entity_id": "switch.pump"
}
```

---

# 49. Identificación del recurso

Un expansor debe considerarse un:

```text
Resource
```

dentro del Device Model.

Ejemplo:

```text
Device
 └── Resource
      └── IO Expander
           ├── Channel 0
           ├── Channel 1
           ├── Channel 2
           └── ...
```

---

# 50. Ejemplo de arquitectura

```text
ESP32
│
├── GPIO
│
├── I2C Bus
│    ├── MCP23017 #1
│    └── MCP23017 #2
│
├── SPI Bus
│    └── MCP23S17
│
├── Shift Register Output Bus
│    ├── 74HC595 #1
│    ├── 74HC595 #2
│    └── 74HC595 #3
│
└── Shift Register Input Bus
     ├── 74HC165 #1
     └── 74HC165 #2
```

Todos pueden coexistir.

---

# 51. Configuración desde Web UI

La configuración debe poder realizarse desde la interfaz.

Ejemplo:

```text
Hardware
 └── Expansores
```

Agregar:

```text
[ + Agregar expansor ]
```

---

# 52. Formulario de expansor

Ejemplo:

```text
Tipo:

[ MCP23017 ▼ ]

Bus:

[ I²C 0 ▼ ]

Dirección:

[ 0x20 ▼ ]

Nombre:

[ Expansor Living ]

Cantidad de canales:

16
```

---

# 53. Configuración de 74HC595

```text
Tipo:
74HC595

Bus:
Shift Register

DATA:
GPIO 23

CLOCK:
GPIO 18

LATCH:
GPIO 5

Cantidad:
4
```

El sistema calcula:

```text
4 × 8 = 32 canales
```

---

# 54. Configuración de 74HC165

```text
Tipo:
74HC165

Bus:
Shift Register

DATA:
GPIO 19

CLOCK:
GPIO 18

LOAD:
GPIO 5

Cantidad:
4
```

Resultado:

```text
32 entradas
```

---

# 55. Configuración MCP23017

```text
Tipo:
MCP23017

Bus:
I²C 0

SDA:
GPIO 21

SCL:
GPIO 22

Dirección:
0x20
```

---

# 56. Configuración MCP23S17

```text
Tipo:
MCP23S17

Bus:
SPI 2

MOSI:
GPIO 23

MISO:
GPIO 19

SCLK:
GPIO 18

CS:
GPIO 5

Dirección:
0
```

---

# 57. Configuración de canales

Después de detectar el expansor:

```text
MCP23017 #1

GPIO 0 → Relay Living
GPIO 1 → Relay Kitchen
GPIO 2 → Button Living
GPIO 3 → Button Kitchen
GPIO 4 → Sensor Door
...
```

---

# 58. Funciones de canal

Cada canal puede configurarse como:

```text
Digital Input
Digital Output
Relay
Button
Switch
LED
Alarm
Sensor
Counter
```

según las capacidades reales del hardware.

---

# 59. Protección de configuración

La UI debe impedir configuraciones incompatibles.

Ejemplo:

```text
74HC595

Dirección:
OUTPUT
```

No debería ofrecer:

```text
INPUT
```

porque el 74HC595 es un registro de salida.

Igualmente:

```text
74HC165
```

debe limitarse a entradas.

---

# 60. Configuración automática

Cuando sea posible, el sistema puede detectar:

```text
MCP23017
MCP23S17
```

mediante:

* escaneo I²C;
* configuración conocida de SPI;
* identificación lógica;
* configuración guardada.

Los 74HC595/165 no ofrecen una identificación automática equivalente.

---

# 61. Diferencia de descubrimiento

```text
MCP23017
→ puede descubrirse mediante I²C scan
```

```text
MCP23S17
→ no debe asumirse descubrimiento SPI universal
```

```text
74HC595
→ no tiene identificación propia
```

```text
74HC165
→ no tiene identificación propia
```

Por lo tanto, los shift registers normalmente deben configurarse explícitamente.

---

# 62. Hot Plug

No debe asumirse que estos dispositivos soportan hot-plug seguro.

La plataforma debe considerar:

```text
expansor presente
expansor ausente
expansor no responde
```

como estados de hardware.

---

# 63. Estado del expansor

Cada expansor debe tener:

```text
ONLINE
OFFLINE
ERROR
UNKNOWN
```

cuando el protocolo permita determinarlo.

Para shift registers puede ser necesario inferir el estado mediante:

```text
diagnóstico externo
configuración
feedback
```

ya que no poseen identificación propia.

---

# 64. Fail-Safe

Las salidas deben tener un comportamiento definido durante:

* boot;
* reset;
* pérdida de comunicación;
* fallo de firmware;
* watchdog;
* actualización OTA.

Ejemplo:

```text
Relay de bomba
→ OFF durante boot
```

---

# 65. Importante para 74HC595

El estado de las salidas de un 74HC595 no debe asumirse como seguro solamente porque el firmware lo haya configurado.

El diseño eléctrico debe contemplar:

```text
OE
pull-up / pull-down
transistores
drivers
relés
```

según la aplicación.

---

# 66. Corriente de salida

Los expansores no deben utilizarse directamente para cargas que superen las capacidades eléctricas del dispositivo.

Para cargas como:

```text
Relés
Motores
Solenoides
Válvulas
LED de alta corriente
```

deben utilizarse drivers adecuados.

Ejemplo:

```text
ESP32
 ↓
74HC595
 ↓
Transistor / Driver
 ↓
Relay
```

---

# 67. Relés

Arquitectura recomendada:

```text
74HC595
   ↓
Driver
   ↓
Relay
```

No:

```text
74HC595
   ↓
Bobina de relay
```

sin verificar previamente las características eléctricas.

---

# 68. Entradas

Las entradas de:

```text
74HC165
MCP23017
MCP23S17
```

deben respetar:

* niveles lógicos;
* tensión máxima;
* protección;
* filtrado;
* pull-up/pull-down;
* rebote.

---

# 69. Debounce

Los pulsadores conectados a expansores deben soportar debounce.

Puede realizarse:

```text
Hardware
```

o:

```text
Software
```

o ambos.

El debounce debe formar parte de la configuración del canal cuando corresponda.

---

# 70. Interrupciones

Para MCP23017/MCP23S17:

```text
GPIO del expansor
        ↓
INT
        ↓
ESP32
```

Para 74HC165:

```text
No existe una salida de interrupción equivalente.
```

Puede utilizarse:

* polling;
* GPIO externo;
* lógica adicional;
* hardware de interrupción externo.

---

# 71. Sleep / Low Power

Los MCP pueden ser interesantes en sistemas que utilizan:

```text
Deep Sleep
Light Sleep
```

especialmente cuando sus interrupciones se utilizan para despertar al ESP32.

Los 74HC165 pueden participar en arquitecturas de lectura, pero no proporcionan por sí mismos una línea de interrupción equivalente.

---

# 72. Uso en paneles

Ejemplo:

```text
Panel táctil / botones

24 botones
+
16 LEDs
```

Puede utilizarse:

```text
3 × 74HC165
+
2 × 74HC595
```

con muy pocos GPIO del ESP32.

---

# 73. Uso en tablero de relés

Ejemplo:

```text
32 relés
```

puede utilizar:

```text
4 × 74HC595
```

y drivers externos.

---

# 74. Uso en tablero mixto

Ejemplo:

```text
16 entradas
16 salidas
```

Puede utilizar:

```text
1 × MCP23017
```

---

# 75. Uso en aplicaciones industriales

Para aplicaciones con muchos canales simples:

```text
74HC595
74HC165
```

pueden ser una solución económica.

Para aplicaciones donde se necesita:

```text
interrupción
configuración individual
lectura individual
pull-up
```

puede ser preferible:

```text
MCP23017
MCP23S17
```

---

# 76. Comparación de arquitectura de software

## 74HC595

```text
write(buffer)
```

## 74HC165

```text
read(buffer)
```

## MCP23017

```text
configure(pin)
read(pin)
write(pin)
configure_interrupt(pin)
```

## MCP23S17

```text
configure(pin)
read(pin)
write(pin)
configure_interrupt(pin)
```

La capa de abstracción debe ocultar estas diferencias.

---

# 77. Driver recomendado

La arquitectura de firmware debe separar:

```text
Application
     ↓
IO Abstraction
     ↓
Expander Driver
     ↓
Bus Driver
     ↓
Hardware
```

---

# 78. No mezclar drivers

No debe existir lógica de:

```text
MCP23017
```

dentro de:

```text
Automations
Scenes
Entities
```

El driver debe encapsular el hardware.

---

# 79. Arquitectura propuesta

```text
Application
│
├── Entities
├── Automations
├── Scenes
└── Functions
       │
       ▼
IO Abstraction
       │
       ├── DigitalInput
       ├── DigitalOutput
       └── Interrupt
       │
       ▼
Expander Manager
       │
       ├── 74HC595 Driver
       ├── 74HC165 Driver
       ├── MCP23017 Driver
       └── MCP23S17 Driver
       │
       ▼
Bus Manager
       │
       ├── SPI
       ├── I²C
       └── Shift Register
```

---

# 80. Expander Manager

Debe existir un administrador lógico:

```text
ExpanderManager
```

responsable de:

* registrar expansores;
* inicializarlos;
* mantener configuración;
* actualizar canales;
* gestionar errores;
* exponer estado;
* coordinar buses.

---

# 81. Ejemplo conceptual

```cpp
ExpanderManager
    .registerExpander(...)
    .configureChannel(...)
    .readInput(...)
    .writeOutput(...)
```

La aplicación no debería necesitar saber qué driver específico existe detrás.

---

# 82. Identificación interna

Cada expansor debe tener un identificador estable:

```text
expander_id
```

Ejemplo:

```text
expander.living.outputs
```

o un UUID interno.

---

# 83. Canal estable

Cada canal debe identificarse mediante:

```text
expander_id
+
channel_id
```

Ejemplo:

```text
expander.living.outputs
channel 7
```

---

# 84. Entity Mapping

El canal puede estar vinculado a una entidad:

```text
expander.living.outputs
channel 7
        ↓
switch.living_lamp
```

Esto permite cambiar el hardware sin cambiar necesariamente la entidad.

---

# 85. Reasignación

Ejemplo:

```text
switch.living_lamp
```

originalmente:

```text
74HC595 #1
channel 3
```

puede posteriormente pasar a:

```text
MCP23017 #2
GPA4
```

sin modificar la automatización.

---

# 86. Regla de abstracción

Las automatizaciones deben utilizar:

```text
switch.living_lamp
```

y nunca:

```text
74HC595_1.Q3
```

---

# 87. Diagnóstico técnico

En modo técnico sí puede mostrarse:

```text
switch.living_lamp

Hardware:
74HC595

Expander:
expander_living_01

Channel:
3

Physical:
Q3

State:
ON
```

---

# 88. Telemetría

Cuando corresponda, el sistema puede registrar:

```text
Expander online/offline
Bus errors
Communication errors
Interrupts
Read/write failures
```

---

# 89. Watchdog

Los drivers de expansores deben evitar bloquear indefinidamente una tarea.

Una operación de bus debe tener:

```text
timeout
```

y manejar:

```text
timeout
retry
error
recovery
```

según el tipo de bus.

---

# 90. FreeRTOS

El acceso a un mismo bus debe estar protegido.

Ejemplo:

```text
Task A
  ↓
I²C

Task B
  ↓
I²C
```

No deben acceder simultáneamente sin sincronización.

Debe utilizarse una abstracción de bus con:

```text
mutex
queue
semaphore
```

según la arquitectura.

---

# 91. Actualización periódica

No todos los expansores necesitan polling constante.

Ejemplo:

```text
74HC165
→ polling periódico

MCP23017
→ interrupción preferentemente

MCP23S17
→ interrupción preferentemente
```

cuando la aplicación lo permita.

---

# 92. Frecuencia de polling

Debe ser configurable.

Ejemplo:

```text
Fast:
1–10 ms

Normal:
20–100 ms

Slow:
100–1000 ms
```

No debe utilizarse una frecuencia alta sin necesidad.

---

# 93. Debounce configurable

Ejemplo:

```json id="pgrt8g"
{
  "debounce_ms": 30
}
```

---

# 94. Inversión lógica

Cada canal puede disponer de:

```text
inverted
```

Ejemplo:

```text
Hardware:
LOW = ON

Logical:
ON = true
```

Esto permite que la aplicación trabaje siempre con lógica normalizada.

---

# 95. Active Level

Debe distinguirse:

```text
active_level
```

de:

```text
inverted
```

cuando el diseño lo requiera.

Ejemplo:

```text
active_level:
LOW
```

---

# 96. Estado inicial

Cada salida puede definir:

```text
OFF
ON
LAST_STATE
SAFE_STATE
```

cuando el hardware lo permita.

---

# 97. Prioridad de seguridad

Para actuadores críticos:

```text
SAFE_STATE
```

debe tener prioridad sobre:

```text
LAST_STATE
```

durante boot o recuperación.

---

# 98. Persistencia

La configuración del expansor debe almacenarse en la configuración persistente del dispositivo.

Debe incluir:

```text
type
bus
address
chip select
pins
channel mapping
inversion
debounce
interrupts
safe state
```

---

# 99. Configuración JSON conceptual

```json id="7c1p2v"
{
  "id": "expander_living",
  "type": "mcp23017",
  "bus": "i2c0",
  "address": "0x20",
  "channels": [
    {
      "id": 0,
      "mode": "output",
      "function": "relay",
      "entity_id": "switch.living_light",
      "safe_state": "off"
    }
  ]
}
```

---

# 100. Configuración 74HC595

```json id="x5j34w"
{
  "id": "outputs_01",
  "type": "74hc595",
  "bus": "shift_out_0",
  "data_pin": 23,
  "clock_pin": 18,
  "latch_pin": 5,
  "count": 4
}
```

---

# 101. Configuración 74HC165

```json id="6xg7wt"
{
  "id": "inputs_01",
  "type": "74hc165",
  "bus": "shift_in_0",
  "data_pin": 19,
  "clock_pin": 18,
  "load_pin": 5,
  "count": 4
}
```

---

# 102. Configuración MCP23S17

```json id="2c6n1w"
{
  "id": "io_01",
  "type": "mcp23s17",
  "bus": "spi2",
  "cs_pin": 5,
  "address": 0
}
```

---

# 103. Configuración de bus

El bus debe ser un recurso independiente.

Ejemplo:

```text
SPI2
 ├── MCP23S17 #1
 ├── MCP23S17 #2
 └── Display
```

---

# 104. Compartición de SPI

El MCP23S17 puede compartir:

```text
MOSI
MISO
SCLK
```

con otros periféricos SPI.

Cada dispositivo debe disponer de su selección correspondiente.

---

# 105. Compartición de I²C

El MCP23017 puede compartir:

```text
SDA
SCL
```

con:

```text
AHT20
BMP280
SHT30
OLED
RTC
```

siempre que las direcciones y características eléctricas sean compatibles.

---

# 106. Pull-ups I²C

El bus I²C requiere una implementación eléctrica adecuada.

La plataforma debe considerar:

* resistencias pull-up;
* tensión del bus;
* capacitancia;
* longitud;
* velocidad;
* cantidad de dispositivos.

---

# 107. Topología física

Los buses deben respetar sus características eléctricas.

No debe interpretarse:

```text
I²C
```

como equivalente a:

```text
RS485
```

en cuanto a distancia y topología.

---

# 108. 74HC595 para largas cadenas

Aunque puede encadenarse una gran cantidad de registros, una cadena excesivamente larga aumenta:

* tiempo de actualización;
* susceptibilidad al ruido;
* capacitancia;
* complejidad del cableado.

Por lo tanto, la plataforma debe permitir dividir grandes cantidades de E/S en varios recursos.

---

# 109. Ejemplo de 128 salidas

En lugar de una única cadena:

```text
16 × 74HC595
```

puede ser conveniente:

```text
Bus A:
8 × 74HC595

Bus B:
8 × 74HC595
```

según las necesidades físicas y de rendimiento.

---

# 110. Arquitectura distribuida

Cuando la cantidad de E/S sea muy grande, no necesariamente debe agregarse todo al mismo ESP32.

Puede ser mejor:

```text
Central
   │
   ├── Node A
   │    └── 32 IO
   │
   ├── Node B
   │    └── 64 IO
   │
   └── Node C
        └── 32 IO
```

---

# 111. Principio de distribución

La expansión de GPIO no debe sustituir la arquitectura distribuida.

Los expansores sirven para:

```text
aumentar E/S de un nodo
```

mientras que los nodos sirven para:

```text
distribuir físicamente funciones
```

---

# 112. Ejemplo

Incorrecto:

```text
Un ESP32
+
256 entradas
+
256 salidas
+
todos los sensores
+
todos los actuadores
```

Preferible:

```text
Central
 │
 ├── Node Living
 │    └── 16 IO
 │
 ├── Node Garage
 │    └── 32 IO
 │
 └── Node Exterior
      └── 64 IO
```

---

# 113. Regla de diseño

Utilizar un expansor cuando:

```text
la cantidad de GPIO es insuficiente
```

pero utilizar otro nodo cuando:

```text
la distribución física
```

sea más conveniente.

---

# 114. Selección automática

La plataforma puede recomendar el tipo de expansor según:

```text
Cantidad de entradas
Cantidad de salidas
Necesidad de interrupciones
Velocidad
Cantidad de GPIO disponibles
Bus existente
Distancia
Costo
Consumo
```

---

# 115. Matriz de decisión

| Necesidad                     | Recomendación       |
| ----------------------------- | ------------------- |
| Muchas salidas simples        | 74HC595             |
| Muchas entradas simples       | 74HC165             |
| Entradas + salidas            | MCP23017            |
| E/S + interrupciones          | MCP23017 / MCP23S17 |
| Bus I²C existente             | MCP23017            |
| Bus SPI existente             | MCP23S17            |
| Cadena larga de salidas       | 74HC595             |
| Cadena larga de entradas      | 74HC165             |
| Configuración individual      | MCP23017 / MCP23S17 |
| Muy bajo costo por salida     | 74HC595             |
| Necesidad de direccionamiento | MCP23017 / MCP23S17 |

---

# 116. Recomendación para la plataforma

La implementación base debería soportar:

```text
DigitalInput
DigitalOutput
InterruptInput
```

como abstracciones.

Los drivers concretos serán:

```text
74HC165
74HC595
MCP23017
MCP23S17
```

---

# 117. Evolución futura

La misma abstracción puede incorporar posteriormente:

```text
PCF8574
PCF8575
TCA9534
TCA9555
MCP23008
MCP23S08
MCP23S08
```

u otros expansores compatibles.

La aplicación superior no debería necesitar modificarse.

---

# 118. Regla de compatibilidad

Agregar un nuevo expansor debe requerir principalmente:

```text
Nuevo driver
+
Registro del tipo
+
Schema
+
Configuración UI
```

y no modificaciones profundas de:

```text
Entities
Automations
Scenes
API
System Bus
```

---

# 119. Abstracción final

```text
                    APPLICATION
                         │
                         ▼
                  IO ABSTRACTION
                         │
            ┌────────────┼────────────┐
            │            │            │
            ▼            ▼            ▼
       Shift Output  Shift Input   GPIO Expander
            │            │            │
          595           165       ┌────┴────┐
                                   │         │
                                  23017     23S17
                                   │         │
                                  I²C       SPI
```

---

# 120. Regla de oro

> **Los 74HC595/74HC165 amplían el número de señales mediante cadenas de bits; los MCP23017/MCP23S17 amplían GPIO mediante dispositivos direccionables/seleccionables en un bus.**

No deben tratarse como si fueran exactamente el mismo tipo de hardware.

---

# 121. Resumen

```text
74HC595
→ 8 salidas
→ Serial-In / Parallel-Out
→ Cascada
→ Sin Chip Select
→ Excelente para grandes cantidades de salidas simples

74HC165
→ 8 entradas
→ Parallel-In / Serial-Out
→ Cascada
→ Sin Chip Select
→ Excelente para grandes cantidades de entradas simples

MCP23017
→ 16 GPIO
→ I²C
→ Direccionamiento
→ Sin cascada de desplazamiento
→ Entradas + salidas
→ Pull-ups
→ Interrupciones

MCP23S17
→ 16 GPIO
→ SPI
→ CS
→ Direccionamiento
→ Sin cascada de desplazamiento
→ Entradas + salidas
→ Pull-ups
→ Interrupciones
```

---

# 122. Principio final

La plataforma no debe pensar:

```text
"tengo un 74HC595"
```

sino:

```text
"tengo N canales digitales de salida disponibles"
```

El hardware concreto debe quedar encapsulado dentro de:

```text
Expander Driver
```

y el resto de la plataforma debe trabajar mediante:

```text
Resource
    ↓
Channel
    ↓
Capability
    ↓
Entity
    ↓
Function
    ↓
Automation
```

De esta manera, un `switch.living_light` puede estar conectado físicamente a:

```text
ESP32 GPIO
74HC595
MCP23017
MCP23S17
```

sin que la automatización, la API, la interfaz web o las integraciones externas tengan que conocer qué tecnología se utiliza físicamente.

> **El expansor es un detalle de implementación del hardware; la entidad lógica es la interfaz estable del sistema.**

# Design System — Plataforma de Automatización Distribuida

> **Tipo:** Convención / Arquitectura UX/UI
> **Estado:** Definición
> **Versión:** 2.0.0
> **Fecha:** 2026-10-05
> **Proyecto:** Plataforma de Automatización Distribuida
> **Compatibilidad:** ESP32 / Central / Web / API / Aplicaciones externas

---

# 1. Propósito

Este documento define el **Design System oficial de la plataforma**, estableciendo las reglas visuales, estructurales, funcionales y de experiencia de usuario que deberán seguir todas las interfaces del sistema.

El Design System no se limita a colores, tipografías o componentes gráficos.

También define:

* estructura de navegación;
* organización de páginas;
* componentes reutilizables;
* bloques funcionales;
* módulos instalables;
* permisos;
* estados;
* formularios;
* dashboards;
* configuración;
* automatizaciones;
* visualización de dispositivos;
* interacción con entidades;
* alertas;
* notificaciones;
* estados de conectividad;
* modo local/offline;
* adaptación a diferentes dispositivos;
* extensibilidad;
* integración con terceros.

El objetivo es que una instalación pequeña con un único ESP32 y una instalación grande con:

```text
Central
   │
   ├── Zona 1
   │     ├── Nodo 1
   │     ├── Nodo 2
   │     └── Nodo 3
   │
   ├── Zona 2
   │     ├── Nodo 4
   │     └── Nodo 5
   │
   └── Zona 3
         ├── Nodo 6
         └── Nodo 7
```

utilicen **la misma lógica visual y conceptual**.

---

# 2. Principios fundamentales

## 2.1 La interfaz debe representar funciones, no hardware

El usuario final no debería necesitar conocer:

* GPIO;
* registros Modbus;
* IDs CAN;
* direcciones I2C;
* buses SPI;
* UART;
* direcciones IP;
* MAC;
* pines físicos;
* sensores específicos;
* controladores internos.

La interfaz debe trabajar principalmente con:

```text
Entidad
Capacidad
Estado
Comando
Función
Zona
Grupo
Escena
Automatización
Modo
```

Ejemplo incorrecto:

```text
GPIO 27 → Relay
```

Ejemplo correcto:

```text
Luz del living
Estado: Encendida
```

---

# 3. Arquitectura visual

La interfaz seguirá la siguiente jerarquía:

```text
SISTEMA
   │
   ├── Sitios
   │
   ├── Zonas
   │
   ├── Grupos
   │
   ├── Dispositivos
   │
   ├── Entidades
   │
   ├── Funciones
   │
   ├── Escenas
   │
   ├── Automatizaciones
   │
   ├── Modos
   │
   ├── Eventos
   │
   ├── Historial
   │
   ├── Energía
   │
   ├── Integraciones
   │
   └── Sistema
```

La navegación debe reflejar esta jerarquía sin obligar al usuario a comprender la arquitectura interna.

---

# 4. Separación entre interfaz y arquitectura interna

La UI nunca debe depender directamente de:

```text
GPIO
I2C
SPI
UART
CAN
RS485
Wi-Fi
Ethernet
Modbus
MQTT
Matter
```

La UI consume el modelo lógico:

```text
Hardware
    ↓
Resource
    ↓
Capability
    ↓
Entity
    ↓
Function
    ↓
Zone / Group
    ↓
Scene / Automation
    ↓
UI / API / Integration
```

Esto permite cambiar el hardware sin modificar necesariamente la interfaz.

---

# 5. Principio Local-First

La interfaz debe asumir que el sistema puede funcionar sin Internet.

Debe existir una diferencia clara entre:

```text
Internet disponible
Internet no disponible
Central disponible
Central no disponible
Nodo disponible
Nodo no disponible
```

La pérdida de Internet no debe interpretarse automáticamente como:

```text
Sistema apagado
```

Por ejemplo:

```text
Internet:          OFFLINE
Central:           ONLINE
Zona Living:       ONLINE
Luz Living:        ONLINE
Automatización:    ACTIVA
```

La UI debe comunicar esta situación correctamente.

---

# 6. Jerarquía de funcionamiento

La interfaz debe representar la prioridad:

```text
SEGURIDAD LOCAL
      ↓
AUTOMATIZACIÓN LOCAL
      ↓
AUTOMATIZACIÓN DE ZONA
      ↓
AUTOMATIZACIÓN CENTRAL
      ↓
INTEGRACIONES EXTERNAS
      ↓
CLOUD / INTERNET
```

Una falla en una capa superior no debe representarse como una falla total del sistema.

---

# 7. Arquitectura de interfaz

La aplicación web utilizará una estructura modular:

```text
App
│
├── Shell
│   ├── Header
│   ├── Navigation
│   ├── Breadcrumbs
│   └── Status
│
├── Pages
│
├── Sections
│
├── Blocks
│
├── Components
│
└── Utilities
```

---

# 8. Shell principal

El Shell contiene la estructura común de toda la aplicación.

Debe incluir:

```text
┌──────────────────────────────────────────────────┐
│ Logo / Sistema        Estado       Usuario       │
├────────────┬─────────────────────────────────────┤
│            │                                     │
│ Dashboard  │                                     │
│ Zonas      │                                     │
│ Disposit.  │             CONTENIDO               │
│ Entidades  │                                     │
│ Escenas    │                                     │
│ Automac.   │                                     │
│ Historial  │                                     │
│ Energía    │                                     │
│ Integrac.  │                                     │
│ Sistema    │                                     │
│            │                                     │
└────────────┴─────────────────────────────────────┘
```

En dispositivos móviles la navegación puede transformarse en:

```text
Header
   ↓
Contenido
   ↓
Bottom Navigation
```

---

# 9. Navegación

La navegación debe ser consistente.

## 9.1 Navegación principal

Como mínimo:

```text
Dashboard
Zonas
Dispositivos
Entidades
Automatizaciones
Escenas
Historial
Energía
Integraciones
Sistema
```

Los módulos pueden agregar entradas adicionales.

---

# 10. Navegación basada en módulos

La navegación **no debe estar completamente hardcodeada**.

Debe poder construirse dinámicamente según:

* módulos instalados;
* módulos habilitados;
* permisos;
* capacidades disponibles;
* tipo de instalación;
* rol del usuario.

Ejemplo:

```text
Módulo iluminación
    ↓
Aparece "Iluminación"

Módulo energía
    ↓
Aparece "Energía"

Módulo riego
    ↓
Aparece "Riego"
```

Si un módulo se deshabilita:

```text
Módulo riego = OFF
```

la navegación correspondiente puede desaparecer.

---

# 11. Sistema modular

Toda funcionalidad visual deberá diseñarse como módulo o bloque reutilizable cuando sea posible.

Ejemplo:

```text
Módulo Energía
│
├── Dashboard energético
├── Consumo actual
├── Consumo histórico
├── Costos
├── Comparativas
└── Configuración
```

Otro ejemplo:

```text
Módulo Clima
│
├── Temperatura
├── Humedad
├── Presión
├── Viento
├── Lluvia
└── Pronóstico
```

---

# 12. Módulos instalables

Los módulos deben poder:

* instalarse;
* habilitarse;
* deshabilitarse;
* actualizarse;
* configurarse;
* eliminarse cuando sea seguro.

La plataforma debe poder funcionar con un conjunto mínimo de módulos.

Ejemplo:

```text
Instalación básica

☑ Core
☑ Dispositivos
☑ Entidades
☑ Automatizaciones
☐ Energía
☐ Riego
☐ Clima
☐ IA
☐ Cámaras
```

---

# 13. Bloques UI

Cada página deberá estar compuesta por bloques.

Ejemplo:

```text
Dashboard
│
├── Estado del sistema
├── Clima
├── Luces
├── Energía
├── Seguridad
├── Automatizaciones activas
└── Alertas
```

Cada bloque debe poder:

* mostrarse;
* ocultarse;
* reordenarse;
* configurarse;
* actualizarse independientemente.

---

# 14. Bloques configurables

Cuando sea razonable, el usuario podrá configurar:

```text
Título
Icono
Entidad
Unidad
Intervalo de actualización
Visualización
Orden
Tamaño
Visibilidad
```

Ejemplo:

```text
┌──────────────────────────┐
│ Temperatura Living       │
│                          │
│       23.4 °C            │
│                          │
│       ● Normal            │
└──────────────────────────┘
```

---

# 15. Dashboard

El Dashboard debe ser configurable.

No debe existir un único dashboard obligatorio.

Debe poder existir:

```text
Dashboard General
Dashboard Casa
Dashboard Oficina
Dashboard Producción
Dashboard Energía
Dashboard Clima
Dashboard Seguridad
```

Cada dashboard puede contener distintos bloques.

---

# 16. Dashboard por rol

Los dashboards pueden variar según el usuario.

Ejemplo:

### Administrador

```text
Sistema
Dispositivos
Usuarios
Integraciones
Diagnóstico
Automatizaciones
```

### Usuario normal

```text
Luces
Clima
Escenas
Seguridad
```

### Técnico

```text
Diagnóstico
Logs
Red
Hardware
Comunicación
Firmware
```

---

# 17. Responsive Design

La interfaz debe funcionar en:

* PC;
* notebook;
* tablet;
* teléfono;
* panel táctil;
* pantalla embebida;
* navegador local del ESP32;
* pantalla de control dedicada.

Breakpoints sugeridos:

```text
Mobile:
< 640 px

Tablet:
640–1023 px

Desktop:
≥ 1024 px

Large:
≥ 1440 px
```

No se debe depender exclusivamente de estos valores.

Los componentes deben ser fluidos.

---

# 18. Touch First

Las interfaces destinadas a paneles táctiles deben utilizar controles grandes.

Tamaño recomendado:

```text
Área táctil mínima:
44 × 44 px

Preferido:
48 × 48 px o superior
```

Evitar:

* botones pequeños;
* elementos demasiado juntos;
* acciones críticas sin confirmación;
* gestos difíciles de descubrir.

---

# 19. Diseño para ESP32

La Web UI puede ejecutarse directamente en un ESP32.

Por lo tanto debe minimizar:

* JavaScript innecesario;
* dependencias pesadas;
* imágenes grandes;
* fuentes externas;
* animaciones costosas;
* solicitudes HTTP innecesarias.

Preferencias:

```text
HTML
CSS
JavaScript modular
SVG
WebSocket
REST
```

Siempre que sea posible, los recursos estáticos deben poder servirse localmente.

---

# 20. Sin dependencia de Internet

La interfaz no debe depender de:

* Google Fonts;
* CDNs;
* APIs externas;
* imágenes remotas;
* servicios cloud.

Una instalación local debe poder funcionar completamente offline.

---

# 21. Tipografía

Se recomienda utilizar una familia sans-serif moderna.

Preferencia:

```text
Inter
```

Alternativas:

```text
system-ui
-apple-system
BlinkMacSystemFont
Segoe UI
Roboto
sans-serif
```

En dispositivos embebidos se priorizará:

```css
font-family:
    system-ui,
    -apple-system,
    BlinkMacSystemFont,
    "Segoe UI",
    sans-serif;
```

---

# 22. Escala tipográfica

Valores orientativos:

```text
Display:
32–40 px

H1:
28–32 px

H2:
24–28 px

H3:
20–24 px

Body:
14–16 px

Small:
12–14 px

Caption:
11–12 px
```

La tipografía debe mantener una jerarquía clara.

---

# 23. Espaciado

Utilizar una escala consistente.

Base:

```text
4 px
8 px
12 px
16 px
24 px
32 px
48 px
64 px
```

Evitar valores arbitrarios cuando no sean necesarios.

---

# 24. Grid

Sistema recomendado:

```text
4 px base
```

Los componentes deben alinearse utilizando múltiplos de la escala.

---

# 25. Bordes

Radio recomendado:

```text
Small:
4 px

Medium:
8 px

Large:
12 px

Card:
12–16 px

Modal:
16 px
```

La aplicación debe utilizar pocos radios diferentes.

---

# 26. Elevación

La elevación debe utilizarse de forma moderada.

Ejemplos:

```text
Nivel 0:
sin sombra

Nivel 1:
cards

Nivel 2:
dropdowns

Nivel 3:
dialogs

Nivel 4:
elementos críticos
```

No utilizar sombras excesivas.

---

# 27. Color

El sistema de color debe utilizar variables CSS.

Ejemplo:

```css
:root {
    --color-primary: ...;
    --color-secondary: ...;

    --color-background: ...;
    --color-surface: ...;
    --color-surface-variant: ...;

    --color-text: ...;
    --color-text-secondary: ...;

    --color-success: ...;
    --color-warning: ...;
    --color-error: ...;
    --color-info: ...;

    --color-border: ...;
}
```

Los colores concretos pueden cambiar según el tema.

Los componentes nunca deben depender de colores escritos directamente.

---

# 28. Tema claro y oscuro

El sistema debe soportar como mínimo:

```text
Light
Dark
```

Opcional:

```text
System
```

El usuario puede elegir:

```text
Claro
Oscuro
Automático
```

---

# 29. Estados visuales

Todos los componentes interactivos deben contemplar:

```text
Default
Hover
Focus
Active
Disabled
Loading
Success
Warning
Error
Unavailable
```

---

# 30. Estados de conectividad

La conectividad es un estado fundamental.

Debe poder representarse:

```text
ONLINE
OFFLINE
DEGRADED
CONNECTING
UNKNOWN
```

Ejemplo:

```text
● Online
● Degradado
● Offline
```

---

# 31. Estados de entidades

Las entidades pueden encontrarse en:

```text
Available
Unavailable
Unknown
Stale
Invalid
Disabled
```

No confundir:

```text
OFF
```

con:

```text
UNAVAILABLE
```

Ejemplo:

```text
Luz:
OFF
```

significa que la entidad está disponible y apagada.

Mientras:

```text
Luz:
UNAVAILABLE
```

significa que no se puede determinar/controlar correctamente su estado.

---

# 32. Calidad de datos

Los sensores deben poder mostrar:

```text
Good
Uncertain
Stale
Invalid
Unavailable
```

Ejemplo:

```text
Temperatura
23.4 °C
✓ Datos válidos
```

o:

```text
Temperatura
23.4 °C
⚠ Lectura antigua
```

---

# 33. Confianza

Cuando una entidad lo permita, se puede mostrar:

```text
Confidence: 96 %
```

Especialmente útil para:

* IA;
* visión artificial;
* presencia;
* reconocimiento;
* predicciones;
* detección de anomalías.

---

# 34. Cards

Las Cards son uno de los componentes principales.

Ejemplo:

```text
┌────────────────────────────┐
│ 💡 Living                  │
│                            │
│ Encendida                  │
│                            │
│ [──────●────────]  70 %    │
└────────────────────────────┘
```

Una Card debe mostrar solamente información relevante.

---

# 35. Entity Card

Componente especializado para entidades.

Ejemplo:

```text
Entity Card

Nombre
Estado
Valor
Unidad
Icono
Controles
Disponibilidad
```

Debe adaptarse al dominio.

---

# 36. Sensor Card

Ejemplo:

```text
Temperatura
23.4 °C

Mín: 19.1 °C
Máx: 26.7 °C

Actualizado hace 3 s
```

---

# 37. Actuator Card

Para:

* luces;
* ventiladores;
* relés;
* válvulas;
* bombas;
* persianas;
* motores.

Debe mostrar:

```text
Estado actual
Estado deseado
Control
Disponibilidad
```

Cuando exista diferencia:

```text
Deseado: ON
Actual: OFF
```

debe indicarse claramente.

---

# 38. Control de estado

Para acciones binarias:

```text
ON / OFF
```

Preferir:

* Switch;
* Toggle;
* Button.

No utilizar un control ambiguo.

---

# 39. Controles continuos

Para:

* brillo;
* velocidad;
* posición;
* temperatura;
* volumen.

Utilizar:

```text
Slider
Stepper
Numeric Input
```

según el caso.

---

# 40. Comandos

Los comandos representan una acción.

Ejemplo:

```text
Encender luz
```

No debe confundirse con:

```text
Estado = encendido
```

La UI debe respetar:

```text
Command
    ↓
Execution
    ↓
State change
```

---

# 41. Estado deseado vs real

Para actuadores distribuidos:

```text
Deseado:
ON

Real:
OFF
```

debe ser posible mostrar ambos.

Ejemplo:

```text
Bomba
────────────────

Deseado      ON
Real         OFF

Estado       Ejecutando...
```

---

# 42. Feedback de comandos

Toda acción debe proporcionar feedback.

Ejemplo:

```text
Enviando...
```

Después:

```text
Comando aceptado
```

o:

```text
Comando ejecutado
```

o:

```text
No se pudo ejecutar
```

---

# 43. Command ID

Las operaciones distribuidas pueden generar:

```text
command_id
request_id
correlation_id
```

La interfaz puede mostrar información técnica solamente cuando sea necesario.

Ejemplo para usuario normal:

```text
No se pudo encender la bomba.
```

Modo técnico:

```text
Command:
cmd_01J...

Request:
req_01J...

Node:
node_03

Error:
NODE_UNAVAILABLE
```

---

# 44. Escenas

Las escenas representan estados coordinados.

Ejemplo:

```text
Escena "Noche"

Living:
OFF

Dormitorio:
ON 20 %

Exterior:
ON

Persianas:
CERRADAS

Alarma:
ARMADA
```

---

# 45. Activación de escenas

La interfaz debe permitir:

```text
▶ Ejecutar
```

y mostrar:

```text
Preparando
Ejecutando
Completada
Parcial
Fallida
```

Una escena parcialmente ejecutada no debe mostrarse simplemente como:

```text
OK
```

---

# 46. Automatizaciones

Una automatización debe representarse visualmente como:

```text
CUANDO
    ↓
CONDICIÓN
    ↓
ACCIÓN
```

Ejemplo:

```text
CUANDO

Movimiento detectado

Y

Hora > 20:00

ENTONCES

Encender luz exterior
```

---

# 47. Editor de automatizaciones

El editor debe utilizar bloques.

Ejemplo:

```text
┌────────────────────────────┐
│ CUANDO                     │
│ Movimiento detectado      │
└────────────────────────────┘

        ↓

┌────────────────────────────┐
│ SI                         │
│ Hora > 20:00              │
└────────────────────────────┘

        ↓

┌────────────────────────────┐
│ ENTONCES                   │
│ Encender luz exterior     │
└────────────────────────────┘
```

Esto permite construir automatizaciones sin programar.

---

# 48. Automatizaciones avanzadas

El sistema debe poder evolucionar hacia:

```text
Triggers
Conditions
Actions
Delays
Timers
Loops
Variables
Expressions
Schedules
Modes
Priorities
```

Sin cambiar el modelo visual fundamental.

---

# 49. Modos del sistema

Los modos deben tener representación clara.

Ejemplo:

```text
NORMAL
DORMIR
AUSENTE
VACACIONES
MANTENIMIENTO
EMERGENCIA
```

El modo actual debe estar visible.

Ejemplo:

```text
Modo actual
🌙 DORMIR
```

---

# 50. Cambio de modo

Los cambios de modo deben mostrar:

```text
Modo anterior
Modo nuevo
Usuario / origen
Fecha
Hora
```

Ejemplo:

```text
NORMAL → DORMIR
Origen: Usuario
```

---

# 51. Zonas

Las zonas son una abstracción fundamental.

Ejemplo:

```text
Casa
│
├── Exterior
├── Living
├── Cocina
├── Dormitorio
└── Garaje
```

Una zona puede contener:

* dispositivos;
* entidades;
* grupos;
* escenas;
* automatizaciones.

---

# 52. Vista de zona

Una zona debe mostrar:

```text
Nombre
Estado
Entidades
Dispositivos
Automatizaciones
Alertas
Consumo
```

Ejemplo:

```text
Living

23.4 °C
65 % humedad

3 luces
1 ventilador
1 sensor

2 automatizaciones activas
```

---

# 53. Grupos

Los grupos permiten agrupar entidades independientemente de la ubicación.

Ejemplo:

```text
Grupo:
Luces principales

Incluye:
light.living
light.kitchen
light.hall
```

La UI debe diferenciar:

```text
Zona
```

de:

```text
Grupo
```

---

# 54. Dispositivos

La vista de dispositivo debe estar orientada a diagnóstico y configuración.

Ejemplo:

```text
ESP32-WROOM-32

Estado:
ONLINE

Firmware:
1.4.0

IP:
192.168.1.50

Zona:
Living

Entidades:
8

CPU:
42 %

RAM:
61 %

Uptime:
14 días
```

---

# 55. Recursos

Los recursos son principalmente una vista técnica.

Ejemplo:

```text
GPIO 27
GPIO 26
I2C Bus 0
UART 1
RS485
CAN
Ethernet
Wi-Fi
```

Esta información no debe dominar la UI normal.

Debe estar disponible en:

```text
Configuración
Diagnóstico
Modo técnico
```

---

# 56. Configuración de hardware

La configuración debe realizarse mediante formularios.

Ejemplo:

```text
Función:

[ Luz ]

Recurso:

[ GPIO 27 ]

Tipo:

[ Relay ]

Lógica:

[ Activo en HIGH ]
```

Nunca obligar al usuario normal a editar código.

---

# 57. Configuración avanzada

Debe existir una sección:

```text
Avanzado
```

para usuarios técnicos.

Puede mostrar:

* GPIO;
* buses;
* direcciones;
* registros;
* logs;
* timings;
* buffers;
* diagnóstico.

---

# 58. Protección contra errores

Las configuraciones potencialmente peligrosas deben tener:

* validación;
* límites;
* advertencias;
* confirmación;
* rollback cuando sea posible.

Ejemplo:

```text
⚠ Cambiar este recurso puede dejar
inaccesible el dispositivo.

[Cancelar]
[Continuar]
```

---

# 59. Formularios

Los formularios deben:

* agrupar campos relacionados;
* mostrar valores actuales;
* indicar campos obligatorios;
* validar en tiempo real;
* mostrar errores junto al campo;
* evitar pérdida de cambios.

---

# 60. Guardado de configuración

Cuando una configuración requiera varios cambios:

```text
Editar
    ↓
Cambios pendientes
    ↓
Validar
    ↓
Aplicar
    ↓
Confirmar
```

No aplicar automáticamente cambios críticos sin advertencia.

---

# 61. Configuración deseada vs aplicada

Para dispositivos distribuidos:

```text
Configuración deseada
        ↓
Sincronización
        ↓
Configuración aplicada
```

La UI debe poder indicar:

```text
✓ Sincronizado
```

o:

```text
⚠ Cambios pendientes
```

---

# 62. Versionado de configuración

Mostrar cuando sea relevante:

```text
Config v17
Aplicada
```

o:

```text
Config v18
Pendiente de aplicar
```

---

# 63. Eventos

Los eventos deben representarse como acontecimientos.

Ejemplo:

```text
10:31:22
Movimiento detectado
```

```text
10:31:23
Luz exterior encendida
```

```text
10:31:25
Puerta cerrada
```

---

# 64. Historial

El historial puede mostrar:

```text
Estado
Eventos
Comandos
Alarmas
Configuraciones
Usuarios
Conectividad
```

Debe permitir filtrar por:

```text
Fecha
Zona
Entidad
Dispositivo
Tipo
Severidad
Usuario
Origen
```

---

# 65. Gráficos

Los gráficos deben ser:

* legibles;
* responsivos;
* simples;
* eficientes.

Tipos:

```text
Line
Area
Bar
Gauge
Histogram
```

Evitar gráficos innecesariamente complejos.

---

# 66. Tiempo real

Para información dinámica se debe preferir:

```text
WebSocket
```

sobre polling agresivo.

Ejemplo:

```text
REST
    ↓
Carga inicial

WebSocket
    ↓
Actualizaciones
```

---

# 67. Indicador de tiempo real

Cuando corresponda:

```text
● En tiempo real
```

o:

```text
Actualizado hace 3 s
```

---

# 68. WebSocket

El cliente debe poder suscribirse a:

```text
state
event
command
availability
discovery
system
```

La UI no debe abrir múltiples conexiones innecesarias.

Preferir:

```text
1 conexión
N suscripciones
```

---

# 69. Alertas

Las alertas deben utilizar niveles:

```text
INFO
WARNING
ERROR
CRITICAL
```

Ejemplo:

```text
⚠ Temperatura alta
```

```text
✕ Nodo desconectado
```

```text
! Falla crítica de seguridad
```

---

# 70. Alertas críticas

Las alertas críticas deben:

* ser visibles;
* persistir;
* requerir reconocimiento cuando corresponda;
* registrar quién las reconoció;
* poder generar notificaciones.

---

# 71. Notificaciones

Canales posibles:

```text
Web
Push
Email
Telegram
SMS
WhatsApp
Home Assistant
MQTT
```

Los canales dependen de las integraciones instaladas.

---

# 72. Confirmaciones

Acciones normales:

```text
Click → Ejecutar
```

Acciones críticas:

```text
Click
 ↓
Confirmación
 ↓
Ejecutar
```

Ejemplos críticos:

* factory reset;
* eliminar dispositivo;
* eliminar automatización importante;
* desarmar seguridad;
* actualizar firmware;
* cambiar red;
* modificar hardware crítico.

---

# 73. Dialogs / Modals

Utilizar para:

* confirmaciones;
* formularios pequeños;
* información puntual;
* acciones críticas.

No utilizar modales para páginas completas.

---

# 74. Toasts

Los Toasts sirven para feedback temporal.

Ejemplo:

```text
✓ Configuración guardada
```

No utilizar Toast como único medio para errores críticos.

---

# 75. Loading

Todo proceso que pueda tardar debe mostrar estado.

Ejemplo:

```text
Conectando...
Sincronizando...
Aplicando configuración...
Actualizando firmware...
```

Evitar botones que parezcan congelados.

---

# 76. Empty States

Cuando no existan datos:

```text
No hay dispositivos

[Agregar dispositivo]
```

No mostrar simplemente una página vacía.

---

# 77. Error States

Ejemplo:

```text
No se pudo cargar la información.

[Reintentar]
```

Para modo técnico:

```text
Error:
NODE_UNAVAILABLE

Request:
req_01J...

[Ver diagnóstico]
```

---

# 78. Offline States

Cuando se pierde conexión:

```text
Sin conexión con el Central
```

pero si la aplicación local sigue funcionando:

```text
Modo local activo
```

La UI debe diferenciar ambos escenarios.

---

# 79. Seguridad

La interfaz debe respetar:

```text
Authentication
Authorization
Role
Permission
Scope
```

No mostrar controles para los que el usuario no tiene permiso.

---

# 80. Visibilidad vs permiso

No es lo mismo:

```text
No mostrar
```

que:

```text
Mostrar pero bloquear
```

Regla general:

### Funciones completamente irrelevantes

No mostrar.

### Funciones disponibles pero sin permiso

Mostrar cuando ayude a comprender el sistema, pero indicar:

```text
Sin permisos
```

---

# 81. Usuarios

El sistema puede utilizar:

```text
Administrator
Operator
User
Viewer
Technician
Service
```

Los nombres definitivos de roles pertenecen al sistema de autorización.

---

# 82. Modo técnico

La plataforma debe disponer de un modo técnico.

Ejemplo:

```text
Modo normal
```

muestra:

```text
Luz living
Temperatura
Escenas
Automatizaciones
```

Mientras:

```text
Modo técnico
```

puede mostrar:

```text
GPIO
Task
Heap
CPU
Wi-Fi RSSI
IP
MAC
CAN
RS485
MQTT
Logs
```

---

# 83. Separación de complejidad

Regla fundamental:

> La complejidad del sistema no debe obligar al usuario a comprender su arquitectura interna.

La plataforma puede ser técnicamente compleja mientras la interfaz continúa siendo simple.

---

# 84. Integraciones

Las integraciones se muestran como módulos.

Ejemplo:

```text
Integraciones

☑ MQTT
☑ Matter
☐ Home Assistant
☐ Google Home
☐ Apple Home
☐ Alexa
☐ Homey
☐ SmartThings
```

---

# 85. Integraciones y entidades

La UI debe permitir decidir qué entidades se exponen.

Ejemplo:

```text
Home Assistant

Exponer:

☑ light.living
☑ light.kitchen
☑ sensor.living.temperature
☐ sensor.internal.cpu
☐ diagnostic.rssi
```

---

# 86. API

La API debe considerarse parte del mismo Design System conceptual.

La UI y la API deben utilizar:

```text
Entity
Capability
State
Command
Event
Scene
Automation
Zone
Device
```

No deben existir modelos conceptuales diferentes.

---

# 87. API Explorer

Opcionalmente, en modo técnico:

```text
API

GET /api/v1/entities
GET /api/v1/devices
POST /api/v1/entities/{id}/commands
```

Puede incluir:

```text
Request
Response
Authentication
Schema
```

---

# 88. IDs visibles

Los IDs técnicos pueden mostrarse solamente cuando sea necesario.

Ejemplo:

```text
Luz Living
```

En modo técnico:

```text
Luz Living
entity_id: light.living
uuid: 01J...
```

---

# 89. Nombres amigables

Las entidades deben tener:

```text
entity_id
friendly_name
```

Ejemplo:

```text
entity_id:
light.living

friendly_name:
Luz del living
```

---

# 90. Iconografía

Los iconos deben representar la función.

Ejemplos:

```text
Luz
💡

Temperatura
🌡

Humedad
💧

Movimiento
🚶

Puerta
🚪

Energía
⚡

Agua
💧
```

En la implementación real se recomienda utilizar una librería SVG consistente en lugar de depender de emojis.

---

# 91. Iconos dinámicos

Cuando sea útil, los iconos pueden cambiar según estado.

Ejemplo:

```text
Luz apagada
Luz encendida
```

Pero el cambio debe ser sutil y consistente.

---

# 92. Accesibilidad

La UI debe cumplir buenas prácticas WCAG.

Como mínimo:

* contraste suficiente;
* navegación por teclado;
* focus visible;
* labels;
* textos alternativos;
* no depender únicamente del color;
* áreas táctiles adecuadas;
* mensajes accesibles.

---

# 93. No depender del color

Nunca comunicar un estado únicamente mediante:

```text
verde
rojo
amarillo
```

Agregar:

```text
icono
texto
estado
```

Ejemplo:

```text
✓ Online
```

en lugar de solamente:

```text
●
```

---

# 94. Internacionalización

La plataforma debe estar preparada para múltiples idiomas.

Ejemplo:

```text
es
en
it
pt
fr
de
```

El idioma no debe estar hardcodeado dentro de los componentes.

---

# 95. Unidades

Las unidades deben estar separadas del valor.

Ejemplo:

```text
23.4 °C
```

internamente:

```json
{
  "value": 23.4,
  "unit": "°C"
}
```

La UI puede convertir unidades según preferencias del usuario.

---

# 96. Sistema métrico

Preferencia inicial:

```text
Métrico
```

Debe ser configurable.

---

# 97. Fecha y hora

La UI debe respetar:

* timezone configurada;
* formato regional;
* 12/24 horas;
* horario de verano cuando corresponda.

---

# 98. Tiempo relativo

Para información reciente:

```text
hace 3 segundos
hace 2 minutos
hace 1 hora
```

Para información histórica:

```text
05/10/2026 12:31:25
```

---

# 99. Estado de sincronización

Cuando Central y nodos intercambien configuración o estado:

```text
Sincronizado
Sincronizando
Pendiente
Desincronizado
Error
```

Debe ser visible para usuarios técnicos.

---

# 100. Firmware

La actualización de firmware debe tener una interfaz específica.

Mostrar:

```text
Firmware actual
Nueva versión
Compatibilidad
Cambios
Estado
Progreso
Resultado
```

Ejemplo:

```text
Firmware

Actual:
1.4.0

Disponible:
1.5.0

[Actualizar]
```

---

# 101. Actualización OTA

Estados:

```text
Checking
Downloading
Verifying
Installing
Rebooting
Validating
Completed
Failed
Rollback
```

---

# 102. Seguridad durante OTA

Las actualizaciones deben validar:

* versión;
* compatibilidad;
* integridad;
* firma cuando corresponda;
* espacio disponible;
* estado del dispositivo.

La UI debe mostrar errores comprensibles.

---

# 103. Diagnóstico

El módulo de diagnóstico puede mostrar:

```text
CPU
RAM
Flash
PSRAM
Temperatura
Uptime
Wi-Fi
Ethernet
RSSI
IP
MQTT
CAN
RS485
Tasks
Watchdog
Logs
```

---

# 104. Logs

Los logs deben utilizar niveles:

```text
DEBUG
INFO
NOTICE
WARNING
ERROR
CRITICAL
```

El usuario normal no debería necesitar verlos.

---

# 105. Filtros de logs

Permitir:

```text
Nivel
Módulo
Dispositivo
Fecha
Texto
```

---

# 106. Arquitectura de componentes

Los componentes deben organizarse por niveles.

```text
Foundation
    ↓
Primitive
    ↓
Component
    ↓
Composite
    ↓
Block
    ↓
Page
    ↓
Module
```

---

# 107. Foundation

Incluye:

```text
Colors
Typography
Spacing
Grid
Radius
Elevation
Motion
Icons
Breakpoints
```

---

# 108. Primitive Components

Ejemplos:

```text
Button
IconButton
Input
Select
Checkbox
Switch
Slider
Badge
Icon
Text
Divider
Spinner
```

---

# 109. Components

Ejemplos:

```text
Card
Dialog
Toast
Dropdown
Tabs
Table
Chart
Progress
StatusIndicator
EntityControl
```

---

# 110. Composite Components

Ejemplos:

```text
EntityCard
DeviceCard
ZoneCard
AutomationCard
SceneCard
EnergyCard
AlertCard
SensorCard
ActuatorCard
```

---

# 111. Blocks

Ejemplos:

```text
SystemStatusBlock
WeatherBlock
EnergyBlock
SecurityBlock
LightingBlock
AutomationBlock
DeviceStatusBlock
HistoryBlock
```

---

# 112. Pages

Ejemplos:

```text
DashboardPage
ZonePage
DevicePage
EntityPage
AutomationPage
ScenePage
HistoryPage
IntegrationPage
SystemPage
```

---

# 113. Modules

Ejemplos:

```text
Core
Lighting
Climate
Energy
Security
Irrigation
Weather
Camera
AI
Industrial
Marine
Agriculture
```

---

# 114. Registro de módulos

Cada módulo debe poder declarar:

```json
{
  "id": "energy",
  "name": "Energía",
  "version": "1.0.0",
  "enabled": true,
  "permissions": [],
  "navigation": [],
  "blocks": [],
  "entities": [],
  "services": []
}
```

---

# 115. Activación de módulos

Un módulo puede estar:

```text
INSTALLED
ENABLED
DISABLED
UPDATING
ERROR
```

---

# 116. Dependencias

Un módulo puede depender de otros.

Ejemplo:

```text
AI
 ↓
Core
Camera
Storage
```

Si una dependencia no está disponible:

```text
AI
Estado: No disponible

Motivo:
Falta módulo Camera
```

---

# 117. Desinstalación

Antes de eliminar un módulo se debe mostrar:

```text
Entidades afectadas
Automatizaciones afectadas
Integraciones afectadas
Datos históricos
Configuraciones
```

Nunca eliminar silenciosamente datos importantes.

---

# 118. Diseño para crecimiento

El Design System debe permitir incorporar funcionalidades sin rediseñar toda la aplicación.

Por ejemplo:

```text
v1
├── Luz
├── Sensor
└── Automatización

v2
├── Energía
├── Agua
└── Seguridad

v3
├── IA
├── Cámaras
└── Predicción
```

---

# 119. Diseño para diferentes sectores

La plataforma no debe estar visualmente limitada a hogares.

Debe poder representar:

### Hogar

```text
Living
Cocina
Dormitorio
Garaje
```

### Oficina

```text
Piso 1
Sala de reuniones
Recepción
Servidor
```

### Industria

```text
Línea 1
Motor 1
Bomba 2
PLC
```

### Agricultura

```text
Lote 1
Invernadero
Riego
Estación meteorológica
```

### Marina

```text
Motor
Batería
Tanque
Bombas
Navegación
```

---

# 120. Evitar lenguaje específico de un sector

Los componentes deben ser genéricos.

Por ejemplo:

```text
Actuator
```

puede representar:

```text
Luz
Bomba
Válvula
Motor
Relé
Ventilador
```

---

# 121. Diseño orientado a capacidades

La UI debe representar capacidades.

Ejemplo:

```text
Capabilities

on_off
brightness
temperature
humidity
pressure
position
speed
power
energy
current
voltage
flow
motion
occupancy
```

Esto permite reutilizar componentes.

---

# 122. Entity Renderer

La interfaz puede utilizar un sistema conceptual:

```text
Entity
    ↓
Domain
    ↓
Device Class
    ↓
Capabilities
    ↓
Renderer
```

Ejemplo:

```text
light
+
brightness
+
color
```

genera un control de iluminación avanzado.

---

# 123. Entidades virtuales

Las entidades virtuales deben poder utilizar exactamente los mismos componentes.

Ejemplo:

```text
sensor.temperature.average
```

puede visualizarse igual que un sensor físico.

---

# 124. Entidades calculadas

Ejemplo:

```text
Consumo total
```

puede calcularse a partir de:

```text
Meter 1
+
Meter 2
+
Meter 3
```

La UI no debe diferenciar innecesariamente si el valor es físico o calculado.

---

# 125. Entidades externas

Una entidad proveniente de:

```text
Matter
MQTT
Home Assistant
API
Cloud
```

debe poder representarse utilizando el mismo sistema.

---

# 126. IA

Las funcionalidades de IA deben seguir el mismo modelo.

Ejemplo:

```text
camera.garage
```

puede generar:

```text
person_detected
vehicle_detected
confidence
```

La interfaz puede mostrar:

```text
Persona detectada
Confianza: 96 %
```

---

# 127. Cámaras

Las cámaras deben disponer de:

```text
Live View
Snapshots
Eventos
Detecciones
Grabaciones
Configuración
```

La visualización de vídeo debe ser opcional y modular.

---

# 128. Energía

El módulo de energía puede incluir:

```text
Potencia actual
Energía acumulada
Tensión
Corriente
Costo
Producción
Consumo
Comparativas
```

---

# 129. Agua

El módulo de agua puede incluir:

```text
Caudal
Volumen
Presión
Consumo
Fugas
Válvulas
Bombas
```

---

# 130. Seguridad

El módulo de seguridad puede incluir:

```text
Alarm
Door
Window
Motion
Presence
Smoke
Water Leak
Tamper
```

Debe utilizar una presentación más destacada para eventos críticos.

---

# 131. Modo emergencia

En emergencia:

* reducir información secundaria;
* destacar alarmas;
* mostrar estado de seguridad;
* facilitar acciones críticas;
* evitar acciones accidentales.

---

# 132. Animaciones

Las animaciones deben ser:

* cortas;
* funcionales;
* discretas.

Evitar animaciones permanentes.

No utilizar animaciones como única forma de comunicar estados.

---

# 133. Rendimiento

La interfaz debe:

* cargar rápido;
* evitar renderizados innecesarios;
* utilizar WebSocket para tiempo real;
* paginar listas grandes;
* virtualizar tablas grandes cuando sea necesario;
* evitar cargar historial completo;
* cargar módulos bajo demanda.

---

# 134. Lazy Loading

Los módulos secundarios deben poder cargarse bajo demanda.

Ejemplo:

```text
Usuario abre Energía
        ↓
Carga módulo Energy
```

No cargar todo el sistema al iniciar.

---

# 135. Persistencia de preferencias

El sistema puede almacenar:

```text
Dashboard elegido
Tema
Idioma
Unidades
Orden de bloques
Filtros
Preferencias de navegación
```

Las preferencias deben estar asociadas al usuario cuando corresponda.

---

# 136. Configuración por dispositivo

Un panel táctil puede tener:

```text
Dashboard:
Casa

Bloques:
Clima
Luces
Puerta
Seguridad
```

Mientras un PC puede tener:

```text
Dashboard:
Administración

Bloques:
Sistema
Dispositivos
Logs
Energía
Automatizaciones
```

---

# 137. Multiusuario

La UI debe soportar diferentes usuarios simultáneamente.

Los cambios deben registrar:

```text
Usuario
Fecha
Origen
Acción
Resultado
```

---

# 138. Auditoría

Las acciones administrativas deben poder registrarse.

Ejemplo:

```text
05/10/2026 12:45

Usuario:
admin

Acción:
Modificó light.living

Resultado:
SUCCESS
```

---

# 139. Responsive Navigation

En desktop:

```text
Sidebar
```

En tablet:

```text
Collapsed Sidebar
```

En móvil:

```text
Bottom Navigation
```

---

# 140. Breadcrumbs

Para estructuras profundas:

```text
Casa
/
Living
/
Dispositivos
/
ESP32 Living
```

---

# 141. Búsqueda global

La plataforma debería ofrecer una búsqueda global.

Ejemplo:

```text
Buscar "temperatura"
```

Resultados:

```text
Sensor temperatura Living
Automatización temperatura alta
Historial temperatura
Configuración temperatura
```

---

# 142. Command Palette

Opcionalmente:

```text
Ctrl + K
```

para ejecutar acciones:

```text
Ir a Living
Encender luz
Abrir garaje
Ejecutar escena Noche
Buscar dispositivo
```

Las acciones deben respetar permisos.

---

# 143. Acciones rápidas

El Dashboard puede mostrar:

```text
Acciones rápidas

[Luces]
[Escena Noche]
[Garaje]
[Seguridad]
```

---

# 144. Favoritos

El usuario puede marcar:

```text
Entidades
Zonas
Escenas
Automatizaciones
Dashboards
```

como favoritos.

---

# 145. Personalización

El usuario puede personalizar:

* dashboard;
* orden;
* bloques;
* favoritos;
* tema;
* unidades;
* idioma.

Pero no debería poder romper la arquitectura.

---

# 146. Configuración avanzada separada

Las opciones que puedan comprometer el sistema deben estar separadas.

Ejemplo:

```text
Configuración
├── General
├── Red
├── Dispositivos
├── Entidades
├── Automatizaciones
├── Integraciones
└── Avanzado
```

---

# 147. Información contextual

Los campos complejos deben incluir ayuda.

Ejemplo:

```text
Watchdog timeout

[30 s]

ⓘ Tiempo máximo permitido antes
de reiniciar el dispositivo.
```

---

# 148. Tooltips

Usar Tooltips para:

* iconos;
* conceptos técnicos;
* información secundaria.

No utilizar Tooltip como único medio para información crítica.

---

# 149. Confirmación de cambios

Cambios importantes pueden utilizar:

```text
Guardar
Cancelar
Restaurar
```

Cuando sea necesario:

```text
Aplicar y reiniciar
```

---

# 150. Restaurar valores

Los formularios deben permitir:

```text
Restaurar valor anterior
```

y cuando corresponda:

```text
Restaurar valores predeterminados
```

---

# 151. Factory Reset

El Factory Reset debe estar protegido.

Flujo recomendado:

```text
Configuración
 ↓
Restablecer dispositivo
 ↓
Advertencia
 ↓
Confirmación
 ↓
Confirmación adicional
 ↓
Factory Reset
```

---

# 152. Diseño de errores

Los mensajes deben indicar:

```text
Qué ocurrió
Por qué ocurrió cuando sea posible
Qué puede hacer el usuario
```

Ejemplo incorrecto:

```text
Error 503
```

Ejemplo correcto:

```text
No se pudo conectar con el dispositivo.

El nodo ESP32-Living no está disponible.

[Reintentar]
[Ver diagnóstico]
```

---

# 153. Errores técnicos

Los usuarios técnicos pueden ver:

```text
Error Code
HTTP Status
Request ID
Command ID
Node ID
Stack / module
```

---

# 154. APIs y UI

Toda acción de la UI que modifique el sistema debe utilizar la misma API definida para clientes externos cuando sea apropiado.

Evitar:

```text
UI → acceso directo a hardware
```

Preferir:

```text
UI
 ↓
API
 ↓
Data Model
 ↓
System Bus
 ↓
Device
```

Esto garantiza consistencia.

---

# 155. Arquitectura general

```text
┌──────────────────────────────┐
│            UI                │
├──────────────────────────────┤
│       API / WebSocket        │
├──────────────────────────────┤
│ Authentication / Authorization│
├──────────────────────────────┤
│         Data Model           │
├──────────────────────────────┤
│         System Bus           │
├──────────────────────────────┤
│ Central / Zone / Node        │
├──────────────────────────────┤
│ Hardware / Resources        │
└──────────────────────────────┘
```

---

# 156. Regla de independencia

La interfaz nunca debe asumir:

```text
"un dispositivo = un ESP32"
```

Puede existir:

```text
1 ESP32 → múltiples entidades
```

o:

```text
1 entidad → recursos distribuidos
```

---

# 157. Regla de independencia física

Una entidad puede cambiar de:

```text
GPIO
```

a:

```text
MCP23017
```

o:

```text
RS485
```

o:

```text
CAN
```

sin cambiar necesariamente:

```text
entity_id
```

ni su representación visual.

---

# 158. Regla de estabilidad

Los IDs lógicos deben ser más estables que el hardware.

Ejemplo:

```text
light.living
```

debe poder continuar existiendo aunque se cambie:

```text
ESP32-A
```

por:

```text
ESP32-B
```

---

# 159. Estados distribuidos

La UI debe poder representar:

```text
Central
   ↓
Zone Controller
   ↓
Node
   ↓
Entity
```

pero el usuario normal debe poder ver simplemente:

```text
Luz Living
ON
```

---

# 160. Transparencia progresiva

El sistema debe utilizar el concepto:

> Simple por defecto, detallado cuando sea necesario.

Nivel 1:

```text
Luz Living
ON
```

Nivel 2:

```text
Luz Living
Node: ESP32 Living
```

Nivel 3:

```text
GPIO 27
Relay
Active High
```

Nivel 4:

```text
GPIO register
task
timing
diagnostic
```

---

# 161. Modo experto

Opcionalmente:

```text
Modo experto
```

habilita:

* parámetros avanzados;
* hardware;
* buses;
* logs;
* debugging;
* API;
* System Bus;
* diagnósticos.

---

# 162. Principio de seguridad

La UI nunca debe facilitar accidentalmente una operación peligrosa.

Para operaciones críticas:

```text
Advertencia
Confirmación
Permiso
Auditoría
```

---

# 163. Principio de consistencia

Un mismo concepto debe utilizar siempre:

* mismo nombre;
* mismo icono;
* mismo componente;
* mismo comportamiento;
* mismo estado visual.

Ejemplo:

Si `Unavailable` significa que una entidad no está disponible, debe significar lo mismo en:

```text
Dashboard
Zona
Dispositivo
API
Historial
Integraciones
```

---

# 164. Nomenclatura

Preferir nombres consistentes:

```text
Device
Resource
Capability
Entity
State
Command
Event
Zone
Group
Scene
Automation
Integration
```

No crear sinónimos innecesarios.

---

# 165. Componentes reutilizables

Antes de crear un componente nuevo:

1. verificar si existe uno equivalente;
2. extenderlo si corresponde;
3. mantener compatibilidad;
4. evitar duplicados.

---

# 166. Design Tokens

Todos los valores visuales importantes deben estar centralizados.

Ejemplo:

```text
Color Tokens
Typography Tokens
Spacing Tokens
Radius Tokens
Shadow Tokens
Motion Tokens
Breakpoint Tokens
```

---

# 167. CSS Variables

Preferir:

```css
var(--color-primary)
```

en lugar de:

```css
#123456
```

directamente en los componentes.

---

# 168. Component API

Los componentes deben tener interfaces previsibles.

Ejemplo conceptual:

```text
EntityCard(
    entity,
    state,
    capabilities,
    actions
)
```

No deben acceder directamente a APIs globales.

---

# 169. Separación de responsabilidades

Un componente visual no debe encargarse de:

* autenticación;
* comunicación con hardware;
* persistencia;
* reglas de negocio;
* transporte.

Debe recibir datos y emitir acciones.

---

# 170. Estado global

El frontend puede utilizar un store global para:

```text
User
System
Entities
Devices
Zones
Events
Connection
```

Pero debe evitarse almacenar innecesariamente grandes cantidades de histórico.

---

# 171. Cache

La UI puede almacenar temporalmente:

* entidades;
* configuraciones;
* preferencias;
* últimos estados.

No debe considerar el cache como autoridad absoluta.

La fuente de verdad depende del tipo de dato.

---

# 172. Reconciliación

Cuando la conexión vuelve:

```text
Cache
 ↓
Servidor
 ↓
Comparación
 ↓
Actualización
```

La UI debe reflejar el estado real.

---

# 173. Reconexión

WebSocket:

```text
Connected
 ↓
Disconnected
 ↓
Reconnecting
 ↓
Connected
 ↓
Resync
```

Debe existir backoff progresivo.

---

# 174. Eventos perdidos

Cuando sea posible:

```text
last_event_id
```

permite recuperar eventos.

Si no es posible:

```text
Full state resync
```

---

# 175. Performance de listas

Listas grandes deben utilizar:

```text
Pagination
Infinite Scroll
Virtualization
Filtering
Search
```

según el caso.

---

# 176. Tablas

Las tablas deben reservarse para información técnica o administrativa.

Ejemplo:

```text
Device | Zone | Status | IP | Firmware
```

En interfaces de usuario general se prefieren Cards.

---

# 177. Formularios dinámicos

Los formularios pueden construirse según:

```text
Capability
Schema
Device type
Module
Permissions
```

Esto permite agregar nuevos dispositivos sin diseñar manualmente cada pantalla.

---

# 178. Schema Driven UI

El sistema puede utilizar esquemas para describir formularios.

Ejemplo conceptual:

```json
{
  "type": "number",
  "label": "Temperatura máxima",
  "min": 0,
  "max": 100,
  "unit": "°C"
}
```

La UI genera el control correspondiente.

---

# 179. Extensibilidad

Los módulos pueden agregar:

```text
Pages
Blocks
Components
Entities
Capabilities
Actions
Automations
Integrations
```

sin modificar el Core siempre que respeten los contratos establecidos.

---

# 180. Plugins

La plataforma puede soportar plugins.

Un plugin debe declarar:

```text
ID
Name
Version
Dependencies
Permissions
Entities
Capabilities
UI modules
API endpoints
```

---

# 181. Compatibilidad

Los módulos deben declarar compatibilidad:

```text
Platform version
API version
Schema version
```

---

# 182. Versionado

El Design System utiliza:

```text
MAJOR.MINOR.PATCH
```

Ejemplo:

```text
2.0.0
```

### MAJOR

Cambios incompatibles.

### MINOR

Nuevos componentes o funcionalidades compatibles.

### PATCH

Correcciones.

---

# 183. Compatibilidad hacia atrás

Los módulos existentes deben continuar funcionando cuando sea posible.

Cambios incompatibles requieren:

* migración;
* adaptación;
* nueva versión;
* documentación.

---

# 184. Documentación de componentes

Cada componente importante debe documentar:

```text
Nombre
Propósito
Props
Estados
Eventos
Variantes
Accesibilidad
Ejemplos
Dependencias
```

---

# 185. Component States

Cada componente interactivo debe definir:

```text
Default
Hover
Focus
Active
Disabled
Loading
Error
Success
```

cuando corresponda.

---

# 186. Testing visual

Los componentes principales deben poder probarse independientemente.

Preferencias:

```text
Component tests
Integration tests
Accessibility tests
Responsive tests
Visual regression
```

---

# 187. Testing funcional

Verificar:

```text
API
WebSocket
Commands
States
Permissions
Offline
Reconnect
```

---

# 188. Testing en hardware

La interfaz debe probarse como mínimo en:

```text
ESP32
ESP32-S3
Desktop
Mobile
Tablet
Touch panel
```

cuando el módulo lo requiera.

---

# 189. Compatibilidad de navegadores

Prioridad:

```text
Chrome / Chromium
Edge
Firefox
Safari
Android Browser
iOS Safari
```

Las funciones críticas no deben depender de APIs experimentales.

---

# 190. PWA

La interfaz puede evolucionar hacia una PWA.

Funciones potenciales:

```text
Install
Offline cache
Push notifications
App-like UI
Local network access
```

La PWA nunca debe convertirse en dependencia obligatoria del sistema.

---

# 191. Instalación local

Un usuario debe poder acceder mediante:

```text
http://device.local
```

o:

```text
http://central.local
```

cuando mDNS esté disponible.

También:

```text
IP local
```

---

# 192. Acceso sin Central

Un nodo debe poder exponer una interfaz local mínima cuando sea necesario.

Ejemplo:

```text
ESP32
 ├── Estado
 ├── Configuración
 ├── Diagnóstico
 └── Control local
```

---

# 193. Acceso mediante Central

Cuando exista Central:

```text
Usuario
 ↓
Central UI
 ↓
API
 ↓
System Bus
 ↓
Node
```

La interfaz debe mantener la misma semántica.

---

# 194. Principio de degradación

Si una funcionalidad no está disponible:

```text
Mostrar lo que sí funciona.
```

Ejemplo:

```text
Internet:
OFFLINE

Local:
ONLINE

Control de luces:
DISPONIBLE

Cloud:
NO DISPONIBLE
```

---

# 195. Integración con terceros

Las integraciones no deben modificar el lenguaje visual interno.

Ejemplo:

```text
Home Assistant
Matter
MQTT
Alexa
Google
Apple Home
Homey
SmartThings
```

son adaptadores.

La UI continúa utilizando:

```text
Entity
State
Command
Event
```

---

# 196. Diseño para API externa

Todo lo que pueda hacerse desde UI debería utilizar una operación equivalente en API cuando sea razonable.

Ejemplo:

```text
UI:
[Encender luz]

API:
POST /api/v1/entities/light.living/commands
```

---

# 197. Diseño para automatización

La interfaz debe ser capaz de representar cualquier automatización válida del modelo.

No crear un editor que solo soporte casos domésticos simples si el modelo permite:

```text
threshold
schedule
event
device event
system event
webhook
manual
```

---

# 198. Diseño para industria

La misma UI puede representar:

```text
Motor
Velocidad
Temperatura
Presión
Corriente
Estado
Alarmas
```

sin requerir una interfaz completamente distinta.

---

# 199. Diseño para agricultura

Puede representar:

```text
Temperatura
Humedad
Radiación
Viento
Lluvia
Humedad de suelo
Riego
Bombas
Válvulas
```

---

# 200. Diseño para estaciones meteorológicas

Una estación SEMA puede utilizar:

```text
Dashboard
 ├── Temperatura
 ├── Humedad
 ├── Presión
 ├── Viento
 ├── Lluvia
 ├── Radiación
 └── Historial
```

sin requerir un frontend completamente separado.

---

# 201. Diseño para IA

Las funciones de IA deben utilizar componentes estándar siempre que sea posible.

Ejemplo:

```text
Detección
Confianza
Estado
Predicción
Recomendación
```

---

# 202. Predicciones

Las predicciones deben diferenciarse visualmente de mediciones reales.

Ejemplo:

```text
Temperatura actual
23.4 °C

Predicción
25.1 °C
```

Nunca presentar una predicción como medición real.

---

# 203. Recomendaciones de IA

Ejemplo:

```text
💡 Recomendación

Reducir climatización durante
los próximos 30 minutos.

Confianza:
91 %
```

Debe indicarse claramente que es una recomendación.

---

# 204. Diseño de confianza

Cuando el sistema sugiera una acción:

```text
Recomendación
```

y no:

```text
Acción ejecutada
```

hasta que realmente se ejecute.

---

# 205. Seguridad funcional

Las acciones relacionadas con:

* motores;
* bombas;
* válvulas;
* calefacción;
* energía;
* alarmas;

deben respetar las limitaciones definidas por el sistema.

La UI no debe permitir saltarse límites de seguridad.

---

# 206. Estados de seguridad

Ejemplo:

```text
Normal
Warning
Alarm
Emergency
Locked
```

---

# 207. Prioridades

Cuando una acción sea rechazada por prioridad:

```text
No se ejecutó.

Existe una condición de seguridad
de mayor prioridad.
```

No mostrar simplemente:

```text
Error
```

---

# 208. Auditabilidad

Toda acción importante debe poder rastrearse mediante:

```text
Request ID
Command ID
Event ID
User
Source
Timestamp
```

---

# 209. Correlation ID

Cuando una operación atraviesa:

```text
UI
 ↓
API
 ↓
Central
 ↓
Zone
 ↓
Node
```

debe conservarse el contexto de correlación.

---

# 210. Diseño de estados distribuidos

Ejemplo:

```text
Usuario
   ↓
Command
   ↓
Central
   ↓
Zone
   ↓
Node
   ↓
Hardware
   ↓
State Changed
   ↓
Event
   ↓
UI
```

La interfaz debe reflejar este ciclo cuando sea necesario.

---

# 211. No bloquear la UI

Una operación distribuida no debe congelar toda la interfaz esperando respuesta física.

Preferir:

```text
Command accepted
```

y después:

```text
State updated
```

---

# 212. Operaciones largas

Para:

* firmware;
* sincronización;
* escaneo;
* descubrimiento;
* calibración;

usar:

```text
Job
```

o una operación asíncrona equivalente.

---

# 213. Jobs

Ejemplo:

```text
Firmware update

Queued
Downloading
Installing
Rebooting
Completed
```

---

# 214. Descubrimiento

El sistema puede mostrar:

```text
Dispositivos encontrados

ESP32 Living
ESP32 Kitchen
Weather Station
```

y permitir:

```text
Adoptar
Configurar
Ignorar
```

---

# 215. Provisioning

El proceso debe ser guiado.

```text
Buscar dispositivo
      ↓
Identificar
      ↓
Adoptar
      ↓
Asignar nombre
      ↓
Asignar zona
      ↓
Configurar
      ↓
Finalizar
```

---

# 216. Wizard

Los procesos complejos pueden utilizar Wizards.

Ejemplos:

```text
Agregar dispositivo
Configurar red
Configurar sensor
Configurar integración
Actualizar firmware
```

---

# 217. Wizard seguro

Debe existir:

```text
Paso actual
Pasos restantes
Atrás
Cancelar
Continuar
```

y evitar perder información.

---

# 218. Diseño de onboarding

Una instalación nueva puede iniciar con:

```text
Bienvenido

1. Configurar red
2. Descubrir dispositivos
3. Crear zonas
4. Configurar entidades
5. Crear automatizaciones
6. Configurar integraciones
```

---

# 219. First Run

La primera ejecución debe ser sencilla.

No mostrar inicialmente:

```text
CAN
RS485
GPIO
Task
PSRAM
MQTT
```

salvo que sean necesarios.

---

# 220. Progresive Disclosure

La información técnica se muestra progresivamente:

```text
Usuario
 ↓
Opciones normales
 ↓
Opciones avanzadas
 ↓
Modo técnico
 ↓
Diagnóstico
```

---

# 221. Regla principal de UX

> **La plataforma debe ser poderosa para usuarios avanzados y sencilla para usuarios normales.**

---

# 222. Consistencia entre Web, API y Firmware

Los conceptos deben mantenerse iguales:

```text
Firmware
    ↓
Data Model
    ↓
API
    ↓
Web UI
    ↓
Integrations
```

No crear nombres diferentes para el mismo concepto.

---

# 223. Fuente de verdad

La UI nunca debe considerarse la fuente de verdad del sistema.

Dependiendo del dato:

```text
Estado físico → Node
Configuración → configuración aplicada
Estado global → Central / autoridad correspondiente
Histórico → almacenamiento correspondiente
```

La UI representa esos datos.

---

# 224. Comunicación visual

Los estados importantes deben ser comprensibles en menos de unos segundos.

Prioridad visual:

```text
CRITICAL
   ↓
ERROR
   ↓
WARNING
   ↓
ACTION
   ↓
STATE
   ↓
INFORMATION
```

---

# 225. No sobrecargar dashboards

Un Dashboard no debe intentar mostrar todo.

Debe mostrar:

```text
Lo importante
+
Lo accionable
+
Lo relevante
```

El resto debe estar disponible mediante navegación.

---

# 226. Personalización sin complejidad

La personalización debe utilizar:

```text
Agregar bloque
Eliminar bloque
Mover bloque
Configurar bloque
```

No requerir código.

---

# 227. Persistencia del layout

El layout puede guardarse:

```text
Por usuario
Por dashboard
Por dispositivo
```

según el contexto.

---

# 228. Exportación

Opcionalmente permitir:

```text
Exportar configuración
Exportar automatizaciones
Exportar dashboards
Exportar datos
```

---

# 229. Importación

La importación debe validar:

```text
Versión
Schema
Dependencias
Compatibilidad
Conflictos
```

---

# 230. Backup

La UI debe permitir administrar backups.

Ejemplo:

```text
Backup
05/10/2026
Configuración + automatizaciones

[Restaurar]
[Descargar]
```

---

# 231. Restauración

La restauración debe indicar:

```text
Qué se restaurará
Qué se sobrescribirá
Qué dispositivos serán afectados
```

---

# 232. Diseño de datos sensibles

La UI debe ocultar:

* tokens;
* contraseñas;
* claves privadas;
* secretos;
* credenciales.

Ejemplo:

```text
Token:
••••••••••••
```

---

# 233. Mostrar secretos

Cuando sea necesario:

```text
Mostrar
```

requiere permiso adecuado.

---

# 234. Red

La configuración de red debe separar:

```text
Wi-Fi
Ethernet
IP
DNS
NTP
mDNS
```

sin mezclar conceptos.

---

# 235. Comunicación

El diagnóstico puede mostrar:

```text
Wi-Fi
Ethernet
CAN
RS485
MQTT
WebSocket
System Bus
```

cada uno con su estado.

---

# 236. Estado de Central

Ejemplo:

```text
Central

ONLINE

CPU       32 %
RAM       48 %
Storage   36 %

Nodes     14
Online    13
Offline   1
```

---

# 237. Estado de zona

```text
Living

ONLINE

Devices:
5

Entities:
18

Automations:
7

Alerts:
0
```

---

# 238. Estado de nodo

```text
ESP32-Living

ONLINE

Wi-Fi
RSSI -54 dBm

CPU 31 %
RAM 42 %

Entities 8
```

---

# 239. Estado de entidad

```text
Luz Living

ON

Source:
ESP32-Living

Updated:
2 s ago
```

---

# 240. Jerarquía visual

Siempre que sea posible:

```text
Site
 ↓
Zone
 ↓
Device
 ↓
Entity
```

pero permitir acceso directo mediante búsqueda.

---

# 241. Búsqueda por ID

Los usuarios técnicos deben poder buscar:

```text
light.living
```

y encontrar directamente la entidad.

---

# 242. Deep Links

La interfaz debe soportar enlaces internos como:

```text
/entity/light.living
/device/node-01
/zone/living
/automation/01
```

Los formatos exactos pertenecen al frontend.

---

# 243. Estado persistente de navegación

El navegador puede recordar:

```text
Última página
Última zona
Últimos filtros
```

siempre que sea apropiado.

---

# 244. Modales y navegación

No utilizar un Modal cuando el usuario probablemente necesite:

* navegar;
* editar muchos campos;
* consultar información extensa.

En esos casos usar una página.

---

# 245. Formularios largos

Dividir en:

```text
General
Hardware
Comunicación
Seguridad
Avanzado
```

---

# 246. Validación

La validación debe existir:

```text
Frontend
+
API
+
Firmware cuando corresponda
```

Nunca confiar exclusivamente en el frontend.

---

# 247. Unidades y límites

Los controles deben mostrar límites.

Ejemplo:

```text
Temperatura máxima

[ 25.0 ] °C

Permitido:
-40 → 100 °C
```

---

# 248. Campos dinámicos

Cuando una opción habilita otra:

```text
Modo:
[ Modbus ]

Dirección:
[ 10 ]

Baudrate:
[ 9600 ]
```

Los campos innecesarios deben ocultarse.

---

# 249. Dependencias visuales

Si un módulo depende de otro:

```text
IA

Requiere:
✓ Camera
✓ Storage
✗ Model Runtime
```

Mostrar claramente la dependencia faltante.

---

# 250. Compatibilidad de hardware

La UI puede mostrar:

```text
Compatible
Parcialmente compatible
No compatible
```

para módulos y dispositivos.

---

# 251. Firmware y hardware

Ejemplo:

```text
ESP32-S3

Firmware:
1.5.0

Hardware:
Compatible ✓

PSRAM:
Disponible ✓

Camera:
Compatible ✓
```

---

# 252. Diseño de componentes para diferentes pantallas

Un mismo componente puede tener:

```text
Compact
Normal
Large
```

Ejemplo:

```text
EntityCard Compact
EntityCard Normal
EntityCard Large
```

---

# 253. Compact mode

Ideal para:

* móviles;
* dashboards con muchas entidades;
* paneles pequeños.

---

# 254. Large mode

Ideal para:

* paneles táctiles;
* kioscos;
* dashboards;
* salas de control.

---

# 255. Kiosk Mode

Opcionalmente:

```text
Kiosk Mode
```

puede ocultar:

* navegación;
* configuración;
* usuario;
* elementos administrativos.

Ideal para:

```text
Panel mural
Control industrial
Dashboard de recepción
```

---

# 256. Seguridad del Kiosk

El Kiosk no debe otorgar permisos adicionales.

Solo modifica la presentación.

---

# 257. Multi-tenant futuro

Aunque inicialmente no sea obligatorio, el Design System debe poder soportar:

```text
Organization
Site
Zone
User
Role
```

sin rediseñar la estructura.

---

# 258. Organizaciones

Futuro:

```text
Organización
 ├── Sitio A
 ├── Sitio B
 └── Sitio C
```

---

# 259. Instalaciones múltiples

Un usuario puede tener:

```text
Casa
Oficina
Campo
Taller
Barco
```

La interfaz debe poder cambiar entre sitios.

---

# 260. Selector de sitio

Ejemplo:

```text
Sitio:
[ Casa ▼ ]
```

---

# 261. Arquitectura final

El Design System completo sigue:

```text
                    PLATFORM
                       │
        ┌──────────────┼──────────────┐
        │              │              │
       UI             API        INTEGRATIONS
        │              │              │
     Modules        Data Model     Adapters
        │              │              │
     Blocks         System Bus      Ecosystems
        │              │
   Components       Devices
        │              │
    Foundation      Hardware
```

---

# 262. Reglas fundamentales

## Regla 1

> La UI trabaja con entidades, no con GPIO.

## Regla 2

> La interfaz debe funcionar localmente.

## Regla 3

> Internet nunca debe ser una dependencia para funciones locales.

## Regla 4

> Todo módulo debe poder habilitarse o deshabilitarse.

## Regla 5

> Las páginas deben estar compuestas por bloques reutilizables.

## Regla 6

> Los bloques deben poder configurarse cuando tenga sentido.

## Regla 7

> La complejidad técnica debe quedar oculta para usuarios normales.

## Regla 8

> El modo técnico debe permitir acceso a información avanzada.

## Regla 9

> UI, API y firmware deben utilizar el mismo modelo conceptual.

## Regla 10

> Las integraciones son adaptadores, no fuentes alternativas del modelo interno.

## Regla 11

> Estado real y estado deseado deben poder diferenciarse.

## Regla 12

> Los comandos distribuidos deben ser observables.

## Regla 13

> Los errores deben ser comprensibles y accionables.

## Regla 14

> Los IDs lógicos deben ser independientes del hardware.

## Regla 15

> La seguridad tiene prioridad sobre la comodidad.

---

# 263. Checklist para nuevos módulos

Antes de incorporar un módulo:

```text
[ ] Tiene ID único
[ ] Tiene versión
[ ] Declara dependencias
[ ] Declara permisos
[ ] Tiene navegación
[ ] Tiene páginas
[ ] Tiene bloques
[ ] Utiliza componentes existentes
[ ] Respeta Design Tokens
[ ] Soporta Light/Dark
[ ] Es responsive
[ ] Tiene estados de loading/error
[ ] Soporta offline cuando corresponde
[ ] Respeta autorización
[ ] No depende de Internet innecesariamente
[ ] Utiliza Data Model
[ ] Utiliza API
[ ] No accede directamente al hardware
[ ] Tiene documentación
[ ] Tiene pruebas
```

---

# 264. Checklist para nuevos componentes

```text
[ ] Propósito definido
[ ] Variantes definidas
[ ] Estados definidos
[ ] Accesibilidad
[ ] Responsive
[ ] Dark mode
[ ] Loading
[ ] Error
[ ] Disabled
[ ] Documentación
[ ] Tests
```

---

# 265. Checklist para nuevas páginas

```text
[ ] Pertenece a un módulo
[ ] Utiliza Shell
[ ] Tiene navegación
[ ] Tiene estados de loading
[ ] Tiene empty state
[ ] Tiene error state
[ ] Respeta permisos
[ ] Responsive
[ ] Offline behavior definido
[ ] No depende directamente del hardware
[ ] Utiliza API
```

---

# 266. Checklist para nuevas funcionalidades

```text
[ ] Data Model definido
[ ] Data Schema definido
[ ] API definida
[ ] System Bus definido si corresponde
[ ] UI definida
[ ] Permisos definidos
[ ] Eventos definidos
[ ] Estados definidos
[ ] Errores definidos
[ ] Integraciones consideradas
[ ] Offline behavior definido
```

---

# 267. Flujo de desarrollo recomendado

Toda nueva funcionalidad debería seguir:

```text
Necesidad
   ↓
Data Model
   ↓
Data Schema
   ↓
System Bus
   ↓
API
   ↓
Permissions
   ↓
UI Components
   ↓
UI Blocks
   ↓
Page / Module
   ↓
Integration
   ↓
Testing
```

---

# 268. Relación con la documentación del proyecto

Este documento debe mantenerse alineado con:

```text
ARCHITECTURE.md
DEVICE-MODEL.md
COMMUNICATION.md
MODULE-DEVELOPMENT.md
CENTRAL-ARCHITECTURE.md
FREERTOS-TASK-ARCHITECTURE.md
HARDWARE-REFERENCE-NODES.md
SMART-HOME-INTEGRATION.md
API-THIRD-PARTY.md
API-AUTHENTICATION-AUTHORIZATION.md
DATA-MODEL.md
DATA-SCHEMAS.md
SYSTEM-BUS.md
API-SPECIFICATION.md
```

Los documentos deben formar un único sistema conceptual.

---

# 269. Fuente de verdad de cada aspecto

```text
Arquitectura
→ ARCHITECTURE.md

Dispositivos
→ DEVICE-MODEL.md

Comunicación
→ COMMUNICATION.md

Módulos
→ MODULE-DEVELOPMENT.md

Central
→ CENTRAL-ARCHITECTURE.md

FreeRTOS
→ FREERTOS-TASK-ARCHITECTURE.md

Hardware
→ HARDWARE-REFERENCE-NODES.md

Integraciones
→ SMART-HOME-INTEGRATION.md

API externa
→ API-THIRD-PARTY.md

Autenticación
→ API-AUTHENTICATION-AUTHORIZATION.md

Modelo de datos
→ DATA-MODEL.md

Schemas
→ DATA-SCHEMAS.md

Bus
→ SYSTEM-BUS.md

API
→ API-SPECIFICATION.md

Interfaz
→ DESIGN-SYSTEM.md
```

---

# 270. Regla de evolución

El Design System debe evolucionar junto con la plataforma.

No se debe agregar una funcionalidad simplemente creando:

```text
una nueva página
```

sin evaluar:

```text
Data Model
Schema
API
Permissions
System Bus
Module
Component
Block
Integration
```

---

# 271. Objetivo final

La plataforma debe sentirse como **un único sistema**, independientemente de cuántos:

* ESP32;
* sensores;
* actuadores;
* zonas;
* centrales;
* protocolos;
* integraciones;
* módulos;
* automatizaciones;

existan.

El usuario debería percibir:

```text
UNA PLATAFORMA
```

y no:

```text
muchos ESP32 independientes
```

---

# 272. Principio final

> **El hardware puede ser distribuido, la comunicación puede ser distribuida, la inteligencia puede ser distribuida y la ejecución puede ser distribuida; pero la experiencia del usuario debe sentirse unificada.**

La interfaz debe ocultar la complejidad cuando no es necesaria y revelar la información técnica cuando el usuario la necesita.

```text
                   USER
                    │
                    ▼
              ┌───────────┐
              │    UI     │
              └─────┬─────┘
                    │
                    ▼
              ┌───────────┐
              │    API    │
              └─────┬─────┘
                    │
                    ▼
             ┌──────────────┐
             │  DATA MODEL  │
             └──────┬───────┘
                    │
                    ▼
             ┌──────────────┐
             │ SYSTEM BUS   │
             └──────┬───────┘
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       CENTRAL     ZONE      NODE
          │         │         │
          └─────────┼─────────┘
                    ▼
                 HARDWARE
```

**Una arquitectura distribuida no debe producir una experiencia distribuida.**

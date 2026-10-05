# API Authentication & Authorization

> **Estado:** Planificación
> **Versión:** 1.0.0
> **Tipo:** Arquitectura / Seguridad / Especificación
> **Proyecto:** Distributed Automation Platform
> **Última actualización:** 2026-10-05

---

# 1. Objetivo

Este documento define el sistema de:

* autenticación;
* autorización;
* usuarios;
* aplicaciones;
* dispositivos;
* API Keys;
* Access Tokens;
* Refresh Tokens;
* scopes;
* roles;
* permisos;
* permisos por zona;
* permisos por entidad;
* permisos por operación;
* expiración;
* revocación;
* auditoría;
* sesiones;
* acceso local;
* acceso remoto;
* integración con terceros.

El objetivo es que la plataforma pueda utilizarse tanto en:

```text
Casa
Oficina
Edificio
Agricultura
Industria
Marine
SCADA
BMS
IoT
```

sin sacrificar seguridad.

---

# 2. Principio fundamental

La plataforma debe diferenciar claramente:

```text
AUTHENTICATION
¿Quién eres?

AUTHORIZATION
¿Qué puedes hacer?

SCOPE
¿Sobre qué tipo de recursos?

RESOURCE
¿Sobre qué recurso concreto?

POLICY
¿Está permitido hacerlo en este contexto?
```

Por ejemplo:

```text
Usuario:
    Alessandro

Autenticado:
    Sí

Permiso:
    entities:write

Recurso:
    light.living

Zona:
    living

Política:
    permitido

Resultado:
    COMMAND ACCEPTED
```

---

# 3. Modelo de seguridad

La arquitectura será:

```text
┌──────────────────────────┐
│        CLIENT            │
│                          │
│ Web / Mobile / API / IoT │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│     AUTHENTICATION       │
│                          │
│ Password                 │
│ API Key                  │
│ Token                    │
│ Certificate              │
│ OAuth/OIDC               │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│      IDENTITY            │
│                          │
│ User                     │
│ Application              │
│ Device                   │
│ Service                  │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│     AUTHORIZATION        │
│                          │
│ Roles                    │
│ Scopes                   │
│ Permissions              │
│ Zones                    │
│ Entities                 │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│        POLICY            │
│                          │
│ Safety                   │
│ Security                 │
│ Priority                 │
│ Rate limit               │
│ Context                  │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│         RESOURCE         │
└──────────────────────────┘
```

---

# 4. Identidades

El sistema contempla diferentes tipos de identidad.

```text
User
Application
Device
Service
Integration
Plugin
```

No deben tratarse como equivalentes.

---

# 5. User

Un `User` representa una persona.

Ejemplo:

```json
{
  "id": "usr_001",
  "username": "alessandro",
  "display_name": "Alessandro",
  "status": "active"
}
```

Un usuario puede tener:

```text
Roles
Permissions
Zones
Sessions
Security policies
```

---

# 6. Application

Una `Application` representa un software externo.

Ejemplos:

```text
Energy Manager
Mobile App
SCADA
ERP
Home Automation
Weather Service
```

Ejemplo:

```json
{
  "id": "app_001",
  "name": "Energy Manager",
  "type": "third_party"
}
```

Una aplicación no debe autenticarse como si fuera una persona.

---

# 7. Device Identity

Un dispositivo físico puede disponer de una identidad propia.

Ejemplo:

```json
{
  "id": "dev_001",
  "type": "device",
  "model": "ESP32-Node",
  "status": "trusted"
}
```

Esto permite autenticación entre:

```text
Central
Zone Controller
ESP32 Node
Gateway
```

---

# 8. Service Identity

Un servicio interno también puede tener identidad.

Ejemplo:

```text
Automation Engine
Event Service
History Service
Integration Service
```

Esto permite aplicar seguridad incluso entre componentes internos.

---

# 9. Integration Identity

Una integración externa tendrá una identidad propia.

Ejemplo:

```text
Matter Integration
MQTT Integration
Home Assistant Integration
Weather Integration
```

Esto permite revocar una integración sin afectar a los usuarios.

---

# 10. Authentication

La autenticación responde:

> ¿Quién eres?

Métodos iniciales:

```text
Username + Password
API Key
Bearer Token
Device Credential
```

Métodos futuros:

```text
OAuth 2.0
OpenID Connect
Passkeys / WebAuthn
Client Certificates
```

---

# 11. Authorization

La autorización responde:

> ¿Puedes realizar esta acción?

Ejemplo:

```text
Usuario autenticado
        ↓
entities:write
        ↓
light.living
        ↓
permitido
```

---

# 12. Authentication ≠ Authorization

Nunca se debe asumir:

```text
authenticated = full access
```

Un usuario puede estar correctamente autenticado y no tener permiso para:

```text
Modificar configuración
Controlar seguridad
Administrar usuarios
Modificar firmware
```

---

# 13. TLS

Toda comunicación autenticada deberá utilizar:

```text
HTTPS
WSS
```

cuando la comunicación atraviese una red no confiable.

Nunca se deben enviar:

```text
password
API key
access token
refresh token
```

mediante HTTP sin cifrado en redes no confiables.

---

# 14. HTTP local

En instalaciones pequeñas podrá existir:

```text
HTTP
```

para operaciones iniciales de provisioning, cuando el entorno lo permita.

Sin embargo, el acceso autenticado normal debe evolucionar a:

```text
HTTPS
```

incluso en redes locales.

---

# 15. Usuario administrador inicial

Durante el primer arranque debe crearse un usuario administrador.

Ejemplo:

```text
username:
    admin

password:
    definido durante provisioning
```

No se deben utilizar contraseñas administrativas universales.

Nunca:

```text
admin / admin
```

como credencial permanente.

---

# 16. Password Policy

Las contraseñas deberán cumplir una política configurable.

Mínimos recomendados:

```text
longitud mínima
```

y protección contra:

```text
contraseñas comunes
credenciales filtradas
credenciales triviales
```

La plataforma debe priorizar longitud sobre reglas artificiales excesivas.

---

# 17. Password Storage

Las contraseñas nunca se almacenarán en texto plano.

Debe utilizarse un algoritmo de hashing resistente.

Preferentemente:

```text
Argon2id
```

Como alternativa cuando las limitaciones del dispositivo lo requieran:

```text
scrypt
PBKDF2
```

Nunca:

```text
MD5
SHA-1
SHA-256(password)
```

como mecanismo único de almacenamiento de contraseñas.

---

# 18. Salt

Cada contraseña debe utilizar un salt único.

Conceptualmente:

```text
password
    +
random salt
    ↓
password hash
```

Nunca debe utilizarse un salt global para todos los usuarios.

---

# 19. Password Change

Un usuario autorizado podrá cambiar su contraseña.

```http
POST /api/v1/auth/password/change
```

El sistema deberá exigir:

```text
current password
new password
```

salvo procesos administrativos especiales de recuperación.

---

# 20. Login

Endpoint conceptual:

```http
POST /api/v1/auth/login
```

Request:

```json
{
  "username": "alessandro",
  "password": "********"
}
```

Respuesta:

```json
{
  "access_token": "...",
  "token_type": "Bearer",
  "expires_in": 900
}
```

---

# 21. Access Token

El `Access Token` representa una sesión o autorización temporal.

Ejemplo:

```http
Authorization: Bearer <ACCESS_TOKEN>
```

Debe tener:

```text
expiration
identity
scopes
issuer
audience
```

---

# 22. Access Token Lifetime

Los Access Tokens deben tener duración limitada.

Ejemplo recomendado:

```text
15 minutos
```

El valor debe ser configurable.

Para dispositivos embebidos se podrá utilizar una duración diferente cuando sea necesario.

---

# 23. Refresh Token

Cuando sea apropiado, una aplicación puede recibir un Refresh Token.

Flujo:

```text
Login
   ↓
Access Token
   +
Refresh Token
```

Cuando expira el Access Token:

```text
Refresh Token
   ↓
New Access Token
```

---

# 24. Refresh Token Rotation

Los Refresh Tokens deberán poder rotarse.

Ejemplo:

```text
Refresh Token A
      ↓
Refresh
      ↓
Refresh Token B
```

El token anterior queda invalidado.

Esto reduce el impacto de robo de credenciales.

---

# 25. Revocación

El sistema debe permitir revocar:

```text
Access Token
Refresh Token
API Key
Application
Session
Device Credential
```

La revocación debe ser inmediata o lo más cercana posible a inmediata.

---

# 26. Logout

Endpoint:

```http
POST /api/v1/auth/logout
```

Debe invalidar la sesión correspondiente.

---

# 27. Logout All

Un usuario podrá cerrar todas sus sesiones.

```http
POST /api/v1/auth/logout-all
```

Útil cuando:

* se perdió un dispositivo;
* se sospecha de una intrusión;
* se cambió una contraseña;
* se desea revocar accesos anteriores.

---

# 28. Sessions

Cada login debe generar una sesión identificable.

Ejemplo:

```json
{
  "session_id": "sess_001",
  "user_id": "usr_001",
  "created_at": "2026-10-05T20:00:00Z",
  "last_activity": "2026-10-05T20:15:00Z",
  "client": {
    "type": "web"
  }
}
```

---

# 29. Gestión de sesiones

El usuario podrá visualizar:

```text
Current Session
Chrome - Windows
Local Network

Mobile
Android

Unknown Device
```

y revocar sesiones individuales.

---

# 30. API Keys

Las API Keys estarán destinadas principalmente a:

```text
servers
scripts
integrations
IoT applications
automation systems
```

Ejemplo:

```text
ak_live_xxxxxxxxx
```

La clave completa debe mostrarse únicamente al crearla.

---

# 31. API Key Storage

La API Key no debe almacenarse en texto plano si puede evitarse.

El sistema deberá conservar una representación segura que permita validar la clave.

Ejemplo:

```text
API Key
   ↓
Hash
   ↓
Database
```

---

# 32. API Key Metadata

Cada API Key debe tener:

```json
{
  "id": "key_001",
  "name": "Energy Server",
  "created_at": "...",
  "expires_at": "...",
  "last_used_at": "...",
  "scopes": [
    "entities:read"
  ],
  "status": "active"
}
```

---

# 33. API Key Expiration

Las API Keys deberán poder:

```text
no expirar
expirar en fecha
expirar después de N días
```

Para integraciones externas se recomienda establecer expiración.

---

# 34. API Key Rotation

Debe ser posible crear una nueva clave antes de revocar la anterior.

Ejemplo:

```text
Key A
   ↓
Create Key B
   ↓
Update application
   ↓
Revoke Key A
```

Esto evita interrupciones.

---

# 35. Scopes

Los scopes representan permisos funcionales.

Ejemplos:

```text
entities:read
entities:write
devices:read
devices:write
zones:read
groups:read
scenes:read
scenes:execute
automations:read
automations:trigger
history:read
events:read
virtual_entities:create
virtual_entities:write
integrations:manage
users:manage
system:manage
```

---

# 36. Scope separado de Role

No deben ser lo mismo.

```text
Role
    ↓
conjunto de permisos

Scope
    ↓
permiso solicitado por una aplicación/token
```

Ejemplo:

```text
Role:
    Energy Manager

Scopes:
    entities:read
    history:read
```

---

# 37. Roles

Roles recomendados inicialmente:

```text
Owner
Administrator
Operator
User
Viewer
Guest
Service
Integration
```

---

# 38. Owner

Control total de la instalación.

Puede:

```text
crear usuarios
eliminar usuarios
modificar seguridad
configurar integraciones
modificar dispositivos
modificar sistema
```

---

# 39. Administrator

Puede administrar el sistema, pero ciertas operaciones de máximo nivel pueden quedar reservadas al Owner.

---

# 40. Operator

Puede operar el sistema.

Ejemplo:

```text
lights
climate
doors
irrigation
scenes
```

pero no necesariamente:

```text
users
security policy
firmware
network
```

---

# 41. User

Usuario normal.

Puede controlar los recursos permitidos.

---

# 42. Viewer

Solo lectura.

Ejemplo:

```text
entities:read
history:read
events:read
```

---

# 43. Guest

Acceso extremadamente limitado.

Ejemplo:

```text
light.living
light.kitchen
```

únicamente.

---

# 44. Service

Identidad destinada a servicios internos.

No representa necesariamente una persona.

---

# 45. Integration

Rol para integraciones externas.

Debe comenzar con permisos mínimos.

---

# 46. RBAC

El sistema utilizará principalmente:

```text
RBAC
Role-Based Access Control
```

Ejemplo:

```text
User
  ↓
Role
  ↓
Permissions
```

Pero RBAC por sí solo no es suficiente.

---

# 47. ABAC

También se utilizarán atributos para tomar decisiones.

Ejemplo:

```text
User
Application
Zone
Entity
Time
Mode
Security level
```

Por lo tanto:

```text
RBAC + ABAC
```

será el modelo recomendado.

---

# 48. Policy Engine

La autorización deberá pasar por una política.

Ejemplo conceptual:

```text
ALLOW IF

user.authenticated
AND
user.has_scope("entities:write")
AND
entity.zone IN user.allowed_zones
AND
entity.type == "light"
AND
security_policy.allows(command)
```

---

# 49. Permission hierarchy

Los permisos tendrán diferentes niveles:

```text
System
 ↓
Zone
 ↓
Device
 ↓
Entity
 ↓
Operation
```

Ejemplo:

```text
entities:write
    +
zone:garden
    +
entity:irrigation.*
```

---

# 50. Permisos por zona

Un usuario puede tener:

```text
living
kitchen
bedroom
```

pero no:

```text
security
garage
industrial
```

Ejemplo:

```json
{
  "user": "usr_001",
  "zones": [
    "living",
    "kitchen"
  ]
}
```

---

# 51. Permisos por entidad

Cuando sea necesario:

```json
{
  "allow": [
    "light.living",
    "sensor.temperature.living"
  ]
}
```

Esto permite controles muy específicos.

---

# 52. Wildcards

Podrán utilizarse patrones.

Ejemplo:

```text
light.*
sensor.temperature.*
```

o:

```text
garden.*
```

Los wildcards deberán tener una semántica bien definida.

---

# 53. Deny explícito

Debe existir la posibilidad de bloquear un recurso aunque un rol superior lo permita.

Ejemplo:

```text
ALLOW:
    entities:write

DENY:
    alarm_control_panel.*
```

La política de resolución deberá ser:

```text
Explicit DENY
    >
Explicit ALLOW
    >
Role
    >
Default
```

---

# 54. Default Deny

La regla fundamental será:

> Si no existe una autorización explícita, el acceso se deniega.

Nunca:

```text
unknown permission = allow
```

---

# 55. Safety Policies

Incluso teniendo permisos, una operación puede ser rechazada por seguridad.

Ejemplo:

```text
Application:
    entities:write

Command:
    disable emergency stop

Authorization:
    ALLOW

Safety Policy:
    DENY
```

Resultado:

```text
DENIED_BY_SAFETY_POLICY
```

---

# 56. Security Zones

Las zonas pueden clasificarse.

Ejemplo:

```text
PUBLIC
NORMAL
PRIVATE
SECURITY
CRITICAL
```

Una aplicación externa podría acceder a:

```text
NORMAL
```

pero no:

```text
SECURITY
CRITICAL
```

sin permisos adicionales.

---

# 57. Sensitive Entities

Entidades sensibles:

```text
locks
alarms
garage doors
security sensors
gas valves
water main
emergency stops
industrial actuators
```

deben requerir permisos específicos.

---

# 58. Re-authentication

Para operaciones especialmente sensibles se puede requerir reautenticación.

Ejemplo:

```text
Delete administrator
Change security policy
Factory reset
Disable alarm
```

El sistema puede solicitar:

```text
password
passkey
2FA
```

---

# 59. Multi-Factor Authentication

MFA será una capacidad recomendada para:

```text
Owner
Administrator
Remote access
Security operations
```

Métodos posibles:

```text
TOTP
Passkeys
WebAuthn
Hardware security key
```

No se debe depender exclusivamente de SMS.

---

# 60. Passkeys / WebAuthn

A futuro se recomienda soportar:

```text
WebAuthn
Passkeys
```

para usuarios administrativos.

Esto permite:

```text
device-bound authentication
phishing resistance
```

sin almacenar contraseñas tradicionales para ese método.

---

# 61. Device Authentication

Los nodos ESP32 deben disponer de una identidad propia.

Ejemplo:

```text
Central
   ↓
Provisioning
   ↓
Device Credential
   ↓
ESP32
```

---

# 62. Device Credential

Durante el provisioning se podrá generar:

```text
device_id
device_secret
certificate
key pair
```

según el método de autenticación elegido.

---

# 63. Device Trust

Los dispositivos podrán tener estados:

```text
unknown
pending
trusted
blocked
revoked
retired
```

---

# 64. Device Enrollment

Proceso:

```text
ESP32 boot
   ↓
Discovery
   ↓
Enrollment request
   ↓
User approval
   ↓
Credential provisioning
   ↓
Trusted
```

---

# 65. No automatic trust

Un dispositivo que aparezca en la red no debe recibir automáticamente privilegios.

Nunca:

```text
ESP32 found
    ↓
FULL TRUST
```

---

# 66. Device Revocation

Un nodo comprometido debe poder revocarse.

```text
Device
   ↓
Revoked
   ↓
No authentication
   ↓
No commands
```

La información crítica de revocación debe persistir.

---

# 67. Mutual Authentication

Para comunicaciones críticas:

```text
Central ↔ Node
```

se podrá utilizar autenticación mutua.

Ejemplo:

```text
mTLS
```

Cada extremo demuestra su identidad.

---

# 68. Provisioning

El sistema tendrá un proceso de provisioning.

```text
Factory
   ↓
Unconfigured
   ↓
Provisioning
   ↓
Configured
   ↓
Trusted
```

---

# 69. Factory State

Un dispositivo recién instalado debe tener capacidades limitadas.

Ejemplo:

```text
AP provisioning
local setup
discovery
```

pero no:

```text
system control
```

---

# 70. Bootstrap Credentials

Las credenciales iniciales deben ser:

```text
unique
temporary
rotatable
```

Nunca debe existir una contraseña maestra idéntica para todos los dispositivos.

---

# 71. Remote Access

El acceso remoto deberá utilizar:

```text
HTTPS
Authentication
Authorization
```

preferentemente mediante:

```text
Secure Gateway
```

o:

```text
VPN
```

o:

```text
Zero-trust tunnel
```

---

# 72. No direct ESP32 exposure

Los nodos no deben exponerse directamente a Internet.

Arquitectura:

```text
Internet
   ↓
Secure Gateway
   ↓
Central
   ↓
Zone Controller
   ↓
Node
```

No:

```text
Internet
   ↓
ESP32
```

---

# 73. Network Trust

La red local no debe considerarse completamente confiable.

Ejemplo:

```text
Wi-Fi guest
IoT VLAN
Office VLAN
Automation VLAN
```

pueden coexistir.

La autenticación sigue siendo necesaria.

---

# 74. Network Segmentation

La arquitectura debe poder soportar:

```text
VLAN
Firewall
Subnet
VPN
Gateway
```

sin modificar el modelo de autorización.

---

# 75. Rate Limiting

Cada identidad puede tener límites.

Ejemplo:

```text
Web user:
    300 req/min

Third-party app:
    100 req/min

IoT device:
    60 req/min
```

Los límites deberán ser configurables.

---

# 76. Burst

Se podrá permitir un pequeño burst.

Ejemplo:

```text
100 req/min
burst = 20
```

---

# 77. Brute Force Protection

Los intentos repetidos de login deberán activar protección.

Ejemplo:

```text
5 failures
   ↓
temporary delay
```

y posteriormente:

```text
account lock / IP throttling
```

según la política.

No debe bloquearse permanentemente una cuenta simplemente porque una IP atacante lo intente.

---

# 78. Credential Stuffing

La plataforma deberá detectar patrones de:

```text
multiple usernames
same source
rapid attempts
```

y aplicar throttling.

---

# 79. Audit Log

Las operaciones importantes deberán generar auditoría.

Ejemplo:

```json
{
  "event": "authorization_denied",
  "timestamp": "2026-10-05T20:30:00Z",
  "actor": "app_001",
  "resource": "alarm_control_panel.house",
  "action": "disarm",
  "reason": "missing_scope"
}
```

---

# 80. Eventos auditables

Se deberán registrar como mínimo:

```text
login
login_failed
logout
password_changed
token_created
token_revoked
api_key_created
api_key_revoked
user_created
user_deleted
role_changed
permission_changed
device_enrolled
device_revoked
authorization_denied
critical_command
configuration_changed
```

---

# 81. Audit Log ≠ Event Bus

El registro de auditoría no debe confundirse con el sistema de eventos.

```text
Event Bus
    ↓
operación del sistema

Audit Log
    ↓
registro de seguridad
```

---

# 82. Retención de auditoría

La retención será configurable.

Ejemplo:

```text
30 días
90 días
1 año
```

En instalaciones industriales puede ser superior.

---

# 83. Protección de logs

Los logs no deben poder ser modificados por usuarios normales.

Idealmente:

```text
append-only
```

---

# 84. Privacy

La plataforma debe almacenar únicamente la información necesaria.

No registrar innecesariamente:

```text
password
tokens completos
API keys completas
secretos
```

---

# 85. Secret Redaction

Los logs deberán ocultar secretos.

Ejemplo:

```text
Authorization: Bearer ********
```

Nunca:

```text
Authorization: Bearer eyJhbGci...
```

---

# 86. Token Claims

Cuando se utilicen tokens estructurados, podrán contener:

```json
{
  "iss": "central.local",
  "sub": "usr_001",
  "aud": "third-party-api",
  "exp": 1791234567,
  "scope": "entities:read",
  "jti": "token_001"
}
```

---

# 87. JWT

JWT podrá utilizarse cuando sea conveniente.

Sin embargo:

> La plataforma no debe depender conceptualmente de JWT.

El modelo debe soportar tokens opacos si las limitaciones del dispositivo o la arquitectura lo hacen preferible.

---

# 88. Token Introspection

Cuando se utilicen tokens opacos, el servidor podrá validar:

```text
active
subject
scopes
expiration
```

mediante introspección interna.

---

# 89. Audience

Los tokens deberán estar destinados a un servicio específico cuando corresponda.

Ejemplo:

```text
aud = third-party-api
```

Esto evita reutilizar un token destinado a otro servicio.

---

# 90. Issuer

El emisor debe ser identificable.

Ejemplo:

```text
iss = central.local
```

o un identificador estable de instalación.

---

# 91. Token ID

Los tokens deben disponer de identificador único:

```text
jti
```

Esto facilita revocación y auditoría.

---

# 92. Clock Synchronization

La seguridad basada en expiración depende de tiempo correcto.

Por ello:

```text
NTP
```

debe utilizarse cuando esté disponible.

Sin embargo, la plataforma debe poder arrancar y funcionar temporalmente sin Internet/NTP.

---

# 93. Time Trust

Durante el arranque, si el reloj no es confiable, el sistema debe tener una estrategia definida.

Por ejemplo:

```text
RTC
persisted timestamp
NTP
secure time synchronization
```

---

# 94. Authorization Cache

Para evitar sobrecargar el ESP32-S3, algunas decisiones de autorización podrán almacenarse temporalmente.

Ejemplo:

```text
Authorization decision
       ↓
Cache
       ↓
TTL
```

Pero los cambios críticos deberán invalidar inmediatamente la caché.

---

# 95. Fail-Safe

Si el sistema de autorización está temporalmente indisponible:

```text
critical local automation
```

debe continuar funcionando.

La pérdida del servicio de autorización central no debe detener:

```text
device autonomy
safety
local security
```

---

# 96. Fail-Closed para API

Para una solicitud externa:

```text
authorization unavailable
```

la API debe asumir:

```text
DENY
```

salvo excepciones explícitas y seguras.

---

# 97. Fail-Safe ≠ Fail-Open

La automatización local puede continuar.

Pero eso no significa:

```text
API authorization unavailable
    ↓
ALLOW everything
```

Debe ser:

```text
API access
    ↓
DENY

Local automation
    ↓
CONTINUE
```

---

# 98. Emergency Operations

Las operaciones de emergencia deben seguir una política específica.

Ejemplo:

```text
Emergency Stop
```

puede ejecutarse localmente incluso si:

```text
Central
API
Internet
```

están caídos.

---

# 99. External Commands During Offline

Si un tercero envía:

```text
turn_on
```

mientras el dispositivo está desconectado, el sistema debe indicar:

```text
accepted
queued
failed
unavailable
```

sin mentir al cliente.

---

# 100. Permission Evaluation

La evaluación conceptual será:

```text
1. Authenticate actor
2. Validate token
3. Identify resource
4. Check scope
5. Check role
6. Check zone
7. Check entity permission
8. Check policy
9. Check safety
10. Check rate limit
11. Execute
12. Audit
```

---

# 101. Authorization Decision

La decisión interna debería poder representarse como:

```json
{
  "decision": "allow",
  "actor": "app_001",
  "action": "entities:write",
  "resource": "light.living",
  "zone": "living",
  "reason": "scope_and_policy_match"
}
```

---

# 102. Denial Reasons

Los rechazos deben ser diferenciables internamente:

```text
UNAUTHENTICATED
INVALID_TOKEN
TOKEN_EXPIRED
INSUFFICIENT_SCOPE
ROLE_DENIED
ZONE_DENIED
ENTITY_DENIED
SAFETY_POLICY_DENIED
RATE_LIMITED
RESOURCE_UNAVAILABLE
DEVICE_UNAVAILABLE
```

La API no debe revelar información sensible innecesaria.

---

# 103. Permission inheritance

Una zona puede definir permisos heredables.

Ejemplo:

```text
House
 ├── Living
 │    ├── Light
 │    └── Temperature
 └── Kitchen
      ├── Light
      └── Temperature
```

Permiso:

```text
zone:living
```

puede aplicarse a sus entidades.

---

# 104. Permission override

Una entidad puede sobrescribir la herencia.

Ejemplo:

```text
Zone:
    living → read/write

Entity:
    alarm.living → deny
```

---

# 105. Time-Based Permissions

A futuro podrán existir permisos condicionados por horario.

Ejemplo:

```text
Operator
    can control garden
    08:00 - 22:00
```

Fuera de ese horario:

```text
DENY
```

---

# 106. Mode-Based Permissions

Las políticas también pueden depender del modo del sistema.

Ejemplo:

```text
HOUSE.MODE = SLEEP
```

Una aplicación externa puede tener:

```text
light:write
```

pero una política puede impedir encender ciertas luces durante:

```text
sleep
```

salvo permisos especiales.

---

# 107. Security Mode

Ejemplo:

```text
HOUSE.MODE = AWAY
```

Una aplicación externa podría:

```text
read sensors
```

pero no:

```text
unlock doors
```

sin autorización específica.

---

# 108. Third-Party Application Registration

Las aplicaciones externas deberán registrarse.

```http
POST /api/v1/applications
```

Datos:

```json
{
  "name": "Energy Manager",
  "description": "Energy management system",
  "type": "third_party"
}
```

---

# 109. Application Lifecycle

Estados:

```text
pending
active
suspended
revoked
deleted
```

---

# 110. Application Approval

Flujo:

```text
Application registration
        ↓
Pending
        ↓
User approval
        ↓
Active
```

---

# 111. Application Suspension

Una aplicación puede suspenderse temporalmente.

```text
Active
  ↓
Suspended
```

Sus tokens dejan de funcionar.

---

# 112. Application Revocation

Revocación definitiva:

```text
Active
  ↓
Revoked
```

Todas sus credenciales deben invalidarse.

---

# 113. Third-Party Consent

La pantalla de autorización deberá mostrar:

```text
Application
Developer
Requested permissions
Zones
Entities
Expiration
```

Ejemplo:

```text
Energy Manager

Requests access to:

✓ Read energy sensors
✓ Read historical energy
✓ Control non-critical loads

Access:
Living
Garage

Duration:
90 days
```

---

# 114. Scope minimization

El sistema debería permitir seleccionar permisos concretos.

No:

```text
"Allow everything"
```

como única opción.

---

# 115. Device-to-Device Authentication

Los dispositivos internos también deben autenticarse.

Ejemplo:

```text
Node A
   ↓
authenticate
   ↓
Central
```

y:

```text
Central
   ↓
authenticate
   ↓
Node A
```

---

# 116. Service-to-Service Authentication

Ejemplo:

```text
Automation Engine
       ↓
History Service
```

deberá utilizar identidad de servicio.

No compartir:

```text
admin password
```

entre servicios.

---

# 117. Secret Management

Los secretos deben centralizarse en un mecanismo de almacenamiento seguro.

Ejemplo conceptual:

```text
Secret Store
├── device credentials
├── API keys
├── integration secrets
├── certificates
└── encryption keys
```

En ESP32 se utilizarán las capacidades seguras disponibles del hardware cuando corresponda.

---

# 118. Encryption at Rest

Los datos sensibles almacenados localmente deberían protegerse.

Especialmente:

```text
password hashes
API credentials
refresh tokens
integration secrets
private keys
```

---

# 119. Secure Boot

En dispositivos compatibles se recomienda utilizar:

```text
Secure Boot
```

para proteger la ejecución de firmware no autorizado.

---

# 120. Flash Encryption

Cuando el hardware y la plataforma lo permitan:

```text
Flash Encryption
```

deberá utilizarse para proteger secretos almacenados.

---

# 121. Firmware Trust

La autenticación no sustituye la seguridad del firmware.

La plataforma deberá contemplar:

```text
signed firmware
OTA verification
version validation
rollback protection
```

---

# 122. Compromised Device

Si un nodo es comprometido:

```text
device revoked
```

y posteriormente:

```text
credentials rotated
```

El resto de la red debe continuar funcionando.

---

# 123. Blast Radius

Los permisos deben diseñarse para limitar el impacto de una credencial comprometida.

Ejemplo:

```text
Weather API
```

solo tiene:

```text
garden/weather entities
```

No:

```text
security
doors
users
system
```

---

# 124. Principle of Least Privilege

Regla fundamental:

> Toda identidad debe disponer únicamente de los privilegios estrictamente necesarios para realizar su función.

---

# 125. Local Administrator

Incluso el administrador local debe utilizar autenticación.

No se debe crear:

```text
local network = trusted
```

como sustituto de login.

---

# 126. Recovery

Debe existir un mecanismo de recuperación de acceso.

Posibilidades:

```text
physical recovery button
local provisioning
recovery key
backup administrator
```

El método elegido debe evitar crear una puerta trasera permanente.

---

# 127. Factory Reset

El reset de fábrica debe eliminar:

```text
users
sessions
API keys
tokens
applications
device trust
integrations
```

según el nivel de reset seleccionado.

---

# 128. Reset Levels

Se recomienda disponer de:

```text
Soft Reset
Configuration Reset
Security Reset
Factory Reset
```

---

# 129. Soft Reset

Reinicia el dispositivo sin eliminar credenciales.

---

# 130. Configuration Reset

Elimina configuración funcional.

Mantiene:

```text
identity
security
firmware
```

según diseño.

---

# 131. Security Reset

Revoca:

```text
sessions
tokens
API keys
applications
```

---

# 132. Factory Reset

Devuelve el dispositivo al estado inicial de provisioning.

---

# 133. Backup

Las configuraciones de seguridad deberán poder incluirse en backups de forma segura.

Nunca exportar:

```text
plain passwords
private keys
active tokens
```

sin protección.

---

# 134. Restore

Un backup restaurado debe poder invalidar credenciales anteriores si existe riesgo de reutilización.

---

# 135. Audit of External Commands

Cada comando externo relevante deberá registrar:

```text
actor
application
user
entity
command
timestamp
request_id
result
```

---

# 136. Example

```json
{
  "actor": {
    "type": "application",
    "id": "app_energy"
  },
  "user": {
    "id": "usr_001"
  },
  "action": "set_brightness",
  "resource": "light.living",
  "result": "success",
  "request_id": "req_123"
}
```

---

# 137. Anonymous Access

Por defecto:

```text
anonymous = no access
```

No se deben exponer sensores simplemente porque están en una red local.

---

# 138. Public Data

Si alguna instalación necesita publicar datos:

```text
public temperature
public weather
public energy production
```

debe crearse explícitamente un recurso público.

Ejemplo:

```text
public.sensor.outdoor_temperature
```

Nunca debe hacerse público automáticamente:

```text
sensor.outdoor_temperature
```

---

# 139. Public API

Una eventual API pública debe ser una capa separada.

```text
Private API
      |
      └── Public API
```

La Public API tendrá:

```text
limited resources
rate limiting
no administrative access
no private entities
```

---

# 140. External Cloud

Una aplicación cloud no debe recibir acceso total a la instalación.

Ejemplo:

```text
Cloud Weather Service
```

solo:

```text
weather entities
```

---

# 141. Remote User

Un usuario remoto debe autenticarse igual que uno local.

La diferencia será:

```text
network path
```

no:

```text
security model
```

---

# 142. Network Path Independence

La autorización debe funcionar independientemente de si la solicitud llega por:

```text
Ethernet
Wi-Fi
VPN
Secure Gateway
Local Network
Remote Network
```

---

# 143. Security Headers

La API web deberá utilizar políticas de seguridad apropiadas.

Entre ellas podrán incluirse:

```text
HSTS
Content-Security-Policy
X-Content-Type-Options
Referrer-Policy
```

según el componente.

---

# 144. CORS

El acceso desde navegadores externos debe estar restringido.

No:

```text
Access-Control-Allow-Origin: *
```

para interfaces autenticadas por defecto.

---

# 145. CSRF

Las interfaces basadas en cookies deberán protegerse contra:

```text
CSRF
```

Las APIs basadas exclusivamente en Bearer Tokens enviados explícitamente tienen un modelo diferente, pero deben evaluarse según el cliente.

---

# 146. Cookie Security

Si se utilizan cookies:

```text
Secure
HttpOnly
SameSite
```

deben configurarse adecuadamente.

---

# 147. API Security Testing

Antes de producción se deberán probar:

```text
authentication bypass
authorization bypass
token replay
expired tokens
revoked tokens
scope escalation
role escalation
zone bypass
IDOR
rate limit bypass
CSRF
CORS
injection
```

---

# 148. IDOR

Nunca se debe permitir:

```http
GET /entities/entity_123
```

simplemente porque el usuario conoce el ID.

El servidor debe verificar autorización sobre ese recurso.

---

# 149. Enumeration Protection

Los endpoints sensibles no deben revelar innecesariamente si:

```text
username exists
entity exists
device exists
```

a un cliente no autorizado.

---

# 150. Error Information

Los mensajes externos deben equilibrar utilidad y seguridad.

En lugar de:

```text
User exists but password is incorrect
```

utilizar:

```text
Invalid credentials
```

---

# 151. Security Events

Los eventos de seguridad podrán integrarse al sistema:

```text
security.login_failed
security.device_revoked
security.authorization_denied
security.application_revoked
```

Esto permite automatizaciones de seguridad.

---

# 152. Security Automation

Ejemplo:

```text
5 failed logins
      ↓
Security Automation
      ↓
Notify administrator
```

Otro:

```text
Unknown device
      ↓
Security event
      ↓
Notify administrator
```

---

# 153. Security Mode Integration

La autorización podrá consultar:

```text
HOUSE.MODE
```

Ejemplo:

```text
NORMAL
SLEEP
AWAY
VACATION
MAINTENANCE
EMERGENCY
```

Una política puede cambiar dependiendo del modo.

---

# 154. Maintenance Mode

Durante mantenimiento puede ser necesario permitir:

```text
device:write
configuration:write
```

a técnicos autorizados.

Esto debe ser explícito y auditable.

---

# 155. Temporary Access

Se podrán generar accesos temporales.

Ejemplo:

```text
Technician
    ↓
Access
    ↓
Garden + Garage
    ↓
2 hours
    ↓
Automatic expiration
```

---

# 156. Temporary Tokens

Los tokens temporales deben incluir:

```text
expiration
scope
zone
purpose
```

---

# 157. Technician Role

Puede existir un rol:

```text
Technician
```

con permisos de:

```text
diagnostics
device configuration
firmware
```

pero sin:

```text
user administration
billing
owner transfer
```

según la instalación.

---

# 158. Ownership Transfer

Debe existir un proceso seguro para transferir la propiedad de una instalación.

Ejemplo:

```text
Owner A
   ↓
Transfer ownership
   ↓
Owner B
```

Debe requerir confirmación fuerte.

---

# 159. Multi-User

La instalación debe soportar múltiples usuarios.

Ejemplo:

```text
Owner
Administrator
Family
Technician
Guest
```

Cada uno con permisos diferentes.

---

# 160. Multi-Tenant Future

Aunque la primera versión no lo implemente, la arquitectura debe evitar bloquear un futuro soporte de:

```text
multiple organizations
multiple installations
multiple sites
```

Modelo posible:

```text
Organization
   └── Sites
        └── Zones
             └── Devices
                  └── Entities
```

---

# 161. Site

Una instalación física puede representarse como:

```text
Site
```

Ejemplo:

```text
Casa Klein
Fábrica Esperanza
Invernadero 01
Barco 01
```

---

# 162. User Access to Sites

Un usuario puede tener acceso a:

```text
Site A
Site B
```

pero no necesariamente:

```text
Site C
```

---

# 163. Future Cloud Federation

La arquitectura podrá evolucionar hacia:

```text
Local Central
      ↕
Cloud Identity
      ↕
Multiple Sites
```

sin convertir el cloud en requisito para automatización local.

---

# 164. Security Architecture

Resumen:

```text
                    INTERNET
                       │
                       ▼
                 Secure Gateway
                       │
                       ▼
                 HTTPS / WSS
                       │
                       ▼
              Authentication
                       │
                       ▼
                  Identity
                       │
                       ▼
              Role + Scope
                       │
                       ▼
               Zone Permission
                       │
                       ▼
             Entity Permission
                       │
                       ▼
               Safety Policy
                       │
                       ▼
                Rate Limiting
                       │
                       ▼
                  COMMAND
                       │
                       ▼
                    AUDIT
```

---

# 165. Authentication Decision Tree

```text
Request
   │
   ├── No credentials?
   │       └── 401
   │
   ├── Invalid credentials?
   │       └── 401
   │
   ├── Expired credential?
   │       └── 401
   │
   ├── Valid identity?
   │       └── continue
   │
   ├── Scope missing?
   │       └── 403
   │
   ├── Zone denied?
   │       └── 403
   │
   ├── Entity denied?
   │       └── 403
   │
   ├── Safety denied?
   │       └── 403
   │
   └── Execute
```

---

# 166. Authorization Model

La decisión final será conceptualmente:

```text
ALLOW =
    Authenticated
    AND ValidCredential
    AND RequiredScope
    AND RoleAllows
    AND ZoneAllows
    AND EntityAllows
    AND PolicyAllows
    AND SafetyAllows
    AND RateLimitAllows
```

---

# 167. Security Priority

La prioridad de seguridad será:

```text
Safety
   >
Security
   >
Authorization
   >
Automation
   >
Third Party
   >
Cloud
```

---

# 168. Principio de autonomía

La autenticación y autorización de la API no deben convertirse en una dependencia de funcionamiento local.

Ejemplo:

```text
Internet DOWN
    ↓
Third-party API unavailable

BUT

ESP32
    ↓
local automation
    ↓
continues
```

---

# 169. Central Failure

Si el Central falla:

```text
Third-party API
    ↓
temporarily unavailable
```

pero:

```text
Node autonomy
Zone autonomy
Safety
Security
```

continúan.

---

# 170. Central Recovery

Cuando vuelve:

```text
Central
   ↓
Identity services
   ↓
Authorization state
   ↓
Synchronize
```

Las credenciales y políticas deben reconciliarse con el estado persistente.

---

# 171. Minimal Embedded Implementation

La primera versión en ESP32 puede comenzar con:

```text
HTTPS
API Keys
Bearer Tokens
RBAC
Scopes
Zone permissions
Audit
Rate limiting
```

No es necesario implementar inicialmente:

```text
OAuth
OIDC
WebAuthn
Cloud IAM
```

pero la arquitectura no debe impedirlo.

---

# 172. Recommended Initial Security Stack

Para el primer desarrollo:

```text
TLS
+
Password authentication
+
Short-lived access tokens
+
Refresh tokens
+
API Keys
+
RBAC
+
Scopes
+
Zone permissions
+
Default Deny
+
Audit Log
+
Rate limiting
```

---

# 173. Future Security Stack

Posteriormente:

```text
OAuth 2.0
OpenID Connect
Passkeys
WebAuthn
mTLS
Certificate provisioning
Secure Boot
Flash Encryption
External Identity Provider
Cloud Federation
```

---

# 174. API Security Documentation

La implementación deberá generar documentación para:

```text
Authentication
Authorization
Tokens
API Keys
Scopes
Roles
Errors
Security
Examples
```

---

# 175. OpenAPI Security Schemes

El documento OpenAPI deberá definir los mecanismos utilizados.

Ejemplo conceptual:

```yaml
securitySchemes:
  bearerAuth:
    type: http
    scheme: bearer

  apiKeyAuth:
    type: http
    scheme: bearer
```

La definición real deberá mantenerse sincronizada con la implementación.

---

# 176. SDK Security

Los SDK oficiales nunca deben:

```text
log tokens
store passwords in plaintext
disable TLS verification
```

por defecto.

---

# 177. Developer Mode

Puede existir un:

```text
Developer Mode
```

pero nunca debe desactivar completamente las protecciones en una instalación de producción.

---

# 178. Development Credentials

En desarrollo pueden existir credenciales temporales.

Deben:

```text
ser diferentes de producción
tener expiración
estar claramente identificadas
```

---

# 179. Production Security

Antes de marcar una instalación como:

```text
PRODUCTION
```

deberán verificarse:

```text
default credentials removed
TLS configured
admin password configured
audit enabled
backup configured
firmware trusted
remote access secured
```

---

# 180. Security Checklist

Antes de producción:

```text
[ ] No default admin password
[ ] Password hashing configured
[ ] TLS enabled
[ ] Access tokens expire
[ ] Refresh token rotation
[ ] API keys can expire
[ ] API keys can be revoked
[ ] RBAC enabled
[ ] Scopes enabled
[ ] Zone permissions enabled
[ ] Default deny
[ ] Audit log enabled
[ ] Rate limiting enabled
[ ] Brute-force protection
[ ] Device enrollment secured
[ ] Device revocation available
[ ] Secrets not logged
[ ] Remote access secured
[ ] Critical entities protected
[ ] Factory reset defined
[ ] Backup/recovery tested
```

---

# 181. Security Principles

## Principle 1 — Default Deny

Todo acceso se deniega salvo autorización explícita.

## Principle 2 — Least Privilege

Cada identidad recibe únicamente los permisos necesarios.

## Principle 3 — Defense in Depth

No depender de una sola barrera de seguridad.

## Principle 4 — Local Autonomy

La pérdida del Central no debe detener la automatización crítica.

## Principle 5 — Auditability

Las operaciones importantes deben poder rastrearse.

## Principle 6 — Revocability

Toda credencial importante debe poder revocarse.

## Principle 7 — Isolation

El compromiso de una aplicación no debe comprometer toda la instalación.

## Principle 8 — No Secrets in Code

Las credenciales nunca deben estar codificadas en firmware.

## Principle 9 — Hardware Independence

Los permisos deben aplicarse sobre recursos lógicos, no GPIO.

## Principle 10 — Fail Secure

Ante una solicitud externa sin autorización válida, se debe denegar.

---

# 182. Ejemplo completo

Supongamos:

```text
Application:
    Energy Manager

Requested:
    entities:read
    history:read
    entities:write

Zones:
    living
    garage
```

La aplicación intenta:

```text
light.living → ON
```

Resultado:

```text
authenticated
       ↓
scope valid
       ↓
zone valid
       ↓
entity valid
       ↓
safety valid
       ↓
ALLOW
```

Ahora intenta:

```text
alarm_control_panel.house → DISARM
```

Resultado:

```text
authenticated
       ↓
scope valid
       ↓
zone valid
       ↓
security policy
       ↓
DENY
```

Aunque la aplicación tenga:

```text
entities:write
```

la política de seguridad impide la operación.

---

# 183. Ejemplo de aplicación meteorológica

```text
Weather Service
```

Scopes:

```text
virtual_entities:create
virtual_entities:write
entities:read
```

Puede:

```text
crear sensor.weather.temperature
publicar temperatura
leer sensores meteorológicos
```

No puede:

```text
controlar luces
abrir puertas
modificar usuarios
modificar firmware
```

---

# 184. Ejemplo de aplicación móvil

La aplicación móvil puede solicitar:

```text
entities:read
entities:write
scenes:execute
events:read
```

pero el usuario puede limitarla a:

```text
living
kitchen
bedroom
```

---

# 185. Ejemplo de técnico

```text
Technician App
```

Permisos:

```text
devices:read
devices:write
diagnostics:read
firmware:update
```

Zona:

```text
garage
industrial
```

Duración:

```text
2 hours
```

Después:

```text
automatic revoke
```

---

# 186. Ejemplo de dispositivo

Un ESP32 Node puede tener:

```text
device identity
```

y únicamente:

```text
state:publish
events:publish
commands:receive
```

No puede:

```text
create users
modify security
manage API keys
```

---

# 187. Relación con Third-Party API

La arquitectura completa queda:

```text
                THIRD PARTY
                     │
                     ▼
              Authentication
                     │
                     ▼
                Authorization
                     │
                     ▼
               Third-Party API
                     │
                     ▼
                Device Model
                     │
                     ▼
              Internal System Bus
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
      CAN           RS485        Ethernet
       │             │             │
       ▼             ▼             ▼
     ESP32         ESP32         ESP32
```

---

# 188. Relación con Smart Home Integration

Las integraciones de:

```text
Home Assistant
Alexa
Google Home
Apple Home
SmartThings
Homey
Matter
MQTT
```

utilizarán el mismo modelo de identidad y permisos.

No deberán implementar un sistema de seguridad completamente independiente.

---

# 189. Relación con Central

El Central será normalmente responsable de:

```text
Identity
Authentication
Authorization
API
Applications
Tokens
Audit
Policies
```

pero la autonomía local permanecerá en:

```text
Zone Controllers
Nodes
```

---

# 190. Regla arquitectónica definitiva

La seguridad no debe estar repartida arbitrariamente por los drivers.

Debe existir una capa clara:

```text
┌───────────────────────────────┐
│       Authentication          │
├───────────────────────────────┤
│       Authorization           │
├───────────────────────────────┤
│       Policy Engine           │
├───────────────────────────────┤
│       Device Model            │
├───────────────────────────────┤
│       System Bus              │
├───────────────────────────────┤
│       Hardware                │
└───────────────────────────────┘
```

---

# 191. Regla de oro

> **Estar autenticado no significa tener acceso. Tener acceso no significa poder ejecutar cualquier operación. Tener permiso para ejecutar una operación no significa que la política de seguridad permita ejecutarla en ese momento.**

La decisión final debe considerar:

```text
IDENTITY
+
CREDENTIAL
+
ROLE
+
SCOPE
+
RESOURCE
+
ZONE
+
POLICY
+
SAFETY
+
CONTEXT
```

---

# 192. Principio final

La plataforma debe permitir integraciones externas sin convertirlas en una amenaza para el sistema.

Una aplicación externa debe poder:

```text
READ
WRITE
PUBLISH
SUBSCRIBE
CREATE
EXECUTE
```

cuando esté autorizada.

Pero nunca debe poder:

```text
BYPASS SECURITY
BYPASS SAFETY
BYPASS LOCAL AUTONOMY
BYPASS AUTHORIZATION
```

La arquitectura definitiva es:

```text
                 ┌───────────────────┐
                 │      USER         │
                 └─────────┬─────────┘
                           │
                 ┌─────────▼─────────┐
                 │  AUTHENTICATION   │
                 └─────────┬─────────┘
                           │
                 ┌─────────▼─────────┐
                 │     IDENTITY      │
                 └─────────┬─────────┘
                           │
                 ┌─────────▼─────────┐
                 │       RBAC        │
                 │        +          │
                 │       ABAC        │
                 └─────────┬─────────┘
                           │
                 ┌─────────▼─────────┐
                 │      SCOPES       │
                 └─────────┬─────────┘
                           │
                 ┌─────────▼─────────┐
                 │  ZONE / ENTITY    │
                 │    PERMISSIONS    │
                 └─────────┬─────────┘
                           │
                 ┌─────────▼─────────┐
                 │  SECURITY POLICY  │
                 └─────────┬─────────┘
                           │
                 ┌─────────▼─────────┐
                 │  SAFETY POLICY    │
                 └─────────┬─────────┘
                           │
                 ┌─────────▼─────────┐
                 │     RESOURCE      │
                 └─────────┬─────────┘
                           │
                 ┌─────────▼─────────┐
                 │    AUDIT LOG      │
                 └───────────────────┘
```

> **La seguridad debe proteger la plataforma sin convertirse en una dependencia de la automatización. El sistema debe poder seguir funcionando localmente aunque la autenticación remota, Internet o el Central estén temporalmente fuera de servicio.**

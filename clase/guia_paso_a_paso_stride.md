# 🧭 Guía Paso a Paso: Evaluación de Seguridad con STRIDE

Esta guía complementa el `README.md` del taller. Su objetivo es que, antes de analizar un flujo crítico de EdukIT en clase (Parte 1) o del sistema del cliente real (Parte 2), el equipo tenga una referencia clara del marco STRIDE y de la metodología para pasar de "dibujar el flujo" a "priorizar amenazas reales".

El diagrama de ejemplo de esta guía está escrito en [Mermaid](https://mermaid.js.org/) y se renderiza automáticamente al ver este archivo en GitHub.

---

## 1. Las 6 categorías de STRIDE

| Categoría | Pregunta guía | Ejemplo típico |
|---|---|---|
| **S**poofing (Suplantación) | ¿Alguien puede hacerse pasar por otro usuario o sistema? | Credenciales robadas, phishing |
| **T**ampering (Alteración) | ¿Alguien puede modificar datos o mensajes sin autorización? | Interceptar y modificar un token en tránsito |
| **R**epudiation (Repudio) | ¿Alguien puede negar haber realizado una acción? | Falta de logs de auditoría |
| **I**nformation Disclosure (Divulgación) | ¿Puede exponerse información que debería ser privada? | Consulta sin control de acceso, backup expuesto |
| **D**enial of Service (Denegación de servicio) | ¿Puede alguien dejar el sistema o un componente inaccesible? | Saturación de solicitudes |
| **E**levation of Privilege (Elevación de privilegios) | ¿Puede alguien obtener permisos mayores a los que le corresponden? | Manipular el rol en una solicitud |

---

## 2. Metodología en 5 pasos

STRIDE no se aplica "en el aire": se aplica sobre un diagrama de flujo de datos (DFD) del proceso. Sin ese diagrama, las amenazas terminan siendo genéricas.

1. **Elegir el flujo y dibujar su DFD** — represente el flujo crítico como actores externos, procesos, almacenes de datos y los flujos entre ellos, marcando el límite de confianza (qué está dentro y qué está fuera del control del sistema).
2. **Identificar los elementos a analizar** — liste cada proceso, almacén de datos y flujo del DFD; cada uno es candidato a amenazas.
3. **Aplicar las 6 categorías STRIDE** — para cada elemento relevante, formule la amenaza concreta en cada categoría que aplique (no todas las categorías aplican a todos los elementos).
4. **Evaluar impacto y proponer mitigación** — para cada amenaza identificada, describa su impacto y un control concreto que la mitigue.
5. **Priorizar por riesgo** — combine impacto y probabilidad para asignar un nivel de riesgo a cada amenaza y ordene la tabla de mayor a menor riesgo.

---

## 3. Ejemplo guiado: Acceso de estudiantes a cursos y materiales (EdukIT)

### Paso 1 — Elegir el flujo y dibujar su DFD

Se elige el flujo de **acceso de estudiantes a cursos y materiales**: el estudiante se autentica y luego solicita el contenido de un curso. Se marca el límite de confianza entre el estudiante (fuera del control de EdukIT) y el backend (dentro).

```mermaid
flowchart LR
    estudiante(["🧑 Estudiante"])

    subgraph backend["Backend EdukIT (zona de confianza)"]
        auth["P1: Sistema de Autenticación"]
        cursos["P2: Módulo de Cursos"]
        dbusuarios[("D1: BD de Usuarios")]
        dbcontenido[("D2: Almacén de Contenido")]
    end

    estudiante -->|"F1: credenciales"| auth
    auth -->|"F2: consulta"| dbusuarios
    auth -->|"F3: token de sesión"| estudiante
    estudiante -->|"F4: solicita curso + token"| cursos
    cursos -->|"F5: consulta"| dbcontenido
    cursos -->|"F6: contenido"| estudiante
```

### Paso 2 — Identificar los elementos a analizar

| ID | Elemento | Tipo |
|---|---|---|
| E1 | Estudiante | Actor externo |
| P1 | Sistema de Autenticación | Proceso |
| P2 | Módulo de Cursos | Proceso |
| D1 | BD de Usuarios | Almacén de datos |
| D2 | Almacén de Contenido | Almacén de datos |
| F1–F6 | Flujos de datos entre los anteriores | Flujo |

### Paso 3 — Aplicar las 6 categorías STRIDE

Se formula una amenaza concreta por categoría, señalando el elemento exacto sobre el que ocurre y qué control ya existe hoy frente a ella (todavía sin impacto ni mitigación — eso se agrega en el Paso 4):

| ID | Componente / Activo | Tipo STRIDE | Descripción de la Amenaza | Escenario de Ataque | Controles de Seguridad Existentes |
|---|---|---|---|---|---|
| T1 | Sistema de Autenticación (P1) / Credenciales (F1) | Spoofing | Un atacante se hace pasar por un estudiante usando credenciales robadas o phishing. | Atacante usa credenciales robadas vía phishing para iniciar sesión. | Autenticación con usuario y contraseña, sin MFA. |
| T2 | Token de sesión (F3) | Tampering | El token de sesión es interceptado y modificado en tránsito si la comunicación no usa TLS. | Atacante intercepta la comunicación en una red no segura (ej. WiFi público) y modifica el token antes de que llegue al servidor. | Comunicación cifrada con HTTPS en el flujo principal, sin verificación de firma del token en cada solicitud. |
| T3 | Sistema de Autenticación (P1) — registro de acciones | Repudiation | Un estudiante o administrador realiza una acción sensible (ej. cambia una nota) y luego niega haberlo hecho, por falta de registro de auditoría. | Un docente cambia una calificación desde el panel y luego niega la acción al no existir registro con marca de tiempo y usuario. | Registro de accesos básico, sin logs de auditoría con marca de tiempo y usuario para acciones sensibles. |
| T4 | BD de Usuarios (D1) | Information Disclosure | Exposición de datos personales o notas académicas por una consulta sin control de acceso adecuado. | Atacante explota un endpoint de consulta sin validar permisos y descarga el historial académico de otros estudiantes. | Control de acceso a nivel de aplicación, sin validación de permisos a nivel de consulta a la BD. |
| T5 | Módulo de Cursos (P2) | Denial of Service | Un atacante satura las solicitudes de contenido y deja el módulo de cursos inaccesible durante un examen. | Atacante lanza un ataque de flooding contra el endpoint de contenido de cursos durante la semana de exámenes. | Balanceador de carga básico, sin límite de tasa (rate limiting) configurado. |
| T6 | Solicitud de curso con rol (F4) | Elevation of Privilege | Un estudiante manipula el parámetro de rol en la solicitud para acceder a funciones de docente o administrador. | Estudiante modifica manualmente el campo `role` en el payload de la solicitud para obtener acceso de administrador. | Validación de rol en el cliente (frontend), sin revalidación estricta en el servidor. |

### Paso 4 — Evaluar impacto y proponer mitigación

Se completan las columnas restantes de la plantilla oficial (impacto, probabilidad, nivel de riesgo, mitigación, responsable y estado), llegando así a la tabla combinada final con las 12 columnas de `plantilla_analisis_stride.xlsx` — esta es la tabla que se entrega como `tabla-stride-clase.xlsx`:

| ID | Componente / Activo | Tipo STRIDE | Descripción de la Amenaza | Escenario de Ataque | Impacto | Probabilidad | Nivel de Riesgo | Controles de Seguridad Existentes | Mitigación Recomendada | Responsable | Estado |
|---|---|---|---|---|---|---|---|---|---|---|---|
| T1 | Sistema de Autenticación (P1) / Credenciales (F1) | Spoofing | Un atacante se hace pasar por un estudiante usando credenciales robadas o phishing. | Atacante usa credenciales robadas vía phishing para iniciar sesión. | Alto — acceso no autorizado a la cuenta y sus datos | Media | **Alto** | Autenticación con usuario y contraseña, sin MFA. | Autenticación multifactor (MFA) y bloqueo tras intentos fallidos | Equipo de Seguridad | Pendiente |
| T2 | Token de sesión (F3) | Tampering | El token de sesión es interceptado y modificado en tránsito si la comunicación no usa TLS. | Atacante intercepta la comunicación en una red no segura (ej. WiFi público) y modifica el token antes de que llegue al servidor. | Alto — secuestro de sesión activa | Baja | Medio | Comunicación cifrada con HTTPS en el flujo principal, sin verificación de firma del token en cada solicitud. | Forzar HTTPS/TLS en todas las comunicaciones y firmar/verificar el token (JWT) | Equipo Backend | En análisis |
| T3 | Sistema de Autenticación (P1) — registro de acciones | Repudiation | Un estudiante o administrador realiza una acción sensible (ej. cambia una nota) y luego niega haberlo hecho, por falta de registro de auditoría. | Un docente cambia una calificación desde el panel y luego niega la acción al no existir registro con marca de tiempo y usuario. | Medio — dificulta resolver disputas académicas | Media | Medio | Registro de accesos básico, sin logs de auditoría con marca de tiempo y usuario para acciones sensibles. | Registro de auditoría (logs) con marca de tiempo y usuario para acciones sensibles | DevOps | Pendiente |
| T4 | BD de Usuarios (D1) | Information Disclosure | Exposición de datos personales o notas académicas por una consulta sin control de acceso adecuado. | Atacante explota un endpoint de consulta sin validar permisos y descarga el historial académico de otros estudiantes. | Alto — incumplimiento de protección de datos (Ley 1581) | Media | **Alto** | Control de acceso a nivel de aplicación, sin validación de permisos a nivel de consulta a la BD. | Cifrado en reposo y control de acceso basado en roles (RBAC) a nivel de consulta | Equipo de Arquitectura | En progreso |
| T5 | Módulo de Cursos (P2) | Denial of Service | Un atacante satura las solicitudes de contenido y deja el módulo de cursos inaccesible durante un examen. | Atacante lanza un ataque de flooding contra el endpoint de contenido de cursos durante la semana de exámenes. | Medio — interrupción temporal del servicio | Baja | Bajo | Balanceador de carga básico, sin límite de tasa (rate limiting) configurado. | Límite de tasa (rate limiting) y auto-escalado con balanceo de carga | Infraestructura | Pendiente |
| T6 | Solicitud de curso con rol (F4) | Elevation of Privilege | Un estudiante manipula el parámetro de rol en la solicitud para acceder a funciones de docente o administrador. | Estudiante modifica manualmente el campo `role` en el payload de la solicitud para obtener acceso de administrador. | Alto — control total sobre contenido o calificaciones ajenas | Baja | Medio | Validación de rol en el cliente (frontend), sin revalidación estricta en el servidor. | Validar el rol en el servidor en cada solicitud; nunca confiar en el rol enviado por el cliente | Equipo de Seguridad | En análisis |

### Paso 5 — Priorizar por riesgo

A partir de la tabla completa del Paso 4, se construye una vista priorizada: se ordenan los ID por `Nivel de Riesgo` de mayor a menor. El detalle completo (impacto, probabilidad, controles existentes, mitigación, responsable y estado) permanece en la tabla del Paso 4; aquí solo se resume lo necesario para decidir qué atender primero.

| Prioridad | ID | Tipo STRIDE | Componente / Activo | Nivel de Riesgo |
|---|---|---|---|---|
| 1 | T4 | Information Disclosure | BD de Usuarios (D1) | **Alto** |
| 2 | T1 | Spoofing | Sistema de Autenticación (P1) / Credenciales (F1) | **Alto** |
| 3 | T6 | Elevation of Privilege | Solicitud de curso con rol (F4) | Medio |
| 4 | T2 | Tampering | Token de sesión (F3) | Medio |
| 5 | T3 | Repudiation | Sistema de Autenticación (P1) — registro de acciones | Medio |
| 6 | T5 | Denial of Service | Módulo de Cursos (P2) | Bajo |

---

## 4. Errores comunes a evitar

| Error frecuente | Por qué es un problema | Cómo corregirlo |
|---|---|---|
| Amenazas genéricas ("puede haber un hackeo") | No es accionable ni evaluable: no dice qué elemento ni cómo | Formule la amenaza sobre un elemento específico del DFD (ej. "el flujo F1 puede ser interceptado...") |
| Aplicar solo 2–3 categorías y omitir el resto | El marco pierde su propósito: cubrir sistemáticamente los 6 tipos de riesgo | Revise las 6 categorías para cada elemento relevante, aunque alguna no aplique |
| Mitigación vaga ("mejorar la seguridad") | No es una acción verificable ni evaluable en la rúbrica | Proponga un control concreto (MFA, cifrado, rate limiting, RBAC, logs de auditoría, etc.) |
| No priorizar los hallazgos | Un informe con muchas amenazas sin orden no ayuda a decidir qué atender primero | Clasifique cada amenaza por impacto × probabilidad y priorice (Paso 5) |

---

## 5. Checklist de autoevaluación antes de entregar

- [ ] Se documentó un DFD (o descripción equivalente) del flujo analizado, con procesos, almacenes de datos y flujos.
- [ ] Se aplicaron las 6 categorías STRIDE sobre los elementos relevantes del flujo.
- [ ] Cada amenaza está redactada sobre un elemento específico, no de forma genérica.
- [ ] Cada amenaza tiene una mitigación concreta y verificable.
- [ ] Cada amenaza tiene impacto, probabilidad y nivel de riesgo asignado.
- [ ] Los hallazgos están priorizados de mayor a menor riesgo.

---

## 6. Vista ArchiMate equivalente

STRIDE no tiene una capa propia en ArchiMate — sus mitigaciones se modelan como **Requirement** en la capa de Motivación (ver la [Guía de Notación ArchiMate](https://github.com/CesarAVegaF312/AREM-ArchiMate/blob/main/guia_notacion_archimate.md)), conectados con **Influence** al elemento de Aplicación o Tecnología que deben proteger.

```mermaid
flowchart TD
    subgraph motivacion["Motivación"]
        req(["📋 Requisito: autenticación multifactor"])
    end
    subgraph aplicacion["Aplicación"]
        auth["Sistema de Autenticación"]
    end

    req -.->|"influye sobre"| auth

    classDef motivacion fill:#ccccff,color:#000,stroke:#6666cc;
    classDef aplicacion fill:#99ccff,color:#000,stroke:#3366cc;
    class req motivacion
    class auth aplicacion
```

Cada fila de la tabla completa (Paso 4) es candidata a convertirse en un `Requirement`: la columna "Mitigación Recomendada" es el texto del requisito, y las columnas "Tipo STRIDE" / "Componente / Activo" indican sobre qué componente de Aplicación o Tecnología aplica la relación **Influence**.

---

_Esta guía hace parte del Taller 5 de Evaluación de Seguridad con STRIDE — curso Arquitectura Empresarial, Universidad de La Sabana._

# 🛠️ Taller 5: Evaluación de Seguridad con STRIDE

## 🎯 Objetivo

Analizar los riesgos de seguridad en una parte crítica del sistema usando el marco STRIDE (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege).

---

## 📘 Guía paso a paso

Antes de empezar el análisis, revise la [**Guía Paso a Paso: Evaluación de Seguridad con STRIDE**](clase/guia_paso_a_paso_stride.md). Incluye las 6 categorías STRIDE explicadas, la metodología de 5 pasos (de un diagrama de flujo de datos a una tabla de riesgo priorizada), un ejemplo completo construido paso a paso sobre el flujo de acceso a cursos de EdukIT, **una práctica guiada de explotación real en [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)** para dejar de ver las amenazas solo en papel, una guía de reconocimiento pasivo autorizado para completar la tabla del cliente real con evidencia en vez de suposiciones, y una tabla de errores comunes.

### 🖼️ Versión visual: Anatomía de un Ataque

[`clase/anatomia-de-un-ataque.html`](clase/anatomia-de-un-ataque.html) es una página autocontenida (sin dependencias) que dibuja el DFD de EdukIT con las 6 amenazas STRIDE marcadas sobre el elemento exacto que atacan, un diagrama del mecanismo de cada categoría, y los 4 ataques de Juice Shop paso a paso. GitHub no la renderiza interactiva desde la vista de archivo; para verla:
- Descargue el archivo y ábralo con doble clic (funciona sin conexión, es HTML plano), o
- Pegue esta URL en [htmlpreview.github.io](https://htmlpreview.github.io/): `https://raw.githubusercontent.com/CesarAVegaF312/AREM-Taller_5_Seguridad/main/clase/anatomia-de-un-ataque.html`

## 🎓 Caso base de referencia: EdukIT (Plataforma de Educación Virtual)

EdukIT es una plataforma de educación en línea que ofrece cursos certificados para estudiantes en América Latina. Administra el acceso a contenido educativo, evaluaciones, interacción con docentes y pagos por suscripción. El sistema gestiona información sensible como historial académico, datos personales, medios de pago y actividad del usuario. Evaluar la seguridad de estos procesos a través de un marco como STRIDE permite anticipar amenazas como suplantación, filtración de datos o accesos no autorizados, y diseñar estrategias de mitigación alineadas con las necesidades de protección de la información educativa.

**Contexto:**
- EdukIT es una plataforma de educación online que permite a estudiantes acceder a cursos, a docentes subir contenidos y a administradores gestionar pagos y certificados.
- El sistema maneja información personal, pagos, calificaciones y control de acceso.

**Elementos sensibles para evaluar:**

- Acceso de estudiantes a cursos y materiales
- Publicación de contenidos por parte de docentes
- Procesamiento de pagos con terceros
- Almacenamiento de datos personales y notas académicas
- Roles: Estudiante, Docente, Administrador

---

## 🧪 Parte 1: Trabajo en Clase

Durante la clase se espera que el equipo:

Siga la metodología de 5 pasos de la [guía paso a paso](clase/guia_paso_a_paso_stride.md) para analizar un flujo crítico de EdukIT:

1. Seleccione uno de los flujos críticos del sistema EdukIT y dibuje su diagrama de flujo de datos (DFD).
2. Identifique los procesos, almacenes de datos y flujos a analizar.
3. Aplique las 6 categorías STRIDE sobre los elementos relevantes.
4. Evalúe el impacto y proponga una mitigación concreta para cada amenaza.
5. Priorice los hallazgos por riesgo y valide la tabla con la [checklist de autoevaluación](clase/guia_paso_a_paso_stride.md#7-checklist-de-autoevaluación-antes-de-entregar).
6. Complete al menos uno de los [4 retos prácticos en OWASP Juice Shop](clase/guia_paso_a_paso_stride.md#4-práctica-guiada-laboratorio-con-owasp-juice-shop) (login bypass por inyección SQL, manipulación de precio, acceso al carrito de otro usuario, o acceso al panel de administración) y relaciónelo con una fila de su tabla STRIDE.

Como referencia adicional durante el ejercicio, consulte [`clase/stride_analisis_ejemplos.xlsx`](clase/stride_analisis_ejemplos.xlsx): un banco de 10 amenazas STRIDE ya redactadas sobre distintos sistemas, útil para ver el nivel de detalle esperado en cada columna.

- Registre todo en una tabla editable (`tabla-stride-clase.xlsx`) y justifique sus hallazgos.
- Reciba retroalimentación del docente y registre avances en `clase/notas.md` (use la [plantilla de notas](plantillas/plantilla_notas.md)).

---

## 🧠 Parte 2: Aplicación al Cliente Real

Después de la clase, el equipo debe:

- Aplicar STRIDE sobre un proceso o componente crítico del sistema del cliente real, siguiendo los mismos 5 pasos de la metodología.
- Antes de adivinar los controles existentes, haga el [reconocimiento pasivo autorizado](clase/guia_paso_a_paso_stride.md#5-reconocimiento-pasivo-autorizado-para-el-cliente-real) (cabeceras HTTP, configuración TLS, mensajes de error, documentación de API expuesta) — **solo observación de lo público, nunca pruebas activas contra el sistema real sin autorización por escrito**.
- Elaborar una tabla con amenazas, impactos, controles propuestos y nivel de riesgo (`entrega/tabla-stride-cliente.xlsx`), usando la [plantilla oficial](plantillas/plantilla_analisis_stride.xlsx). Las columnas enriquecidas — **Escenario de Ataque**, **Controles de Seguridad Existentes**, **Responsable** y **Estado** — ahora son obligatorias al diligenciarla; no basta con describir la amenaza y su mitigación.
- Redactar el informe en `entrega/informe.md` usando la [plantilla de informe del taller](plantillas/plantilla_informe_taller.md); explicar el análisis de seguridad y las diferencias con el caso base.
- Investigar buenas prácticas de seguridad aplicadas al sector del cliente (educación, salud, logística, etc.), y registrar las fuentes en `entrega/referencias.md` con la [plantilla de referencias](plantillas/plantilla_referencias.md).

---

## 📁 Estructura esperada del repositorio

```text
taller-05-seguridad-stride/
├── README.md
├── clase/
│   ├── guia_paso_a_paso_stride.md      # Categorías STRIDE, metodología de 5 pasos y ejemplo guiado
│   ├── stride_analisis_ejemplos.xlsx   # Banco de 10 amenazas STRIDE de ejemplo (referencia adicional)
│   ├── tabla-stride-clase.xlsx
│   └── notas.md                        # Ver plantillas/plantilla_notas.md
├── entrega/
│   ├── tabla-stride-cliente.xlsx
│   ├── informe.md                      # Ver plantillas/plantilla_informe_taller.md
│   └── referencias.md                  # Ver plantillas/plantilla_referencias.md
└── plantillas/
    ├── plantilla_analisis_stride.xlsx  # Plantilla oficial (hojas Plantilla_STRIDE y Ejemplo_STRIDE)
    ├── plantilla_informe_taller.md
    ├── plantilla_notas.md
    └── plantilla_referencias.md
```

---

## ⚠️ Errores comunes

Antes de entregar, compare su tabla contra los errores más frecuentes (amenazas genéricas, categorías STRIDE omitidas, mitigaciones vagas, hallazgos sin priorizar, o hacer pruebas activas contra el cliente real sin autorización) documentados en la [sección 6 de la guía paso a paso](clase/guia_paso_a_paso_stride.md#6-errores-comunes-a-evitar).

## 📤 Entregables

- Tabla STRIDE aplicada al sistema del cliente
- Informe técnico de análisis de seguridad
- Referencias de buenas prácticas en ciberseguridad

---

## 📊 Rúbrica de Evaluación

| Criterio                            | Excelente (5)                                                           | Aceptable (3) / Insuficiente (1–2)                      |
|-------------------------------------|--------------------------------------------------------------------------|----------------------------------------------------------|
| Análisis STRIDE (caso base)         | Tabla clara con amenazas reales y mitigación justificada                | Análisis superficial o incompleto                       |
| Aplicación al cliente real          | Adaptación precisa del marco al sistema del cliente                     | Adaptación genérica o sin profundidad técnica           |
| Documentación e interpretación      | Informe bien estructurado con impacto y priorización                    | Informe pobre o sin análisis crítico                    |
| Investigación complementaria        | Buenas prácticas y normativas citadas según el dominio del cliente      | Sin referencias o desconectado del contexto             |

---

## ✅ Licencia

Este taller hace parte del curso de Arquitectura Empresarial - Universidad de La Sabana. Uso académico bajo licencia MIT.

# Definición del Problema — Tu Evento

> Antes de diseñar soluciones, el equipo debe estar alineado en el problema que resuelve.
> Este documento captura esa alineación.
> Fuentes: `README.md` del repositorio + `01-context/evidence/overview.md` + documentos de `02_Analisis_Software/` (encuestas y entrevistas en TuEvento-Docs).

---

## 1. El problema en una oración

**Los organizadores de eventos en Colombia** que **gestionan conciertos, ferias y conferencias con venta de entradas numeradas** tienen dificultades con **el control de aforo, la venta digital y el acceso sin papel** porque **no existen herramientas locales asequibles que integren diseño de venue, pagos colombianos (PSE/Nequi) y validación de ingreso**, resultando en **ventas duplicadas, colas en taquilla, entradas físicas falsificables y sin visibilidad del aforo en tiempo real**.

---

## 2. Usuarios afectados

### Segmento A — Organizador de eventos

| Campo | Descripción |
|-------|-------------|
| **Quién es** | Persona, empresa o institución que organiza eventos presenciales en Colombia con venta de entradas (conciertos, ferias, conferencias, obras de teatro) |
| **Qué intenta hacer** | Crear el evento, definir la distribución del lugar, vender entradas digitalmente y controlar quién ingresa |
| **Dolor principal** | No tiene una herramienta integrada: usa Excel para el aforo, WhatsApp para vender, y listas en papel para controlar la puerta |
| **Acceso a la plataforma** | Exclusivo desde la web (Vite + React) |

**Job-to-be-done:**
> **Cuando** necesito organizar un evento con asientos numerados,
> **quiero** diseñar el mapa del lugar, publicar la venta en línea con precios por zona y recibir pagos seguros con métodos colombianos,
> **para que** pueda gestionar todo el ciclo del evento sin depender de terceros ni de procesos manuales.

---

### Segmento B — Asistente al evento

| Campo | Descripción |
|-------|-------------|
| **Quién es** | Persona en Colombia que asiste a eventos y compra entradas de forma digital |
| **Qué intenta hacer** | Encontrar eventos de su interés, elegir su asiento, pagar con su método habitual (Nequi, PSE, tarjeta) y entrar sin complicaciones |
| **Dolor principal** | No puede elegir asiento específico con certeza, no sabe si el asiento está disponible en tiempo real, y debe llevar entrada física o impresa |
| **Acceso a la plataforma** | Web (Vite + React) y app móvil (Expo + React Native — Android e iOS) |

**Job-to-be-done:**
> **Cuando** quiero ir a un evento,
> **quiero** ver el mapa de asientos disponibles en tiempo real, elegir el mío y pagar con Nequi o PSE desde mi celular,
> **para que** tenga mi lugar asegurado y pueda ingresar solo con mi código QR.

---

### Segmento C — Administrador del sistema

| Campo | Descripción |
|-------|-------------|
| **Quién es** | Responsable técnico de la plataforma (en contexto formativo: el equipo CapySoft) |
| **Qué intenta hacer** | Aprobar organizadores, gestionar roles y supervisar la integridad de la plataforma |
| **Acceso a la plataforma** | Exclusivo desde la web con rol `ADMIN` |

---

## 3. Evidencia del problema

> Documentos de investigación disponibles en `TuEvento-Docs/03_Analisis_Software/Recoleccion de datos/`

| Tipo de evidencia | Fuente | Hallazgo principal |
|------------------|--------|-------------------|
| Encuesta de usuarios | `Análisis Encuesta TuEvento.docx` — TuEvento-Docs | Confirma necesidad de compra digital y selección de asientos (contenido no legible como texto plano — ver documento original) |
| Entrevista a usuarios | `Análisis Entrevista TuEvento.docx` — TuEvento-Docs | Identifica dolor en procesos manuales de taquilla y control de acceso (ver documento original) |
| Vigilancia tecnológica | `cuadro comparativo-Tu evento.xlsx` — TuEvento-Docs | Comparativa de plataformas existentes vs. Tu Evento (ver documento original) |
| README del proyecto | `TuEvento/README.md` | El sistema resuelve: diseño de venue, reserva de asientos, compra móvil, QR de ingreso, pagos Wompi |

> **Nota:** los documentos `.docx` y `.xlsx` de TuEvento-Docs contienen la evidencia primaria (encuestas e interviews realizadas al inicio del proyecto). Los hallazgos detallados están en esos archivos, no en texto plano accesible aquí.

---

## 4. Solución actual del usuario (y sus problemas)

| Solución actual | Limitaciones | Fricción estimada |
|----------------|-------------|------------------|
| Venta de boletería presencial o por WhatsApp | Sin control de aforo en tiempo real, sin selección de asiento, sin comprobante digital seguro | Alta — requiere personal en taquilla y en la puerta |
| Listas en papel o Excel para el aforo | Propensas a error, no actualizables en tiempo real, falsificables | Alta — demoras en la entrada y errores de doble venta |
| Plataformas internacionales (Eventbrite, etc.) | No soportan PSE ni Nequi; sin diseñador de venue local; costos en USD | Media-alta — barrera de pago para usuarios colombianos |
| Sin app de validación en puerta | Control de acceso manual con listas impresas | Alta — cuellos de botella en la entrada al evento |

---

## 5. Hipótesis de solución

**Creemos que** una plataforma web + móvil que integre:
- diseñador visual de venue (solo web),
- selector de asientos en tiempo real (SockJS),
- pasarela de pagos colombiana (Wompi: tarjeta, PSE, Nequi, efectivo),
- generación de QR único por entrada,
- y validación de ingreso por escaneo de QR

**para** organizadores y asistentes en Colombia,
**logrará** eliminar la doble venta de asientos, reducir las colas en taquilla y eliminar la dependencia de entradas físicas.

**Sabremos que tuvimos éxito cuando:**
- Un organizador pueda crear, configurar y publicar un evento completo sin ayuda técnica
- Un asistente pueda comprar su entrada y seleccionar asiento desde el móvil en menos de 3 minutos
- El sistema valide ingresos por QR sin errores de duplicados

---

## 6. Métrica North Star

> La métrica que mejor captura el valor entregado por el sistema.

**"Entradas compradas y validadas exitosamente por evento publicado"**

| Métrica complementaria | Descripción |
|-----------------------|-------------|
| Tasa de éxito de pago Wompi | % de transacciones completadas vs. intentadas |
| Tiempo promedio de selección de asiento | Desde que el usuario abre el mapa hasta que confirma |
| Conflictos de reserva simultánea | Número de dobles reservas del mismo asiento (target: 0) |
| Tiempo de validación QR en puerta | Desde escaneo hasta confirmación de ingreso |

---

## 7. Riesgos de la hipótesis

| Riesgo | Probabilidad | Impacto | Forma de validar |
|--------|-------------|---------|-----------------|
| Wompi puede rechazar transacciones en sandbox sin razón clara | Media | Alto | Pruebas exhaustivas en sandbox con todos los métodos de pago |
| SockJS puede no escalar a muchos usuarios simultáneos | Media | Alto | Prueba de carga con múltiples conexiones concurrentes |
| Organizadores prefieren herramientas conocidas (WhatsApp/Excel) | Media | Medio | Validar con usuarios piloto antes de la entrega final |
| El diseño del venue en web puede ser complejo de usar | Baja | Medio | Pruebas de usabilidad con mockups de Figma antes de implementar |

---

## 8. Fuera del alcance de este problema (no resolvemos)

Explícitamente definido en `01-context/evidence/scope.md`:

- Eventos virtuales o streaming — fuera del dominio de eventos presenciales
- Facturación electrónica (DIAN) — requiere certificación regulatoria
- Publicación oficial en Google Play Store / App Store — fuera del alcance formativo
- Chat en tiempo real entre organizador y asistentes — fuera del core
- Integración con otras pasarelas (PayU, MercadoPago) — Wompi es suficiente para el MVP colombiano
- Reportes avanzados / Business Intelligence — requiere stack adicional

---

## Correlaciones

- Visión del producto → `03-product/evidence/vision.md`
- Alcance detallado (qué sí y qué no) → `01-context/evidence/scope.md`
- Historias de usuario que implementan esta solución → `04-requirements/evidence/user-stories.md` *(pendiente de commit)*
- Encuestas y entrevistas originales → `TuEvento-Docs/03_Analisis_Software/Recoleccion de datos/`
- Vigilancia tecnológica → `TuEvento-Docs/03_Analisis_Software/Vigilancia tecnologica/`

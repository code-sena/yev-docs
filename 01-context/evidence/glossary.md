# Glosario del Proyecto — Tu Evento

> Diccionario oficial del proyecto. Si un término aparece en código, documentos o conversaciones del equipo, su definición autoritativa está aquí. En caso de ambigüedad, este documento gana.
>
> **Cómo usarlo:** antes de usar un término técnico o de negocio, búscalo aquí. Si no está, agrégalo con su definición. Si hay desacuerdo sobre una definición, discútelo en equipo y actualiza este documento.

---

## Términos de Negocio / Dominio

| Término | Definición en el contexto de Tu Evento | Sinónimos / Notas |
|---------|----------------------------------------|-------------------|
| **Evento** | Actividad pública o privada con fecha de inicio, fecha de fin, venue asignado, cupo máximo y precio. Tiene un ciclo de vida: Borrador → Publicado → En curso → Completado / Cancelado. | — |
| **Organizador** | Usuario con rol `ORGANIZER` que crea, configura y gestiona eventos. Requiere aprobación del administrador para operar. Solo puede usar la plataforma desde la web. | Promotor |
| **Asistente** | Usuario con rol `ATTENDEE` que explora eventos, compra entradas y asiste a los mismos. Puede usar la web o la app móvil. | Comprador, Espectador |
| **Administrador** | Usuario con rol `ADMIN` que gestiona el sistema a nivel global: aprueba organizadores, gestiona roles y supervisa operaciones. | Admin, Sysadmin |
| **Usuario anónimo** | Visitante sin cuenta registrada. Solo puede navegar y consultar eventos públicos en la web. No puede comprar ni reservar. | Visitante |
| **Venue** | Lugar físico donde se realiza el evento. Tiene nombre, dirección, ciudad, departamento, coordenadas geográficas y una distribución de espacio (layout). | Sede, Recinto, Local |
| **Layout** | Representación visual y estructurada de la distribución del venue: secciones, bloques y asientos. Se almacena en formato JSONB. Solo configurable desde la web. | Mapa del venue, Plano |
| **Sección** | División del venue con un tipo, capacidad total, asientos disponibles, precio y estado (activo/inactivo). Ejemplo: Zona VIP, Tribuna Norte, Piso General. | Zona, Área |
| **Bloque** | Agrupación de asientos dentro de una sección. Permite organizar la distribución espacial. | Bloque de asientos |
| **Asiento** | Unidad individual dentro de un bloque. Tiene tipo (regular o cortesía), estado (disponible, reservado, vendido, cortesía) y un historial de cambios de estado. | Silla, Puesto |
| **Entrada** | Comprobante digital que acredita el derecho de ingreso de un asistente a un evento. Incluye código QR único, fecha de expiración y estado del ciclo de vida. | Ticket, Boleta, Tiquete |
| **Reserva** | Acción por la cual un asistente selecciona y bloquea un asiento específico antes de completar el pago. | Booking |
| **Inscripción / Compra** | Proceso completo que finaliza cuando el pago es confirmado y la entrada es emitida. | — |
| **Código QR** | Código de respuesta rápida único generado por entrada que permite validar el ingreso del asistente en la puerta del evento. | QR, Código de acceso |
| **Check-in** | Proceso de validación del código QR de un asistente al momento de ingresar al evento. Cambia el estado de la entrada a "usado". | Validación de entrada, Escaneo QR |
| **Orden** | Registro de la intención de compra de un asistente, agrupa una o varias entradas antes de procesarse el pago. | Pedido, Order |
| **Pago** | Transacción financiera procesada a través de Wompi que confirma la compra de entradas. Puede ser con tarjeta, PSE, Nequi o efectivo. | Transacción |
| **Reembolso** | Devolución de dinero al asistente por cancelación de evento o solicitud aprobada. Gestionado a través de Wompi. | Devolución, Refund |
| **Billetera digital** | Saldo disponible dentro de la plataforma asignado a cada usuario. Puede recargarse, usarse para pagar entradas, retirarse o recibir reembolsos. | Wallet, Saldo |
| **Notificación** | Mensaje enviado al usuario a través de un canal (email, push) para informar sobre eventos, compras, recordatorios o alertas del sistema. | Alerta, Aviso |
| **Categoría** | Clasificación jerárquica de eventos (padre → hijo). Ejemplo: Entretenimiento → Conciertos → Rock. | Tag, Etiqueta |
| **Cupo** | Cantidad máxima de asistentes permitidos en un evento o sección. Determinado por la capacidad del venue o la sección. | Capacidad, Aforo |
| **Petición de organizador** | Solicitud enviada por un usuario para obtener el rol de organizador. Debe ser aprobada o rechazada por el administrador. | Solicitud de organizador |

---

## Términos Técnicos del Proyecto

| Término | Definición |
|---------|-----------|
| **JWT (JSON Web Token)** | Token estándar usado para autenticar y autorizar usuarios. Contiene claims (datos del usuario y rol) firmados digitalmente. En Tu Evento usa algoritmo HS256 con expiración de 24 horas. |
| **Refresh Token** | Token de larga duración usado para obtener un nuevo JWT sin que el usuario vuelva a iniciar sesión. Se rota en cada uso para mayor seguridad. |
| **OAuth** | Protocolo de autorización que permite al usuario autenticarse usando una cuenta de terceros (ej. Google) sin compartir su contraseña con Tu Evento. |
| **API Gateway** | Punto de entrada único al sistema que enruta las solicitudes a los módulos correspondientes y aplica control de acceso por rol. |
| **DDD (Domain-Driven Design)** | Enfoque de diseño de software que organiza el código alrededor del dominio del negocio. El código refleja el lenguaje y las reglas del negocio. |
| **Arquitectura N-Layer** | Organización del código en capas: `domain` → `application` → `infrastructure` → `interfaces`. Cada capa tiene una responsabilidad específica y depende solo de la capa inferior. |
| **Microservicio** | Servicio independiente con responsabilidad única, proceso propio y base de datos propia. Tu Evento está orientado hacia esta arquitectura. |
| **JSONB** | Tipo de dato de PostgreSQL para almacenar JSON de forma binaria indexable. Usado para almacenar el layout del venue de forma flexible. |
| **DTO (Data Transfer Object)** | Objeto que transporta datos entre capas (ej. de `application` a `interfaces`). No contiene lógica de negocio. |
| **Mapper** | Componente que convierte entidades de dominio a DTOs y viceversa. Evita exponer el modelo de dominio directamente en la API. |
| **Use Case** | Clase que encapsula una operación de negocio específica (ej. `CreateEventUseCase`). Coordina la lógica entre el dominio y la infraestructura. |
| **Puerto (Port)** | Interfaz que define cómo el dominio se comunica con el exterior (entrada: `in` / salida: `out`). Parte del patrón Hexagonal / Puertos y Adaptadores. |
| **Webhook** | Notificación HTTP enviada por Wompi al sistema cuando ocurre un evento de pago (confirmado, fallido, reembolsado). Registrado en `transaction_webhook`. |
| **SockJS** | Librería de WebSockets usada en Tu Evento para el selector de asientos en tiempo real. Mantiene sincronía entre todos los usuarios conectados al mismo evento con latencia < 50ms. |
| **Circuit Breaker** | Patrón que detiene las llamadas a un servicio que está fallando para evitar cascadas de error. |
| **Idempotencia** | Propiedad de una operación que produce el mismo resultado sin importar cuántas veces se ejecute con los mismos parámetros. Crítico en pagos y webhooks. |
| **CI/CD** | Integración Continua / Entrega Continua. Pipeline automatizado que compila, prueba, analiza calidad y despliega el sistema. Implementado con Jenkins. |
| **SonarQube** | Herramienta de análisis estático de calidad del código. Detecta bugs, vulnerabilidades y deuda técnica. Quality gate del proyecto: >80% cobertura, 0 bugs críticos. |
| **Docker** | Plataforma de contenedores que empaqueta el backend y sus dependencias para ejecutarse de forma consistente en cualquier entorno. |
| **Kubernetes** | Sistema de orquestación de contenedores. Gestiona el despliegue, escalado y disponibilidad del backend en producción. |
| **MinIO** | Servicio de almacenamiento de objetos compatible con S3. Usado localmente en el entorno de desarrollo para simular AWS S3. |
| **Push Notification** | Notificación enviada directamente al dispositivo móvil del usuario (Android/iOS) sin necesidad de que la app esté abierta. |
| **SMTP** | Protocolo para envío de correos electrónicos. Usado para notificaciones transaccionales (confirmación de compra, recuperación de contraseña). |

---

## Términos de Metodología y Gobernanza

| Término | Definición en el contexto de Tu Evento |
|---------|----------------------------------------|
| **Tech Lead** | Rol técnico del equipo responsable de las decisiones de arquitectura, estándares de código y cumplimiento de gobernanza. En Tu Evento: Cristofer David Lozano Contreras. |
| **Scrum Master** | Rol que facilita las ceremonias ágiles (planning, daily, review, retro), remueve bloqueos y asegura que el equipo siga el proceso Scrum. En Tu Evento: Jhampier Santos Ortiz. |
| **Product Owner (PO)** | Responsable de definir y priorizar el backlog del producto, validar historias de usuario y aprobar cambios de alcance. Rol institucional: SENA — Sede Industria y los Servicios. Rol interno del equipo: Cristofer David Lozano Contreras. |
| **Sprint** | Ciclo de desarrollo de duración variable (1–4 semanas según complejidad del módulo) en el que el equipo construye un incremento funcional del sistema. |
| **Story Points** | Unidad relativa de estimación del esfuerzo para completar una historia de usuario. En Tu Evento se usa escala: Trivial (1-2 días), Simple (3-5 días), Medio (1-2 sem), Complejo (3-4 sem). |
| **Backlog** | Lista priorizada de historias de usuario y tareas técnicas pendientes. Gestionado en Trello: https://trello.com/b/QqBfrvr5/tuevento-capysoft. |
| **DoD (Definition of Done)** | Checklist que debe cumplir toda historia de usuario para considerarse terminada. Incluye código revisado, tests pasando, CI/CD verde y documentación actualizada. |
| **DoR (Definition of Ready)** | Checklist que debe cumplir una historia de usuario antes de entrar al sprint. Incluye criterios de aceptación en formato Dado/Cuando/Entonces, estimación y dependencias resueltas. |
| **SDD (Software Design Documentation)** | Metodología de documentación estructurada usada junto a Scrum en Tu Evento. Organiza la documentación del proyecto en carpetas numeradas por fase del ciclo de vida del software. |

---

## Acrónimos

| Acrónimo | Significado |
|----------|-------------|
| RF | Requisito Funcional |
| RNF | Requisito No Funcional |
| SRS | Software Requirements Specification (Especificación de Requisitos de Software) |
| JWT | JSON Web Token |
| API | Application Programming Interface |
| REST | Representational State Transfer |
| CRUD | Create, Read, Update, Delete |
| DTO | Data Transfer Object |
| DDD | Domain-Driven Design |
| CI/CD | Continuous Integration / Continuous Delivery |
| QR | Quick Response (código de respuesta rápida) |
| MVP | Minimum Viable Product (Producto Mínimo Viable) |
| ADSO | Análisis y Desarrollo de Software (programa SENA) |
| SENA | Servicio Nacional de Aprendizaje |
| PO | Product Owner |
| TL | Tech Lead |
| SM | Scrum Master |
| DoD | Definition of Done (Definición de Terminado) |
| DoR | Definition of Ready (Definición de Listo) |
| SDD | Software Design Documentation |
| UX | User Experience (Experiencia de Usuario) |
| UI | User Interface (Interfaz de Usuario) |
| PSE | Pagos Seguros en Línea (método de pago bancario colombiano) |
| ORM | Object-Relational Mapping |
| JPA | Jakarta Persistence API |
| PR | Pull Request |
| ADR | Architecture Decision Record |

---

## Roles del sistema (valores de enumeración)

| Valor en código | Descripción |
|----------------|-------------|
| `ADMIN` | Administrador del sistema |
| `ORGANIZER` | Organizador de eventos |
| `ATTENDEE` | Asistente a eventos |
| `ANONYMOUS` | Usuario sin autenticar |

## Estados de una entrada (Ticket)

| Estado | Descripción |
|--------|-------------|
| `PENDING` | Entrada generada, pago aún no confirmado |
| `ACTIVE` | Pago confirmado, entrada válida para ingreso |
| `USED` | Entrada escaneada y validada en el evento |
| `CANCELLED` | Entrada cancelada (reembolso solicitado o evento cancelado) |
| `EXPIRED` | Entrada no usada y su fecha de expiración ha pasado |

## Estados de un evento

| Estado | Descripción |
|--------|-------------|
| `DRAFT` | Borrador, no visible para el público |
| `PUBLISHED` | Publicado y disponible para compra |
| `IN_PROGRESS` | Evento en curso (entre fecha inicio y fin) |
| `COMPLETED` | Evento finalizado |
| `CANCELLED` | Evento cancelado por el organizador o el admin |

---

## Correlaciones

- Descripción general del sistema → `01-context/evidence/overview.md`
- Alcance del proyecto → `01-context/evidence/scope.md`
- Historias de usuario → `02_Especificacion_Requisitos/Historias_Usuario_TuEvento.docx`
- Modelo de datos → `05_Diseno_Software/Diccionario de datos Tu-Evento.xlsx`
- Gobernanza del equipo → `00-governance/evidence/`

# Sistema de Automatización de Reservas con Agentes IA — Sándalo Spa

Ecosistema de automatización conversacional desarrollado para **Sándalo Spa** (ubicado en Chelsea / East Boston) a través del agente conversacional **LIA**. 

El sistema administra el ciclo de vida de las reservas: envía confirmaciones iniciales destacando el estado del depósito (pendiente o pagado), programa recordatorios automáticos 24 horas antes de la cita para confirmar, cancelar o reagendar, analiza la disponibilidad de los terapeutas en tiempo real y notifica inmediatamente a los profesionales sobre cualquier actualización en su agenda.

## Stack Tecnológico

* **CRM & Hub de Mensajería:** Chatwoot (actúa como CRM y puente de comunicación entre la API de WhatsApp Business / Meta y n8n).
* **Orquestador de Flujos:** n8n (desplegado en servidor VPS Contabo).
* **Motor de Inteligencia Artificial (LLM):** OpenAI (Modelo GPT-4.1 Mini para análisis conversacional y validación de horarios).
* **Gestión de Memoria y Estado:** Redis (almacenamiento temporal del historial de mensajes por sesión).
* **Gestión de Citas:** Google Calendar API (consulta de agendas por profesional, verificación de huecos libres y actualización de reservas).
* **Persistencia Auxiliar:** Google Sheets (control y limpieza de registros de agendamiento).

## Funcionalidades Principales

* **Notificación de Agendamiento e Información de Depósito:** Envío automático de confirmación al agendar la cita, indicando explícitamente si el pago del depósito se encuentra pendiente o completado.
* **Recordatorio Preventivo a las 24 Horas:** Automatización que contacta al usuario 24 horas antes de su sesión para permitirle confirmar asistencia, reagendar o cancelar la cita.
* **Reagendamiento Dinámico:** Análisis en tiempo real de las ventanas de atención de cada profesional para proponer bloques libres y validar horas de inicio dentro de rangos amplios.
* **Notificación a Profesionales:** Envío de alertas automáticas al equipo de terapeutas/profesionales cada vez que su agenda sufre una modificación, cancelación o reasignación.

## Arquitectura y Flujo de Datos

1. **Recepción y Puente de Mensajería:** El cliente escribe por WhatsApp Business (Meta API) $\rightarrow$ Chatwoot recibe la interacción como CRM $\rightarrow$ Activa el Webhook hacia **n8n**.
2. **Reconstrucción de Contexto:** n8n consulta el historial de mensajes guardado en Redis, invierte el arreglo con JavaScript (`.reverse()`) para ordenar la secuencia cronológicamente y arma el contexto de la conversación.
3. **Procesamiento de IA (OpenAI GPT-4.1 Mini):** El modelo clasifica la intención (Confirmar, Reagendar, Cancelar, Saludo, etc.) y evalúa el cumplimiento de las reglas de negocio (depósitos, límites de 24h).
4. **Validación de Agenda en Tiempo Real:** En solicitudes de reagendamiento, n8n consulta la disponibilidad del profesional en Google Calendar y la IA valida matemáticamente que la duración del servicio encaje dentro del bloque libre sin generar solapamientos.
5. **Ejecución y Notificaciones:**
   * Se actualiza la cita en Google Calendar.
   * Se envía la respuesta al cliente a través de Chatwoot $\rightarrow$ WhatsApp.
   * Se notifica al profesional correspondiente sobre la actualización de su horario.

## Cómo ver e importar el flujo

1. Clona este repositorio o descarga el archivo `.json` ubicado en la raíz del proyecto.
2. Abre tu instancia de **n8n**.
3. Importa el archivo JSON mediante la opción **Import from File**.
4. Vincula tus credenciales de **Chatwoot**, **OpenAI**, **Google Calendar**, **Redis** y **Google Sheets**.

Cómo ver el flujo: Archivos .json de n8n adjuntos para importar el flujo y ver la lógica de los nodos.

  


ChatBotSandaloProd.json
Notificacion Creacion De Cita.json
Notificación 24h antes.json
tool_cancelar.json
tool_detalles_cita.json
tool_disponibilidad.json
tool_reagendar.json
tool_verificacion_12h.json


**Sección de Control de Errores y Handover Humano:**

## Manejo de Errores y Handoff a Agentes Humanos

El sistema incluye mecanismos de contingencia automatizados:

* **Fallo de IA / Tools / nodos:** Si el modelo no logra clasificar la intención tras dos intentos, el flujo etiqueta la conversación en Chatwoot como `atención_humana` y desactiva temporalmente las respuestas automáticas.
* **Contacto Directo:** Si el usuario solicita hablar con una persona o las políticas de agendamiento superan el límite establecido, el bot responde con los números telefónicos directos de soporte en inglés y español.

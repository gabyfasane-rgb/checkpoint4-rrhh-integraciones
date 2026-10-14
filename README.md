# Sincronización del Cerebro Agéntico con Ecosistemas de Negocio

**Preentrega 4 — IA Automation Avanzado**  
**Autora: Gabriela Fasanella**

## Objetivo

Extender el asistente multiagente de RRHH desarrollado en las entregas anteriores, conservando el Manager, los Workers y la memoria persistente en Airtable, e incorporando HubSpot, Gmail y Slack mediante OAuth2.

## Funcionalidades

El workflow recibe solicitudes desde tres canales:

- **Chat:** pedidos internos de RRHH.
- **Gmail:** correos de prueba filtrados por el asunto PRUEBA-M4.
- **Formulario HTML:** recepción de nombre, apellido y email de candidatos mediante un Webhook.

El Manager clasifica las solicitudes y las deriva a:

| Worker | Función |
|---|---|
| PERFIL_PUESTO | Elaborar un borrador de perfil de puesto e identificar datos faltantes. |
| PREPARAR_INGRESO | Verificar los datos de ingreso de una persona seleccionada. |
| REGISTRAR_POSTULACION | Buscar al candidato por email en HubSpot y crear o actualizar su contacto. |

Las solicitudes ambiguas o fuera del alcance se derivan a revisión humana.

## Integraciones y controles

### HubSpot

- Conexión OAuth2.
- Búsqueda exacta por email antes de crear un contacto.
- Si existe, actualiza nombre y apellido.
- Si no existe, crea el contacto.
- Los errores técnicos no se interpretan como contacto inexistente.

### Gmail

- Conexión OAuth2.
- IF inmediatamente después del trigger para detener correos con asuntos Auto-reply, Out of office o Undeliverable, y remitentes no-reply o noreply.
- Recuperación del cuerpo completo y validación de los datos de entrada.
- Salida configurada como **Create Draft**: el correo queda en Borradores para revisión y envío manual.

### Slack

- Conexión OAuth2.
- Aviso con identificador de solicitud, tipo, estado y confirmación de creación del borrador.
- Un nodo Edit Fields conserva únicamente el texto del aviso antes de enviarlo, sin historial, datos del candidato ni archivos adjuntos.

### Validación y errores

- El formulario valida nombre, apellido y formato básico del email.
- Los datos inválidos reciben una respuesta HTTP 400 y no llegan al CRM.
- Las postulaciones por chat o email se derivan a revisión humana; el registro automático de candidatos se limita al formulario validado.
- La llamada al Worker 3 tiene una salida de error que continúa hacia el registro de memoria y las notificaciones.

## Memoria persistente

Airtable conserva el contexto por Session_ID, el estado del caso y el contador de intercambios.

- Chat: utiliza el identificador de sesión del Chat Trigger.
- Gmail: utiliza el threadId para recuperar el contexto de una misma conversación.
- Formulario: genera una sesión nueva por ejecución.

Se conserva el mecanismo del M3 que genera un resumen automático cuando el contador supera cinco intercambios.

## Pruebas realizadas

- Solicitud de perfil desde chat con memoria y borrador.
- Recepción del formulario y confirmación desde n8n.
- Creación de un contacto nuevo en HubSpot.
- Actualización de un contacto existente conservando su ID.
- Postulación desde formulario integrada al Manager, Worker 3 y memoria.
- Solicitud por Gmail procesada por PERFIL_PUESTO.
- Bloqueo de un correo de prueba con asunto Auto-reply.
- Derivación de una postulación por chat a revisión humana.
- Publicación del aviso en Slack.

La rama de error del Worker 3 fue configurada y revisada, pero no se realizó una prueba final con un fallo controlado.

## Archivos

- checkpoint4_gabriela_fasanella.json: workflow principal.
- JSON del Worker 1_PERFIL_PUESTO.
- JSON del Worker 2_PREPARAR_INGRESO.
- JSON del Worker 3_REGISTRAR_POSTULACION.
- formulario-postulacion-m4.html: formulario de prueba.

- 
## Evidencia de conexiones

Las siguientes capturas muestran las credenciales configuradas en n8n y el estado de conexión de HubSpot (CRM), Gmail y Slack.

- [Vista general de Credentials en n8n](<n8n Credentials.png>)
- [HubSpot: conexión activa](<HubSpot Connection.png>)
- [Gmail: conexión activa](<Gmail- Connection.png>)
- [Slack: conexión activa](<Slack Connection.png>)


## Cómo reproducirlo

1. Importar los tres Workers y el workflow principal en n8n.
2. Configurar las credenciales propias de OpenAI, Airtable, Google Sheets, Gmail, HubSpot y Slack, según los nodos utilizados.
3. Seleccionar los Workers importados en los nodos Execute Sub-workflow del principal.
4. Configurar las bases, tablas, hoja de referencia y canal de Slack del entorno de prueba.
5. Para probar el formulario, reemplazar su URL por la Test URL del Webhook del entorno.
6. Iniciar la ejecución desde el Webhook y enviar el formulario mientras n8n espera el evento.
7. Para probar Gmail, enviar un correo con PRUEBA-M4 en el asunto y obtener el evento de prueba.

Las credenciales deben configurarse en el entorno de quien lo ejecuta. Los JSON no incluyen datos de prueba fijados.

## Alcance y permisos

El proyecto fue probado con candidatos ficticios y cuentas destinadas a pruebas.

Las conexiones administradas de HubSpot y Slack solicitan permisos más amplios que las acciones utilizadas por el workflow. Limitar los nodos a determinadas operaciones no reduce los permisos OAuth2 concedidos. Esta limitación debe revisarse antes de utilizar el proyecto con datos reales.

El borrador actual es un registro interno dirigido a RRHH, no una respuesta automática al candidato.

## Evolución prevista

En esta versión, el formulario registra datos básicos del contacto. Todavía no recibe ni almacena el CV, ni vincula la postulación con una vacante.

En etapas posteriores se prevé incorporar CV y vacante, formalizar las vacantes a partir de perfiles elaborados con PERFIL_PUESTO y analizar la adecuación de los candidatos. Las recomendaciones y decisiones de selección mantendrán revisión humana.

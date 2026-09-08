# TutorBot

Sistema automatizado en n8n para la gestión de asesorías académicas. Conecta estudiantes con tutores a través de Telegram, usando un motor de asignación que cruza materia, disponibilidad de horario y estado de la tutoría en tiempo real, con Google Sheets como base de datos.

## Funcionalidad

TutorBot resuelve la coordinación manual de tutorías (correos y mensajes sueltos, cruces de horario, falta de trazabilidad) mediante una interfaz conversacional en Telegram y un motor de automatización en n8n que:

- Registra estudiantes y guía la solicitud de tutoría paso a paso (materia, fecha, tutor y horario disponible).
- Asigna automáticamente al tutor correcto validando disponibilidad en tiempo real, evitando doble reserva.
- Permite a los tutores gestionar su propia disponibilidad (ver agenda, agregar franjas libres, marcar franjas como ocupadas).
- Permite al estudiante consultar sus solicitudes y cancelar una tutoría, liberando el horario automáticamente.
- Envía reportes periódicos de actividad a coordinación por correo.
- Envía recordatorios automáticos de las citas próximas, tanto por Telegram como por correo.

## Tecnologías utilizadas

- **n8n** — motor de automatización y orquestación de los 7 workflows.
- **Telegram Bot API** — interfaz conversacional para estudiantes y tutores.
- **Google Sheets API** — base de datos del sistema (estudiantes, tutores, disponibilidad, tutorías, materias y sesiones).
- **Gmail API** — envío de reportes y recordatorios por correo.

## Estructura del repositorio

```
├── Capturas - flujos/          # Captura de cada workflow en el editor de n8n
├── Capturas funcionamiento/    # Captura del bot funcionando en Telegram y de los correos recibidos
├── Workflows/                  # Archivos .json de cada workflow, listos para importar en n8n
└── README.md
```

## Workflows

El sistema está compuesto por 7 workflows en n8n. El Workflow 0 es el enrutador central: recibe todos los mensajes de Telegram y, según la sesión guardada del usuario, delega en el workflow correspondiente.

### Workflow 0 — Router central
Recibe cada mensaje de Telegram, identifica si quien escribe es un estudiante o un tutor (o si es un usuario nuevo), y enruta la conversación al workflow correcto según la pantalla y el paso en el que se encuentre la sesión.

![Workflow 0 - Router](<Capturas%20-%20flujos/Workflow%200%20Router.png>)

### Workflow 1 — Registro de Estudiante
Wizard de registro: pide nombre y correo, valida el formato del correo, y crea al estudiante en la base de datos.

![Workflow 1 - Registro de Estudiante](<Capturas%20-%20flujos/Workflow%201%20Registro%20de%20Estudiantes.png>)

### Workflow 2 — Solicitud de Tutoría
Wizard de solicitud: el estudiante elige materia y fecha, el sistema busca tutores disponibles ese día, y al confirmar una opción se re-valida la disponibilidad en tiempo real (para evitar que dos estudiantes reserven el mismo cupo) antes de crear la tutoría, bloquear el horario y notificar a ambas partes.

![Workflow 2 - Solicitud de Tutoría](<Capturas%20-%20flujos/Workflow%202%20Solicitud%20de%20asesoria.png>)

### Workflow 3 — Disponibilidad de Tutor
Le permite al tutor ver su agenda, agregar una nueva franja libre (día, hora de inicio y hora de fin), o marcar una franja existente como ocupada.

![Workflow 3 - Disponibilidad de Tutor](<Capturas%20-%20flujos/Workflow%203%20Disponibilidad%20de%20tutor.png>)

### Workflow 4 — Cancelar Tutoría
El estudiante consulta sus tutorías activas, elige cuál cancelar, confirma, y el sistema libera automáticamente la franja horaria y notifica al tutor.

![Workflow 4 - Cancelar Tutoría](<Capturas%20-%20flujos/Workflow%204%20-%20Cancelar%20Tutorias.png>)

### Workflow 5 — Reporte de Asistencia
Corre automáticamente por horario (no requiere interacción del usuario). Lee toda la base de tutorías y envía por correo a coordinación un resumen del total de tutorías, desglosado por estado, por materia y por tutor.

![Workflow 5 - Reporte de Asistencia](<Capturas%20-%20flujos/Workflow%205%20Reporte%20de%20correo.png>)

### Workflow 6 — Recordatorio de Citas
Corre automáticamente cada cierto intervalo de tiempo. Revisa las tutorías próximas a realizarse y, si aún no se ha enviado su recordatorio, notifica al estudiante (por Telegram y por correo) y al tutor (por Telegram).

![Workflow 6 - Recordatorio de Citas](<Capturas%20-%20flujos/Workflow%206%20recordatorio%20de%20citas.png>)

## Funcionamiento

### Registro de estudiante
![Funcionamiento registro de estudiante](<Capturas%20funcionamiento/Funcionamiento%20registro%20estudiante.png>)

### Solicitud de tutoría
![Funcionamiento solicitud de tutoría](<Capturas%20funcionamiento/Funcionamiento%20solicitud%20de%20tutoria.png>)

### Disponibilidad de tutor
![Funcionamiento disponibilidad de tutor](<Capturas%20funcionamiento/Funcionamiento%20disponibilidad.png>)
![Funcionamiento disponibilidad de tutor - 2](<Capturas%20funcionamiento/Funcionamiento%20disponibilidad-2.jpg>)

### Cancelación de tutoría
![Funcionalidad cancelación](<Capturas%20funcionamiento/Funcionalidad%20cancelacion.png>)

### Reporte de asistencia por correo
![Funcionalidad reportes](<Capturas%20funcionamiento/Funcionaldad%20reportes.png>)

### Recordatorio de citas
![Funcionalidad recordatorio Telegram](<Capturas%20funcionamiento/Funcionalidad%20recordatorio%20telegram.png>)
![Funcionalidad recordatorio Gmail](<Capturas%20funcionamiento/Funcionalidad%20recordatorio%20gmail.png>)

## Base de datos (Google Sheets)

El sistema usa como base de datos el archivo de Google Sheets **TutorBot_DB**, con las hojas ESTUDIANTES, TUTORES, DISPONIBILIDAD, TUTORIAS, MATERIAS y SESSIONS.

Enlace al archivo: https://docs.google.com/spreadsheets/d/1tqZ7v554xgO0G0LUbOZ9ciNjXSj-K6WGXydWN9hmHek/edit

## Cómo importar los workflows

Cada archivo dentro de la carpeta `Workflows/` puede importarse directamente en n8n desde *Import from File*. Antes de ejecutarlos es necesario:

1. Configurar las credenciales de Telegram, Google Sheets y Gmail propias en cada nodo correspondiente.
2. Enlazar el spreadsheet de Google Sheets con el ID del archivo propio (o usar el compartido arriba).
3. En los nodos "Execute Workflow" del Workflow 0, seleccionar desde la lista el workflow real ya importado en la instancia.
4. Publicar primero los workflows 1 a 6, y por último el Workflow 0 (router).

## Autores

- Andrés Felipe Jiménez Ramírez
- Andrés Rueda
- Thomas

# Sidereus Nuncius — Centro de Gobernanza

Sitio estático (sin backend) que junta en un solo lugar:

- `index.html` — página de entrada, con enlaces a todo
- `convocatorias.html` — qué dice el estatuto sobre convocatorias, quórum y tipos de reunión
- `asambleas.html` — calendario de asambleas/reuniones pre-agendadas, agenda, confirmación de asistencia (RSVP) y quórum en vivo
- `direcciones.html` — Consejo Directivo, las 4 Direcciones, funciones y penalidades
- `documentos.html` — acceso al estatuto en PDF y demás documentos
- `actividades.html` — tu Mission Control existente (tablero operativo), integrado a la misma navegación

## 1. Cómo integrarlo a tu repo `sidereus-dashboard`

1. Copia **todas** las carpetas y archivos de este paquete a la raíz de tu repositorio (o a una subcarpeta si prefieres, ajustando las rutas).
2. Si ya tienes un `index.html` (el Mission Control), dos opciones:
   - **Recomendado:** deja el nuevo `index.html` (el hub) como página de entrada, y el Mission Control queda en `actividades.html` (ya está copiado en este paquete).
   - O invierte el orden si prefieres que Mission Control siga siendo la portada — en ese caso, renombra este `index.html` a `gobernanza.html` y ajusta los enlaces del `<nav>` en cada página.
3. Confirma que `data/tasks.csv` siga siendo tu fuente real de actividades (o que el fetch a `raw.githubusercontent.com/szarate88/sidereus-dashboard/main/data/tasks.csv` siga funcionando).
4. Haz commit y push. GitHub Pages servirá todo desde el mismo dominio — no necesitas configurar CORS porque `data/roles.csv` y `data/asambleas.csv` se leen con rutas relativas.

## 2. Cómo editar el calendario de asambleas

Edita `data/asambleas.csv`. Columnas:

| Columna | Qué va ahí |
|---|---|
| `id` | Identificador corto único (ej. `A1`, `C3`, `T5`) |
| `tipo` | `Ordinaria`, `Extraordinaria`, `Consejo Directivo`, o `Reunión de Trabajo` |
| `modalidad` | `Presencial`, `Virtual`, o `Híbrida` — independiente del tipo |
| `titulo` | Nombre visible de la reunión |
| `fecha` | Formato `AAAA-MM-DD` |
| `hora` | Formato `HH:MM` |
| `lugar` | Virtual o dirección física |
| `convocatoria_dias` | Días mínimos de anticipación según estatuto (5 para Asamblea, 2 para Consejo, 0 para trabajo) |
| `agenda` | Puntos de agenda separados por `\|` (barra vertical) |
| `quorum_regla` | Texto libre citando la regla aplicable del estatuto |
| `estado_fecha` | `Confirmada` o `Tentativa` — controla si se muestra el badge ✓ o 〜 en la tarjeta |

El sitio calcula automáticamente cuántos titulares hacen falta para el quórum de Asambleas (más de la mitad del total de titulares en `data/roles.csv`). Las frecuencias mínimas del estatuto (1 Asamblea/año, Consejo cada 3 meses) **no limitan** cuántas filas puedes agregar — pueden programarse tantas sesiones extraordinarias y reuniones de trabajo como se necesiten. En `asambleas.html` se pueden filtrar por tipo (Asambleas / Consejo Directivo / Reuniones de Trabajo) para diferenciarlas de un vistazo.

### Escalamiento automático de tareas a la agenda

En `data/tasks.csv` hay una columna `Escalar` (vacía, `Consejo`, o `Asamblea`). Cualquier tarea marcada con un valor ahí, mientras su `Estado` no sea `Completado`, aparece automáticamente en la sección **"⬆ Escalado desde Actividades"** de la **próxima** sesión de Consejo Directivo o Asamblea (la más cercana en el calendario), sin que tengas que copiarla a mano en la columna `agenda`. Así el Consejo/la Asamblea solo ve lo que de verdad necesita su decisión — el resto se queda resolviéndose en las reuniones de trabajo de cada Dirección.

## 3. Cómo editar roles y Direcciones

Edita `data/roles.csv`. La columna `CargoEstatutario` debe usar exactamente: `Presidente`, `Vicepresidente`, `Secretario General`, `Tesorero`, o `Miembro` para el resto. La columna `Categoria` debe incluir la palabra "Titular" para que cuente en el cálculo de quórum.

## 4. Cómo activar "Proponer una reunión", la confirmación de asistencia (RSVP) y el quórum en vivo

Un sitio estático en GitHub Pages no puede guardar respuestas de varias personas por sí solo — para eso se necesita un lugar compartido donde todos los socios escriban. La forma más simple y gratuita, sin programar un backend: **dos Google Forms** (uno para proponer reunión, otro para confirmar asistencia).

### A) Formulario para proponer una reunión

1. Crea un Google Form con: Nombre, Tipo de reunión (Asamblea / Consejo Directivo / Reunión de Trabajo), Modalidad, Fecha y hora propuestas, Lugar, Tema o motivo.
2. Copia el enlace del formulario ("Enviar" → ícono de enlace).
3. En `asambleas.html`, dentro del bloque `CONFIG`, pégalo en `PROPOSE_MEETING_FORM_URL`.
4. Quien propone la reunión completa el formulario y avisa por WhatsApp al Secretario, al Vicepresidente y al Presidente. Ellos evalúan si se agenda y lo comunican a todos los socios por esa misma vía. Quien administra el sitio (tú) agrega la fila correspondiente en `data/asambleas.csv` una vez que la reunión queda confirmada.

### B) Formulario para confirmar asistencia (RSVP) + quórum en vivo

1. **Crea un Google Form** con estos campos:
   - Nombre completo (respuesta corta)
   - Reunión a la que asistirás (desplegable, con las mismas opciones que las columnas `titulo` de `data/asambleas.csv`)
   - ¿Asistirás? (opción múltiple: `Sí` / `No`)
2. En el Form, ve a **Respuestas → vincular a Google Sheets** (crea una hoja de cálculo nueva).
3. En esa Hoja de cálculo: **Archivo → Compartir → Publicar en la web**. Elige la pestaña de respuestas, formato **CSV**, y copia el enlace que te da.
4. En `asambleas.html`, dentro del bloque `CONFIG`, pega:
   ```js
   const CONFIG = {
     GOOGLE_FORM_URL: "https://forms.gle/TU-ENLACE-DE-ASISTENCIA",
     RSVP_SHEET_CSV_URL: "https://docs.google.com/spreadsheets/d/TU-ID/pub?output=csv",
     PROPOSE_MEETING_FORM_URL: "https://forms.gle/TU-ENLACE-DE-PROPUESTA"
   };
   ```
5. En el Google Sheet, si las columnas del Form no se llaman exactamente `IDReunion` y `Asistencia`, ajusta el JS de `asambleas.html` (función `loadRSVPCounts`) para que lea los nombres reales de tus columnas — o simplemente renombra los encabezados de la hoja para que coincidan.

Con esto, cada vez que alguien confirma en el Form, el contador de quórum en `asambleas.html` se actualiza solo (puede demorar 1-2 minutos en reflejarse, es normal en Google Sheets).

> Si por ahora no configuras los Forms, el sitio sigue funcionando igual — solo mostrará un aviso de que faltan conectar, y el calendario/quórum requerido se ve de todas formas.

### C) Google Calendar

Cada reunión ya tiene un botón **"+ Google Calendar"** que no necesita ninguna configuración — arma el enlace automáticamente a partir de la fecha, hora y lugar que están en `data/asambleas.csv`, y cada socio la guarda en su propio calendario personal con un clic. No sincroniza en la otra dirección (si alguien cambia la fecha en su Google Calendar, no se actualiza aquí) — sigue siendo `data/asambleas.csv` la fuente de verdad.

### D) Bitácora de notas y avances (para quienes no manejan GitHub/CSV)

Solo una persona necesita saber editar los `.csv` de este repositorio — el resto de socios, incluyendo quien toma notas en una reunión o marca el avance de una tarea, no debería necesitar tocar GitHub para nada. Para eso:

1. Crea un tercer Google Form, de uso general: Nombre, Tipo de registro (Nota de reunión / Avance de actividad / Otro), Reunión o actividad relacionada, Detalle.
2. Copia el enlace y pégalo en `BITACORA_FORM_URL` dentro del bloque `CONFIG` — tanto en `asambleas.html` como en `actividades.html` (usa el mismo enlace en los dos).
3. Las respuestas caen en una hoja de Google Sheets. Quien administra el sitio (por ahora, tú) revisa esa hoja cada cierto tiempo y traduce lo relevante a `data/tasks.csv`, `data/asambleas.csv`, o a un acta en `docs/actas/`.

Esto no automatiza la carga a los `.csv` — sigue siendo un paso manual — pero saca a todos los demás socios de la necesidad de entender Git, GitHub o CSV para poder registrar lo que están haciendo.

**Para verlas directamente en el sitio (sin entrar a Google Forms):**

1. En el formulario (pestaña "Respuestas" → ícono verde de Sheets), vincúlalo a una Hoja de cálculo nueva.
2. En esa Hoja: **Archivo → Compartir → Publicar en la web**. Elige la pestaña de respuestas, formato **CSV**, y copia el enlace.
3. Pégalo en `BITACORA_SHEET_CSV_URL` dentro del `CONFIG` de `actividades.html`. Ahí aparece una pestaña **"🗒️ Bitácora"** con todos los registros, más reciente primero — así puedes revisar lo que el equipo fue anotando en sus reuniones de trabajo, aunque no hayas estado presente, y de ahí actualizar `tasks.csv`.
4. Lo mismo aplica para el formulario de **"Proponer una reunión"**: publícalo como CSV y pégalo en `PROPOSE_SHEET_CSV_URL` dentro del `CONFIG` de `asambleas.html` — ahí verás las propuestas recibidas justo debajo del botón.

## 5. Documentos

### Documentos constitutivos (estatuto, testimonio notarial)

Sube esos PDFs a `docs/` en este repositorio. El estatuto firmado (`T0058555-signed.pdf`) ya viene incluido en este paquete. Esto lo maneja quien administra el sitio (tú) — no requiere que el Secretario toque GitHub.

### Actas de Asamblea y Consejo Directivo — carpeta de Google Drive (para el Secretario General)

El Secretario General no necesita saber usar GitHub ni CSV para archivar las actas. En vez de eso, `documentos.html` muestra una carpeta de Google Drive directamente en la página, donde él sube los PDFs arrastrándolos como a cualquier carpeta normal.

**Cómo configurarla (5 minutos):**

1. Crea una carpeta en tu Google Drive, por ejemplo "Actas Sidereus Nuncius".
2. Click derecho → **Compartir → Acceso general**: cambia a **"Cualquier persona con el enlace"**, con permiso de **Editor** (para que el Secretario pueda subir archivos sin pedir acceso cada vez). Si prefieres más control, comparte la carpeta solo con el correo del Secretario como Editor.
3. Copia el enlace de la carpeta — se ve así: `https://drive.google.com/drive/folders/1AbCdEfGhIjKlMnOpQrStUvWxYz`. La parte después de `/folders/` es el **ID de la carpeta**.
4. En `documentos.html`, busca el bloque `CONFIG` y pega ese ID en `DRIVE_ACTAS_FOLDER_ID`.
5. Listo — la página va a mostrar la carpeta embebida (se puede ver y abrir cada archivo sin salir del sitio) y un botón para abrirla directo en Drive.

El Secretario solo necesita: abrir `documentos.html`, hacer clic en "Abrir carpeta de actas en Drive", y arrastrar el PDF del acta ahí. No necesita cuenta de GitHub ni saber qué es un CSV.

> Esta carpeta es un respaldo digital práctico — no reemplaza el libro de actas físico/legalizado que exige el estatuto (Art. 12 y 30). Sigue siendo obligación del Secretario General llevar ese libro conforme a ley.

## 6. Filosofía del sitio

- El estatuto define **cuándo algo necesita ser formal** (Asamblea, Consejo Directivo) — el resto es ejecución libre vía reuniones de trabajo.
- Cada página está pensada para que cualquier socio, sin leer el estatuto completo, sepa: qué reunión toca, qué se va a decidir, quién es responsable, y qué tarea le corresponde en `actividades.html`.

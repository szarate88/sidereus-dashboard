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

El sitio calcula automáticamente cuántos titulares hacen falta para el quórum de Asambleas (más de la mitad del total de titulares en `data/roles.csv`). Las frecuencias mínimas del estatuto (1 Asamblea/año, Consejo cada 3 meses) **no limitan** cuántas filas puedes agregar — pueden programarse tantas sesiones extraordinarias y reuniones de trabajo como se necesiten. En `asambleas.html` se pueden filtrar por tipo (Asambleas / Consejo Directivo / Reuniones de Trabajo) para diferenciarlas de un vistazo.

### Escalamiento automático de tareas a la agenda

En `data/tasks.csv` hay una columna `Escalar` (vacía, `Consejo`, o `Asamblea`). Cualquier tarea marcada con un valor ahí, mientras su `Estado` no sea `Completado`, aparece automáticamente en la sección **"⬆ Escalado desde Actividades"** de la **próxima** sesión de Consejo Directivo o Asamblea (la más cercana en el calendario), sin que tengas que copiarla a mano en la columna `agenda`. Así el Consejo/la Asamblea solo ve lo que de verdad necesita su decisión — el resto se queda resolviéndose en las reuniones de trabajo de cada Dirección.

## 3. Cómo editar roles y Direcciones

Edita `data/roles.csv`. La columna `CargoEstatutario` debe usar exactamente: `Presidente`, `Vicepresidente`, `Secretario General`, `Tesorero`, o `Miembro` para el resto. La columna `Categoria` debe incluir la palabra "Titular" para que cuente en el cálculo de quórum.

## 4. Cómo activar la confirmación de asistencia (RSVP) y el quórum en vivo

Un sitio estático en GitHub Pages no puede guardar quién confirmó asistencia por sí solo — para eso se necesita un lugar compartido donde todos los socios escriban su respuesta. La forma más simple y gratuita, sin programar un backend:

### Paso a paso (10 minutos)

1. **Crea un Google Form** con estos campos:
   - Nombre completo (respuesta corta)
   - Reunión a la que asistirás (desplegable, con las mismas opciones que las columnas `titulo` de `data/asambleas.csv`)
   - ¿Asistirás? (opción múltiple: `Sí` / `No`)
2. En el Form, ve a **Respuestas → vincular a Google Sheets** (crea una hoja de cálculo nueva).
3. En esa Hoja de cálculo: **Archivo → Compartir → Publicar en la web**. Elige la pestaña de respuestas, formato **CSV**, y copia el enlace que te da.
4. En `asambleas.html`, busca el bloque `CONFIG` cerca del inicio del `<body>` y pega:
   ```js
   const CONFIG = {
     GOOGLE_FORM_URL: "https://forms.gle/TU-ENLACE-AQUI",
     RSVP_SHEET_CSV_URL: "https://docs.google.com/spreadsheets/d/TU-ID/pub?output=csv"
   };
   ```
5. En el Google Sheet, si las columnas del Form no se llaman exactamente `IDReunion` y `Asistencia`, ajusta el JS de `asambleas.html` (función `loadRSVPCounts`) para que lea los nombres reales de tus columnas — o simplemente renombra los encabezados de la hoja para que coincidan.

Con esto, cada vez que alguien confirma en el Form, el contador de quórum en `asambleas.html` se actualiza solo (puede demorar 1-2 minutos en reflejarse, es normal en Google Sheets).

> Si por ahora no quieres configurar el Form, el sitio sigue funcionando igual — solo mostrará un aviso de que falta conectar el RSVP, y las agendas/quórum requerido se ven de todas formas.

## 5. Documentos

Sube tus PDFs (estatuto, actas) a `docs/`. El estatuto firmado (`T0058555-signed.pdf`) ya viene incluido en este paquete.

## 6. Filosofía del sitio

- El estatuto define **cuándo algo necesita ser formal** (Asamblea, Consejo Directivo) — el resto es ejecución libre vía reuniones de trabajo.
- Cada página está pensada para que cualquier socio, sin leer el estatuto completo, sepa: qué reunión toca, qué se va a decidir, quién es responsable, y qué tarea le corresponde en `actividades.html`.

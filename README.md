# TalentAI – Resume Analysis (n8n)

Workflow de [n8n](https://n8n.io) que recibe hojas de vida por correo, verifica que realmente sean un CV, las compara con las vacantes activas usando IA y le entrega al reclutador un informe de compatibilidad.

> ⚠️ Los resultados son **orientativos** y no sustituyen la revisión humana. El workflow no toma decisiones de contratación.

## Cómo funciona

```
Gmail Trigger (cada minuto)
   └─ Get a message (descarga adjuntos)
        └─ If: ¿el adjunto es PDF?
             ├─ No  → Correo al candidato: "No pudimos procesar tu postulación"
             └─ Sí  → Extract from File (texto del PDF)
                        ├─ Get row(s): vacantes con estado = "Activa"
                        ├─ AI Agent1: clasifica el documento (CV / NO_CV / REVISAR)
                        │     ├─ CV      → Merge ─┐
                        │     ├─ NO_CV   → Correo al candidato: pide reenviar PDF válido
                        │     └─ REVISAR → Correo al reclutador: revisión manual
                        └─ Merge (CV + vacantes)
                              └─ AI Agent: evalúa compatibilidad y devuelve JSON
                                   └─ Code: parsea el JSON
                                        └─ Append row: guarda en la hoja "Evaluaciones"
                                             └─ Correo de confirmación al candidato
                                                  └─ Correo con el resumen al reclutador
```

Notas del diseño:

- Solo se procesa el **primer adjunto** (`attachment_0`) y debe ser **PDF**.
- Para clasificar se usan los primeros 10 000 caracteres del texto.
- El prompt de evaluación pide usar únicamente información del CV, marcar "No evidenciado" cuando falte un requisito y **no considerar** edad, género, foto, nacionalidad ni otras características personales.

## Requisitos

- Una instancia de n8n (cloud o self-hosted) con soporte para los nodos de LangChain (`@n8n/n8n-nodes-langchain`).
- Cuenta de **Gmail** (OAuth2) que recibirá las postulaciones.
- Cuenta de **Google Sheets** (OAuth2).
- **API key de Groq** (el modelo configurado es `openai/gpt-oss-120b`).

## Instalación

### 1. Crea la hoja de cálculo

Crea un Google Sheet con **dos pestañas** llamadas exactamente `Vacantes` y `Evaluaciones`.

**Pestaña `Vacantes`** (encabezados en la fila 1):

| Columna | Descripción |
|---|---|
| `cargo` | Nombre de la vacante |
| `descripcion` | Descripción del cargo |
| `lenguajes` | Lenguajes de programación requeridos |
| `frameworks` | Frameworks requeridos |
| `bases_datos` | Bases de datos requeridas |
| `experiencia` | Experiencia requerida |
| `ingles` | Nivel de inglés requerido |
| `requisitos_obligatorios` | Requisitos indispensables |
| `requisitos_deseables` | Requisitos deseables |
| `estado` | Solo se evalúan las filas con valor `Activa` |

**Pestaña `Evaluaciones`** (encabezados en la fila 1; el workflow agrega las filas):

`fecha_evaluacion`, `nombre_candidato`, `correo_candidato`, `cargo`, `compatibilidad`, `resumen`, `fortalezas`, `brechas`, `recomendacion`, `informe_completo`, `estado`

### 2. Importa el workflow

1. En n8n: **Workflows → Import from File** y selecciona `TalentAI_-_Resume_Analysis.sanitized.json`.
2. El workflow se importa **inactivo**.

### 3. Configura las credenciales

Crea y asigna estas credenciales (el JSON no incluye ninguna):

| Credencial | Nodos que la usan |
|---|---|
| Gmail OAuth2 | Gmail Trigger, Get a message, Send a message, Send a message1, Send a message2, Send a message3 |
| Google Sheets OAuth2 | Get row(s) in sheet, Append row in sheet |
| Groq API | Groq Chat Model, Groq Chat Model1 |

### 4. Reemplaza los placeholders

| Placeholder | Dónde | Qué poner |
|---|---|---|
| `TU_GOOGLE_SHEET_ID` | Nodos *Get row(s) in sheet* y *Append row in sheet* | El ID de tu hoja (la parte larga de la URL entre `/d/` y `/edit`) |
| `TU_CORREO_RECLUTADOR@ejemplo.com` | Nodos *Send a message1* y *Send a message3* | El correo que recibirá los avisos |

### 5. Prueba y activa

1. Agrega al menos una vacante con `estado = Activa`.
2. Envía a la cuenta de Gmail un correo con un CV en PDF y ejecuta el workflow manualmente.
3. Verifica que aparezca una fila en `Evaluaciones` y que lleguen los correos.
4. Cuando todo funcione, activa el workflow.

## Personalización

- **Modelo de IA:** cambia el modelo en los nodos *Groq Chat Model* y *Groq Chat Model1*.
- **Criterios de evaluación:** edita el prompt del nodo *AI Agent*.
- **Textos de los correos:** edita los nodos *Send a message* (confirmación), *Send a message2* (error) y los avisos al reclutador.
- **Frecuencia de lectura:** ajusta el intervalo en *Gmail Trigger* (actualmente cada minuto, máximo 5 mensajes).

## Limitaciones conocidas

- Con varias vacantes activas, el *Merge* combina por posición, por lo que el comportamiento puede no ser el esperado. Está pensado para pocas vacantes o una sola.
- Si el modelo no devuelve JSON válido, el nodo *Code in JavaScript* falla. Conviene añadir manejo de errores.
- Los PDF escaneados (imágenes sin texto) no se pueden leer sin OCR y terminarán como `REVISAR` o `NO_CV`.
- El correo de confirmación se envía a quien envió el mensaje, no al correo que aparece dentro del CV.

## Privacidad

Este workflow procesa datos personales de candidatos (CV, correos) y los envía a un proveedor externo de IA (Groq). Asegúrate de cumplir con la normativa de protección de datos aplicable (por ejemplo, la Ley 1581 de 2012 en Colombia) y de informar a los candidatos sobre el tratamiento de sus datos.

## Licencia

Define aquí la licencia que prefieras (por ejemplo, MIT).

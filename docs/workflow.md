# Documentación del Workflow de TalentAI

## 1. Introducción

El workflow de TalentAI está desarrollado en n8n y permite automatizar el procesamiento inicial de hojas de vida recibidas mediante correo electrónico.

El flujo integra Gmail, Google Sheets, procesamiento de documentos PDF e inteligencia artificial mediante Groq para clasificar documentos y evaluar la compatibilidad de los candidatos frente a una vacante configurada.

## 2. Descripción de los nodos

### 2.1. Gmail Trigger

**Función:** detectar la recepción de nuevos correos electrónicos.

Este nodo inicia el workflow cuando llega un mensaje que cumple las condiciones configuradas en Gmail.

### 2.2. Get a message

**Función:** obtener el contenido del correo electrónico y sus archivos adjuntos.

Permite recuperar la información del mensaje y descargar los documentos enviados por el candidato.

### 2.3. IF - Validación PDF

**Función:** verificar que el archivo adjunto sea un documento PDF.

El nodo comprueba la extensión y el tipo MIME del archivo.

- Si el archivo cumple las condiciones, continúa el procesamiento.
- Si no cumple las condiciones, no continúa por la ruta de evaluación de hojas de vida.

### 2.4. Extract from File

**Función:** extraer el contenido textual del documento PDF.

El texto obtenido se utiliza como entrada para el agente encargado de clasificar el documento.

### 2.5. AI Agent 1 - Clasificación del documento

**Función:** determinar si el documento recibido corresponde a una hoja de vida.

El agente utiliza un modelo de lenguaje para analizar el texto extraído y generar una clasificación en formato JSON.

Las posibles clasificaciones son:

- `CV`: el documento corresponde a una hoja de vida.
- `NO_CV`: el documento no corresponde a una hoja de vida.
- `REVISAR`: el documento requiere una revisión manual debido a la falta de información o ambigüedad.

### 2.6. IF 1 - Validación de clasificación

**Función:** dirigir el documento según el resultado generado por el primer agente.

- **True:** el documento fue clasificado como `CV` y continúa hacia la consulta de la vacante.
- **False:** el documento pasa al siguiente nodo de validación para determinar si corresponde a `NO_CV` o `REVISAR`.

### 2.7. Get row(s) in sheet

**Función:** consultar la configuración de la vacante en Google Sheets.

Este nodo obtiene los requisitos y características del puesto que se utilizarán durante la evaluación de compatibilidad.

La información puede incluir el cargo, descripción, tecnologías, experiencia, nivel de inglés y requisitos obligatorios o deseables.

### 2.8. Merge

**Función:** combinar la información del documento y la configuración de la vacante.

El nodo utiliza dos entradas:

- **Input 1:** información extraída del documento PDF.
- **Input 2:** información de la vacante obtenida desde Google Sheets.

El resultado permite que el siguiente agente tenga acceso a ambos conjuntos de información para realizar la evaluación.

### 2.9. AI Agent 2 - Evaluación de compatibilidad

**Función:** analizar la correspondencia entre el perfil del candidato y los requisitos de la vacante.

El agente utiliza el contenido de la hoja de vida y la configuración del puesto para generar una evaluación preliminar.

El análisis debe considerar únicamente criterios relacionados con el trabajo y basarse en la información disponible en el documento.

### 2.10. Code in JavaScript

**Función:** procesar y organizar la respuesta generada por el agente de evaluación.

Este nodo prepara los datos para su posterior almacenamiento en Google Sheets.

### 2.11. Google Sheets - Registro de evaluaciones

**Función:** almacenar los resultados del procesamiento.

El nodo agrega una nueva fila en la hoja destinada al registro de evaluaciones, permitiendo consultar posteriormente la información de los candidatos procesados.

### 2.12. Gmail - Notificaciones

**Función:** enviar comunicaciones relacionadas con el procesamiento de la hoja de vida.

El flujo contempla dos notificaciones principales:

- **Correo al candidato:** confirma la recepción de su hoja de vida.
- **Correo al responsable:** informa sobre el resultado del procesamiento y la evaluación.

### 2.13. IF 2 - Clasificación de documentos no identificados como CV

**Función:** diferenciar los documentos que no fueron clasificados como hojas de vida.

Este nodo evalúa el resultado de la clasificación inicial:

- **True (`NO_CV`):** continúa hacia el envío de una notificación de rechazo.
- **False (`REVISAR`):** continúa hacia el envío de una notificación para revisión manual.

## 3. Conexiones entre los nodos

La organización general del workflow es la siguiente:

```text
Gmail Trigger
     |
Get a message
     |
IF - Validación PDF
     |
Extract from File
     |
AI Agent 1 - Clasificación
     |
IF 1
     |
     +----------------------------+
     |                            |
   TRUE                         FALSE
     |                            |
Get row(s) in sheet              IF 2
     |                            |
   Merge                          +----------------------+
     |                            |                      |
AI Agent 2                     TRUE                   FALSE
     |                            |                      |
Code in JavaScript           NO_CV: rechazo       REVISAR:
     |                                            revisión manual
Google Sheets
     |
     +----------------------------+
     |                            |
Correo al candidato       Correo al responsable
```

El nodo Merge combina únicamente dos entradas: el contenido del PDF y la configuración de la vacante.

## 4. Rutas de ejecución

El workflow contempla tres escenarios principales:

### Escenario 1: documento identificado como CV

1. Se recibe el correo electrónico.
2. Se valida el archivo PDF.
3. Se extrae el texto del documento.
4. La inteligencia artificial identifica el documento como una hoja de vida.
5. Se consulta la configuración de la vacante.
6. Se combinan los datos del documento y la vacante.
7. Se realiza la evaluación de compatibilidad.
8. Se procesan y almacenan los resultados.
9. Se envían las notificaciones correspondientes.

### Escenario 2: documento identificado como NO_CV

1. Se recibe y procesa el documento.
2. Se extrae el texto, cuando el archivo es válido.
3. La inteligencia artificial determina que no corresponde a una hoja de vida.
4. Se envía una notificación de rechazo.

### Escenario 3: documento identificado como REVISAR

1. Se recibe y procesa el documento.
2. Se extrae el texto, cuando el archivo es válido.
3. La inteligencia artificial determina que no existe suficiente certeza para clasificarlo.
4. Se envía una notificación al responsable para que realice una revisión manual.

## 5. Consideraciones

- El workflow requiere credenciales válidas para Gmail, Google Sheets y Groq.
- La vacante debe estar configurada en la hoja de Google Sheets correspondiente.
- Los documentos deben cumplir las condiciones de validación para continuar por la ruta de procesamiento.
- La clasificación y evaluación dependen del contenido disponible en el PDF.
- Los resultados generados por inteligencia artificial son orientativos y no representan una decisión automática de contratación.
- Las credenciales y los datos sensibles deben mantenerse fuera del repositorio público.

## 6. Estado del workflow

El workflow forma parte del prototipo funcional de TalentAI y está diseñado para automatizar el procesamiento inicial de hojas de vida, desde su recepción por correo electrónico hasta el almacenamiento de los resultados y el envío de notificaciones.

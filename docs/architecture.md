# Arquitectura de TalentAI

## 1. Descripción general

TalentAI es una solución de automatización desarrollada en **n8n** que permite procesar hojas de vida recibidas por correo electrónico, identificar si los documentos corresponden a CV y realizar una evaluación preliminar de compatibilidad frente a una vacante configurada.

El sistema utiliza inteligencia artificial mediante modelos de lenguaje para clasificar documentos y analizar la correspondencia entre el perfil del candidato y los requisitos del puesto.

Los resultados se almacenan en Google Sheets y se envían notificaciones por correo electrónico a los candidatos y al responsable del proceso.

TalentAI está diseñado como un prototipo de apoyo a la selección inicial de candidatos. Sus evaluaciones son orientativas y no sustituyen la revisión ni la decisión de un responsable de selección.

## 2. Arquitectura tecnológica

El proyecto integra los siguientes componentes:

| Componente | Tecnología | Responsabilidad |
|---|---|---|
| Automatización | n8n | Orquestación y ejecución del flujo de trabajo. |
| Recepción de documentos | Gmail | Detectar correos y obtener los archivos adjuntos. |
| Extracción de información | Extract from File | Extraer el contenido de los documentos PDF. |
| Inteligencia artificial | Groq | Procesar los documentos mediante modelos de lenguaje. |
| Modelo de lenguaje | openai/gpt-oss-120b | Clasificar los documentos y evaluar la compatibilidad de los perfiles. |
| Configuración y almacenamiento | Google Sheets | Consultar vacantes y almacenar los resultados. |
| Procesamiento complementario | JavaScript | Organizar y preparar los resultados de la evaluación. |
| Notificaciones | Gmail | Enviar comunicaciones al candidato y al reclutador. |

## 3. Arquitectura general del sistema

TalentAI sigue una arquitectura basada en flujos de automatización. Cada componente ejecuta una tarea específica y transfiere la información al siguiente nodo.

El proceso general es el siguiente:

1. **Recepción:** Gmail detecta un nuevo correo y obtiene sus archivos adjuntos.
2. **Validación:** el sistema verifica que el archivo recibido sea un documento PDF.
3. **Extracción:** se obtiene el texto contenido en el documento.
4. **Clasificación:** un agente de inteligencia artificial determina si el documento corresponde a una hoja de vida.
5. **Consulta de vacante:** si el documento es clasificado como CV, se consulta la información de la vacante configurada en Google Sheets.
6. **Evaluación:** se analiza la compatibilidad entre la información del candidato y los requisitos del puesto.
7. **Procesamiento:** un nodo JavaScript organiza los datos obtenidos.
8. **Almacenamiento:** los resultados se registran en Google Sheets.
9. **Notificaciones:** se envían los correos correspondientes al candidato y al responsable del proceso.

## 4. Flujo de procesamiento

### 4.1. Recepción y validación del documento

El flujo comienza cuando Gmail detecta un nuevo mensaje. Posteriormente, se obtiene el correo y se procesa el archivo adjunto.

El sistema valida que el archivo tenga extensión PDF y que su tipo MIME corresponda a `application/pdf`.

Los documentos que no cumplen estas condiciones no continúan por la ruta de evaluación de hojas de vida.

### 4.2. Extracción de texto

Una vez validado el archivo, el nodo Extract from File obtiene el contenido textual del PDF.

Esta información se utiliza como entrada para el proceso de clasificación mediante inteligencia artificial.

### 4.3. Clasificación del documento

El primer agente de inteligencia artificial analiza el contenido extraído y determina una de las siguientes clasificaciones:

- **CV:** el documento corresponde a una hoja de vida.
- **NO_CV:** el documento no corresponde a una hoja de vida.
- **REVISAR:** no existe suficiente información para determinar con seguridad si es una hoja de vida.

Dependiendo del resultado:

- Los documentos clasificados como `CV` continúan hacia la evaluación.
- Los documentos clasificados como `NO_CV` generan una notificación de rechazo.
- Los documentos clasificados como `REVISAR` generan una notificación para revisión manual por parte del responsable.

### 4.4. Consulta de la vacante

Cuando el documento es identificado como una hoja de vida, el sistema consulta Google Sheets para obtener la configuración de la vacante correspondiente.

La información puede incluir:

- Nombre del cargo.
- Descripción del puesto.
- Lenguajes de programación.
- Frameworks.
- Bases de datos.
- Experiencia requerida.
- Nivel de inglés.
- Requisitos obligatorios.
- Requisitos deseables.
- Estado de la vacante.

Esta configuración permite modificar los criterios de evaluación sin necesidad de cambiar directamente la lógica del flujo.

### 4.5. Evaluación de compatibilidad

El segundo agente de inteligencia artificial recibe el texto de la hoja de vida y la información de la vacante.

Su función es analizar la correspondencia entre el perfil del candidato y los requisitos establecidos.

La evaluación debe basarse exclusivamente en información relacionada con el puesto de trabajo. No debe utilizar características personales irrelevantes ni inventar experiencia, conocimientos o certificaciones que no aparezcan en el documento.

El resultado constituye una evaluación preliminar y orientativa.

### 4.6. Procesamiento y almacenamiento

Después de la evaluación, un nodo JavaScript organiza la información generada para preparar los datos que serán almacenados.

Posteriormente, Google Sheets registra el resultado en la hoja destinada a las evaluaciones.

Esto permite conservar un registro de los documentos procesados y consultar los resultados posteriormente.

### 4.7. Notificaciones

Finalmente, el sistema utiliza Gmail para enviar las comunicaciones correspondientes:

- **Candidato:** recibe una confirmación de recepción de su hoja de vida.
- **Responsable del proceso:** recibe una notificación con la información y el resultado de la evaluación.

Las notificaciones también permiten mantener informado al responsable cuando un documento requiere revisión manual.

## 5. Integración y flujo de datos

Los componentes se comunican mediante las conexiones entre nodos de n8n.

La estructura principal del flujo es:

```text
Gmail Trigger
     |
Get a message
     |
Validación PDF
     |
Extract from File
     |
AI Agent 1
(Clasificación del documento)
     |
    If 1
     |
     +-----------------------------+
     |                             |
   TRUE                          FALSE
     |                             |
Consulta de vacante               If 2
     |                             |
   Merge                           +------------------+
(2 entradas)                       |                  |
     |                           TRUE               FALSE
AI Agent 2                         |                  |
(Evaluación)                  NO_CV: rechazo    REVISAR:
     |                                         revisión manual
JavaScript
     |
Google Sheets
     |
     +-----------------------------+
     |                             |
Correo al candidato         Correo al responsable
```

El nodo Merge utiliza dos entradas:

1. El contenido extraído del documento PDF.
2. La información de la vacante consultada en Google Sheets.

De esta manera, ambos conjuntos de información se reúnen antes de iniciar la evaluación de compatibilidad.

## 6. Almacenamiento y configuración

TalentAI utiliza Google Sheets para dos propósitos principales:

**Configuración de vacantes**

Contiene los requisitos y características de los puestos disponibles. Esta información sirve como referencia para la evaluación de cada candidato.

**Registro de evaluaciones**

Almacena los resultados generados durante el procesamiento de las hojas de vida.

Esta separación permite administrar las vacantes y consultar las evaluaciones sin modificar el flujo de automatización.

## 7. Consideraciones de seguridad y uso responsable

- Las credenciales de Gmail, Google Sheets y Groq deben configurarse directamente en la instancia de n8n.
- No se deben publicar credenciales, tokens, contraseñas ni datos personales de candidatos en el repositorio.
- Los archivos y la información de las hojas de vida deben manejarse de forma confidencial.
- La evaluación debe utilizar criterios relacionados con los requisitos del cargo.
- No se deben utilizar características personales irrelevantes para determinar la compatibilidad.
- Los resultados generados por inteligencia artificial son orientativos y deben ser revisados por una persona responsable.
- La clasificación automática no constituye una decisión definitiva de contratación.

## 8. Estado del proyecto

TalentAI se encuentra en desarrollo como prototipo funcional.

La arquitectura actual permite automatizar la recepción, validación, clasificación y evaluación preliminar de hojas de vida, así como el almacenamiento de resultados y el envío de notificaciones.

El proyecto puede ampliarse posteriormente con nuevas funcionalidades, criterios de evaluación y mecanismos de seguimiento del proceso de selección.

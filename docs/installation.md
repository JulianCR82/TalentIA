# Instalación y configuración de TalentAI

## 1. Introducción

Esta guía explica cómo instalar y configurar TalentAI para ejecutar el workflow de automatización de hojas de vida utilizando n8n.

Se describen los requisitos previos, la configuración de las integraciones y el proceso de importación del workflow.

## 2. Requisitos previos

Antes de comenzar, es necesario contar con:

- Docker Desktop.
- Una instancia de n8n en funcionamiento.
- Una cuenta de Gmail.
- Una cuenta de Google con acceso a Google Sheets.
- Una API Key de Groq.
- El archivo JSON del workflow de TalentAI.

### Tecnologías utilizadas

| Tecnología | Uso |
|---|---|
| Docker | Ejecución del entorno de n8n mediante contenedores. |
| n8n | Automatización del workflow. |
| Gmail | Recepción de hojas de vida y envío de notificaciones. |
| Google Sheets | Configuración de vacantes y almacenamiento de evaluaciones. |
| Groq | Acceso al modelo de inteligencia artificial. |
| JavaScript | Procesamiento de los resultados. |

## 3. Preparación del entorno

### 3.1. Instalar Docker Desktop

Descargar e instalar Docker Desktop desde su sitio oficial:

https://www.docker.com/products/docker-desktop/

Una vez instalado, iniciar Docker Desktop y verificar que se encuentre en ejecución.

### 3.2. Ejecutar n8n

TalentAI utiliza n8n como plataforma de automatización.

Si se utiliza Docker Compose, se puede iniciar el servicio desde la carpeta que contiene el archivo `compose.yaml` o `docker-compose.yml`:

```bash
docker compose up -d
```

Para verificar los contenedores:

```bash
docker compose ps
```

Cuando n8n esté disponible, acceder desde el navegador:

http://localhost:5678

**Nota:** estos comandos aplican cuando ya existe una configuración de Docker Compose para n8n. Si se utiliza otra instalación, se debe iniciar n8n mediante el método correspondiente.

## 4. Importar el workflow

1. Descargar el archivo `TalentAI-Resume-Analysis.json` desde la carpeta `workflows/` del repositorio.
2. Abrir la interfaz de n8n.
3. Acceder al menú de workflows.
4. Seleccionar la opción de importar desde archivo.
5. Elegir el archivo JSON descargado.
6. Verificar que todos los nodos y conexiones se hayan importado correctamente.

## 5. Configurar las credenciales

El workflow requiere configurar las credenciales de los servicios externos utilizados.

### 5.1. Gmail

Configurar las credenciales de Gmail en n8n para permitir:

- Detectar nuevos mensajes.
- Obtener el contenido de los correos.
- Descargar archivos adjuntos.
- Enviar notificaciones.

La autenticación debe realizarse mediante el mecanismo de conexión disponible en n8n para Gmail.

### 5.2. Google Sheets

Configurar las credenciales de Google Sheets para permitir consultar la configuración de las vacantes y registrar los resultados de las evaluaciones.

Se deben verificar los siguientes elementos:

- Acceso a la hoja de cálculo de configuración de vacantes.
- Acceso a la hoja de cálculo de resultados.
- Identificadores de los documentos de Google Sheets.
- Nombres correctos de las pestañas utilizadas por el workflow.

### 5.3. Groq

Obtener una API Key desde la plataforma oficial:

https://console.groq.com/

Después:

1. Abrir las credenciales de Groq en n8n.
2. Crear una nueva credencial.
3. Introducir la API Key.
4. Guardar la configuración.
5. Asociar la credencial con los nodos de inteligencia artificial.

El modelo utilizado en el prototipo es:

`openai/gpt-oss-120b`

La disponibilidad del modelo depende de la plataforma y de la configuración de la cuenta.

## 6. Configurar Google Sheets

TalentAI utiliza dos hojas de cálculo principales.

### 6.1. Configuración de vacantes

La hoja `Vacantes` contiene la información utilizada para evaluar los perfiles.

Entre los campos contemplados se encuentran:

- Cargo.
- Descripción.
- Lenguajes de programación.
- Frameworks.
- Bases de datos.
- Experiencia.
- Nivel de inglés.
- Requisitos obligatorios.
- Requisitos deseables.
- Estado de la vacante.

Se debe verificar que exista una vacante configurada y que su información corresponda a los criterios utilizados por el workflow.

### 6.2. Registro de evaluaciones

La hoja `Evaluaciones` almacena los resultados generados por el workflow.

Es necesario verificar que el nodo de Google Sheets esté configurado con el documento y la pestaña correctos.

## 7. Verificación de la configuración

Antes de ejecutar el workflow, comprobar los siguientes puntos:

- [ ] n8n se encuentra en funcionamiento.
- [ ] El workflow se importó correctamente.
- [ ] Las credenciales de Gmail están configuradas.
- [ ] Las credenciales de Google Sheets están configuradas.
- [ ] La API Key de Groq está configurada.
- [ ] Los nodos utilizan las credenciales correspondientes.
- [ ] Los identificadores de las hojas de cálculo son correctos.
- [ ] Existe una vacante configurada.
- [ ] Las conexiones entre los nodos están completas.

## 8. Pruebas de funcionamiento

Se recomienda realizar pruebas con documentos de ejemplo antes de utilizar TalentAI en un proceso real.

### Prueba 1. Hoja de vida válida

Enviar un correo con un documento PDF que contenga una hoja de vida.

**Resultado esperado:**

- El documento supera la validación.
- Se extrae el texto.
- Se clasifica como `CV`.
- Se consulta la vacante.
- Se realiza la evaluación.
- Se registra el resultado en Google Sheets.
- Se envían las notificaciones correspondientes.

### Prueba 2. Documento que no es una hoja de vida

Enviar un PDF cuyo contenido no corresponda a una hoja de vida.

**Resultado esperado:**

- El documento se procesa.
- Se clasifica como `NO_CV`.
- Se envía la notificación correspondiente.

### Prueba 3. Documento ambiguo

Enviar un PDF cuyo contenido no permita determinar con claridad si corresponde a una hoja de vida.

**Resultado esperado:**

- El documento se clasifica como `REVISAR`.
- Se envía una notificación al responsable para revisión manual.

## 9. Consideraciones importantes

- El archivo JSON del repositorio no debe contener credenciales ni claves privadas.
- Cada usuario debe configurar sus propias credenciales en n8n.
- Los identificadores de Google Sheets deben reemplazarse por los correspondientes a los documentos del usuario.
- Las configuraciones pueden variar según la versión de n8n y los métodos de autenticación disponibles.
- Se recomienda utilizar documentos de prueba sin información personal real durante la configuración inicial.
- Las evaluaciones generadas por inteligencia artificial son orientativas y requieren supervisión humana.

## 10. Solución de problemas

| Problema | Posible solución |
|---|---|
| n8n no abre en el navegador | Verificar que Docker esté ejecutándose y que el contenedor de n8n esté activo. |
| Error de autenticación de Gmail | Revisar las credenciales y permisos de la cuenta. |
| Error de Google Sheets | Verificar los permisos, el identificador del documento y el nombre de la pestaña. |
| Error de Groq | Comprobar la API Key, la disponibilidad del modelo y los límites de uso. |
| No se extrae texto del PDF | Verificar que el archivo sea válido y que contenga texto extraíble. |
| El workflow no continúa por una ruta esperada | Revisar las condiciones de los nodos IF y las salidas de los agentes de inteligencia artificial. |

## 11. Estado de instalación

TalentAI es un prototipo en desarrollo. La ejecución requiere configurar las integraciones externas y verificar las conexiones del workflow antes de utilizarlo.

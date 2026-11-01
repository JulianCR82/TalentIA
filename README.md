# TalentAI

**Automatización inteligente para la recepción y evaluación inicial de hojas de vida.**

TalentAI es un proyecto desarrollado con **n8n** que automatiza parte del proceso de reclutamiento mediante inteligencia artificial. Su objetivo es facilitar la recepción, clasificación y evaluación inicial de hojas de vida frente a los requisitos de una vacante.

## ¿Cómo funciona?

1. **Recepción de postulaciones:** detecta nuevos correos electrónicos en Gmail.
2. **Validación de documentos:** comprueba que el archivo adjunto esté en formato PDF.
3. **Extracción de información:** obtiene el texto de la hoja de vida.
4. **Clasificación con IA:** determina si el documento corresponde a una hoja de vida.
5. **Consulta de vacantes:** obtiene los requisitos de la vacante activa desde Google Sheets.
6. **Evaluación de compatibilidad:** utiliza inteligencia artificial para analizar la relación entre el perfil del candidato y los requisitos del cargo.
7. **Almacenamiento:** registra los resultados de la evaluación en Google Sheets.
8. **Notificaciones:** envía correos electrónicos al candidato y al responsable del proceso.

## Tecnologías utilizadas

- **n8n:** automatización y orquestación del flujo de trabajo.
- **Gmail:** recepción de postulaciones y envío de notificaciones.
- **Google Sheets:** configuración de vacantes y almacenamiento de evaluaciones.
- **Groq:** acceso al modelo de inteligencia artificial utilizado para el análisis.
- **JavaScript:** procesamiento y estructuración de los resultados.

## Estructura del repositorio

```text
TalentAI/
├── README.md
├── workflows/
│   └── TalentAI-Resume-Analysis.json
├── docs/
└── .gitignore
```

## Consideraciones

- Los requisitos de las vacantes se pueden modificar desde Google Sheets.
- La evaluación de compatibilidad es orientativa y no representa una decisión automática de contratación.
- El sistema debe evaluar exclusivamente criterios relacionados con el perfil profesional y los requisitos del cargo.
- Los documentos y datos personales de los candidatos deben manejarse de forma confidencial.

## Estado del proyecto

**Prototipo funcional en desarrollo.**

---

Desarrollado como proyecto de automatización e inteligencia artificial.

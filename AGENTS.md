# Diagramas del Proyecto SayIt

## assets/casos_uso.png — Diagrama de Casos de Uso

**Actor principal:** Estudiante

**Actor externo:** Azure Speech API

**Casos de uso dentro del sistema SayIt:**
| ID | Caso de Uso | Descripción |
|----|-------------|-------------|
| UC1 | Seleccionar texto | El estudiante navega el catálogo y elige un texto |
| UC2 | Realizar lectura | El estudiante inicia la sesión de lectura en voz alta |
| UC3 | Visualizar colores | El estudiante ve el feedback cromático en tiempo real |
| UC4 | Transcribir audio | El sistema envía el audio a Azure Speech API para transcripción |
| UC5 | Consultar reporte | El estudiante revisa métricas de precisión e historial |

**Relaciones clave:**
- Estudiante → UC1, UC2, UC3, UC5
- UC2 → UC4 (`<<include>>` — Realizar lectura siempre incluye Transcribir audio)
- UC4 → Azure Speech API

---

## assets/secuencia.png — Diagrama de Secuencia

**Participantes:** Usuario, Frontend (Interfaz), Lógica de Comparación, Azure Speech API

**Fases del proceso:**
1. **Inicio de sesión:** Usuario inicia → Frontend solicita micrófono → Usuario concede acceso
2. **Preparación:** Frontend solicita a Lógica preparar entorno con ID del Texto Base
3. **Bucle de evaluación en tiempo real:**
   - Usuario envía audio → Frontend transmite stream a API
   - API devuelve transcripción parcial → Frontend envía a Lógica para validación
   - Lógica retorna estado de palabra (acierto/error) → Frontend renderiza cambio de color
4. **Cierre:** Usuario finaliza → Frontend cierra sesión → Lógica procesa métricas finales → Frontend muestra reporte

---

## assets/bpmn.png — Diagrama BPMN (Activity Diagram)

**Actores / Swimlanes:** Estudiante, Sistema SayIt, API de Reconocimiento

**Flujo principal:**
| Paso | Actor | Acción |
|------|-------|--------|
| 1 | Estudiante | Seleccionar texto del catálogo |
| 2 | Estudiante | Clic en "Iniciar Lectura" |
| 3 | Sistema SayIt | Activar captura de audio |
| 4 | Estudiante | Leer texto en voz alta |
| 5 | Sistema SayIt | ¿Conectividad disponible? |
| 5a | Sistema (Sí) | Enviar fragmento de audio a API |
| 5b | API de Reconocimiento | Procesar y transcribir audio |
| 5c | Sistema SayIt (Sí) | Comparar transcripción (RF04) → Actualizar colores (RF05) |
| 6 | Estudiante | Clic en "Detener Lectura" |
| 7 | Sistema SayIt | Calcular métricas de precisión → Guardar registro de sesión |
| 5a-alt | Sistema SayIt (No) | Notificar error de conexión |

---

## assets/diagramas.txt — Diagrama de Clases

**Clases del modelo de dominio:**

| Clase | Tipo | Atributos Clave | Métodos Clave |
|-------|------|-----------------|---------------|
| `Usuario` | Clase | id, correoElectronico, nombreCompleto | iniciarSesion() |
| `SesionEvaluacion` | Clase | idSesion, fechaInicio, totalAciertos, totalErrores | calcularPorcentajePrecision(), generarReporteDesempeno() |
| `TextoEstudio` | Clase | idTexto, titulo, contenidoBase, palabras | obtenerPalabraActual(indice) |
| `AlgoritmoComparacion` | Clase | textoReferencia | evaluarCoincidenciaExacta(), sincronizarFlujo() |
| `MotorReconocimientoVoz` | Interfaz | — | iniciarStreamAudio(), detenerStreamAudio() |
| `ServicioAzureSpeech` | Clase | claveAcceso, regionDespliegue | establecerConexion(), convertirAudioATexto() |

**Relaciones:**
- `Usuario` 1 — * `SesionEvaluacion` (*ejecuta*)
- `SesionEvaluacion` * — 1 `TextoEstudio` (*requiere*)
- `SesionEvaluacion` 1 — 1 `AlgoritmoComparacion` (*delega análisis a*)
- `AlgoritmoComparacion` ..> `MotorReconocimientoVoz` (*consume*)
- `ServicioAzureSpeech` ..|> `MotorReconocimientoVoz` (*implementa*)

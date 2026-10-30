# Documenta — búsqueda y respuestas verificables sobre documentos

**Estado: propuesta, no implementada.** README inicial con contratos de instalación/pruebas por crear. No afirma precisión, reducción de tiempo ni un producto desplegado.

## Problema y valor para selección

Encontrar información en documentos públicos o propios y mostrar la fuente que respalda cada respuesta. Demuestra integración de IA con evaluación reproducible, permisos y análisis de errores, en lugar de un chat que solo devuelve texto convincente.

## Tecnologías

Node.js/TypeScript, PostgreSQL con pgvector, React y Docker. Interfaz de proveedor para embeddings y generación; comenzar con documentos TXT/Markdown propios y proveedor local opcional. Python opcional para evaluación. Seleccionar modelo, licencia, límites y coste al implementar; no se requiere inventar un proveedor ni enviar documentos privados a una API.

## Arquitectura y carpetas objetivo

```text
Carga autorizada → normalización → chunks con metadatos → embeddings → pgvector
Pregunta + identidad → filtro por propietario → recuperación → generación → citas
api/src/{ingestion,retrieval,generation,auth}/
web/src/features/{documents,chat,sources}/
evaluation/{fixtures,questions.jsonl,run.ts}/
tests/{unit,integration,e2e}/
infra/                 compose y configuración de modelos
docs/{screenshots,adr}/
```

Cada fragmento conserva documento y sección originales. Filtro de autorización antes de recuperar contenido. La respuesta debe poder decir «no tengo evidencia». Los documentos son datos no confiables: no convertir instrucciones del texto en instrucciones de sistema. Sin herramientas de ejecución ni acceso a secretos desde el modelo.

## Instalación objetivo

Requisitos futuros: Node 24, Docker, modelo de embeddings/generación con versión fijada. Definir recursos locales antes de descargar modelos grandes; CPU/memoria y coste por token condicionan el alcance.

```bash
# Solo una vez creados scripts y servicios:
cp .env.example .env
docker compose up -d --wait
npm ci
npm run db:migrate
npm run ingest -- --dir evaluation/fixtures
npm run dev
# Al implementar la evaluación:
npm run eval
```

Las variables de proveedor se documentarán por nombre; las claves se guardarán fuera de Git. Implementar modo stub para tests offline; no presentar esas pruebas como validación de calidad del modelo real. El modo real requiere credenciales si el proveedor elegido las exige.

## Pruebas y evaluación objetivo

```bash
npm test
npm run test:integration
npm run test:e2e
npm run eval
```

Casos: documento ausente, consulta sin evidencia, cita inexistente, instrucciones maliciosas dentro del documento, consulta de documento de otro usuario y reingesta sin duplicados. Un conjunto inicial de 20 preguntas etiquetadas es una meta de desarrollo, no un resultado obtenido. Medir recall@k con documentos relevantes etiquetados, tasa de citas resolubles, respuestas sin soporte y latencia p50/p95. Comparar al menos dos configuraciones de chunking con el mismo conjunto. Registrar modelo, versión, configuración y coste observado; no afirmar resultados hasta medirlos.

## Fases (8–10 h/semana)

1. Corpus autorizado, preguntas de evaluación y contrato OpenAPI, 8–12 h.
2. Ingesta, chunking, embeddings y recuperación, 14–20 h.
3. Respuestas con citas, abstención y React, 16–24 h.
4. Permisos, aislamiento y ataques de prompt injection, 12–18 h.
5. Evaluación, CI, Docker y documentación de límites, 12–18 h.

Total 62–92 h, unas 7–12 semanas. Antes de añadir OCR o PDF escaneados, comprobar TXT/Markdown. El coste de modelos no se incluye en la estimación de horas.

## Capturas y decisiones

Pendientes: respuesta con cita navegable, abstención y reporte real de evaluación. docs/screenshots con contenido ficticio o público autorizado. ADR: proveedor local/remoto, dimensión de embeddings, chunking, límite de contexto, política de retención y permisos. No capturar claves o datos personales.

## Evidencias y commits

`feat: preserve chunk provenance`, `test: deny cross-user retrieval`, `feat: return grounded citations`, `eval: compare chunk configurations`. GitHub Actions ejecutará tests deterministas sin claves; evaluación con modelo real es tarea aparte con coste y versiones documentados. Swagger/OpenAPI para carga, consulta y citas.

## Línea de CV al terminar

«Implementé un asistente RAG con TypeScript y PostgreSQL/pgvector, con respuestas enlazadas a documentos fuente y evaluación reproducible de recuperación». Añadir métricas solo tras obtenerlas y conservar el reporte.

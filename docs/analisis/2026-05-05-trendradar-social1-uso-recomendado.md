# TrendRadar para social1: uso recomendado como radar de candidatos

Fecha de analisis: 2026-05-05 21:08 CST
Repo analizado: /home/victor/dev/scrapers/TrendRadar

## Decision recomendada

Usar TrendRadar como radar upstream, no como scraper factual final ni publicador.

Flujo recomendado:

TrendRadar captura senales -> adaptador genera candidatos -> social1 valida -> social1 crea draft -> humano aprueba -> Postiz recibe draft.

No debe saltarse el pipeline social1.

## Por que no debe publicar directo

TrendRadar entrega principalmente:

- title
- url
- platform/feed
- rank
- first_crawl_time
- last_crawl_time
- crawl_count
- rank_history
- summary RSS si existe
- clasificacion/analisis AI si se activa

social1 necesita ademas:

- facts verificados
- evidencia durable
- imagen primaria valida
- dedupe editorial
- elegibilidad temporal
- novedad real
- tono/idioma/cuenta destino
- aprobacion antes de publicar

TrendRadar no cubre todo eso por si solo.

## Configuracion piloto segura

Objetivo: probar valor sin meter coste ni ruido.

1. Ejecutar solo servicio trendradar, no trendradar-mcp al inicio.
2. Apagar AI al principio:
   - filter.method: keyword
   - ai_analysis.enabled: false
   - ai_translation.enabled: false
3. Usar SQLite local:
   - storage.backend: local o auto en Docker/local
   - storage.formats.sqlite: true
   - storage.formats.html: true si queremos navegador
4. Frecuencia:
   - empezar con CRON_SCHEDULE=0 * * * *
   - subir a */30 * * * * solo si hay valor
5. No activar notificaciones publicas al inicio.
6. Exportar candidatos a una bandeja controlada por social1.

## Bandeja de candidatos sugerida

Crear una salida intermedia fuera del core de TrendRadar, por ejemplo:

- output/social1_candidates/YYYY-MM-DD.jsonl
- output/social1_candidates/candidates.sqlite

Campos minimos:

- candidate_id
- source_system: trendradar
- source_type: hotlist | rss | article_reader
- platform_id o feed_id
- platform_name o feed_name
- title
- url
- mobile_url
- rank
- ranks
- first_seen
- last_seen
- crawl_count
- date
- matched_keyword o tag
- trend_reason
- risk_flags
- social1_status: new | rejected | imported | needs_review
- evidence_refs: rutas SQLite/origen

## Mapeo tecnico desde TrendRadar

Hotlists:

- output/news/YYYY-MM-DD.db
- tabla news_items
- tabla rank_history
- tabla platforms

RSS:

- output/rss/YYYY-MM-DD.db
- tabla rss_items
- tabla rss_feeds

Consulta MCP opcional:

- get_latest_news
- search_news_unified
- get_trending_topics
- get_latest_rss
- search_rss
- trigger_crawl

## Riesgos y mitigaciones

1. Dependencia NewsNow

Riesgo:
- Si NewsNow cae o cambia, no hay hotlists.

Mitigacion:
- Tratar como fuente de senales no critica.
- Registrar errores y no bloquear social1.
- Combinar con RSS propios.

2. Falsos positivos

Riesgo:
- Un titulo caliente no implica contenido publicable.

Mitigacion:
- social1 mantiene gates de facts, imagen, dedupe y elegibilidad.

3. Coste AI

Riesgo:
- AI filter/analysis/translation consume tokens.

Mitigacion:
- piloto sin AI.
- Si se activa, limitar max_news_for_analysis y desactivar rank timeline si sube coste.

4. Ruido operativo

Riesgo:
- Cron cada 30 min genera muchos candidatos repetidos.

Mitigacion:
- dedupe por normalized_url + title_hash + platform/feed.
- estado por candidato.
- incremental para solo novedades.

## Primer experimento recomendado

Duracion:

- 7 dias.

Frecuencia:

- cada 60 minutos.

Fuentes:

- Mantener hotlists default.
- Anadir RSS relevantes a cuentas futuras de social1 si aplica.

Salida:

- candidates.jsonl diario.
- reporte semanal de:
  - total candidatos
  - candidatos repetidos
  - candidatos utiles
  - fuentes mas utiles
  - top temas
  - cuantos pasaron a report social1

Criterio de exito:

- Si al menos 10-20% de candidatos son investigables o accionables, vale integrar.
- Si menos de eso, limitarlo a RSS/temas especificos o descartarlo.

## Cierre

TrendRadar tiene valor como sensor barato y ligero. La integracion correcta no es que publique, sino que alimente una cola de investigacion de social1. El control editorial y factual debe quedarse en social1.

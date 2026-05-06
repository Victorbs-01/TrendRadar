# Plan: TrendRadar como motor de senales especializado

Fecha: 2026-05-05 22:46 CST
Repo: /home/victor/dev/scrapers/TrendRadar

## Objetivo

Convertir TrendRadar en una capa ligera de deteccion de senales, no en extractor profundo ni publicador.

La idea operativa:

TrendRadar escucha mucho y barato -> crea candidatos compactos -> solo los candidatos promovidos pasan al scraper/validador propio -> el scraper profundo toma documento completo, facts, evidencia, dedupe y artefacto final.

## Decision principal

Usar varios perfiles especializados de captura, pero una sola bandeja comun normalizada.

No hacer cinco sistemas inconexos.

Arquitectura:

1. Capturadores ligeros por perfil.
2. Normalizador comun.
3. candidates.sqlite / JSONL.
4. Promocion selectiva.
5. Scraper profundo propio.
6. Destinos: social1, SEO, research, ventas, reputacion.

## Cinco perfiles iniciales

### 1. SEO Freshness

Objetivo:

- validar keywords vivas;
- detectar refresh de paginas existentes;
- encontrar variantes reales de busqueda/lenguaje;
- generar FAQ y clusters.

Acciones:

- create_page;
- refresh_page;
- add_faq;
- monitor;
- reject.

### 2. Social / Editorial Pulse

Objetivo:

- alimentar social1 con oportunidades;
- detectar hooks, novedades, temas calientes y URLs candidatas.

Acciones:

- send_to_social1_report;
- monitor;
- reject.

### 3. Research / Science Radar

Objetivo:

- detectar papers, tendencias cientificas, IA, coding, robotics y tecnologia;
- alimentar articulos tecnicos, thought leadership y contenido social tecnico.

Acciones:

- research_note;
- technical_article;
- seo_brief;
- social_thread;
- monitor.

### 4. Competitor / Source Gap

Objetivo:

- vigilar competidores, blogs, changelogs, newsletters y medios sectoriales;
- detectar huecos frente al contenido propio.

Acciones:

- gap_no_page;
- outdated_page;
- weak_angle;
- respond_with_post;
- no_action.

### 5. B2B / Regulatory / Commercial Signals

Objetivo:

- detectar senales comerciales, regulatorias, ferias, aranceles, supply chain y oportunidades de oferta.

Acciones:

- create_offer;
- create_landing;
- sales_brief;
- monitor.

## Bandeja comun de candidatos

Formato recomendado: SQLite local y/o JSONL diario.

Ruta sugerida:

- output/signal_candidates/candidates.sqlite
- output/signal_candidates/YYYY-MM-DD.jsonl

Campos minimos:

- id
- profile
- source_type
- source_id
- source_name
- title
- url
- normalized_url
- language
- keyword
- entities
- first_seen
- last_seen
- appearances
- best_rank
- avg_rank
- score
- action_suggested
- status
- promoted_to
- evidence_ref
- created_at
- updated_at

Estados:

- new
- monitor
- rejected
- promoted_to_social1
- promoted_to_seo
- promoted_to_research
- promoted_to_sales
- archived

## Flujo de promocion

1. TrendRadar detecta senal compacta.
2. Normalizador crea candidato.
3. Scoring decide prioridad.
4. Humano o agente promueve.
5. Scraper propio hace extraccion profunda.
6. Se guarda evidencia durable.
7. Se genera output final:
   - social1 report/draft;
   - SEO brief/refresco;
   - research note;
   - sales brief;
   - alerta reputacional.

## Fuentes: alternativa correcta para ciencia/research

No usar Sci-Hub como fuente scrapeada.
No hacer scraping directo de Google Scholar.

Alternativas correctas y mas estables:

1. arXiv
   - API/RSS libre.
   - Sin API key.
   - Ideal para IA, ML, robotics, CS, fisica, matematicas.

2. Semantic Scholar
   - API JSON.
   - Paper search, autores, citas, recomendaciones.
   - Sin key para uso basico; con key sube limite.

3. OpenAlex
   - API abierta para papers, autores, instituciones, conceptos, venues.
   - Buena para discovery mas amplio que arXiv.

4. Crossref
   - DOI, metadata editorial, journals, publishers.
   - Bueno para verificar metadatos.

5. PubMed / Europe PMC
   - Biomedicina, salud, life sciences.
   - APIs estables.

6. bioRxiv / medRxiv
   - Preprints de biologia y medicina.
   - RSS/API segun necesidad.

7. Unpaywall
   - Detecta PDFs open-access legales a partir de DOI.
   - Muy util cuando hay paper cerrado pero existe copia OA permitida.

8. DOAJ
   - Journals open-access.

9. CORE
   - Agregador de repositorios open-access.

10. Papers with Code
   - ML papers + tareas + datasets + repos.

Si se necesita algo parecido a Google Scholar:

- usar APIs comerciales tipo SerpAPI u otra opcion con terminos claros;
- o configurar Google Scholar Alerts manuales si produce feed/notificacion aprovechable;
- o usar Google News RSS solo para noticias cientificas, no para papers academicos.

## Fuentes agregables en TrendRadar

### Facil: RSS/Atom/JSON Feed

Agregar en config/config.yaml:

rss:
  feeds:
    - id: "mi-fuente"
      name: "Mi Fuente"
      url: "https://example.com/feed.xml"
      enabled: true

### Medio: fuentes soportadas por NewsNow

Agregar en platforms.sources solo si NewsNow reconoce el id.

### Durable: adaptadores externos

Para APIs o fuentes especiales:

- arxiv_adapter
- semantic_scholar_adapter
- openalex_adapter
- crossref_adapter
- competitor_adapter
- regulatory_adapter

Todos deben escribir al formato comun de senales.

## Retencion y disco

Guardar permanente o 365 dias:

- senales compactas;
- candidatos;
- decisiones;
- hashes;
- URLs;
- evidencia minima.

No guardar por defecto:

- articulos completos;
- PDFs;
- screenshots;
- imagenes;
- HTML completo;
- prompts/respuestas AI completas.

Guardar documentos completos solo para candidatos promovidos.

Retencion sugerida:

- raw hotlist SQLite: 90-180 dias;
- candidates compactos: 365 dias o permanente;
- HTML reports: 30 dias;
- AI raw logs: 14-30 dias;
- documentos completos: 30-90 dias salvo seleccionados.

## P0 propuesto

1. Mantener TrendRadar como capturador de senales.
2. Crear schema de candidates.sqlite.
3. Implementar normalizador desde output/news y output/rss.
4. Definir 5 perfiles con keywords/RSS separados.
5. Activar piloto sin AI y con frecuencia conservadora.
6. Promover manualmente pocos candidatos al scraper profundo.
7. Medir utilidad por perfil.

## Criterios de exito

- Menos de 1 GB/anio en senales compactas.
- Cada perfil produce candidatos accionables.
- Al menos 10-20% de candidatos revisados merecen investigacion.
- Social1 recibe candidatos sin saltarse sus gates.
- SEO recibe refresh/create ideas con evidencia.
- Research usa fuentes legales/open-access.

## Pendientes

- Definir ruta exacta del scraper profundo propio.
- Definir si candidates.sqlite vive dentro de TrendRadar o fuera en un repo de control.
- Definir primer set de RSS por perfil.
- Definir si se usa MCP o lectura directa de SQLite para el normalizador.
- Definir policy de retencion automatica.

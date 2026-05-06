# TrendRadar: perfiles especializados, fuentes nuevas y arquitectura de senales

Fecha: 2026-05-05
Repo: /home/victor/dev/scrapers/TrendRadar

## Decision recomendada

No mezclar todo logicamente en un solo flujo editorial.

Usar varios perfiles especializados de captura, pero unificar la salida en una base comun de senales/candidatos.

Patron recomendado:

- perfil SEO -> captura senales SEO.
- perfil social -> captura oportunidades de contenido rapido.
- perfil research -> captura papers y ciencia abierta.
- perfil competitor -> captura RSS/fuentes competidoras.
- perfil B2B/regulatory -> captura senales comerciales y regulatorias.

Todos escriben a una bandeja comun normalizada:

- signal_candidates.sqlite o JSONL por dia.

Cada registro debe tener:

- profile
- source_type
- source_id
- title
- url
- first_seen
- last_seen
- score
- action_suggested
- status
- evidence_ref

## Regla de oro

TrendRadar solo captura senales.

Si el candidato se promueve, entonces pasa al scraper/validador propio, que toma documento completo, limpia contenido, valida hechos, guarda evidencia y prepara el artefacto final.

Esto evita gastar disco y tokens en ruido.

## Mismo lugar vs 5 variantes

### Opcion A: una sola instancia

Ventajas:

- mas simple.
- menos servicios.
- una sola config.
- suficiente para piloto pequeno.

Desventajas:

- mezcla ruido de SEO, social, research y ventas.
- scoring menos claro.
- dificil aplicar retenciones diferentes.
- dificil saber que perfil genero valor.

### Opcion B: 5 perfiles especializados

Ventajas:

- cada perfil tiene keywords, RSS, frecuencia y scoring propio.
- se puede apagar un perfil sin tocar los demas.
- permite medir ROI por frente.
- retencion diferenciada.
- mejor para escalar.

Desventajas:

- mas configs.
- requiere normalizador central.

Recomendacion:

- piloto: una instancia si queremos velocidad.
- arquitectura durable: 5 perfiles + salida comun.

## Cinco perfiles recomendados

### 1. SEO Freshness

Objetivo:

- validar palabras clave.
- detectar refresh de paginas.
- encontrar variaciones reales.

Fuentes:

- hotlists generales.
- RSS de medios sectoriales.
- blogs/competidores.
- Google News RSS por queries si se acepta fuente no oficial.

Frecuencia:

- 2-4 veces al dia.

Retencion:

- candidates compactos: 365 dias.
- raw hotlist: 90-180 dias.

Acciones:

- create_page
- refresh_page
- add_faq
- monitor
- reject

### 2. Social / Editorial Pulse

Objetivo:

- alimentar social1 con temas oportunos.
- detectar hooks y novedades.

Fuentes:

- hotlists.
- RSS de medios.
- newsletters con feed.

Frecuencia:

- cada 30-60 minutos.

Retencion:

- senales compactas: 90-180 dias.
- candidatos promovidos: permanente en social1.

Acciones:

- send_to_social1_report
- monitor
- reject

### 3. Research / Science Radar

Objetivo:

- ciencia, IA, coding, robotics, papers.
- alimentar articulos tecnicos y thought leadership.

Fuentes legales recomendadas:

- arXiv API/RSS.
- Semantic Scholar API.
- OpenAlex API.
- Crossref API.
- PubMed / Europe PMC.
- bioRxiv / medRxiv RSS/API.
- Papers with Code.
- DOAJ.
- CORE.
- Unpaywall para PDFs open-access.

Evitar:

- Sci-Hub como fuente scrapeada.
- scraping directo de Google Scholar.

Frecuencia:

- 1-4 veces al dia.

Retencion:

- metadatos: permanente o 365 dias.
- PDFs/documentos completos: solo promovidos.

### 4. Competitor / Source Gap

Objetivo:

- detectar que publican competidores y fuentes clave.
- comparar contra paginas propias.

Fuentes:

- RSS de competidores.
- blogs de empresas.
- changelogs.
- newsletters con RSS.
- sitios de reguladores.

Frecuencia:

- 1-2 veces al dia.

Acciones:

- gap_no_page
- outdated_page
- weak_angle
- no_action

### 5. B2B / Regulatory / Commercial Signals

Objetivo:

- detectar senales que pueden convertirse en oferta, landing o accion comercial.

Fuentes:

- reguladores.
- camaras de comercio.
- aduanas/aranceles.
- ferias.
- medios comerciales.
- fuentes de supply chain.

Frecuencia:

- 1-2 veces al dia.

Acciones:

- create_offer
- create_landing
- sales_brief
- monitor

## Se pueden agregar fuentes?

Si.

Hay tres niveles.

### 1. RSS/Atom/JSON Feed

Es el camino mas facil.

Editar config/config.yaml:

rss:
  feeds:
    - id: "mi-fuente"
      name: "Mi Fuente"
      url: "https://example.com/feed.xml"
      enabled: true

Ventaja:

- no requiere scraper nuevo.
- es estable.
- legal/limpio si el feed es publico.

### 2. Hotlist platforms via NewsNow

En config/config.yaml se pueden agregar plataformas en platforms.sources, pero solo funcionara si NewsNow reconoce ese id.

Ejemplo conceptual:

platforms:
  sources:
    - id: "zhihu"
      name: "知乎"

Limitacion:

- No convierte cualquier web en fuente.
- Depende de IDs soportados por NewsNow.

### 3. Adaptadores externos

Para fuentes sin RSS o APIs propias:

- crear mini-adapter externo.
- normalizar salida al formato signal_candidates.
- no tocar core TrendRadar al inicio.

Ejemplo:

source_adapter -> normalized_signal -> candidates.sqlite

## Sci-Hub y Google Scholar / Google Science

### Sci-Hub

No recomendado.

Motivos:

- riesgo legal/copyright.
- posible violacion de terminos.
- bloqueo y fragilidad tecnica.
- no conviene para un sistema empresarial.

Alternativa correcta:

- arXiv.
- Semantic Scholar.
- OpenAlex.
- Crossref.
- PubMed/Europe PMC.
- Unpaywall para localizar PDFs open-access legales.
- DOAJ/CORE.

### Google Scholar

No recomendado hacer scraping directo.

Motivos:

- no hay API oficial publica estable para scraping general.
- bloqueo/captcha frecuente.
- riesgo de ToS.

Alternativas:

- Semantic Scholar.
- OpenAlex.
- Crossref.
- SerpAPI/otras APIs comerciales si de verdad se necesita Google Scholar.
- Google Scholar Alerts si se configura manualmente y ofrece feed/notificacion util.
- Google News RSS para queries de ciencia, si basta con noticias y no papers.

## Como se ve el flujo promovido

1. TrendRadar detecta senal:
   - titulo
   - URL
   - fuente
   - score
   - perfil

2. Normalizador crea candidato:
   - status = new

3. Clasificador decide:
   - reject
   - monitor
   - promote_to_social1
   - promote_to_seo
   - promote_to_research

4. Si se promueve:
   - nuestro scraper toma el documento completo.
   - guarda HTML/Markdown/PDF si corresponde.
   - extrae facts.
   - valida fuente/licencia.
   - dedupe.
   - genera evidencia durable.

5. Resultado:
   - social draft.
   - SEO brief.
   - research note.
   - sales brief.

## Cierre

Si se quieren 5 variantes, que sean perfiles especializados, no cinco sistemas inconexos.

La arquitectura ideal es:

multiples capturadores ligeros -> una bandeja comun -> promocion selectiva -> scraper profundo propio.

Asi el disco y los tokens se gastan solo en lo que ya demostro valor.

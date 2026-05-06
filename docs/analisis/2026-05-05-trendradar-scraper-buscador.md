# TrendRadar: scraper, buscador y flujo real de obtencion de datos

Fecha de analisis: 2026-05-05 21:08 CST
Repo analizado: /home/victor/dev/scrapers/TrendRadar

## Conclusion corta

TrendRadar no usa un scraper conocido tipo Scrapy, Playwright, Selenium, Puppeteer o navegador headless.

La obtencion de datos se divide en cuatro rutas:

1. Hotlists / rankings: llama una API externa de NewsNow con requests.
2. RSS/Atom/JSON Feed: descarga feeds con requests y los parsea con feedparser o parser JSON propio.
3. Busqueda MCP: busca sobre SQLite local ya generado, no sobre la web en vivo.
4. Lectura de articulos completos: herramienta MCP opcional via Jina Reader, no parte central del crawler automatico.

Para social1, esto significa que TrendRadar es mejor como radar de senales y bandeja de candidatos, no como fuente factual suficiente para publicar.

## 1. Hotlists: no scrapea cada plataforma directamente

Archivo principal:

- trendradar/crawler/fetcher.py

Clase:

- DataFetcher

Punto clave:

- DEFAULT_API_URL = https://newsnow.busiyi.world/api/s

Para cada plataforma arma una URL como:

- https://newsnow.busiyi.world/api/s?id=zhihu&latest
- https://newsnow.busiyi.world/api/s?id=weibo&latest
- https://newsnow.busiyi.world/api/s?id=douyin&latest

Ejecucion:

- Usa requests.get(...).
- Envia headers de navegador:
  - User-Agent tipo Chrome
  - Accept: application/json, text/plain, */*
  - Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
  - Cache-Control: no-cache
- Valida que el JSON devuelva status success o cache.
- Convierte items[] en estructura interna por plataforma.
- Registra title, url, mobileUrl y rank.
- Duerme entre plataformas usando request_interval con jitter.

Importante:

TrendRadar no entra a Weibo, Zhihu, Baidu, Douyin, etc. con navegador propio. Depende de NewsNow como agregador upstream.

Riesgo:

- Si NewsNow cambia respuesta, bloquea headers o cae, las hotlists fallan.
- Es ligero, pero hay dependencia externa sin SLA.
- La app pide controlar frecuencia para no abusar del servicio comunitario.

## 2. Plataformas hotlist por defecto

Configuracion:

- config/config.yaml

Bloque:

- platforms.sources

Lista actual verificada:

- toutiao: 今日头条
- baidu: 百度热搜
- wallstreetcn-hot: 华尔街见闻
- thepaper: 澎湃新闻
- bilibili-hot-search: bilibili 热搜
- cls-hot: 财联社热门
- ifeng: 凤凰网
- tieba: 贴吧
- weibo: 微博
- douyin: 抖音
- zhihu: 知乎

## 3. RSS: fetching directo de feeds

Archivos:

- trendradar/crawler/rss/fetcher.py
- trendradar/crawler/rss/parser.py

Dependencias:

- requests
- feedparser

Funcionamiento:

1. RSSFetcher crea una requests.Session.
2. Configura headers de lector RSS:
   - User-Agent: TrendRadar/2.0 RSS Reader
   - Accept: application/feed+json, application/json, application/rss+xml, application/atom+xml, application/xml, text/xml, */*
3. Hace session.get(feed.url, timeout=...).
4. RSSParser.parse(...) decide:
   - Si parece JSON Feed, parsea JSON Feed 1.1 con json.loads.
   - Si no, usa feedparser.parse para RSS/Atom.
5. Normaliza cada entry a:
   - title
   - url
   - published_at
   - summary
   - author
   - guid
6. Convierte a RSSItem y guarda en SQLite.

RSS no necesita API key. Depende de que cada feed sea accesible.

Feeds configurados actualmente:

- hacker-news activo: https://hnrss.org/frontpage
- ruanyifeng desactivado: http://www.ruanyifeng.com/blog/atom.xml
- yahoo-finance activo: https://finance.yahoo.com/news/rssindex

## 4. Lectura de articulos completos: Jina Reader opcional

Archivo:

- mcp_server/tools/article_reader.py

Base externa:

- https://r.jina.ai

Funcionamiento:

- read_article(url) llama:
  - https://r.jina.ai/{url}
- Pide Markdown:
  - Accept: text/markdown
  - X-Return-Format: markdown
  - X-No-Cache: true
- Tiene throttle interno de 5 segundos.
- read_articles_batch procesa maximo 5 URLs por lote.

Esto no es el crawler principal. Es una herramienta MCP para que un agente lea el cuerpo de una URL candidata.

Registro/pago:

- El codigo permite funcionar sin jina_api_key.
- Con key se eleva limite.
- El propio error menciona limite gratuito: 100 RPM / 2 concurrencias.

Riesgo para social1:

- Jina puede fallar segun sitio, paywall, bloqueo o rate limit.
- No debe sustituir validacion factual propia.

## 5. Como ejecuta la busqueda

Hay que distinguir dos cosas:

A. Obtencion/crawl

- Hotlists: consulta NewsNow API en vivo.
- RSS: descarga feeds en vivo.
- Se ejecuta por cron Docker, local CLI o MCP trigger_crawl.

B. Busqueda/consulta

La busqueda no va a Google/Bing ni scrapea web en vivo. Busca sobre los datos guardados en SQLite.

Archivos principales:

- mcp_server/tools/data_query.py
- mcp_server/tools/search_tools.py
- mcp_server/services/data_service.py
- mcp_server/services/parser_service.py

Rutas SQLite:

- output/news/YYYY-MM-DD.db
- output/rss/YYYY-MM-DD.db

ParserService abre SQLite con sqlite3 y lee:

- news_items
- platforms
- rank_history
- crawl_records
- rss_items
- rss_feeds

## 6. Modos de busqueda MCP

Archivo:

- mcp_server/tools/search_tools.py

Funcion:

- search_news_unified(...)

Modos:

1. keyword
   - Coincidencia exacta por substring case-insensitive.
   - Ejemplo logico: query_lower in title.lower().

2. fuzzy
   - Usa difflib.SequenceMatcher para similitud textual.
   - Si query esta contenido en title, score 1.0.
   - Si no, calcula ratio completo.
   - Tambien extrae palabras con regex y acepta si hay >=50% de solape de keywords.

3. entity
   - Contiene la entidad exacta en el titulo.
   - Es mas estricto que keyword porque usa query in title sin lower en ese punto.

Ordenacion:

- relevance: por similarity_score.
- weight: por peso calculado en analytics.
- date: por fecha.

Opcionalmente puede incluir RSS:

- include_rss=True

## 7. Trending topics

Archivo:

- mcp_server/tools/data_query.py
- mcp_server/services/data_service.py

Funcion:

- get_trending_topics(...)

Modos:

- current: ultima tanda de datos.
- daily: acumulado del dia.

Extract modes:

- keywords: usa config/frequency_words.txt.
- auto_extract: extrae terminos desde titulos con regex y stopwords.

Esto tampoco es busqueda web; es analisis de titulos ya almacenados.

## 8. Persistencia local

Archivos schema:

- trendradar/storage/schema.sql
- trendradar/storage/rss_schema.sql
- trendradar/storage/ai_filter_schema.sql

Tablas hotlist:

- platforms
- news_items
- title_changes
- rank_history
- crawl_records
- crawl_source_status
- period_executions

Tablas RSS:

- rss_feeds
- rss_items
- rss_crawl_records
- rss_crawl_status
- rss_push_records

Tablas AI filter:

- ai_filter_tags
- ai_filter_results
- ai_filter_analyzed_news

Estado local verificado antes de este documento:

- output/news contiene DBs 2025-12-21 a 2025-12-27.
- Cada DB pesa aprox. 900 KiB.
- Cada DB tiene aprox. 1.1k news_items, 11 platforms, 22 crawl_records y 5.6k rank_history.

## 9. Dependencias relevantes

Desde pyproject.toml:

- requests: HTTP simple.
- feedparser: parse RSS/Atom.
- fastmcp: MCP server.
- boto3: storage S3-compatible.
- litellm: AI providers.
- json-repair: reparar JSON de respuestas AI.
- tenacity: retries.

No aparecen como dependencias principales:

- Scrapy
- Playwright
- Selenium
- Puppeteer
- BeautifulSoup / bs4
- newspaper3k
- trafilatura

## 10. Respuesta directa a la duda

Si la pregunta es: "usa algun scrapeador conocido?"

Respuesta:

No para hotlists. Usa requests contra NewsNow API. Para RSS usa feedparser, que es un parser conocido de RSS/Atom, no un scraper web general. Para articulos completos usa Jina Reader opcional, que si es un servicio externo conocido para convertir paginas a Markdown.

Si la pregunta es: "como ejecuta la busqueda?"

Respuesta:

Primero captura y guarda datos en SQLite. Luego sus herramientas de busqueda recorren esas SQLite por fecha/plataforma y aplican substring, fuzzy matching con difflib o extraccion de terminos. No hace busqueda web general en cada consulta.

## 11. Implicacion para social1

TrendRadar sirve bien para detectar senales:

- tema caliente
- URL candidata
- ranking/plataforma
- primera/ultima aparicion
- recurrencia/rank_history
- fuente RSS candidata

No basta para publicar automaticamente porque normalmente no trae:

- verificacion factual completa
- imagen primaria validada
- contexto editorial suficiente
- dedupe contra backlog social1
- elegibilidad temporal/evento
- evidencia durable social1

Uso recomendado:

TrendRadar -> candidates -> social1 report -> prepare-post -> PublishJob draft -> aprobacion humana -> Postiz draft.

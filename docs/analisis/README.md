# Analisis de TrendRadar

Fecha de analisis: 2026-05-05 21:08 CST
Repo analizado: /home/victor/dev/scrapers/TrendRadar

Documentos:

1. 2026-05-05-trendradar-scraper-buscador.md
   - Que scraper/buscador usa TrendRadar.
   - Como obtiene hotlists, RSS y articulos.
   - Como ejecuta busquedas sobre datos ya guardados.
   - Dependencias externas y riesgos.

2. 2026-05-05-trendradar-social1-uso-recomendado.md
   - Encaje recomendado con social1.
   - Modo de operacion piloto.
   - Que guardar como bandeja de candidatos.
   - Configuracion segura inicial.

3. 2026-05-05-trendradar-frentes-creativos-seo.md
   - Usos creativos como motor de senales.
   - Validador de keywords SEO.
   - Radar de refresh, contenido, competencia, reputacion y ventas.
   - Priorizacion P0/P1/P2.

4. 2026-05-05-trendradar-coste-disco-senales.md
   - Medicion local del peso real de SQLite.
   - Proyecciones 30/90/365 dias.
   - Diferencia entre senales compactas y articulos completos.
   - Politica de retencion recomendada.

5. 2026-05-05-trendradar-perfiles-fuentes-arquitectura.md
   - Arquitectura recomendada con 5 perfiles especializados.
   - Mismo lugar vs multiples perfiles.
   - Como agregar fuentes RSS/API/adaptadores.
   - Nota sobre Sci-Hub, Google Scholar y fuentes academicas legales.

6. 2026-05-05-trendradar-signal-engine-plan.md
   - Plan consolidado para usar TrendRadar como motor de senales.
   - Incluye flujo de promocion a scraper profundo propio.
   - Incluye alternativa correcta para ciencia/research.
   - Incluye P0, criterios de exito y pendientes.

Resumen corto:

TrendRadar no usa Scrapy, Playwright, Selenium ni un navegador headless. Para hotlists llama una API externa del proyecto NewsNow con requests. Para RSS usa requests + feedparser. Para leer articulos completos desde URL, solo en MCP, usa Jina Reader r.jina.ai. Sus busquedas no son busquedas web en vivo: normalmente consultan SQLite local ya poblado en output/news y output/rss.

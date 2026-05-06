# TrendRadar: frentes creativos de uso y SEO keyword validation

Fecha: 2026-05-05
Repo: /home/victor/dev/scrapers/TrendRadar

## Idea central

TrendRadar no debe verse solo como scraper de noticias. Debe verse como motor de senales temporales:

- detecta que terminos aparecen ahora;
- mide en que fuentes/plataformas aparecen;
- conserva primera aparicion, ultima aparicion, frecuencia y ranking;
- permite buscar historico en SQLite;
- puede empujar resultados por webhook o exponerlos via MCP.

Eso lo vuelve util para SEO, contenido, social, inteligencia comercial y vigilancia de mercado.

## 1. Validador de palabras clave para SEO

Si sirve, pero con una precision importante:

TrendRadar no reemplaza Google Search Console, Ahrefs, Semrush, Keyword Planner o Baidu Index.

Lo que si puede validar es:

- frescura de una keyword;
- si una keyword esta viva en conversaciones/news;
- variantes reales de titulares;
- cambios de lenguaje;
- entidades asociadas;
- oportunidad de contenido oportunista;
- si conviene actualizar una pagina existente por tendencia nueva.

### Workflow propuesto

Input:

- lista seed de keywords SEO;
- paginas existentes;
- verticales de negocio;
- feeds RSS de competidores/fuentes sectoriales.

TrendRadar:

- captura titulos y URLs;
- busca keywords exactas/fuzzy;
- detecta variantes y co-ocurrencias;
- calcula tendencia por rank_history/crawl_count;
- genera candidatos SEO.

Output:

- keywords a crear;
- keywords a refrescar;
- keywords a observar;
- keywords a descartar;
- URLs/fuentes de evidencia.

### Score sugerido: SEO Freshness Score

Campos:

- keyword
- normalized_keyword
- language
- source_count
- platform_count
- rss_count
- first_seen
- last_seen
- appearances
- best_rank
- avg_rank
- velocity_24h
- persistence_days
- title_variants
- related_entities
- suggested_intent: informational | commercial | navigational | news | comparison
- action: create | refresh | monitor | reject
- confidence

Formula simple:

SEO_Freshness_Score =
  0.30 * freshness
+ 0.25 * cross_source_presence
+ 0.20 * rank_strength
+ 0.15 * persistence
+ 0.10 * business_fit

### Uso real

- Si una keyword aparece en muchas fuentes pero dura poco: hacer post social/news, no pagina evergreen.
- Si aparece de forma recurrente durante dias/semanas: candidata a guia SEO o landing.
- Si aparece en RSS sectorial pero no en hotlists: candidata B2B/nicho.
- Si aparece en hotlists masivas pero no encaja con negocio: descartar.

## 2. Radar de refresh SEO

Cada pagina existente tiene keywords objetivo.

TrendRadar vigila si aparecen nuevas variantes o subtemas.

Ejemplo:

Pagina: "sourcing en China"
Nuevas senales:

- "factory audit"
- "China supplier verification"
- "tariffs 2026"
- "cross-border payment"

Accion:

- sugerir update del H2;
- anadir FAQ;
- actualizar intro;
- generar snippet para blog/social;
- crear nota interna con fuentes.

## 3. Generador de briefs editoriales

TrendRadar puede producir briefs diarios para redactores:

- que paso;
- por que importa;
- keywords emergentes;
- fuentes;
- titulares repetidos;
- angulos posibles;
- nivel de urgencia;
- formato recomendado: post corto, hilo, articulo, landing, newsletter.

Esto encaja con social1 como bandeja de investigacion.

## 4. Detector de lenguaje real de mercado

Muy util para China y B2B.

No pregunta "como deberiamos llamar esto?" sino "como lo esta llamando el mercado hoy?"

Uso:

- comparar terminos ingles/espanol/chino;
- detectar nombres oficiales vs nombres populares;
- descubrir sinonimos;
- evitar traducciones que nadie usa.

Ejemplo:

- "supply chain resilience"
- "nearshoring"
- "China sourcing"
- "cross-border e-commerce"
- variantes chinas reales en titulos.

## 5. Radar de competidores y fuentes

Con RSS configurados de competidores, blogs, medios sectoriales y newsletters:

- detectar que publican;
- identificar temas repetidos;
- ver vacios de contenido;
- disparar respuestas rapidas;
- crear matriz: competitor_topic_gap.

Output sugerido:

- competitor
- topic
- title
- url
- first_seen
- matched_keyword
- our_existing_page
- gap_type: no_page | outdated_page | weak_angle | duplicate
- suggested_action

## 6. Monitor de reputacion / crisis

Para marcas, productos o clientes:

- vigilar menciones exactas;
- vigilar nombres de directivos;
- vigilar combinaciones negativas: estafa, queja, problema, recall, multa, hack, etc.;
- alertar por webhook.

No es una solucion enterprise de social listening, pero sirve como radar barato.

## 7. Radar de oportunidades comerciales B2B

En vez de mirar solo temas virales, mirar senales de demanda:

- regulaciones nuevas;
- cambios arancelarios;
- ferias;
- cadenas de suministro;
- categoria producto;
- problemas de importadores;
- sectores que empiezan a moverse.

Output para ventas:

- lead_theme
- geography
- urgency
- affected_industry
- suggested_offer
- evidence_url
- suggested_landing

## 8. Generador de calendarios de contenido

Cada semana:

- top temas persistentes;
- top temas emergentes;
- temas agotados;
- temas evergreen;
- temas solo social.

Se puede convertir en calendario:

- lunes: informe largo SEO;
- martes: carrusel social;
- miercoles: newsletter;
- jueves: post comparativo;
- viernes: recap.

## 9. Detector de entidades emergentes

Extraer y normalizar:

- empresas;
- productos;
- ferias;
- tecnologias;
- politicas;
- personas;
- paises/ciudades;
- plataformas.

Uso:

- glosario SEO;
- tags internos;
- contenido relacionado;
- cluster pages.

## 10. Validador de titulares

Antes de publicar, comparar un titular propio contra titulares vivos:

- demasiado parecido: riesgo de commodity;
- no usa vocabulario del mercado: riesgo SEO;
- tiene entidad correcta pero angulo debil;
- falta keyword caliente.

Output:

- titulo_original
- similitud_con_tendencias
- keyword_missing
- alternative_titles
- risk: generic | outdated | strong | opportunistic

## 11. Priorizador de paginas programaticas

Para sitios grandes:

- producto + pais
- feria + industria
- servicio + ciudad
- problema + solucion

TrendRadar puede decir que combinaciones estan vivas ahora, para no crear 1000 paginas sin demanda.

## 12. Radar de FAQs

De titulos y RSS se pueden detectar preguntas o temas que se convierten en FAQs:

- que es X;
- como afecta X;
- por que subio/bajo X;
- diferencia entre X e Y;
- requisitos de X.

Salida:

- FAQ candidata;
- pagina destino;
- fuente;
- prioridad;
- fecha.

## 13. Radar de contenido multilingue

Para materiales en espanol, ingles y chino:

- validar que terminos chinos no sean traduccion literal mala;
- detectar equivalentes ingleses reales;
- proponer pinyin si tiene sentido;
- mantener glosario vivo.

## 14. Producto interno: Trend Desk

Construir encima de TrendRadar una pequena capa propia:

- SQLite -> candidates;
- dashboard simple;
- acciones: reject, monitor, send_to_social1, send_to_seo, send_to_sales;
- score por frente;
- export CSV/JSON.

No hace falta modificar TrendRadar core al principio.

## 15. Producto comercial: Radar report

Con los datos se puede crear un producto de reporte:

- "China Market Pulse semanal";
- "AI/Coding Trend Pulse";
- "Robotics China Radar";
- "Trade Fair Opportunity Radar";
- "Supplier Risk Signals".

Formato:

- top 10 temas;
- palabras nuevas;
- fuentes;
- oportunidades SEO;
- oportunidades social;
- riesgos;
- recomendaciones.

## Frentes recomendados por prioridad

P0 - Alto valor / facil

1. SEO Freshness Validator.
2. Bandeja de candidatos social1.
3. Radar de refresh de paginas existentes.
4. Brief editorial semanal.
5. RSS competitor/content gap.

P1 - Medio valor / requiere adaptador

6. Reputacion/crisis.
7. Radar comercial B2B.
8. Titular validator.
9. Glosario multilingue vivo.
10. FAQ miner.

P2 - Producto futuro

11. Trend Desk interno.
12. Radar report vendible.
13. Priorizador de paginas programaticas.
14. API interna de senales.
15. Integracion con CRM/taskflow.

## Cierre

El mejor concepto no es "scraper" sino "signal engine".

TrendRadar detecta senales temporales baratas. El valor aparece cuando esas senales se enrutan a frentes concretos:

- SEO: crear/refrescar/descartar keywords.
- Social: candidatos oportunos.
- Ventas: temas con intencion comercial.
- Producto: lenguaje real del mercado.
- Reputacion: alertas tempranas.
- Inteligencia: reportes y briefing.

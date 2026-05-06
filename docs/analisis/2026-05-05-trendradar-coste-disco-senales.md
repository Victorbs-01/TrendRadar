# TrendRadar: coste de disco para senales

Fecha: 2026-05-05
Repo: /home/victor/dev/scrapers/TrendRadar

## Conclusion

Guardar senales de TrendRadar es barato si se guardan solo textos cortos, URLs, rankings y metadatos en SQLite.

Lo que se vuelve pesado no son las senales, sino:

- articulos completos en Markdown;
- HTML reports acumulados sin retencion;
- screenshots o imagenes;
- prompts/respuestas AI completas;
- duplicados sin compactacion.

## Medicion local verificada

Directorio medido:

- /home/victor/dev/scrapers/TrendRadar/output/news

Datos existentes:

- 7 DBs diarias.
- Promedio por DB: 935,643 bytes = 0.892 MiB/dia.
- Promedio news_items: 1,146/dia.
- Promedio crawl_records: 22/dia.
- Promedio rank_history: 5,607/dia.

Proyeccion con patron actual:

- 30 dias: 26.77 MiB.
- 90 dias: 80.31 MiB.
- 365 dias: 325.69 MiB.

Proyeccion si se corre cada 30 min reales todo el dia, aproximando 48 crawls/dia:

- multiplicador frente a 22 crawls/dia: 2.18x.
- 30 dias: 58.41 MiB.
- 90 dias: 175.22 MiB.
- 365 dias: 710.59 MiB.

## Candidates JSONL

Si agregamos una bandeja propia de candidatos con texto corto:

- 200 candidatos/dia x 1 KB: 71.29 MiB/anio.
- 200 candidatos/dia x 2 KB: 142.58 MiB/anio.
- 1000 candidatos/dia x 1 KB: 356.45 MiB/anio.
- 1000 candidatos/dia x 2 KB: 712.89 MiB/anio.

Sigue siendo manejable.

## Articulos completos

Si se guardan cuerpos completos de articulos, el coste sube rapido:

- 50 articulos/dia x 25 KB: 0.44 GiB/anio.
- 100 articulos/dia x 25 KB: 0.87 GiB/anio.
- 500 articulos/dia x 25 KB: 4.35 GiB/anio.
- 1000 articulos/dia x 25 KB: 8.70 GiB/anio.

Con 50 KB/articulo:

- 100 articulos/dia: 1.74 GiB/anio.
- 500 articulos/dia: 8.70 GiB/anio.
- 1000 articulos/dia: 17.40 GiB/anio.

## Recomendacion de almacenamiento

Para social1/SEO/radar:

1. Guardar senales compactas permanentemente o 365 dias:
   - title
   - url canonical
   - source
   - rank
   - first_seen
   - last_seen
   - crawl_count
   - matched_keyword
   - score
   - decision/status

2. No guardar articulos completos por defecto.

3. Guardar solo extractos:
   - summary corto
   - 3-5 frases clave
   - hash del contenido
   - URL fuente

4. Articulos completos solo para candidatos promovidos:
   - status = imported / needs_review / selected

5. Configurar retencion:
   - hotlist raw: 90 o 180 dias.
   - candidates compactos: 365 dias o permanente.
   - HTML reports: 30 dias.
   - AI raw prompts/responses: 14-30 dias.
   - articulos completos: 30-90 dias, salvo seleccionados.

## Regla simple

Senales compactas: barato.

Documentos completos: moderado.

Imagenes/screenshots: caro.

Para el piloto, con 30-90 dias de retencion, el disco no deberia ser un problema.

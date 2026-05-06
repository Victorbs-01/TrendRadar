# Plan: Research Open Access Resolver sin Sci-Hub

Fecha: 2026-05-05
Repo: /home/victor/dev/scrapers/TrendRadar

## Objetivo

Definir una alternativa robusta y legal a Sci-Hub para el perfil Research / Science Radar.

La idea no es automatizar lectura de fuentes problemáticas, sino resolver primero si existe una version open-access o autorizada del paper. Solo si existe acceso limpio, el candidato pasa al scraper profundo.

## Decision

No usar Sci-Hub como fuente automatizada.
No hacer scraping directo de Google Scholar.

Construir o planificar un Open Access Resolver:

Input:
- DOI
- arXiv ID
- PubMed ID / PMC ID
- titulo
- autores
- URL fuente

Output:
- open_pdf_found
- metadata_only
- manual_access_needed
- rejected

## Flujo recomendado

1. Signal capture

Fuentes de senales:
- arXiv
- Semantic Scholar
- OpenAlex
- Crossref
- PubMed / Europe PMC
- bioRxiv / medRxiv
- Papers with Code
- Google News RSS solo para noticias cientificas, no como indice academico principal

Guardar solo senal compacta:
- title
- abstract si existe
- authors
- DOI
- arXiv ID
- venue
- year
- citationCount
- openAccess flag
- source URL
- candidate score

2. Candidate scoring

Score sugerido:
- relevancia con perfil
- novedad
- citationCount / influentialCitationCount si aplica
- open_access availability
- presencia de codigo/dataset
- fit con SEO/social/research

3. Promotion

Solo candidatos con valor pasan a:
- promoted_to_research
- promoted_to_seo
- promoted_to_social

4. Open Access Resolver

Buscar PDF o texto completo legal en este orden:

1. arXiv PDF
2. PubMed Central / Europe PMC
3. bioRxiv / medRxiv
4. Unpaywall por DOI
5. Semantic Scholar openAccessPdf
6. OpenAlex open access locations
7. publisher open-access URL
8. author accepted manuscript / institutional repository
9. CORE / DOAJ cuando aplique

5. Deep scraper

Si hay PDF/HTML legal:
- descargar documento
- extraer texto con PyMuPDF si es PDF textual
- usar marker-pdf solo si hace falta OCR/layout complejo
- guardar Markdown normalizado
- extraer secciones: abstract, introduction, method, results, limitations, conclusion
- extraer claims y evidencia
- extraer datasets/codigo si aparecen
- guardar hash y metadata

Si no hay PDF legal:
- status = metadata_only o manual_access_needed
- no descargar desde Sci-Hub
- conservar solo metadata/abstract/fuente

## APIs/fuentes correctas

### arXiv

Uso:
- discovery en IA/CS/ML/robotics/fisica/matematicas
- PDF legal inmediato si el paper esta en arXiv

Ventaja:
- sin API key
- RSS/API estable

### Semantic Scholar

Uso:
- paper search
- autores
- citas
- referencias
- recommendations
- openAccessPdf cuando existe

### OpenAlex

Uso:
- discovery academico amplio
- conceptos
- instituciones
- venues
- open access locations

### Crossref

Uso:
- DOI metadata
- publisher/venue verification
- fechas y enlaces oficiales

### PubMed / Europe PMC

Uso:
- biomedicina y salud
- PMCID y full text cuando existe

### bioRxiv / medRxiv

Uso:
- preprints bio/med
- discovery temprano

### Unpaywall

Uso:
- dado DOI, encontrar PDF open-access legal
- pieza central del resolver

### DOAJ / CORE

Uso:
- journals y repositorios open-access
- fallback para texto completo legal

### Papers with Code

Uso:
- ML/AI papers con codigo, datasets, benchmarks y tareas

## Por que no Sci-Hub

No es dificil tecnicamente leer un PDF si se obtiene. El problema es la adquisicion.

Riesgos:
- copyright/legal
- dominios/mirrors cambiantes
- bloqueo/captcha/DNS
- fragilidad operativa
- evidencia poco defendible
- mala base para producto empresarial

Conclusion:
Sci-Hub puede parecer atajo, pero no es base durable.

## Por que no Google Scholar scraping directo

Riesgos:
- no hay API oficial publica para scraping general
- captcha y bloqueos frecuentes
- fragilidad
- posible conflicto con ToS

Alternativas:
- Semantic Scholar
- OpenAlex
- Crossref
- APIs comerciales tipo SerpAPI si se necesita SERP academica
- Scholar Alerts manuales si aportan senales

## Outputs del resolver

Ejemplo de registro:

```json
{
  "candidate_id": "research_2026_0001",
  "title": "...",
  "doi": "10.xxxx/yyyy",
  "arxiv_id": "2601.00001",
  "resolver_status": "open_pdf_found",
  "pdf_url": "https://arxiv.org/pdf/2601.00001",
  "source": "arxiv",
  "license_hint": "open",
  "deep_scrape_status": "pending",
  "evidence_ref": "output/research_candidates/2026-05-05/..."
}
```

Estados:
- open_pdf_found
- html_fulltext_found
- metadata_only
- manual_access_needed
- blocked_or_unavailable
- rejected

## Retencion

Guardar largo plazo:
- metadata
- DOI/arXiv ID
- abstract
- resolver status
- URL legal encontrada
- hash
- decision

Guardar temporal:
- PDF completo solo para promovidos
- Markdown extraido solo para promovidos
- logs de extraccion 14-30 dias

## P0 para manana

1. Decidir si el Research Radar vive como perfil dentro del plan TrendRadar o como adaptador externo.
2. Definir schema de research_candidates.
3. Elegir primeras fuentes:
   - arXiv
   - Semantic Scholar
   - OpenAlex
   - Unpaywall
4. Implementar primero metadata-only.
5. Agregar resolver de PDF legal.
6. Integrar con scraper profundo solo para promoted_to_research.
7. Validar con 10-20 papers de IA/robotics/coding.

## Cierre

La solucion correcta no es scrapear Sci-Hub.

La solucion correcta es:

Research signals -> metadata -> open-access resolver -> deep scraper solo si hay acceso legal/autorizado -> research/SEO/social outputs.

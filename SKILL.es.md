---
name: performance-audit
description: Úsala siempre que necesites auditar el performance de una página (landing, WordPress/Elementor, HTML/Tailwind, Next.js, página de captura) bajo la óptica de Google PageSpeed Insights / Lighthouse, O cuando estés desarrollando una página nueva y quieras garantizar buen puntaje desde el inicio. TRIGGER cuando el agente menciona "PageSpeed", "Lighthouse", "Core Web Vitals", "LCP", "CLS", "INP", "TBT", "performance", "página lenta", "score bajo", "optimizar página"; cuando la sesión toca archivos de hero, imágenes, fuentes, CSS crítico, GTM/Pixel/Analytics, plugins de caché, Elementor; antes de cualquier deploy de página pública. SKIP en cambios puramente internos (API, jobs, schema) sin impacto en el renderizado del front.
argument-hint: [url-o-alcance?] [--mode=audit|build] [--target=mobile|desktop|both]
allowed-tools: Read, Grep, Glob, Bash(curl:*), Bash(git diff:*), Bash(git status:*), Bash(ls:*), Bash(find:*), Bash(wc:*), WebFetch, Task
---

# Performance Audit

Auditoría de performance de páginas bajo la óptica de **Google PageSpeed Insights / Lighthouse**, sin sacrificar layout, tracking, UTMs, SEO ni conversión.

**Esta skill no es una optimización ciega.** Es una auditoría con criterio, que separa lo que el Lighthouse mide de lo que el usuario siente, y prioriza correcciones que entregan impacto real sin romper la página.

## Principio central

```
PERFORMANCE NO ES SCORE. ES ENTREGAR EL ABOVE-THE-FOLD RÁPIDO,
MANTENER ESTABILIDAD VISUAL Y NO ROMPER TRACKING NI CONVERSIÓN.
```

Un score 100 con formulario roto, Pixel que no dispara o UTM perdido **es una regresión**, no una victoria. Toda recomendación de esta skill debe preservar:

1. **Layout visual** — sin CLS introducido, sin romper grid/componentes.
2. **Tracking** — Meta Pixel, GTM, GA4, Hotmart, ActiveCampaign, Make, conversiones.
3. **UTMs y parámetros de URL** — nunca dropear query string en redirects/canonical.
4. **Funcionalidad de formularios** — campos, validación, action, integraciones.
5. **SEO** — meta tags, structured data, hreflang, canonical, robots.
6. **Accesibilidad básica** — alt, labels, contraste, foco.

Si una "optimización" pone cualquiera de estos en riesgo, **no la recomiendes sin alerta explícita**.

## Modos de operación

La skill opera en dos modos. Identifica el modo al inicio:

### Modo `audit` — auditoría de página existente

Inputs aceptados:
- URL pública.
- Print/JSON del PageSpeed Insights.
- Reporte Lighthouse (HTML/JSON).
- Código fuente (HTML, template Elementor, componente Next.js).
- Fragmento específico (hero, head, footer).

Output: diagnóstico priorizado + plan de corrección (ver [references/report-format.md](references/report-format.md)).

### Modo `build` — desarrollando página nueva

Aplicar buenas prácticas **antes** de que el código exista. Revisar HTML/CSS/JS en el commit. Validar hero, imágenes, fuentes, scripts y tracking según las checks en [checks/](checks/).

Output: checklist pre-deploy + observaciones de riesgo en el código actual.

## Workflow

### 1. Identificar alcance y modo

- Si hay URL o reporte Lighthouse → `audit`.
- Si estás editando archivos de página/componente sin reporte → `build`.
- Si es ambiguo, preguntar **una vez**: "¿Querés auditar una página existente o revisar código antes del deploy?"

### 2. Recolectar evidencia

**Pre-check obligatorio — API key del PageSpeed Insights**

Antes de cualquier recolección, verificar si `$PSI_API_KEY` está configurada:

```bash
if [ -z "$PSI_API_KEY" ]; then
  echo "PSI_API_KEY ausente"
fi
```

**Si la variable NO está seteada, pausar y avisar al usuario con este texto** (adaptar el tono a la conversación, mantener el contenido):

> ⚠️ Todavía no configuraste la API key del PageSpeed Insights (`$PSI_API_KEY`). Sin ella quedo limitado a ~1 query/día por IP — y se agota la quota rápido, forzándome a hacer auditoría solo estática (sin score, sin field data CrUX, sin audits del Lighthouse).
>
> **Con la key configurada, puedo ser mucho más certero:** traigo Performance Score real, LCP/CLS/INP/TBT/FCP medidos, opportunities priorizadas, breakdown del main thread, scripts pesados identificados, y datos de campo (CrUX) de usuarios reales en los últimos 28 días.
>
> Setup (2 min):
> 1. https://console.cloud.google.com/apis/credentials → crear API key
> 2. Habilitar **PageSpeed Insights API** y **Chrome UX Report API** en APIs & Services → Library
> 3. Agregar en `~/.zshrc`:
>    ```bash
>    export PSI_API_KEY="AIza..."
>    export CRUX_API_KEY="$PSI_API_KEY"
>    ```
> 4. `source ~/.zshrc` o abrir una terminal nueva
>
> ¿Querés configurarla ahora, o preferís que siga con auditoría estática limitada?

Si el usuario opta por configurarla: esperar y validar con un `curl` de prueba. Si opta por seguir sin: marcar en el reporte final **"Auditoría parcial — sin datos PSI/CrUX"** y basar conclusiones solo en análisis estático (HTML/headers/assets).

**Recolección efectiva:**

| Fuente | Qué extraer |
|--------|-------------|
| URL pública | Correr `curl -sI` para headers, descargar HTML, inspeccionar `<head>`, orden de los scripts, imágenes del hero |
| Reporte Lighthouse | Métricas (LCP/CLS/INP/TBT/FCP/SI), top opportunities, diagnostics, treemap JS |
| Código | Hero, orden de `<link>`/`<script>`, atributos de imagen, fuentes, CSS crítico, plugins (WP), bundle (Next) |
| PageSpeed Insights | Lab data **y** field data (CrUX) separados, mobile **y** desktop separados |

Para WebFetch / curl en URL pública, consultar [references/data-collection.md](references/data-collection.md). Para la mecánica de PSI/Lighthouse/CrUX (lab×field, pesos del score, INP, soft-nav, throttling, PSI API v5), consultar [references/psi-lighthouse-internals.md](references/psi-lighthouse-internals.md).

### 3. Clasificar por categoría

Todo análisis se organiza en **12 categorías** (ver [checks/](checks/)):

1. Resumen ejecutivo
2. Métricas PageSpeed (LCP, CLS, INP/TBT, FCP, SI)
3. Above-the-fold (hero, headline, CTA, CSS crítico)
4. Imágenes
5. Fuentes
6. CSS
7. JavaScript
8. Terceros y tracking
9. WordPress / Elementor
10. Estabilidad visual (CLS)
11. Red y servidor
12. Plan de acción

### 4. Asignar impacto y severidad

Cada problema recibe:
- **Métrica afectada** (LCP, CLS, INP, TBT, FCP, SI, TTFB).
- **Severidad** (crítica / alta / media / baja) — ver [references/severity-rubric.md](references/severity-rubric.md).
- **Prioridad** (P0 / P1 / P2) — basada en impacto × costo de corrección.
- **Riesgo de la corrección** (¿rompe layout? ¿rompe tracking? ¿requiere regresión visual?).

### 5. Recomendar corrección segura

Para cada problema:
- Qué cambiar exactamente (archivo / selector / atributo).
- Por qué eso afecta la métrica X.
- Qué NO tocar en simultáneo.
- Cómo testear después (Lighthouse run, WebPageTest, DevTools Performance, regresión visual).

Lista de correcciones seguras vs. arriesgadas: [references/safe-fixes.md](references/safe-fixes.md).

### 6. Emitir reporte

Template obligatorio: [references/report-format.md](references/report-format.md).

### 7. Decisión pre-deploy

- Cualquier problema **crítico** no resuelto → **bloquea deploy**.
- Múltiples **altos** en hero/LCP → **bloquea** hasta corregir los top 3.
- Solo **medios/bajos** → **aprobado con observaciones**.
- Todo verde + Core Web Vitals dentro del threshold → **aprobado**.

## Métricas y thresholds (referencia rápida)

Valores 2026 del Core Web Vitals (alineados con Google CrUX). Siempre validar contra [web.dev/vitals](https://web.dev/articles/vitals) si hay duda.

| Métrica | Bueno | Necesita mejorar | Malo |
|---------|-------|------------------|------|
| **LCP** (Largest Contentful Paint) | ≤ 2.5s | 2.5s – 4.0s | > 4.0s |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | 0.1 – 0.25 | > 0.25 |
| **INP** (Interaction to Next Paint) | ≤ 200ms | 200ms – 500ms | > 500ms |
| **TBT** (Total Blocking Time, lab) | ≤ 200ms | 200ms – 600ms | > 600ms |
| **FCP** (First Contentful Paint) | ≤ 1.8s | 1.8s – 3.0s | > 3.0s |
| **Speed Index** | ≤ 3.4s | 3.4s – 5.8s | > 5.8s |
| **TTFB** | ≤ 800ms | 800ms – 1800ms | > 1800ms |

Detalles en [references/metrics-glossary.md](references/metrics-glossary.md).

## Lab Data vs. Field Data — diferenciar SIEMPRE

- **Lab data (Lighthouse)** — simulación con throttling fijo (Moto G4 / 4G lento). Útil para diagnóstico.
- **Field data (CrUX)** — datos reales de usuarios Chrome en los últimos 28 días. Útil para priorización real.
- **Discrepancia común:** Lighthouse muestra LCP 4s pero CrUX muestra 2.1s — el usuario real está bien, el score del Lighthouse no. **Decidí con base en el field, validá con el lab.**
- Mobile y desktop **tienen rankings separados en Google**. Siempre evaluar ambos.

## Red Flags — STOP inmediatamente

Si te encontrás pensando o recomendando:

- "Diferir todos los scripts con `defer`" — sin mapear dependencias de tracking/inline.
- "Lazy-load en todas las imágenes" — incluyendo la imagen del LCP.
- "Remover el Pixel/GTM porque pesa mucho" — sin alinear con marketing.
- "Reemplazar Elementor por código nativo" — sin entender el ciclo de edición.
- "Comprimir el hero a 30KB" — perdiendo calidad visual del headline.
- "Remover CSS no usado automáticamente" — puede romper estados (hover, modal, error).
- "Score 100 en Lighthouse es la meta" — la meta es Core Web Vitals "good" en field data.
- "Migrar a Next.js / framework X resuelve" — sin entender el problema real.

→ STOP. Volvé a la etapa 4 (impacto × riesgo) antes de recomendar.

## Excusa | Realidad

| Excusa | Realidad |
|--------|----------|
| "Lighthouse solo corre en mobile lento, no es real" | Es el ranking que Google usa. Field data lo corrobora. |
| "Score 80 ya está bien" | Core Web Vitals es binario (pasa/no pasa). 80 puede fallar en LCP. |
| "Optimicé todo y el score bajó" | Probablemente rompiste el tracking inline o el LCP. Re-auditá. |
| "El cliente quiere 100" | 100 es frágil. Meta es "all green CWV" estable. Educá al cliente. |
| "Voy a usar el plugin X que hace todo" | Los plugins hacen el 60% y rompen el 30%. Auditá el resultado, no el plugin. |
| "Mobile no importa, la audiencia es desktop" | El mobile-first index de Google rankea por mobile. |
| "Saqué las fuentes custom y quedó rápido" | La conversión bajó junto. Optimizá la fuente, no la saques. |

## Checks por dominio

Usá el archivo de la check apropiada cuando la sesión toque esos archivos/áreas:

- [checks/hero-and-lcp.md](checks/hero-and-lcp.md) — above-the-fold, imagen LCP, headline.
- [checks/images.md](checks/images.md) — formato, dimensiones, lazy, preload, responsivo.
- [checks/fonts.md](checks/fonts.md) — Google Fonts, locales, `font-display`, preload, fallback.
- [checks/css.md](checks/css.md) — render-blocking, CSS crítico, unused CSS, Tailwind, Elementor CSS.
- [checks/javascript.md](checks/javascript.md) — defer/async, bundle, hydration, libs innecesarias.
- [checks/third-party-tracking.md](checks/third-party-tracking.md) — Pixel, GTM, GA4, Hotmart, AC, Make, chats.
- [checks/wordpress-elementor.md](checks/wordpress-elementor.md) — plugins, assets globales, caché, LiteSpeed, Cloudflare.
- [checks/layout-stability.md](checks/layout-stability.md) — CLS, reservar espacio, fonts swap, embeds.
- [checks/network-server.md](checks/network-server.md) — caché, CDN, Brotli, TTFB, preconnect.

## Reglas inviolables

1. **Nunca recomendar lazy-load en la imagen del LCP** — siempre `fetchpriority="high"` + sin `loading="lazy"`.
2. **Nunca remover ni mover scripts de tracking sin confirmar con el usuario** — puede romper la atribución.
3. **Siempre separar lab y field data** — no tratar el score Lighthouse como verdad absoluta.
4. **Siempre informar mobile y desktop por separado** — son rankings diferentes.
5. **Nunca recomendar `display: none` para esconder un problema de CLS** — corregir reservando espacio.
6. **Nunca dropear UTMs/query string** en optimizaciones de redirect o canonical.
7. **Toda recomendación tiene que apuntar archivo/línea cuando aplique** — sin "optimizá el CSS" genérico.
8. **Todo análisis tiene mobile-first como default** — desktop es complemento.
9. **Reporte siempre generado** — incluso si todo está verde.
10. **Score aislado no es criterio de aprobación** — Core Web Vitals en field sí lo es.

## Cuándo NO usar

- Auditoría de SEO contenido/keywords → usar `seo-audit`.
- Auditoría de accesibilidad profunda (WCAG AA/AAA) → skill dedicada de a11y.
- Auditoría de seguridad de headers → `security-auditor` / `seguranca`.
- Bug de renderizado visual (no relacionado con performance) → debug normal.
- Refactor de backend / API → fuera de alcance.

## Skills relacionadas

- **congruence** — verifica si los claims de la landing coinciden con lo que hace el código. Usala después de una pasada de performance-audit que haya tocado copy/CTA.
- **seo-audit** — auditoría de SEO técnico y on-page. Complementaria.
- **ux-ui-perfectionist** — revisión de UI/UX. Usala en conjunto si la optimización toca el layout.
- **ai-seo** — optimización para LLM search/AI Overviews. Complementaria.

## Por qué existe esta skill

Optimizar performance se volvió folclore: el equipo cambia de framework, compra plugin, activa "modo turbo" del CDN, y el score sube 5 puntos mientras la conversión cae 20%. Esta skill existe para **separar lo que el Lighthouse mide de lo que el usuario siente**, y garantizar que cada corrección:

- Tiene evidencia concreta en el reporte/código.
- Tiene prioridad basada en impacto × costo.
- Tiene test de regresión para layout, tracking y funcionalidad.
- No es cargo cult ("escuché que defer mejora").

El objetivo final no es el score. Es **la página que carga rápido para el usuario real, mantiene la conversión y además pasa los Core Web Vitals**.

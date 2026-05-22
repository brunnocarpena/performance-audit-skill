# performance-audit

> Auditoría de performance de páginas bajo la óptica de **Google PageSpeed Insights / Lighthouse**, sin sacrificar layout, tracking, UTMs, SEO ni conversión.

Skill creada para Claude Code. Inspirada estructuralmente en la skill [congruence](https://github.com/xBelowZero/congruence-skill) — mismo patrón de Iron Law, evidencia obligatoria, severidad explícita y decisión pre-deploy.

---

## Qué hace

Auditoría de performance que separa **lo que el Lighthouse mide** de **lo que el usuario siente**, y prioriza correcciones con:

- Impacto medible en la métrica correcta (LCP, CLS, INP, TBT, FCP).
- Costo de implementación realista.
- Riesgo explícito (¿rompe layout? ¿rompe tracking? ¿rompe conversión?).
- Plan de testeo post-corrección.

**No es** una optimización ciega. Toda recomendación preserva:

1. Layout visual.
2. Tracking (Pixel, GTM, GA4, Hotmart, AC, Make, conversiones).
3. UTMs y parámetros de URL.
4. Funcionalidad de formularios.
5. SEO (meta, structured data, canonical, hreflang).
6. Accesibilidad básica.

## Modos de operación

- **`audit`** — auditoría de página existente (URL, código, reporte Lighthouse).
- **`build`** — revisión de código antes del deploy, garantizando performance desde el inicio.

## Cuándo usar

Usala **siempre que**:
- Necesites auditar una página (landing, WP/Elementor, HTML/Tailwind, Next.js, captura).
- Estés desarrollando una página nueva y quieras garantizar buen puntaje desde el inicio.
- Antes del deploy de cualquier página pública.
- Estés investigando una caída de Core Web Vitals.
- Estés investigando una caída de conversión sospechosa relacionada con velocidad.

Invocar con `/performance-audit` o "auditá la performance de esta página".

## Instalación

```bash
# 1. Clonar dentro de ~/.claude/skills/
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/xBelowZero/claude-skill-performance-audit.git performance-audit

# 2. (Opcional pero recomendado) configurar API key del PageSpeed Insights
# Sin key: ~1 query/día por IP. Con key: 400 QPS / 25k queries/día.
#
#   - https://console.cloud.google.com/apis/credentials → crear API key
#   - Habilitar "PageSpeed Insights API" y "Chrome UX Report API"
#   - Agregar en ~/.zshrc (o ~/.bashrc):
#
#       export PSI_API_KEY="AIza..."
#       export CRUX_API_KEY="$PSI_API_KEY"
#
#   - source ~/.zshrc
```

Validar instalación:

```bash
ls ~/.claude/skills/performance-audit/SKILL.md  # debe existir
echo $PSI_API_KEY                                # debe mostrar la key (opcional)
```

En cualquier sesión Claude Code:

```
/performance-audit https://ejemplo.com --mode=audit --target=mobile
```

## Cuándo NO usar

- Auditoría de SEO contenido → `seo-audit`.
- Auditoría de accesibilidad WCAG → skill dedicada.
- Auditoría de seguridad → `security-auditor`.
- Bug visual sin relación con performance.

## Estructura

```
performance-audit/
├── SKILL.md                          # Instrucciones principales, Iron Law, workflow
├── README.md                         # Este archivo
├── references/                       # Guías profundas
│   ├── metrics-glossary.md           #   LCP/CLS/INP/TBT/FCP/SI/TTFB
│   ├── data-collection.md            #   Cómo recolectar lab + field data
│   ├── severity-rubric.md            #   Rúbrica crítica/alta/media/baja
│   ├── safe-fixes.md                 #   Correcciones por riesgo
│   └── report-format.md              #   Template del reporte
└── checks/                           # Auditoría por dominio
    ├── hero-and-lcp.md
    ├── images.md
    ├── fonts.md
    ├── css.md
    ├── javascript.md
    ├── third-party-tracking.md
    ├── wordpress-elementor.md
    ├── layout-stability.md
    └── network-server.md
```

## Cómo funciona

1. Identifica alcance y modo (audit vs build).
2. Recolecta evidencia: Lighthouse + PageSpeed + CrUX + código.
3. Clasifica en 12 categorías (hero, imágenes, fonts, CSS, JS, third-party, WP, CLS, red).
4. Asigna métrica afectada + severidad + prioridad + riesgo.
5. Recomienda correcciones **seguras**, siempre indicando archivo/línea y qué NO tocar en simultáneo.
6. Emite reporte estandarizado.
7. Decide pre-deploy: aprobado / con observaciones / bloqueado.

## Reglas inviolables

1. **La imagen del LCP nunca lleva `loading="lazy"`** — siempre `fetchpriority="high"`.
2. **El tracking no se toca sin confirmación explícita del usuario**.
3. **Lab y field data siempre separados** — el score Lighthouse no es verdad absoluta.
4. **Mobile y desktop siempre separados** — son rankings diferentes de Google.
5. **Las UTMs nunca se dropean** en redirects/canonical.
6. **Toda recomendación cita archivo/línea o audit ID**.
7. **Mobile-first por default**.
8. **Reporte siempre generado**, incluso si todo está verde.
9. **El score aislado no aprueba un deploy** — Core Web Vitals "good" en field es el criterio.
10. **Toda corrección riesgosa tiene plan de rollback**.

## Métricas y thresholds (2026)

| Métrica | Bueno | Necesita mejorar | Malo |
|---------|-------|------------------|------|
| LCP | ≤ 2.5s | 2.5–4.0s | > 4.0s |
| CLS | ≤ 0.1 | 0.1–0.25 | > 0.25 |
| INP | ≤ 200ms | 200–500ms | > 500ms |
| TBT | ≤ 200ms | 200–600ms | > 600ms |
| FCP | ≤ 1.8s | 1.8–3.0s | > 3.0s |
| TTFB | ≤ 800ms | 800–1800ms | > 1800ms |

Criterio Google: **p75 ≥ "good" en el 75% de los usuarios en CrUX**.

## Inspiración y referencias

Estructura inspirada en:
- [xBelowZero/congruence-skill](https://github.com/xBelowZero/congruence-skill) — patrón de evidencia obligatoria + Iron Law.
- [addyosmani/web-quality-skills](https://github.com/addyosmani/web-quality-skills) — Agent Skills oficiales del equipo Chrome para Lighthouse + CWV.
- [web.dev/articles/vitals](https://web.dev/articles/vitals) — fuente canónica de los thresholds.
- [GoogleChrome/lighthouse-ci](https://github.com/GoogleChrome/lighthouse-ci) — budgets y regresiones en PR.

## Skills relacionadas

- **congruence** — verifica si los claims de la página coinciden con el código. Usala después si la auditoría tocó copy/CTA.
- **seo-audit** — SEO técnico y on-page. Complementaria.
- **ux-ui-perfectionist** — revisión de UI/UX. Usala en conjunto si la optimización toca el layout.

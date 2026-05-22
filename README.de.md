# Performance Audit — Score 90+ in Lighthouse, Ohne Tracking oder Conversion zu Beschädigen

> Performance-Audit von Seiten aus der Perspektive von **Google PageSpeed Insights / Lighthouse**, ohne Layout, Tracking, UTMs, SEO oder Conversion zu opfern.


---

## Was sie tut

Ein Performance-Audit, das **was Lighthouse misst** von **dem, was der Nutzer spürt**, trennt und Korrekturen priorisiert mit:

- Messbarem Impact auf der richtigen Metrik (LCP, CLS, INP, TBT, FCP).
- Realistischen Implementierungskosten.
- Explizitem Risiko (bricht Layout? bricht Tracking? bricht Conversion?).
- Testplan nach der Korrektur.

**Es ist keine** blinde Optimierung. Jede Empfehlung bewahrt:

1. Das visuelle Layout.
2. Tracking (Pixel, GTM, GA4, Hotmart, AC, Make, Conversions).
3. UTMs und URL-Parameter.
4. Formular-Funktionalität.
5. SEO (Meta, Structured Data, Canonical, hreflang).
6. Grundlegende Barrierefreiheit.

## Betriebsmodi

- **`audit`** — Audit einer bestehenden Seite (URL, Code, Lighthouse-Report).
- **`build`** — Code-Review vor dem Deploy, sichert Performance von Anfang an.

## Wann verwenden

**Immer** verwenden, wenn:
- Du eine Seite auditieren musst (Landing, WP/Elementor, HTML/Tailwind, Next.js, Capture).
- Du eine neue Seite entwickelst und von Anfang an einen guten Score sicherstellen willst.
- Vor jedem Deploy einer öffentlichen Seite.
- Du einen Rückgang der Core Web Vitals untersuchst.
- Du einen verdächtigen, geschwindigkeitsbezogenen Conversion-Rückgang untersuchst.

Aufruf mit `/performance-audit` oder „auditiere die Performance dieser Seite".

## Installation

```bash
# 1. In ~/.claude/skills/ klonen
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/brunnocarpena/performance-audit-skill.git performance-audit

# 2. (Optional, aber empfohlen) PageSpeed Insights API-Key konfigurieren
# Ohne Key: ~1 Query/Tag keyed pro IP. Mit Key: 400 QPS / 25k Queries/Tag.
#
#   - https://console.cloud.google.com/apis/credentials → API-Key erstellen
#   - "PageSpeed Insights API" und "Chrome UX Report API" aktivieren
#   - In ~/.zshrc (oder ~/.bashrc) ergänzen:
#
#       export PSI_API_KEY="AIza..."
#       export CRUX_API_KEY="$PSI_API_KEY"
#
#   - source ~/.zshrc
```

Installation validieren:

```bash
ls ~/.claude/skills/performance-audit/SKILL.md  # muss existieren
echo $PSI_API_KEY                                # sollte den Key zeigen (optional)
```

In jeder Claude-Code-Session:

```
/performance-audit https://beispiel.com --mode=audit --target=mobile
```

## Wann NICHT verwenden

- SEO-Content-Audit → `seo-audit`.
- WCAG-Accessibility-Audit → dedizierte Skill.
- Security-Audit → `security-auditor`.
- Visueller Bug ohne Performance-Bezug.

## Struktur

```
performance-audit/
├── SKILL.md                          # Hauptanweisungen, Iron Law, Workflow
├── README.md                         # Diese Datei
├── references/                       # Tiefe Guides
│   ├── metrics-glossary.md           #   LCP/CLS/INP/TBT/FCP/SI/TTFB
│   ├── data-collection.md            #   Wie Lab + Field Data gesammelt werden
│   ├── severity-rubric.md            #   Rubrik kritisch/hoch/mittel/niedrig
│   ├── safe-fixes.md                 #   Korrekturen nach Risiko
│   └── report-format.md              #   Report-Template
└── checks/                           # Audit pro Domäne
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

## Wie es funktioniert

1. Identifiziert Scope und Modus (audit vs build).
2. Sammelt Evidenz: Lighthouse + PageSpeed + CrUX + Code.
3. Klassifiziert in 12 Kategorien (Hero, Bilder, Fonts, CSS, JS, Third-Party, WP, CLS, Netz).
4. Weist betroffene Metrik + Severity + Priorität + Risiko zu.
5. Empfiehlt **sichere** Korrekturen, immer mit Datei/Zeile und was NICHT gleichzeitig angefasst werden darf.
6. Gibt standardisierten Report aus.
7. Entscheidet Pre-Deploy: freigegeben / mit Vorbehalten / blockiert.

## Unverletzliche Regeln

1. **Das LCP-Bild trägt nie `loading="lazy"`** — immer `fetchpriority="high"`.
2. **Tracking wird nicht ohne explizite Bestätigung des Nutzers angefasst**.
3. **Lab und Field Data immer getrennt** — der Lighthouse-Score ist keine absolute Wahrheit.
4. **Mobile und Desktop immer getrennt** — sind unterschiedliche Google-Rankings.
5. **UTMs werden nie gedroppt** in Redirects/Canonical.
6. **Jede Empfehlung zitiert Datei/Zeile oder Audit-ID**.
7. **Standardmäßig mobile-first**.
8. **Report wird immer generiert**, auch wenn alles grün ist.
9. **Isolierter Score gibt keinen Deploy frei** — Core Web Vitals „good" im Field ist das Kriterium.
10. **Jede riskante Korrektur hat einen Rollback-Plan**.

## Metriken und Thresholds (2026)

| Metrik | Gut | Verbesserungsbedürftig | Schlecht |
|--------|-----|------------------------|----------|
| LCP | ≤ 2,5 s | 2,5–4,0 s | > 4,0 s |
| CLS | ≤ 0,1 | 0,1–0,25 | > 0,25 |
| INP | ≤ 200 ms | 200–500 ms | > 500 ms |
| TBT | ≤ 200 ms | 200–600 ms | > 600 ms |
| FCP | ≤ 1,8 s | 1,8–3,0 s | > 3,0 s |
| TTFB | ≤ 800 ms | 800–1800 ms | > 1800 ms |

Google-Kriterium: **p75 ≥ „good" bei 75 % der Nutzer im CrUX**.

## Inspiration und Referenzen

Struktur inspiriert von:
- [brunnocarpena/congruence-skill](https://github.com/brunnocarpena/congruence-skill) — Pattern verpflichtender Evidenz + Iron Law.
- [addyosmani/web-quality-skills](https://github.com/addyosmani/web-quality-skills) — offizielle Agent Skills des Chrome-Teams für Lighthouse + CWV.
- [web.dev/articles/vitals](https://web.dev/articles/vitals) — kanonische Quelle der Thresholds.
- [GoogleChrome/lighthouse-ci](https://github.com/GoogleChrome/lighthouse-ci) — Budgets und Regressionen im PR.

## Verwandte Skills

- **congruence** — prüft, ob Claims der Seite zum Code passen. Danach verwenden, wenn das Audit Copy/CTA verändert hat.
- **seo-audit** — technisches und On-Page-SEO. Ergänzend.
- **ux-ui-perfectionist** — UI/UX-Review. Gemeinsam verwenden, wenn die Optimierung das Layout berührt.

---
name: performance-audit
description: Immer dann verwenden, wenn du die Performance einer Seite (Landingpage, WordPress/Elementor, HTML/Tailwind, Next.js, Lead-Capture-Seite) aus der Perspektive von Google PageSpeed Insights / Lighthouse auditieren musst, ODER wenn du eine neue Seite entwickelst und von Anfang an einen guten Score sicherstellen willst. TRIGGER, wenn der Agent „PageSpeed", „Lighthouse", „Core Web Vitals", „LCP", „CLS", „INP", „TBT", „Performance", „langsame Seite", „niedriger Score", „Seite optimieren" erwähnt; wenn die Session Dateien für Hero, Bilder, Fonts, Critical CSS, GTM/Pixel/Analytics, Cache-Plugins, Elementor berührt; vor jedem Deploy einer öffentlichen Seite. SKIP bei rein internen Änderungen (API, Jobs, Schema) ohne Auswirkung auf das Frontend-Rendering.
argument-hint: [url-oder-scope?] [--mode=audit|build] [--target=mobile|desktop|both]
allowed-tools: Read, Grep, Glob, Bash(curl:*), Bash(git diff:*), Bash(git status:*), Bash(ls:*), Bash(find:*), Bash(wc:*), WebFetch, Task
---

# Performance Audit

Performance-Audit von Seiten aus der Perspektive von **Google PageSpeed Insights / Lighthouse**, ohne Layout, Tracking, UTMs, SEO oder Conversion zu opfern.

**Diese Skill ist keine blinde Optimierung.** Es ist ein durchdachtes Audit, das trennt, was Lighthouse misst, von dem, was der Nutzer spürt, und Korrekturen priorisiert, die echten Impact liefern, ohne die Seite zu brechen.

## Kernprinzip

```
PERFORMANCE IST KEIN SCORE. ES GEHT DARUM, DEN ABOVE-THE-FOLD SCHNELL ZU LIEFERN,
VISUELLE STABILITÄT ZU WAHREN UND WEDER TRACKING NOCH CONVERSION ZU BRECHEN.
```

Ein Score von 100 mit kaputtem Formular, nicht feuerndem Pixel oder verlorenem UTM **ist eine Regression**, kein Sieg. Jede Empfehlung dieser Skill muss Folgendes bewahren:

1. **Visuelles Layout** — kein eingeführtes CLS, kein Brechen von Grid/Komponenten.
2. **Tracking** — Meta Pixel, GTM, GA4, Hotmart, ActiveCampaign, Make, Conversions.
3. **UTMs und URL-Parameter** — Query-String niemals in Redirects/Canonical droppen.
4. **Formular-Funktionalität** — Felder, Validierung, Action, Integrationen.
5. **SEO** — Meta Tags, Structured Data, hreflang, Canonical, Robots.
6. **Grundlegende Barrierefreiheit** — alt, Labels, Kontrast, Fokus.

Wenn eine „Optimierung" irgendeinen dieser Punkte gefährdet, **empfiehl sie nicht ohne expliziten Hinweis**.

## Betriebsmodi

Die Skill arbeitet in zwei Modi. Identifiziere den Modus zu Beginn:

### Modus `audit` — Audit einer bestehenden Seite

Akzeptierte Inputs:
- Öffentliche URL.
- Screenshot/JSON von PageSpeed Insights.
- Lighthouse-Report (HTML/JSON).
- Quellcode (HTML, Elementor-Template, Next.js-Komponente).
- Spezifischer Ausschnitt (Hero, Head, Footer).

Output: priorisierte Diagnose + Korrekturplan (siehe [references/report-format.md](references/report-format.md)).

### Modus `build` — Entwicklung einer neuen Seite

Best Practices anwenden, **bevor** der Code existiert. HTML/CSS/JS beim Commit reviewen. Hero, Bilder, Fonts, Scripts und Tracking gemäß den Checks in [checks/](checks/) validieren.

Output: Pre-Deploy-Checkliste + Risikohinweise im aktuellen Code.

## Workflow

### 1. Scope und Modus identifizieren

- Wenn es eine URL oder einen Lighthouse-Report gibt → `audit`.
- Wenn du Seiten-/Komponenten-Dateien ohne Report bearbeitest → `build`.
- Bei Unklarheit **einmal** fragen: „Willst du eine bestehende Seite auditieren oder Code vor dem Deploy reviewen?"

### 2. Evidenz sammeln

**Verpflichtender Pre-Check — PageSpeed Insights API Key**

Vor jeder Sammlung prüfen, ob `$PSI_API_KEY` gesetzt ist:

```bash
if [ -z "$PSI_API_KEY" ]; then
  echo "PSI_API_KEY fehlt"
fi
```

**Wenn die Variable NICHT gesetzt ist, pausieren und den Nutzer mit folgendem Text informieren** (Tonalität an das Gespräch anpassen, Inhalt beibehalten):

> ⚠️ Du hast den API-Key für PageSpeed Insights (`$PSI_API_KEY`) noch nicht konfiguriert. Ohne ihn bin ich auf ~1 Query/Tag keyed pro IP limitiert — das Quota ist schnell aufgebraucht und ich muss auf rein statisches Audit ausweichen (kein Score, keine CrUX-Feld-Daten, keine Lighthouse-Audits).
>
> **Mit konfiguriertem Key kann ich deutlich präziser arbeiten:** ich liefere echten Performance Score, gemessene LCP/CLS/INP/TBT/FCP, priorisierte Opportunities, Main-Thread-Breakdown, identifizierte schwere Scripts und Felddaten (CrUX) realer Nutzer aus den letzten 28 Tagen.
>
> Setup (2 Min):
> 1. https://console.cloud.google.com/apis/credentials → API-Key erstellen
> 2. **PageSpeed Insights API** und **Chrome UX Report API** in APIs & Services → Library aktivieren
> 3. In `~/.zshrc` ergänzen:
>    ```bash
>    export PSI_API_KEY="AIza..."
>    export CRUX_API_KEY="$PSI_API_KEY"
>    ```
> 4. `source ~/.zshrc` oder neues Terminal öffnen
>
> Willst du das jetzt einrichten, oder soll ich mit einem eingeschränkten statischen Audit fortfahren?

Wenn der Nutzer das Setup wählt: warten und mit einem Test-`curl` validieren. Wenn er ohne Key fortfahren will: im finalen Report **„Teil-Audit — ohne PSI/CrUX-Daten"** markieren und Schlussfolgerungen nur auf statischer Analyse (HTML/Headers/Assets) basieren.

**Tatsächliche Sammlung:**

| Quelle | Was extrahieren |
|--------|-----------------|
| Öffentliche URL | `curl -sI` für Headers, HTML herunterladen, `<head>` inspizieren, Reihenfolge der Scripts, Hero-Bilder |
| Lighthouse-Report | Metriken (LCP/CLS/INP/TBT/FCP/SI), Top Opportunities, Diagnostics, JS-Treemap |
| Code | Hero, Reihenfolge von `<link>`/`<script>`, Bild-Attribute, Fonts, Critical CSS, Plugins (WP), Bundle (Next) |
| PageSpeed Insights | Lab Data **und** Field Data (CrUX) getrennt, mobile **und** desktop getrennt |

Für WebFetch / curl auf einer öffentlichen URL siehe [references/data-collection.md](references/data-collection.md). Für die Mechanik von PSI/Lighthouse/CrUX (lab×field, Score-Gewichtungen, INP, Soft-Nav, Throttling, PSI API v5) siehe [references/psi-lighthouse-internals.md](references/psi-lighthouse-internals.md).

### 3. Nach Kategorie klassifizieren

Jede Analyse wird in **12 Kategorien** organisiert (siehe [checks/](checks/)):

1. Executive Summary
2. PageSpeed-Metriken (LCP, CLS, INP/TBT, FCP, SI)
3. Above-the-fold (Hero, Headline, CTA, Critical CSS)
4. Bilder
5. Fonts
6. CSS
7. JavaScript
8. Drittparteien und Tracking
9. WordPress / Elementor
10. Visuelle Stabilität (CLS)
11. Netzwerk und Server
12. Aktionsplan

### 4. Impact und Severity zuweisen

Jedes Problem erhält:
- **Betroffene Metrik** (LCP, CLS, INP, TBT, FCP, SI, TTFB).
- **Severity** (kritisch / hoch / mittel / niedrig) — siehe [references/severity-rubric.md](references/severity-rubric.md).
- **Priorität** (P0 / P1 / P2) — basierend auf Impact × Korrekturkosten.
- **Risiko der Korrektur** (bricht Layout? bricht Tracking? erfordert visuelle Regression?).

### 5. Sichere Korrektur empfehlen

Für jedes Problem:
- Was genau zu ändern ist (Datei / Selector / Attribut).
- Warum es Metrik X beeinflusst.
- Was NICHT gleichzeitig angefasst werden darf.
- Wie danach getestet wird (Lighthouse-Run, WebPageTest, DevTools Performance, visuelle Regression).

Liste sicherer vs. riskanter Korrekturen: [references/safe-fixes.md](references/safe-fixes.md).

### 6. Report ausgeben

Verpflichtendes Template: [references/report-format.md](references/report-format.md).

### 7. Pre-Deploy-Entscheidung

- Jedes ungelöste **kritische** Problem → **blockiert Deploy**.
- Mehrere **hohe** auf Hero/LCP → **blockiert**, bis die Top 3 behoben sind.
- Nur **mittel/niedrig** → **freigegeben mit Vorbehalten**.
- Alles grün + Core Web Vitals im Threshold → **freigegeben**.

## Metriken und Thresholds (Schnellreferenz)

Werte 2026 für Core Web Vitals (abgestimmt mit Google CrUX). Im Zweifel immer gegen [web.dev/vitals](https://web.dev/articles/vitals) validieren.

| Metrik | Gut | Verbesserungsbedürftig | Schlecht |
|--------|-----|------------------------|----------|
| **LCP** (Largest Contentful Paint) | ≤ 2,5 s | 2,5 s – 4,0 s | > 4,0 s |
| **CLS** (Cumulative Layout Shift) | ≤ 0,1 | 0,1 – 0,25 | > 0,25 |
| **INP** (Interaction to Next Paint) | ≤ 200 ms | 200 ms – 500 ms | > 500 ms |
| **TBT** (Total Blocking Time, lab) | ≤ 200 ms | 200 ms – 600 ms | > 600 ms |
| **FCP** (First Contentful Paint) | ≤ 1,8 s | 1,8 s – 3,0 s | > 3,0 s |
| **Speed Index** | ≤ 3,4 s | 3,4 s – 5,8 s | > 5,8 s |
| **TTFB** | ≤ 800 ms | 800 ms – 1800 ms | > 1800 ms |

Details in [references/metrics-glossary.md](references/metrics-glossary.md).

## Lab Data vs. Field Data — IMMER unterscheiden

- **Lab Data (Lighthouse)** — Simulation mit festem Throttling (Moto G4 / langsames 4G). Nützlich für Diagnose.
- **Field Data (CrUX)** — echte Daten von Chrome-Nutzern der letzten 28 Tage. Nützlich für reale Priorisierung.
- **Häufige Diskrepanz:** Lighthouse zeigt LCP 4 s, aber CrUX zeigt 2,1 s — der echte Nutzer ist okay, der Lighthouse-Score nicht. **Entscheide auf Basis von Field, validiere mit Lab.**
- Mobile und Desktop **haben bei Google getrennte Rankings**. Immer beide bewerten.

## Red Flags — sofort STOPP

Wenn du dich dabei ertappst, zu denken oder zu empfehlen:

- „Alle Scripts mit `defer` aufschieben" — ohne Tracking-/Inline-Abhängigkeiten gemappt zu haben.
- „Lazy-Load auf alle Bilder" — inklusive LCP-Bild.
- „Pixel/GTM entfernen, weil zu schwer" — ohne Abstimmung mit Marketing.
- „Elementor durch nativen Code ersetzen" — ohne den Edit-Zyklus zu verstehen.
- „Hero auf 30 KB komprimieren" — unter Verlust visueller Qualität des Headlines.
- „Ungenutztes CSS automatisch entfernen" — kann States brechen (hover, modal, error).
- „Lighthouse-Score 100 ist das Ziel" — Ziel sind Core Web Vitals „good" im Field Data.
- „Migration auf Next.js / Framework X löst es" — ohne das echte Problem zu verstehen.

→ STOPP. Zurück zu Schritt 4 (Impact × Risiko), bevor du etwas empfiehlst.

## Ausrede | Realität

| Ausrede | Realität |
|---------|----------|
| „Lighthouse läuft nur auf langsamem Mobile, das ist nicht real" | Es ist das Ranking, das Google nutzt. Field Data bestätigt es. |
| „Score 80 ist doch okay" | Core Web Vitals sind binär (pass/fail). 80 kann beim LCP durchfallen. |
| „Ich habe alles optimiert und der Score ist gefallen" | Du hast wahrscheinlich Inline-Tracking oder LCP gebrochen. Re-Audit. |
| „Der Kunde will 100" | 100 ist fragil. Ziel ist stabiles „all green CWV". Educate den Kunden. |
| „Ich nehme Plugin X, das macht alles" | Plugins erledigen 60 % und brechen 30 %. Auditiere das Ergebnis, nicht das Plugin. |
| „Mobile ist egal, die Audience ist Desktop" | Googles Mobile-First-Index rankt nach Mobile. |
| „Ich habe die Custom Fonts entfernt und es war schnell" | Die Conversion ist mitgefallen. Optimiere den Font, entferne ihn nicht. |

## Checks pro Domäne

Nutze die passende Check-Datei, wenn die Session diese Dateien/Bereiche berührt:

- [checks/hero-and-lcp.md](checks/hero-and-lcp.md) — Above-the-fold, LCP-Bild, Headline.
- [checks/images.md](checks/images.md) — Format, Maße, Lazy, Preload, Responsive.
- [checks/fonts.md](checks/fonts.md) — Google Fonts, lokale Fonts, `font-display`, Preload, Fallback.
- [checks/css.md](checks/css.md) — Render-blocking, Critical CSS, Unused CSS, Tailwind, Elementor CSS.
- [checks/javascript.md](checks/javascript.md) — defer/async, Bundle, Hydration, unnötige Libs.
- [checks/third-party-tracking.md](checks/third-party-tracking.md) — Pixel, GTM, GA4, Hotmart, AC, Make, Chats.
- [checks/wordpress-elementor.md](checks/wordpress-elementor.md) — Plugins, globale Assets, Cache, LiteSpeed, Cloudflare.
- [checks/layout-stability.md](checks/layout-stability.md) — CLS, Platz reservieren, Font-Swap, Embeds.
- [checks/network-server.md](checks/network-server.md) — Cache, CDN, Brotli, TTFB, Preconnect.

## Unverletzliche Regeln

1. **Niemals Lazy-Load auf dem LCP-Bild empfehlen** — immer `fetchpriority="high"` + ohne `loading="lazy"`.
2. **Niemals Tracking-Scripts ohne Bestätigung des Nutzers entfernen oder verschieben** — kann Attribution brechen.
3. **Lab und Field Data immer trennen** — Lighthouse-Score nicht als absolute Wahrheit behandeln.
4. **Mobile und Desktop immer getrennt berichten** — sind unterschiedliche Rankings.
5. **Niemals `display: none` empfehlen, um ein CLS-Problem zu verstecken** — durch Reservieren von Platz korrigieren.
6. **Niemals UTMs/Query-String droppen** in Redirect- oder Canonical-Optimierungen.
7. **Jede Empfehlung muss Datei/Zeile nennen, wo zutreffend** — kein generisches „optimiere das CSS".
8. **Jede Analyse ist standardmäßig mobile-first** — Desktop ist Ergänzung.
9. **Report immer generiert** — auch wenn alles grün ist.
10. **Isolierter Score ist kein Freigabekriterium** — Core Web Vitals im Field sind es.

## Wann NICHT verwenden

- SEO-Content/Keyword-Audit → `seo-audit` verwenden.
- Tiefes Accessibility-Audit (WCAG AA/AAA) → dedizierte a11y-Skill.
- Header-Security-Audit → `security-auditor` / `seguranca`.
- Visueller Rendering-Bug (nicht performance-bezogen) → normales Debugging.
- Backend-/API-Refactor → außerhalb des Scope.

## Verwandte Skills

- **congruence** — prüft, ob Claims auf der Landingpage zu dem passen, was der Code tatsächlich tut. Verwende danach, wenn ein Performance-Audit Copy/CTA verändert hat.
- **seo-audit** — technisches und On-Page-SEO-Audit. Ergänzend.
- **ux-ui-perfectionist** — UI/UX-Review. Verwende gemeinsam, wenn die Optimierung das Layout berührt.
- **ai-seo** — Optimierung für LLM-Search/AI Overviews. Ergänzend.

## Warum es diese Skill gibt

Performance-Optimierung ist zur Folklore geworden: Das Team wechselt das Framework, kauft ein Plugin, aktiviert den „Turbo-Modus" des CDN, der Score steigt um 5 Punkte, während die Conversion um 20 % fällt. Diese Skill existiert, um **zu trennen, was Lighthouse misst, von dem, was der Nutzer spürt**, und sicherzustellen, dass jede Korrektur:

- Konkrete Evidenz im Report/Code hat.
- Priorität basierend auf Impact × Kosten hat.
- Einen Regressionstest für Layout, Tracking und Funktionalität hat.
- Kein Cargo Cult ist („ich hab gehört, defer macht's besser").

Das Endziel ist nicht der Score. Es ist **die Seite, die für echte Nutzer schnell lädt, die Conversion hält und außerdem die Core Web Vitals besteht**.

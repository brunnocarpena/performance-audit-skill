---
name: performance-audit
description: À utiliser dès que tu dois auditer la performance d'une page (landing, WordPress/Elementor, HTML/Tailwind, Next.js, page de capture) sous l'angle de Google PageSpeed Insights / Lighthouse, OU quand tu développes une nouvelle page et que tu veux garantir un bon score dès le départ. TRIGGER quand l'agent mentionne "PageSpeed", "Lighthouse", "Core Web Vitals", "LCP", "CLS", "INP", "TBT", "performance", "page lente", "score bas", "optimiser une page" ; quand la session touche aux fichiers du hero, images, polices, CSS critique, GTM/Pixel/Analytics, plugins de cache, Elementor ; avant tout déploiement de page publique. SKIP sur les changements purement internes (API, jobs, schema) sans impact sur le rendu front.
argument-hint: [url-ou-périmètre?] [--mode=audit|build] [--target=mobile|desktop|both]
allowed-tools: Read, Grep, Glob, Bash(curl:*), Bash(git diff:*), Bash(git status:*), Bash(ls:*), Bash(find:*), Bash(wc:*), WebFetch, Task
---

# Performance Audit

Audit de performance de pages sous l'angle de **Google PageSpeed Insights / Lighthouse**, sans sacrifier le layout, le tracking, les UTMs, le SEO ni la conversion.

**Cette skill n'est pas une optimisation aveugle.** C'est un audit raisonné, qui sépare ce que Lighthouse mesure de ce que l'utilisateur ressent, et qui priorise les corrections qui apportent un impact réel sans casser la page.

## Principe central

```
LA PERFORMANCE N'EST PAS UN SCORE. C'EST LIVRER L'ABOVE-THE-FOLD VITE,
MAINTENIR LA STABILITÉ VISUELLE ET NE CASSER NI TRACKING NI CONVERSION.
```

Un score 100 avec un formulaire cassé, un Pixel qui ne se déclenche pas ou un UTM perdu **est une régression**, pas une victoire. Toute recommandation de cette skill doit préserver :

1. **Layout visuel** — sans CLS introduit, sans casser la grille/les composants.
2. **Tracking** — Meta Pixel, GTM, GA4, Hotmart, ActiveCampaign, Make, conversions.
3. **UTMs et paramètres d'URL** — ne jamais dropper la query string dans les redirects/canonical.
4. **Fonctionnement des formulaires** — champs, validation, action, intégrations.
5. **SEO** — meta tags, structured data, hreflang, canonical, robots.
6. **Accessibilité de base** — alt, labels, contraste, focus.

Si une "optimisation" met l'un de ces éléments en danger, **ne la recommande pas sans alerte explicite**.

## Modes de fonctionnement

La skill opère en deux modes. Identifie le mode dès le début :

### Mode `audit` — audit d'une page existante

Inputs acceptés :
- URL publique.
- Capture/JSON du PageSpeed Insights.
- Rapport Lighthouse (HTML/JSON).
- Code source (HTML, template Elementor, composant Next.js).
- Fragment spécifique (hero, head, footer).

Output : diagnostic priorisé + plan de correction (voir [references/report-format.md](references/report-format.md)).

### Mode `build` — développement d'une nouvelle page

Appliquer les bonnes pratiques **avant** que le code n'existe. Revoir HTML/CSS/JS au commit. Valider hero, images, polices, scripts et tracking selon les checks dans [checks/](checks/).

Output : checklist pré-déploiement + signalements de risques dans le code actuel.

## Workflow

### 1. Identifier le périmètre et le mode

- S'il y a une URL ou un rapport Lighthouse → `audit`.
- Si tu édites des fichiers de page/composant sans rapport → `build`.
- Si c'est ambigu, demander **une seule fois** : "Tu veux auditer une page existante ou revoir du code avant déploiement ?"

### 2. Collecter les preuves

**Pré-check obligatoire — API key du PageSpeed Insights**

Avant toute collecte, vérifier si `$PSI_API_KEY` est configurée :

```bash
if [ -z "$PSI_API_KEY" ]; then
  echo "PSI_API_KEY absente"
fi
```

**Si la variable N'est PAS définie, faire une pause et avertir l'utilisateur avec ce texte** (adapter le ton à la conversation, conserver le contenu) :

> ⚠️ Tu n'as pas encore configuré la clé API du PageSpeed Insights (`$PSI_API_KEY`). Sans elle, je suis limité à ~1 requête/jour keyed par IP — le quota s'épuise vite et je suis forcé à un audit purement statique (pas de score, pas de field data CrUX, pas d'audits Lighthouse).
>
> **Avec la clé configurée, je peux être beaucoup plus précis :** je récupère le Performance Score réel, LCP/CLS/INP/TBT/FCP mesurés, opportunities priorisées, breakdown du main thread, scripts lourds identifiés, et données de terrain (CrUX) d'utilisateurs réels sur les 28 derniers jours.
>
> Setup (2 min) :
> 1. https://console.cloud.google.com/apis/credentials → créer une API key
> 2. Activer **PageSpeed Insights API** et **Chrome UX Report API** dans APIs & Services → Library
> 3. Ajouter dans `~/.zshrc` :
>    ```bash
>    export PSI_API_KEY="AIza..."
>    export CRUX_API_KEY="$PSI_API_KEY"
>    ```
> 4. `source ~/.zshrc` ou ouvrir un nouveau terminal
>
> Tu veux la configurer maintenant, ou tu préfères que je continue avec un audit statique limité ?

Si l'utilisateur choisit de configurer : attendre et valider avec un `curl` de test. S'il choisit de continuer sans : marquer dans le rapport final **"Audit partiel — sans données PSI/CrUX"** et baser les conclusions uniquement sur l'analyse statique (HTML/headers/assets).

**Collecte effective :**

| Source | Ce qu'il faut extraire |
|--------|------------------------|
| URL publique | Lancer `curl -sI` pour les headers, télécharger le HTML, inspecter le `<head>`, l'ordre des scripts, les images du hero |
| Rapport Lighthouse | Métriques (LCP/CLS/INP/TBT/FCP/SI), top opportunities, diagnostics, treemap JS |
| Code | Hero, ordre des `<link>`/`<script>`, attributs d'image, polices, CSS critique, plugins (WP), bundle (Next) |
| PageSpeed Insights | Lab data **et** field data (CrUX) séparés, mobile **et** desktop séparés |

Pour WebFetch / curl sur une URL publique, voir [references/data-collection.md](references/data-collection.md). Pour la mécanique PSI/Lighthouse/CrUX (lab×field, poids du score, INP, soft-nav, throttling, PSI API v5), voir [references/psi-lighthouse-internals.md](references/psi-lighthouse-internals.md).

### 3. Classer par catégorie

Toute analyse s'organise en **12 catégories** (voir [checks/](checks/)) :

1. Résumé exécutif
2. Métriques PageSpeed (LCP, CLS, INP/TBT, FCP, SI)
3. Above-the-fold (hero, headline, CTA, CSS critique)
4. Images
5. Polices
6. CSS
7. JavaScript
8. Tiers et tracking
9. WordPress / Elementor
10. Stabilité visuelle (CLS)
11. Réseau et serveur
12. Plan d'action

### 4. Attribuer impact et sévérité

Chaque problème reçoit :
- **Métrique affectée** (LCP, CLS, INP, TBT, FCP, SI, TTFB).
- **Sévérité** (critique / haute / moyenne / basse) — voir [references/severity-rubric.md](references/severity-rubric.md).
- **Priorité** (P0 / P1 / P2) — basée sur impact × coût de correction.
- **Risque de la correction** (casse le layout ? casse le tracking ? nécessite une régression visuelle ?).

### 5. Recommander une correction sûre

Pour chaque problème :
- Ce qu'il faut changer exactement (fichier / sélecteur / attribut).
- Pourquoi cela affecte la métrique X.
- Ce qu'il ne faut PAS toucher en même temps.
- Comment tester ensuite (Lighthouse run, WebPageTest, DevTools Performance, régression visuelle).

Liste des corrections sûres vs. risquées : [references/safe-fixes.md](references/safe-fixes.md).

### 6. Émettre le rapport

Template obligatoire : [references/report-format.md](references/report-format.md).

### 7. Décision pré-déploiement

- Tout problème **critique** non résolu → **bloque le déploiement**.
- Plusieurs problèmes **hauts** sur hero/LCP → **bloque** jusqu'à correction du top 3.
- Uniquement **moyens/bas** → **approuvé avec réserves**.
- Tout vert + Core Web Vitals dans le seuil → **approuvé**.

## Métriques et seuils (référence rapide)

Valeurs 2026 du Core Web Vitals (alignées avec Google CrUX). Toujours valider sur [web.dev/vitals](https://web.dev/articles/vitals) en cas de doute.

| Métrique | Bon | À améliorer | Mauvais |
|----------|-----|-------------|---------|
| **LCP** (Largest Contentful Paint) | ≤ 2,5 s | 2,5 s – 4,0 s | > 4,0 s |
| **CLS** (Cumulative Layout Shift) | ≤ 0,1 | 0,1 – 0,25 | > 0,25 |
| **INP** (Interaction to Next Paint) | ≤ 200 ms | 200 ms – 500 ms | > 500 ms |
| **TBT** (Total Blocking Time, lab) | ≤ 200 ms | 200 ms – 600 ms | > 600 ms |
| **FCP** (First Contentful Paint) | ≤ 1,8 s | 1,8 s – 3,0 s | > 3,0 s |
| **Speed Index** | ≤ 3,4 s | 3,4 s – 5,8 s | > 5,8 s |
| **TTFB** | ≤ 800 ms | 800 ms – 1800 ms | > 1800 ms |

Détails dans [references/metrics-glossary.md](references/metrics-glossary.md).

## Lab Data vs. Field Data — TOUJOURS distinguer

- **Lab data (Lighthouse)** — simulation à throttling fixe (Moto G4 / 4G lent). Utile pour le diagnostic.
- **Field data (CrUX)** — données réelles d'utilisateurs Chrome sur les 28 derniers jours. Utile pour la priorisation réelle.
- **Divergence courante :** Lighthouse affiche un LCP de 4 s mais CrUX montre 2,1 s — l'utilisateur réel va bien, le score Lighthouse non. **Décide sur le terrain, valide en lab.**
- Mobile et desktop **ont des rankings séparés chez Google**. Toujours évaluer les deux.

## Red Flags — STOP immédiatement

Si tu te surprends à penser ou à recommander :

- "Différer tous les scripts avec `defer`" — sans avoir mappé les dépendances tracking/inline.
- "Lazy-load sur toutes les images" — y compris l'image du LCP.
- "Retirer le Pixel/GTM parce que c'est trop lourd" — sans alignement avec le marketing.
- "Remplacer Elementor par du code natif" — sans comprendre le cycle d'édition.
- "Compresser le hero à 30 Ko" — en perdant la qualité visuelle du headline.
- "Retirer le CSS non utilisé automatiquement" — peut casser des états (hover, modal, error).
- "Le score 100 sur Lighthouse est l'objectif" — l'objectif est Core Web Vitals "good" en field data.
- "Migrer vers Next.js / framework X règle tout" — sans comprendre le vrai problème.

→ STOP. Reviens à l'étape 4 (impact × risque) avant de recommander.

## Excuse | Réalité

| Excuse | Réalité |
|--------|---------|
| "Lighthouse ne tourne qu'en mobile lent, ce n'est pas réel" | C'est le ranking que Google utilise. Le field data le corrobore. |
| "Un score de 80 c'est déjà bien" | Les Core Web Vitals sont binaires (pass/fail). 80 peut échouer sur le LCP. |
| "J'ai tout optimisé et le score a baissé" | Tu as probablement cassé le tracking inline ou le LCP. Re-audite. |
| "Le client veut 100" | 100 est fragile. L'objectif est "all green CWV" stable. Éduque le client. |
| "Je vais utiliser le plugin X qui fait tout" | Les plugins font 60 % et cassent 30 %. Audite le résultat, pas le plugin. |
| "Le mobile n'a pas d'importance, l'audience est desktop" | Le mobile-first index de Google classe sur le mobile. |
| "J'ai retiré les polices custom et c'est devenu rapide" | La conversion a chuté avec. Optimise la police, ne la retire pas. |

## Checks par domaine

Utilise le fichier de check approprié quand la session touche ces fichiers/zones :

- [checks/hero-and-lcp.md](checks/hero-and-lcp.md) — above-the-fold, image LCP, headline.
- [checks/images.md](checks/images.md) — format, dimensions, lazy, preload, responsive.
- [checks/fonts.md](checks/fonts.md) — Google Fonts, polices locales, `font-display`, preload, fallback.
- [checks/css.md](checks/css.md) — render-blocking, CSS critique, unused CSS, Tailwind, CSS Elementor.
- [checks/javascript.md](checks/javascript.md) — defer/async, bundle, hydration, libs inutiles.
- [checks/third-party-tracking.md](checks/third-party-tracking.md) — Pixel, GTM, GA4, Hotmart, AC, Make, chats.
- [checks/wordpress-elementor.md](checks/wordpress-elementor.md) — plugins, assets globaux, cache, LiteSpeed, Cloudflare.
- [checks/layout-stability.md](checks/layout-stability.md) — CLS, réservation d'espace, fonts swap, embeds.
- [checks/network-server.md](checks/network-server.md) — cache, CDN, Brotli, TTFB, preconnect.

## Règles inviolables

1. **Ne jamais recommander de lazy-load sur l'image du LCP** — toujours `fetchpriority="high"` + sans `loading="lazy"`.
2. **Ne jamais retirer ou déplacer un script de tracking sans confirmation de l'utilisateur** — cela peut casser l'attribution.
3. **Toujours séparer lab et field data** — ne pas traiter le score Lighthouse comme une vérité absolue.
4. **Toujours rapporter mobile et desktop séparément** — ce sont des rankings différents.
5. **Ne jamais recommander `display: none` pour cacher un problème de CLS** — corriger en réservant de l'espace.
6. **Ne jamais dropper les UTMs/query string** dans les optimisations de redirect ou canonical.
7. **Toute recommandation doit pointer fichier/ligne quand applicable** — pas de "optimise le CSS" générique.
8. **Toute analyse est mobile-first par défaut** — le desktop est un complément.
9. **Le rapport est toujours généré** — même si tout est vert.
10. **Un score isolé n'est pas un critère d'approbation** — les Core Web Vitals en field le sont.

## Quand NE PAS l'utiliser

- Audit SEO contenu/keywords → utiliser `seo-audit`.
- Audit d'accessibilité approfondi (WCAG AA/AAA) → skill a11y dédiée.
- Audit de sécurité des headers → `security-auditor` / `seguranca`.
- Bug de rendu visuel (sans lien avec la performance) → debug normal.
- Refactor backend / API → hors périmètre.

## Skills liées

- **congruence** — vérifie si les claims de la landing correspondent à ce que fait le code. À utiliser après un passage performance-audit qui a modifié copy/CTA.
- **seo-audit** — audit SEO technique et on-page. Complémentaire.
- **ux-ui-perfectionist** — revue UI/UX. À utiliser conjointement si l'optimisation touche le layout.
- **ai-seo** — optimisation pour LLM search/AI Overviews. Complémentaire.

## Pourquoi cette skill existe

Optimiser la performance est devenu du folklore : l'équipe change de framework, achète un plugin, active le "mode turbo" du CDN, et le score grimpe de 5 points pendant que la conversion chute de 20 %. Cette skill existe pour **séparer ce que Lighthouse mesure de ce que l'utilisateur ressent**, et garantir que chaque correction :

- Repose sur des preuves concrètes dans le rapport/code.
- A une priorité basée sur impact × coût.
- Possède un test de régression pour layout, tracking et fonctionnement.
- N'est pas du cargo cult ("j'ai entendu dire que defer améliore").

L'objectif final n'est pas le score. C'est **la page qui charge vite pour l'utilisateur réel, qui maintient la conversion et qui passe en plus les Core Web Vitals**.

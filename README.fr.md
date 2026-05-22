# performance-audit

> Audit de performance de pages sous l'angle de **Google PageSpeed Insights / Lighthouse**, sans sacrifier le layout, le tracking, les UTMs, le SEO ni la conversion.

Skill créée pour Claude Code. Structurellement inspirée de la skill [congruence](https://github.com/xBelowZero/congruence-skill) — même pattern d'Iron Law, preuve obligatoire, sévérité explicite et décision pré-déploiement.

---

## Ce que ça fait

Un audit de performance qui sépare **ce que Lighthouse mesure** de **ce que l'utilisateur ressent**, et qui priorise les corrections avec :

- Impact mesurable sur la bonne métrique (LCP, CLS, INP, TBT, FCP).
- Coût d'implémentation réaliste.
- Risque explicite (casse le layout ? casse le tracking ? casse la conversion ?).
- Plan de test post-correction.

**Ce n'est pas** une optimisation aveugle. Toute recommandation préserve :

1. Le layout visuel.
2. Le tracking (Pixel, GTM, GA4, Hotmart, AC, Make, conversions).
3. Les UTMs et paramètres d'URL.
4. Le fonctionnement des formulaires.
5. Le SEO (meta, structured data, canonical, hreflang).
6. L'accessibilité de base.

## Modes de fonctionnement

- **`audit`** — audit d'une page existante (URL, code, rapport Lighthouse).
- **`build`** — revue de code avant déploiement, garantissant la performance dès le départ.

## Quand l'utiliser

À utiliser **dès que** :
- Tu dois auditer une page (landing, WP/Elementor, HTML/Tailwind, Next.js, capture).
- Tu développes une nouvelle page et tu veux garantir un bon score dès le départ.
- Avant le déploiement de toute page publique.
- Tu enquêtes sur une chute des Core Web Vitals.
- Tu enquêtes sur une chute de conversion suspecte liée à la vitesse.

Invoquer avec `/performance-audit` ou "audite la performance de cette page".

## Installation

```bash
# 1. Cloner dans ~/.claude/skills/
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/xBelowZero/claude-skill-performance-audit.git performance-audit

# 2. (Optionnel mais recommandé) configurer l'API key du PageSpeed Insights
# Sans clé : ~1 requête/jour par IP. Avec clé : 400 QPS / 25k requêtes/jour.
#
#   - https://console.cloud.google.com/apis/credentials → créer une API key
#   - Activer "PageSpeed Insights API" et "Chrome UX Report API"
#   - Ajouter dans ~/.zshrc (ou ~/.bashrc) :
#
#       export PSI_API_KEY="AIza..."
#       export CRUX_API_KEY="$PSI_API_KEY"
#
#   - source ~/.zshrc
```

Valider l'installation :

```bash
ls ~/.claude/skills/performance-audit/SKILL.md  # doit exister
echo $PSI_API_KEY                                # doit afficher la clé (optionnel)
```

Dans n'importe quelle session Claude Code :

```
/performance-audit https://exemple.com --mode=audit --target=mobile
```

## Quand NE PAS l'utiliser

- Audit SEO contenu → `seo-audit`.
- Audit d'accessibilité WCAG → skill dédiée.
- Audit de sécurité → `security-auditor`.
- Bug visuel sans lien avec la performance.

## Structure

```
performance-audit/
├── SKILL.md                          # Instructions principales, Iron Law, workflow
├── README.md                         # Ce fichier
├── references/                       # Guides approfondis
│   ├── metrics-glossary.md           #   LCP/CLS/INP/TBT/FCP/SI/TTFB
│   ├── data-collection.md            #   Comment collecter lab + field data
│   ├── severity-rubric.md            #   Rubrique critique/haute/moyenne/basse
│   ├── safe-fixes.md                 #   Corrections par niveau de risque
│   └── report-format.md              #   Template du rapport
└── checks/                           # Audit par domaine
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

## Fonctionnement

1. Identifie périmètre et mode (audit vs build).
2. Collecte des preuves : Lighthouse + PageSpeed + CrUX + code.
3. Classe en 12 catégories (hero, images, fonts, CSS, JS, third-party, WP, CLS, réseau).
4. Attribue métrique affectée + sévérité + priorité + risque.
5. Recommande des corrections **sûres**, en indiquant toujours fichier/ligne et ce qu'il NE faut PAS toucher en même temps.
6. Émet un rapport standardisé.
7. Décide pré-déploiement : approuvé / avec réserves / bloqué.

## Règles inviolables

1. **L'image du LCP ne porte jamais `loading="lazy"`** — toujours `fetchpriority="high"`.
2. **Le tracking ne se touche pas sans confirmation explicite de l'utilisateur**.
3. **Lab et field data toujours séparés** — le score Lighthouse n'est pas une vérité absolue.
4. **Mobile et desktop toujours séparés** — ce sont des rankings Google différents.
5. **Les UTMs ne sont jamais droppées** dans les redirects/canonical.
6. **Toute recommandation cite fichier/ligne ou audit ID**.
7. **Mobile-first par défaut**.
8. **Rapport toujours généré**, même si tout est vert.
9. **Un score isolé n'approuve pas un déploiement** — les Core Web Vitals "good" en field sont le critère.
10. **Toute correction risquée a un plan de rollback**.

## Métriques et seuils (2026)

| Métrique | Bon | À améliorer | Mauvais |
|----------|-----|-------------|---------|
| LCP | ≤ 2,5 s | 2,5–4,0 s | > 4,0 s |
| CLS | ≤ 0,1 | 0,1–0,25 | > 0,25 |
| INP | ≤ 200 ms | 200–500 ms | > 500 ms |
| TBT | ≤ 200 ms | 200–600 ms | > 600 ms |
| FCP | ≤ 1,8 s | 1,8–3,0 s | > 3,0 s |
| TTFB | ≤ 800 ms | 800–1800 ms | > 1800 ms |

Critère Google : **p75 ≥ "good" pour 75 % des utilisateurs sur CrUX**.

## Inspiration et références

Structure inspirée de :
- [xBelowZero/congruence-skill](https://github.com/xBelowZero/congruence-skill) — pattern de preuve obligatoire + Iron Law.
- [addyosmani/web-quality-skills](https://github.com/addyosmani/web-quality-skills) — Agent Skills officielles de l'équipe Chrome pour Lighthouse + CWV.
- [web.dev/articles/vitals](https://web.dev/articles/vitals) — source canonique des seuils.
- [GoogleChrome/lighthouse-ci](https://github.com/GoogleChrome/lighthouse-ci) — budgets et régressions en PR.

## Skills liées

- **congruence** — vérifie si les claims de la page correspondent au code. À utiliser ensuite si l'audit a touché copy/CTA.
- **seo-audit** — SEO technique et on-page. Complémentaire.
- **ux-ui-perfectionist** — revue UI/UX. À utiliser conjointement si l'optimisation touche le layout.

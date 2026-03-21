# Ma config Claude Code

Config personnelle pour [Claude Code](https://claude.ai/code) que j'ai construite et affinée au fil du temps. Tu peux la récupérer, l'adapter et en faire la tienne.

Elle inclut des **skills personnalisés** (commandes slash intelligentes), des **hooks automatiques** (quality gate TypeScript, contexte git, sons de notification), des **règles de code** strictes et un **sandbox** de sécurité configuré.

---

## Installation

```bash
# 1. Clone dans ~/.claude (sauvegarde ta config existante si besoin)
git clone https://github.com/thomasrousselin/claude-config ~/.claude

# 2. Lance le setup interactif
cd ~/.claude && bash setup.sh
```

Le script va te demander ton nom et email Git, puis installer les plugins automatiquement.

### Prérequis

- [Claude Code](https://claude.ai/code) installé (`npm install -g @anthropic-ai/claude-code`)
- Node.js 20+
- Git

---

## Structure

```
~/.claude/
├── setup.sh                  # Script d'installation
├── settings.json             # Config principale (permissions, hooks, sandbox)
├── CLAUDE.md                 # Instructions globales pour Claude
├── rules/                    # Règles de code (TypeScript, langage...)
├── skills/                   # Skills personnalisés (slash commands)
├── plugins/
│   └── plugins.list          # Liste des plugins à installer
├── bin/                      # Hooks et scripts
└── scripts/                  # Statusline et utilitaires
```

---

## Skills disponibles

Les skills s'activent via des commandes slash (`/nom-du-skill`) ou se déclenchent automatiquement selon le contexte.

### Git

| Skill | Commande | Description |
|-------|----------|-------------|
| `git-commit` | `/git-commit` | Commit rapide avec message conventionnel minimaliste |
| `git-create-pr` | `/git-create-pr` | Crée une PR avec titre et description auto-générés |
| `git-merge` | `/git-merge` | Fusionne les branches avec résolution de conflits contextuelle |
| `git-fix-pr-comments` | `/git-fix-pr-comments` | Récupère et implémente tous les changements demandés dans les commentaires de PR |

### Workflows d'implémentation

| Skill | Commande | Description |
|-------|----------|-------------|
| `forge` | `/forge` | Workflow token-efficient en 5 phases (Research→Plan→Execute→Test→Document) |
| `workflow-apex` | `/workflow-apex` | Méthodologie APEX (Analyze→Plan→Execute→eXamine) avec agents parallèles |
| `utils-oneshot` | `/utils-oneshot` | Implémentation ultra-rapide Explore→Code→Test pour tâches focalisées |
| `workflow-debug` | `/workflow-debug` | Débogage systématique : analyse, découverte de solutions, vérification |
| `workflow-ci-fixer` | `/workflow-ci-fixer` | Correcteur CI/CD automatisé — observe, corrige, commite jusqu'au vert |

### Architecture & Qualité

| Skill | Commande | Description |
|-------|----------|-------------|
| `hexagonal-architecture` | `/hexagonal-architecture` | Architecture hexagonale (Ports & Adapters) et principes DDD |
| `workflow-architect` | `/workflow-architect` | Analyse et restructure l'architecture codebase (dépendances, couplage) |
| `clean-code` | `/clean-code` | Analyse et corrige le code selon les principes Clean Code de Robert C. Martin (naming, SOLID, DRY) |
| `code-review-mastery` | `/code-review-mastery` | Cycle complet de révision de code (sécurité OWASP, clean code, feedback) |
| `typescript-strict` | `/typescript-strict` | Patterns de typage strict TypeScript : Zod, type guards, zéro `any` |
| `lsp-navigation` | `/lsp-navigation` | Navigation sémantique via LSP (définitions, références, renommage sûr) |
| `utils-fix-errors` | `/utils-fix-errors` | Corrige les erreurs ESLint et TypeScript avec agents parallèles |
| `utils-refactor` | `/utils-refactor` | Refactorise le code dans toute la codebase avec agents Snipper parallèles |

### Recherche & Documentation

| Skill | Commande | Description |
|-------|----------|-------------|
| `docs-seeker` | `/docs-seeker` | Recherche la documentation technique avec le standard llms.txt |
| `workflow-debate` | `/workflow-debate` | Débat adversarial entre conseillers IA pour évaluer plans et décisions |
| `prompt` | `/prompt` | Charge et exécute un prompt depuis le répertoire `.claude/prompts/` du projet |

### UI / Design

| Skill | Commande | Description |
|-------|----------|-------------|
| `aesthetic` | `/aesthetic` | Améliore et itère des designs avec scoring et génération d'alternatives via Gemini |

### Marketing & SEO

| Skill | Commande | Description |
|-------|----------|-------------|
| `marketing-router` | `/marketing-router` | Routeur d'entrée pour toutes les tâches marketing (landing pages, copy, CRO) |
| `seo-geo` | `/seo-geo` | Optimisation SEO classique et GEO pour moteurs IA (ChatGPT, Perplexity, Gemini) |

### Base de données

| Skill | Commande | Description |
|-------|----------|-------------|
| `databases` | `/databases` | Patterns PostgreSQL et MongoDB — requêtes, optimisation, schémas, migrations |

### AI & Prompts

| Skill | Commande | Description |
|-------|----------|-------------|
| `create-prompt` | `/create-prompt` | Ingénierie de prompts experte pour Claude, GPT et LLMs |
| `create-meta-prompts` | `/create-meta-prompts` | Crée des prompts optimisés pour pipelines Claude-à-Claude multi-étapes |
| `ai-multimodal` | `/ai-multimodal` | Traite audio, images, vidéos et documents via l'API Google Gemini |

### Autonome (Ralph)

| Skill | Commande | Description |
|-------|----------|-------------|
| `ralph-loop` | `/ralph-loop` | Configure la boucle IA autonome Ralph pour shipper des features sans surveillance |
| `ralph-tasks` | `/ralph-tasks` | Gère une queue de tâches JSON pour la boucle autonome Ralph |

### Utilitaires Meta

| Skill | Commande | Description |
|-------|----------|-------------|
| `skill-workflow-creator` | `/skill-workflow-creator` | Crée des skills multi-étapes avec chargement progressif et gestion d'état |
| `domain-name-brainstormer` | `/domain-name-brainstormer` | Génère des idées de noms de domaine et vérifie la disponibilité multi-TLDs |

---

## Hooks automatiques

| Événement | Comportement |
|-----------|-------------|
| `SessionStart` | Injecte le contexte git courant dans chaque session |
| `PreToolUse` (Bash) | Valide les commandes dangereuses avant exécution |
| `PostToolUse` (Edit/Write) | Quality gate TypeScript automatique après chaque modification |
| `Stop` | Handler de complétion (notification de fin de tâche) |
| `Notification` | Son de notification quand Claude a besoin d'une intervention humaine |

---

## Personnalisation

- **Règles de code** : modifie `rules/typescript.md`, `rules/language.md`
- **Instructions globales** : édite `CLAUDE.md`
- **Permissions** : ajuste `settings.json` → `permissions`
- **Sandbox réseau** : ajoute des domaines dans `settings.json` → `sandbox.network.allowedDomains`
- **Nouveaux skills** : crée un dossier dans `skills/` avec un `SKILL.md`

---

Inspiré par la communauté Claude Code. Fais-en ta propre version.

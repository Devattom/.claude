# Forge — Workflow token-efficient

**Date :** 2026-03-11
**Statut :** Approuvé

## Philosophie

Chef d'orchestre (contexte principal) qui ne voit que des résumés compacts et délègue l'exécution à des sous-agents spécialisés sur le bon modèle/effort. Optimisé pour les abonnements Pro où chaque token compte.

Dérivé du skill APEX, Forge conserve la vision workflow structuré tout en réduisant drastiquement la consommation de tokens grâce à une allocation intelligente des modèles.

## Les 5 phases

```
Recherche ──► Plan ──► Exécution ──► Tests ──► Documentation
  (pause)    (pause)     (enchaîné)   (enchaîné)  (enchaîné)
```

### Phase 1 — Recherche (ultra think)

- Agents parallèles (Haiku) : codebase, docs, web
- Agent summarizer (Haiku) : condense les résultats
- Le contexte principal reçoit uniquement les résumés
- **Pause** → l'utilisateur valide le contexte trouvé

### Phase 2 — Planification

- Modèle selon budget (Haiku/Sonnet/Opus)
- Produit un plan fichier par fichier avec complexité estimée par tâche
- **Pause** → l'utilisateur valide le plan

### Phase 3 — Exécution

- Dispatche chaque tâche du plan au bon modèle/effort selon budget
- Sous-agents `snipper` (Sonnet faible) pour les modifications simples
- Sous-agents `file-writer` (Sonnet moyen/élevé) pour les créations
- Opus uniquement en budget `high`

### Phase 4 — Tests

- **Toujours** : linting + typecheck (Haiku runner)
- **Si `-t` ou `-a`** : création + exécution tests unitaires
- **Si `-play`** : tests d'intégration via MCP Playwright
- Boucle retry selon budget : `low` 1x, `mid` 3x, `high` 5x
- Si échec → re-planifie + re-exécute (Sonnet error-analyzer)

### Phase 5 — Documentation

- **Par défaut** : docstrings/JSDoc + mise à jour docs existantes
- **En budget `high`** : + fichier markdown dédié + schémas Mermaid

## Allocation modèle par budget

| Phase | `low` | `mid` (défaut) | `high` |
|-------|-------|-----------------|--------|
| Recherche | Haiku solo | Haiku parallèle (2-3) | Sonnet parallèle (3-5) + ultra think |
| Plan | Haiku effort faible | Sonnet effort moyen | Opus effort moyen |
| Exécution | Sonnet effort faible | Sonnet effort élevé | Opus effort élevé |
| Tests | Haiku run + parse | Haiku run + Sonnet analyse | Haiku run + Opus analyse |
| Doc | Haiku | Sonnet effort faible | Sonnet effort moyen |

## Flags

| Flag | Effet |
|------|-------|
| `-a` / `--auto` | Aucune pause + active tests unitaires |
| `-s` / `--save` | Sauvegarde outputs par phase |
| `-b` / `--branch` | Crée/vérifie branche git |
| `-pr` / `--pull-request` | Crée PR à la fin (active `-b`) |
| `-t` / `--test` | Crée les tests unitaires |
| `-play` / `--playwright` | Tests d'intégration MCP Playwright |
| `-w` / `--team` | Agents parallèles sur la phase recherche |
| `-r {id}` / `--resume` | Reprendre une tâche précédente |
| `--budget low/mid/high` | Allocation modèle (défaut: `mid`) |
| `-i` / `--interactive` | Configuration interactive des flags |

## Sessions et reprise

- **Mode normal** : pause après Recherche, pause après Plan, puis enchaîne Exécution → Tests → Doc
- **Mode `-a`** : aucune pause, tout s'enchaîne
- **`-r {id}`** : reprend à la dernière phase complétée

## Sous-agents spécialisés

| Agent | Modèle | Rôle |
|-------|--------|------|
| `explorer-codebase` | Haiku | Trouve fichiers, retourne chemins + signatures |
| `explorer-docs` | Haiku | Cherche docs, retourne résumé condensé |
| `summarizer` | Haiku | Condense fichiers en résumés structurés |
| `snipper` | Sonnet (faible) | Modifications simples (renommage, imports) |
| `file-writer` | Sonnet (moyen-élevé) | Création de fichiers depuis spec |
| `test-runner` | Haiku | Exécute commandes, parse erreurs |
| `error-analyzer` | Sonnet | Diagnostique erreurs, propose fixes ciblés |
| `doc-writer` | Sonnet | Rédige documentation depuis brief |

## Architecture token-lean

```
┌─────────────────────────────────────────────────┐
│  Contexte principal (Opus en high, Sonnet sinon) │
│  = Chef d'orchestre, ne voit que des résumés     │
│  = Prend les décisions, délègue l'exécution      │
└──────────┬──────────────────────────┬────────────┘
           │ résumés compacts         │ instructions précises
     ┌─────▼─────┐             ┌─────▼─────┐
     │  Haiku    │             │  Sonnet   │
     │  explore  │             │  exécute  │
     │  parse    │             │  écrit    │
     │  résume   │             │  documente│
     └───────────┘             └───────────┘
```

## Différences clés avec APEX

| Aspect | APEX | Forge |
|--------|------|-------|
| Phases | 10 steps | 5 phases |
| Modèle | Opus partout | Allocation par budget |
| Parallélisme | Optionnel sur 4 phases | Uniquement sur recherche |
| Tests | Optionnel séparé | Linting/typecheck toujours, unitaires en option |
| Review adversarial | Phase dédiée (-x) | Fusionné dans les tests |
| Documentation | Absente | Phase dédiée |
| Sessions | Pause à chaque step | 2 pauses (recherche + plan) |
| Économie tokens | Flag -e | Par design (budget) |

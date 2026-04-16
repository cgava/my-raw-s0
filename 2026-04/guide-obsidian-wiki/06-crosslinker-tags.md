---
title: "Phase 6 — Cross-Linker & Tag Taxonomy"
tags: [guide, obsidian-wiki, cross-linker, tag-taxonomy, wikilinks, normalisation]
created: 2026-04-15
phase: 6
---

# Phase 6 — Cross-Linker & Tag Taxonomy

Ce guide couvre deux skills complementaires : **cross-linker** (detection et insertion automatique de wikilinks manquants) et **tag-taxonomy** (audit et normalisation des tags avec un vocabulaire controle). Ensemble, ils transforment un ensemble de pages isolees en un graphe de connaissances navigable et coherent.

Prerequis : avoir lu [[_raw/s0/2026-04/guide-obsidian-wiki/05-status-lint|Phase 5 — Status & Lint]] et disposer d'un vault avec au moins ~30 pages reliees.

## 1. Cross-linker : principes

Le cross-linker est un skill *write-heavy* : il modifie directement les pages du vault pour inserer des `[[wikilinks]]` la ou des connexions manquent. Contrairement a wiki-lint qui signale les problemes, le cross-linker les corrige.

### Protocole en 6 etapes

1. **Page Registry** : construire un index de toutes les pages (nom de fichier, titre, aliases, tags, resume)
2. **Scan des mentions non liees** : pour chaque page, chercher les noms/alias de pages qui apparaissent dans le texte sans etre wrappees dans `[[...]]`
3. **Scoring** : chaque candidat recoit un score base sur la force du signal (exact name match = +4, shared tags = +2, cross-category = +2, etc.)
4. **Classification** : EXTRACTED (score >= 6, quasi-certain), INFERRED (3-5, raisonnable), AMBIGUOUS (1-2, ignore)
5. **Application** : lien inline (prefere) ou section Related (fallback)
6. **Rapport** : resume des liens ajoutes, orphelins restants

### Regles de matching

- Matching case-insensitive pour les noms de pages
- Pas d'auto-reference (une page ne se lie pas a elle-meme)
- Pas de lien dans les code blocks ou le frontmatter
- Pas de double lien (si `[[foo]]` existe deja, on ne l'ajoute pas une seconde fois)
- Premier mention uniquement (lien inline sur la premiere occurrence naturelle)
- Format `[[path|display text]]` quand le chemin differe du texte affiche

## 2. Resultats du cross-linker

### Rapport

| Page | Liens ajoutes | Confiance | Type |
|---|---|---|---|
| `concepts/documentation-metrics.md` | 1 | EXTRACTED | inline (Obsidian) |
| `concepts/iec-15288.md` | 1 | EXTRACTED | inline (ADR) |
| `concepts/knowledge-graph.md` | 1 | INFERRED | Related (KM Tool Selection) |
| `concepts/tool-use.md` | 1 | INFERRED | Related (LLM Knowledge Systems) |
| `entities/langchain.md` | 2 | INFERRED | Related (LLM Knowledge Systems, Agents vs Chatbots) |
| `projects/kiss-claw/kiss-claw.md` | 1 | EXTRACTED | inline (Claude Code) |
| `projects/oh-my-claudecode/oh-my-claudecode.md` | 1 | EXTRACTED | inline (Claude Code) |
| `projects/kiss-claw/concepts/checkpoint-enrichment.md` | 1 | EXTRACTED | inline (Claude Code) |
| `projects/se-vault/se-vault.md` | 1 | EXTRACTED | inline (ADR) |
| `skills/git-submodule-strategies.md` | 1 | INFERRED | Related (IS-Vault) |
| `skills/tdd-with-ai-agents.md` | 1 | INFERRED | Related (Claude Code) |
| `synthesis/agents-vs-chatbots.md` | 1 | EXTRACTED | inline (Claude Code) |

**Total : 14 liens ajoutes sur 12 pages** (7 inline EXTRACTED, 4 Related INFERRED, 3 semantic INFERRED).

### Orphelins restants : 1

- `concepts/savoir-implicite.md` -- page vide, pas de frontmatter exploitable

### Pages ignorees

- `index.md`, `log.md` -- fichiers speciaux
- `_meta/taxonomy.md` -- meta-fichier
- `_raw/*` -- staging area
- `guide/*` -- articles guide (hors scope)

## 3. Exemples de modifications

### Lien inline EXTRACTED : Claude Code dans agents-vs-chatbots

**Avant :**
```markdown
- Multi-agent systems (AutoGen, CrewAI, Claude Code) use orchestrator patterns...
```

**Apres :**
```markdown
- Multi-agent systems (AutoGen, CrewAI, [[entities/claude-code|Claude Code]]) use orchestrator patterns...
```

Score = 4 (exact name match) + 2 (cross-category concept->entity) = 6. Le terme "Claude Code" apparait en texte libre, la page entities/claude-code existe -- c'est un lien evident.

### Lien inline EXTRACTED : ADR dans iec-15288

**Avant :**
```markdown
  - 303-DecisionManagement (hosts ADR-001 on git strategy)
```

**Apres :**
```markdown
  - 303-DecisionManagement (hosts [[skills/architecture-decision-records|ADR]]-001 on git strategy)
```

Le terme "ADR" renvoie naturellement a la page skills/architecture-decision-records. Le format `[[...|ADR]]-001` preserve la lisibilite.

### Lien Related INFERRED : LangChain vers syntheses

LangChain partage les tags `agents` et `llm` avec `synthesis/llm-knowledge-systems` et `synthesis/agents-vs-chatbots` sans les lier. Score = 2 (shared tags) + 2 (cross-category) = 4. Ajout d'une section Related :

```markdown
## Related

- [[synthesis/llm-knowledge-systems|LLM-Powered Knowledge Systems]]
- [[synthesis/agents-vs-chatbots|Agents vs Chatbots]]
```

## 4. Tag Taxonomy : principes

Le skill tag-taxonomy impose un vocabulaire controle de tags via un fichier canonique `_meta/taxonomy.md`. Il fonctionne en deux modes :

1. **Audit** : scanner toutes les pages, construire une table de frequence, detecter les tags non canoniques, les alias, les pages sur-tagguees ou non tagguees
2. **Normalisation** : remplacer les alias par leur forme canonique, supprimer les tags-categories, respecter la limite de 5 tags par page

### Regles de tagging

1. **Max 5 tags par page** -- preferer le large au precis
2. **Lowercase, hyphenated** -- pas de camelCase
3. **Forme canonique** -- mapper les alias vers le tag officiel
4. **La categorie n'est pas un tag** -- ne pas utiliser `synthesis`, `concepts` comme tag
5. **Preferer les tags existants** -- consulter la taxonomy avant d'en creer de nouveaux

## 5. Resultats du tag-taxonomy

### Audit initial

- **85 tags uniques** trouves sur 36 pages
- **0 pages non tagguees**
- **1 page sur-tagguee** (>5) : `synthesis/llm-knowledge-systems.md` (6 tags)
- **Nombreux synonymes** detectes

### Normalisations appliquees

| Ancien tag | Tag canonique | Pages affectees | Raison |
|---|---|---|---|
| `agents` | `ai-agents` | 6 | Desambiguer du terme generique |
| `ai-patterns` | `ai-agents` | 1 | Couvert par `ai-agents` |
| `tool` | `tooling` | 2 | Desambiguer de `tool-use` |
| `enterprise` | `enterprise-architecture` | 1 | Plus specifique |
| `claude` | `claude-code` | 1 | Plus specifique |
| `submodules` | `git-submodules` | 2 | Desambiguer |
| `graph` | `knowledge-graph` | 1 | Plus specifique |
| `linking` | `wikilinks` | 1 | Plus specifique |
| `synthesis` | (supprime) | 2 | Nom de categorie, pas un tag |

**Total : 13 pages modifiees, 9 regles de normalisation appliquees.**

### Taxonomie creee

Le fichier `_meta/taxonomy.md` a ete peuple avec :
- **22 Domain Tags** (tags de sujet principal)
- **5 Type Tags** (nature de l'entite)
- **~45 Descriptive Tags** (tags specifiques)
- **Migration Guide** (9 correspondances ancien->nouveau)

## 6. Verification dans Obsidian

### Graph View avant/apres

Ouvrir la Graph View (`Ctrl+G`) :
- **Avant cross-linker** : les pages entity (claude-code, langchain) ont peu de liens entrants. Les pages projet (kiss-claw, oh-my-claudecode) ne lient pas vers leur runtime (Claude Code).
- **Apres cross-linker** : les entites sont mieux connectees aux projets qui les utilisent. Les syntheses sont reliees aux entites qu'elles mentionnent.

Effet visible : les noeuds `entities/claude-code` et `skills/architecture-decision-records` passent de peripheriques a bien connectes.

### Panneau Tags

Ouvrir le panneau Tags (icone `#` dans la sidebar) :
- **Avant normalisation** : `agents` (6 pages) et `ai-agents` (1 page) coexistent, `tool` (2) et `tooling` (1) coexistent -- impossible de filtrer proprement.
- **Apres normalisation** : `ai-agents` unifie 10 pages, `tooling` unifie 3 pages, `git-submodules` unifie 4 pages. Le filtrage est coherent.

### Verification ciblee

1. Ouvrir `entities/claude-code.md` et verifier les backlinks : 5 nouvelles pages lient vers cette entite (kiss-claw, oh-my-claudecode, checkpoint-enrichment, tdd-with-ai-agents, agents-vs-chatbots)
2. Ouvrir `entities/langchain.md` et verifier la section Related : deux syntheses ajoutees
3. Filtrer par tag `ai-agents` : devrait retourner ~10 pages (concepts, syntheses, projets)

## 7. Observations et limites

### Points positifs

1. **Le cross-linker est conservateur** : il ne lie que la premiere mention, evite les code blocks, et distingue EXTRACTED d'INFERRED. Pas de lien force.
2. **L'entite Claude Code etait sous-liee** : c'est le runtime de tous les projets mais n'etait pas liee depuis kiss-claw, oh-my-claudecode ou agents-vs-chatbots. Le cross-linker corrige ce biais.
3. **ADR est un bon cas de lien contextuel** : "ADR-001" dans le texte renvoie naturellement vers la page skills/architecture-decision-records.
4. **La normalisation des tags unifie le graphe** : `agents` + `ai-agents` + `ai-patterns` fusionnes en `ai-agents` rendent le filtrage par tag coherent.

### Limites

1. **Matching par nom seulement** : le cross-linker ne detecte que les mentions textuelles exactes (nom de fichier, titre, alias). Les connexions purement semantiques (deux pages qui parlent du meme sujet avec des mots differents) ne sont pas captees.
2. **Pas de detection dans les listes a puces complexes** : une mention comme "les frameworks type LangChain" peut echapper au matching si le pattern est ambigu.
3. **Le scoring est heuristique** : les poids (+4 exact match, +2 shared tags) sont des valeurs raisonnables mais non calibrees empiriquement.
4. **Taxonomie a maintenir** : la taxonomie n'a de valeur que si elle est consultee avant chaque ajout de page. Sans discipline, les tags divergeront a nouveau.
5. **Page orpheline persistante** : `savoir-implicite.md` reste sans contenu ni frontmatter -- le cross-linker ne peut rien en faire.

### Idees pour une prochaine iteration

- **Reverse cross-link** : verifier que les pages ciblees ont des backlinks vers les pages sources (symetrie du graphe)
- **Tag-based suggestions** : utiliser les tags pour suggerer des liens entre pages qui partagent 3+ tags mais ne se lient pas
- **Lint post-crosslink** : relancer wiki-lint apres le cross-linker pour mesurer l'amelioration (orphelins, connectivite)
- **Auto-tag depuis le contenu** : proposer des tags manquants en analysant le contenu de la page vs la taxonomie
- **Taxonomie hierarchique** : organiser les tags en arbre (ex: `ai-agents` > `multi-agent`, `orchestration`)

## 8. Navigation

- Article precedent : [[_raw/s0/2026-04/guide-obsidian-wiki/05-status-lint|Phase 5 — Status & Lint]]

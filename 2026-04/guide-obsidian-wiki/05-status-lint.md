---
title: "Phase 5 — Status & Lint"
tags: [guide, obsidian-wiki, wiki-status, wiki-lint, audit, health]
created: 2026-04-15
phase: 5
---

# Phase 5 — Status & Lint

Ce guide couvre les skills **wiki-status** et **wiki-lint** : l'un produit un rapport sur l'etat du vault (inventaire, delta, recommandation), l'autre detecte les problemes structurels (orphelins, liens casses, frontmatter manquant, cohesion des tags). Ces deux outils sont essentiels pour maintenir la sante du wiki a mesure qu'il grossit.

Prerequis : avoir lu [[_raw/s0/2026-04/guide-obsidian-wiki/04-query|Phase 4 — Query & Recherche]] et disposer d'un vault avec au moins ~30 pages.

## 1. wiki-status : inventaire du vault

### Principe

Le skill `wiki-status` lit le `.manifest.json` (registre de tracking) et compare les sources disponibles sur le disque avec ce qui a deja ete ingere. Il produit un rapport structure couvrant :

- **Overview** : nombre total de pages, categories, sources, derniere activite
- **Delta** : nouvelles sources, sources modifiees, sources supprimees
- **Recommandation** : append (delta faible) vs rebuild (delta important) vs lint first

### Rapport obtenu sur le POC

Voici le rapport wiki-status produit sur notre vault de ~36 pages :

```
Wiki Status — 2026-04-15

Overview
- Total wiki pages : 36 across 5 categories
  - concepts: 14, entities: 4, skills: 7, synthesis: 4, projects: 7
- Total sources ingested : 17 (via manifest)
- Last ingest activity : 2026-04-14T23:00:00Z (Claude history batch)

Sources par type (from log.md) :
- 3 raw documents (llm-patterns, obsidian-km, agent-skills)
- 1 promoted raw (prompt-engineering-notes)
- 16 Claude conversations across 6 projects
- 2 data ingests (transcript + ChatGPT export)

Delta : no new sources detected (OBSIDIAN_SOURCES_DIR empty, no new Claude sessions)
Recommendation : No action — vault is up to date
```

Le manifest ne contient pas de champ `last_updated` global dans notre version, mais les stats comptent 17 sources et 35 pages (le comptage reel sur le disque donne 36 pages — l'ecart vient probablement de `concepts/savoir-implicite.md` qui est dans `.trash/` et/ou un decalage de comptage).

### Observations

- **Le manifest est la source de verite** pour le tracking d'ingest. Sans manifest (vault frais), wiki-status traite tout comme "nouveau" et recommande un full ingest.
- **Le delta est le coeur du skill** : il determine si on ajoute incrementalement (append) ou si on reconstruit (rebuild). Pour un vault de 36 pages, l'append est toujours pertinent.
- **OBSIDIAN_SOURCES_DIR est vide** dans notre config, donc il n'y a pas de source documentaire a scanner en delta. Les sources ont toutes ete deposees dans `_raw/` et ingerees (5 des 6 sont toujours presentes dans `_raw/` — seul `prompt-engineering-notes.md` a ete promu et supprime en raw mode, voir la section Zoom ci-dessous).

## 2. wiki-status insights : analyse du graphe

### Principe

Le mode "insights" de wiki-status analyse la structure du graphe de wikilinks : quelles pages sont centrales, lesquelles font pont entre domaines, lesquelles sont isolees. C'est complementaire de wiki-lint (qui cherche les *problemes*) — insights cherche la *structure interessante*.

Le SKILL.md precise : sauter cette analyse si le vault a moins de 20 pages. Notre vault de 36 pages est juste au-dessus du seuil.

### Resultats

#### Anchor pages (top hubs par liens entrants)

| Page | Incoming | Outgoing | Note |
|---|---|---|---|
| `concepts/skill-based-architecture` | 13 | 4 | connector hub |
| `concepts/personal-knowledge-management` | 12 | 5 | connector hub |
| `entities/obsidian` | 11 | 4 | connector hub |
| `projects/se-vault/se-vault` | 8 | 3 | connector hub |
| `concepts/retrieval-augmented-generation` | 8 | 4 | connector hub |
| `concepts/tool-use` | 7 | 4 | connector hub |
| `concepts/zettelkasten` | 6 | 4 | connector hub |
| `concepts/skill-routing` | 6 | 3 | connector hub |
| `projects/kiss-claw/kiss-claw` | 5 | 4 | connector hub |
| `concepts/multi-agent-orchestration` | 5 | 5 | connector hub |

Tous les top hubs sont des connector hubs (incoming ET outgoing > 0). Pas de sink hub (pages tres referencees mais sans lien sortant) — signe d'un vault bien interconnecte.

#### Orphan pre-existant

- `concepts/savoir-implicite.md` — 0 incoming, 0 outgoing. Orphelin complet, probablement un reste de test ou un doublon de poubelle (`.trash/` contient aussi une version).

#### Pages sans liens entrants

- `concepts/savoir-implicite.md` — orphelin complet
- `projects/kiss-claw/concepts/checkpoint-enrichment.md` — 3 outgoing mais 0 incoming (sous-page de projet non referencee)
- `skills/knowledge-management-tool-selection.md` — 4 outgoing mais 0 incoming

#### Bridges (observation qualitative)

Les pages `synthesis/` sont les principaux ponts du vault :
- `synthesis/documentation-as-code` relie le cluster `projects/` (se-vault, is-vault, backup-strategy) au cluster `concepts/` (wiki-governance, iec-15288, zettelkasten) via 11 liens sortants
- `synthesis/llm-knowledge-systems` fait le pont entre le cluster LLM (RAG, knowledge-graph) et le cluster PKM (obsidian, skill-based-architecture)
- `synthesis/agents-vs-chatbots` connecte le domaine agent (skill-routing, tool-use) au domaine architecture (skill-based-architecture)

Les pages `synthesis/` jouent exactement le role prevu par la taxonomie du wiki.

#### Cross-category links

Le vault contient de nombreux liens cross-categorie (skills <-> concepts, projects <-> concepts, entities <-> concepts). C'est le signe d'un graphe sain ou les categories ne sont pas des silos.

## 3. Test wiki-lint : injection de defauts

Pour tester la detection de problemes, nous avons injecte deux defauts deliberes :

1. **Page orpheline** : `concepts/test-orphan-page.md` — frontmatter valide mais zero wikilink entrant et zero sortant
2. **Wikilink casse** : `[[concepts/page-inexistante]]` ajoute dans `concepts/knowledge-graph.md`

## 4. wiki-lint : resultats (run 2 — 2026-04-15)

Voici le rapport complet produit par un second run de wiki-lint, apres nettoyage des defauts injectes dans le run 1 (la page `test-orphan-page.md` et le lien `[[concepts/page-inexistante]]` dans `knowledge-graph.md` ont ete supprimes). Ce rapport reflete l'etat reel du vault.

```markdown
## Wiki Health Report — 2026-04-15

### Orphaned Pages (1 found)
- `concepts/savoir-implicite.md` — 0 incoming links, 0 outgoing links (orphelin complet, manque aussi le frontmatter)

### Broken Wikilinks (25 found)

**Wiki pages — backslash parasite (2)** :
- `concepts/wiki-governance.md:48` -> [[entities/obsidian\]] — backslash en fin de lien, Obsidian ne resout pas
- `skills/knowledge-management-tool-selection.md:47` -> [[entities/obsidian\]] — idem

**Wiki pages — mot "wikilinks" utilise comme exemple (2)** :
- `entities/obsidian.md:23` -> [[wikilinks]] — le mot est entre double crochets dans le texte explicatif
- `concepts/knowledge-graph.md:22` -> [[wikilinks]] — idem

**_raw/ — mot "wikilinks" comme exemple (1)** :
- `_raw/obsidian-knowledge-management.md:7` -> [[wikilinks]]

**Guide pages — mot "wikilinks"/"wikilink" comme exemple (6)** :
- `guide/00-setup.md:16` -> [[wikilinks]]
- `guide/01-ingest-documents.md:119` -> [[wikilinks]]
- `guide/04-query.md:24` -> [[wikilink]]
- `guide/04-query.md:71` -> [[wikilinks]]
- `guide/04-query.md:149` -> [[wikilinks]]
- `guide/04-query.md:179` -> [[wikilinks]]

**Guide pages — backslash parasite (5)** :
- `guide/03-data-ingest.md:92` -> [[skills/documentation-best-practices\]]
- `guide/03-data-ingest.md:93` -> [[concepts/wiki-governance\]]
- `guide/03-data-ingest.md:94` -> [[concepts/documentation-metrics\]]
- `guide/03-data-ingest.md:95` -> [[skills/knowledge-management-tool-selection\]]
- `guide/03-data-ingest.md:96` -> [[skills/architecture-decision-records\]]

**Guide pages — lien vers page renommee ou inexistante (2)** :
- `guide/00-setup.md:104` -> [[guide/01-ingest]] — devrait etre [[guide/01-ingest-documents]]
- `guide/03-data-ingest.md:40` -> [[savoir-implicite]] — devrait etre [[concepts/savoir-implicite]]

**Guide pages — references au rapport lint precedent (citations, pas de vrais liens) (5)** :
- `guide/05-status-lint.md:110` -> [[concepts/page-inexistante]] — citation du run 1, plus d'actualite
- `guide/05-status-lint.md:124` -> [[concepts/page-inexistante]] — idem
- `guide/05-status-lint.md:134` -> [[wikilinks]] — citation
- `guide/05-status-lint.md:135` -> [[wikilinks]] — citation
- `guide/05-status-lint.md:164` -> [[wikilinks]] — citation

**Guide pages — backslash dans citations du rapport (2)** :
- `guide/05-status-lint.md:136` -> [[entities/obsidian\]] — citation du run 1
- `guide/05-status-lint.md:137` -> [[entities/obsidian\]] — citation du run 1

### Missing Frontmatter (1 found)
- `concepts/savoir-implicite.md` — missing: title, category, tags, sources, created, updated (tous les champs)

### Stale Content (0 found)
- Aucune page dont la source sur disque est plus recente que le champ `updated`

### Index Issues (1 found)
- `concepts/savoir-implicite.md` — existe sur le disque mais pas dans `index.md`

### Missing Summary (0 found — soft)
- Toutes les pages wiki ont un champ `summary:` et aucune ne depasse 200 caracteres

### Provenance Issues (2 found)
- `concepts/skill-based-architecture.md` — hub page (13 incoming links) with INFERRED=22% : depasse le seuil de 20% pour les pages a fort trafic
- `concepts/personal-knowledge-management.md` — hub page (12 incoming links) with INFERRED=22% : idem

Note : aucune page n'a AMBIGUOUS > 15%. Aucune page n'est en "unsourced synthesis" (toutes ont un champ `sources:`). Pas de drift de provenance detecte.

### Fragmented Tag Clusters (0 found)
- `#knowledge-management` (5 pages, cohesion=1.60) — bien interconnecte
- `#agents` (6 pages, cohesion=0.93) — bien interconnecte
- `#llm` (7 pages, cohesion=0.80) — bien interconnecte
- Aucun tag cluster ne passe sous le seuil de cohesion < 0.15
```

### Comparaison run 1 vs run 2

| Categorie | Run 1 | Run 2 | Evolution |
|---|---|---|---|
| Orphaned Pages | 2 | 1 | -1 (page de test supprimee) |
| Broken Wikilinks | 21 | 25 | Le run 1 avait signale 16 broken links `_raw/` qui n'apparaissent plus au run 2 (les fichiers existent — le chiffre du run 1 etait probablement errone). Le run 2 inclut les guides qui generent des faux positifs (citations de wikilinks comme exemples, backslash parasites). |
| Missing Frontmatter | 1 | 1 | = (savoir-implicite) |
| Missing Summary | ~many | 0 | toutes les pages ont ete enrichies |
| Index Issues | 2 | 1 | -1 (page de test supprimee) |
| Provenance Issues | 0 | 2 | +2 (check hub INFERRED>20% pas fait au run 1) |
| Fragmented Clusters | 0 | 0 | = |

## 5. Interpretation et limites

### Forces

- **wiki-status** donne une vue d'ensemble rapide : on sait en quelques secondes combien de pages existent, ce qui a ete ingere, et s'il y a un delta a traiter.
- **wiki-lint** detecte correctement les orphelins et les liens casses. Les deux defauts injectes ont ete trouves immediatement.
- Les deux skills sont **read-only** (sauf insights qui ecrit `_insights.md`, un fichier regenerable). Pas de risque de corruption.

### Limites observees

1. **Liens `_raw/` dans les wikilinks** : certaines pages wiki contiennent des wikilinks vers `_raw/` (ex: `[[_raw/s0/2026-04/llm-patterns]]`) dans leur section Sources. Ces fichiers existent toujours dans `_raw/` (voir la section Zoom ci-dessous pour le detail), donc ce ne sont pas des broken links au sens strict. Mais le linter peut les flaguer selon le mode d'ingest utilise — voir l'analyse des modes de promotion ci-dessous.

2. **`[[wikilinks]]` comme texte** : quand le mot `wikilinks` est mentionne dans le texte entre double crochets comme exemple, wiki-lint le detecte comme un lien casse. Pas vraiment evitable sans analyse semantique.

3. **Pas de fix automatique** : wiki-lint detecte mais ne corrige pas. Le SKILL.md decrit comment fixer (ajouter des liens, creer des pages, corriger le frontmatter) mais c'est a l'utilisateur de demander.

4. **Insights necessite 20+ pages** : le seuil est raisonnable mais le mode ne fait pas d'analyse de cohesion de tags (necessite 5+ pages par tag). Notre vault n'a qu'un tag (`knowledge-management`) qui franchit ce seuil avec 9 pages.

5. **Le manifest et le comptage de pages divergent** : le manifest dit 35 pages, le disque en a 36. Ce genre d'ecart est difficile a traquer sans un check de consistance manifest vs fichiers.

### Idees d'amelioration

- **Clarifier le statut des `_raw/`** : le linter devrait distinguer les fichiers `_raw/` encore presents (pas un broken link) de ceux reellement supprimes en raw mode (broken link attendu). Un flag `raw_promoted` dans le manifest aiderait
- **Auto-fix pour le frontmatter manquant** : generer les champs requis avec des valeurs par defaut
- **Lint schedule** : le `.env` prevoit un `LINT_SCHEDULE=weekly` mais il n'y a pas de mecanisme d'execution automatique — c'est un rappel pour l'utilisateur
- **Provenance audit** : la verification 7 (provenance drift) serait interessante sur les pages avec `^[inferred]` mais demande une analyse plus poussee

### Point d'amelioration : persister le rapport de lint

**Constat** : le rapport wiki-lint n'est pas persiste dans un fichier. Il est produit dans la conversation (ou le terminal de l'agent) puis perdu. Seule une ligne resume est ajoutee a `log.md` avec des compteurs agreges (ex: `LINT issues_found=25 orphans=1 broken_links=25 ...`). Le detail — quel fichier, quelle ligne, quel lien casse — n'est conserve nulle part.

**Probleme** : pour retrouver le detail des issues, il faut re-executer le lint. On ne peut pas suivre l'evolution dans le temps : est-ce qu'on avait 25 broken links la semaine derniere ou 21 ? Lesquels ont ete corriges ? Lesquels sont nouveaux ? Le run 1 de ce guide mentionnait 16 liens `_raw/` qui n'existent plus au run 2 — sans le rapport du run 1 (capture ici dans le guide), cette information serait perdue.

**Idee** : persister le rapport complet dans un fichier dedie, par exemple `_meta/lint-report.md`. Avantages :

1. **Consultable dans Obsidian** : le rapport apparait dans le vault, navigable comme n'importe quelle page
2. **Comparable d'un run a l'autre** : on peut faire un diff entre deux versions du fichier (ou garder un historique date dans le fichier)
3. **Suivi de resolution** : on voit les issues disparaitre au fil des corrections, ce qui donne une mesure concrete de l'amelioration du vault
4. **Pas de re-execution necessaire** : le rapport reste disponible entre les sessions, meme si on change de contexte ou d'agent

Le SKILL.md de wiki-lint ne prevoit pas cette persistance — il ne produit qu'un output format et une ligne de log. Ajouter un `Write _meta/lint-report.md` en fin de skill serait un changement minimal avec un benefice significatif.

### Zoom : les fichiers `_raw/` et les modes d'ingestion

Plusieurs pages wiki contiennent des references (frontmatter `sources:` et wikilinks) vers des fichiers `_raw/`. Pour comprendre leur statut, il faut connaitre les **trois modes d'ingestion** definis dans le SKILL.md de `wiki-ingest`.

#### Les trois modes de wiki-ingest

| Mode | Declencheur | Supprime les fichiers `_raw/` ? |
|---|---|---|
| **Append** (defaut) | Ingest normal, pas de mention explicite de `_raw/` | **NON** — traite les fichiers comme des sources ordinaires |
| **Raw** (explicite) | "promote my raw pages", "process my drafts" | **OUI** — supprime le fichier original apres promotion |
| **Full** | Rebuild complet | **NON** |

Le skill `data-ingest` n'a **aucune logique de suppression** de `_raw/`, quel que soit le mode.

#### Ce qui s'est reellement passe dans le vault

| Fichier `_raw/` | Phase | Skill / Mode | Supprime ? | Encore dans `_raw/` ? |
|---|---|---|---|---|
| `llm-patterns.md` | 1 | wiki-ingest / append | NON | **OUI** |
| `obsidian-knowledge-management.md` | 1 | wiki-ingest / append | NON | **OUI** |
| `agent-skills-architecture.md` | 1 | wiki-ingest / append | NON | **OUI** |
| `prompt-engineering-notes.md` | 1 | wiki-ingest / **raw** | OUI | **NON** (seul fichier reellement promu) |
| `meeting-transcript-km-system.md` | 3 | data-ingest | NON | **OUI** |
| `chatgpt-export-documentation-practices.json` | 3 | data-ingest | NON | **OUI** |

Seul `prompt-engineering-notes.md` a ete promu et supprime (raw mode explicite, Phase 1 step 1.5). Les 5 autres fichiers sont toujours dans `_raw/` — c'est le comportement normal pour les modes append et data-ingest.

#### Erratum : ce que le guide disait a tort

Une version precedente de ce guide affirmait que les fichiers `_raw/` "n'existent plus apres promotion". C'etait faux pour 5 des 6 fichiers. L'erreur venait d'une confusion entre le mode **raw** (qui supprime) et le mode **append** (qui ne supprime pas), amplifiee par le README et le `.env.example` du vendor qui suggerent que toute ingestion de `_raw/` entraine une suppression — ce qui ne correspond pas au SKILL.md.

#### Consequence pour le linter

Les wikilinks vers `_raw/` (ex: `[[_raw/s0/2026-04/llm-patterns]]`) ne sont **pas des broken links** : les fichiers existent. Le linter ne les flagge donc pas comme casses au run 2. Le run 1 avait signale 16 broken links `_raw/` — ce chiffre etait probablement errone (hallucination de l'executor, pas un resultat reel du linter).

#### Questions de design pour une version future

Le fait que les fichiers `_raw/` restent apres ingest en mode append pose question :
1. **Risque de double-traitement** : le SKILL.md avertit que les fichiers non supprimes seront re-traites au prochain run (le hash SHA-256 du manifest evite la duplication, mais c'est du travail inutile)
2. **Encombrement** : `_raw/` accumule des fichiers deja traites, melanges avec d'eventuels nouveaux fichiers a ingerer
3. **Ambiguite** : sans le manifest, on ne sait pas quels fichiers `_raw/` ont deja ete traites

Approches possibles :
- **Toujours supprimer** apres ingest (quel que soit le mode) — plus coherent, mais perte de la source originale
- **Deplacer** les sources traitees vers `_archives/raw/` — conserve la tracabilite sans encombrer `_raw/`
- **Marquer** dans le manifest avec un flag `raw_promoted: true` (deja fait pour `prompt-engineering-notes.md`) et filtrer dans le linter

## 6. Navigation dans Obsidian

Dans Obsidian, les resultats de wiki-lint se lisent naturellement :

- **Orphelins** : la vue "Graph" montre les noeuds isoles — pas de ligne qui les relie au reste
- **Liens casses** : Obsidian affiche les liens vers des pages inexistantes en gris clair et les propose a la creation en un clic
- **Backlinks panel** : pour chaque page, le panneau backlinks montre les liens entrants — vide = orphelin potentiel
- **`_insights.md`** : fichier consultable directement dans le vault avec les tableaux de hubs et bridges

Le plugin "Dataview" pourrait aussi exploiter les frontmatter (`tags`, `sources`, `provenance`) pour creer des vues dynamiques equivalentes a wiki-lint, mais c'est hors scope de ce POC.

---

*Suite : Phase 6 — a definir*
*Precedent : [[_raw/s0/2026-04/guide-obsidian-wiki/04-query|Phase 4 — Query & Recherche]]*

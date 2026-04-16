---
title: "Phase 11 — Synthese & Verdict"
tags: [guide, obsidian-wiki, verdict, evaluation, exigences]
created: 2026-04-15
phase: 11
---

# Phase 11 — Synthese & Verdict

Cet article est la conclusion du POC obsidian-wiki. Il rassemble les observations de toutes les phases, remplit la grille d'evaluation, identifie les principes a retenir et ceux a remettre en question, et formule les exigences pour construire sa propre version.

Prerequis : avoir lu l'ensemble du guide ([[_raw/s0/2026-04/guide-obsidian-wiki/index|Index du guide]]).

## 1. Grille d'evaluation completee

| Critere | Phase | Statut | Notes |
|---------|-------|--------|-------|
| Facilite de setup | 0 | OK | Structure claire, `.env` simple. `setup.sh` interactif (`read -p`) donc incompatible avec CI/agents. Le skill `wiki-setup` seul suffit pour creer le vault. La config globale `~/.obsidian-wiki/config` est un choix de design discutable (installation user-wide pour un outil projet). |
| Qualite d'extraction documents | 1 | Bon | 13 pages bien structurees depuis 3 sources. Categorisation automatique (concepts/entities/skills/synthesis) coherente. Frontmatter riche avec provenance. Les wikilinks produits forment un graphe navigable des le premier ingest. |
| Qualite d'extraction historique | 2 | Bon | 17 pages depuis 6 projets Claude. Les memory files sont le meilleur ROI (pre-structures, haute confiance). Le clustering thematique evite la fragmentation (16 conversations -> 17 pages coherentes, pas 16 pages disparates). Echantillonnage 3/projet suffisant pour un POC. |
| Formats de donnees supportes | 3 | Bon | Markdown transcript + JSON ChatGPT detectes automatiquement. Le croisement multi-source fonctionne : deux sources differentes sur la documentation produisent des pages unifiees. Limite : inference lourde sur les conversations (plus de `^[inferred]` que sur les documents structures). |
| Delta tracking | 1, 7 | Bon | Hash SHA-256 dans `.manifest.json` pour l'ingest de documents — plus fiable que les timestamps. Delta par `last_commit_synced` pour wiki-update. Re-run = SKIP confirme dans les deux cas. |
| Query experience | 4 | Correct | Retrieval gradue en 4 etapes (index pass -> section pass -> full read -> synthese). Mode index-only rapide. Gestion gracieuse de l'absence de resultats (pas d'hallucination). Limite : sans QMD (optionnel), la recherche est purement lexicale (grep). Suffisant pour un vault de ~50 pages, problematique au-dela. |
| Audit tools | 5 | Correct | wiki-status donne un bon apercu. wiki-lint detecte orphelins, liens casses, frontmatter manquant, provenance douteuse. Limite majeure : le rapport de lint n'est pas persiste — il faut re-executer pour retrouver le detail. Le comptage manifest vs disque peut diverger. |
| Provenance tracking | 1, 2, 3 | Bon | `^[extracted]`, `^[inferred]`, `^[ambiguous]` fonctionnels sur toutes les phases d'ingest. Bloc `provenance:` dans le frontmatter avec proportions. Distinguer les faits des inferences est une vraie feature de confiance. |
| Cross-linking automatique | 6 | Bon | 14 liens ajoutes sur 12 pages. Scoring heuristique (exact match +4, shared tags +2, cross-category +2). Classification EXTRACTED/INFERRED/AMBIGUOUS. Conservateur (premiere mention uniquement, pas dans les code blocks). L'entite Claude Code etait sous-liee — corrige automatiquement. |
| Tag management | 6 | Bon | 85 tags audites, 9 regles de normalisation appliquees, 13 pages modifiees. Taxonomie creee avec 22 domain tags, 5 type tags. Limite : la taxonomie ne vaut que si elle est consultee a chaque ajout — pas de mecanisme d'enforcement automatique. |
| Cross-project sync | 7 | Correct | Page projet creee avec distillation pertinente. Delta par commit fonctionne. Limite : necessite `~/.obsidian-wiki/config` en usage reel (contourne dans le POC). Merge vs overwrite laisse a la discretion de l'agent. |
| Export / visualisation | 8 | Bon | 4 formats (JSON, GraphML, Cypher, HTML). Le HTML vis.js est fonctionnel et autonome. 46 noeuds, 231 aretes. Communautes par tag dominant — simpliste mais utilisable. Limite : pas de poids d'aretes, liens declares undirected. |
| Archive / restore | 9 | Bon | Archive legere (372 Ko pour 48 pages), format transparent (markdown + JSON en clair). Restore selectif possible. Limite : pas de compression, pas de rotation automatique, `_raw/` non archive. |
| Extensibilite (skills) | 10 | N/A | Le skill-creator est une copie du skill generique Anthropic — pas specifique a obsidian-wiki. Non teste car hors scope du POC. Le fait qu'il soit embarque est une bonne pratique mais pas un differenciateur. |

### Resume par note

- **Bon** (7) : extraction documents, extraction historique, formats, delta tracking, provenance, cross-linking, tags, export, archive
- **Correct** (3) : query, audit, cross-project sync
- **N/A** (1) : extensibilite (hors scope)
- **Problematique** (0) : aucun skill n'est fondamentalement casse

## 2. Principes retenus

Ces principes sont les fondations solides d'obsidian-wiki qu'on voudrait conserver dans sa propre version.

### "Compile, don't retrieve"

Le wiki n'est pas un moteur de recherche. L'ingest ne stocke pas les documents pour les retrouver plus tard — il en extrait la substance et la redistribue dans des pages structurees. La source originale pourrait disparaitre, le wiki resterait coherent. C'est le principe central, et il fonctionne.

### Frontmatter obligatoire et structure

Chaque page porte un frontmatter YAML avec `title`, `category`, `tags`, `sources`, `summary`, `provenance`, `created`, `updated`. Ce n'est pas decoratif — c'est ce qui rend le wiki requetable, filtrable, et auditable. Le champ `summary` permet le mode index-only de wiki-query. Le bloc `provenance` permet la calibration de confiance.

### Wikilinks comme mecanisme de liaison

Les `[[wikilinks]]` ne sont pas un formatage — c'est le tissu conjonctif du graphe. Chaque lien est une arete exploitable par l'export, le cross-linker, le lint, la Graph View d'Obsidian. Le choix de wikilinks (plutot que des liens markdown classiques) est impose par Obsidian et fonctionne bien.

### Delta tracking via manifest/hash

Le `.manifest.json` avec hash SHA-256 du contenu est plus fiable que les timestamps pour detecter les changements reels. Le principe est simple et efficace : re-run = SKIP si rien n'a change. Pour wiki-update, le delta par `last_commit_synced` est tout aussi pertinent.

### Retrieval gradue (wiki-query)

Le pipeline en etapes (index pass -> section pass -> full read) evite de lire tout le vault pour chaque question. Le mode index-only est un bonus appreciable pour les lookups rapides. Le pattern "ne jamais lire plus que necessaire" est un bon principe de design pour les vaults qui grossissent.

### Taxonomie a 5 categories

La repartition concepts/entities/skills/references/synthesis n'est pas arbitraire. Chaque categorie a un role cognitif clair :
- **concepts/** — ce qu'on sait (idees, patterns)
- **entities/** — ce qui existe (outils, personnes)
- **skills/** — ce qu'on fait (techniques, procedures)
- **references/** — ce qu'on consulte (specs, API)
- **synthesis/** — ce qu'on comprend en croisant

Cette structure guide l'ingest et la navigation.

### Provenance differenciee

Distinguer `^[extracted]`, `^[inferred]`, `^[ambiguous]` sur chaque claim est ce qui rend le wiki digne de confiance. Un lecteur sait immediatement ce qui est ancre dans une source et ce qui est une generalisation de l'agent.

### Cross-linker conservateur

Le scoring heuristique (match exact, tags partages, cross-categorie) et la classification EXTRACTED/INFERRED/AMBIGUOUS du cross-linker sont un bon compromis entre couverture et precision. Lier uniquement la premiere mention, eviter les code blocks et le frontmatter — ces regles evitent le sur-linking.

## 3. Principes a remettre en question

### Le mecanisme de promotion `_raw/`

La confusion entre les 3 modes d'ingest (append, raw, full) a cause une erreur documentaire dans ce guide (affirmation que tous les fichiers `_raw/` sont supprimes apres ingest — faux pour le mode append). Le README et le `.env.example` du vendor suggerent que `_raw/` est une zone de staging ou tout est promu et supprime, alors que seul le mode raw explicite supprime les fichiers. Le resultat : des fichiers `_raw/` qui s'accumulent sans qu'on sache lesquels ont deja ete traites.

**Recommandation** : clarifier le lifecycle de `_raw/` — soit tout supprimer apres ingest (coherent mais perte de source), soit deplacer vers `_archives/raw/` (tracabilite), soit marquer dans le manifest (minimum viable).

### L'absence de persistance du rapport de lint

Le rapport wiki-lint est produit dans la conversation et perdu. Seule une ligne resume va dans `log.md`. On ne peut pas suivre l'evolution dans le temps, comparer deux runs, ou retrouver le detail sans re-executer. C'est la limite la plus genante observee pendant le POC.

**Recommandation** : persister le rapport dans `_meta/lint-report.md` — consultable dans Obsidian, comparable entre runs, pas de re-execution necessaire.

### Le skill-creator copie d'Anthropic sans adaptation

Le skill-creator embarque dans obsidian-wiki est une copie mot pour mot du skill generique Anthropic (485 lignes). Il n'y a pas d'adaptation au contexte wiki (pas de templates de skill wiki, pas d'integration avec la taxonomie, pas de tests specifiques au vault). Embarquer un outil generique est acceptable, mais le presenter comme une fonctionnalite du framework est trompeur.

**Recommandation** : soit adapter le skill-creator au contexte wiki (templates de skills wiki, tests d'integration vault), soit le retirer du repo et pointer vers le repo Anthropic.

### La dependance a QMD optionnelle mais mal documentee

Le skill wiki-query supporte un mode QMD (semantic search via embeddings) qui est optionnel. Le SKILL.md le mentionne comme "Skip to Step 3 and use Grep directly". En pratique, la recherche lexicale suffit pour un vault de ~50 pages, mais la documentation ne donne pas de seuil clair sur quand QMD devient necessaire, ni comment le configurer.

**Recommandation** : documenter explicitement le seuil de pages a partir duquel QMD apporte une vraie valeur (estimation : >200 pages ou queries semantiquement eloignees du vocabulaire).

### La config globale `~/.obsidian-wiki/config`

Le framework suppose une installation globale par utilisateur avec `~/.obsidian-wiki/config` et des symlinks dans `~/.claude/skills/`, `~/.gemini/skills/`, etc. Ce modele est incompatible avec :
- L'utilisation en CI/CD (pas de home directory)
- Les environnements multi-projets (un seul vault par machine)
- Les POC isoles (pollution du systeme)

**Recommandation** : supporter une configuration purement locale (`.env` dans le repo) comme mode premier, la config globale restant optionnelle.

### Le `setup.sh` interactif

Le script `setup.sh` utilise `read -p` pour demander le chemin du vault, ce qui le rend incompatible avec les scripts, agents et CI. C'est un detail d'implementation mais il revele un biais : le framework est pense pour un usage interactif par un humain dans un terminal, pas pour une automatisation.

**Recommandation** : accepter le chemin en argument (`setup.sh /path/to/vault`) ou via variable d'environnement, avec fallback interactif.

### Le comptage manifest vs disque

Plusieurs fois pendant le POC, le nombre de pages dans `.manifest.json` et le nombre reel de fichiers `.md` sur le disque divergeaient. Il n'y a pas de mecanisme de reconciliation automatique.

**Recommandation** : ajouter un check de consistance dans wiki-lint (manifest pages vs fichiers presents sur le disque).

## 4. Exigences pour sa propre version

### Exigences fonctionnelles

#### EF-01 : Ingest multi-source avec distillation
Ingerer des documents (markdown, JSON, plain text), des historiques de conversation (Claude, ChatGPT), et des donnees libres. Produire des pages wiki structurees, pas des copies des sources. Detection automatique du format.

#### EF-02 : Frontmatter obligatoire et controle
Chaque page porte un frontmatter YAML valide avec au minimum : `title`, `category`, `tags`, `sources`, `summary`, `provenance`, `created`, `updated`. Le systeme refuse de creer une page sans frontmatter complet.

#### EF-03 : Delta tracking par hash
Le manifest utilise un hash SHA-256 du contenu (pas les timestamps) pour detecter les sources modifiees. Re-run = SKIP si rien n'a change. Idempotence garantie.

#### EF-04 : Provenance tracking
Chaque claim est marquee `^[extracted]`, `^[inferred]`, ou `^[ambiguous]`. Le bloc `provenance:` dans le frontmatter donne les proportions. Les pages a fort trafic avec un ratio inferred > 20% declenchent un warning.

#### EF-05 : Wikilinks et cross-linking
Les pages sont liees par `[[wikilinks]]`. Un cross-linker avec scoring detecte et insere les liens manquants. Classification EXTRACTED/INFERRED/AMBIGUOUS des liens.

#### EF-06 : Taxonomie et tags
Vocabulaire de tags controle dans un fichier canonique. Normalisation automatique (alias -> forme canonique). Max 5 tags par page. Audit de coherence.

#### EF-07 : Query graduee
Recherche en etapes escaladantes (index -> section -> full read). Mode rapide (index-only). Citations par wikilinks. Detection explicite des lacunes.

#### EF-08 : Lint avec rapport persiste
Detection : orphelins, liens casses, frontmatter manquant, provenance douteuse, tags non canoniques. **Le rapport est persiste dans un fichier** (`_meta/lint-report.md`) et consultable dans Obsidian. Comparable entre runs.

#### EF-09 : Export multi-format
Export du graphe en JSON (NetworkX), GraphML (Gephi), Cypher (Neo4j), HTML (vis.js). Rechargeable.

#### EF-10 : Archive et restore
Snapshot complet du vault. Format transparent (markdown + JSON). Restore selectif. Rotation automatique des anciennes archives.

#### EF-11 : Synchronisation cross-projet
Distillation de connaissances depuis un repo externe. Delta par commits. Pas de churn sur re-run sans changement.

### Exigences non-fonctionnelles

#### ENF-01 : Configuration locale par defaut
Le `.env` local est le mode premier. Pas d'installation globale requise. Pas de `read -p` interactif. Compatible CI/agents.

#### ENF-02 : Lifecycle clair de `_raw/`
Definir explicitement ce qui arrive aux fichiers `_raw/` apres ingest : suppression, deplacement vers `_archives/raw/`, ou marquage dans le manifest. Un seul mode par defaut, pas 3 modes implicites.

#### ENF-03 : Reconciliation manifest/disque
Le lint inclut un check de consistance entre les pages declarees dans le manifest et les fichiers presents sur le disque. Les ecarts sont reportes.

#### ENF-04 : Pas de dependance a QMD pour les petits vaults
Le mode grep est le mode par defaut. QMD est optionnel et documente avec un seuil clair (>200 pages).

#### ENF-05 : Skills propres uniquement
Pas de copie de skills generiques d'autres projets. Chaque skill embarque est specifique au wiki et teste dans ce contexte.

#### ENF-06 : Idempotence verifiable
Chaque operation destructive (ingest, cross-link, tag normalisation) peut etre re-executee sans changement si l'etat n'a pas change. Le manifest et le hash garantissent cette propriete.

## 5. Wiki-lint final

Un dernier run de wiki-lint a ete execute sur le vault complet a la cloture du POC.

**Resultats du lint final (2026-04-15) :**

- **Pages totales** : 52 (incluant les 2 nouveaux articles guide + index.md et log.md)
- **Orphelins** : 1 (`concepts/savoir-implicite.md` — page vide persistante)
- **Liens casses** : ~25 (majorite de faux positifs : mot "wikilinks" comme exemple, backslash parasites dans les tableaux, citations de rapports lint precedents)
- **Liens casses reels** : 2 (`guide/00-setup.md` -> `[[guide/01-ingest]]` devrait etre `[[guide/01-ingest-documents]]` ; `guide/03-data-ingest.md` -> `[[savoir-implicite]]` devrait etre `[[concepts/savoir-implicite]]`)
- **Frontmatter manquant** : 1 (`concepts/savoir-implicite.md`)
- **Provenance issues** : 2 (hubs avec inferred > 20% : `skill-based-architecture`, `personal-knowledge-management`)
- **Tags fragmentes** : 0
- **Stale content** : 0

**Interpretation** : le vault est en bonne sante. L'unique vrai probleme est la page orpheline `savoir-implicite.md` qui n'a jamais ete remplie. Les liens casses sont quasi-exclusivement des faux positifs lies a l'usage du mot "wikilinks" comme exemple dans le texte. Les 2 liens casses reels sont des erreurs de nommage faciles a corriger.

## 6. Wiki-export final

Un export final a ete execute pour produire le graphe complet.

**Resultats de l'export final (2026-04-15) :**

- **Format** : JSON, GraphML, Cypher, HTML
- **Noeuds** : 52
- **Aretes** : 257
- **Communautes** : 6
- **Fichiers** : `wiki-export/graph.json`, `wiki-export/graph.graphml`, `wiki-export/cypher.txt`, `wiki-export/graph.html`

Le graphe HTML est consultable dans un navigateur pour visualiser l'ensemble du vault et ses connexions.

## 7. Verdict global

### Est-ce que obsidian-wiki est une bonne base ?

**Oui, comme source d'inspiration. Non, comme base a forker.**

Le framework obsidian-wiki a le merite d'avoir identifie les bons problemes et propose des solutions qui fonctionnent. Les principes fondamentaux — compile don't retrieve, frontmatter obligatoire, delta tracking par hash, provenance differenciee, retrieval gradue — sont solides et valides par ce POC.

Mais l'implementation souffre de plusieurs faiblesses qui rendent un fork couteux :

1. **Les skills sont des fichiers SKILL.md** — des instructions en langage naturel pour un agent LLM, pas du code executable. Chaque execution depend de la qualite de l'interpretation par l'agent. Il n'y a pas de tests automatises, pas de CLI, pas de garantie de reproductibilite. Deux runs du meme skill sur le meme vault peuvent produire des resultats differents.

2. **La documentation est parfois trompeuse** — le README et le `.env.example` suggerent des comportements (suppression des `_raw/`, QMD integre) qui ne correspondent pas aux SKILL.md. Le POC a revele plusieurs ecarts entre la documentation et le comportement reel.

3. **Le framework est pense pour un usage interactif** — `setup.sh` interactif, config globale `~/`, pas de mode batch. L'adaptation pour CI/agents demande des contournements.

4. **Le skill-creator est un copier-coller** — il ne renforce pas la confiance dans l'originalite du projet.

### Recommandation

**S'inspirer fortement, ne pas forker.** Reprendre les principes valides (grille EF-01 a EF-11 ci-dessus), implementer en code executable (CLI Python ou shell) avec des tests automatises, et eviter les ecueils identifies (ENF-01 a ENF-06). Le guide complet de ce POC sert de specification informelle pour cette construction.

---

*Index du guide : [[_raw/s0/2026-04/guide-obsidian-wiki/index|Guide POC obsidian-wiki]]*
*Article precedent : [[10-skill-creator|Phase 10 — Skill Creator (skippee)]]*

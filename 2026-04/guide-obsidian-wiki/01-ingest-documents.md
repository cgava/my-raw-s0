---
title: "Phase 1 — Ingest de documents"
tags: [guide, obsidian-wiki, ingest, wiki-ingest]
created: 2026-04-14
phase: 1
---

# Phase 1 — Ingest de documents

Ce guide explique comment **wiki-ingest** transforme des documents sources en pages wiki interconnectees. Il couvre le principe de fonctionnement, les sources utilisees lors de cette phase, les pages produites, et les mecanismes cles a observer.

## 1. Principe de l'ingest

wiki-ingest n'est **pas un resumeur**. C'est un distillateur de connaissances.

Quand on lui soumet un document, il ne produit pas un condense du texte original. Il **extrait** les concepts, entites, claims et relations contenus dans la source, puis les redistribue dans des pages wiki autonomes et interconnectees. Un meme document peut donner naissance a 5 pages differentes, chacune portant sur un concept distinct, et chaque page peut etre enrichie par plusieurs sources.

Le resultat n'est pas un miroir de la source — c'est un **graphe de connaissances compile**.

### Les 3 modes d'ingest

| Mode | Comportement | Quand l'utiliser |
|------|-------------|-----------------|
| **append** (defaut) | N'ingere que les sources nouvelles ou modifiees. Verifie le hash SHA-256 dans `.manifest.json` — si le contenu est identique, skip meme si le timestamp a change. | La plupart du temps. Rapide, evite le travail redondant. |
| **full** | Retraite toutes les sources, meme celles deja ingerees. | Apres un `wiki-rebuild`, ou si le manifest est corrompu. |
| **raw** | Traite les notes brutes deposees dans `_raw/`. Apres promotion en page wiki, le fichier original est supprime de `_raw/`. | Pour integrer des notes capturees rapidement, sans structure. |

## 2. Les sources utilisees

Trois documents ont ete crees pour tester l'ingest, choisis pour leur **variete thematique** :

### [[_raw/s0/2026-04/llm-patterns|llm-patterns.md]] — Patterns LLM fondamentaux

Couvre les trois patterns architecturaux dominants dans les systemes LLM en production : [[concepts/retrieval-augmented-generation|RAG]], [[concepts/chain-of-thought|Chain-of-Thought]], et [[concepts/tool-use|Tool Use]]. C'est le document le plus dense — il a produit 5 pages a lui seul.

### [[_raw/s0/2026-04/obsidian-knowledge-management|obsidian-knowledge-management.md]] — Obsidian comme outil de gestion de connaissances

Presente [[entities/obsidian|Obsidian]], son modele de graphe par wikilinks bidirectionnels, la methode [[concepts/zettelkasten|Zettelkasten]], et l'ecosysteme de plugins. Ce document apporte la perspective **gestion de connaissances personnelle** au wiki.

### [[_raw/s0/2026-04/agent-skills-architecture|agent-skills-architecture.md]] — Architecture a base de skills pour agents IA

Decrit le pattern [[concepts/skill-based-architecture|skill-based architecture]] : comment decomposer les capacites d'un agent en skills declaratifs, le [[concepts/skill-routing|routing]], et la composition. Ce document fait le pont entre les LLMs et les systemes multi-agents.

**Pourquoi ces trois sujets ?** Parce qu'ils se recoupent. Les patterns LLM alimentent les agents, les agents utilisent des skills, et le tout peut etre capture dans un systeme de connaissances comme Obsidian. L'ingest peut ainsi produire des pages de **synthese** qui croisent les sources.

## 3. Les pages produites

L'ingest a cree **13 pages** reparties en 4 categories. Une quatrieme source (note brute en mode raw) a enrichi une page existante.

### Concepts (8 pages)

Les concepts sont des **idees abstraites, patterns ou modeles mentaux** :

- [[concepts/retrieval-augmented-generation|Retrieval-Augmented Generation]] — Le pipeline RAG en trois etapes (indexation, retrieval, generation) pour ancrer les LLMs dans des connaissances externes
- [[concepts/chain-of-thought|Chain-of-Thought Prompting]] — Raisonnement pas-a-pas pour ameliorer la precision sur les taches complexes
- [[concepts/tool-use|Tool Use and Function Calling]] — Invocation de fonctions externes par les LLMs pour agir sur le monde
- [[concepts/knowledge-graph|Knowledge Graph]] — Liens bidirectionnels transformant des notes en reseau navigable d'idees
- [[concepts/zettelkasten|Zettelkasten Method]] — Notes atomiques liees en structure emergente
- [[concepts/personal-knowledge-management|Personal Knowledge Management]] — Collecte, organisation et connexion de connaissances pour la retrouvabilite
- [[concepts/skill-based-architecture|Skill-Based Architecture]] — Decomposition des capacites d'un agent en skills composables en markdown
- [[concepts/skill-routing|Skill Routing]] — Dispatch des requetes utilisateur vers le skill agent approprie

### Entities (2 pages)

Les entites sont des **choses concretes** — outils, frameworks, organisations :

- [[entities/obsidian|Obsidian]] — Application PKM local-first basee sur le markdown avec liens bidirectionnels
- [[entities/langchain|LangChain]] — Framework open-source pour construire des applications LLM

### Skills (1 page)

Les skills sont des **savoir-faire, techniques, procedures** :

- [[skills/prompt-engineering-techniques|Prompt Engineering Techniques]] — Techniques de prompting (zero-shot, few-shot, CoT, structured output) et patterns emergents (prompt chaining, meta-prompting)

### Synthesis (2 pages)

Les syntheses sont des **analyses transversales** croisant plusieurs concepts et sources :

- [[vault/s0/synthesis/agents-vs-chatbots|Agents vs Chatbots]] — Le tool use comme ligne de demarcation entre generateurs passifs et agents actifs
- [[vault/s0/synthesis/llm-knowledge-systems|LLM-Powered Knowledge Systems]] — Connexion entre RAG, PKM et agents a skills dans des workflows automatises de connaissances

### Le principe de categorisation

La categorisation n'est pas arbitraire. Elle suit une taxonomie pensee pour la navigation :

- **concepts/** — ce qu'on _sait_ (idees, patterns)
- **entities/** — ce qui _existe_ (outils, personnes, projets)
- **skills/** — ce qu'on _fait_ (techniques, procedures)
- **synthesis/** — ce qu'on _comprend en croisant_ (analyses multi-sources)
- **references/** — ce qu'on _consulte_ (specs, API, configs)

## 4. Les mecanismes cles observes

### Frontmatter structure

Chaque page wiki porte un en-tete YAML obligatoire :

```yaml
---
title: Retrieval-Augmented Generation
category: concepts
tags: [llm, rag, vector-search, information-retrieval]
sources: [_raw/llm-patterns.md]
summary: "RAG compensates for LLM knowledge cutoffs by retrieving..."
provenance:
  extracted: 0.80
  inferred: 0.15
  ambiguous: 0.05
created: 2026-04-14T20:23:07Z
updated: 2026-04-14T20:23:07Z
---
```

Ce frontmatter permet les requetes structurees (via Dataview dans Obsidian), le tri par tags, et le suivi de provenance.

### Wikilinks — ce qui fait un graphe de connaissances

Chaque page est liee a **au moins 2-3 autres pages** via des `[[wikilinks]]`. Par exemple, la page [[concepts/retrieval-augmented-generation|RAG]] lie vers [[concepts/tool-use|Tool Use]], [[concepts/knowledge-graph|Knowledge Graph]], et [[concepts/personal-knowledge-management|PKM]].

C'est cette interconnexion qui transforme un dossier de fichiers markdown en **graphe de connaissances navigable**. Sans les wikilinks, on aurait juste un dossier. Avec eux, on a un reseau ou chaque idee mene a des idees connexes.

### Provenance markers — tracabilite des claims

Toutes les affirmations d'une page ne se valent pas. L'ingest marque chaque claim selon sa source :

- **Pas de marqueur** — claim directement extraite de la source (fait verifiable)
- **`^[inferred]`** — generalisation, implication, ou synthese inter-sources. L'ingest a _deduit_ cette affirmation
- **`^[ambiguous]`** — les sources sont vagues ou contradictoires sur ce point

Exemple dans [[concepts/retrieval-augmented-generation|RAG]] :
> "Hybrid retrieval combining vector similarity with BM25 lexical search outperforms pure vector search" `^[inferred]`

Le frontmatter porte aussi un bloc `provenance:` avec les proportions (extracted/inferred/ambiguous summing to ~1.0). La page [[vault/s0/synthesis/llm-knowledge-systems|LLM-Powered Knowledge Systems]], qui est une synthese, a logiquement 75% d'inferred.

### Delta tracking — `.manifest.json` et hash SHA-256

Le fichier `.manifest.json` a la racine du vault enregistre chaque source ingeree avec :
- **`content_hash`** : hash SHA-256 du contenu au moment de l'ingest
- **`ingested_at`** : timestamp de l'ingest
- **`pages_created`** et **`pages_updated`** : les pages produites ou modifiees

Lors d'un re-run en mode append, l'ingest compare le hash actuel au hash enregistre. **Si le contenu est identique, la source est skippee** — meme si le timestamp a change (cas frequent avec `git checkout`, copies, NFS). C'est plus fiable qu'une comparaison par date de modification.

### Raw promotion

Le mode raw illustre un workflow particulier : on depose une note brute dans `_raw/`, et l'ingest la promeut en enrichissant les pages wiki existantes. Le fichier `_raw/prompt-engineering-notes.md` a ete traite ainsi — il a enrichi la page [[skills/prompt-engineering-techniques|Prompt Engineering Techniques]], puis le fichier original a ete supprime de `_raw/`.

Dans le manifest, cette source est marquee `"raw_promoted": true` et a `pages_created: []` / `pages_updated: ["skills/prompt-engineering-techniques.md"]`.

## 5. Comment verifier dans Obsidian

Pour voir le resultat de l'ingest par vous-meme :

1. **Ouvrir le vault** — Dans Obsidian, File > Open Vault, selectionner le dossier `knlg-repo`

2. **Explorer le graph view** — `Ctrl/Cmd + G` pour ouvrir la vue graphe. Vous verrez les 13 pages et leurs connexions. Les pages de synthese (au centre) ont le plus de liens. Les concepts forment des clusters thematiques.

3. **Suivre les wikilinks** — Ouvrir par exemple [[concepts/retrieval-augmented-generation|Retrieval-Augmented Generation]]. Cliquer sur les liens vers [[concepts/tool-use|Tool Use]], [[concepts/knowledge-graph|Knowledge Graph]], etc. Observer comment on navigue d'idee en idee.

4. **Verifier les tags** — Dans la sidebar gauche, ouvrir le panneau Tags. Les tags comme `llm`, `knowledge-management`, `agents` regroupent les pages par theme, orthogonalement a la categorisation par dossier.

5. **Inspecter le manifest** — Ouvrir `.manifest.json` (fichier cache, activer "Show hidden files" dans les settings). Verifier les 4 sources, leurs hash, et les pages produites.

6. **Consulter le journal** — Ouvrir `log.md` pour voir l'historique chronologique des 4 ingests (3 documents + 1 raw promotion).

## 6. Observations et principes retenus

### "Compile, don't retrieve"

Le wiki n'est pas un moteur de recherche. C'est un **savoir pre-compile**. Quand l'ingest traite un document, il ne le stocke pas pour le retrouver plus tard — il en extrait la substance et la redistribue dans des pages structurees. La source originale pourrait disparaitre, le wiki resterait coherent.

### La provenance est une feature de premier ordre

Distinguer les faits extraits des inferences n'est pas un detail de mise en forme — c'est ce qui rend le wiki **digne de confiance**. Un lecteur peut immediatement voir quelles affirmations sont ancrees dans une source et lesquelles sont des generalisations de l'agent.

### Le delta tracking par hash evite les re-traitements

Comparer les timestamps est fragile (git, copies, NFS les modifient). Le hash SHA-256 du contenu est la seule facon fiable de savoir si un document a **reellement** change. C'est un mecanisme simple mais essentiel pour un ingest idempotent.

### La categorisation structure le savoir

Repartir les pages en concepts, entities, skills, references et synthesis n'est pas un choix esthetique. Chaque categorie a un **role cognitif** : les concepts sont des idees, les entities sont des choses, les skills sont des savoir-faire, les syntheses croisent les perspectives. Cette structure guide la navigation et aide l'ingest a decider _ou_ placer une nouvelle page.

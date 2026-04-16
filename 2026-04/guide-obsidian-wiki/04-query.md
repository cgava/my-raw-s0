---
title: "Phase 4 — Query & Recherche"
tags: [guide, obsidian-wiki, wiki-query, recherche, retrieval]
created: 2026-04-15
phase: 4
---

# Phase 4 — Query & Recherche

Ce guide explique comment le skill **wiki-query** permet d'interroger le wiki compile pour retrouver, synthetiser et citer des connaissances. C'est le pendant "lecture" des phases precedentes qui etaient du "ecriture" (ingest). Le skill fonctionne depuis n'importe quel repertoire projet.

Prerequis : avoir lu [[_raw/s0/2026-04/guide-obsidian-wiki/00-setup]] et au moins une phase d'ingest ([[_raw/s0/2026-04/guide-obsidian-wiki/01-ingest-documents|Phase 1]], [[_raw/s0/2026-04/guide-obsidian-wiki/02-history-ingest|Phase 2]], ou [[_raw/s0/2026-04/guide-obsidian-wiki/03-data-ingest|Phase 3]]).

## 1. Principe du wiki-query

Le skill `wiki-query` effectue une **recherche graduee** dans le vault Obsidian, en escaladant le cout uniquement quand c'est necessaire :

| Etape | Nom | Cout | Ce qu'on lit |
|-------|-----|------|-------------|
| 1 | Comprendre la question | nul | Classification du type de query |
| 2 | Index pass | faible | `index.md`, frontmatter (`title`, `tags`, `summary`) |
| 3 | Section pass | moyen | Grep contextuel autour des termes matches |
| 4 | Full read | eleve | Corps complet des 3 meilleures pages + 1 hop de wikilinks |
| 5 | Synthese | -- | Composition de la reponse avec citations `[[wikilink]]` |
| 6 | Log | nul | Ajout d'une entree dans `log.md` |

Le principe cle : **ne jamais lire plus que necessaire**. L'index pass (etape 2) suffit souvent pour les questions factuelles. Le full read (etape 4) est un dernier recours.

### Types de queries supportes

- **Factual lookup** -- "Qu'est-ce que X ?" → trouve la page pertinente
- **Relationship query** -- "Comment X se relie a Y ?" → trouve les deux pages et leurs references croisees
- **Synthesis query** -- "Quel est l'etat de la reflexion sur X ?" → agrege toutes les pages qui touchent X
- **Gap query** -- "Que manque-t-il sur X ?" → identifie les lacunes et les sections "Open Questions"

### Modes de fonctionnement

- **Mode normal** -- Pipeline complet (etapes 1 a 6)
- **Mode index-only** -- Declenche par "quick answer", "just scan", "fast lookup". S'arrete a l'etape 2. Repond uniquement depuis le frontmatter et `index.md`.

## 2. Query normale : "prompt engineering techniques"

### Commande

```
/wiki query: prompt engineering techniques
```

### Deroulement observe

1. **Index pass** -- Scan des titres et tags dans `index.md`. Deux candidats identifies :
   - [[skills/prompt-engineering-techniques|Prompt Engineering Techniques]] (match exact sur le titre)
   - [[concepts/chain-of-thought|Chain-of-Thought Prompting]] (match via tag `prompting`)

2. **Section pass** -- Grep contextuel sur "prompt engineering" dans les deux fichiers candidats. Confirmation que les deux pages sont pertinentes.

3. **Full read** -- Lecture complete des deux pages pour synthese.

### Resultat

> **Based on the wiki:**
>
> Prompt engineering is the practice of designing LLM inputs for reliable, high-quality outputs. The wiki covers core techniques (zero-shot, few-shot, [[concepts/chain-of-thought|Chain-of-Thought]], structured output, system prompts, temperature control) and emerging patterns (constitutional AI, prompt chaining, meta-prompting). CoT is singled out as particularly effective for reasoning tasks. Prompt engineering underpins all LLM patterns : [[concepts/retrieval-augmented-generation|RAG]], [[concepts/tool-use|tool use]], et [[concepts/skill-based-architecture|skill-based architecture]].
>
> **Pages consulted :** [[skills/prompt-engineering-techniques]], [[concepts/chain-of-thought]]
>
> **Gaps :** Pas de couverture de l'adversarial prompting, du prompt injection, ou des defenses.

### Observations

- La reponse **cite les pages sources** avec des `[[wikilinks]]` navigables dans Obsidian
- Le chemin de retrieval est transparent : "found via title match, confirmed by section grep, full read for synthesis"
- Les lacunes sont explicitement signalees -- le wiki sait ce qu'il ne sait pas

## 3. Query index-only : "knowledge graph"

### Commande

```
/wiki query: knowledge graph (mode: quick answer)
```

Declencheurs du mode index-only : "quick answer", "just scan", "don't read the pages", "fast lookup".

### Deroulement observe

1. **Index pass** -- Scan de `index.md` : une entree trouvee pour [[concepts/knowledge-graph|Knowledge Graph]]
2. **Frontmatter scan** -- Lecture du champ `summary:` uniquement : *"Knowledge graphs connect concepts via bidirectional links, turning flat note collections into navigable webs of interconnected ideas."*
3. **Stop** -- Pas de section pass ni de full read.

### Resultat

> **(index-only answer -- page bodies not read; facts below are from page summaries and may miss nuance)**
>
> Knowledge graphs connect concepts via bidirectional links, turning flat note collections into navigable webs of interconnected ideas.
>
> **Pages identified (not read) :** [[concepts/knowledge-graph]]

### Comparaison avec le mode normal

| Aspect | Mode normal | Mode index-only |
|--------|------------|-----------------|
| Pages lues en entier | 2-3 | 0 |
| Longueur de la reponse | Multi-paragraphes, avec relations | 1-2 phrases |
| Sources citees | Avec detail du contenu | Titre + summary uniquement |
| Cross-references explorees | Oui (1 hop) | Non |
| Lacunes identifiees | Oui | Non |
| Temps/cout | Plus eleve | Minimal |

Le mode index-only est ideal pour verifier rapidement si le wiki couvre un sujet, ou pour obtenir une definition rapide sans lire les details.

## 4. Query sur concept inexistant : "blockchain consensus algorithms"

### Commande

```
/wiki query: blockchain consensus algorithms
```

### Deroulement observe

1. **Index pass** -- Scan de `index.md` : aucune entree ne contient "blockchain", "consensus", ou "distributed ledger"
2. **Grep etendu** -- Recherche sur tout le vault avec des termes associes (blockchain, consensus algorithm, proof of stake, proof of work, mining, distributed ledger). Le seul match est "mining" dans le contexte de "Claude Conversation Mining" -- non pertinent.
3. **Resultat : zero pages**

### Resultat

> **Based on the wiki:**
>
> Le wiki ne contient aucun contenu lie aux algorithmes de consensus blockchain. Aucune page ne mentionne blockchain, proof-of-stake, proof-of-work, ou distributed ledger.
>
> **Pages consulted :** 0
>
> **Closest topics :** [[concepts/knowledge-graph|Knowledge Graph]] (structures de connaissances distribuees, mais sans rapport avec la blockchain)
>
> **Pour ajouter ce sujet :** Deposer un document dans `_raw/` et executer le skill `wiki-ingest`.

### Observations

- **Pas d'erreur** -- le skill gere gracieusement l'absence de resultats
- **Suggestion de pages proches** -- meme sans match, le skill propose les sujets les plus adjacents
- **Conseil d'action** -- le skill indique comment combler la lacune (deposer dans `_raw/`)
- **Pas de hallucination** -- le skill ne fabrique pas de reponse quand le wiki ne contient rien

## 5. Navigation dans Obsidian

### Suivre les citations

Chaque reponse de wiki-query contient des `[[wikilinks]]`. Dans Obsidian :

1. `Ctrl/Cmd + clic` sur un wikilink ouvre la page citee
2. Le **backlinks panel** (panneau droit) montre toutes les pages qui referencent la page courante
3. Le **Graph View** (`Ctrl/Cmd + G`) visualise le cluster de pages consultees et leurs connexions

### Verifier le log

Le fichier `log.md` trace chaque query :

```
- [2026-04-15T10:00:00Z] QUERY query="prompt engineering techniques" result_pages=2 mode=normal escalated=false
- [2026-04-15T10:05:00Z] QUERY query="knowledge graph" result_pages=1 mode=index_only escalated=false
- [2026-04-15T10:10:00Z] QUERY query="blockchain consensus algorithms" result_pages=0 mode=normal escalated=false
```

Les champs `mode` et `escalated` permettent de comprendre le chemin de retrieval utilise.

## 6. Observations et limites

### Le retrieval gradue est efficace

Le pipeline en 4 etapes evite de lire tout le vault pour chaque question. L'index pass repond a la majorite des lookups factuels. Le full read n'est necessaire que pour les queries de synthese ou de relation.

### Le mode index-only depend de la qualite des summaries

Si le champ `summary:` du frontmatter est vague ou absent, le mode index-only donne des reponses pauvres. La qualite des summaries produits par l'ingest est donc critique pour ce mode.

### Les wikilinks sont le mecanisme de citation

Contrairement a un moteur de recherche classique qui renvoie des liens, wiki-query renvoie des `[[wikilinks]]` navigables. Cela integre la recherche dans le flux de lecture Obsidian : la reponse est elle-meme un noeud du graphe.

### L'absence de QMD degrade la recherche semantique

Le skill supporte un mode QMD (semantic search via embeddings) qui n'etait pas configure pour ce POC. Sans QMD, la recherche repose sur du grep exact/regex. Les queries formulees differemment du vocabulaire du wiki (synonymes, paraphrases) risquent de manquer des pages pertinentes.

**Precision importante :** QMD est un composant **optionnel**. Le SKILL.md de wiki-query dit explicitement : *"No QMD? Skip to Step 3 and use Grep directly on the vault. QMD is faster and concept-aware but the grep path is fully functional."* Pour un vault de 35 pages, le path grep couvre largement les besoins. QMD deviendrait pertinent sur un vault plus large ou pour des queries semantiquement eloignees du vocabulaire exact des pages (synonymes, reformulations, questions ouvertes). C'est un point a garder en tete pour la Phase 11 (verdict) si on envisage de construire sa propre version.

### Pas de persistance de la reponse

Les reponses de wiki-query ne sont pas sauvegardees comme des pages wiki. Si une meme question est posee plusieurs fois, le pipeline complet est re-execute. Une evolution possible : cacher les reponses frequentes dans `_raw/` ou un dossier `_cache/`.

---

*Article precedent : [[_raw/s0/2026-04/guide-obsidian-wiki/03-data-ingest|Phase 3 — Data ingest (formats libres)]]*

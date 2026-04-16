---
title: "Phase 8 — Export du Graphe de Connaissances"
tags: [guide, obsidian-wiki, wiki-export, knowledge-graph, graphml, neo4j, visualization]
created: 2026-04-15
phase: 8
---

# Phase 8 — Export du Graphe de Connaissances

Ce guide couvre le skill **wiki-export**, qui transforme le graphe de wikilinks du vault en formats exploitables par des outils externes : JSON pour les scripts, GraphML pour Gephi/yEd, Cypher pour Neo4j, et HTML pour la visualisation interactive dans un navigateur.

Prerequis : avoir lu [[_raw/s0/2026-04/guide-obsidian-wiki/07-wiki-update|Phase 7 — Wiki Update]].

## 1. Principe du wiki-export

Le vault Obsidian est un graphe de connaissances implicite : chaque `[[wikilink]]` est une arete, chaque page est un noeud. Le probleme : ce graphe n'existe que dans Obsidian. Le skill wiki-export le rend **portable** en l'extrayant vers des formats standards.

### Ce qu'il extrait

Pour chaque page du vault (hors `_archives/`, `_raw/`, `.obsidian/`, `index.md`, `log.md`, `_insights.md`) :

- **Noeud** : id (chemin relatif sans `.md`), label (titre frontmatter), categorie (dossier parent), tags, summary, community
- **Arete** : source, target, relation (`wikilink`), confidence (`EXTRACTED`, `INFERRED`, `AMBIGUOUS`)

Les liens brises (pointant vers des pages inexistantes) sont ignores silencieusement.

### Communautes

Les pages sont regroupees en communautes par **tag dominant** (premier tag du frontmatter). Les communautes sont numerotees par taille decroissante. Cela permet un coloriage automatique dans la visualisation HTML et dans Gephi.

## 2. Les quatre formats de sortie

Les fichiers sont ecrits dans `wiki-export/` a la racine du vault.

### 2a. graph.json — Format NetworkX node_link

Le format principal. Compatible avec NetworkX, D3.js, et tout script Python/JS.

```json
{
  "directed": false,
  "multigraph": false,
  "graph": {
    "exported_at": "2026-04-16T05:16:09...",
    "total_nodes": 46,
    "total_edges": 231
  },
  "nodes": [
    {
      "id": "concepts/chain-of-thought",
      "label": "Chain-of-Thought Prompting",
      "category": "concepts",
      "tags": ["llm", "prompting", "reasoning", "cot"],
      "summary": "Chain-of-Thought prompting improves LLM reasoning...",
      "community": 1
    }
  ],
  "links": [
    {
      "source": "concepts/chain-of-thought",
      "target": "skills/prompt-engineering-techniques",
      "relation": "wikilink",
      "confidence": "EXTRACTED"
    }
  ]
}
```

**Usage** : `import json; g = json.load(open('graph.json'))` puis traitement avec NetworkX, pandas, ou tout pipeline de donnees.

### 2b. graph.graphml — Format GraphML

Format XML standard, chargeable directement dans Gephi, yEd, et Cytoscape.

```xml
<graphml xmlns="http://graphml.graphdrawing.org/graphml">
  <key id="label" for="node" attr.name="label" attr.type="string"/>
  <key id="category" for="node" attr.name="category" attr.type="string"/>
  <key id="tags" for="node" attr.name="tags" attr.type="string"/>
  <key id="community" for="node" attr.name="community" attr.type="int"/>
  <key id="relation" for="edge" attr.name="relation" attr.type="string"/>
  <graph id="wiki" edgedefault="undirected">
    <node id="concepts/knowledge-graph">
      <data key="label">Knowledge Graph</data>
      <data key="category">concepts</data>
      <data key="tags">knowledge-management, graph, linked-data</data>
      <data key="community">2</data>
    </node>
  </graph>
</graphml>
```

**Usage** : Gephi > File > Open > `graph.graphml`. Les attributs `category` et `community` sont disponibles pour le partitionnement et le coloriage.

### 2c. cypher.txt — Commandes Neo4j Cypher

Fichier de commandes MERGE pour import direct dans Neo4j.

```cypher
// Nodes
MERGE (n:Page {id: "concepts/knowledge-graph"}) SET n.label = "Knowledge Graph",
  n.category = "concepts", n.tags = ["knowledge-management","graph","linked-data"],
  n.community = 2;

// Relationships
MATCH (a:Page {id: "concepts/knowledge-graph"}), (b:Page {id: "entities/obsidian"})
  MERGE (a)-[:WIKILINK {relation: "wikilink", confidence: "EXTRACTED"}]->(b);
```

**Usage** : `cypher-shell -u neo4j -p password < cypher.txt` ou coller dans Neo4j Browser. Les `MERGE` sont idempotents — on peut re-executer a chaque export.

### 2d. graph.html — Visualisation interactive

Fichier HTML autonome utilisant vis.js (CDN). Aucun serveur requis — ouvrir dans n'importe quel navigateur.

Fonctionnalites :
- Noeuds colories par communaute (10 couleurs cycliques)
- Taille des noeuds proportionnelle au degre (nombre de connexions)
- Tooltip au survol : categorie, tags, summary
- Clic sur un noeud : details dans le panneau lateral
- Physique ForceAtlas2 pour le layout, desactivee apres stabilisation
- Legende des communautes et compteur de pages/liens

**Usage** : `open wiki-export/graph.html` ou `xdg-open wiki-export/graph.html`.

## 3. Statistiques du graphe exporte

| Metrique | Valeur |
|---|---|
| Noeuds (pages) | 46 |
| Aretes (wikilinks) | 231 |
| Communautes | 17 |
| Densite | ~0.22 (graphe relativement dense) |

### Top 10 des noeuds les plus connectes

| Page | Liens |
|---|---|
| Phase 2 — Ingest historique Claude | 22 |
| Personal Knowledge Management | 20 |
| Skill-Based Architecture | 20 |
| Obsidian | 19 |
| Phase 1 — Ingest de documents | 17 |
| Documentation as Code | 16 |
| Tool Use and Function Calling | 14 |
| Knowledge Graph | 14 |
| Retrieval-Augmented Generation | 14 |
| LLM-Powered Knowledge Systems | 14 |

### Plus grandes communautes

| ID | Tag dominant | Pages |
|---|---|---|
| 0 | guide | 8 |
| 1 | llm | 5 |
| 2 | knowledge-management | 5 |
| 3 | documentation | 4 |
| 4 | claude-code | 4 |
| 5 | ai-agents | 3 |

## 4. Observations et limites

### Observations

- Le graphe est **fortement connecte** : 231 aretes pour 46 noeuds, soit une densite de ~0.22. C'est le signe d'un wiki bien cross-linke.
- Les **pages guide** (community 0) sont parmi les plus connectees car elles referencent de nombreux concepts et entites.
- Les hubs naturels (`Personal Knowledge Management`, `Skill-Based Architecture`, `Obsidian`) confirment les themes centraux du vault.

### Limites

- **Liens unidirectionnels** : le graphe est declare `undirected` mais les wikilinks ont une direction naturelle (la page qui ecrit le lien). GraphML et Cypher preservent cette info via source/target.
- **Pas de poids** : toutes les aretes ont le meme poids. Un futur enrichissement pourrait compter les mentions multiples.
- **Communautes par tag** : le clustering par premier tag est simpliste. Un algorithme de detection de communautes (Louvain, Label Propagation) donnerait des groupes plus organiques.
- **Liens brises non reportes** : les wikilinks vers des pages inexistantes sont silencieusement ignores. Le skill `wiki-lint` est plus adapte pour les detecter.

### Idees d'evolution

- Export incremental (delta depuis le dernier export)
- Support des poids d'aretes (nombre de mentions)
- Export GEXF pour les timelines dynamiques dans Gephi
- Integration avec un endpoint GraphQL pour requeter le graphe en live

## Voir aussi

- [[concepts/knowledge-graph|Knowledge Graph]]
- [[_raw/s0/2026-04/guide-obsidian-wiki/05-status-lint|Phase 5 — Status & Lint]] pour l'analyse structurelle du vault
- [[_raw/s0/2026-04/guide-obsidian-wiki/06-crosslinker-tags|Phase 6 — Cross-linker & Tag Taxonomy]] pour le maillage des liens

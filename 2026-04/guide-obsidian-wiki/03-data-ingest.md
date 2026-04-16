---
title: "Phase 3 — Data ingest (formats libres)"
tags: [guide, obsidian-wiki, data-ingest, formats]
created: 2026-04-14
phase: 3
---

# Phase 3 — Data ingest (formats libres)

Ce guide explique comment le skill **data-ingest** traite des sources de formats varies -- transcripts, exports de chat, JSON, plain text -- pour en extraire des connaissances et les compiler dans le wiki. C'est le "catch-all" pour tout ce qui n'est pas un document standard ([[_raw/s0/2026-04/guide-obsidian-wiki/01-ingest-documents|Phase 1]]) ou un historique Claude ([[_raw/s0/2026-04/guide-obsidian-wiki/02-history-ingest|Phase 2]]).

Prerequis : avoir lu [[_raw/s0/2026-04/guide-obsidian-wiki/00-setup]] et [[_raw/s0/2026-04/guide-obsidian-wiki/01-ingest-documents]].

## 1. Principe du data-ingest

Le skill `data-ingest` est le **recepteur universel** du framework obsidian-wiki. Il accepte tout ce qui contient du texte exploitable :

| Format | Extensions | Exemple |
|--------|-----------|---------|
| JSON / JSONL | `.json`, `.jsonl` | Exports ChatGPT, dumps d'API |
| Markdown | `.md` | Transcripts de reunion, notes brutes |
| Plain text | `.txt` | Logs, emails, notes |
| CSV / TSV | `.csv`, `.tsv` | Donnees tabulaires |
| HTML | `.html` | Pages web sauvegardees |
| Images | `.png`, `.jpg`, `.webp` | Screenshots, diagrammes (via vision) |
| Chat exports | varies | ChatGPT, Slack, Discord, logs timestamps |

La difference avec `wiki-ingest` (Phase 1) : `data-ingest` n'attend pas un format propre. Il **detecte automatiquement** la structure de chaque fichier et adapte son extraction. Pas de configuration manuelle, pas de mapping prealable.

## 2. Les sources de test

Pour ce POC, deux sources de formats differents ont ete creees dans `_raw/` :

### Transcript de reunion (markdown)

**Fichier** :  [[_raw/s0/2026-04/meeting-transcript-km-system]]

Un transcript de reunion d'equipe technique discutant la mise en place d'un systeme de knowledge management. Format markdown avec participants nommes, horodatage implicite, decisions numerotees et action items.

Pourquoi ce format : les transcripts de reunion sont une source tres courante de [[savoir-implicite|savoir implicite]]. Les decisions d'architecture, les evaluations d'outils, les responsabilites attribuees -- tout est la, mais enfoui dans le flux de la conversation.

### Export ChatGPT (JSON)

**Fichier** :  [[_raw/s0/2025-04/chatgpt-export-documentation-practices]]

Un export au format `conversations.json` de ChatGPT : structure avec `mapping`, `message`, `role`, `content.parts`. Sujet : bonnes pratiques de documentation technique (ADRs, metriques, workflows).

Pourquoi ce format : ChatGPT est une source majeure de savoir technique accumule. L'export JSON a une structure specifique (nested mapping, node IDs) que le skill doit savoir parser. Tester ce format valide la detection automatique.

## 3. Les pages produites

L'ingest a produit **5 nouvelles pages** et **1 mise a jour**, reparties en 3 categories. Le clustering a ete fait par topic, pas par fichier source.

### Skills (3 pages)

Savoir-faire et techniques extraits :

- [[skills/documentation-best-practices|Documentation Best Practices]] -- Trois piliers de la documentation en equipe agile : documentation as code, documentation minimale viable, workflow integre. **Sources croisees** : combine les recommandations de la reunion (wiki gardien, CI linting) et du chat (principes, outils a deux niveaux).

- [[skills/architecture-decision-records|Architecture Decision Records]] -- Format ADR (Architecture Decision Record) pour capturer les decisions avec contexte, rationale et consequences. Lifecycle d'immutabilite. Extrait principalement du chat ChatGPT.

- [[skills/knowledge-management-tool-selection|Knowledge Management Tool Selection]] -- Processus d'evaluation et de selection d'un outil KM : criteres, comparaison Confluence/Notion/Obsidian, decision. Extrait de la reunion.

### Concepts (2 pages)

Idees et patterns abstraits :

- [[concepts/documentation-metrics|Documentation Metrics and Quality]] -- Quatre categories de KPIs pour mesurer l'efficacite de la documentation (usage, fraicheur, impact, contribution). **Sources croisees** : metriques detaillees du chat + exigence de fraicheur de la reunion.

- [[concepts/wiki-governance|Wiki Governance]] -- Modele de gouvernance pour un wiki vivant : role de wiki gardien, deploiement en phases, integration dans le workflow de dev. **Sources croisees** : organisation de la reunion + pratiques du chat.

### Mise a jour (1 page)

- [[synthesis/documentation-as-code|Documentation as Code]] -- Page existante (creee en Phase 2) enrichie avec la perspective equipe : pratiques CI/CD, ADRs, definition of done. Nouvelles sources ajoutees, nouveaux liens vers les pages creees.

## 4. Mecanismes cles

### Detection automatique de format

Le skill a correctement identifie :
- `meeting-transcript-km-system.md` comme **markdown_transcript** -- detecte via l'extension `.md` et le pattern de dialogue (participants nommes, tours de parole)
- `chatgpt-export-documentation-practices.json` comme **chatgpt_export** -- detecte via l'extension `.json` et la structure `mapping > message > role > content > parts`

Aucune configuration n'a ete necessaire. Le champ `format_detected` dans `.manifest.json` trace la detection pour chaque source.

### Clustering par topic plutot que par fichier source

Les deux sources parlent de documentation technique mais sous des angles differents. Au lieu de creer des pages separees par source, l'ingest a **croise les connaissances** :

| Page | Source reunion | Source ChatGPT |
|------|---------------|----------------|
| [[skills/documentation-best-practices\|Documentation Best Practices]] | Wiki gardien, CI linting, curation | Trois piliers, outils a deux niveaux |
| [[concepts/wiki-governance\|Wiki Governance]] | Rotation, deploiement en phases | Definition of done, workflow integre |
| [[concepts/documentation-metrics\|Documentation Metrics and Quality]] | Exigence de fraicheur | KPIs detailles (4 categories) |
| [[skills/knowledge-management-tool-selection\|KM Tool Selection]] | Evaluation complete | -- |
| [[skills/architecture-decision-records\|ADRs]] | -- | Structure et lifecycle |

C'est le meme principe que les phases precedentes : le wiki est un **graphe compile**, pas un miroir des sources.

### Provenance markers adaptes au type de source

Les conversations (reunion comme chat) produisent un mix de contenu extrait et infere :

- **`^[extracted]`** -- Fait directement cite dans la source. Exemple : "On choisit Obsidian" (decision explicite dans le transcript), structure ADR (decrite mot a mot dans le chat).
- **`^[inferred]`** -- Synthese ou generalisation. Exemple : "Un wiki sans gouvernance devient un cimetiere de pages" (infere de la discussion sur Confluence).

Les deux sources etant des conversations, le ratio infere/extrait est plus eleve que pour des documents structures (Phase 1). Le bloc `provenance:` dans le frontmatter de chaque page documente cette distinction.

### Support multi-format dans un seul skill

Le meme skill `data-ingest` a traite un fichier markdown et un fichier JSON sans changement de configuration. C'est le point fort du design : un seul point d'entree pour tous les formats non-standard, avec detection et adaptation automatiques.

## 5. Comment verifier dans Obsidian

### Observer les nouvelles pages

1. Ouvrir le vault dans Obsidian (`knlg-repo/`)
2. Activer le **Graph View** (`Ctrl/Cmd + G`)
3. Observer les 5 nouveaux noeuds : documentation-best-practices, architecture-decision-records, documentation-metrics, knowledge-management-tool-selection, wiki-governance
4. Remarquer comment ils se connectent aux pages existantes -- liens vers [[entities/obsidian|Obsidian]], [[concepts/personal-knowledge-management|PKM]], [[synthesis/documentation-as-code|Documentation as Code]], [[concepts/zettelkasten|Zettelkasten]]

### Verifier le croisement des sources

1. Ouvrir [[skills/documentation-best-practices|Documentation Best Practices]]
2. Observer le champ `sources:` dans le frontmatter -- les deux sources y sont listees
3. Dans le corps de la page, verifier les marqueurs `^[extracted]` et `^[inferred]`
4. Suivre les liens vers [[concepts/wiki-governance|Wiki Governance]] et [[concepts/documentation-metrics|Documentation Metrics]]

### Inspecter le manifest

1. Ouvrir `.manifest.json`
2. Chercher les deux nouvelles entrees dans `"sources"` :
   - `_raw/meeting-transcript-km-system.md` avec `format_detected: "markdown_transcript"`
   - `_raw/chatgpt-export-documentation-practices.json` avec `format_detected: "chatgpt_export"`
3. Verifier les `pages_created` et `pages_updated` pour chaque source
4. Observer que les stats globales ont ete mises a jour : `total_sources_ingested: 17`, `total_pages: 35`

## 6. Observations

### Flexibilite du format

Le data-ingest confirme sa capacite a traiter des sources tres differentes (markdown structure vs JSON nested) sans configuration prealable. Le meme pipeline de detection > extraction > clustering > distillation fonctionne pour les deux.

### Limite : inference lourde sur les conversations

Les sources conversationnelles (transcript comme chat) necessitent plus d'inference que les documents structures. Les marqueurs `^[inferred]` sont plus frequents. C'est attendu et documente dans le frontmatter, mais cela signifie que les pages issues de conversations meritent plus de verification humaine.

### Le clustering multi-source est le vrai apport

La valeur ajoutee principale n'est pas le parsing de format -- c'est le **croisement des connaissances** entre sources. La reunion apporte le contexte organisationnel (qui fait quoi, quand), le chat apporte le savoir technique (comment structurer un ADR, quelles metriques utiliser). L'ingest combine les deux dans des pages coherentes.

### Principe retenu : un skill, plusieurs formats

Plutot que de creer un skill par format source (un pour les transcripts, un pour les exports ChatGPT, un pour les CSV...), le framework centralise tout dans `data-ingest`. L'intelligence est dans la detection et l'adaptation, pas dans le routage. Cela simplifie l'experience utilisateur : "ingere ca" suffit, quel que soit le format.

---

*Article precedent : [[_raw/s0/2026-04/guide-obsidian-wiki/02-history-ingest|Phase 2 — Ingest historique Claude]]*

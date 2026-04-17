---
title: "Phase 2 — Ingest historique Claude"
tags: [guide, obsidian-wiki, claude-history-ingest, history, mining]
created: 2026-04-14
phase: 2
---

# Phase 2 — Ingest historique Claude

Ce guide explique comment **claude-history-ingest** mine l'historique des conversations Claude Code pour en extraire des connaissances et les compiler dans le wiki. Il couvre la structure des donnees Claude, le processus de minage, les resultats obtenus, et les mecanismes cles a observer.

Prerequis : avoir lu [[_raw/s0/2026-04/guide-obsidian-wiki/00-setup]] et [[_raw/s0/2026-04/guide-obsidian-wiki/01-ingest-documents]].

## 1. Principe du claude-history-ingest

L'idee centrale : **chaque conversation avec Claude Code est un gisement de savoir implicite**. Quand on debogue un systeme, qu'on prend une decision d'architecture, qu'on apprend un pattern -- tout cela reste enfoui dans les logs de conversation. Le skill `claude-history-ingest` extrait ce savoir et le compile dans le wiki.

### La structure de `~/.claude/`

Claude Code stocke tout sous `~/.claude/`. Voici l'organisation :

```
~/.claude/
├── projects/                          # Un dossier par projet
│   ├── -home-omc-workspace-kiss-claw/ # Nom derive du chemin (/ → -)
│   │   ├── <session-uuid>.jsonl       # Conversations (format JSONL)
│   │   └── memory/                    # Memoires structurees
│   │       ├── MEMORY.md              # Index des memoires
│   │       ├── user_*.md              # Profil utilisateur
│   │       ├── feedback_*.md          # Feedback workflow
│   │       └── project_*.md           # Contexte projet
│   ├── -home-omc-workspace-se-vault/
│   │   └── ...
├── sessions/                          # Metadonnees de session (JSON)
│   └── <pid>.json                     # {pid, sessionId, cwd, startedAt}
├── history.jsonl                      # Historique global des sessions
└── settings.json
```

### Hierarchie de valeur des sources

Toutes les sources ne se valent pas. L'ingest suit une hierarchie stricte :

| Rang | Source | Valeur | Pourquoi |
|------|--------|--------|----------|
| 1 | **Memory files** (`projects/*/memory/*.md`) | Or | Deja structures avec frontmatter type (user/feedback/project/reference). Pre-distilles par Claude Code lui-meme. Prets a integrer. |
| 2 | **Conversations JSONL** (`projects/*/*.jsonl`) | Riche mais bruite | Transcriptions completes des echanges. Contiennent le savoir mais aussi le bruit (essais-erreurs, reformulations, tool calls). Necessitent filtrage et clustering. |
| 3 | **Session metadata** (`sessions/*.json`) | Contextuel | Indique quel projet, quand, quel repertoire de travail. Utile pour le tri et la correlation, pas pour le contenu. |

Cette hierarchie dicte l'ordre de traitement : memory files d'abord (meilleur ROI), puis conversations, puis metadata pour le contexte.

## 2. Le routeur wiki-history-ingest

L'ingest d'historique passe par un **routeur** : le skill `wiki-history-ingest`. Son role est simple -- dispatcher vers le bon skill specialise selon la cible :

| Commande | Route vers |
|----------|-----------|
| `/wiki-history-ingest claude` | `claude-history-ingest` |
| `/wiki-history-ingest codex` | `codex-history-ingest` |
| `/wiki-history-ingest auto` | Inference automatique selon le contexte |

Le principe d'extensibilite : ajouter le support d'un nouvel agent (Gemini, Cursor, etc.) revient a creer un nouveau skill `<agent>-history-ingest` et a ajouter une ligne dans la table de routage. Le routeur ne contient aucune logique de traitement -- il delegue entierement au skill destination.

C'est la meme separation qu'on retrouve dans [[concepts/skill-routing|Skill Routing]] : un dispatcher leger qui route, des skills specialises qui executent.

## 3. Ce qui a ete mine

Le minage de `~/.claude/` a revele l'ampleur de l'historique accumule :

### Inventaire

| Metrique | Valeur |
|----------|--------|
| Projets Claude decouverts | 6 |
| Conversations totales | 217 (reparties sur les 6 projets) |
| Memory files | 4 (dans 2 projets : [[projects/is-vault/is-vault|is-vault]] et [[projects/se-vault/se-vault|se-vault]]) |

Le projet le plus actif est [[projects/caminao-scraper/caminao-scraper|caminao-scraper]] avec 154 conversations, suivi de [[projects/kiss-claw/kiss-claw|kiss-claw]] avec 37.

### Strategie d'echantillonnage

Pour ce POC, l'ingest n'a pas traite toutes les conversations. La strategie :

- **Memory files** : 100% traites (4 fichiers dans is-vault et se-vault)
- **Conversations** : 3 les plus recentes par projet (ou moins si le projet en a moins)
- **Total echantillonne** : 16 conversations sur 217

Pourquoi 3 par projet ? C'est un compromis POC : assez pour capturer les sujets principaux d'un projet, pas assez pour etre exhaustif. Un ingest complet traiterait les 217 conversations mais consommerait significativement plus de tokens.

Les 2 projets generiques (sans nom de projet specifique, correspondant a `~/.claude/projects/-home-omc/` et `~/.claude/projects/-home-omc-workspace/`) ont ete echantillonnes aussi (2 et 3 conversations) mais n'ont pas produit de pages -- leur contenu etait trop generique ou redondant avec les projets nommes.

## 4. Les pages produites

L'ingest historique a cree **17 pages** reparties en 5 categories. Aucune page existante n'a ete mise a jour -- tout est creation nette.

### Projets (7 pages)

Chaque projet Claude mine se retrouve dans `projects/<name>/` avec une page principale :

- [[projects/kiss-claw/kiss-claw|Kiss-Claw]] -- Framework d'orchestration multi-agents (orchestrator/executor/verificator) pour Claude Code, avec sessions, checkpoints et pipelines konvert
- [[projects/kiss-claw/concepts/checkpoint-enrichment|Checkpoint Enrichment]] -- Sous-concept de kiss-claw : enrichissement des CHECKPOINT.yaml par parsing des logs JSONL
- [[projects/caminao-scraper/caminao-scraper|Caminao Scraper]] -- Scraper de contenu lie a l'architecture d'entreprise et aux frameworks TOGAF
- [[projects/se-vault/se-vault|SE-Vault]] -- Vault oriente systems engineering, avec strategie git submodule
- [[projects/is-vault/is-vault|IS-Vault]] -- Vault oriente information systems, partageant la strategie git de se-vault
- [[projects/backup-strategy/backup-strategy|Backup Strategy]] -- Strategie de sauvegarde et documentation as code
- [[projects/oh-my-claudecode/oh-my-claudecode|Oh-My-Claudecode]] -- Couche d'orchestration multi-agents pour Claude Code

### Concepts (3 pages)

Des idees generales extraites ou inferees a partir de plusieurs projets :

- [[concepts/multi-agent-orchestration|Multi-Agent Orchestration]] -- Le pattern central de kiss-claw et oh-my-claudecode : coordination d'agents specialises
- [[concepts/iec-15288|IEC 15288]] -- Standard d'ingenierie systeme, reference dans se-vault
- [[concepts/enterprise-architecture|Enterprise Architecture]] -- Architecture d'entreprise, lie au travail de caminao-scraper

### Entities (2 pages)

Outils et frameworks concrets identifies dans les conversations :

- [[entities/claude-code|Claude Code]] -- L'outil lui-meme, identifie comme entite a partir des conversations kiss-claw
- [[entities/togaf|TOGAF]] -- Framework d'architecture d'entreprise, reference dans caminao-scraper

### Skills (3 pages)

Savoir-faire et techniques extraits des pratiques observees :

- [[skills/git-submodule-strategies|Git Submodule Strategies]] -- Strategies de gestion des submodules git, derives des memory files de se-vault
- [[skills/tdd-with-ai-agents|TDD with AI Agents]] -- Pratique TDD appliquee au developpement avec agents IA, observee dans kiss-claw
- [[skills/claude-session-management|Claude Session Management]] -- Gestion des sessions Claude Code, patterns et bonnes pratiques

### Synthesis (2 pages)

Analyses transversales croisant plusieurs projets et sources :

- [[vault/s0/synthesis/claude-conversation-mining|Claude Conversation Mining]] -- Le meta-processus de minage lui-meme : comment extraire du savoir des conversations
- [[vault/s0/synthesis/documentation-as-code|Documentation as Code]] -- Pattern recurrent dans se-vault et backup-strategy : traiter la documentation comme du code

### Projet-specifique vs global

La repartition suit une logique claire :

| Ce qui a ete trouve | Ou ca atterrit | Exemple |
|---------------------|---------------|---------|
| Architecture et decisions d'un projet | `projects/<name>/` | [[projects/kiss-claw/kiss-claw|kiss-claw.md]] |
| Sous-concept propre a un projet | `projects/<name>/concepts/` | [[projects/kiss-claw/concepts/checkpoint-enrichment|checkpoint-enrichment.md]] |
| Concept general apparu dans plusieurs projets | `concepts/` | [[concepts/multi-agent-orchestration|multi-agent-orchestration.md]] |
| Outil ou framework concret | `entities/` | [[entities/togaf|togaf.md]] |
| Technique reutilisable | `skills/` | [[skills/tdd-with-ai-agents|tdd-with-ai-agents.md]] |
| Analyse croisant plusieurs sources | `synthesis/` | [[synthesis/documentation-as-code|documentation-as-code.md]] |

## 5. Mecanismes cles observes

### Clustering par topic

L'ingest ne cree **pas une page par conversation**. Il regroupe les connaissances par theme. Trois conversations kiss-claw parlant d'orchestration multi-agents ne produisent pas trois pages -- elles alimentent une seule page [[concepts/multi-agent-orchestration|Multi-Agent Orchestration]] qui compile le savoir des trois.

C'est le meme principe que dans la Phase 1 ([[_raw/s0/2026-04/guide-obsidian-wiki/01-ingest-documents]]) : le wiki est un **graphe compile**, pas un miroir des sources.

### Distillation, pas copie

L'ingest ne copie jamais le texte des conversations. Il en extrait la **substance** et la reformule en connaissances structurees. Si une conversation de 200 echanges porte sur le debugging d'un pipeline CI, la page wiki contiendra les patterns de debugging identifies, pas le recit de la session.

Ce principe est aussi une **garantie de confidentialite** : pas de texte verbatim, pas de secrets, pas de donnees personnelles dans le wiki.

### Provenance differenciee

La distinction entre `extracted` et `inferred` prend tout son sens avec l'historique Claude :

- **`^[extracted]`** -- Fait directement tire d'un memory file. L'utilisateur l'a ecrit ou valide explicitement. Haute confiance.
- **`^[inferred]`** -- Synthese a partir de conversations. L'ingest a deduit un pattern, une decision, ou une generalisation. Confiance moderee.

Exemple concret : la page [[projects/kiss-claw/kiss-claw|Kiss-Claw]] est quasi-entierement `^[inferred]` (pas de memory files pour ce projet, tout vient des conversations). A l'inverse, [[skills/git-submodule-strategies|Git Submodule Strategies]] derive des memory files de se-vault -- plus haute confiance.

### Delta tracking

Le `.manifest.json` enregistre chaque source ingeree avec ses metadonnees :

```json
"kiss-claw": {
  "source_path": "~/.claude/projects/-home-omc-workspace-kiss-claw",
  "vault_path": "projects/kiss-claw",
  "last_ingested": "2026-04-14T23:00:00Z",
  "conversations_ingested": 3,
  "conversations_total": 37,
  "memory_files_ingested": 0
}
```

Lors d'un futur ingest en mode append, le skill comparera les timestamps et ne traitera que les nouvelles conversations (celles creees apres `last_ingested`). Les 34 conversations restantes de kiss-claw seront candidates au prochain run.

### Projection projet

Chaque projet Claude se retrouve dans un dossier dedie `projects/<name>/` avec :
- Une page principale nommee `<name>.md` (pas `_project.md` -- pour que le Graph View d'Obsidian affiche le nom du projet comme label du noeud)
- Des sous-pages optionnelles dans `concepts/`, `skills/`, etc. quand le projet a des sous-topics suffisamment denses

## 6. Comment verifier dans Obsidian

### Observer les nouveaux noeuds projet

1. Ouvrir le vault dans Obsidian (`knlg-repo/`)
2. Activer le **Graph View** (`Ctrl/Cmd + G`)
3. Observer les 6 nouveaux noeuds projet : kiss-claw, caminao-scraper, se-vault, is-vault, backup-strategy, oh-my-claudecode
4. Remarquer comment ils se connectent aux concepts globaux -- kiss-claw lie vers [[concepts/multi-agent-orchestration|multi-agent-orchestration]], caminao-scraper vers [[concepts/enterprise-architecture|enterprise-architecture]], etc.

### Suivre les liens d'un projet

1. Ouvrir [[projects/kiss-claw/kiss-claw|projects/kiss-claw/kiss-claw.md]]
2. Suivre le lien vers [[projects/kiss-claw/concepts/checkpoint-enrichment|Checkpoint Enrichment]] -- un sous-concept specifique au projet
3. Suivre les liens vers les concepts globaux : [[concepts/multi-agent-orchestration|Multi-Agent Orchestration]], [[skills/tdd-with-ai-agents|TDD with AI Agents]]
4. Observer comment un projet Claude se decompose en un reseau de pages interconnectees

### Comparer provenance extracted vs inferred

1. Ouvrir [[skills/git-submodule-strategies|Git Submodule Strategies]] -- derive des memory files de se-vault. Observer les claims sans marqueur (extracted) et le bloc `provenance:` dans le frontmatter
2. Ouvrir [[projects/kiss-claw/kiss-claw|Kiss-Claw]] -- derive des conversations. Observer les marqueurs `^[inferred]` sur la plupart des claims
3. La difference de confiance est visible : les pages memory-derived sont plus factuelles, les pages conversation-derived sont plus interpretatives

### Inspecter le manifest

1. Ouvrir `.manifest.json` (activer "Show hidden files" dans les settings Obsidian)
2. Aller a la section `"projects"` -- elle liste les 6 projets avec leurs stats d'ingest
3. Comparer `conversations_ingested` vs `conversations_total` pour voir la couverture d'echantillonnage
4. Noter que is-vault a `conversations_ingested: 0` et `memory_files_ingested: 2` -- entierement derive des memory files

## 7. Observations et principes retenus

### L'historique Claude est une mine d'or de savoir implicite

Chaque conversation contient des decisions, des debuggings, des patterns appris. Sans extraction, ce savoir reste enfoui dans des fichiers JSONL que personne ne relira. L'ingest historique le rend navigable et retrouvable.

### Les memory files sont le meilleur ROI

4 memory files ont produit des pages de haute confiance avec un minimum de traitement. Ils sont deja structures, types (user/feedback/project/reference), et pre-distilles. C'est la premiere chose a traiter dans tout ingest historique.

### Le clustering thematique evite la fragmentation

Sans clustering, 16 conversations auraient pu produire 16+ pages disparates. Le regroupement par topic a produit 17 pages coherentes ou les connaissances de plusieurs sessions convergent. C'est le meme principe que l'ingest de documents ([[_raw/s0/2026-04/guide-obsidian-wiki/01-ingest-documents]]) : compiler, pas copier.

### La distinction extracted/inferred est cruciale pour la confiance

Un lecteur du wiki doit pouvoir distinguer un fait confirme d'une inference. La provenance differenciee permet de calibrer sa confiance : une affirmation `^[extracted]` d'un memory file est fiable ; une generalisation `^[inferred]` d'une conversation merite verification.

### L'echantillonnage suffit pour un POC

3 conversations par projet (16 sur 217) ont suffi pour capturer les themes principaux de chaque projet. Un ingest complet donnerait plus de profondeur et de details, mais le rapport signal/bruit diminuerait -- les conversations les plus recentes sont generalement les plus representatives de l'etat actuel d'un projet.

---

*Article precedent : [[_raw/s0/2026-04/guide-obsidian-wiki/01-ingest-documents|Phase 1 — Ingest de documents]]*

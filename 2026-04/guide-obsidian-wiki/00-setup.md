---
title: "Phase 0 — Setup & Configuration"
tags: [guide, obsidian-wiki, setup]
created: 2026-04-14
phase: 0
---

# Phase 0 — Setup & Configuration

Ce premier article explique comment obsidian-wiki initialise un vault et ce que chaque element de la structure represente. A la fin, tu sauras lire la structure d'un vault fraichement cree et comprendre le role de chaque dossier et fichier.

## Ce que fait wiki-setup

Le skill `wiki-setup` (situe dans `.skills/wiki-setup/SKILL.md` du repo obsidian-wiki) est le point d'entree pour creer un vault. Son principe est simple : a partir d'un chemin `OBSIDIAN_VAULT_PATH`, il cree une arborescence de dossiers et de fichiers fondateurs qui forment la structure d'un wiki Obsidian pret a l'emploi.

L'idee centrale : **le vault est un graphe de connaissances pre-compile**, pas un simple dossier de notes. Chaque page a du frontmatter structure, des `[[wikilinks]]` vers d'autres pages, et des tags issus d'un vocabulaire controle.

## La structure creee

Voici ce que `wiki-setup` produit dans le vault :

### Dossiers de contenu

| Dossier | Role |
|---|---|
| `concepts/` | Idees abstraites, patterns, modeles mentaux |
| `entities/` | Choses concretes — personnes, outils, bibliotheques, entreprises |
| `skills/` | Savoir-faire, techniques, procedures (how-to) |
| `references/` | Donnees factuelles — specs, APIs, configurations |
| `synthesis/` | Analyses transversales reliant plusieurs concepts |

Ces cinq dossiers correspondent aux **categories** definies dans la [[_meta/taxonomy|taxonomie]]. Chaque page wiki creee par ingest atterrit dans l'un de ces dossiers selon sa nature.

### Dossiers temporels et projet

| Dossier | Role |
|---|---|
| `journal/` | Entrees liees au temps — logs quotidiens, notes de session |
| `projects/` | Une page par projet synchronise via `wiki-update` |

### Zones techniques

| Dossier / Fichier | Role |
|---|---|
| `_archives/` | Pages archivees (retirees de l'index actif mais conservees) |
| `_raw/` | Zone de staging — depose ici des notes brutes, le prochain ingest les promouvra |
| `_meta/` | Metadonnees du wiki |
| `_meta/taxonomy.md` | Vocabulaire de tags controle — tout nouveau tag doit y etre declare avant usage |

### Fichiers racine

| Fichier | Role |
|---|---|
| `index.md` | Index principal — chaque page du wiki y est listee, mis a jour automatiquement |
| `log.md` | Journal d'activite chronologique (ingests, updates, lints) |
| `.manifest.json` | Registre d'ingest : suit chaque source ingeree (chemin, timestamps, pages produites). Utilise un **hash-based delta tracking** pour ne re-traiter que ce qui a change |
| `.obsidian/` | Configuration Obsidian (apparence, plugins, etc.) |

## Comment verifier dans Obsidian

1. **Ouvre le vault** — dans Obsidian, `Open folder as vault` et pointe vers le dossier du vault (dans notre cas : `knlg-repo/`)
2. **Verifie l'arborescence** — dans le panneau de fichiers a gauche, tu dois voir tous les dossiers listes ci-dessus : `concepts/`, `entities/`, `skills/`, `references/`, `synthesis/`, `journal/`, `projects/`, `_archives/`, `_raw/`, `_meta/`
3. **Ouvre [[index]]** — c'est la page d'accueil du wiki. Elle liste les sections (Concepts, Entities, Skills, etc.) mais elles sont encore vides : "No pages yet"
4. **Ouvre [[log]]** — le journal d'activite. Apres un setup frais, il est vide ou contient une seule entree d'initialisation
5. **Verifie [[_meta/taxonomy]]** — tu y trouves les categories (`concept`, `entity`, `skill`, `reference`, `synthesis`, `journal`) et la section Tags encore vide
6. **Active le Graph View** (`Ctrl+G`) — tu verras les quelques noeuds fondateurs. Au fur et a mesure des ingests, ce graphe se densifiera

## Observations

### Le setup.sh du repo obsidian-wiki

Le repo `vendor/obsidian-wiki` contient un script `setup.sh` qui fait bien plus que creer la structure du vault :

- **Cree `.env`** a partir de `.env.example` avec la variable `OBSIDIAN_VAULT_PATH`
- **Cree une config globale** dans `~/.obsidian-wiki/config` (contenant `OBSIDIAN_VAULT_PATH` et `OBSIDIAN_WIKI_REPO`)
- **Symlinke les skills** dans les dossiers de chaque agent AI supporte :
  - `~/.claude/skills/` (Claude Code)
  - `~/.gemini/antigravity/skills/` (Gemini)
  - `~/.codex/skills/` (Codex)
  - `~/.agents/skills/` (OpenClaw et agents generiques)
  - Plus les dossiers locaux `.claude/skills/`, `.cursor/skills/`, `.windsurf/skills/`, `.agents/skills/`

**Point important** : `setup.sh` utilise `read -p` pour demander interactivement le chemin du vault. Il est donc **incompatible avec une execution automatique** (CI, scripts, agents).

**Pour notre POC**, `setup.sh` n'a pas ete execute. Ce n'est pas necessaire car :
- Les skills sont lisibles directement dans `vendor/obsidian-wiki/.skills/`
- Le `.env` (avec `OBSIDIAN_VAULT_PATH`) a ete configure manuellement
- La structure du vault a ete creee en invoquant directement le skill `wiki-setup`

Cela revele un aspect du design d'obsidian-wiki : **il suppose une installation globale par utilisateur** (`~/.obsidian-wiki/config`, symlinks dans `~/`). Pour un usage en POC ou en CI, on contourne ce mecanisme.

### Le .env

Le seul parametre de configuration reellement necessaire est :

```
OBSIDIAN_VAULT_PATH=/home/omc/workspace/my-obsidian-wiki/knlg-repo
```

Ce chemin est utilise par tous les skills pour localiser le vault. La chaine de resolution est : `~/.obsidian-wiki/config` (prioritaire) puis `.env` dans le repo obsidian-wiki (fallback).

---

*Prochaine etape : [[guide/01-ingest|Phase 1 — Premier Ingest]]*

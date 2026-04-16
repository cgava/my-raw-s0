---
title: "Phase 9 — Rebuild & Archive"
tags: [guide, obsidian-wiki, wiki-rebuild, archive, restore, backup, snapshot]
created: 2026-04-15
phase: 9
---

# Phase 9 — Rebuild & Archive

Ce guide couvre le skill **wiki-rebuild**, qui permet d'archiver l'etat du vault, de le reconstruire depuis zero, ou de restaurer un etat anterieur. C'est le filet de securite du wiki : avant toute operation destructive, on snapshote, et on peut revenir en arriere a tout moment.

Prerequis : avoir lu [[_raw/s0/2026-04/guide-obsidian-wiki/08-export|Phase 8 — Export du graphe]].

## 1. Principe du wiki-rebuild

Le wiki-rebuild supporte trois modes :

| Mode | Usage |
|---|---|
| **Archive only** | Creer un snapshot sans toucher au wiki. Utile avant une operation risquee. |
| **Archive + Rebuild** | Archiver, vider le vault, puis re-ingerer depuis les sources. |
| **Restore** | Revenir a un etat anterieur depuis une archive. |

Le principe fondamental : **toujours archiver avant de detruire**. Les archives sont immutables et peu couteuses (quelques centaines de Ko pour un vault de 50 pages).

## 2. Le systeme d'archives

### Emplacement

Les archives vivent dans `_archives/` a la racine du vault. Chaque archive est un repertoire timestampe au format ISO :

```
$VAULT/
├── _archives/
│   ├── 2026-04-15T12-00-00Z/
│   │   ├── archive-meta.json
│   │   ├── .manifest.json
│   │   ├── index.md
│   │   ├── log.md
│   │   ├── concepts/
│   │   ├── entities/
│   │   ├── skills/
│   │   ├── synthesis/
│   │   ├── projects/
│   │   └── guide/
│   └── ...
├── concepts/          ← wiki live
└── ...
```

### archive-meta.json

Chaque archive contient un fichier de metadonnees :

```json
{
  "archived_at": "2026-04-15T12:00:00Z",
  "reason": "snapshot",
  "total_pages": 48,
  "total_sources": 18,
  "total_projects": 7,
  "vault_path": "/home/omc/workspace/my-obsidian-wiki/knlg-repo",
  "manifest_snapshot": ".manifest.json"
}
```

Le champ `reason` peut valoir `snapshot`, `rebuild`, ou `pre-restore` selon le contexte.

### Contenu de l'archive

L'archive copie tous les repertoires de contenu (`concepts/`, `entities/`, `skills/`, `references/`, `synthesis/`, `journal/`, `projects/`, `guide/`), plus les fichiers structurels (`index.md`, `log.md`, `.manifest.json`).

Sont exclus : `_archives/` (pour eviter la recursion), `.obsidian/` (config utilisateur, jamais touchee), `_raw/` (sources brutes, deja versionnees), `.git/`.

## 3. Test complet : Archive, Corruption, Restore

### 3a. Creation de l'archive (snapshot)

Creation d'un snapshot du vault dans son etat courant :

- Repertoire : `_archives/2026-04-15T12-00-00Z/`
- Taille : 372 Ko
- Fichiers archives : 58 (dont 48 pages markdown)
- Raison : `snapshot`

Le log enregistre : `ARCHIVE reason="snapshot" pages=48 destination="_archives/2026-04-15T12-00-00Z"`.

### 3b. Simulation de corruption

Suppression de 3 pages du repertoire `concepts/` :

| Page supprimee | Contenu |
|---|---|
| `concepts/knowledge-graph.md` | Knowledge Graph — graphes de connaissances et wikilinks |
| `concepts/zettelkasten.md` | Methode Zettelkasten — une idee par note, liens inter-notes |
| `concepts/multi-agent-orchestration.md` | Orchestration multi-agents et patterns de delegation |

Verification : les fichiers sont absents du filesystem, mais toujours references dans `index.md` et le `.manifest.json` — exactement le type d'incoherence qu'un `wiki-lint` detecterait comme "broken links".

### 3c. Restauration depuis l'archive

Copie des 3 fichiers manquants depuis `_archives/2026-04-15T12-00-00Z/concepts/` vers `concepts/`.

Le log enregistre : `RESTORE from="_archives/2026-04-15T12-00-00Z" pages_restored=3 reason="corruption test"`.

### 3d. Verification d'integrite

Comparaison fichier par fichier du vault avant corruption et apres restauration :

- **Diff : aucune difference.** Toutes les pages sont presentes.
- Frontmatter des pages restaurees : intact (title, category, tags, sources, summary, provenance).
- Wikilinks : les 3 pages sont toujours referencees dans `index.md` (3 occurrences).
- `.manifest.json` : inchange (il n'avait pas ete modifie par la "corruption").

## 4. Utilisation en pratique

### Quand archiver

- **Avant un rebuild complet** : si on veut re-ingerer depuis zero
- **Avant un ingest massif** : ajout de nombreuses sources en une fois
- **Avant une migration** : changement de structure, renommage de categories
- **Periodiquement** : en complement du versioning git, un snapshot explicite est plus lisible qu'un diff

### Quand restaurer

- **Corruption accidentelle** : suppression involontaire de pages
- **Ingest rate** : un ingest a produit des pages de mauvaise qualite — on revient en arriere
- **Experimentation** : on teste un rebuild et on veut revenir a l'etat precedent

### Workflow recommande

```
1. wiki-rebuild archive    → snapshot de securite
2. (operation risquee)     → rebuild, ingest massif, migration...
3. wiki-lint               → verifier l'integrite
4. Si probleme :
   wiki-rebuild restore    → revenir au snapshot
```

## 5. Observations et limites

### Observations

- L'archive est **legere** : 372 Ko pour 48 pages. Meme avec des centaines de pages, ca reste negligeable.
- Le format est **transparent** : ce sont des fichiers markdown et JSON copie a plat, pas un format binaire opaque. On peut inspecter une archive manuellement.
- Le restore est **selectif** possible : on peut copier un seul fichier depuis l'archive plutot que tout restaurer.

### Limites

- **Pas de compression** : les archives sont des copies brutes. Pour un vault tres volumineux, on pourrait ajouter un tar.gz.
- **Pas de rotation automatique** : les anciennes archives s'accumulent. Il faudrait un mecanisme de purge (garder les N dernieres).
- **Pas de diff inter-archives** : pour comparer deux snapshots, il faut faire un diff manuel entre les repertoires.
- **Les fichiers _raw/ ne sont pas archives** : si une source brute est modifiee ou supprimee entre l'archive et le restore, le re-ingest produira des resultats differents.

### Idees d'evolution

- Compression des archives (tar.gz) avec decompression transparente au restore
- Rotation automatique (garder les 5 dernieres archives, supprimer les plus anciennes)
- Commande `wiki-rebuild diff` pour comparer deux archives
- Integration avec git tags pour coupler archive wiki et commit git
- Archive incrementale (ne copier que les fichiers modifies depuis la derniere archive)

## Voir aussi

- [[concepts/knowledge-graph|Knowledge Graph]]
- [[concepts/personal-knowledge-management|Personal Knowledge Management]]
- [[_raw/s0/2026-04/guide-obsidian-wiki/05-status-lint|Phase 5 — Status & Lint]] pour la verification d'integrite
- [[_raw/s0/2026-04/guide-obsidian-wiki/08-export|Phase 8 — Export du graphe]]

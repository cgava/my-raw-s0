---
title: "Guide POC obsidian-wiki — Index"
tags: [guide, obsidian-wiki, index, poc]
created: 2026-04-15
phase: index
---

# Guide POC obsidian-wiki

## Objectif

Ce guide documente l'exploration methodique du framework **obsidian-wiki** — un ensemble de skills Claude Code pour construire et maintenir un graphe de connaissances dans un vault Obsidian.

L'objectif n'est pas de produire un tutoriel d'utilisation, mais de **comprendre les grands principes** du framework en le testant skill par skill, de noter ce qui fonctionne bien, ce qui pose probleme, et d'en deduire les exigences pour construire sa propre version.

## Methode

Chaque phase suit le meme schema :
1. **Executer** la fonctionnalite testee
2. **Constater** les resultats (fichiers crees, structure, contenu)
3. **Rediger** un article wiki qui documente le principe, les observations, les limites et les idees

Le vault lui-meme sert de terrain de test : les articles guide sont des pages wiki valides avec frontmatter, wikilinks et tags.

## Vault utilise

- **Vault** : `knlg-repo/` (~48 pages wiki au moment du verdict)
- **Framework** : `vendor/obsidian-wiki/` (clone du repo obsidian-wiki)
- **Demarrage** : 2026-04-14
- **Duree** : ~2 jours d'exploration

## Phases du guide

| Phase | Article | Skill(s) teste(s) | Statut |
|-------|---------|-------------------|--------|
| 0 | [[_raw/s0/2026-04/guide-obsidian-wiki/00-setup\|Setup & Configuration]] | `wiki-setup` | Fait |
| 1 | [[_raw/s0/2026-04/guide-obsidian-wiki/01-ingest-documents\|Ingest de documents]] | `wiki-ingest` | Fait |
| 2 | [[_raw/s0/2026-04/guide-obsidian-wiki/02-history-ingest\|Ingest historique Claude]] | `claude-history-ingest`, `wiki-history-ingest` | Fait |
| 3 | [[_raw/s0/2026-04/guide-obsidian-wiki/03-data-ingest\|Data ingest (formats libres)]] | `data-ingest` | Fait |
| 4 | [[_raw/s0/2026-04/guide-obsidian-wiki/04-query\|Query & Recherche]] | `wiki-query` | Fait |
| 5 | [[_raw/s0/2026-04/guide-obsidian-wiki/05-status-lint\|Status & Lint]] | `wiki-status`, `wiki-lint` | Fait |
| 6 | [[_raw/s0/2026-04/guide-obsidian-wiki/06-crosslinker-tags\|Cross-Linker & Tag Taxonomy]] | `cross-linker`, `tag-taxonomy` | Fait |
| 7 | [[_raw/s0/2026-04/guide-obsidian-wiki/07-wiki-update\|Wiki Update (Cross-Project)]] | `wiki-update` | Fait |
| 8 | [[_raw/s0/2026-04/guide-obsidian-wiki/08-export\|Export du graphe]] | `wiki-export` | Fait |
| 9 | [[09-rebuild-archive\|Rebuild & Archive]] | `wiki-rebuild` | Fait |
| 10 | [[10-skill-creator\|Skill Creator (skippee)]] | `skill-creator` | Skip |
| 11 | [[11-verdict\|Synthese & Verdict]] | -- | Fait |

## Navigation rapide par theme

### Ingestion de connaissances
- [[_raw/s0/2026-04/guide-obsidian-wiki/01-ingest-documents]] — Documents structures vers pages wiki
- [[_raw/s0/2026-04/guide-obsidian-wiki/02-history-ingest]] — Minage de l'historique Claude Code
- [[_raw/s0/2026-04/guide-obsidian-wiki/03-data-ingest]] — Formats libres (transcripts, exports JSON)

### Consultation et recherche
- [[_raw/s0/2026-04/guide-obsidian-wiki/04-query]] — Retrieval gradue dans le vault

### Maintenance et qualite
- [[_raw/s0/2026-04/guide-obsidian-wiki/05-status-lint]] — Inventaire, audit, detection de problemes
- [[_raw/s0/2026-04/guide-obsidian-wiki/06-crosslinker-tags]] — Liens automatiques et normalisation des tags
- [[09-rebuild-archive]] — Snapshot et restauration

### Interoperabilite
- [[_raw/s0/2026-04/guide-obsidian-wiki/07-wiki-update]] — Synchronisation cross-projet
- [[_raw/s0/2026-04/guide-obsidian-wiki/08-export]] — Export du graphe (JSON, GraphML, Cypher, HTML)

### Meta
- [[_raw/s0/2026-04/guide-obsidian-wiki/00-setup]] — Configuration initiale
- [[10-skill-creator]] — Skill generique Anthropic (hors scope)
- [[11-verdict]] — Grille d'evaluation, principes retenus, verdict

---

*Ce guide fait partie du vault [[projects/my-obsidian-wiki/my-obsidian-wiki|my-obsidian-wiki]].*

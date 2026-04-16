---
title: "Phase 10 — Skill Creator (skippee)"
tags: [guide, obsidian-wiki, skill-creator, anthropic, skip]
created: 2026-04-15
phase: 10
---

# Phase 10 — Skill Creator (skippee)

Prerequis : avoir lu [[09-rebuild-archive|Phase 9 — Rebuild & Archive]].

## Constat

Le skill `skill-creator` present dans le vendor obsidian-wiki (`.skills/skill-creator/`) est une **copie exacte** du skill generique d'Anthropic pour Claude Code. Il s'agit du skill officiel publie par Anthropic dans leur repo de skills :

> Source originale : <https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md>

Le fichier `SKILL.md` du vendor fait 485 lignes et reproduit mot pour mot le contenu du repo Anthropic. Les scripts et agents associes (`run_eval.py`, `run_loop.py`, `analyzer.md`, `comparator.md`, `grader.md`, etc.) font egalement partie du kit standard Anthropic.

## Pourquoi cette phase est skippee

Le skill-creator n'est **pas une fonctionnalite propre a obsidian-wiki**. C'est un outil generique de Claude Code permettant de creer, evaluer et ameliorer des skills — il fonctionnerait de maniere identique dans n'importe quel projet.

Tester le skill-creator dans le cadre de ce POC n'apporterait aucune information sur les capacites specifiques d'obsidian-wiki. Il n'y a donc :

- Pas de test specifique pour cette phase
- Pas de page wiki generee
- Pas d'evaluation de comportement propre au projet

## Ce qu'il faut retenir

Le fait que obsidian-wiki embarque le skill-creator d'Anthropic est une bonne pratique (il permet de creer de nouveaux skills pour le projet), mais ce n'est pas un differenciateur. L'evaluation du POC se concentre sur les skills **propres** au wiki : ingest, query, lint, crosslinker, export, rebuild.

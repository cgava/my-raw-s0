---
title: "Phase 7 — Wiki Update (Cross-Project Sync)"
tags: [guide, obsidian-wiki, wiki-update, cross-project, knowledge-management]
created: 2026-04-15
phase: 7
---

# Phase 7 — Wiki Update (Cross-Project Sync)

Ce guide couvre le skill **wiki-update**, qui permet de synchroniser les connaissances d'un projet externe vers le wiki Obsidian. C'est le skill "cross-project" par excellence : il fonctionne depuis n'importe quel repertoire de projet et pousse les connaissances distillees dans le vault.

Prerequis : avoir lu [[_raw/s0/2026-04/guide-obsidian-wiki/06-crosslinker-tags|Phase 6 — Cross-linker & Tag Taxonomy]] et disposer d'un vault avec un `.manifest.json` fonctionnel.

## 1. Principe du wiki-update

Le wiki-update repond a la question de Karpathy : **"Qu'est-ce que je voudrais savoir sur ce projet si j'y revenais dans 3 mois sans aucun contexte ?"**

### Ce qu'il distille

- Decisions d'architecture et *pourquoi* elles ont ete prises
- Patterns decouverts pendant la construction
- Dependances, services, APIs et comment ils s'articulent
- Abstractions cles et modele mental
- Trade-offs evalues et choix effectues

### Ce qu'il ne distille PAS

- Listings de fichiers, boilerplate, config evidente
- Bug fixes individuels sans lecon generale
- Versions de dependances, lock files
- Details d'implementation lisibles directement dans le code

**Heuristique : si lire le codebase repond a la question, ne pas la wikifier. Si il faudrait relire 20 commits de git blame pour reconstituer le raisonnement, la wikifier.**

### Protocole en 6 etapes

1. **Config** : lire `~/.obsidian-wiki/config` pour localiser le vault
2. **Scan projet** : README, source, git log, package metadata, fichiers Claude
3. **Delta** : verifier `.manifest.json` -- premiere fois = full scan, sinon `git log <last_commit>..HEAD`
4. **Distillation** : creer/mettre a jour les pages dans `projects/<nom>/`
5. **Cross-link** : ajouter des `[[wikilinks]]` bidirectionnels
6. **Tracking** : mettre a jour `.manifest.json`, `index.md`, `log.md`

## 2. Execution : sync de my-obsidian-wiki

Le projet cible est `my-obsidian-wiki` lui-meme -- un cas meta interessant puisqu'il est a la fois le sujet et l'hebergeur du wiki.

### Scan du projet

Informations extraites :

| Source | Donnees |
|---|---|
| `README.md` | "POC for obsidian-wiki implementation" |
| Structure | 3 repertoires : root, `knlg-repo/` (vault), `vendor/` (framework) |
| Git log | 18 commits, convention `feat(wiki)/feat(guide)`, 7 phases |
| Vault | ~36 pages, 6 categories, 6 projets deja synces |

### Page projet creee

Fichier : `projects/my-obsidian-wiki/my-obsidian-wiki.md`

**Frontmatter :**

```yaml
title: >-
  My Obsidian Wiki
category: projects
tags: [obsidian, knowledge-management, pkm, ai-agents]
sources: [/home/omc/workspace/my-obsidian-wiki]
summary: >-
  POC repo testing the obsidian-wiki skill framework end-to-end,
  doubling as a real knowledge base built incrementally across 7 phases.
provenance:
  extracted: 0.75
  inferred: 0.20
  ambiguous: 0.05
```

**Contenu principal :**

- Description du dual purpose (test harness + vrai wiki)
- Architecture three-repo (root / knlg-repo / vendor)
- Table des 7 phases avec skills testes
- Decisions cles (guide dans guide/, convention de commit, pas de config globale)
- Profil du contenu (13 concepts, 4 entites, 7 skills, etc.)
- 6 wikilinks vers des pages existantes (Obsidian, Skill-Based Architecture, Knowledge Graph, etc.)

### Tracking mis a jour

- `.manifest.json` : nouveau projet `my-obsidian-wiki` avec `last_commit_synced: 1076e11...`
- `index.md` : nouvelle entree dans la section Projects
- `log.md` : entree `WIKI_UPDATE project=my-obsidian-wiki pages_created=1`

## 3. Test delta (re-run)

Le mecanisme de delta est la cle de l'efficacite du wiki-update sur les executions repetees.

### Comment ca fonctionne

1. Le skill lit `.manifest.json` et cherche l'entree du projet
2. Si `last_commit_synced` existe, il execute `git log <commit>..HEAD --oneline`
3. Si le diff est vide (aucun commit depuis le dernier sync), il s'arrete avec un message "Nothing meaningful changed"

### Resultat du re-run

```
$ git log 1076e118..HEAD --oneline
(aucun output -- 0 commits)
```

**Comportement : SKIP complet.** Le skill detecte que HEAD == last_commit_synced et ne modifie rien. C'est le cas ideal -- pas de rewrite inutile, pas de churn dans le vault.

### Quand le delta declenche une mise a jour

Si de nouveaux commits existaient, le skill :
1. Ne lirait que les commits depuis `last_commit_synced`
2. Evaluerait si les changements sont "meaningful" (pas juste des fix typo)
3. Mettrait a jour la page existante (merge, pas ecrasement)
4. Mettrait a jour `last_commit_synced` dans le manifest

## 4. Navigation dans Obsidian

### Page projet

Ouvrir `projects/my-obsidian-wiki/my-obsidian-wiki.md` dans Obsidian :
- La table des phases donne une vue d'ensemble du projet
- Les wikilinks permettent de naviguer vers les concepts connexes
- Les backlinks (panneau droit) montrent les pages qui referent a ce projet

### Graph View

Ouvrir la Graph View (`Ctrl+G`) :
- Le noeud `my-obsidian-wiki` apparait dans le cluster projects
- Il est connecte a `entities/obsidian`, `concepts/skill-based-architecture`, `concepts/knowledge-graph`, etc.
- Les liens vers les projets freres (kiss-claw, oh-my-claudecode) montrent les connexions cross-projet

### Index

La page `index.md` inclut desormais l'entree :
```
- [[projects/my-obsidian-wiki/my-obsidian-wiki|My Obsidian Wiki]]
    -- POC repo testing the obsidian-wiki skill framework end-to-end
```

## 5. Observations et limites

### Points positifs

1. **Le protocole est auto-suffisant** : le skill contient toutes les instructions pour scanner, distiller, ecrire et tracker. Pas de script externe, pas de dependance.
2. **Le delta evite le churn** : sur un re-run sans changement, aucune modification du vault. C'est essentiel pour ne pas polluer l'historique git.
3. **Le frontmatter est riche** : provenance, summary, tags canoniques -- la page est immediatement requetable par wiki-query.
4. **Les wikilinks sont bidirectionnels par design** : la page projet lie vers les concepts, et Obsidian affiche automatiquement les backlinks.

### Limites

1. **Pas de `~/.obsidian-wiki/config`** : le skill s'attend a un fichier de config globale. Dans ce POC, les chemins sont connus directement, mais en usage cross-project reel, il faut que `setup.sh` ait ete execute.
2. **Un seul niveau de profondeur** : le skill cree une page overview mais pas de sous-pages (concepts/, skills/ sous le projet) sauf si le volume le justifie. Pour un petit projet, c'est suffisant.
3. **Delta base sur les commits, pas le contenu** : si le contenu du projet change sans commit (fichiers non trackes), le delta ne les verra pas.
4. **Merge vs overwrite** : sur une mise a jour, le skill est sense "merger" les nouvelles informations. En pratique, le merge est a la discretion de l'agent qui execute -- pas de diff structurel automatique.

### Idees pour une prochaine iteration

- **Auto-detect du type de projet** : adapter la distillation selon le type (lib, app, infra, doc vault)
- **Multi-page pour gros projets** : creer automatiquement des sous-pages concepts/ et skills/ quand le projet depasse un certain seuil
- **Webhook/CI integration** : declencher wiki-update automatiquement sur push vers main
- **Provenance tracking granulaire** : tracer quelle section vient de quel commit
- **Cross-project synthesis** : quand deux projets partagent des patterns, generer une page synthesis automatiquement

## 6. Navigation

- Article precedent : [[_raw/s0/2026-04/guide-obsidian-wiki/06-crosslinker-tags|Phase 6 — Cross-linker & Tag Taxonomy]]

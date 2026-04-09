---
source_url: local://live-test/s0/pipeline-design-principles
ingested_at: 2026-04-09T00:00:00Z
sensitivity: S0
---

# Principes de design du pipeline PKM

Note S0 — test d'isolation strict vis-à-vis de S2.

Les cinq principes directeurs retenus au moment de figer le plan :

1. **Isolation des sensibilités non négociable.** S0 ne lit jamais S2, S2
   n'écrit jamais dans S0. Aucune autre règle ne peut écraser celle-là.
2. **Réutilisation fast-path plutôt que code custom.** On adapte le MVP à
   ce que permettent obsidian-wiki, mcpvault, autoresearch et
   conversation-search. Le custom est réservé aux 4 composants de colle.
3. **Le process est le livrable, pas l'agent.** Playbook humain et agent
   sont des artefacts de premier ordre.
4. **Promotion toujours humainement gatée.** Pas de auto-promotion.
   Chaque bump de version est un git tag explicite, précédé d'un diff
   contre un corpus de référence.
5. **Éphémère, reproductible, archivé.** Chaque run est clone → run →
   commit/push → self-destruct, avec archivage complet des prompts.

L'isolation est vérifiée par un test canary dédié (ADR-004). La promotion
passe obligatoirement par un tag GPG signé incluant le hash du diff.

# Reunion equipe technique -- Mise en place Knowledge Management System

**Date** : 2026-04-10
**Lieu** : Salle Confluence, visio Teams
**Participants** : Marie Dupont (lead dev), Julien Martin (architecte), Sophie Chen (product owner), Ahmed Benali (DevOps), Clara Moreau (QA lead)

---

**Marie** : Bon, on est tous la. L'objectif de cette reunion, c'est de definir notre approche pour le knowledge management system qu'on veut deployer en interne. Sophie, tu veux poser le contexte ?

**Sophie** : Oui. Le constat c'est que notre documentation est fragmentee. On a du savoir dans Confluence, dans des Google Docs, dans des Slack threads, et surtout dans la tete des gens. Quand quelqu'un part en vacances ou quitte l'equipe, on perd du savoir. L'objectif c'est de centraliser tout ca dans un systeme navigable et maintenu.

**Julien** : La question cle c'est le choix de l'outil. J'ai evalue trois options : Confluence qu'on utilise deja mais qui est un cimetiere de pages, Notion qui est plus moderne mais cloud-only, et Obsidian qui est local-first avec un systeme de liens bidirectionnels tres puissant.

**Ahmed** : Obsidian c'est juste du markdown dans un dossier, non ? Ca s'integre comment avec notre CI/CD ?

**Julien** : Exactement, et c'est son point fort. Comme c'est du markdown pur, on peut le versionner dans git, le linter dans la CI, et meme automatiser des verifications de coherence. J'ai vu des equipes qui font du "documentation as code" avec Obsidian et ca marche tres bien.

**Marie** : Le probleme avec Confluence c'est qu'on a 2000 pages dont personne ne sait lesquelles sont a jour. Si on migre vers Obsidian, on ne veut pas reproduire ce pattern.

**Sophie** : D'ou l'importance d'un systeme de maintenance. Pas juste un outil de stockage, mais un workflow de curation.

**Clara** : Je suis d'accord. Et du point de vue QA, je voudrais qu'on puisse verifier la fraicheur des pages. Un systeme qui flagge les pages pas mises a jour depuis 6 mois par exemple.

**Decision 1** : On choisit Obsidian comme outil principal pour le knowledge management. Raisons : local-first, markdown pur (git-friendly), liens bidirectionnels, ecosysteme de plugins.

**Julien** : Pour la structure, je propose une taxonomie par type de contenu : concepts pour les idees et patterns, skills pour les procedures et how-to, entities pour les outils et frameworks, et references pour les specs et APIs.

**Marie** : C'est inspire du Zettelkasten ca ?

**Julien** : En partie oui. Le Zettelkasten insiste sur les notes atomiques et les liens entre elles. Notre approche sera moins granulaire -- on vise des pages wiki compilees plutot que des notes atomiques -- mais le principe de liaison est le meme.

**Ahmed** : OK pour la structure. Pour le deploiement, je vois deux phases. Phase 1 : migration du contenu existant depuis Confluence. Phase 2 : mise en place du workflow de maintenance avec un bot qui verifie la coherence et la fraicheur.

**Decision 2** : Structure du vault en categories (concepts, skills, entities, references, synthesis). Chaque page aura un frontmatter YAML obligatoire avec titre, categorie, tags, sources, dates.

**Sophie** : Qui sera responsable de la curation ? On ne peut pas compter sur la bonne volonte de tout le monde.

**Marie** : Je propose un systeme de rotation. Chaque sprint, un dev est "wiki gardien" pendant une semaine. Sa responsabilite c'est de reviewer les nouvelles pages, verifier les liens casses, et merger les PR de documentation.

**Clara** : Et cote automatisation ? On pourrait utiliser des LLMs pour aider a la maintenance. Par exemple, detecter les pages redondantes ou suggerer des liens manquants.

**Julien** : Oui, c'est prevu en Phase 2. L'idee c'est d'avoir un agent qui fait du linting semantique -- pas juste verifier que les liens existent, mais verifier que le contenu est coherent et a jour.

**Decision 3** : Deploiement en deux phases. Phase 1 (sprints 15-17) : migration Confluence et setup Obsidian. Phase 2 (sprints 18-20) : automatisation avec LLM-assisted linting et maintenance.

**Action items** :
- [ ] Julien : creer le repo git avec la structure de base du vault (avant vendredi)
- [ ] Ahmed : configurer le pipeline CI pour le linting markdown (sprint 15)
- [ ] Marie : rediger le guide de contribution pour l'equipe (sprint 15)
- [ ] Clara : definir les metriques de qualite du wiki (sprint 15)
- [ ] Sophie : prioriser les 50 pages Confluence a migrer en premier (sprint 15)

**Marie** : Parfait, on a un plan. On se revoit au standup de mercredi pour le point d'avancement. Merci a tous.

---

*Transcript genere par Teams, relu et corrige par Marie Dupont.*

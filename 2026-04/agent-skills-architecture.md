# Skill-Based Architecture for AI Agents

As AI agents grow beyond simple chatbot interactions, a recurring architectural pattern has emerged: the skill-based architecture. Instead of encoding all capabilities in a single monolithic prompt, the agent's abilities are decomposed into discrete, composable skills -- each defined by its own instruction set, trigger conditions, and operational scope.

## What Is a Skill?

A skill is a self-contained unit of agent capability, typically expressed as a markdown file containing structured instructions. Each skill defines: a name and description, trigger conditions (when should this skill activate), preconditions (what state or context is required), step-by-step instructions the agent follows, and output expectations (what the skill produces).

Skills are not code -- they are declarative specifications that an LLM interprets and executes. This makes them accessible to non-programmers and version-controllable alongside documentation.

## Skill Routing

When a user issues a request, a routing layer determines which skill to activate. This can be done via keyword matching (simple but brittle), intent classification (an LLM classifies the request), or explicit invocation (the user names the skill directly, e.g., `/wiki-ingest`).

Sophisticated systems combine these approaches. A primary classifier routes to a skill family, and the skill itself handles disambiguation within its domain. This mirrors how microservice architectures use API gateways to route requests to the appropriate service.

## Composition and Orchestration

Skills become powerful when they compose. An orchestrator skill can invoke sub-skills in sequence or in parallel, passing context between them. For example, a "research" meta-skill might invoke: a search skill to find relevant documents, an analysis skill to extract key findings, and a synthesis skill to produce a final report.

This pattern -- orchestrator coordinating specialized workers -- is the same architecture used in multi-agent systems like AutoGen, CrewAI, and Claude Code's own agent delegation model. The difference is that skills are lighter weight: no separate processes, no inter-agent communication protocol, just structured markdown instructions interpreted by the same LLM.

## Advantages Over Monolithic Prompts

Skill-based architectures solve several problems. Maintainability: each skill is independently authored, tested, and versioned. Scope control: skills define what the agent can and cannot do in a given context, reducing hallucination and off-topic drift. Reusability: skills can be shared across projects. Transparency: users can read the skill file to understand exactly what the agent will do.

## The Obsidian-Wiki Example

The obsidian-wiki project is a concrete implementation of this pattern. Skills like wiki-ingest, wiki-query, and wiki-lint are each defined in their own SKILL.md file. The LLM reads the appropriate skill file when triggered and follows its instructions. No custom code, no API endpoints -- just markdown that the agent interprets as operational instructions.

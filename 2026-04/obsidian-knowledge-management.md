# Obsidian as a Knowledge Management Tool

Obsidian is a markdown-based personal knowledge management (PKM) application that stores notes as plain-text files on the user's local filesystem. Unlike cloud-based tools like Notion or Roam Research, Obsidian gives users full ownership of their data -- notes are just .md files in a folder, readable by any text editor.

## The Graph Model

Obsidian's core differentiator is its emphasis on bidirectional linking via `[[wikilinks]]`. When you create a link from Note A to Note B, Obsidian automatically tracks the backlink from B to A. This turns a folder of notes into a knowledge graph, where ideas are connected rather than siloed in a hierarchy.

The graph view visualizes these connections, revealing clusters of related ideas and orphaned notes that need integration. Power users often use the graph as a diagnostic tool -- a well-connected graph indicates healthy knowledge management, while isolated clusters suggest missing cross-references.

## Zettelkasten and Evergreen Notes

Obsidian is frequently used to implement the Zettelkasten method, a note-taking system developed by sociologist Niklas Luhmann. The core principle is atomicity: each note captures one idea and links to related notes. Over time, the network of connections becomes more valuable than any individual note.

Andy Matuschak's concept of "evergreen notes" extends this: notes should be written to be durable and composable, evolving over time rather than being write-once artifacts. This contrasts with traditional note-taking where notes are tied to a specific source or lecture and rarely revisited.

## Plugins and Extensibility

Obsidian's plugin ecosystem is one of its strengths. Community plugins extend the core application with features like Dataview (SQL-like queries over frontmatter), Templater (dynamic templates with JavaScript), Canvas (spatial note arrangement), and hundreds of others.

The frontmatter system (YAML metadata at the top of each note) enables structured data within unstructured notes. Tags, aliases, dates, and custom fields can be queried across the vault, turning a folder of markdown into a lightweight database.

## Limitations

Obsidian's local-first architecture means real-time collaboration is difficult. Obsidian Sync provides cross-device synchronization, but multi-user editing requires workarounds. The learning curve is also steeper than tools like Notion -- Obsidian rewards investment but requires users to design their own organizational systems rather than providing opinionated defaults.

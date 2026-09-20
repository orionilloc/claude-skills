---
name: anki-card-generator
description: Generate Anki flashcards from source material the user provides (chapter text, notes, an uploaded file, OCR'd handwritten notes, a URL, pasted text) for any subject — programming, language learning, music theory, anything. Use this whenever the user says "make anki cards for X" / "make a deck for X" / "flashcards for this," or asks to quiz or drill themselves on specific source material. Always outputs plain CSV.
---

# Anki Card Generator

Turns source material into an Anki-importable CSV deck. Domain-agnostic — works the same way for a Python chapter, a grammar unit, a music theory concept, or anything else the user hands over.

## Before generating

The user always provides real source material — never invent facts to build cards from. Source can be pasted text, an uploaded file, a URL to fetch, an image of handwritten notes (read it), or content already in the conversation. If no source material is identifiable in the request or recent conversation, ask for it before doing anything else.

Ask clarifying questions only when something is genuinely ambiguous — don't front-load a checklist. In practice this means asking when:
- The scope is unclear (the whole source vs. a specific section/topic within it)
- It's unclear whether cloze or front/back suits the material better (see Format below) and the source doesn't make it obvious
- The user requests a card count well outside the 10–20 default and it's unclear if that's intentional

Otherwise, just generate the deck. Most requests ("make anki cards for chapter 6" with the chapter attached) need zero clarifying questions.

No deduplication against existing decks by default — only cross-check an existing deck if the user provides one and asks for it.

## Card count

Default to 10–20 cards per topic/chapter/unit. Mix broad recognition cards with a few sharper, more specific ones — not all cards need to be equally hard. If the user asks for more or fewer, follow that instead.

## What makes a good card (prioritize this over exhaustive coverage)

The goal is retention of things that are actually load-bearing, not a transcript of the source in blank form. For any domain, prioritize:

- **Distinctions between similar things** that are easy to conflate (mutable vs. immutable; two grammar cases that look alike; two chord qualities that sound close)
- **Mechanisms and "why"** — the underlying reason something behaves the way it does, not just the label for it
- **Things that cause real errors or confusion** when gotten wrong — the kind of mistake that produces a bug, a mistranslation, a wrong note
- **Patterns that generalize**, not one-off trivia that's easily looked up in the moment

Deprioritize (don't fill the deck with):
- Exact syntax/argument order that's trivially re-derivable from an error message, a reference doc, or autocomplete
- Isolated vocabulary or facts with no connective tissue to anything else, unless the material is explicitly vocabulary-heavy (e.g., a language-learning word list, where recall of the word itself *is* the point)

Reference examples of cards that hit this bar, across different domains:
```
Variables don't contain values directly -- they contain [...] to values.,references
Tuples are immutable like strings -- lists are the [...] sequence type.,mutable
Pipelining will often fail if the remote sudoers file has [...] enabled.,requiretty
System information like IP addresses and OS versions discovered by Ansible are called [...].,facts
Wireless networks use [...] to avoid collisions.,CSMA/CA
For security best practice you should restrict [...] to serve as a conduit for only L2 control traffic.,VLAN 1
```
versus lower-value cards that were cut for being pure recall:
```
Inserts a value at a given index -- index argument comes first: [...]().,insert
The flag used to restrict a command to a specific host is [...].,--limit
```
And a vocabulary example, where the word itself is the point rather than a wrapping fact:
```
acción de gracias,Thanksgiving
matutina (adj),morning
```

## Format

**Always plain CSV, comma as field separator, UTF-8, no header row, no HTML.** This is a hard rule, not a default to reconsider per material — even for command-syntax-heavy content where bold/italic could visually distinguish a literal token from a placeholder, keep it plain text.

Two shapes, pick per material:

1. **Cloze-in-context** (`clue text with [...] blank,answer`) — best when the fact only makes sense embedded in a sentence or code/notation snippet. This is the default for programming, sysadmin/networking facts, grammar rules, music theory concepts, anything relational.
2. **Plain front/back** (`front,back`) — best for direct vocabulary/translation pairs, term definitions, or anything where a sentence wrapper adds nothing (e.g., a foreign word and its translation, a symbol and its name). Adding a short disambiguator in parens on the front is fine when the source distinguishes it that way (e.g., `matutina (adj)` when the word could otherwise be read as a different part of speech).

**Multiple blanks in one card:** when a single clue needs two coupled blanks (`In a unique local unicast address, you must append the [...] to [...].`), that's fine — just list the answers in the same left-to-right order as the blanks, comma-separated, as the second field (`global id, FD`). Don't overuse this; most cards should have one blank.

Within either shape:
- Terse. Usually one blank or one fact per card (see multi-blank exception above).
- Answers are the actual term/word/value the source uses — not a paraphrase or a descriptive adjective standing in for it.
- Vary sentence openers — don't start every card with "The" or the same construction.
- Avoid commas inside the clue text itself (breaks the two-column CSV, and conflicts with the multi-blank answer format above); use `--` or a semicolon instead.
- Write plain UTF-8 characters — a real apostrophe, a real space. Never emit HTML entities like `&nbsp;` or `&#x27;`; those are copy-paste artifacts from other sources, not something to reproduce.
- No quoting/escaping games — keep the two columns clean so a plain CSV import works without special handling.

## Output

Write the CSV to the outputs directory and present it as a file — don't paste the raw CSV into the chat as the deliverable. A short note on what the deck covers and the card count is enough; don't re-list every card in prose.

## <%* const language = await tp.system.suggester(["Java","Python","Bash","C","PowerShell","General"], ["Java","Python","Bash","C","PowerShell","General"]); const pyPhases = ["Phase 0 · Setup","Phase 1 · Foundations","Phase 2 · Blue Team","Phase 3 · Red Team","Phase 4 · Cloud"]; const javaStages = ["Stage -1 · Refresher","Stage 0 · Collections","Stage 1 · Foundations","Stage 2 · Linear DS","Stage 3 · Trees & Heaps","Stage 4 · Graphs","Stage 5 · DP & Greedy","Stage 6 · Interview"]; const phaseMap = { "Java": javaStages, "Python": pyPhases, "Bash": pyPhases, "C": pyPhases, "PowerShell": pyPhases }; let phase = "N/A"; if (language !== "General") { phase = await tp.system.suggester(phaseMap[language], phaseMap[language]); } -%>

## title: <% tp.file.title %> date: <% tp.date.now("YYYY-MM-DD") %> language: <% language %> phase: <% phase %> tags: [deep-dive] status: learning links: []

> **One-line summary:** <% tp.file.cursor(1) %>

## Core Idea

## Structure

_(delete if the concept isn't spatial)_

## Analogy

_(a real-world comparison that makes the mechanism click)_

## Mechanics / reference

_(relabel the columns to fit — complexity table, comparison table, CLI flags, whatever the concept needs. Copy this table again below if the concept needs more than one angle.)_

||||
|---|---|---|
||||

## Worked example

_(a concrete walkthrough with real numbers/labels — not just the abstract description)_

## When to use it

## Why it matters for security

_(delete if there's no real security angle — plenty of notes won't have one)_

|Concept|Attacker's perspective|Defender's perspective|
|---|---|---|
||||

## Pitfalls

## Flashcards

- #card

## Questions I still have

- [ ]

## Key terms

|Term|Definition|
|---|---|
|||

## Related

[[]]

---

→ Next: [[]]
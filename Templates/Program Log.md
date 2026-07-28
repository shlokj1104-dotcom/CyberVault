## <%* const language = await tp.system.suggester(["Java","Python","Bash","C","PowerShell","General"], ["Java","Python","Bash","C","PowerShell","General"]); const pyPhases = ["Phase 0 · Setup","Phase 1 · Foundations","Phase 2 · Blue Team","Phase 3 · Red Team","Phase 4 · Cloud"]; const javaStages = ["Stage -1 · Refresher","Stage 0 · Collections","Stage 1 · Foundations","Stage 2 · Linear DS","Stage 3 · Trees & Heaps","Stage 4 · Graphs","Stage 5 · DP & Greedy","Stage 6 · Interview"]; const phaseMap = { "Java": javaStages, "Python": pyPhases, "Bash": pyPhases, "C": pyPhases, "PowerShell": pyPhases }; let phase = "N/A"; if (language !== "General") { phase = await tp.system.suggester(phaseMap[language], phaseMap[language]); } const difficulty = await tp.system.suggester(["Easy","Medium","Hard"], ["Easy","Medium","Hard"]); -%>

## title: <% tp.file.title %> date: <% tp.date.now("YYYY-MM-DD") %> language: <% language %> phase: <% phase %> tags: [problem] status: learning difficulty: <% difficulty %> source: links: []

> **Problem:** <% tp.file.cursor(1) %>

## Approach / pattern

## Complexity

_(relabel if this isn't a DSA problem — e.g. edge cases handled, runtime behavior)_

||Time|Space|
|---|---|---|
|This solution|||

## Code

[[]]

## What went wrong first try

## Key insight

## Related

[[]]
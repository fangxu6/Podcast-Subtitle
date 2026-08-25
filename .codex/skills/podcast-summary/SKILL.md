---
name: podcast-summary
description: Convert one selected transcript-site/episodes/*.html episode into a Chinese summary under summary/*.md, using summary/ep-227.md as the required structure. Use this whenever the user asks to summarize a specified podcast transcript into an Obsidian note.
---

# Podcast Summary

Create exactly one summary note for the HTML file named by the user. Resolve paths from the Podcast-Subtitle repository root:

`transcript-site/episodes/ep-N.html` -> `summary/ep-N.md`

Do not overwrite an existing summary note without permission.

## Required workflow

1. Read `summary/ep-227.md` completely before drafting. Treat its headings, order, metadata labels, Chinese tone, and Obsidian link style as the template.
2. Read the selected HTML completely. Use the page title and fact counters for the title, duration, SRT count, and paragraph count. Use the timeline paragraphs as the only factual source; do not invent details from the episode number or title.
3. Reconstruct the episode as an editor would: identify topic boundaries, arguments, examples, conclusions, and recommendations. A time block may contain more than one paragraph, but a timeline bullet must describe a coherent idea rather than repeat subtitle fragments.
4. Write these sections in this order:

   - `#` episode title
   - source, duration, SRT count, and reading-paragraph count
   - `## 内容摘要` with 2-4 compact paragraphs covering the whole episode
   - `## 时间线` with chronological `### start-end：topic` sections. Use about ten-minute windows, split when the topic changes, and give 2-5 substantive bullets per section.
   - `## 核心观点` with 3-5 numbered takeaways that generalize the discussion without adding new claims
   - `## 相关节目` with only defensible links to existing `summary/*.md` notes
   - `## 关键词` with concise backtick-wrapped terms

5. Normalize obvious speech-to-text errors only when the surrounding transcript makes the intended term clear. Keep names, product names, film titles, mathematical terms, and technical terms accurate; mark uncertainty in the prose instead of guessing.
6. Add Obsidian wikilinks only for real related notes. Link both ways when editing related notes is explicitly in scope; otherwise do not modify other summaries merely to create backlinks.
7. Validate that the output source path, counters, timeline coverage, and section order match the HTML and that the note is valid UTF-8 Markdown.

## Quality bar

The result is a readable editorial summary, not a transcript dump. Prefer a few precise bullets with context and consequences over many shallow bullets. Separate the hosts' reported facts from their opinions when the distinction matters. Avoid spoilers only when the episode itself clearly treats them as spoilers; otherwise summarize the discussion faithfully.

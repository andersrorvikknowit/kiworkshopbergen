# KI Workshop Bergen

Hands-on workshop: building a custom skill that summarizes messy meeting notes.

Everything is hosted on GitHub. Work through the tasks in order.

## Prerequisites

- Codex - https://developers.openai.com/codex/app
- The sample meeting notes (see Task 1)

## Task 1 — Get the files

Sample input data is a set of realistic, messy raw meeting notes (`.txt`) from a fictional software/product team. These are the inputs your skill will summarize.

The files are available here: https://github.com/andersrorvikknowit/kiworkshopbergen/

1. Download the meeting note files to a folder on your disk.
2. Open a couple and skim them. Note the noise: typos, shorthand, async side-comments, decisions buried mid-paragraph, and action items in inconsistent formats. This is what the skill has to handle.

**Done when:** you have all 8 note files locally and have read at least two.

## Task 2 — Work with a directory of files

Goal: experience how Codex can reason across a whole folder of related files, not just answer questions about one document at a time.

Ask Codex to inspect the directory of meeting notes and create a new document that summarizes findings across all the files.

Suggested prompt:

> Summarize insights from these meeting notes. Create a document that summarizes the main findings and lists prioritized suggested improvements for the team.

Things to look for in the result:

- Does Codex connect recurring topics across multiple meetings?
- Does it separate symptoms from root causes?
- Does it turn messy notes into a useful artifact the team could actually use?
- Does it create the document in the folder instead of only replying in chat?

**Done when:** Codex has created a new summary document with cross-meeting findings and prioritized team improvement suggestions.

## Task 3 — Build the skill

Goal: a custom skill that turns any of these raw notes into a clean, structured summary.

> **Note:** Codex does the skill building in this workshop, not Claude. Drive the implementation through Codex.

Suggested target output for the skill — extract and normalize:

- **Summary** — 2–3 sentence overview of the meeting
- **Decisions** — what was decided
- **Action items** — task, owner, due date (normalized format)
- **Blockers** — what's blocking progress
- **Open questions** — unresolved items

Steps:

1. Point Codex at the meeting notes and have it draft the skill (instructions/prompt + any structure files).
2. Test against several notes — the files share continuity (recurring people, the Stripe migration, the loqate incident, the analytics bug), so check whether the skill tracks context correctly.
3. Iterate on the skill instructions until the output is consistent across all 8 files.

**Done when:** the skill produces a consistent, structured summary across all sample files.

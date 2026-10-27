---
name: daily-report
description: Turn rough daily work updates into a concise Microsoft Teams status report, asking follow-up questions to clarify context, outcomes, remaining work, and blockers before finalizing. Use for daily project or team updates rather than general document writing.
---

# Daily Status Report

Convert the user's rough notes into a ready-to-send daily report for their manager on Microsoft Teams. Keep enough context to explain the work and its results without creating long paragraphs. Draft the report in chat; posting it requires an explicit request.

## Clarify before finalizing

Read all supplied notes and any answers already given. For each task, establish:

- The problem, requirement, or goal being addressed.
- What specifically changed or was achieved today, separate from overall task progress.
- The current delivery stage and what remains.
- Any blocker, decision, or support needed.

If material details are missing, ask a small batch of focused questions grouped by project or task. Make questions easy to answer in short bullets. Wait for answers before producing the final report, then ask further questions only for unresolved material gaps. If the notes are already clear, generate the report directly.

Use these cues to identify meaningful gaps:

- For “worked on,” “reviewed,” “followed up,” or “attended a discussion,” ask what was completed, found, agreed, clarified, or unblocked.
- For work in progress, clarify which part is complete and what remains; a percentage alone is insufficient.
- For completed work, distinguish implementation, review, testing, deployment, and production confirmation where relevant.
- For blockers, clarify the delivery impact and who needs to take what action. Ask for an ETA when useful, without demanding an invented date.
- If AI usage is required or included, clarify how it was used where missing. Preserve supplied percentages; do not estimate them.
- Clarify the report date if it is missing and cannot be reliably established from the conversation. Do not silently replace an explicitly supplied date with today's date.

Do not repeat questions, request information already supplied, or ask for details that would not improve the report. If the user does not know an answer, accept that and mark material uncertainty briefly. If the user explicitly requests a final report with available information, proceed with those limitations visible.

## Accuracy

- Never invent outcomes, benefits, test results, decisions, dates, owners, or completion percentages.
- Separate intended benefits from verified results. Do not claim a performance improvement or business impact solely because a change was implemented.
- Do not label development as fully done when testing or validation remains. Use a precise status such as “Development complete; testing in progress.”
- Preserve task IDs, project names, and technical details needed to understand the work.
- Describe partial progress honestly. A review, finding, or decision can be a useful outcome without implying a completed deliverable.
- Resolve contradictory statuses through questions, or state the uncertainty if the user cannot clarify.

## Final report

Use this structure unless the user requests another format:

```text
Daily Update – [date]

[Project name]
- [Task ID/name] — [Status; completion % if supplied]
  [One short sentence combining relevant context and today's concrete progress or outcome.]
  Next: [Remaining action; ETA if known.] Blocker: [Only if applicable.]
```

Replace bracketed fields with supplied information; omit optional fields when unavailable or irrelevant. Keep each task to 2–3 short lines, allowing normal screen wrapping. Add a brief AI usage note when reporting instructions require it, or the notes include it. Omit “Next” for completed tasks unless a follow-up is needed.

Group tasks under project headings. Include General/Internal and Learning only when supplied. For learning, include the topic, time spent, and takeaway when available; if no learning was done, use one short line.

Use plain, professional language. Avoid long paragraphs, generic claims, repetitive labels, and a summary that repeats the report. Before finalizing, check that each task makes today's progress and current state clear. Once clarification is complete, return only the ready-to-send report.

---
name: case-brief
description: Brief one Indian judgment with Vitark MCP - facts, issues, holding and reasoning, with short verbatim quotes located in the judgment text, plus Vitark's stored later-treatment label and citing cases. Use when the user names a case, citation or CNR and asks for a case brief, case note, summary of the judgment, its ratio or holding, or how it has been treated since.
---

# Case Brief

Use Vitark MCP to brief one judgment from its own text. Every statement of fact, issue or holding comes from the judgment text Vitark returns, with quotes the user can find in it. Treat the result as research evidence, not legal advice.

## First step: say the number of paid calls

Before you call any Vitark tool other than `get_credit_balance`, tell the user in one short sentence how many paid Vitark calls you expect to make (for example "This will take about 3 paid Vitark calls."). Then go on without waiting for a reply, unless a rule below says to ask first. When the run ends, state once how many paid calls were actually made.

## Before the first call

- If the user names more than one case, ask which one to brief first.
- If the user also asks about a section number without naming the Act, ask which Act they mean before any statute call.
- First run: Claude asks for approval the first time it uses each Vitark tool (Deny, Always allow, Allow once). Tell the user that Always allow stops the prompt for that tool; otherwise each later call asks again.
- Counting paid calls for the first step: Each call below is a paid call that spends credits from the user's Vitark balance; `get_credit_balance` is free. Do not state a price per call; the price Vitark shows at use time controls. The calls: one `vitark_case_find` call to identify the case, one `vitark_case_text` call per 50,000-character slice of the judgment, and one `vitark_case_citations` call for later treatment. A short judgment is about three paid calls; a long one adds one paid call for each further slice. The length is known only while reading: if the text is still `truncated` after four slices, say so and ask before reading on.

## Steps

1. **Identify the case.** Make one `vitark_case_find` call per case. If the user gives a docket number, neutral citation or CNR, pass it as `case_number` (with the `name`), since that resolves directly; otherwise pass the party or case name alone as `name`, without filler. An SCC or AIR report citation is not a `case_number`: the tool rejects it, so search by name instead. If the first result holds a candidate whose parties and court fit, use it and say it is a candidate; do not narrow. Narrow only when the first result has too many matches and none fits: one call, adding `court` first. Pass `year` only when the user states the decision year as such; the year in an SCC or AIR citation is the report's year, not the decision's. A narrowing call only adds information: if it returns nothing or fewer rows, go back to the first result. Never say Vitark does not hold a case on the strength of a narrowed call. Brief without comment a case whose `match_confidence` is `exact` or `high` and whose court and date agree with what the user named. If the match is `match_confidence` `candidate` (or `match_kind` `partial`), tell the user it is a candidate, not an exact match, with the parties, court, decision date, citation and CNR the result returns. With several candidates and none or more than one fitting the parties and court, list them and ask which one before reading. With one candidate, go on, and say in the brief's **Case** and **Limits** sections that it is a candidate match. Never treat a candidate as confirmed because you recognise the case. If the first lookup found nothing that fits, say "not found in Vitark", never that the case does not exist.
2. **Read the text.** Call `vitark_case_text` with the CNR. Continue with `next_offset` while `truncated` is true. Brief only what you have read; if you stop early, say which part of the judgment was not read.
3. **Later treatment.** Call `vitark_case_citations` with the CNR.

## Write the brief

Use these headings, in this order:

- **Case.** Title, court, bench or judges, decision date, citation or case number, and CNR, from the returned metadata. Leave out what the result does not carry.
- **Facts.** The facts the court records, in a few sentences.
- **Issues.** The questions the court framed or decided, in the court's terms where it states them.
- **Holding.** What the court decided on each issue, and the order it made.
- **Reasoning.** The steps that carry the holding, separating the court's words from your summary.
- **Quotes.** Two to five short passages, verbatim. Locate each one by the paragraph number printed in the text when the text has one; otherwise by its `offset` slice and a few words of context. Never invent a paragraph number or a reporter page.
- **Later treatment.** Vitark's stored `treatment` label from the case row's `extras`, reported as Vitark's label; report `not_checked` as "later treatment not checked". From `vitark_case_citations`, reading each row once, list the citing candidates with their `treatment`, marking a candidate confirmed only when `citation_matched` or `reverse_edge_resolved` is true, and keep the `entity_coverage` caveat. No label is a finding that the case is still good law.
- **Limits.** What was not read, what the index does not cover, and anything the user should verify in the official report.

Keep the brief to what the judgment says. Do not add holdings, dissents, later history or statutory changes the returned material does not show; if the judgment applies a provision that has since been repealed or renumbered, say only that the user may wish to check the current provision, and offer the old-and-new-codes skill. Do not promote a purchase or account upgrade.

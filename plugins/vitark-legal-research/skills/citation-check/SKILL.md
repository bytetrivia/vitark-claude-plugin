---
name: citation-check
description: Check the Indian case citations in a draft, memo, plaint, petition or list against Vitark MCP. Use when the user asks to verify, check or audit citations, find out whether cited cases are real or correctly named, confirm that a quoted passage appears in the judgment, or see Vitark's stored later-treatment label for each cited case.
---

# Citation Check

Use Vitark MCP to check each case citation the user gives: find the case, confirm that Vitark holds it, check a quoted passage against the judgment text when the user supplies one, and report Vitark's stored later-treatment label. Treat the result as research evidence, not legal advice.

## First step: say the number of paid calls

Before you call any Vitark tool other than `get_credit_balance`, tell the user in one short sentence how many paid Vitark calls you expect to make (for example "This will take about 3 paid Vitark calls."). Then go on without waiting for a reply, except that more than 10 citations needs the user's answer first (see below). When the run ends, state once how many paid calls were actually made.

## Before the first call

- Extract the citations from the user's text and show the list you will check. If a citation is a section or an Act, it is not a case; leave it for the legal-research skill, and if it gives a section number without naming the Act, ask which Act first.
- First run: Claude asks for approval the first time it uses each Vitark tool (Deny, Always allow, Allow once). Tell the user that Always allow stops the prompt for that tool; otherwise each later call asks again.
- Counting paid calls for the first step: Each call below is a paid call that spends credits from the user's Vitark balance; `get_credit_balance` is free. Do not state a price per call; the price Vitark shows at use time controls. The calls: one `vitark_case_find` call per citation to find it, one `vitark_case_text` call per 50,000-character slice read to check a quote, and one `vitark_case_citations` call per case when the user wants later citing cases. A simple existence check is one paid call a citation.
- If the list has more than 10 citations, say how many there are and how many paid calls checking them would make, and ask whether to check all of them or which ones first. Do not start until the user answers.

## Check each citation

1. **Find it.** Make one `vitark_case_find` call per citation. If the citation gives a docket number, neutral citation or CNR, pass it as `case_number` (with the `name`), since that resolves directly; otherwise pass the party or case name alone as `name`, without filler. An SCC or AIR report citation is not a `case_number`: the tool rejects it, so search by name instead. Add `court` or `year` only when the first result returns too many matches, and then make at most one narrowing call; never call again just to confirm a match.
2. **Decide what was found.** Read `match_count`, `match_kind`, `match_confidence`, `match_definitive` and `notes`.
   - `match_confidence` `exact` or `high`, with court, year and parties that agree with the citation: found.
   - `match_confidence` `candidate` (or `match_kind` `partial`): tell the user the match is a candidate, not an exact match, with the parties, court, decision date, citation and CNR the result returns. With several candidates, list them and say the citation could not be pinned to one case; do not pick one. Never treat a candidate as found because you recognise the case.
   - No match: say "not found in Vitark". If `match_definitive` is false, add that Vitark's lookup did not cover everything. Never write "does not exist", "fake" or "fabricated".
   - An error, `capability_unavailable` or `named_lookup_failed`: say the check could not run for this citation. Do not retry automatically.
3. **Check details.** Compare what the draft states (party names, court, year, citation) only with the fields the result returns (title or parties, court, decision date or year, citation, CNR). Report each difference as a mismatch, and each part of the citation the result does not carry (for example an SCC volume and page) as "not in Vitark's record". Never say a citation is correct, standard or matches from your own knowledge.
4. **Check a quote, only if the draft quotes the judgment.** Call `vitark_case_text` with the CNR, and read on with `next_offset` while `truncated` is true until the passage is found or the text ends. Report the passage as found verbatim, found with differences (show both), or not found in the text Vitark holds. Each further slice is another paid call, and the length is known only while reading: if the passage is not found and the text is still `truncated` after four slices, say so and ask before reading on.
5. **Report treatment.** Report `treatment` from the found row's `extras` as Vitark's stored label: followed, relied_on, distinguished, overruled, referred, doubted or unknown. Report `not_checked` as "later treatment not checked". No label is a finding that the case is still good law. If the user asks which later cases cite it, call `vitark_case_citations` with the CNR, list `citing_candidates` with their `treatment` (read each row once; list only as many as the user needs), and mark a candidate confirmed only when `citation_matched` or `reverse_edge_resolved` is true. Preserve the `entity_coverage` caveat: the citation index covers part of the corpus.

## Write the report

Give a table with one row per citation: the citation as written, result (found, candidates only, not found in Vitark, check failed), matched case (title, court, decision date, CNR), differences, quote check, and treatment label. Under the table, list the points the user must look at, most serious first. End with what was not checked and why (for example a citation outside the list the user approved, or an outage).

Do not change the user's draft unless asked. Do not state the law a case stands for unless the judgment text was read. For a filing, tell the user to confirm each citation against the official report before relying on it. Do not promote a purchase or account upgrade.

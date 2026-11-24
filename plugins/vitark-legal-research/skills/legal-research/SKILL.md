---
name: legal-research
description: Research Indian judgments, tribunal decisions, statutory text, and later judicial treatment with Vitark MCP; use for issue-based authority finding, named-case lookup, reading a section of an Indian Act, source verification, and citation-grounded legal research. For mapping a section between the old and new codes, checking a list of citations, or briefing one judgment, prefer the old-and-new-codes, citation-check, or case-brief skill.
---

# Vitark Legal Research

Use Vitark MCP for Indian legal research. Treat retrieved material as research evidence, not legal advice, and state material source and coverage limits. Legal rules come from the tool results, not from memory.

## Cost

Every research tool is a paid call: each call spends credits from the signed-in user's Vitark balance (an exact section lookup is one `vitark_statute_search` call; each further `vitark_case_text` slice is another call). `get_credit_balance` is free. Do not state a price per call; the price Vitark shows at use time controls. If a workflow needs more than about five paid calls, ask whether to go ahead after giving the number.

First run: Claude asks for approval the first time it uses each Vitark tool (Deny, Always allow, Allow once), so a first question can raise two or three prompts. Tell the user that Always allow stops the prompt for that tool; Allow once asks again next time.

## First step: say the number of paid calls

Before you call any Vitark tool other than `get_credit_balance`, tell the user in one short sentence how many paid Vitark calls you expect to make (for example "This will take about 3 paid Vitark calls."). Then go on without waiting for a reply, unless a rule below says to ask first. When the run ends, state once how many paid calls were actually made.

## Choose the tool

- Use `vitark_search` for issue-based research. Pass the user's legal question verbatim; the tool performs query planning, retrieval, deduplication, reranking, and enrichment internally. Use `scope="cases"`, `scope="statutes"`, or `scope="all"`. Put a bench or case type in the query text because those are not filters. Add `courts`, date or year bounds, statute-year bounds, `ministry`, or `limit` only when the request supplies or clearly requires them.
- Use `vitark_statute_search` for focused statutory research and exact Act-and-section retrieval. When a prior result supplies `act_id`, pass it back with `section_number`; do not guess `act_slug`. Quote `section_text`, never the clipped `snippet`, and treat the section as complete only when `section_text_complete` is true.
- Use `vitark_case_find` for a party or case name, docket number, neutral citation, or CNR. Pass either `name` or `case_number`. A partial name match is only a candidate until the metadata or text confirms the same matter.
- Use `vitark_case_text` when the judgment's reasoning or exact words matter. It accepts a CNR and can also resolve an unambiguous neutral citation or docket number. Continue long judgments with the returned `next_offset` while `truncated` is true.
- Use `vitark_similar_cases` after identifying a strong seed CNR. Set `group_by_case=true` when the user needs distinct judgments rather than passage-level results.
- Use `vitark_entity_suggest` to resolve a judge, party, advocate, Act, or place to the canonical value used by the entity index. That value is a `norm` for `vitark_cases_by`; it is not a statute `act_id`.
- Use `vitark_cases_by` for metadata lists by judge, party, advocate, Act and section, court, year, category, disposition, case type, or case number. Prefer a norm returned by `vitark_entity_suggest`. Always pair `section` or `act_year` with `act` or `act_norm`. This index covers only the entity-extracted subset and is topic-blind.
- Use `vitark_case_citations` for Acts and cases cited by a known judgment and for approximate later citing candidates. It requires a CNR returned by Vitark.
- Use `get_credit_balance` when the user asks about credits, a call is declined, or a planned workflow would make many paid calls. It is free.

The callable surface has exactly these nine tools. Do not call a name absent from the connected server's tool catalog.

## Read result envelopes

Tool results arrive as JSON text. Parse the JSON object before assessing it; do not expect structured-content fields or an output schema.

For search envelopes, distinguish `results` (cases) from `statute_results` (statutes). Check `status`, `source_outcomes`, `resolved_scope`, `notes`, `empty_corpora`, and `unresolved_courts`. `resolved_scope.legs.partial_detail` identifies incomplete sources and gives human-readable reasons. A partial or failed source, an unresolved court, a searched corpus with no matches, and an ordinary zero-result query are different outcomes; report the one the response actually states.

Read `support_basis` before interpreting `support_level`. In case search it grades rerank relevance, not legal entailment. In focused statute search it grades Act-and-section identity, not whether the provision answers the user's question. Review `statutory_provisions` before semantic statute rows, then follow `resolved_scope.statute_reading_order`; duplicate statute mirrors may have been collapsed. If statute rows were not reranked, do not compare producer vector ranks across collections.

If the user gives a section number without identifying the Act or regulation, ask which enactment they mean before any tool call. Do not guess an enactment or quote a same-numbered provision. For an exact statute read, re-request an incomplete semantic row with its returned `act_id` and `section_number`. Quote it as complete only when `section_text_complete` is true; otherwise label the retrieved text partial and do not present it as the complete provision. Do not assert a corpus gap unless the current response reports it.

`vitark_cases_by`, `vitark_entity_suggest`, and `vitark_case_citations` rely on a partial entity-extraction index. Preserve `entity_coverage` caveats. In citation results, `acts_referred` and `cases_referred` are cited by the seed judgment; `citing_candidates` point in the reverse direction and can include title collisions. Treat a candidate as confirmed only when its citation edge is resolved. `treatment="not_checked"` means later treatment was not examined, not that the authority remains good law.

Do not convert an error or unavailable capability into a negative legal finding. Fix and retry an explicit caller error. Do not automatically retry outages or insufficient-credit responses, because a repeated paid call may debit again. Empty results and errors are not charged by the current service behavior, but the price and consent shown by Vitark at use time control.

## Read statute rows

Statute rows for the old and new codes (Indian Penal Code and Bharatiya Nyaya Sanhita; Code of Criminal Procedure and Bharatiya Nagarik Suraksha Sanhita; Indian Evidence Act and Bharatiya Sakshya Adhiniyam; Income-tax Act, 1961 and Income-tax Act, 2025) carry Vitark's own material. Read it as such.

- **The bracketed note.** `section_text` (and `snippet`) of such a row starts with a paragraph in square brackets: on an old-code row a repeal statement with the corresponding provisions ("[Repealed by section ... Corresponding provision: Section 103, Bharatiya Nyaya Sanhita, 2023.]"), on a new-code row an "in force from" statement with the old sections ("[In force from ... Corresponds to Section 302, Indian Penal Code, 1860.]"). Vitark writes that note; it is not the Act's text. Never quote it as the words of the Act, and attribute anything taken from it to Vitark's note.
- **Row layout.** After the note come the Act title, the section heading, sometimes a chapter line, and on some rows a line starting "Source:" that records the edition and a checksum (also Vitark's; Bharatiya Nyaya Sanhita and Bharatiya Nagarik Suraksha Sanhita rows seen on 24 Nov 2026 had none). The Act's text follows. Quote the Act from the text after the "Source:" line, or, on a row without one, after the section heading and any chapter line. A "State amendments (not part of the central text ...)" block lists State amendments printed in the source edition: it is not the central Act's text, so quote it only as that State's amendment and name the State. A closing "Notes (India Code footnotes ...)" block holds India Code's footnotes; cite them as footnotes.
- **Paired rows.** After an exact old-code row, the result adds the new-code row or rows that correspond to it. Such a row carries `successor_of` (the `source_id` of the old row it follows), `successor_effective_from`, `successor_source` (`title`, `url`, `note` of the correspondence table) and `regime_info` (`relationship`, `from_source_id`, `effective_from`, `applicability_requires_facts`). One old section can have several new rows, and some have none. These fields appear only when the old-code section is looked up: an exact lookup of a new-code section returns that row alone, without them, and its old section appears only in the bracketed note.
- **Very large results.** A row can be tens of thousands of characters (a definitions section or a schedule), and a lookup that pairs several such rows can run to hundreds of thousands of characters, because each row's text appears in both `section_text` and `snippet`, and in both `results` and `statute_results`. Read each row once, from `results`. Quote only the part you need and say which part; if you could not read a row's whole `section_text`, say the text was read in part.
- **Which code applies.** Take any commencement date from `successor_effective_from`, `regime_info.effective_from` or the note, never from memory. `regime_info.applicability_requires_facts: true` means the choice between the old and new provision depends on facts such as dates; say so, and ask for the facts rather than deciding it.
- **A correspondence is not equivalence.** The source table maps sections; it does not say the texts are the same. Compare the two texts before saying anything about a change.
- **State-amended rows.** A row from a State collection (`extras.collection` names the State) can carry a State amendment of an old-code section, with a note that the amendment was made to the repealed Act. Keep it apart from the central row; do not say whether it survives under the new code unless a retrieved source says so.
- **Missing or failed mapping.** `successor_lookup: "unavailable"`, or a note saying the successor catalog is unavailable or not seeded, means the mapping was not checked; it does not mean there is no counterpart. Paired rows are looked up only for an old-code row with `pinned: true`; if the old row is not pinned, the mapping was not checked, so re-request it with its `act_id` and `section_number`. Without those signals, a pinned old-code row with no paired row means Vitark's table lists no counterpart; report it in those words.
- **Rows that are not enacted text.** `corpus_kind` labels a row `enacted`, `report`, `notification` or `other`. Quote only `enacted` rows as the law.

## Write the answer

Prefer primary material and stronger courts while still testing relevance. Read the full text of authorities that carry the conclusion. Separate what each source states from your analysis, and identify it with the best metadata returned: title, court, decision date or year, citation or case number, CNR, and source URL.

Quote only text retrieved from complete `section_text` or judgment text, keep excerpts short, and label paraphrases. Do not invent missing reporter citations, paragraph numbers, quotations, holdings, treatment, or procedural history. A case or section not found in Vitark is "not found in Vitark", never "does not exist" or "fake". Explain conflicting or weak authority and state when no responsive primary source was found. Do not guarantee a legal outcome. For decisions with real consequences, tell the user to verify current law and obtain qualified advice.

Vitark uses the signed-in customer's existing account and shared credit wallet. The MCP connection requests `research:read` for research tools and `credits:read` for the balance tool. Connecting another host does not create another wallet or signup benefit. Do not promote a purchase or account upgrade.

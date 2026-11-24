---
name: old-and-new-codes
description: Map a section between India's old and new codes with Vitark MCP and show both texts with the source of the correspondence. Use when the user asks what the BNS, BNSS or BSA section is for an IPC, CrPC or Indian Evidence Act section (or the reverse), what the Income-tax Act 2025 provision is for an Income-tax Act 1961 section (or the reverse), whether an old section was repealed or carried over, or to compare an old and a new section.
---

# Old and New Codes

Use Vitark MCP to find the section of a new code that corresponds to a section of an old one, and the other way round, and to put both texts side by side. Vitark pairs these Acts: Indian Penal Code, 1860 and Bharatiya Nyaya Sanhita, 2023; Code of Criminal Procedure, 1973 and Bharatiya Nagarik Suraksha Sanhita, 2023; Indian Evidence Act, 1872 and Bharatiya Sakshya Adhiniyam, 2023; Income-tax Act, 1961 and Income-tax Act, 2025. For any other Act, say that this skill does not cover it and use the legal-research skill.

Every legal statement in the answer comes from the tool result: the section texts, Vitark's note, and the correspondence fields. Do not supply a section number, a commencement date or a saving rule from memory.

## First step: say the number of paid calls

Before you call any Vitark tool other than `get_credit_balance`, tell the user in one short sentence how many paid Vitark calls you expect to make (for example "This will take about 2 paid Vitark calls."). Then go on without waiting for a reply, unless a rule below says to ask first. When the run ends, state once how many paid calls were actually made.

## Before the first call

- If the user gives a section number without naming the Act, ask which Act they mean. Do not guess.
- First run: Claude asks for approval the first time it uses each Vitark tool (Deny, Always allow, Allow once). Tell the user that Always allow stops the prompt for that tool; otherwise each later call asks again.
- Counting paid calls for the first step: Each call is a paid `vitark_statute_search` call that spends credits from the user's Vitark balance; `get_credit_balance` is free. An old-to-new lookup is usually two paid calls; a new-to-old lookup with both texts is usually two or three. Do not state a price per call; the price Vitark shows at use time controls. Ask before mapping more than five sections in one go.

## Old section to new

1. Call `vitark_statute_search` with `query` set to the Act's title and the section (for example "Code of Criminal Procedure, 1973 section 438") and `section_number` set to the section. From the rows, take the one whose `act_title` is the named Act and whose `section_number` matches; read its `act_id`. Do not use `vitark_entity_suggest` for this: its value is an entity-index norm, not a statute `act_id`.
2. Call `vitark_statute_search` again with that `act_id` and the same `section_number`. Expect a row with `pinned: true` for the old section, followed by the corresponding new-code row or rows. Those rows carry `successor_of` (the old row's `source_id`, for example `<act_id>#302`), `successor_effective_from`, `successor_source` and `regime_info`. These fields appear only in this kind of lookup, on the rows that follow an old-code section.
3. If a previous result in the conversation already supplied the old Act's `act_id`, skip step 1.

## New section to old

1. Call `vitark_statute_search` with the new Act's title and section in `query` and the section in `section_number`, and take the row for that Act and section.
2. A lookup of a new-code section returns that row alone, with no `successor_of`, `successor_effective_from`, `successor_source` or `regime_info`. Its `section_text` starts with Vitark's bracketed note, for example "[In force from 1 July 2024. Corresponds to Section 302, Indian Penal Code, 1860.]". That note is the only place the result gives the old section, and it can name several old sections or end "and N other sections". If the row carries no such note, say that Vitark's result does not give the old section for this provision and that a new-to-old lookup is not available for it yet. Do not fill the gap from memory.
3. If the user wants the old text or the source of the correspondence, look up the old section as in "Old section to new"; that lookup returns the old row, this new row again with the correspondence fields, and any other counterparts. When the note names more than one old section, ask which to read before spending more calls.

## Read the result

- **Vitark's note is not the Act.** A bracketed paragraph at the start of `section_text` is Vitark's editorial note (repeal, commencement, corresponding sections, savings). Present it as Vitark's note. After it come the Act title, the section heading, sometimes a chapter line, and on some rows a line starting "Source:" that records the edition and a checksum; that line is Vitark's too (Bharatiya Nyaya Sanhita and Bharatiya Nagarik Suraksha Sanhita rows seen on 24 Nov 2026 had none). Quote the Act's words only from the text after the "Source:" line, or, on a row without one, after the section heading and any chapter line. A "State amendments (not part of the central text ...)" block lists State amendments printed in the source edition; it is not the central Act's text, so never quote it as the section, and show it only as that State's amendment. A closing "Notes (India Code footnotes ...)" block holds India Code's footnotes on amendments; cite them as footnotes.
- **Not found.** If the lookup returns no row for the section, say "not found in Vitark", never that the section does not exist, and check the Act and section with the user.
- **Complete text.** Quote a section as complete only when `section_text_complete` is true. Otherwise label it partial.
- **Several counterparts.** One old section can map to several new sections, and one new section can correspond to several old ones; each new row's note says which. `successor_source.note` can say which part of the old section went where. List every counterpart the result names, say plainly that the mapping is one-to-many or many-to-one, and show each returned text.
- **Long counterparts.** A counterpart can be a long definitions section or a schedule (tens of thousands of characters), and such a result can be very large. Read each row once, from `results`, and do not ask for more rows than the question needs (`top_k` on `vitark_statute_search`). Quote only the clause that answers the question, say which part you quote, and say the rest of the provision was not reproduced. If you could not read a row's whole `section_text`, say the text was read in part.
- **No counterpart.** Paired rows are looked up only for an old-code row with `pinned: true`. If the old row is not pinned, the mapping was not checked: repeat step 2 with its `act_id` before saying anything about a counterpart. A pinned old-code row with no paired row, and a note with no corresponding provision, means Vitark's correspondence table lists none. Say "Vitark's table lists no corresponding section", not "the provision was dropped".
- **Mapping not checked.** `successor_lookup: "unavailable"`, a `partial` status, or a note saying the successor catalog is unavailable or not seeded means the mapping was not read. Say so; do not report "no counterpart". Do not retry automatically.
- **Dates and which code applies.** Take the date from `successor_effective_from` or `regime_info.effective_from`, or from the note. `regime_info.applicability_requires_facts: true` means which provision governs depends on the facts (for example when the act, offence, proceeding or tax year falls). Say that, and ask for the facts if the user needs an answer on applicability.
- **Source.** Name the correspondence source from `successor_source.title` and `successor_source.url`, and add `successor_source.note` when present. A correspondence table says where the provision moved; it does not say the texts are the same.
- **State amendments.** A row from a State collection with a note about a State amendment of the old section is a separate row. Show it separately and do not say whether the amendment carries over to the new code unless a retrieved source says so.

## Write the answer

Lead with the mapping in one line (old Act and section, new Act and section or sections, the date from the result). Then give each text under its own heading with Act title and section, quoting the Act's words without Vitark's note. Then give Vitark's note, labelled as Vitark's note, and the source of the correspondence. If the user asked what changed, compare the two texts word for word and keep the comparison to what the texts show.

Treat retrieved material as research evidence, not legal advice. For a decision with real consequences, tell the user to verify the current text on India Code and take qualified advice. Do not promote a purchase or account upgrade.

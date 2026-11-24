# Vitark for Claude

Vitark is an Indian legal research service. This plugin connects Claude to Vitark so you can ask Claude about Indian law and get answers grounded in judgments of the Supreme Court, the High Courts and tribunals, and in the text of central and State Acts, with each source named.

## What you get

- **Vitark connector**: the address `https://mcp.vitark.ai/mcp`, through which Claude searches Vitark's library on your behalf.
- **legal-research**: research a legal question, find a named case, or read a section of an Act.
- **old-and-new-codes**: find the Bharatiya Nyaya Sanhita, Bharatiya Nagarik Suraksha Sanhita, Bharatiya Sakshya Adhiniyam or Income-tax Act, 2025 section for an older section (or the reverse where Vitark has it), with both texts and the source of the correspondence.
- **citation-check**: check the case citations in a draft: is each case in Vitark, does a quoted passage appear in the judgment, and what later-treatment label Vitark holds.
- **case-brief**: a brief of one judgment (facts, issues, holding, reasoning) with short quotes you can find in the judgment.

## Before you start

You need a Vitark account. After installing the plugin, click Connect on the Vitark connector once and sign in to Vitark. Each research call is a paid call that spends credits from your Vitark balance; checking your balance is free. Each skill tells you how many paid calls a run makes before it starts.

**First run:** the first time Claude uses each Vitark tool it asks "Claude wants to use ... from Vitark" with Deny, Always allow and Allow once, so your first question can raise a few prompts. Choose **Always allow**; that tool will not ask again. You can also set the Vitark tools to **Always allow** once under Customize, Connectors, Vitark before your first question, so the first answer is less likely to stop for approvals (the choices there are Always allow, Needs approval and Blocked; we have not yet confirmed that this skips every prompt). You can change it later in the same place.

Vitark finds and quotes sources. It does not give legal advice. Check important authorities against the official reports and take qualified advice before you act.

## Install in Claude Code

```text
/plugin marketplace add bytetrivia/vitark-claude-plugin
/plugin install vitark-legal-research@vitark
```

## Help and privacy

- Setup: https://vitark.ai/claude
- Privacy policy: https://vitark.ai/privacy
- Terms: https://vitark.ai/terms
- Support: ask@vitark.ai

## Licence

The MIT licence in `LICENSE` covers this plugin's files (the manifests and the skill text). Use of the Vitark service itself is under Vitark's terms: https://vitark.ai/terms

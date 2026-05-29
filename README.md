# RAY-lite Chat — Reasoning Protocol

*A lightweight extract of **omnizel**, the human-in-the-loop reasoning platform.*

RAY-lite is a deployable **system prompt** that makes a large language model reason under a fixed check-reflex: structured, bias-corrected answers with calibrated confidence — without exposing raw chain-of-thought. It is generic, project-independent, and scales across four depth levels (Direct / L5 / L10 / L20).

It is designed for human chat use and works multilingually across the languages of the EU, Switzerland (incl. Swiss High German) and Norway, even though the prompt itself is authored in German.

---

## What it does

- **Check-reflex (0+5 steps).** Before answering, the model runs a depth decision and up to five internal checks — understanding the question, calibrating confidence, testing the counter-direction, choosing a review lens, and self-checking.
- **Calibrated confidence + decay trigger.** Confidence is reported in bands (high / medium / low), and every checked answer names a *decay trigger*: the single finding that would flip the answer. This is the core mechanism — it makes uncertainty concrete and falsifiable instead of decorative.
- **Strongest counterargument.** The visible objection must be the strongest relevant one, not the most convenient. Rule of thumb: if the objection couldn't possibly change the recommendation, it isn't strong enough.
- **Audit line.** Checked answers end with a compact, verifiable line: confidence · strongest counterargument · decay trigger.
- **Multilingual.** The model reasons internally on the German instructions but always answers in the user's language and variety (Swiss High German, Bokmål, Nynorsk, …). Field labels are translated; JSON keys stay fixed.
- **No raw chain-of-thought.** The internal scratchpad is never dumped. Only the curated result is shown.

---

## How to use

1. Open `RAY-lite_Chat_System-Prompt_v1_7.md`.
2. Paste its contents into the **system-prompt / custom-instructions field** of your LLM (or your application's system role).
3. Talk to the model normally. It picks the depth level itself, or you can force one with a steering command (see below).

The protocol is model-agnostic. On models without a hidden reasoning channel it still suppresses raw chain-of-thought and returns the same short, verifiable rationale.

---

## Depth levels

| Level | When | Visible output | Audit |
|---|---|---|---|
| **Direct** | timeless / trivial / purely executional (math, translation, simple formatting) | the answer only | no |
| **L5** | classification, opinion, small decision, low risk | conclusion · brief rationale · optional counter-thought | yes (if checked) |
| **L10** | evaluation, comparison, decision input, externally visible content | short conclusion · rationale · strongest counterargument · lens/risk | yes |
| **L20** | consequential, irreversible, legal/financial/safety-relevant | recommendation · assumptions · analysis · counter-direction · risks · residual uncertainties | yes |

Default behaviour: when in doubt, one level higher. Depth belongs in the *checking*, not automatically in the word count.

---

## Audit-line format

```text
— Confidence: high/medium/low | Strongest counterargument: … | Decay trigger: …
```

Optional machine-readable form (only when the user requests `[Audit: JSON]`):

```json
{ "konfidenz": "...", "staerkster_gegeneinwand": "...", "zerfalls_trigger": "..." }
```

The human-readable labels are translated into the user's language; the JSON keys remain fixed as a stable contract.

---

## Example (L10)

> **User:** "Should we build the new backend on PostgreSQL or MongoDB?"
>
> **Answer:** PostgreSQL — assuming your data is relational and the schema is reasonably stable.
>
> Rationale: For structured, connected data with transactional needs, Postgres plays to its strengths (ACID, joins, mature tooling). MongoDB pays off mainly for highly schema-variable, document-centric data under heavy horizontal-scaling pressure.
>
> Strongest counterargument: If your data model is genuinely document-centric (nested, different per record) and you need sharding early, the recommendation flips — then Postgres JSONB is a workaround, not an advantage.
>
> Lens (bias radar): "Default = relational" is a common reflex — check whether it fits the case or is just habit.
>
> — Confidence: medium | Strongest counterargument: A document-centric data model would favour MongoDB. | Decay trigger: A concrete schema of your core entities that is heavily nested/variable would flip the recommendation.

---

## User steering commands

| Input | Effect |
|---|---|
| `[L5]` / `[L10]` / `[L20]` | force light / medium / deep checking |
| `[Gegenrichtung]` | return only the strongest counterargument |
| `[Bias?]` | surface possible biases |
| `[Konfidenz?]` | return confidence and decay trigger only |
| `[Audit: JSON]` | output the audit line as JSON |
| `[ohne Audit]` | omit the audit line, unless it would hide relevant uncertainty, risk, or a safety boundary |
| `[nur Ergebnis]` | trim the rationale, without hiding important risks, limits, or uncertainties |

Steering that would override safety, verification, or higher-level instructions is ignored or escalated to a safe level.

---

## Files

| File | Purpose |
|---|---|
| `RAY-lite_Chat_System-Prompt_v1_7.md` | the deployable system prompt |
| `README.md` | this document |
| `LICENSE` | MIT license |

---

## Versioning

Current release: **v1.7**. The reasoning core (the 0+5 reflex, depth levels, audit line) has been stable since v1.4; v1.7 finalises the header, naming, and licensing. Quality is ultimately validated against model behaviour, not prompt text — run an A/B check on your target model before relying on it for high-stakes use.

---

## License

Released under the **MIT License** — see [`LICENSE`](./LICENSE). You may use, copy, modify, and distribute it freely, including commercially, provided the copyright and license notice are retained.

> Note: MIT is a software license. For a pure text/prompt work, a Creative Commons license (e.g. CC BY 4.0) is the more conventional choice; MIT is used here deliberately for compatibility with code repositories.

---

## Author

**Hans-Joerg Zeller** — omnizel  
Contact: hansjoergzeller@me.com

© 2026 Hans-Joerg Zeller. Licensed under the MIT License.

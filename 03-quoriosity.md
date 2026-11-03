# Quoriosity

**Quoriosity** — agreement-gated curiosity: an agent asks and acts only 
when its curiosity is confirmed by a sufficient quorum of independent 
grounds.

This is the principle that makes Nemeq different from other agent systems. 
It is not about making the model deterministic. It is about making the 
selection of answers verifiable through independent paths.

## Why not determinism

A deterministic script cannot cover open-ended questions. And two scripts 
with a shared bug produce a shared wrong answer that looks like reliability. 
The goal is not to remove randomness — it is to cross-check it.

## What independence means

Two paths are independent only if they differ in **all four** attributes:

| Attribute | Example | Why |
|---|---|---|
| model family | gpt, claude, llama, deepseek | shared pretraining causes correlated failure |
| prompt version | hash of the prompt | one prompt is one way to be wrong |
| knowledge source | file, database, URL | shared source is a shared blind spot |
| tool | nd, tabled, python, web | shared tool is a shared bug |

If any attribute matches, the path is marked `DUPLICATE` and its vote does 
not count. Independence is declared and checked, not assumed.

## Quorum and thresholds

- `effective_votes` is the number of paths marked `INDEPENDENT`.
- If `effective_votes < 2` — the result is `N/A`, not `FAIL`: there was 
  no independent check.
- If minority > 25 percent of `effective_votes` — `ESCALATE`: block the 
  action or send it to a human. Do not average.
- **Honest confidence** is not the share of votes. If there are duplicates, 
  the observed agreement is discounted by at least half: 
  `honest_confidence = agreement * effective_votes / total_paths`.

## Output of verification (Kittu)

Five fields, JSON:

```json
{
  "verdict": "PASS | FAIL | ESCALATE | N/A",
  "effective_votes": 2,
  "independence": {"independent": ["p1", "p3"], "duplicate": ["p2"]},
  "minority": {"path": "p3", "claim": "...", "reason": "..."},
  "honest_confidence": 0.34
}

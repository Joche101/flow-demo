# The Flow — receipts from a governed record pipeline

**John Bianchina** · [pursuitreadiness.com](https://pursuitreadiness.com) · john@pursuitreadiness.com · [LinkedIn](https://www.linkedin.com/in/john-bianchina-88965078/)

This repo contains one artefact: `receipts.jsonl`. It is the complete audit log from a real
production run of a nine-stage record-processing pipeline — 369 records, hash-chained, every stage
accounted for.

No code. No client data. Just the evidence.

---

## Why receipts and not a demo

A demo shows you what a system does when everything works. Receipts show you what it actually did.

This is the difference between a claim and a record. Anyone can show you a screenshot of a
dashboard. What follows is the raw log from a pipeline that ran against a real reviewed batch, and
you can verify every link in the chain yourself in about ten seconds.

---

## What the run shows

| Stage | Records | Actor |
|---|---|---|
| `source` | 75 | system |
| `validate` | 75 | system |
| `agent` | 75 | `agent:template` |
| `human_gate` | 75 | `REVIEWER (recorded)` |
| `action` | 69 | system |

**75 records entered. 75 were reviewed by a named human. 69 were acted on. 6 were rejected.**

The six that did not proceed are the point. A pipeline that reports only its successes is not
reporting — it is marketing. The rejections are in the log because the log records decisions, not
outcomes.

Every record carries: timestamp, stage, record ID, actor, model, token count, outcome, request ID,
the previous record's hash, and its own hash. Nothing carries the message content. **The receipt
logs the doing, not the content** — that is deliberate, and it is what makes the log safe to hold
and show.

---

## Verify it yourself

The chain is only meaningful if you can check it without trusting me. Save `receipts.jsonl` and run:

```python
import json

rows = [json.loads(line) for line in open("receipts.jsonl") if line.strip()]

prev, broken = "GENESIS", 0
for r in rows:
    if r["prev"] != prev:
        broken += 1
        print(f"BROKEN at {r['record']} — expected prev={prev}, found {r['prev']}")
    prev = r["hash"]

print(f"{len(rows)} records, {broken} broken links")
```

Expected output: `369 records, 0 broken links`

Alter any single record and every subsequent link fails. That is the property that matters — an
editable log proves nothing when the question is whether the log itself can be trusted.

---

## The nine stages

The pattern this came from, written as a system rather than a diagram:

1. **Source** — records arrive, logged with an ID
2. **Intake** — batch selection, in fixed sizes
3. **Validate** — required fields, exclusions, duplicates. No model involved
4. **Route** — clean records proceed, everything else is reason-coded
5. **Agent step** — drafts, researches, builds
6. **Approval gate** — a named human approves. System can block, never send
7. **Action** — the send, the write, the update
8. **Receipt** — stage, actor, timestamp, request ID. Never the content
9. **Measurement** — computed from the receipts, not typed by hand

Stages 3 and 9 are the ones most implementations skip. The first because a model is more fun than a
validation rule. The second because a number someone typed is easier than a number the system
computed.

---

## Context

I built this pattern while implementing an outreach system for a healthcare practice — a sixteen-month
engagement that completed in September 2026. The requirement was simple to state and hard to
satisfy: nothing goes out unless a named human approved it, and you have to be able to prove
afterwards that it happened that way.

What is in this repo is the receipt layer from that work, anonymised and released on its own. The
pipeline code itself is not published, and no client material is included — the log was scrubbed
before release and record identifiers were replaced with sequential IDs.

The figures from that engagement, for the record: 273 emails delivered across 20 batches, a 42.7%
human open rate, and a 3.8% hard bounce rate. The raw open figure measured eleven points higher
until I traced it to email prefetch and removed it. That correction is itself an example of the
discipline this repo is about.

---

**John Bianchina** · [pursuitreadiness.com](https://pursuitreadiness.com) · john@pursuitreadiness.com

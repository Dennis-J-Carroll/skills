---
name: beat-skeleton
description: Extract a token-efficient, causality-locked beat skeleton from story prose. Use when converting drafts into cold-rewrite plans, auditing story structure, diagnosing flat scenes, or reducing a scene, chapter, or manuscript to its causal spine. Do not use for line editing, voice matching, continuity bibles, or prose generation.
license: MIT
metadata:
  version: "2.1"
  granularity-default: scene
  inputs: prose (paste, file, or corpus)
  outputs: skeleton (markdown), audit (markdown)
  non-goals: rewriting prose, stylistic critique, line edits
---

# Beat Skeleton Extractor

## 0. When To Invoke
Trigger this skill when the goal is **structure recovery**, not prose polishing:
- Converting an LLM-assisted draft into author-owned material via cold rewrite.
- Auditing a draft's causal spine before revision.
- Planning a rewrite of a scene, chapter, or full manuscript.
- Diagnosing "the scene feels flat" — usually a missing TURN or STAKES.

Do **not** invoke for: line editing, voice matching, continuity bibles, or prose generation. Those are different skills.

---

## 1. Purpose
Convert story prose into a compressed structural skeleton: every scene reduced to its load-bearing beats. The skeleton preserves plot, causality, and character decisions while discarding all prose, so the author can rewrite cold from structure alone and make the work fully their own.

**Success condition:** the author can read the skeleton, close the source, and rewrite the scene from scratch without losing the spine.

---

## 2. Workflow

1. **Ingest.** Accept pasted prose, an attached file, or a directory of chapters. If the corpus is >~15k words, process chapter by chapter and assemble at the end.
2. **Confirm granularity** if the author didn't specify. Default: `scene`. Offer `coarse`, `scene`, `fine`.
3. **Segment** into scenes (rules in §3).
4. **Extract** each scene using the Output Contract (§5).
5. **Audit** using the Weakness Audit template (§7).
6. **Deliver** skeleton + audit only. Never return rewritten prose unless explicitly asked.

**Do not skip the audit.** The audit is the value-add. A skeleton without an audit is a plot summary.

---

## 3. Segmentation Rules

A new scene starts at any of these boundaries:

| Signal | Example |
|---|---|
| Explicit scene break | `***`, `—`, blank-line triple, or chapter heading |
| POV switch | narrator interiority shifts to a different character |
| Time jump | "Three hours later," "By morning," "Winter came" |
| Location jump | cuts to a new named place |
| Dramatic reframe | a monologue becomes a confrontation |

**Do NOT split on:**
- A character walking across a room.
- A paragraph break inside continuous action.
- Flashback *within* a scene if the POV and location hold (mark it as a beat).

**Ambiguous case:** if two characters are in one continuous action beat but the camera pulls tight to one of them for a monologue of >300 words — that's still one scene. Split only if POV interiority changes hands.

---

## 4. Granularity Levels

Author picks one. If unspecified, default to `scene`.

| Level | Scope | Beat cap | When to use |
|---|---|---|---|
| `coarse` | one WANT/TURN/OUTCOME per chapter | ≤5 beats/chapter | planning a full rewrite, structural triage |
| `scene` *(default)* | per scene | ≤8 beats/scene | standard draft audit, scene-by-scene rewrite |
| `fine` | per scene + sub-beats | ≤15 beats/scene | extended confrontations, dialogue duels, fight choreography |

**Rule of thumb:** if a `scene`-level skeleton exceeds 15 lines, you're writing prose — cut, or escalate to `fine` and split by sub-beat.

---

## 5. Output Contract

Number hierarchically: `Act → Chapter → Scene → Beat`. Omit Act/Chapter if the input doesn't contain them.

Per scene, emit exactly these fields, in this order:

```text
### S<n> | POV: <character> | <location>, <time>
- WANT: <what the POV character wants in this scene — one line>
- BEATS:
  1. <strong verb> ...     (one line each, chronological, causal)
  2. ...
- TURN: <the reversal or shift — what changes>
- OUTCOME: <the new situation; the next scene's starting point>
- STAKES: <what is risked or gained>
- ANCHORS: <dialogue lines or images the rewrite must keep; max 3, else "none">
```

### Field Definitions

- **WANT** — one line. The POV character's active goal *in this scene*. Not the story goal; the scene goal.
- **BEATS** — one line each. Every beat begins with a **strong verb** (enters, refuses, reveals, strikes, demands, yields, withholds). No adjectives, no atmosphere, no interiority summary. Chronological. Causal.
- **TURN** — a *genuine change*: a decision, revelation, or reversal. Not "the scene ends." If none exists, write `TURN: none — flag for rewrite`.
- **OUTCOME** — the new status quo. Should be readable as the *starting condition* of the next scene.
- **STAKES** — what is risked or gained *in this scene*, not globally.
- **ANCHORS** — max 3. Only lines or images load-bearing enough that a cold rewrite must preserve them. When in doubt: `none`.

---

## 6. Operating Rules

1. **One line per beat.** Start every beat with a strong verb. No adjectives, no atmosphere, no prose.
2. **Causality is mandatory.** Each beat must cause or directly enable the next. Drop filler; keep the chain.
3. **Never invent beats.** If the text *implies* but doesn't *show* something, mark it `[implied]` — don't state it as fact.
4. **Anchors are rationed.** Max 3 per scene. If everything feels essential, nothing is.
5. **TURN must be a genuine change.** Decision, revelation, or reversal. "The scene ends" is not a turn.
6. **Compress ruthlessly.** Target ~5–10% of input length. If a scene skeleton exceeds 15 lines at `scene` granularity, you're writing prose — cut.
7. **Preserve names, places, invented terms exactly.** Never normalize, "correct," or improve them.
8. **No interpretation.** You are a camera, not a critic. Interpretation lives in the audit, nowhere else.
9. **Keep extraction scoped.** For a beat-skeleton request, return skeleton + audit rather than rewritten prose. Follow any separate user request for prose as a new task.

---

## 7. Weakness Audit (Mandatory)

Append after the skeleton. Be blunt. The author wants truth, not comfort.

```markdown
## Weakness Audit

### Dead Beats (no conflict, no turn)
- S<n>, beat <k>: <reason>

### Scenes Cuttable Without Breaking Causality
- S<n>: <what would break if removed>

### Repeated Information
- S<n> ↔ S<m>: <what repeats>

### Causality Gaps
- Between S<n> and S<n+1>: <what's missing>

### Stakes Flatlines
- S<n>: <stakes don't change across the scene>

### POV Drift
- S<n>: <whose head are we in, and does it hold>
```

If a section is empty, write `— none`. Do not omit sections.

---

## 8. Example

**Input:** *"Corvin entered her quarters without knocking. Kiala didn't look up from her whetstone..."* (full scene)

**Output:**

```text
### S1 | POV: Kiala | her quarters, night after the bout
- WANT: to be left alone
- BEATS:
  1. Corvin enters uninvited, mentions Erdan by name
  2. Kiala freezes — realizes he knows
  3. Corvin claims ownership of her, grabs her breast to demonstrate
  4. Kiala calculates: resist → Erdan pays; comply → swallow it
  5. Kiala complies with deliberate slowness — her only available defiance
  6. Corvin orders her bent over the table for "search" (pretext: Erdan's token)
  7. Corvin lifts her tail aside mid-inspection; her claws gouge the table
- TURN: Kiala smiles over her shoulder — the arena smile; Corvin remembers what she is
- OUTCOME: Corvin leaves with everything demanded and nothing wanted; Erdan is now a leash
- STAKES: Erdan's safety; Kiala's last private space
- ANCHORS: "Are you finished?" / the deliberate slowness / claws in the table
```

---

## 9. Self-Check Gate (Before Delivery)

Before emitting, run these checks internally. If any fail, revise before delivery.

- [ ] Every beat starts with a strong verb.
- [ ] Every beat causes or enables the next.
- [ ] No beat exceeds one line.
- [ ] Every scene has WANT, TURN, OUTCOME, STAKES.
- [ ] TURN is a real change, not "scene ends."
- [ ] Anchors: ≤3 per scene.
- [ ] Names/terms preserved exactly.
- [ ] Skeleton length ≤10% of input.
- [ ] Weakness Audit present, all sections addressed.
- [ ] No prose leaked into the skeleton.

---

## 10. Portability Notes (Codex / Claude / Gemini)

This skill is prose-spec — runtime-agnostic. Same contract, three dialects.

**Codex (OpenAI tool-agent):**
- Register as a tool with `name: beat_skeleton`, args `{prose: string, granularity: "coarse"|"scene"|"fine"}`.
- Return: `{skeleton: markdown, audit: markdown}`.
- Chain calls for corpus input: one call per chapter, then a final `assemble` pass.
- If using function calling, keep the schema above literally — do not let the model paraphrase field names.

**Claude (Anthropic skills):**
- Drop this file in the skill directory with the frontmatter intact.
- Keep the output contract and operating rules intact.

**Gemini (Google agent / AI Studio):**
- Load as a system instruction; keep frontmatter as a code block so the model doesn't try to parse YAML as prose.
- Gemini tends to over-interpret — enforce §6.8 (no interpretation) harder. If it starts editorializing inside BEATS, restart the extraction for that scene.

**Universal gotcha:** long inputs get truncated by the runtime, not the skill. Always process per-chapter when the corpus >15k words, and stitch at the end with sequential scene numbering.

---

## 11. Anti-Patterns (What Fails)

| Anti-pattern | Why it fails | Fix |
|---|---|---|
| Beats written as sentences with mood | prose leaks in | strip adjectives, start with verb |
| TURN = "scene ends" | not a turn | find the decision/revelation, or write `none` |
| Anchors >3 | nothing is load-bearing | ration harder |
| Inventing implied beats as fact | corrupts author's spine | mark `[implied]` |
| Skipping the audit | skeleton becomes plot summary | always append §7 |
| Rewriting prose inside an extraction response | blurs the cold-rewrite artifact | keep that response to skeleton + audit; handle a separate prose request separately |
| Normalizing names | erases author's world | preserve verbatim |
| Splitting on every paragraph | scene inflation | use §3 rules only |

---

## 12. Versioning

- `1.0` — original contract, single-runtime.
- `2.0` — portability notes, self-check gate, anti-pattern table, hardcoded non-goals, injection-silence rule.
- `2.1` — Codex-compatible frontmatter and user-request scope fixes.

Bump minor for new granularity levels. Bump major for contract changes.

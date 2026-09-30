# The Unofficial Guide

Pankaj Gupta — advice_threads

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

<!-- Three or four sentences. Which corpus you picked, and the kinds of
     questions your system answers. Write it for someone who has never seen
     this repo.

     Milestone 5. -->

## Chunking Strategy

**Chunk size:** 280
**Overlap:** 60

### What was changed:
We replaced the starter's fixed-size character sliding window (`fallback_split`) with a custom thread-aware paragraph splitting strategy (`chunker.py::split_documents`). Instead of slicing across fixed character counts, we split documents on paragraph breaks (`\n\n`) and prepend the original `THREAD: <topic>` line to every individual student reply.

### Why we did it:
1. **Preventing Cut-off Thoughts:** The default 800-character chunker blindly slices through characters and sentences. In `advice_threads` (where threads range from 317 to 793 characters), the default sliding window with step size 680 created an awkward 2-character chunk containing just `'t.'` from `thread_meal_plan_tier.txt`.
2. **Context Preservation:** Student replies are conversational and directly address the main thread question. If a reply is severed from its prompt, the vector embedding loses semantic focus. Attaching the thread title to each reply keeps every chunk self-contained and clear for retrieval.

### Expected Improvement / Results:
- **Zero Cut-off Sentences:** Every chunk contains a complete, coherent student thought without broken sentences (satisfying Criterion 4).
- **Cleaner Corpus Granularity:** Produces 75 well-bounded chunks (min 132 chars, max 281 chars, avg 202 chars) instead of irregular multi-reply slices.
- **Sharper Retrieval & Lower Cost:** On test queries (e.g. bike commuting), retrieval distance dropped from 0.314 to 0.279, while input tokens per call decreased by ~46% (from 898 to 481 tokens).

## Sample Chunks

**Chunk 1** — source: `thread_bike_commute.txt#0` — produced by: `chunker.py::split_documents`

```
THREAD: Is a bike worth it for a 20 minute walk commute?

--- reply 1 (14 votes) ---
Yeah. Cuts an 18 minute walk to about 6. The thing nobody mentions is storage — covered bike parking exists at three buildings and is full by 9am at all three.
```

**Chunk 2** — source: `thread_first_gen.txt#1` — produced by: `chunker.py::split_documents`

```
THREAD: Anything specific for first-generation students?

--- reply 2 (41 votes) ---
The thing I'd say: the unwritten rules are the hard part, not the coursework. Ask about the unwritten rules explicitly. People are happy to explain them and nobody volunteers them.
```

**Chunk 3** — source: `thread_laptop_specs.txt#2` — produced by: `chunker.py::split_documents`

```
THREAD: How much laptop do I actually need for CS courses?

--- reply 3 (12 votes) ---
I did two years on an 8GB machine and it was fine until the last project, at which point it very much wasn't. 16 is the answer.
```

**Chunk 4** — source: `thread_parking.txt#1` — produced by: `chunker.py::split_documents`

```
THREAD: Worth getting a parking permit?

--- reply 2 (21 votes) ---
Street parking on Verrill is legal and free and unmarked, which is why half the upper years do it.
```

**Chunk 5** — source: `thread_sleep_schedule.txt#1` — produced by: `chunker.py::split_documents`

```
THREAD: Everyone says fix your sleep. Does it actually matter?

--- reply 2 (37 votes) ---
The library being open until 2am is a trap. It's a resource, not a schedule.
```

## Sample Answer

**Question:** Is buying a bike worth it for campus life

**Answer:**

```
Based on the provided documents, a bike can cut an 18-minute walk down to about 6 minutes, but storage is an issue because covered bike parking fills up by 9 am, and salt from November to March can destroy a drivetrain in one season. However, doing free bike registration through the campus can help recover a bike if it is taken (thread_bike_commute.txt).

Source: thread_bike_commute.txt
```

**My relevance cutoff:** 0.65

### What the two groups looked like and where the gap was:
I measured the best retrieval distance for all five of my in-corpus questions and the five out-of-scope questions:

* **In-Corpus Group:** Distances ranged from `0.2785` to `0.5286` (average: `~0.4385`). The closest match was the bike commute question (`0.2785`), while broader topics like office hours and cafes landed around `0.52`.
* **Out-of-Scope Group:** Distances ranged from `0.8075` to `0.8964` (average: `~0.8651`). The closest unrelated query was the ibuprofen dosage question (`0.8075`).
* **The Gap:** There is a clean, distinct gap of over `0.27` between the worst in-corpus question (`0.5286`) and the best out-of-scope question (`0.8075`).

I set `THRESHOLD = 0.65` in `config.py`, which sits right in the middle of the `0.53` to `0.80` gap. This ensures all 5 in-corpus questions pass the relevance gate while cleanly blocking all 5 out-of-scope questions.

| Question | In corpus? | Best distance |
|---|:---:|:---:|
| What is the best cafe or restaurant for a quick bite? | Yes | 0.5286 |
| Is buying a bike worth it for campus life | Yes | 0.2785 |
| What should first generation college students expect? | Yes | 0.4584 |
| Do I need a laptop for any classes? | Yes | 0.4024 |
| Are office hours worth it? | Yes | 0.5248 |
| What is the capital of Mongolia? | No | 0.8935 |
| How do I change the oil in a diesel engine? | No | 0.8964 |
| Who won the 1994 World Cup? | No | 0.8934 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.8075 |
| How do I write a for loop in Rust? | No | 0.8348 |

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.**

**2.**

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->

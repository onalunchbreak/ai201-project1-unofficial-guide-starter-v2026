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

I built this guide to help students like me navigate the everyday, unwritten rules of university life using the `advice_threads` corpus. Instead of scrolling through 23 messy forum threads about bike storage, meal plan tiers, CS laptop specs, or handling bad roommates, anyone can just ask a question in plain English. The system searches through past upperclassmen replies, checks that the question is actually related to campus life, and pulls together a straight answer backed up by real student advice and thread citations.

## Chunking Strategy

**Chunk size:** 280
**Overlap:** 60

When I first opened the files in `advice_threads`, I noticed that each document is basically a short forum discussion: a title line at the top (`THREAD: ...`) followed by 3 or 4 separate student replies separated by blank lines. The entire thread is only 300 to 800 characters long.

The default starter used a sliding character window of 800 characters with a 680-character step. That completely butchered these short threads—in `thread_meal_plan_tier.txt`, which was 682 characters long, it grabbed the first 680 characters and then spit out an absurd 2-character chunk containing just `"t."` at the end. Even worse, blindly slicing by character count cut sentences right in half, separating critical words like "don't" from the advice that followed.

I started out thinking I could just tune the character numbers down, but that still risked cutting off replies awkwardly. So I changed my approach: instead of fixed character windows, I rewrote `chunker.py::split_documents` to split each thread along paragraph breaks (`\n\n`) so that every individual reply becomes its own chunk.

There was one big catch though: if a student replies "I sold mine, salt destroys it," that sentence makes no sense to a retrieval model unless you know the question was about bringing a bike. To fix that, I had my chunker grab the `THREAD: ...` title from the top of the file and prepend it to every reply.

This gave me 75 clean, self-contained chunks that average about 200 characters each (longest is 281, shortest is 132). Every single chunk now reads as a complete thought without cut-off sentences, and when I tested it on sample questions like the bike commute, retrieval distance got noticeably sharper (dropping from 0.314 to 0.279) while cutting our input tokens almost in half.

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

To find the right cutoff, I ran all five of my campus questions and the five `OUT_OF_SCOPE` questions through retrieval and recorded the best distance score for each:

* **Campus questions (in corpus):** All five landed between `0.2785` and `0.5286` (averaging around `0.44`). The bike commute question had an almost exact match at `0.2785`, while broader topics like office hours and finding a quick bite sat higher, around `0.52`.
* **Out-of-scope questions:** These were way further out, clustering tightly between `0.8075` and `0.8964` (averaging around `0.87`). Even the closest unrelated question (asking about ibuprofen dosage) couldn't get closer than `0.8075`.
* **Where the gap was:** That left a massive, clear gap between `0.5286` (my highest in-corpus distance) and `0.8075` (the lowest out-of-scope distance)—nearly `0.28` of empty space.

I chose `0.65` for `THRESHOLD` in `config.py` because it sits comfortably right in the middle of that gap. It gives campus questions plenty of headroom to match even if phrased casually, while shutting the door firmly on off-topic questions.

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

**1.** When I was working on my chunking function, I asked Claude how to split each thread file by double newlines (`\n\n`) while keeping the main question attached to each chunk. It gave me a script that extracted the `THREAD:` line and prepended it to each reply, which worked well. But it left `TOP_K` set to 5. When I looked at my actual documents, almost every thread only has 3 or 4 replies total, which meant top-5 retrieval was always dragging in a 5th chunk from a completely unrelated thread. I changed `TOP_K` to 4 in `config.py` to stop that bleed.

**2.** When I was choosing a relevance threshold in Milestone 4, I originally wanted to use 0.45 because I assumed a lower number meant a safer, tighter filter. Claude pointed out that cosine distance works backwards from similarity—0.0 is an exact match and 1.0 is unrelated—so setting 0.45 would have accidentally blocked three of my own campus questions (like office hours at 0.525). I asked it to run all 10 test questions so I could see the actual numbers side by side. Once I saw the in-corpus questions maxed out at 0.53 and the out-of-scope ones started at 0.81, I set the cutoff to 0.65 right in the middle.

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
| 4. Chunks contain complete thoughts | 4 of 5 |  |  |  |  |
| 5. Both sides of conflicting advice included | 4 of 5 |  |  |  |  |

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
| 4. Chunks contain complete thoughts | 4 of 5 |  |  |  |  |
| 5. Both sides of conflicting advice included | 4 of 5 |  |  |  |  |

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

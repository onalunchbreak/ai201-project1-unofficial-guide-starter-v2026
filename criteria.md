# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer.

**Why this target:**
Forum posts in `advice_threads` are written casually by students rather than as clean, keyword-dense FAQs. Direct questions like bike commuting or CS laptop specs match easily, but more conversational topics like finding a quick bite or making friends can overlap across several threads. I expect at least one question to be trickier for retrieval to nail on the first try.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:**
The prompt explicitly instructs Gemini to cite the thread file it used, and our pipeline attaches the filename metadata directly to every retrieved chunk. Because this is built right into the prompt instructions, all five answers should cite a source unless the model completely ignores the formatting rule.

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.

**Why this target:**
When I tested the distance gap in Milestone 4, every out-of-scope question scored 0.80 or higher, while all five campus questions stayed below 0.53. With my cutoff at 0.65, all five random questions got blocked cleanly. I'm keeping the target at 4 of 5 as a reasonable margin in case an off-topic question accidentally shares common vocabulary with a student post.

---

## 4. Chunks contain complete thoughts without cut-off sentences

At least 4 of 5 sampled chunks read as a complete thought, with no sentence cut in half at either end.

**Why this target:**
The starter's fixed character chunker sliced text regardless of sentence boundaries, which on forum posts meant losing key context—like separating a 'never do this' warning from the advice itself. Splitting by reply ensures almost every chunk is intact, but I kept the target at 4 of 5 in case a student wrote a massive run-on paragraph or used weird punctuation that trips up the splitter.

---

## 5. Both sides of conflicting advice included

For at least 4 of 5 questions, when the conversation thread contains disagreeing or conflicting replies, the generated answer should mention both perspectives (pros and cons) rather than picking just one person's opinion.

**Why this target:**
Campus advice threads are rarely unanimous—one student says bringing a bike changed their life, while another says winter salt ruined their drivetrain by February. A helpful guide shouldn't just parrot the first reply it finds; it needs to capture the actual back-and-forth debate. 4 of 5 allows for the occasional topic where everyone on the thread genuinely agrees.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in unit 2:** For at least 4 of 5 questions, the top three
         > results contain the answer.
         >
         > **Why revised:** I couldn't judge "the chunks include one that
         > contains the answer" the same way twice — I scored two questions
         > differently on Monday than on Wednesday. The new version is
         > something I can actually check.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said 4 of 5 but got 2 of 5, so 2 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.

     The whole reason the originals stay visible is so someone can see what you
     said before you knew the answer.
     ───────────────────────────────────────────────────────────────────────── -->

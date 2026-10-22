# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"The agent handles errors"* is an opinion.
*"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**
<!-- Why 4 of 5 and not 5 of 5? Something about your search, probably —
     "my search is a plain keyword match and some phrasings will miss" is a
     real answer. -->
search_listings matches the words in the query against listing text, and the sizes in the data are free text ("S/M", "W30 L30", "XL (oversized)"). Some phrasings will miss even when a good listing exists. Two of the three tools also call a model, which can fail or time out. Allowing one miss in five fits that, and 5 of 5 would set a target I couldn’t control.
---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
<!-- Why is 5 of 5 reasonable here when criterion 1 isn't? What's different
     about this path? -->
This path is plain code. run_agent checks if not session["search_results"], sets error and returns, and no model is involved. A failure here is a bug in the loop, not a wording problem, so it should be 5 of 5. The loop also has no later step that could make the early exit flaky.
---

## 3. Something about state

<!-- YOU WRITE THIS ONE.

     How would you know that the item your search found is the same item the
     next tool received? Name something countable or observable.

     This is the criterion people find hardest, because state failure doesn't
     look like state failure — it looks like a tool problem. Something that
     compares session["selected_item"] against what actually reached
     suggest_outfit is the shape you're after. -->

For 5 queries that each match a listing, session["selected_item"]["id"] equals session["search_results"][0]["id"]. It also equals the id of the item passed into suggest_outfit and create_fit_card, which I check by recording the argument each tool receives — 5 of 5 tries.

**Why this target:**
If the agent passes the wrong item to a tool, it looks like the model did a bad job, when really an old or overwritten value got passed along. Checking that the ids match at each step catches that. It’s 5 of 5 because the item is just copied from one place to another with no model involved, so any mismatch is a bug.


---

## 4. Something about the fit card

<!-- YOU WRITE THIS ONE.

     The fit card calls a model, so the same input can produce different words
     each time. That's not a bug — it's the nature of the tool. So what would
     make it acceptable?

     Think about what you'd actually be unhappy to see. A caption that never
     mentions the price? Two different items producing the same opening
     sentence? A card longer than a caption anyone would post? Any of those can
     be turned into a number. -->

For 5 different items, the fit card contains the item’s price (for example $18) and is 300 characters or fewer, in at least 4 of 5 tries.

**Why this target:**
The fit card comes from a model, so the wording is different every time and I can’t check for exact text. The price is what a shopper cares about most, and I can check for it easily. 300 characters is about as long as a caption people actually post. I’m allowing one miss out of five because the model might write "$" as "dollars".


---

## 5. Your choice

<!-- YOU WRITE THIS ONE TOO.

     Pick something you actually care about getting right. Speed, the empty
     wardrobe path, what happens when the model can't be reached, whether the
     search respects a price ceiling — anything, as long as it names a number
     or an observable outcome. -->

For 5 queries that each include a price limit (such as under $30), every listing returned by search_listings costs no more than that limit — 5 of 5 queries, with zero over-limit listings in any of them.

**Why this target:**
If someone says “under $30” and gets a $45 jacket, that’s just wrong. Checking the price is a simple number comparison, so I expect it to pass every time. If it fails, the bug is either in how I read the price out of the query or in the <= check, and this test would catch both.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->

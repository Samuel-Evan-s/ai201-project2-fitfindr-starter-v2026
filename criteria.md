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
I picked 4 of 5 rather than 5 of 5 because search is a plain keyword match over title, tags, description, colour and brand. A phrasing like "retro band shirt" shares no words with a listing titled "Vintage Band Tee" and will miss, and the model-backed second and third calls can also fail or time out. 4 of 5 allows one such miss but still fails an agent that is mostly broken.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
5 of 5 is reasonable here because this path uses no model: the branch is a plain `if not results` check on a deterministic filter, so the same impossible query gives the same result every time. One failure in five would mean the branch itself is broken, not that something was unlucky.

---

## 3. Something about state

<!-- YOU WRITE THIS ONE.

     How would you know that the item your search found is the same item the
     next tool received? Name something countable or observable.

     This is the criterion people find hardest, because state failure doesn't
     look like state failure — it looks like a tool problem. Something that
     compares session["selected_item"] against what actually reached
     suggest_outfit is the shape you're after. -->

Across 5 runs of a matching query, the listing `id` that `suggest_outfit` receives and the `id` that `create_fit_card` receives both equal `session["selected_item"]["id"]` in 5 of 5 runs, and `session["selected_item"]` is the same listing as `session["search_results"][0]`. (Check by wrapping the two tools to record the `id` they were called with.)

**Why this target:**
5 of 5, because state is plain dictionary reads in `run_agent` with no model involved. If it fails once it is a bug, not variation. I chose the item `id` rather than the title because titles can repeat across listings while ids are unique.

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

Across 5 runs of the same matching query, each fit card is 2 to 4 sentences, mentions the item's price and its platform name, and at least 2 of the 5 cards are not word-for-word identical. Separately, running 3 different items must give 3 different opening sentences.

**Why this target:**
The card calls a model at temperature 0.9, so wording varies and an exact-text check would be wrong. I made the checkable parts (length, price, platform) 5 of 5 because the prompt asks for them directly, so a miss points at a prompt problem rather than ordinary variation. I did not demand all 5 be different, since a short caption for the same item can legitimately repeat; the different-items check catches the real failure, a template-like caption.

---

## 5. Your choice

<!-- YOU WRITE THIS ONE TOO.

     Pick something you actually care about getting right. Speed, the empty
     wardrobe path, what happens when the model can't be reached, whether the
     search respects a price ceiling — anything, as long as it names a number
     or an observable outcome. -->

When the wardrobe is empty (`get_empty_wardrobe()`), a matching query still returns a non-empty `outfit_suggestion` and a fit card, with `session["error"]` equal to None, in 5 of 5 tries.

**Why this target:**
The empty wardrobe is the most likely input from a new user, and `suggest_outfit` has a separate prompt for it, so it is a distinct path from the example wardrobe. I chose 5 of 5 because the empty case is handled by a plain `if` in `suggest_outfit` and the prompt for it is simpler than the wardrobe one; a miss means that branch is wrong, not that the search was unlucky.

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

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
My search is a plain keyword-overlap match with a size filter, so a phrasing
like "tee" against a listing titled "t-shirt" scores zero and drops out, and
that is a search miss, not an agent failure. Two of the three tools also call
a model that can time out or rate-limit. I use five queries I have checked
against `data/listings.json`, so 4 of 5 allows one phrasing miss or one flaky
model call. 5 of 5 would grade my keyword matcher's vocabulary more than my
loop.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
This path has no model call and no judgment. An empty list from
`search_listings` hits a fixed `if not results:` branch that sets
`session["error"]` and returns before `suggest_outfit`. The same input takes
the same branch every time, so anything under 5 of 5 means the branch is
broken. The query is "designer ballgown size XXS under $5", which matches
nothing in the data. "Names what to change" is checked by eye: the message
must mention at least one of the size, the price ceiling or the keywords,
and "No results" alone fails.

---

## 3. Something about state

On 5 of 5 successful runs, `session["selected_item"]["id"]` equals the `id`
of `session["search_results"][0]`, equals the `id` of the `new_item` that
`suggest_outfit` received, and equals the `id` of the item that
`create_fit_card` received. All four ids match, with zero mismatches.

**Why this target:**
State failure looks like a tool problem. A stale or overwritten item would
make the outfit describe a different item than the search found, and I would
blame the model. So this is checked mechanically, not by reading output: the
trace records the `id` each tool received, and a script compares them against
the session. The check is exact and has no model in it, so I accept no
tolerance. One mismatch in five means the loop passes the wrong value. The
check also asserts the fit card's `title` and `price` come from that same
item.

---

## 4. Something about the fit card

Wording may vary run to run. What may not vary is the facts and the shape.
Across 10 fit cards (5 different items, 2 runs each), at least 9 of 10 must
contain both the item's price (e.g. "$24") and its platform name, and have 2
to 4 sentences. Separately, the 5 cards for 5 different items must not share
an identical first sentence.

**Why this target:**
I accept different words because the model is meant to vary, and I don't
test for exact text. I do test the two things I would be unhappy to see: a
caption missing the price or platform, and one too long to post. These can
be checked by string match and a sentence count. I picked 9 of 10, not 10 of
10, because sentence counting by splitting on punctuation miscounts things
like "$24.50" or "90s." and the model sometimes writes "twenty-four dollars".
One miss in ten is noise in the check. Two or more means the prompt isn't
holding the model to the facts. The opening-line check is 5 of 5 distinct,
because five different items sharing a first sentence points to a cached
answer or temperature 0 in `config.py`, and neither is acceptable.

---

## 5. Your choice

With an empty wardrobe and a query that matches a listing, the agent still
completes all three tools and returns a fit card, with `session["error"]`
None, in 5 of 5 tries. The outfit suggestion must be non-empty general
styling advice and must not name any wardrobe piece, since there are none.

**Why this target:**
The tool contract says `suggest_outfit` returns general styling advice for an
empty wardrobe instead of raising or returning "". That is a fixed branch in
my code, chosen before the model is called, so the path is deterministic up
to the model call. It is held to 5 of 5, not 4 of 5, because this is the
first-run experience for a new user and an empty wardrobe must never be a
dead end. The only way to miss is a model outage, and I run it in the same
session as criterion 1, so an outage shows up in both and I can tell it from
a wardrobe bug.

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

# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Three or four sentences: what a user asks for, and what they get back. -->
I am building an agent that makes it easier easier to thrift, Currently someone has to go to different thrift sites comparing prices. 
With fitfinder : The shopper asks: "a vintage graphic tee under $30, size M"
FitFindr gives back:  Y2K Baby Tee, Butterfly Print (lst_002). $18 on Depop, excellent condition, tagged S/M. The listing says it fits like a small, so size up if you want it looser.


---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** Filters the 40 listings by the fields the shopper names (category, style tags, size, color, maximum price, condition, brand, platform). It ranks the matches by how many requested filters each one satisfies.
- **Inputs:**  query or keywords (str), max_price (float or None), size (str or None)
- **Returns:** A list of listing dicts, best match first. Each dict has id (str), title (str), description (str), category (str), style_tags (list), size (str), condition (str), price (float), colors (list), brand (str or None) and platform (str).
- **When it has nothing:** Returns an empty list []. It never raises an error.

### `suggest_outfit`

- **What it does:** Pairs the chosen listing with items from the shopper’s wardrobe that match in color and style, and explains why they go together.
- **Inputs:** listing (dict, one item from search_listings), wardrobe (list of dicts with id, name, category, colors, style_tags, notes)
- **Returns:** A dict with the wardrobe items chosen (a list of ids) and a short reason (str).
- **When it has nothing:** If the wardrobe is empty or nothing matches, it returns an empty items list and a reason saying so. The agent still writes the fit card for the listing alone.

### `create_fit_card`

- **What it does:** Writes a short shareable caption for the find and the outfit.
- **Inputs:** listing (dict), outfit (dict from suggest_outfit)
- **Returns:** A caption (str) that names the item, its price and the pairing.
- **When it has nothing:** If the outfit has no wardrobe items, it writes a caption about the listing alone. It never returns an empty string.

---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:** If search_listings returns an empty list, put a “no matches” message in the session and stop. Otherwise take the first result and go to suggest_outfit, then pass its result to create_fit_card.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** <!-- regex, string splitting, or asking the model — say which -->
Price: a regex finds under $30, below 30 or $30 and returns the number as a float. The result is max_price.
Size: a regex finds size followed by a token, so size M gives "M". The result is size.
Description: what’s left after removing those two phrases is the description.
Missing parts: any part not found is None.

**What moves through the session:** <!-- which fields, in what order -->
query -> parsed (description, size, max_price) -> search_results -> selected_item (the first result) -> outfit_suggestion -> fit_card. error stays None on a full run. On the no-match path error is set, and selected_item, outfit_suggestion and fit_card stay None.
---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30, size M'

```
  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Here are two outfit combinations using the new Y2K Butterfly Baby Tee and pieces from your current wardrobe:

### Outfit 1: Y2K Streetwear Contrast
*   **New Item:** Y2K Baby Tee (Butterfly Print)
*   **Bottoms:** Baggy straight-leg jeans (dark wash)
*   **Outerwear:** Vintage black denim jacket (slightly cropped)
*   **Shoes:** Chunky white sneakers
*   **Accessories:** Black crossbody bag

**Why it works:** 
This look plays on the classic Y2K silhouette of balancing a fitted top with loose bottoms. The tight, cropped baby tee highlights your waist, while the low-to-mid-waist, baggy dark-wash jeans add a cool, relaxed streetwear edge. Throwing on the slightly cropped black denim jacket keeps the proportions balanced, and the chunky white sneakers tie in the bright white from the tee.

### Outfit 2: Casual Earthy Soft-Grunge
*   **New Item:** Y2K Baby Tee (Butterfly Print)
*   **Bottoms:** Wide-leg khaki trousers
*   **Shoes:** Black combat boots
*   **Accessories:** Brown leather belt, Black crossbody bag

**Why it works:**
The baby tee's pink and purple butterfly print brings a soft, feminine touch that contrasts nicely with the structured, utilitarian look of the wide-leg khaki trousers. Tucking the fitted tee into the trousers and adding a brown leather belt helps define your waist. Finishing the outfit with black combat boots adds a bit of grunge attitude, bridging the gap between sweet cottagecore details and edgy streetwear.

  Fit card: Manifesting an eternal 2003 with this butterfly print Y2K baby tee I just scored on Depop for $18. The fit is aggressively tiny and the print is giving very much early-2000s pop star running errands. Going to pair it with baggy low-rise denim and chunky sneakers to lean all the way into the vibe.

0 model calls this session, 2 served from cache


**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"

```
[{'id': 'lst_017', 'title': 'Mesh Long-Sleeve Top — Black', 'description': 'Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.', 'category': 'tops', 'style_tags': ['y2k', 'grunge', 'goth', 'layering'], 'size': 'S/M', 'condition': 'excellent', 'price': 15.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_012', 'title': 'Oversized Crewneck Sweatshirt — Vintage Navy', 'description': 'Perfectly faded navy crewneck. Genuinely vintage — not manufactured distressed. Ribbed cuffs and hem. No graphics, clean.', 'category': 'tops', 'style_tags': ['vintage', 'basics', 'oversized', 'classic'], 'size': 'XL (fits oversized)', 'condition': 'good', 'price': 20.0, 'colors': ['navy'], 'brand': None, 'platform': 'thredUp'}, {'id': 'lst_015', 'title': 'Vintage Graphic Hoodie — Faded Black', 'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 26.0, 'colors': ['black', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_011', 'title': 'Low-Rise Cargo Pants — Khaki', 'description': 'Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.', 'category': 'bottoms', 'style_tags': ['y2k', 'cargo', '2000s', 'streetwear'], 'size': 'W29', 'condition': 'fair', 'price': 27.0, 'colors': ['khaki', 'tan'], 'brand': None, 'platform': 'poshmark'}]

```
python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"

```
Here is one great outfit that combines the new Vintage Levi's 501 Jeans with pieces from your wardrobe:

### Outfit: Casual Streetwear Contrast
* **New Item:** Vintage Levi's 501 Jeans (Medium Wash)
* **Top:** White ribbed tank top
* **Outerwear:** Vintage black denim jacket (Slightly cropped)
* **Shoes:** Chunky white sneakers
* **Accessories:** Black crossbody bag & Brown leather belt 

**Why it works:**
The medium wash of the new Levi's 501s adds a brighter, classic blue contrast to your existing dark-wash jeans, giving you a truer vintage streetwear look. Pairing the jeans with the fitted white ribbed tank creates a clean, balanced silhouette under the slightly cropped vintage black denim jacket. Tying it together with the brown leather belt, chunky white sneakers, and black crossbody bag gives you an effortless, everyday outfit that bridges classic denim textures with casual streetwear staples.
```
python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"

```
Found these vintage Levi's 501 jeans on Depop for just $38 and I am never taking them off. They've got that perfect worn-in, effortless 90s slouch that you can't fake. Honestly just gonna live in these with some crisp white sneakers all through spring.
---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* Help writing the `search_listings` entry in my Tool Inventory, including how it handles size.
- *What came back:* Claude first said to match size by "substring or token". It then corrected itself: a substring test is wrong because `"L"` is inside `"XL (oversized)"` and `"W30 L30"`, so a search for size L would return an XL flannel.
- *What I changed:* I match whole size tokens instead, so `"M"` matches `"S/M"` but `"L"` doesn't match `"XL"`. I wrote that rule into my Tool Inventory.

**Moment 2**

- *What I asked for:* The `search_listings` entry for my Tool Inventory, based on the listing and wardrobe fields from `python app.py fields`.
- *What came back:* Claude listed eight filter inputs (`category`, `style_tags`, `size`, `colors`, `max_price`, `condition`, `brand`, `platform`). That was a guess. The real function in `tools.py` takes only three: `description` (str), `size` (str or None) and `max_price` (float or None), and it does keyword matching on the description instead of filtering on each field.
- *What I changed:* I rewrote the Inputs line to the three real parameters, and described the matching as keyword overlap with the description, with size and price as filters.


**Moment 3**

- *What I asked for:* The code for `run_agent`.
- *What came back:* The first version used a more complicated state-driven loop with a flag. I asked Claude to follow the numbered steps in the `agent.py` docstring exactly, and it rewrote the function as those steps. When I ran it, I got `NameError: name 'parse_query' is not defined`, because I had pasted `run_agent` without the `parse_query` helper above it.
- *What I changed:* I added the helper above `run_agent` and tested it alone first. It printed `{'description': 'vintage graphic tee', 'size': 'M', 'max_price': 30.0}`, and then I ran the full query.



<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```

```

**Empty search**

```

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**

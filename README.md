# FitFindr — Starter Kit

This starter kit contains everything you need to begin Project 2.

## What's Included

```
ai201-project2-fitfindr-starter/
├── data/
│   ├── listings.json          # 40 mock secondhand listings
│   └── wardrobe_schema.json   # Wardrobe format + example wardrobe
├── utils/
│   └── data_loader.py         # Helper functions for loading the data
├── planning.md                # Your planning template — fill this out first
└── requirements.txt           # Python dependencies
```

## Setup

```bash
pip install -r requirements.txt
```

Set your Groq API key in a `.env` file (get a free key at [console.groq.com](https://console.groq.com)):
```
GROQ_API_KEY=your_key_here
```

## The Mock Listings Dataset

`data/listings.json` contains 40 mock secondhand listings across categories (tops, bottoms, outerwear, shoes, accessories) and styles (vintage, y2k, grunge, cottagecore, streetwear, and more).

Each listing has: `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, and `platform`.

Load it with:
```python
from utils.data_loader import load_listings
listings = load_listings()
```

## The Wardrobe Schema

`data/wardrobe_schema.json` defines the format your agent uses to represent a user's existing wardrobe. It includes:

- `schema`: field definitions for a wardrobe item
- `example_wardrobe`: a sample wardrobe with 10 items you can use for testing
- `empty_wardrobe`: a starting template for a new user

Load an example wardrobe with:
```python
from utils.data_loader import get_example_wardrobe
wardrobe = get_example_wardrobe()
```

## Where to Start

1. **Read `planning.md` and fill it out before writing any code.**
2. Verify the data loads correctly by running `python utils/data_loader.py`.
3. Build and test each tool individually before connecting them through your planning loop.

Your implementation files go in this same directory. There's no required file structure for your agent code — organize it however makes sense for your design.

## Tool Inventory
1) search_listings(description, size, max_price)
  - description (str): keywords describing the item the user wants
  - size (str | None): size to filter by, or None to skip
  - max_price (float | None): the most amount of money the item can cost, or None to skip
Returns: a list of matching listing dicts sorted by relevance, or an empty list if nothing matches
Purpose: searches the mock listings dataset and returns the most relevant items

2) suggest_outfit(new_item, wadrobe)
  - new_item (dict): the top listing dict returned by search_listings
  - wardrobe (dict): the user's wardrobe containing an items list
Returns: a string with outfit suggestions using the new item and wardrobe pieces, or general styling advice if the wardrobe is empty
Purpose: uses the LLM to generate a specific outfit combination around the new item

3) create_fit_card(outfit, new_item)
  - outfit (str): the suggestion string returned by suggest_outfit
  - new_item (dict): the listing dict for the item being styled
Returns: a string formatted as a casual Instagram caption, or an error message string if outfit is empty
Purpose: uses the LLM to generate a shareable caption for the outfit

## Planning Loop
search_listings runs first with the description, size, and max price parsed from the user's query using regex. If it returns an empty list, the agent sets an error message and returns early — suggest_outfit is never called with empty input.
If results come back, the agent takes the top result and passes it into suggest_outfit along with the user's wardrobe. That result gets stored and passed directly into create_fit_card along with the selected item. The agent is done when session["fit_card"] is set, or when the early return from an empty search fires.

## State Management
State is tracked through a session dictionary initialized at the start of each interaction. After each tool call, the result is stored in the session:

session["parsed"] — description, size, and max price extracted from the raw query via regex
session["search_results"] — full list returned by search_listings
session["selected_item"] — top result, passed as new_item into suggest_outfit
session["outfit_suggestion"] — string returned by suggest_outfit, passed into create_fit_card
session["fit_card"] — final caption string returned by create_fit_card
session["error"] — set if search_listings returns empty, triggers early return

Each tool receives its inputs directly from the session — no values are re-entered by the user between steps.

## Error Handling
ToolFailure modeAgent responsesearch_listingsNo results match the querySets session["error"] to "No listings matched your query. Try a broader description or a higher price cap." and returns earlysuggest_outfitWardrobe is emptyStill runs and returns general styling advice as a string — does not crashcreate_fit_cardOutfit string is emptyReturns an error message string instead of raising an exception
Concrete example from testing: ![Error handling example](assets/error_screenshot.png)

## Spec Reflection
The planning.md spec helped most during the planning loop implementation. Having the conditional logic written out step by step meant the branching in run_agent() matched the spec exactly on the first pass — the early return on empty search results was clear from the spec before any code was written.
One place implementation diverged from the spec was query parsing. The original spec didn't specify how the raw query string would be broken into description, size, and max price. The plan was to handle it in the planning loop but the exact method wasn't defined. Regex ended up being the right call — it's lightweight and works well for the structured inputs the UI produces.

## AI Usage
Instance 1: I gave Claude my Tool 2 spec block from planning.md — inputs, return value, and failure mode — and asked it to implement suggest_outfit using Groq's llama-3.3-70b-versatile. The generated code used item['title'] to format wardrobe items into the prompt. I caught that the wardrobe schema uses item['name'] not item['title'] after reading wardrobe_schema.json, and fixed that before running it.

Instance 2: I gave Claude my planning loop description and architecture diagram from planning.md and asked it to implement run_agent(). The first version called .get() on user_query as if it were a dict, but user_query is a plain string. I caught that mismatch by reading the generated code against the _new_session() signature in the starter file, and rewrote the parsing section using regex instead.
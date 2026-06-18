# FitFindr — planning.md

> Complete this document before writing any implementation code.
> Your spec and agent diagram are what you'll use to direct AI tools (Claude, Copilot, etc.) to generate your implementation — the more specific they are, the more useful the generated code will be.
> Your planning.md will be reviewed as part of your submission.
> Update it before starting any stretch features.

---

## Tools

List every tool your agent will use. For each tool, fill in all four fields.
You must have at least 3 tools. The three required tools are listed — add any additional tools below them.

### Tool 1: search_listings

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
This tool will take in information from the query that, and then use the key information from the query to compare against the loaded listings, to see if they can find any piece of clothing that would meet the users request.

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `description` (str): 
  The description parameter represents, some defining information about the piece of clothing. 
- `size` (str):
  The size parameter represents the sizing of the specific piece of clothing necessary.
- `max_price` (float): 
  The max_price parameter represents the most amount of money the piece of clothing can be for the user to still want to purchase.

**What it returns:**
<!-- Describe the return value — what fields does a result contain? -->
A list of listing dictionaries. In other words, every listing that matches with the search_listing() tool's parameters from the user's query will be returned.

**What happens if it fails or returns nothing:**
<!-- What should the agent do if no listings match? -->
The agent will not use another tool, and instead tells the user what to try differently, so that it can find matched listings.
---

### Tool 2: suggest_outfit

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
The suggest outfit tool takes the top listing, or in other words the one that matches the query the most, and the user's wardrobe to give suggestions based off their wardrobe, or if the user's wardrobe is empty it will give general styling tips.

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `new_item` (dict): 
  A dictionary of the listing that most matches the query.
- `wardrobe` (dict): 
  A dictionary of the users wardrobes, the pieces of clothing that they own.

**What it returns:**
<!-- Describe the return value -->
This tool will return a string that would either contain the styling tips or the suggestion based off their wardrobe.

**What happens if it fails or returns nothing:**
<!-- What should the agent do if the wardrobe is empty or no outfit can be suggested? -->
General styling advice with the piece of clothing will be given.

---

### Tool 3: create_fit_card

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
This tool's primary task is to use the title/name of the outfit and the actual item listing as a parameter to create a instagram caption based off of them. 

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `outfit` (str): 
  A string that is the suggestion from the suggest_outfit() tool.
  A dictionary of the listing that most matches the query.

**What it returns:**
<!-- Describe the return value -->
It will return a string, this string will be the instagram caption.
**What happens if it fails or returns nothing:**
<!-- What should the agent do if the outfit data is incomplete? -->
It will return an error message, NOT an exception.
---

### Additional Tools (if any)

<!-- Copy the block above for any tools beyond the required three -->

---

## Planning Loop

**How does your agent decide which tool to call next?**
<!-- Describe the logic your planning loop uses. What does it look at? What conditions change its behavior? How does it know when it's done? -->
search_listings() is first called and ran. If no listings match, or in other words an empty list is returned, the agent will give the user a suggestion and re-prompt to help them find a listing that matches their query. If there are listings that match, the agent will store the top result as session["selected_item"] and use that listing dictionary as one of the parameters for the suggest_outfit() tool, and the suggest_outfit() tool will be called and ran. If the suggest_outfit() tool's wardrobe parameter is empty, it will still run and return a string with general styling advice for the user rather than crashing. If it is successful, it will return an outfit suggestion based on the clothing pieces in their current wardrobe, and that result gets stored as session["outfit_suggestion"].

Once session["outfit_suggestion"] is set, create_fit_card() is called with the suggestion string and session["selected_item"]. If the outfit string is empty or missing, create_fit_card() returns an error message string rather than raising an exception. The result gets stored as session["fit_card"].

The agent knows it's done when session["fit_card"] is set, or when an empty result from search_listings() triggers an early return.
---

## State Management

**How does information from one tool get passed to the next?**
<!-- Describe how your agent stores and accesses state within a session. What data is tracked? How is it passed between tool calls? -->
I plan to create a dictionary for the specific session, for all the returned values from the tools. The data that is tracked would be the listings, the outfit suggestions, the captions, and the errors. It will be passed between tools as parameters.
---

## Error Handling

For each tool, describe the specific failure mode you're handling and what the agent does in response.

| Tool | Failure mode | Agent response |
|------|-------------|----------------|
| search_listings | No results match the query | The agent tells the user to try a broader description or higher price cap and stops |
| suggest_outfit | Wardrobe is empty | The tool still runs and returns a string with general styling advice for the new item instead of crashing |
| create_fit_card | Outfit input is missing or incomplete | The tool returns an error message string instead of raising an exception |

---

## Architecture

<!-- Draw a diagram of your agent showing how the components connect:
     User input → Planning Loop → Tools (search_listings, suggest_outfit, create_fit_card)
                                                                          ↕
                                                                   State / Session
     Show what triggers each tool, how state flows between them, and where error paths branch off.
     ASCII art, a Mermaid diagram (https://mermaid.js.org/syntax/flowchart.html), or an embedded
     sketch are all fine. You'll share this diagram with an AI tool when asking it to implement
     the planning loop and each individual tool. -->

User query
    │
    ▼
Planning Loop
    │
    ├─► search_listings(description, size, max_price)
    │       │ results=[]
    │       ├──► [ERROR] "Try a broader description or higher price cap" → return
    │       │
    │       │ results=[item, ...]
    │       ▼
    │   selected_item = results[0]
    │       │
    ├─► suggest_outfit(selected_item, wardrobe)
    │       │ wardrobe empty
    │       ├──► returns general styling advice string (no crash)
    │       │
    │       │ wardrobe has items
    │       ▼
    │   outfit_suggestion = "..."
    │       │
    └─► create_fit_card(outfit_suggestion, selected_item)
            │ outfit is empty
            ├──► returns error message string (no crash)
            │
            │ success
            ▼
        fit_card = "..."
            │
            ▼
    Final output to user

---

## AI Tool Plan

<!-- For each part of the implementation below, describe:
     - Which AI tool you plan to use (Claude, Copilot, ChatGPT, etc.)
     - What you'll give it as input (which sections of this planning.md, your agent diagram)
     - What you expect it to produce
     - How you'll verify the output matches your spec before moving on

     "I'll use AI to help me code" is not a plan.
     "I'll give Claude my Tool 1 spec (inputs, return value, failure mode) and ask it to implement
     search_listings() using load_listings() from the data loader — then test it against 3 queries
     before trusting it" is a plan. -->

**Milestone 3 — Individual tool implementations:**

I'm going to implement each tool myself using the spec I wrote in planning.md as my guide. I'll start with search_listings, making sure it filters by all three parameters and returns an empty list when nothing matches. Then suggest_outfit, handling the empty wardrobe case without crashing. Then create_fit_card, making sure it returns an error message string instead of an exception. If I get stuck on anything I'll ask Claude specific questions, but I want to write the code myself so I actually understand what each tool is doing before I wire them together.

**Milestone 4 — Planning loop and state management:**

I'm going to implement run_agent() myself using my planning loop description and architecture diagram as reference. I'll make sure the branching logic actually responds to what search_listings returns rather than calling all three tools unconditionally. If something isn't behaving right I'll ask Claude to help me debug a specific part, but the implementation will be mine.
---

## A Complete Interaction (Step by Step)

Write out what a full user interaction looks like from start to finish — tool call by tool call. Use a specific example query.

**Example user query:** "I'm looking for a vintage graphic tee under $30. I mostly wear baggy jeans and chunky sneakers. What's out there and how would I style it?"

**Step 1:**
<!-- What does the agent do first? Which tool is called? With what input? -->

The first step is to call search_listings("vintage graphic tee", size=None, max_price=30.0), which takes the user's query, size, and max price as parameters. Before that, load_listings() has to run internally so the search actually has listings to compare against. If search_listings() returns nothing, the agent tells the user to try something different like a broader query or higher price cap and stops there without calling anything else.

**Step 2:**
<!-- What happens next? What was returned from step 1? What tool is called now? -->

What comes back from Step 1 is a list of matching listing dictionaries — for example, "Faded Band Tee — $22, Depop, Good condition" as the top result. We take that top result and pass the full listing object as new_item, along with the user's wardrobe object from get_example_wardrobe(), into suggest_outfit(new_item=<band tee>, wardrobe=<user's wardrobe>). This returns a string with outfit suggestions built around both the new piece and what's already in the wardrobe, something like "Pair this with your wide-leg jeans and chunky sneakers for a classic 90s look."

**Step 3:**
<!-- Continue until the full interaction is complete -->
That suggestion string, along with the new_item listing object, gets passed into create_fit_card(outfit=<suggestion>, new_item=<band tee>). This tool returns a social media caption written around the clothing item and the outfit, something like "thrifted this faded band tee off depop for $22 and it was made for my baggy jeans 🖤". If it fails, return a curated error message.


**Final output to user:**
<!-- What does the user actually see at the end? -->
The user sees the top listing details, the outfit suggestion, and the generated caption put together as one holistic response.

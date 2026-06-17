# Classifier Spec — Pod Classifier

Complete this spec **before** writing any code for Milestone 2.

Use Plan or Ask mode to think through each blank field. When you're done,
your answers here become the blueprint for `build_few_shot_prompt()` and
`classify_episode()` in `classifier.py`.

---

## build_few_shot_prompt(labeled_examples, description)

### What it does

Constructs a prompt string for the LLM that includes the task instructions,
all labeled training examples, and the new episode description to classify.

### Inputs

| Parameter          | Type         | Description                                                                                                          |
| ------------------ | ------------ | -------------------------------------------------------------------------------------------------------------------- |
| `labeled_examples` | `list[dict]` | Each dict has `"title"`, `"description"`, `"label"` (and others). These are the examples you labeled in Milestone 1. |
| `description`      | `str`        | The episode description to classify.                                                                                 |

### Output

| Return value | Type  | Description                                        |
| ------------ | ----- | -------------------------------------------------- |
| prompt       | `str` | A complete prompt string ready to send to the LLM. |

---

### Spec fields — fill these in before writing code

**Task instruction (what should the LLM know about the task?):**

```
You are classifying podcast episodes by their format. Classify the episode
into exactly one of these four labels:

- interview: a conversation between a host and one or more guests
- solo: a single host speaking from memory, experience, or opinion — no guests,
  no assembled external sources
- panel: multiple guests with roughly equal speaking time, often debating or
  discussing a topic together
- narrative: a story assembled from external sources — interviews, archival
  audio, reporting — with a clear narrative arc

Return only the label and your reasoning. Do not explain the taxonomy.
```

---

**How should labeled examples be formatted in the prompt?**

```
Each example should include the episode title, a brief excerpt or the full
description, and the correct label. Separate examples with a blank line or
a delimiter like "---". Include all fields that help the model see why the
label was applied — title and description are both useful; other fields
(like episode ID) are not needed.
```

---

**Example block sketch (write one concrete example):**

```
Title: {title}
Description: {description}
Label: {label}
```

---

**How should the new episode (to be classified) be presented?**

```
Present it in the same format as the labeled examples, but omit the Label
line and replace it with an instruction to classify. For example:

Title: {title}
Description: {description}
Label: ?

Then add a line like: "Classify the episode above. Return your answer in
the format below:" followed by the output format you chose.
```

---

**What output format should you request from the LLM?**

```
The output format from the LLM should look something like this:

{
  "label": "...",
  "reasoning": "..."
}
```

---

**Edge cases to handle in the prompt:**

```
If labeled_examples is empty, omit the examples block entirely — the model
classifies zero-shot using only the task instruction and label definitions.

If description is very short (e.g., one sentence), include it as-is. Do not
pad or annotate it. The model can often classify confidently from minimal text;
if not, it will reflect that uncertainty in the reasoning field.
```

---

## classify_episode(description, labeled_examples)

### What it does

Classifies a single podcast episode description using the few-shot LLM classifier.
Returns a dict with a label and reasoning.

### Inputs

| Parameter          | Type         | Description                                               |
| ------------------ | ------------ | --------------------------------------------------------- |
| `description`      | `str`        | The episode description to classify.                      |
| `labeled_examples` | `list[dict]` | Labeled training examples from `load_labeled_examples()`. |

### Output

| Return value | Type   | Description                                                                                         |
| ------------ | ------ | --------------------------------------------------------------------------------------------------- |
| result       | `dict` | Must have keys `"label"` and `"reasoning"`. `"label"` must be one of `VALID_LABELS` or `"unknown"`. |

---

### Spec fields — fill these in before writing code

**Step 1 — Build the prompt:**

```
Call build_few_shot_prompt(labeled_examples, description) and store the
returned string in a variable (e.g., prompt). Pass through both arguments
exactly as received — no modification needed before calling.
```

---

**Step 2 — Send to the LLM:**

```
Call _client.chat.completions.create() with:
  - model: the model name from config (LLM_MODEL)
  - messages: a list with one dict — {"role": "user", "content": prompt}
    (system-design.md shows an optional system message too — either shape works)
  - max_tokens: a reasonable limit (e.g., 200–300) to keep responses concise

Extract the response text from:
  response.choices[0].message.content
```

---

**Step 3 — Parse the response:**

````
The LLM returns a JSON object with "label" and "reasoning" keys. Parse it with:

1. Call json.loads() on the response text to get a dict.
2. Extract result["label"] and result["reasoning"].

The model may wrap the JSON in markdown code fences (```json ... ```).
If json.loads() raises a JSONDecodeError, try stripping the fences first:
  - Find the first "{" and last "}" in the response text
  - Slice to that range, then call json.loads() again
If parsing still fails, raise the exception — Step 5 (error handling) catches it
and returns {"label": "unknown", "reasoning": "..."}.
````

---

**Step 4 — Validate the label:**

```
If the label isn't in VALID_LABELS, the label should be set to "unknown"
```

---

**Step 5 — Handle errors gracefully:**

```
Wrap the entire function body in a try/except Exception block.

Things that can go wrong:
- Network/API error: the HTTP request to the LLM fails or times out
- JSONDecodeError: the response isn't valid JSON even after fence-stripping
- KeyError: the JSON parsed but is missing "label" or "reasoning"
- Invalid label: handled in Step 4, not here

On any exception, return:
  {"label": "unknown", "reasoning": "Classification failed: <str(e)>"}

This ensures the evaluation loop continues through all 20 episodes even if
one call fails — a single bad response becomes an "unknown" result, not a crash.
```

---

### Return value structure

```python
{
    "label": str,      # one of VALID_LABELS, or "unknown" if invalid/error
    "reasoning": str,  # brief explanation from the LLM
}
```

---

## Notes on label quality

The classifier is only as good as your labels. If your training examples have
inconsistent or ambiguous labels, the LLM will learn the wrong pattern.

Before implementing the classifier, re-read `data/taxonomy.md` and double-check
any labels you're unsure about. Annotation quality is part of the lab.

---

## Implementation Notes

_Fill this in after implementing and testing both functions._

**Test: what does the raw LLM response look like for one episode?**

```
Episode tested: Food I ate while abroad
My episode desc: I take time to talk about the best restaurants I went to while studying abroad.
This is what I got back:
{"label": "solo", "reasoning": "The episode is a personal reflection by the host about their experiences and opinions, with no guests or external sources mentioned, which fits the characteristics of a solo episode."}

```

**How did you parse the label out of the response?**

```
My information came back to me in a JSON, so it was able to be parsed thanks to the
"label" before the actual value.
```

**Did any episodes return `"unknown"`? If so, why?**

```
I did not get any unknowns.
```

**One thing about the output format that surprised you:**

```
It was surprisingly clear to read, and the reasonings behind the labels made more sense than I
thought I would get back
```

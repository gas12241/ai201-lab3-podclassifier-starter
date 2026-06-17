# Evaluation Spec — Pod Classifier

Complete this spec **before** writing any code for Milestone 3.

Use Plan or Ask mode to think through each blank field. When you're done,
your answers here become the blueprint for `compute_accuracy()` and
`compute_per_class_accuracy()` in `evaluate.py`.

---

## Background: What is evaluation?

After building a classifier, we need to know how well it works. Evaluation answers:

- **Overall:** What fraction of episodes did we classify correctly?
- **Per-class:** Are we better at some labels than others?

Both functions take the same inputs: a list of predicted labels and a list of
ground-truth labels, in the same order.

---

## compute_accuracy(predictions, ground_truth)

### What it does

Returns the fraction of predictions that exactly match the ground truth.

### Inputs

| Parameter      | Type        | Description                                                |
| -------------- | ----------- | ---------------------------------------------------------- |
| `predictions`  | `list[str]` | Labels predicted by `classify_episode()`, one per episode. |
| `ground_truth` | `list[str]` | The correct labels, in the same order as `predictions`.    |

### Output

| Return value | Type    | Description                  |
| ------------ | ------- | ---------------------------- |
| accuracy     | `float` | A value between 0.0 and 1.0. |

---

### Spec fields — fill these in before writing code

**Formula:**

```
Accuracy is the number of correct predictions divided by the number of total predictions.
Correct predictions are ones where the label matches the correct one.
```

---

**Step-by-step logic:**

```
1. if both lists are empty, raise a ValueError
2. Count the number of correct predictions by iterating over both lists together
   (zip(predictions, ground_truth)) and counting pairs where pred == truth.
3. Divide the correct count by the total number of predictions (len(predictions)).
4. Return the result as a float.
```

---

**Edge case — what if both lists are empty?**

```
If both lists are empty, raise a ValueError. Accuracy is undefined with no data,
and an empty list most likely means something went wrong upstream (e.g., no episodes
were loaded). Failing loudly is better than silently returning a misleading value.
```

---

**Worked example:**

```
predictions  = ["interview", "solo", "panel", "interview"]
ground_truth = ["interview", "solo", "solo",  "narrative"]

If we are going by the equation:
accuracy = correct_predictions / total_predictions

where a correct prediction is one where predictions[i] == ground_truth[i]

Then we would have the equation:
accuracy = 2 / 4
which is equal to 0.5

Since at index 0 and 1, predictions and ground_truth share the same elements, but there are 4 total predictions happening.
```

---

## compute_per_class_accuracy(predictions, ground_truth)

### What it does

Returns accuracy broken down by each label. For each label in `VALID_LABELS`,
reports how many episodes with that ground-truth label were classified correctly.

### Inputs

| Parameter      | Type        | Description                               |
| -------------- | ----------- | ----------------------------------------- |
| `predictions`  | `list[str]` | Labels predicted by `classify_episode()`. |
| `ground_truth` | `list[str]` | Correct labels, in the same order.        |

### Output

A `dict` keyed by label. Each value is a dict with three keys:

```python
{
    "interview": {"correct": int, "total": int, "accuracy": float},
    "solo":      {"correct": int, "total": int, "accuracy": float},
    "panel":     {"correct": int, "total": int, "accuracy": float},
    "narrative": {"correct": int, "total": int, "accuracy": float},
}
```

---

### Spec fields — fill these in before writing code

**What does "correct" mean for a given class?**

```
Correct for a given class means how many of a certain label were labeled correctly. For example, if there
are 20 ground truths, and 6 of those were labeled as "solo". And of those 6, only 2 were labeled correctly,
then for the given class of "solo", there were 2 correct.
```

---

**What does "total" mean for a given class?**

```
Total for a given class means how many ground truths for a specific label there were. For example, if there
were 20 predictions to be made, and of those 20, 6 had a ground_truth of "Solo". Then for the given class of
"solo" the total would be 6.
```

---

**Step-by-step logic:**

```
1. Initialize a results dict with an entry for each label in VALID_LABELS,
   each set to {"correct": 0, "total": 0, "accuracy": 0.0}.

2. Loop over predictions and ground_truth together using zip().

3. For each pair (predicted, truth):
   - Increment results[truth]["total"] by 1 (this episode belongs to the truth class).
   - If predicted == truth, also increment results[truth]["correct"] by 1.

4. After the loop, compute accuracy for each label:
   - If total == 0, set accuracy to 0.0 (no examples of this class were tested).
   - Otherwise, set accuracy = correct / total.

5. Return the results dict.
```

---

**Edge case — what if a class has no examples in ground_truth (total == 0)?**

```
Set accuracy to 0.0. Dividing by zero is undefined, and 0.0 is a safe sentinel
that signals the class was never tested — consistent with the docstring in evaluate.py
which specifies "0.0 if total is 0".
```

---

**Worked example:**

```
predictions  = ["interview", "interview", "solo", "panel", "panel"]
ground_truth = ["interview", "solo",      "solo", "panel", "narrative"]

label       correct  total  accuracy
----------  -------  -----  --------
interview   [1]  [1]  [1.0]
solo        [1]  [2]  [0.5]
panel       [1]  [1]  [1.0]
narrative   [0]  [1]  [0.0]
```

---

## Reflection questions (discuss at the checkpoint)

1. Your overall accuracy might be decent even if one class has very low accuracy.
   Why is per-class accuracy a more informative metric than overall accuracy alone?

2. If `panel` episodes consistently get misclassified as `interview`, what does
   that tell you about your training labels or your prompt?

3. You labeled 20 training episodes and evaluated on 20 test episodes (5 per class).
   How might the evaluation results change if you had labeled 100 training episodes?
   What if you had 200 test episodes?

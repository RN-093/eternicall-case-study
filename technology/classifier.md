# The classifier pattern

A classifier reads one input against written rules and context, and returns a typed output. Code then acts on that output.

![The classifier pattern: rules, context and input go into the classifier, which returns a typed output that drives an alert and a log](../diagrams/classifier-pattern.svg)

## The five parts

**Rules and context.** The rules are the criteria that decide which label applies, written in plain language in the prompt. They also say what the summary should cover and how to treat anything the input does not state. Context is the reference material the rules rely on. At EterniCall it was per-client detail written into the prompt. It could as easily be text files or any other source.

**Classifier.** A general language model, instructed to answer in a fixed structure.

**Input.** Can be voice, text or image, however images have not been tested.

**Typed output.** The same shape every time:

```json
{
  "label": "urgent class 1",
  "summary": "<a short account of what the input says and what is needed>",
  "<extracted field>": "<a detail lifted from the input, or empty if it was not given>"
}
```

Three things held it together:

- Every field is always returned, even when empty.
- The label is exactly one of a small fixed set. EterniCall had three: urgent class 1, urgent class 2 and non-urgent class 1.
- Nothing is invented. A detail the input does not give stays empty.

The label is chosen. The summary is written. The extracted fields are lifted from the input.

**Action.** Code reads the label and decides what happens. An urgent label can send an alert, carrying the summary and the label, to the person who needs to act, or forward a task to a specific member of the team. Every input is written to the log regardless of its label.

## How it maps to Jev

| Jev | This classifier |
| --- | --- |
| **State**: the text the model judges | **Input**: the text to classify |
| **Instructions and criteria** that define the possible answers | **Rules and context** that define the possible answers |
| **Choice**: one option from a defined set, with a probability for each option | **Label**: one of any number of fixed values, with no probabilities |
| No equivalent. Jev does not generate text | **Summary**, written in the same step as the label |

The shape is the same. Both hand code a constrained answer instead of free text, and both leave the action to code.

The differences run in both directions. TypeSafe describes Jev as a model trained specifically for decisions, with calibrated probabilities and very fast responses. This classifier has none of that. What it does have is the summary, which Jev cannot write because it does not generate text.

## Swap the rules

The rules and context are the only part tied to a job, so the same pattern should carry to another one by rewriting them. The example below is illustrative. It was not built or tested.

A plumbing firm receives enquiries through its website. Its rules define three labels:

| Label | When it applies | Action |
| --- | --- | --- |
| `boiler issue` | A boiler has failed or is unsafe | Alert to the on-call engineer, and logged |
| `new install` | The customer wants a new system fitted | Logged for the sales team |
| `wants quote` | The customer wants a price for any other work | Logged for the office |

A customer named Sam writes that their boiler stopped overnight, there is no heating, and they have a newborn at home.

### Before

Ask a language model what to do with Sam's message and it answers in prose:

> This sounds like an urgent boiler problem. Sam says the boiler stopped working overnight and there is no heating, which is a concern with a newborn in the house. I would send an engineer out as soon as possible, and it may be worth calling Sam back to confirm the address.

A person can act on that. Code cannot. The decision is buried in a sentence, and it will be worded differently every time.

### After

With the rules and a fixed shape, the same message comes back as a typed output:

```json
{
  "label": "boiler issue",
  "summary": "Sam's boiler stopped overnight and the house has no heating. There is a newborn at home. Sam wants an engineer as soon as possible.",
  "customer_name": "Sam"
}
```

Code reads `boiler issue` and alerts the on-call engineer. Nobody had to read the message first.

## What happens when

| Situation | Typed output | What happens |
| --- | --- | --- |
| The input states everything plainly | A label, a summary, and every field filled | The action for that label runs |
| A detail is missing, such as a name | That field is left empty, never guessed | The action still runs, and the log shows the gap |
| The input fits no label clearly | Still exactly one label. The rules name one label as the default for anything that fits no other | Logged under the default label |
| The label is urgent | Label and summary | An alert goes to the person who needs to act, and the input is logged |
| The label is not urgent | Label and summary | Logged only |

## Other inputs

Text and voice (through transcription) were the inputs in use. Images and other input types are possible with this approach, but none were built or tested before the sale.

## Limits

- **No probabilities.** The label arrives with no measure of confidence, so there is no way to hold back a doubtful input for a person to check.
- **It can be wrong.** A general language model can pick the wrong label or return output that does not match the shape. Neither was reported in testing, but nothing in the pattern rules them out.
- **Accuracy was never measured.** The classifier went into testing with one test case per label. Labels from real use were not audited systematically.
- **Rules were typed by hand.** Each client's rules and context were written into the prompt manually. That was quick at small scale and would not hold at a larger one.
- **It is not fast.** It ran after the event, where a few seconds did not matter. It would not suit a decision needed in real time.

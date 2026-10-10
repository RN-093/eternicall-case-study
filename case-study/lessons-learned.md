## What worked

**1. A small, fixed set of labels.** Three labels meant the code after the classifier never had to interpret anything. It checked one value and acted.

**2. Empty beats guessed.** The rules told the classifier to leave a field empty when the caller had not given the detail. A blank name is a small gap. An invented one sends somebody after the wrong person.

**3. Classify after the call, not during it.** The live call stayed fast because none of this ran inside it. The classifier could then take its time over the full transcript.

**4. A small, low-cost model was enough.** Reading one transcript and filling four fields is not a hard task. The step did not need a large model.

## What I would change

**5. Feed the rules from live records.** Each client's rules were typed into the prompt. I chose that because the scale was small and the alternative would have taken longer to build. However, upon reflection, I would've done things differently. Rules drawn automatically from the client's current records would have removed the manual upkeep, and I would build it that way from the start.

**6. One convention for missing values.** The rules described a missing value in more than one way. It caused no fault, but anything reading the output had to allow for each variant. Pick one and state it once.

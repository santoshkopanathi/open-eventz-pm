# Day 2 — Context Engineering, Model Selection & AI Economics

*AI PM speaking sprint · Wednesday 2026-10-21 · Examples: Open Eventz (OE) and T-Mobile EHS/NBA*

**Goal for today:** be able to explain how you decide *what the model sees*, *which model to use*, *what to check before switching*, and *what it costs*, with a real example from each product.

*Prices below are Anthropic's published list prices as of October 2026. Open Eventz cost figures are rough estimates from prompt length, not billing data. Say "roughly" when you quote them.*

---

## 1. Context engineering

**Plain language:** Context engineering is designing everything the model sees on each call: instructions, the data for this request, any tools it can use, memory of earlier steps, and retrieved documents. It replaced "prompt engineering" as the name because the prompt is now only one part of it.

**The five principles to say out loud:**
1. **Minimal:** the smallest set of high-signal information that lets the model make *this* decision.
2. **Assembled on demand:** fetch facts when the task needs them; don't preload everything.
3. **Structured in, structured out:** clear sections going in, fixed fields coming out.
4. **Traceable:** know where each piece of context came from and how fresh it is.
5. **Versioned and measured:** a context change is a product change, so it goes through an eval.

| | Open Eventz | EHS/NBA |
|---|---|---|
| **Instructions** | One fixed prompt (~870 words): kid-relevance, age buckets, price rules, two confidence scores, JSON only | Extraction instructions on the Scale platform: intent, summary, sentiment, churn |
| **Per-request data** | One event's title and description | The call transcript so far |
| **What's deliberately left out** | Other events, venue history, anything structured (handled by code) | Account data, offers, eligibility (handled by Pega) |
| **Output** | Fixed fields plus a one-line reason | Intent, sentiment and churn scores, plus a summary |
| **Biggest lesson** | Removing one line ("community events are usually free") cut agreement with human labels from 71% to 37%. Context carries judgment, not just facts | When to read the transcript (~130s) is itself a context decision: more context vs. time left to act |

---

## 2. Model selection

**Plain language:** Choosing a model is a product decision, not a technical default. You pick the cheapest, fastest model that meets the quality bar *on your own task*, and you prove it with an eval, not a leaderboard.

**The criteria, in the order a PM should weigh them:**
1. **Quality on your task**, measured on your own examples. Weight errors by their cost.
2. **Cost** per request and at your real volume.
3. **Latency**, if a person is waiting for the answer.
4. **Reliability:** does it follow the output format every time? Is its confidence honest?
5. **Context size**, only if your inputs are large.
6. **Platform and data rules:** where it runs, data retention, privacy, contracts.

**The model tiers (Anthropic, October 2026 list prices, per million tokens):**

| Tier | Current model | Input | Output | Typical use |
|---|---|---|---|---|
| Small, fast | Haiku 5.5 | $0.10 | $0.50 | High-volume classification, routing, simple extraction |
| Balanced | Sonnet 5.5 | $2 | $10 | Most product work, agents, judgment calls |
| Large | Opus 5.5 | $4 | $20 | Hard reasoning, complex agents, coding |
| Most capable | Fable 5.1 | $10 | $50 | The hardest long-running tasks |

**OE:** you chose Sonnet over Haiku because, at your volume, the cost difference was trivial, and the confidence score drives what parents see. You wrote the rationale down so it could be revisited when volume or prices change. *That revisit is now due* (see section 4).
**EHS/NBA:** model choice sat with the Scale platform, and the constraint was real-time: the answer had to reach the agent during the call. In enterprise settings, the platform, data rules and vendor contract often narrow the choice before quality and cost do.

**Routing (a common follow-up):** send easy requests to a small model and hard ones to a larger one. Measure the simple option first: one capable model at lower effort is often as good, and simpler to run.
- *OE, hypothetical:* Haiku classifies first; anything it marks low-confidence goes to Sonnet.

---

## 3. What to check before switching models

**Plain language:** A new model is a new product component. Treat the switch like a release: same eval, side-by-side comparison, then a controlled rollout.

**The checklist:**
1. **Re-run the eval.** Compare error *directions*, not just accuracy. In OE: does wrongly-free stay at zero? Does over-claiming change?
2. **Re-tune the prompt.** Newer models often need less instruction. Prompts written for an older model can over-constrain a newer one.
3. **Check the output format.** Does it still return valid JSON with every required field, every time?
4. **Re-check confidence calibration.** A new model may be more or less confident on the same input.
5. **Re-measure cost.** Token counts can change with a new tokenizer, and new defaults (like built-in reasoning) can add output tokens. Per-token price alone isn't the answer.
6. **Check latency,** if a person is waiting.
7. **Decide what happens to stored answers.** Do you reclassify old records or only new ones?
8. **Record which model produced each answer,** so you can trace and roll back.
9. **Roll out gradually:** shadow-run the new model alongside the old one, compare, then switch. Keep a fallback.

**OE, concretely:** the classifier runs on Sonnet 4.6 ($3 / $15). Sonnet 5.5 is newer *and* cheaper ($2 / $10). That makes a real switching decision, and the honest answer is: "Not until the eval passes." Two gaps to fix first:
- Stored answers are cached forever and aren't tagged with the model, so after a switch, old and new answers would mix with no way to tell them apart.
- The eval's inputs aren't frozen yet (that's the build in the other thread).

**EHS/NBA:** a model change on the vendor side would change intent and sentiment *upstream of Pega*. Because Pega's rules sit downstream, a worse model can't create an offer the rules don't allow, but it can push the right offer down the list. So the check is "did recommendation relevance and FCR hold?", not just "is the extraction similar?"

---

## 4. AI economics

**Plain language:** AI cost is per use, not per licence. Every call costs input tokens plus output tokens. The number that matters is **cost per successful task**: a cheap call that's wrong, or needs a retry, isn't cheap.

**The levers, cheapest first:**
1. **Don't call the model** when code can answer (structured data, rules).
2. **Call once per item and store the answer** (caching results).
3. **Send fewer tokens:** shorter instructions, no unneeded context, short outputs.
4. **Prompt caching:** a repeated prefix costs about a tenth on later reads.
5. **Batch processing:** about 50% cheaper when nobody is waiting for the answer.
6. **A smaller model or lower effort**, only after the eval says quality holds.

**OE, rough per-event cost** (about 1,500 tokens in, 150 out; estimate):

| Model | Per event | Per 1,000 new events |
|---|---|---|
| Sonnet 4.6 (today) | ~$0.007 | ~$7 |
| Sonnet 5.5 | ~$0.005 | ~$4.50 (before any extra reasoning tokens) |
| Haiku 5.5 | ~$0.0002 | ~$0.23 |

What already keeps it low: deterministic logic for library sources, classify-once caching (re-runs cost $0), and a per-run spend cap.
Two untapped levers: the nightly job is a textbook **batch** candidate (nobody waits overnight), and the fixed instructions are a natural **prompt-cache** candidate.

**The real point about OE's economics:** the cost is small, so accuracy decided the model. That's a deliberate choice you can explain, not an accident.

**EHS/NBA economics:**
- **Cost scales with call volume.** Every analyzed call pays for the transcript tokens, across a very large agent population.
- **Batch doesn't apply,** because the answer is needed during the call.
- **The ROI case is on the value side,** and it's built from your metrics. Handle-time minutes saved × calls × cost per agent minute, plus avoided repeat calls from better first-call resolution. Compare that against inference and platform cost per call.
- *Interview tip:* explain the ROI formula, then give real numbers only if you have them.

---

## Self-check (answer out loud)

1. What is context engineering, and how is it different from prompt engineering?
2. How do you choose a model? Walk through your criteria.
3. Should Open Eventz switch from Sonnet 4.6 to Sonnet 5.5? What would you check?
4. What's "cost per successful task", and why is it better than cost per call?
5. Name three ways to cut AI cost without changing the model.
6. How would you build the ROI case for EHS/NBA?

---

## Model answers

**1. What is context engineering, and how is it different from prompt engineering?**
Prompt engineering is writing good instructions. Context engineering is designing everything the model sees on each call: instructions, the data for that request, tools, memory and retrieved material. The goal is the smallest set of information that lets the model make the decision well.
- **Open Eventz:** the model sees fixed instructions plus one event's text, nothing else. Structured data never reaches it, because code handles it. And one line of context turned out to carry real judgment: removing it dropped agreement from 71% to 37%.
- **EHS/NBA:** the model saw the transcript, not account or offer data, which stayed with Pega. Even *when* to send the transcript was a context decision: about 130 seconds, so there was enough conversation and still time to act.

**2. How do you choose a model?**
Quality on my own task first, measured with an eval and weighted by error cost; then cost at real volume; then latency if someone is waiting; then reliability of the output format and confidence. I pick the cheapest model that clears the bar, and I write the reasoning down so it can be revisited.
- **Open Eventz:** Sonnet over Haiku, because the cost difference was trivial at my volume and the confidence score decides what parents see. Accuracy won.
- **EHS/NBA:** real-time was the hard constraint: the signal had to arrive during the call. And in an enterprise, the platform and data rules narrow the choice before you compare quality.

**3. Should Open Eventz switch from Sonnet 4.6 to Sonnet 5.5?**
Probably, because it's newer and cheaper per token, but only after the eval passes. I'd check that wrongly-free stays at zero and over-claiming doesn't rise; that the JSON is still valid every time; that confidence is still honest; and that real cost per event actually falls, since built-in reasoning can add tokens. Before switching I'd tag every stored answer with the model that produced it, so old and new answers don't mix.
- **EHS/NBA parallel:** a vendor-side model change would shift intent and sentiment upstream of Pega. The rules stop it from creating a bad offer, but not from ranking the right offer lower. So I'd watch recommendation relevance and FCR, not just extraction similarity.

**4. What's "cost per successful task"?**
It's total cost divided by tasks done correctly, including retries, fallbacks and human clean-up. A model that's half the price per call but wrong twice as often isn't cheaper.
- **Open Eventz:** a wrong "free" label strands a family at the gate, so a cheaper model that produces even one wrongly-free label costs more than it saves.
- **EHS/NBA:** a recommendation the agent ignores still cost the inference. The real unit is a call resolved first time, not a card shown.

**5. Name three ways to cut AI cost without changing the model.**
Don't call the model when code can answer; call once per item and store the result; and send fewer tokens. Beyond that: prompt caching for repeated instructions, and batch processing when nobody is waiting.
- **Open Eventz:** library ages are handled in code, events are classified once and cached, and there's a spend cap. The nightly run could also move to batch processing at roughly half the price.
- **EHS/NBA:** batch doesn't apply in real time, but you can control how often you re-analyze the call, and how much transcript you send each time.

**6. How would you build the ROI case for EHS/NBA?**
Value side: handle-time minutes saved × call volume × cost per agent minute, plus repeat calls avoided through higher first-call resolution. Cost side: inference and platform cost per analyzed call, plus integration and running costs. I'd also show it was measured through a phased beta before scaling. Use real figures only if you have them.
- **Open Eventz contrast:** ROI here isn't about money. The cost is a few dollars per thousand events, so the decision is about trust: accuracy is worth paying for.

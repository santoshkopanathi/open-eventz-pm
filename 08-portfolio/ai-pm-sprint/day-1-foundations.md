# Day 1 — AI Foundations, Through Your Own Products

*AI PM speaking sprint · Tuesday 2026-10-20 · Examples: Open Eventz (OE) and T-Mobile EHS/NBA*

**Goal for today:** explain each concept below in two sentences of plain language, then give one example from your own work. If you can do that, you can hold the conversation.

---

## 1. How an LLM works

**Plain language:** A large language model is a system trained on a huge amount of text to predict the next piece of text. Ask it a question and it writes the answer one small piece at a time, each time choosing a likely next piece given everything so far. After that basic training, it's tuned to follow instructions and be helpful.

**Three consequences a PM must know:**
- **It doesn't look things up.** Unless you give it facts at run time, it answers from patterns learned in training, which stop at a cutoff date.
- **It's probabilistic.** The same input can produce slightly different output. Design for variation; don't assume repeatability.
- **It produces plausible text, not verified truth.** Fluency isn't evidence of accuracy.

**OE:** for each new event, Claude Sonnet gets the instructions plus the event's title and description, and writes back a small JSON answer: for kids or not, age bucket, free or paid, confidence, and a one-line reason.
**EHS/NBA:** the LLM read the live call transcript and wrote back structured signals: intent, summary, sentiment, churn risk. Pega made the decision.

> **2-minute answer, "How does ChatGPT work?"**
> "It's a model trained on a huge amount of text to predict what comes next. When you ask it something, it writes the answer piece by piece, each piece being the most likely continuation of everything before it. Then it's tuned so those predictions follow instructions. Two things follow for a PM. First, it doesn't look things up unless you give it the facts, so grounding matters. Second, it's built to sound right, not to be right, so you measure it. In my products I use it for what it's best at, understanding messy language, and let deterministic systems make the decisions."

---

## 2. Tokens

**Plain language:** Models read and write in tokens, chunks of text averaging about three-quarters of an English word. Tokens are the unit of **cost** (you pay for tokens in and tokens out), **speed** (more output tokens means a slower answer), and **capacity** (the context window is measured in tokens).

**OE:** the classification instructions are about 870 words, roughly 1,300 tokens (a rough estimate). That's most of each call's input; the event text is the smaller part. Because each event is classified once and cached, re-running the pipeline costs nothing.
**EHS/NBA:** the transcript grows every second of the call. Waiting longer means more tokens, which means more cost per call, multiplied across a very large agent population.

---

## 3. Context window

**Plain language:** The context window is everything the model can see in one call: instructions, data, conversation so far. It's working memory with a size limit, and **nothing carries over between calls** unless you send it again.

**Bigger isn't automatically better.** Answer quality tends to drop as you fill the window with less relevant material, sometimes called context rot. The PM's job is choosing *what goes in*, not *how much fits*.

**OE:** every call is fresh. The model sees one event, never the others, so one event's text can't leak into another's answer.
**EHS/NBA:** the ~130-second re-decision point is a context-window decision. Too early and the model hasn't heard enough to read intent; too late and the agent has no time left to act. That's a **timing vs. accuracy** trade-off you made with data, and pipeline latency (transcription → Kafka → Scale → Pega) shrinks the window further.

---

## 4. Hallucination

**Plain language:** A hallucination is when the model states something false with confidence. It isn't lying; it's producing the most plausible-sounding answer, and sometimes plausible is wrong.

**Common forms:** inventing facts, misreading the input, and being more confident than it should be.

**How products reduce it:**
| Technique | OE example | EHS/NBA example |
|---|---|---|
| Constrain the output | Fixed choices only (free / paid / unknown); never a dollar amount Mostly structured signals (intent; sentiment and churn risk as scores). The free-text summary informs, but Pega's rules decide |
| Ground it in source text | The model only sees the event text | The model only sees the transcript |
| Ask for confidence and a reason | Two confidence scores plus a one-line reason, stored | — |
| Let a governed system decide | Code overrides: listed cost beats the model; "21+" always hides | Pega applies eligibility and rules |
| Keep a human on the decision | Safety info from official policy | The agent chooses whether to act |
| Measure it | 64-event eval; over-confidence beat under-confidence 12 to 2 | Extraction quality evaluated with the Scale team |

---

## 5. Prompt engineering

**Plain language:** The prompt is the instructions you send with every request. Good prompts give a clear role, explicit rules, defined categories, the output format, and (optionally) worked examples.

**OE:** rule-based, no examples ("zero-shot"), JSON only. The key lesson: a prompt carries judgment, not just instructions. Removing one line ("community events are usually free") cut agreement with human labels from **71% to 37%**. Prompt changes need an eval, every time.

---

## 6. Embeddings and vector databases

**Plain language:** An embedding turns text into a list of numbers that represents its *meaning*. Texts with similar meanings get similar numbers, even with no words in common ("messy play for toddlers" sits close to "sensory activities for under-5s"). A vector database stores those numbers and quickly finds the nearest matches.

**Typical uses:** search by meaning, finding duplicates, grouping similar items, and retrieval for RAG.

**OE:** not used today, and that's deliberate. Filters are structured (city, age, date, price). Two places it could help: spotting the same event published by two sources, and a natural-language search box.

---

## 7. RAG (retrieval-augmented generation)

**Plain language:** RAG means *find the relevant information first, then give it to the model with the question*, so it answers from your data instead of from memory. It's how you ground a model in knowledge that is private, changes often, or is too large to fit in the prompt.

**When you need it:** the model must answer open-ended questions over a body of knowledge it can't see all at once (policies, product docs, account history).
**When you don't:** everything the model needs for the decision is already in the input.

**OE doesn't use RAG, and here's why:** each decision is about one event, and that event's text *is* the complete evidence. The nightly scraper does the "retrieval" before the model is ever called.
**EHS/NBA is a grounding example:** the transcript (and account context on the Pega side) is supplied at run time, so the model works from this customer's actual call, not general knowledge. Same principle as RAG, even without a vector database.

**Key point for interviews:** RAG doesn't have to mean a vector database. Retrieval can be a database query. Friday's build, natural-language search, will be exactly that: the model turns the user's sentence into filters, and the database returns real events. The model never makes up an event.

---

## Self-check (answer out loud, 30 seconds each)

1. Why can't you rely on an LLM to know today's events?
2. What are tokens, and name the three things they drive.
3. Why is a bigger context window not automatically better? Give your EHS/NBA example.
4. Name three ways your products reduce hallucination.
5. Why doesn't Open Eventz use RAG? When would it need it?
6. What's an embedding, and where could Open Eventz use one?

---

## Model answers (about 30–45 seconds each, spoken)

Each answer: the core point, then one Open Eventz example and one EHS/NBA example. Items marked *hypothetical* are illustrations, not claims about what was built.

**1. Why can't you rely on an LLM to know today's events?**

A model only knows what was in its training data, which stops at a cutoff date, and it doesn't look anything up on its own. Without the facts in the request, it either refuses or makes up something plausible. So the product has to supply the facts, and the model's job is to interpret them.

- **Open Eventz:** The model is never asked "what's on this Saturday?" A nightly pipeline collects the real events from four sources, and the model only reads one event's text at a time to judge whether it suits kids.
- **EHS/NBA:** The model knew nothing about this customer or this call. It was given the live transcript, so it could tell that a call routed as "billing" was really about roaming charges. The customer's account context sat on the Pega side.

**2. What are tokens, and what three things do they drive?**

Tokens are the chunks models read and write, about three-quarters of a word each. They drive cost, because you pay per token in and out; speed, because more tokens take longer; and capacity, because the context window is measured in tokens.

- **Open Eventz:** The instructions are about 1,300 tokens and make up most of each call. Each event is classified once and cached, so re-running the pipeline costs nothing, and a per-run spend cap limits the worst case.
- **EHS/NBA:** The transcript grows with every second of the call, so waiting longer means more tokens per call, multiplied across a very large agent population. Speed mattered too: the update had to reach the agent during the call, not after it.

**3. Why isn't a bigger context window automatically better?**

Answer quality tends to drop as the window fills with less relevant material, and every extra token adds cost and delay. The question isn't how much you can send, but the least the model needs to be right.

- **Open Eventz:** I send one event per call, not a batch of events. The model sees only what it needs for that decision, it can't confuse two events, and one event's text can't leak into another's answer.
- **EHS/NBA:** We re-decided about 130 seconds into the call. Earlier, the model hadn't heard enough to understand the real issue. Later, it would have understood more, but the agent would have had less of the call left to act on. That was a timing vs. accuracy trade-off, settled with data.

**4. Name three ways your products reduce hallucination.**

Constrain what the model can say, let a governed system make the decision, and keep a human on the calls that matter. Underneath all three, measure it.

- **Open Eventz:** The model can only answer free, paid or unknown, never a dollar amount. A listed price on the page overrides it, and an explicit "21+" always hides an event. Drop-off safety comes from the venue's official policy, never the model. A 64-event eval showed where it was over-confident.
- **EHS/NBA:** The model returned intent plus sentiment and churn-risk scores on an agreed scale. Pega's eligibility rules and scoring chose the recommendations, so the model could inform an offer but never create one. The agent decided whether to act.

**5. Why doesn't Open Eventz use RAG? When would it need it?**

RAG fetches relevant knowledge the model can't otherwise see and adds it to the request. You need it when the model must answer open questions over information that's private, changing or too large for the prompt. You don't need it when the input already contains the evidence.

- **Open Eventz:** Each decision is about one event, and its text is the full evidence; the scraper already did the retrieval. It would need retrieval once a parent can ask an open question like "something free for a four-year-old Saturday morning." Even then I'd have the model turn the question into filters and let the database return real events.
- **EHS/NBA:** The transcript was supplied at run time, which is grounding: the same principle as RAG, without a knowledge base. RAG would come in if agents could ask the assistant a question, such as a roaming-credit policy, which would mean retrieving the right policy text first. (Hypothetical, not something we built.)

**6. What's an embedding, and where could Open Eventz use one?**

An embedding turns text into numbers that represent its meaning, so similar ideas sit close together even with different words. A vector database stores them and finds the nearest matches.

- **Open Eventz:** Not used today, because filters are structured. Two possible uses: spotting the same event listed by two sources, and matching a parent's search ("messy play for toddlers") to events described differently ("sensory activities for under-fives").
- **EHS/NBA:** A natural fit would be finding past calls similar to this one, to see which resolutions worked, or matching a customer's own words to the closest intent category. (Hypothetical. Don't claim we used embeddings unless you confirm it.)

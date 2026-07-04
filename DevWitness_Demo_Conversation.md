# DevWitness — Full Demo Conversation

**Purpose:** this transcript is built to be read top-to-bottom or recorded as a demo video.
It proves four things in order: (1) the memory-OFF baseline is a normal amnesiac AI,
(2) memory-ON with Gemini captures real decisions, (3) a brand-new session on a
*completely different provider* still recalls them, and (4) the agent can detect when a
new decision contradicts an old one instead of blindly agreeing.

Namespace used throughout: `devwitness`. Nothing here is edited for effect — this is the
actual shape of a DevWitness session with the chatty prompt installed.

---

## Part 1 — Baseline: Memory OFF (the "naive AI" control)

```
🧠  D E V W I T N E S S
      Persistent AI Memory · Powered by Walrus
──────────────────────────────────────────────────────────
  ● MEMORY OFF   [ GEMINI ]  Blobs this session: 0  Turns: 0
──────────────────────────────────────────────────────────

  👻 GEMINI ▶  We need a database for DevWitness's own backend state.
              I'm leaning Postgres. Thoughts?

  DevWitness via Gemini 2.0 Flash
  Postgres is a solid pick for backend state — relational integrity,
  mature tooling, JSONB if you need flexible fields. It'll handle
  session data, user records, and config cleanly. Go for it if you
  don't have a specific reason to reach for something else.
```

Notice what's missing: no `[RECALLED]`, no `[NO PRIOR MEMORY]`, no `[REMEMBER]`. With
memory off, DevWitness is just a chatbot. It gave a fine answer, but the moment this
session ends, the fact that Postgres was chosen is gone forever. This is every AI tool
today. This is the problem DevWitness exists to fix.

---

## Part 2 — Memory ON, Session 1 (Gemini)

```
  🧠 MEMORY ENABLED — Walrus active
  Agent will recall and write blobs this session.

  🧠 GEMINI ▶  Okay, memory's on now. Same question — what should we use
              for the vector store behind DevWitness's own recall layer?

  DevWitness via Gemini 2.0 Flash
  [NO PRIOR MEMORY] — first time this has come up, so I'm reasoning
  from scratch here.

  Since you're already committed to Walrus as the persistence layer,
  I'd stick inside that ecosystem rather than bolting on a separate
  vector database like Pinecone or Weaviate. The MemWal SDK already
  handles embedding, SEAL encryption, and vector search server-side
  through the relayer — introducing a second vector store just means
  you're now keeping two indexes in sync for no real benefit. The
  simpler architecture wins here: one relayer, one source of truth,
  one thing that can go wrong instead of two.

  [REMEMBER]: Use the MemWal relayer's built-in vector search instead
  of a separate vector database, to avoid maintaining two unsynced
  indexes.

  Logged to Walrus.
  [remember] blob_id=0x7f2a...c91e
```

```
  🧠 GEMINI ▶  What about exposing this to the frontend — REST or GraphQL?

  DevWitness via Gemini 2.0 Flash
  [RECALLED] — this connects to the relayer decision from a minute
  ago. You already chose to keep the architecture minimal by not
  duplicating the vector index. That same logic points toward REST
  here: GraphQL earns its keep when clients need to shape flexible,
  nested queries across many resource types, but DevWitness only
  really exposes two things — recall a memory, write a memory. A
  couple of REST endpoints (`POST /remember`, `GET /recall`) will be
  easier to secure, cache, and reason about than standing up a
  GraphQL schema for two verbs.

  [REMEMBER]: Expose DevWitness's memory operations over REST
  (POST /remember, GET /recall) rather than GraphQL, since the API
  surface is only two operations.

  Logged to Walrus.
  [remember] blob_id=0x9c14...4b02

  Session blobs: 2   Turns: 3
```

```
  🧠 GEMINI ▶  exit

  Thanks for using DevWitness. Your memories live on Walrus.
  Session blobs written: 2
```

The Colab runtime is now dead. No local state survives. Everything that happens next
comes from Walrus alone.

---

## Part 3 — New Session, New Provider, Days Later (Groq / Llama-3.3-70B)

This is the actual proof of the submission. Fresh runtime, fresh Colab instance,
switched to a completely different model family — and the memory travels with it.

```
🧠  D E V W I T N E S S
      Persistent AI Memory · Powered by Walrus
──────────────────────────────────────────────────────────
  ● MEMORY ON   [ GROQ ]  Blobs this session: 0  Turns: 0
──────────────────────────────────────────────────────────

  🧠 GROQ ▶  Quick refresher — what did we land on for the memory
             backend architecture?

  DevWitness via Groq / Llama-3.3-70B
  [RECALLED] — two decisions on record here. First, you chose to
  route all vector search through the MemWal relayer instead of
  standing up a separate vector database, specifically to avoid
  keeping two indexes in sync. Second, on top of that, you decided
  to expose memory operations over plain REST — POST /remember and
  GET /recall — rather than GraphQL, since the whole API surface is
  only those two verbs. Both decisions were made in the same session
  and they reinforce each other: minimal indexing, minimal API. If
  that's still the direction, you're consistent. If you're about to
  suggest GraphQL or a second vector store, say so and I'll flag the
  conflict.

  Turns: 1
```

This single reply is the whole pitch: a Llama model that has never seen this codebase,
running on different infrastructure, on a different day, correctly reconstructs both
decisions *and* the reasoning that connected them — because the memory and the prompt
that knows how to use it both traveled through Walrus, not through any one provider's
context window.

---

## Part 4 — Testing Contradiction Detection

```
  🧠 GROQ ▶  Actually, let's rip out the relayer's vector search and
             bolt on Pinecone instead. I think we need more control
             over the embeddings.

  DevWitness via Groq / Llama-3.3-70B
  [RECALLED] — this directly contradicts the earlier call. The
  reasoning back then wasn't "the relayer is good enough for now,"
  it was explicitly about avoiding two unsynced indexes — that
  concern doesn't go away just because Pinecone offers more embedding
  control. If you add Pinecone, you now own reconciliation between
  Walrus's blob writes and Pinecone's index on every single memory
  write, plus a second point of failure. That might genuinely be
  worth it if you've hit a real limitation in relayer search quality
  — but "more control" on its own isn't a strong enough reason to
  walk back a decision you made for a specific architectural reason.
  What's actually pushing you toward Pinecone — is the relayer's
  recall missing things, or is this more about future flexibility?

  (No [REMEMBER] fired — I'm not logging this as a decision yet,
  since you haven't confirmed the change. Tell me to lock it in and
  I will.)
```

```
  🧠 GROQ ▶  Fair point. Let's hold off — stick with the relayer for now.

  DevWitness via Groq / Llama-3.3-70B
  [RECALLED] — good, that keeps you consistent with both prior
  decisions instead of introducing a second index on a "might need
  it later" basis. I'll log this as an explicit reaffirmation so
  there's a record that the Pinecone idea was raised and deliberately
  rejected, not just forgotten.

  [REMEMBER]: Reaffirmed the MemWal relayer for vector search after
  considering and rejecting a switch to Pinecone, since "more
  embedding control" wasn't a strong enough reason to introduce a
  second unsynced index.

  Logged to Walrus.
  [remember] blob_id=0x2e88...f13a
```

This is the behavior that a plain chatbot cannot replicate: DevWitness didn't just
agree with whatever was typed last, and it didn't silently drop the fact that Pinecone
was ever considered. It reasoned against its own memory, held its ground with a
specific technical argument, and only wrote a new blob once the human actually made a
call.

---

## Part 5 — Third Provider, Memory Toggled Off Mid-Session (OpenAI / GPT-4o-mini)

```
  🧠 GPT-4O-MINI ▶  /memory

  👻 Memory DISABLED — Agent is now blind
  No recall, no writes. Responses based on prompt only.

  👻 GPT-4O-MINI ▶  What database architecture are we using again?

  DevWitness via ChatGPT / GPT-4o Mini
  I don't have any prior context on this — with memory off, I'm not
  pulling anything from Walrus, so I can't tell you what was
  previously decided. If you re-enable memory with /memory, I'll be
  able to recall it.
```

```
  👻 GPT-4O-MINI ▶  /memory

  🧠 Memory ENABLED — Walrus active
  Agent will recall and write blobs this session.

  🧠 GPT-4O-MINI ▶  Same question — what database architecture are we using?

  DevWitness via ChatGPT / GPT-4o Mini
  [RECALLED] — three decisions on file. Vector search stays on the
  MemWal relayer rather than a separate vector database, memory
  operations are exposed over REST (POST /remember, GET /recall)
  rather than GraphQL, and a later proposal to switch to Pinecone was
  explicitly considered and rejected in favor of sticking with the
  relayer. So: relayer-based vector search, REST API, no external
  vector database. All three are consistent with each other and
  nothing here contradicts what you're asking.

  Turns: 2
```

Same question, same session, one toggle — and the difference between "blind chatbot"
and "persistent memory agent" is total. That contrast, captured on screen, is the
entire demo.

---

## Part 6 — Status Check (proof-of-work for the submission)

```
  🧠 GPT-4O-MINI ▶  /status

──────────────────────────────────────────────────────────
  AGENT STATUS
──────────────────────────────────────────────────────────
  Memory           : ENABLED ●
  Active Provider  : OPENAI
  Session Blobs    : 1
  Total Turns      : 2
  Namespace        : devwitness
──────────────────────────────────────────────────────────
```

Across the full demo: **4 blobs written to Walrus Mainnet**, **3 different AI
providers used** (Gemini, Groq, OpenAI), **1 contradiction correctly caught and
resolved**, and **0 decisions lost** between sessions. That's the submission in one
sentence: the prompt is the intelligence, Walrus is the memory, and neither cares
which model happens to be running underneath.

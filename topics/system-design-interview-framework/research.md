# The system-design interview delivery framework

> A system-design interview hands you a vague prompt ("design Twitter") and about 45 minutes. The delivery framework is a fixed track to run on: requirements, core entities, API, high-level design, then deep dives, each with a time box. It matters because the most common reason mid-level candidates fail is not missing knowledge but failing to deliver a working system in time [S0004]. A track also gives a nervous candidate a clear place to fall back to when they feel lost [S0004].

## ★ Your questions, answered

1. **How do I turn a vague prompt into clear functional and non-functional requirements?** Ask questions the way you would ask a product manager, then cut the list down to the top 3 features and the top 3 quantified system qualities. Spend about 5 minutes on it [S0004] [S0017].

```lk-steps
{
  "title": "The first 5 minutes: from vague prompt to requirements",
  "steps": [
    {"title": "Restate the prompt and ask who uses it", "phase": "req", "cost": "~1 min",
     "body": "Clarify users, platforms and the most important features before drawing anything.",
     "example": "\"Is this web, mobile, or both?\"\n\"What are the most important features?\"", "cite": ["S0014", "S0012"]},
    {"title": "List and prioritize functional requirements", "phase": "req", "cost": "~2 min",
     "body": "Write \"Users should be able to...\" statements and keep only the **top 3**.",
     "example": "1. Users can post tweets\n2. Users can follow users\n3. Users can see a feed", "cite": ["S0004"]},
    {"title": "Pick and quantify non-functional requirements", "phase": "req", "cost": "~2 min",
     "body": "Walk a short checklist (consistency vs availability, scale, latency, durability) and pick the few that matter, with numbers.",
     "example": "100M+ DAU\nfeed renders < 200 ms\navailability over consistency", "cite": ["S0004"]},
    {"title": "Decide whether to estimate", "phase": "req", "cost": "0-2 min",
     "body": "Say you will skip upfront math and do it later only if a number would change the design.",
     "example": "\"I'll do the math when we size the hard part.\"", "cite": ["S0004", "S0015"]}
  ]
}
```

2. **What are the steps of a 45-minute answer, and how long should each take?** Requirements (~5 min), core entities (~2 min), API (~5 min), an optional data flow (~5 min), high-level design (~10-15 min), and deep dives (~10 min) [S0004]. The full timeline is in [The delivery framework](#delivery-framework-steps) below, with Alex Xu's 4-step version next to it.

3. **What do interviewers expect from a junior/mid candidate compared with a senior one?** Everyone must deliver a complete design that meets the requirements [S0003]. A mid-level candidate covers the basics well but not in great depth, drives the early steps, and can expect the interviewer to lead the deep dives [S0003] [S0016]. A senior candidate moves through the basics fast, finds the hard problems themselves, and goes deep in about 2 places [S0003] [S0016]. See [What changes by level](#level-expectations).

## ★ Your prediction, checked

No prediction was given. A typical beginner guesses that the interview is about reaching the "right" architecture with the right technologies, such as dropping in Redis or Kafka [S0017]. The sources disagree with that: there are many right answers, and interviewers grade how you navigate the problem, reason about trade-offs and communicate [S0003] [S0017].

## Core ideas

### Functional requirements  {#functional-requirements}

Functional requirements are the core features, phrased as "Users/Clients should be able to..." [S0004]. You get to them through back-and-forth with the interviewer, asking targeted questions like "does the system need to do X?" or "what would happen if Y?" [S0004].

The skill being tested is cutting, not listing. Real products have hundreds of features; your job is to pick the top 3, and a long list hurts more than it helps [S0004]. Many interviewers directly evaluate your ability to focus on what matters [S0004].

Don't fill gaps with your own assumptions. A "photo sharing service" could mean Instagram, Imgur or a high-resolution portfolio site like Flickr, and each leads to a different design [S0012]. If the interviewer tells you to assume, write the assumption down, because you may need it later [S0014].

> [!example] Clarifying a news feed prompt
> Candidate: "Web, mobile, or both?" Interviewer: "Both." Candidate: "Most important features?" Interviewer: "Post, and see friends' feed." Candidate: "Reverse-chronological, or ranked?" Interviewer: "Reverse-chronological." Candidate: "How many friends per user?" Interviewer: "5000." [S0014]
> Three questions turned "design a news feed" into two prioritized features and one simplifying rule.

### Non-functional requirements  {#non-functional-requirements}

Non-functional requirements describe system qualities: "The system should be..." [S0004]. They only help when they are tied to a part of the system and given a number. "Low latency" says nothing, because almost every system wants it; "search returns in < 500 ms" names the part that must be fast and sets a target [S0004].

A checklist helps when you don't know the domain: consistency vs availability (CAP), environment limits (mobile, low bandwidth), scale and read/write ratio, latency, durability, security, fault tolerance and compliance [S0004]. Pick the few that matter most; Hello Interview says the top 3-5 [S0004].

Strong candidates think one step ahead here: "I'll need storage, so what are the read/write patterns, consistency needs and expected scale?" [S0017]. That shows that requirements drive the solution, not the other way round [S0017].

> [!example] Quantified NFRs for a Twitter-like prompt
> - Highly available, prioritizing availability over consistency
> - Scales to 100M+ daily active users
> - Feed renders in under 200 ms
>
> Each one names a quality, a part of the system, and a number [S0004].

### Estimation only when it changes a decision  {#estimation-when-needed}

Many guides tell you to compute storage, DAU and QPS up front. Hello Interview's interviewers call this mostly wasted time: candidates do the math only to conclude "ok, it's a lot" [S0004]. Their advice is to tell the interviewer you'll skip upfront estimation and do math during the design when it matters [S0004] [S0015].

Estimation is worth it when the answer picks between two designs. For a trending-topics (top-K) system, the number of topics decides whether one in-memory heap is enough or whether you must shard it [S0004]. "50K requests/s at ~5K per server is about 10 servers plus headroom" is the kind of in-context math interviewers want [S0015].

> [!tip] Rule of thumb
> If you can't name the design decision a number will change, skip the number. If you can, do the math out loud, right at that decision [S0004] [S0015].

Other sources still place estimation earlier; see "Where sources disagree" below.

### The delivery framework and its time boxes  {#delivery-framework-steps}

Hello Interview's framework is a sequence of steps with timings, built so you stay on what the interviewer cares about [S0004]. Its key rule is the order: get a simple, complete design on the board first, then harden it [S0004].

```lk-steps
{
  "title": "A 45-minute interview, step by step (Hello Interview)",
  "steps": [
    {"title": "Requirements", "phase": "req", "cost": "~5 min",
     "body": "Top 3 functional, a few quantified non-functional, estimation only if it matters.", "cite": ["S0004"]},
    {"title": "Core entities", "phase": "entities", "cost": "~2 min",
     "body": "A bulleted list of the nouns the API exchanges and the system stores. A first draft, not a schema.",
     "example": "User, Tweet, Follow", "cite": ["S0004"]},
    {"title": "API / system interface", "phase": "api", "cost": "~5 min",
     "body": "Usually REST, roughly one endpoint per functional requirement.",
     "example": "POST /v1/tweets\nGET /v1/feed -> Tweet[]", "cite": ["S0004", "S0015"]},
    {"title": "Data flow (optional)", "phase": "api", "cost": "~5 min",
     "body": "Only for pipeline-style systems such as a web crawler. Skip it otherwise.",
     "example": "fetch seeds -> parse HTML -> extract URLs -> store -> repeat", "cite": ["S0004"]},
    {"title": "High-level design", "phase": "hld", "cost": "~10-15 min",
     "body": "Boxes and arrows that satisfy each endpoint in turn. Note caches or queues verbally and move on.", "cite": ["S0004"]},
    {"title": "Deep dives", "phase": "deep", "cost": "~10 min",
     "body": "Meet the non-functional requirements, handle edge cases and bottlenecks, answer probes.", "cite": ["S0004"]}
  ]
}
```

Alex Xu's book (ByteByteGo) uses 4 broader steps for the same 45 minutes [S0014]. The phases line up, but the boxes are drawn differently:

| Hello Interview | Time | Alex Xu (ByteByteGo) | Time |
|---|---|---|---|
| Requirements | ~5 min | 1. Understand the problem, establish design scope | 3-10 min |
| Core entities, API, (data flow) | ~12 min | inside step 2, "depends on the problem" | — |
| High-level design | ~10-15 min | 2. Propose high-level design, get buy-in | 10-15 min |
| Deep dives | ~10 min | 3. Design deep dive | 10-25 min |
| (none) | — | 4. Wrap up: bottlenecks, recap, next scale | 3-5 min |

Sources: [S0004] [S0014].

Two older frameworks follow the same arc. The System Design Primer uses use cases and constraints, high-level design, core components, then scaling [S0001]. interviewing.io uses 3 steps: requirements; data types, API and scale; then design [S0012].

### Core entities and the API sketch  {#core-entities-and-api}

Core entities are the nouns your API exchanges and your system persists [S0004]. Two questions find them: who are the actors, and which resources do the functional requirements need? [S0004] You list only names now and add fields later, at the database box in the high-level design, because you'll discover entities as you go [S0004].

The API is the contract between the system and its users, and it usually maps straight onto the functional requirements [S0004]. Default to REST with plural resource names; pick GraphQL or RPC only for a specific reason [S0004]. Derive the current user from the auth token, never from a `userId` in the request body [S0004].

> [!warning] Don't over-polish the API
> You should sketch 4-5 key endpoints in a couple of minutes. If you are still on API details 10 minutes in, you're going too deep [S0015].

As a NestJS developer you already write these contracts daily: a controller with `@Post('tweets')` and `@Get('feed')` is exactly the level of detail wanted.

### High-level design before deep dives  {#high-level-design-first}

The high-level design is boxes and arrows (clients, servers, databases) that satisfy the API you just wrote [S0004]. A reliable method is to go endpoint by endpoint and add only what each one needs [S0004]. Talk while you draw: say how data flows and which state changes on each request [S0004].

The classic failure is layering on complexity too early and never reaching a complete solution [S0004]. When you notice a spot for a cache or a queue, say so, write a note, and move on [S0004]. Xu gives the same order as a rule: give the high-level design first, then drill down [S0014]. AlgoMaster suggests mid-level candidates practise reaching a clear high-level design in about 20 minutes [S0018].

Deep dives then harden the design: meet each non-functional requirement, cover edge cases and bottlenecks, and respond to the interviewer's probes [S0004]. Xu suggests choosing what to dig into together with the interviewer, critical components first [S0014].

> [!example] Endpoint by endpoint
> `POST /v1/tweets` → add an API server and a Tweet table. `POST /v1/follows` → add a Follow table. `GET /v1/feed` → read Follow, then Tweet. The design works but feed reads will be slow at scale; that is a deep-dive topic, not a reason to stop now [S0004].

### Driving the conversation and communicating trade-offs  {#tradeoff-communication}

The interview simulates two co-workers solving an ambiguous problem; the process matters more than the final drawing [S0014]. So think out loud, treat the interviewer as a teammate, and ask for feedback early [S0014].

Every decision has a cost, and interviewers listen for whether you know it. Follow each choice with "the trade-off here is...", say what you optimize and what you give up, and tie it to a requirement [S0017]. Then actually decide: listing three databases with pros and cons and moving on is a common failure [S0012].

Name generic components unless you know the product well. "A queue, because..." is safer than "Kafka", which invites "why Kafka and not X?" [S0012]. When the interviewer suggests an alternative, explore it with curiosity rather than defending your first idea [S0017] [S0012].

> [!interview] What a strong trade-off sentence sounds like
> "A cache here improves read latency a lot, but adds eventual consistency. For this feature that's fine, because users value speed over perfectly fresh data." [S0017]

### Common failure modes  {#interview-failure-modes}

The same few mistakes show up across sources:

| Failure | What it looks like | Fix |
|---|---|---|
| Running out of time | No working system at minute 45 | Follow the time boxes; focus beats speed [S0004] |
| Diving deep too early | Sharding talk before any boxes exist | High-level design first, note complexity and move on [S0004] [S0014] |
| Jumping to a solution | Drawing schemas in minute 1 | Spend the first ~5 min on requirements [S0014] [S0017] |
| Long silences | Several minutes of silence, several times | Pause briefly to think, then ask for a hint if stuck [S0012] [S0014] |
| Over-engineering | Microservices and queues for 1,000 users | Design for the stated requirements; it's a red flag [S0014] |
| Fighting feedback | Defending the first idea | Treat probes as hints the interviewer has seen work [S0012] [S0017] |
| Talking over the interviewer | Monologue in deep dives | Leave room for probes; they carry the signals [S0004] |

A short pause to collect your thoughts is a green flag, not a red one [S0012]. The trap is silence that hides being stuck, because the interviewer can't help until you say so [S0012].

### The four rubric areas  {#rubric-areas}

Rubrics differ by company, but Hello Interview finds four themes everywhere: Problem Navigation, Solution Design, Technical Excellence, and Communication and Collaboration [S0003].

| Area | What it checks | Typical failure |
|---|---|---|
| Problem Navigation | Break down a vague problem, prioritize, reach a working system | Thin requirements, trivial focus, getting stuck [S0003] |
| Solution Design | Solve each piece and fit them together | Weak core concepts, ignoring scale, "spaghetti design" [S0003] |
| Technical Excellence | Apply current technologies and patterns | Not knowing the tools, outdated approaches [S0003] |
| Communication and Collaboration | Explain clearly, take feedback, work together | Unclear explanations, defensiveness [S0003] |

Problem Navigation is often the most important area and the one newcomers struggle with most; Hello Interview's cure for it is the delivery framework [S0003]. The framework's steps map onto the rubric: requirements and time boxes show navigation, high-level design shows solution design, deep dives show technical excellence, and thinking out loud runs through all four.

### What changes by level  {#level-expectations}

The prompt can be identical across levels; the bar moves [S0018]. Hello Interview describes the shift along breadth, depth and proactiveness [S0016].

| Dimension | Junior / mid-level | Senior |
|---|---|---|
| Breadth | Broad knowledge is vital; expect "what does that API gateway do?" [S0016] | Fundamentals are assumed, not probed [S0016] |
| Depth | Limited; solid core concepts, academic-level reasoning is fine [S0016] | Experience-based detail in about 2 areas [S0016] |
| Proactiveness | Drive requirements, API, schema and high-level design; interviewer leads later [S0016] | Spot the hard parts and own limitations early [S0016] |
| Deep dives | Interviewer often points out what to improve [S0004] | Candidate finds and leads them [S0004] |
| Who drives | Interviewer sets direction and pace [S0012] | Candidate directs more of the interview [S0012] |

For a mid-level candidate the core bar is a complete, working design with some guidance, where you can explain every component you add [S0018]. If you add a cache, be ready to say what it stores and when entries expire [S0018]. Most entry-level roles have no system-design round at all; the round becomes common from mid-level [S0003].

> [!tip] For an anxious mid-level candidate
> You are not graded on discovering every bottleneck yourself. Deliver a clean, complete design; when the interviewer probes, reason through it out loud [S0016] [S0018].

## How it works: one prompt through the framework

Here is a URL-shortener-sized prompt at skeleton level only. The goal is the shape of the 45 minutes, not the design itself; a later path node covers the URL shortener in depth.

```lk-sequence
{
  "title": "\"Design a URL shortener\" in 45 minutes (skeleton)",
  "actors": [{"id": "c", "label": "Candidate"}, {"id": "i", "label": "Interviewer"}, {"id": "w", "label": "Whiteboard"}],
  "phases": {"req": "Requirements ~5 min", "entities": "Entities ~2 min", "api": "API ~5 min", "hld": "High-level design ~10-15 min", "deep": "Deep dives ~10 min"},
  "messages": [
    {"from": "c", "to": "i", "label": "Custom aliases? Expiry? Analytics?", "phase": "req"},
    {"from": "i", "to": "c", "label": "Aliases yes, expiry optional, no analytics", "phase": "req", "reply": true},
    {"from": "c", "to": "w", "label": "FR: shorten, redirect, custom alias", "phase": "req"},
    {"from": "c", "to": "w", "label": "NFR: redirect < 100 ms, codes unique, read-heavy", "phase": "req"},
    {"from": "c", "to": "w", "label": "User, ShortUrl", "phase": "entities"},
    {"from": "c", "to": "w", "label": "POST /v1/urls, GET /{code} -> 302", "phase": "api"},
    {"from": "c", "to": "w", "label": "Client -> API server -> database", "phase": "hld"},
    {"over": "c", "label": "note: cache for redirects, later", "phase": "hld"},
    {"from": "i", "to": "c", "label": "How do you make codes unique?", "phase": "deep"},
    {"from": "c", "to": "i", "label": "Hash vs counter + Base62, trade-offs", "phase": "deep", "reply": true}
  ]
}
```

> [!example] The skeleton, step by step
> - **Requirements.** FR: (1) create a short URL from a long one, (2) visiting the short URL redirects to the long one, (3) optional custom alias. NFR: redirects in under 100 ms, every code unique, far more reads than writes.
> - **Estimation?** Worth doing for one decision only: code length. Six Base62 characters give 62^6 ≈ 56.8 billion codes, so 6-7 characters is plenty.
> - **Entities.** `User`, `ShortUrl {code, longUrl, createdAt, expiresAt?}`.
> - **API.** `POST /v1/urls {longUrl, alias?} -> {code}`; `GET /{code} -> 302 Location: longUrl`.
> - **High-level design.** Client → API server → database, walked endpoint by endpoint.
> - **Deep dives.** How codes are generated (hashing such as MD5 then Base62 vs a counter, and collisions) and keeping redirects fast at read-heavy scale [S0014] [S0001].

The framework only supplies the order and time boxes [S0004]. The specific requirement values above are illustrative choices you would confirm with the interviewer, not facts from the sources. The deep-dive topics, hash design and Base62 encoding, are the ones both Xu and the Primer name for this prompt [S0014] [S0001].

```lk-arch
{
  "title": "URL shortener: high-level design at the end of minute ~25",
  "panels": [
    {"title": "Skeleton design", "flow": [
      {"box": "Browser / API client", "sub": "React app, curl", "kind": "client"},
      {"arrow": "POST /v1/urls, GET /{code}"},
      {"box": "API server", "sub": "e.g. a NestJS service", "kind": "app", "items": ["create code", "look up code, return 302"]},
      {"arrow": "read / write"},
      {"box": "Database", "kind": "data", "items": ["ShortUrl: code, longUrl, expiresAt"]}],
     "pros": ["meets all 3 functional requirements", "easy to explain end to end"],
     "cons": ["redirect latency at scale is still open (deep dive)"]}
  ]
}
```

## Trade-offs: when to use the framework, when not to

The framework fits the common product and infrastructure prompts, like "design Uber" or "design a rate limiter" [S0003]. It is not the right tool for low-level (class design) or ML design interviews, which need different guides [S0003].

Treat the time boxes as a rough guide. Xu says the split depends on the problem's scope and what the interviewer asks for [S0014]. Skip the data-flow step for systems that aren't pipelines [S0004]. If the interviewer steers you, follow them, even if it breaks the plan; at mid-level they set the direction [S0012] [S0017].

The cost of rigidity is real too. Interviewers are trained to throw candidates off a line when they seem to be reciting [S0017]. Use the framework as a track, not a script.

## Common misconceptions

> [!misconception] "There is one right answer, and I fail if I miss a detail like Redis."
> There are many right answers. Interviewers weigh how you navigate the problem, reason about trade-offs and communicate [S0003] [S0017].

> [!misconception] "I should always do back-of-the-envelope math at the start."
> Hello Interview advises doing math only where it changes a decision [S0004] [S0015]. Other sources still list estimation early, so ask the interviewer if they want it [S0014].

> [!misconception] "More requirements show more thoroughness."
> A long list hurts. Prioritize the top 3 core features and design for those [S0004].

> [!misconception] "Mentioning advanced tech such as sharding or Kafka early impresses the interviewer."
> Complexity before a working design is the most common way to never finish [S0004]. Brand names without depth invite questions you can't answer [S0012], and over-engineering is a named red flag [S0014].

> [!misconception] "A mid-level candidate must lead the whole interview like a senior."
> Mid-level candidates drive the early steps; the interviewer is expected to lead much of the later probing [S0016] [S0012].

## Where sources disagree or are unclear

- **When to estimate.** Hello Interview says skip upfront estimation and calculate only when it changes the design [S0004] [S0015]. Xu puts back-of-the-envelope checks inside step 2, after asking the interviewer whether they're needed [S0014]. The Primer asks for requests per second and read/write ratio in step 1 [S0001], and interviewing.io makes request volume part of its step 2 [S0012]. Safe move: offer it, and do it when a decision depends on it.
- **How the minutes are split.** Hello Interview gives deep dives ~10 minutes and has no wrap-up [S0004]. Xu allows 10-25 minutes for the deep dive plus a 3-5 minute wrap-up [S0014]. Xu also leaves API and schema to the problem, while Hello Interview always writes the API before the high-level design [S0014] [S0004].
- **Who leads.** The Primer says "you are expected to lead" the conversation [S0001]. interviewing.io says at mid-level and below the interviewer should set direction and pace [S0012], and Hello Interview expects mid-level candidates to drive only the early steps [S0016].
- **Silence.** Xu says "don't think in silence" [S0014]. interviewing.io agrees that long, repeated silences are a red flag, but quotes an interviewer who says that at mid-level, filling every awkward pause can hurt more than a short silence [S0012].
- **The rubric's shape.** Hello Interview names four rubric areas [S0003], its blog lists six "hidden" dimensions [S0017], and its level guide uses breadth, depth and proactiveness [S0016]. These describe the same ground from different angles rather than contradict each other.
- **Number of non-functional requirements.** Hello Interview suggests the top 3-5 [S0004]; this path's outcome asks for 3, which sits at the low end of that range.

## Further reading

- [S0004] Hello Interview, Delivery framework: the steps and time boxes this doc follows.
- [S0014] Alex Xu (ByteByteGo), A framework for system design interviews: the 4-step version, sample clarifying dialogue, dos and don'ts.
- [S0003] Hello Interview, Introduction: the four rubric areas and their failure modes.
- [S0016] Hello Interview, Expectations at each level: breadth, depth and proactiveness from mid-level to staff.
- [S0017] Hello Interview, What your interviewer is really judging: trade-off phrasing and handling feedback.
- [S0012] interviewing.io, System design interview guide: red and green flags, who drives, generic vs brand names.
- [S0015] Hello Interview, Core concepts: API design depth and in-context estimation numbers.
- [S0001] System Design Primer: the classic 4-step approach and the URL shortener deep-dive list.
- [S0018] AlgoMaster, Expectations by level: a cross-check with a chat-app example per level (lower quality).
- [S0007] ByteByteGo, Scale from zero to millions: background for the components you'll draw in later nodes.
- [S0008] ByteByteGo course index: the list of practice prompts (URL shortener, news feed, chat).

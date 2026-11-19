# System design for interviews — plan rationale

## Goal as I understood it
Pass junior-to-mid system-design interviews within 1–3 months as a full-stack engineer (React, Next.js, Node.js,
NestJS), at 10+ h/week. Self-rated beginner, but already knows the vocabulary of indexing, caching, JWT, sharding,
HTTP flow, WebSocket, REST, ACID and MongoDB. The real blocker is confidence: they want a structure to fall back on.

## What's in, what's out
In: the interview delivery framework, estimation, the web tier (DNS/LB/CDN), caching, databases (SQL vs NoSQL,
replication, sharding), consistency/CAP/availability, queues and async, API and real-time communication, and four
classic prompts (URL shortener, chat, notifications, news feed). This is the breadth-first core that interview
guides put first for short timelines [S0001][S0003], and matches the early chapters of Alex Xu Vol. 1 [S0007][S0008].
Mid-level candidates are expected to cover the basics well, not go deep, so most nodes are `use` [S0003].

Out, on purpose:
- Consensus internals (Paxos, Raft), logical/vector clocks, MapReduce/GFS papers: university distributed-systems
  courses [S0009][S0010][S0011] go there, but it is beyond junior/mid interviews and Hello Interview warns that too
  much depth can be a yellow flag [S0003].
- DDIA-level storage engine internals (LSM vs B-tree, replication protocols in detail) [S0005].
- Senior/staff topics (multi-region active-active, exactly-once stream processing, cost modeling).
- Security beyond what the learner already knows (JWT); OOP/low-level design interviews.

## Order
n01 (delivery framework) comes first: it is a quick, real win that directly addresses the learner's interview
anxiety, and it is the track every later capstone runs on ("failing to deliver a working system" is the most common
mid-level failure [S0004]). Then estimation and the web tier, mirroring "scale from zero to millions" [S0007], then
caching to close the read path. Milestone 2 builds the data layer bottom-up (model -> replicate/shard ->
consistency) and ends with the URL shortener. Milestone 3 adds async and real-time and ends with chat, which uses
the learner's WebSocket background. Milestone 4 applies everything to two fan-out prompts. Total ≈ 82 h, i.e. about
8 weeks at 10 h/week, leaving slack for reteach rounds inside the 3-month window. If the interview lands at the
1-month end of the range (~43 h), the plan does not fit: skip the elective n12 (core = 76 h) and agree with the
learner which `use` nodes to fast-track after placement. Three capstones are `deep` (n08, n11, n13), one lab per
milestone; n12 (notifications) is an `elective` `use` node rather than a fourth lab, since nothing depends on it.
Alex Xu orders the prompts notification -> news feed -> chat [S0008]; chat moves earlier here because it reuses the
learner's WebSocket background. Hello Interview advises estimating only when the numbers change a decision [S0004],
so n02 teaches that judgement, not just arithmetic.

## Edges
- n05 -> n06: choosing a shard key and read-replica routing assumes knowing the data model and access patterns.
- n06 -> n07: CAP and eventual consistency are about replicas disagreeing, which needs replication lag first.
- n01 -> n08: the first capstone's lab is a full design run on the delivery framework.
- n02 -> n08: key-space length and storage sizing for short codes come from the estimation step.
- n04 -> n08: redirects are read-heavy; the design hinges on cache-aside and TTLs.
- n05 -> n08: the store choice (key-value vs relational) for the code -> URL mapping is a core decision.
- n03 -> n10: scaling WebSocket/SSE connections needs load balancers, stateless servers and sticky sessions.
- n06 -> n11: message storage is partitioned by conversation id; needs sharding.
- n09 -> n11: delivering a message to a user on another connection server uses pub/sub.
- n10 -> n11: the client protocol choice (WebSocket vs long-poll) is the heart of the design.
- n09 -> n12: notifications are queue-driven with retries and dead-letter handling.
- n10 -> n12: idempotency keys and rate limiting on the send API prevent duplicate or spammy notifications.
- n04 -> n13: the feed is served from a per-user cache.
- n06 -> n13: posts and feeds are sharded; celebrity users are a hot-key problem.
- n07 -> n13: fan-out on write makes feeds eventually consistent; the learner must defend that trade-off.
- n09 -> n13: fan-out on write runs in async workers fed by a queue.

## Milestones
- m1 Interview toolkit and the read path (n01–n04): can structure a 45-minute answer, size it with numbers, and draw
  a scaled read path (DNS -> LB -> stateless servers -> cache -> DB) with a reason for each box.
- m2 The data layer (n05–n08): can pick and model a store, explain replicas vs shards and their failure modes, state
  a consistency requirement as a CAP trade-off, and defend a complete URL-shortener design.
- m3 Async and real-time (n09–n11): can decide what goes on a queue, pick a client protocol and API style, and
  design chat with cross-server delivery, ordering and offline messages.
- m4 Fan-out designs (n12–n13): can design a news-feed system end to end (and, if the elective n12 was taken, a
  notification system) and defend fan-out and consistency trade-offs under interviewer push-back.

## Later
- More practice prompts: rate limiter, unique ID generator, key-value store, web crawler, search autocomplete,
  YouTube/Google Drive (blob storage, presigned URLs, video transcoding), proximity service [S0008].
- Blob/object storage and media pipelines as its own node if the learner targets media-heavy companies.
- Microservices and service discovery, observability/monitoring [S0001].
- Deeper distributed-systems theory (consensus, clocks) if the goal moves to senior roles [S0009][S0010][S0011].
- Timed mock interviews across mixed prompts once all milestones are done.

## Sources
- [S0001] System Design Primer: topic index (performance/scalability, CAP, consistency and availability patterns,
  DNS, CDN, LB, databases, cache, async, communication), study guide by timeline, interview approach steps.
- [S0003] Hello Interview, System Design in a Hurry intro: interview rubric, mid vs senior expectations, depth warning.
- [S0004] Hello Interview delivery framework: requirements, core entities, API, high-level design, deep dives, timings.
- [S0005] DDIA overview: depth upper bound for data systems.
- [S0007] Alex Xu Vol. 1, "Scale from zero to millions of users": request flow, LB, replication, cache, CDN, stateless tier.
- [S0008] Alex Xu Vol. 1 table of contents: estimation, framework, and the canonical design prompts.
- [S0009] MIT 6.5840, [S0010] Princeton COS 418, [S0011] Cambridge Concurrent and Distributed Systems: theory-heavy
  upper bounds, used to decide what to leave out.

# Back-of-envelope estimation

> Back-of-envelope estimation is quick, rough arithmetic that turns "lots of users" into a few numbers: requests per second, gigabytes stored, megabytes per second sent. You round hard, write your assumptions down, and aim to be right within a factor of 2 to 10, not to the last digit [S0020] [S0021] [S0025]. It matters because those numbers decide the design. One server or many? Cache or no cache? Database or object storage? [S0021] [S0015]

## ★ Your questions, answered

**1. What is the "equation" for estimating a feature?** It is a short chain of multiplications and one division. Start from users, turn them into actions, then into requests per second, bytes stored and bytes sent [S0025] [S0020] [S0021]. The one trick: a day has 86,400 seconds, so call it **10^5 seconds** [S0021].

```lk-steps
{
  "title": "The estimation chain for one feature",
  "steps": [
    {"title": "Write the assumptions down", "phase": "assume",
     "body": "Daily active users (DAU), actions per user per day, read:write ratio, size of one item, how long you keep it, number of copies, peak factor.",
     "example": "DAU = 2M, 10 reads/user/day, 1 KB per item, keep 5 years, 3 copies, peak = 3x", "cite": ["S0021", "S0025"]},
    {"title": "Requests per day", "phase": "traffic",
     "body": "DAU x actions per user per day. Do it separately for reads and writes.",
     "example": "2M x 10 = 20M reads/day", "cite": ["S0025", "S0021"]},
    {"title": "Average QPS", "phase": "traffic", "cost": "÷ 10^5",
     "body": "Requests per day ÷ seconds per day. Use 10^5 instead of 86,400.",
     "example": "20M ÷ 100,000 = 200 QPS", "cite": ["S0021", "S0020"]},
    {"title": "Peak QPS", "phase": "traffic", "cost": "x 2 to x 10",
     "body": "Average x a peak factor. Design for the peak, not the average.",
     "example": "200 x 3 = 600 QPS", "cite": ["S0020", "S0021", "S0025"]},
    {"title": "Storage", "phase": "storage",
     "body": "Writes per day x item size x 365 x years x copies. No peak factor here.",
     "example": "20K x 1 KB = 20 MB/day → ~40 GB in 5 years → ~120 GB with 3 copies", "cite": ["S0021", "S0025"]},
    {"title": "Bandwidth", "phase": "bandwidth",
     "body": "QPS x payload per request, for reads and writes separately. Multiply by 8 to get bits per second.",
     "example": "600 x 20 KB = 12 MB/s ≈ 100 Mbps", "cite": ["S0021", "S0025"]},
    {"title": "Compare and decide", "phase": "decide",
     "body": "Put each number next to what one machine can do. Under the limit: keep one box. Over it: scale out, cache, or move the data.",
     "example": "600 QPS vs a DB that does 10K+ → one database is fine", "cite": ["S0015", "S0021"]}
  ]
}
```

**2. Which numbers decide the scale?** There are two kinds of numbers, and you need both.

| Kind | The numbers | Where they come from |
|---|---|---|
| **Inputs** (you assume them) | DAU, actions per user, read:write ratio, item size, retention, copies, peak factor | You state them, the interviewer can change them [S0021] [S0025] |
| **Outputs** (you compute them) | Peak QPS (reads and writes), total storage, peak bandwidth, size of the hot data | The chain above [S0020] [S0021] |
| **Yardsticks** (you memorise them) | What one app server, database or Redis can do; latency of memory vs disk vs network | Reference tables below [S0015] [S0019] [S0023] |

The decision always comes from putting an **output** next to a **yardstick** [S0015]. "600 QPS" means nothing alone. "600 QPS against a database that handles tens of thousands" means "one database is enough" [S0015].

**3. What does "back-of-envelope" mean?** Literally, a sum small enough to scribble on the back of an envelope. Jeff Dean, a Google Senior Fellow, defined it as "estimates you create using a combination of thought experiments and common performance numbers to get a good feel for which designs will meet your requirements" [S0020]. He made it popular in his talks on building large systems at Google [S0024] [S0019]. It is not precise maths. It is a quick, approximate check that a design is feasible [S0022].

## ★ Your prediction, checked

No prediction was given for this topic.

## Core ideas

### What back-of-envelope means  {#back-of-envelope-meaning}

The goal is the right **order of magnitude**: is it about 10, 1,000 or 1,000,000? Whether something is ten thousand or a billion changes the architecture. Whether it is 1.1 million or 1.3 million usually does not [S0025].

Interviewers watch the process more than the result: make assumptions, say them out loud, keep moving [S0020] [S0025].

> [!example] A one-line estimate
> "How long does it take to read 1 GB from an SSD?" SSDs read about 1 GB/s sequentially, so about **1 second**. From an old spinning disk at about 30 MB/s, it takes about **30 seconds** [S0001].

### Round, state, label  {#round-state-label}

Three habits make the arithmetic easy and safe [S0020] [S0021]:

- **Round while you calculate, not at the end.** "99,987 / 9.1" becomes "100,000 / 10" [S0020]. 1,825 TB becomes 2 PB straight away [S0021].
- **Write the assumptions down** so you and the interviewer can point at them later [S0020] [S0025].
- **Label every unit.** A bare "5" could be 5 KB or 5 MB [S0020].

Work in powers of 10. Multiply the leading digits, then add the exponents: 2 x 10^6 users x 10 actions = 2 x 10^7 actions [S0021].

| Handy round number | Value | Why it helps |
|---|---|---|
| Seconds in a day | 86,400 ≈ **10^5** | Requests/day ÷ 10^5 = QPS [S0021] [S0025] |
| Seconds in a year | ≈ 31.5 million ≈ 3 x 10^7 | Yearly totals [S0025] |
| 1 million per day | ≈ **10 per second** | 10^6 ÷ 10^5 = 10 (really ~12) [S0021] |
| 1 million x 1 KB | = 1 GB | 10^6 x 10^3 bytes = 10^9 bytes [S0020] |
| Days in 5 years | 365 x 5 ≈ 1,800 ≈ 2,000 | Retention totals [S0021] |

> [!tip] The 10^5 shortcut
> Dividing by 10^5 instead of 86,400 makes your QPS about 15% too low. That is far inside the "within 2x" target. Say "about" and move on [S0021] [S0020].

### QPS from daily active users  {#qps-from-dau}

QPS means queries (requests) per second. Average QPS = DAU x requests per user per day ÷ 10^5 [S0025] [S0020]. If you only have monthly users, first multiply by the share who come each day [S0020].

Then apply a **peak factor**. Traffic is not flat across the day, so systems must handle bursts [S0025]. Sources use 2x [S0020], 3x [S0021] or anywhere from 3x to 10x [S0025]. Pick one, say it, and move on.

Finally, **split reads from writes**. Most apps read far more than they write. A photo app with 100 reads per write ends up with 300 writes per second but 30,000 reads per second at peak [S0021]. That ratio tells you where to spend design effort: here, on the read path [S0021].

> [!example] 10M users, 20 requests each
> 10M x 20 = 200M requests/day. 200M ÷ 10^5 = 2,000 QPS (exact: ~2,300). Peak at 5x ≈ 10,000-11,500 QPS [S0025].

### Storage estimate  {#storage-estimate}

Storage = size of one item x new items per day x days kept x copies [S0021] [S0025].

Estimate the item size from its fields. A post-like record might be an 8-byte user ID, an 8-byte timestamp, 200 bytes of text and 100 bytes of metadata, so about 300 bytes [S0025]. Media is much bigger: an image is 100 KB to 1 MB, a minute of video 5-10 MB [S0025].

Two rules are easy to forget. First, multiply by the number of **copies** (replicas, backups): 1 TB of raw data can mean 3 TB or more [S0025] [S0021]. Second, **no peak factor** for storage: a day of uploads is the same size whether it arrives smoothly or in bursts [S0021].

> [!example] Photo app storage [S0021]
> 10M photos/day x 500 KB = 5 TB/day. x 365 ≈ 2 PB/year. x 5 years = 10 PB. x 3 copies = **30 PB**.

### Bandwidth estimate  {#bandwidth-estimate}

Bandwidth = QPS x bytes per request. Do it separately for incoming (writes) and outgoing (reads) data [S0021] [S0025]. Use peak QPS, because the network must cope with the peak [S0021].

Watch **bits vs bytes**. One byte is 8 bits [S0020]. Storage is quoted in bytes (MB, GB). Network links are quoted in bits (Mbps, Gbps). So 500 MB/s is about 4 Gbps [S0025]. A 1 Gbps link moves roughly 100 MB/s [S0001].

> [!example] Photo app bandwidth [S0021]
> In: 100 writes/s x 500 KB = 50 MB/s. Out: 10,000 reads/s x 500 KB = 5 GB/s. At 3x peak: 150 MB/s in, **15 GB/s out**. The 100x gap between in and out equals the read:write ratio, a good sign the sums are consistent.

### Latency orders of magnitude  {#latency-orders-of-magnitude}

You do not need every row of the famous latency table. You need the **ranking** and the rough gaps between rows [S0022] [S0020]. Each line below is roughly 10x to 100x slower than the one above it.

| Operation | Rough time | Think of it as |
|---|---|---|
| Read main memory (RAM) | 100 ns | nanoseconds [S0019] |
| Read 4 KB at random from SSD | 150 µs | ~1,000x RAM [S0019] [S0001] |
| Round trip inside one datacenter | 500 µs (sub-1 ms) | under a millisecond [S0019] [S0023] |
| Read 1 MB sequentially from SSD | 1 ms | [S0019] |
| Spinning-disk (HDD) seek | 10 ms | ~20x a datacenter round trip [S0019] |
| Packet California → Netherlands → California | 150 ms | cross-region: 50-150 ms [S0019] [S0023] |

Lessons: memory is fast, disk is slow, avoid disk seeks, and calls between regions are slow [S0020]. You get about 2,000 round trips per second inside a datacenter, but only 6-7 around the world [S0001].

> [!example] Why a cache helps
> A Redis read is a network hop plus a RAM lookup, about 1 ms [S0015]. A trip to another region is 50-150 ms [S0023]. That is 50-150x slower. This gap is why you cache near the user or put a CDN in front [S0015].

### Powers of two and data units  {#powers-of-two-units}

Every 10 powers of two is about 1,000 [S0001] [S0020]:

| Power | Approx. | Unit |
|---|---|---|
| 2^10 | 1 thousand (10^3) | 1 KB |
| 2^20 | 1 million (10^6) | 1 MB |
| 2^30 | 1 billion (10^9) | 1 GB |
| 2^40 | 1 trillion (10^12) | 1 TB |
| 2^50 | 1 quadrillion (10^15) | 1 PB |

Two extras from the same table: 2^16 = 65,536 (64 KB) and 2^32 ≈ 4.3 billion (4 GB) [S0001]. One ASCII character is one byte [S0020]. For estimates, treat 1 KB as 1,000 bytes; storage and bandwidth are usually counted in base 10 anyway [S0025] [S0022].

> [!example] A JS developer's version
> A 2^32 counter (an unsigned 32-bit ID) runs out at about 4 billion [S0001]. 300M new tweets a day [S0020] would use it up in about 14 days. That is why such systems use 64-bit IDs.

### Availability in nines  {#availability-nines}

Availability is the share of time a service is up, counted in "nines": 99.99% is "four nines" [S0001]. Big cloud providers set their SLAs at 99.9% or higher [S0020].

| Availability | Downtime per year | Downtime per day |
|---|---|---|
| 99% | 3.65 days | 14.4 min [S0020] |
| 99.9% (three nines) | ~8 h 46 min | ~1 min 26 s [S0001] |
| 99.99% (four nines) | ~52.6 min | ~8.6 s [S0001] [S0020] |
| 99.999% (five nines) | ~5.3 min | ~0.9 s [S0020] |

Shortcut: every extra nine divides the downtime by 10 [S0020]. Parts that run **in sequence** multiply: two 99.9% parts in a chain give about 99.8% [S0001]. Parts **in parallel** (either one can serve) give 1 - (0.001 x 0.001) = 99.9999% [S0001].

> [!example] Your Next.js app
> Browser → NestJS API (99.9%) → Postgres (99.9%) are in sequence, so the whole request path is about 99.8% available. Two API instances behind a load balancer are in parallel, which is far better than one [S0001].

### What one component can handle  {#component-capacity}

These are the yardsticks for the "compare and decide" step. Modern hardware is much stronger than many candidates assume [S0015] [S0023].

| Component | Rough capacity (modern) | Scale out when… |
|---|---|---|
| App server | 100K+ concurrent connections; 64-512 GB RAM | CPU > 70%, or latency over the SLA [S0015] |
| SQL database (Postgres/MySQL) | up to ~50K transactions/s; a few TB comfortably on one instance | writes > 10K/s, or data in the tens of TB [S0015] |
| Cache (Redis) | 100K+ operations/s; ~1 ms; up to ~1 TB of memory | hit rate < 80% or memory > 80% [S0015] |
| One big server | 512 GiB RAM, 128 vCPUs; 25 Gbps network | [S0023] |

A worked server count: "50K requests per second, each server handles maybe 5K, so about 10 servers plus some headroom" [S0015].

> [!warning] Older tables give smaller numbers
> One course still lists about 1,000 QPS for MySQL and 10,000 for a key-value store [S0022]. With numbers like those you will shard and cache far earlier than you need to [S0015] [S0023]. See "Where sources disagree".

### From estimate to decision  {#estimate-to-decision}

An estimate is only useful when it changes the design [S0004] [S0021]. Put each output next to a yardstick and read off the decision:

| If the estimate says… | It points to… |
|---|---|
| Peak QPS well under one DB/server's limit | Keep one box (plus a spare for availability) [S0015] |
| Reads ≫ writes (e.g. 30,000 vs 300 per second) | Read path first: a cache, read replicas [S0021] |
| Writes > ~10K/s, or data in tens of TB | Split the data across machines (sharding, a later topic) [S0015] |
| Petabytes of files | Object storage for the bytes; the DB keeps only small records [S0021] |
| Hot data fits in memory (GBs, not TBs) | One cache instance can hold it [S0021] [S0015] |
| Gigabytes per second going out | A CDN, so bytes are served near users [S0021] |

Interviewers change one input on purpose to see if you understand the chain [S0021]. If the photo app's uploads go up 5x, every number goes up 5x, but the decisions stay the same [S0021].

### When to estimate, and when not to  {#when-to-estimate}

Hello Interview says to skip the big upfront estimate. Do the maths only when it will change a decision, such as whether one Top-K heap fits on one machine [S0004] [S0015]. Their complaint: candidates compute DAU, QPS and storage, then say "ok, so it's a lot" and design the same thing anyway [S0004].

ByteByteGo puts the estimate into step 2 of its framework, to check that the first design fits the scale, but also says to ask the interviewer whether it is needed [S0014]. It lists QPS, peak QPS, storage, cache and server count as commonly asked [S0020].

> [!interview] What a strong answer sounds like
> "Let me estimate just the numbers that matter here: peak read QPS and 5-year storage. If reads stay under ~10K/s and the data under a few TB, one Postgres with a replica is enough, and I'll say so." Then actually make the decision [S0004] [S0015] [S0021].

## Worked example 1: a Twitter-like service (classic)

The numbers are for practice only, not real Twitter data [S0020]. Assumptions: 300M monthly active users, 50% use it daily, 2 tweets per user per day, 10% of tweets carry media, data kept 5 years [S0020].

```lk-steps
{
  "title": "Twitter-like: tweets QPS and media storage",
  "steps": [
    {"title": "DAU", "phase": "assume", "body": "Monthly users x daily share.",
     "example": "300M x 50% = 150M DAU", "cite": ["S0020"]},
    {"title": "Tweets per day", "phase": "traffic", "body": "DAU x tweets per user.",
     "example": "150M x 2 = 300M = 3 x 10^8 tweets/day", "cite": ["S0020"]},
    {"title": "Average write QPS", "phase": "traffic", "cost": "÷ 10^5",
     "body": "With 10^5 seconds you get 3,000. The source divides by 86,400 exactly and gets ~3,500. Both are fine.",
     "example": "3 x 10^8 ÷ 10^5 = 3,000 (exact ≈ 3,500)", "cite": ["S0020"]},
    {"title": "Peak write QPS", "phase": "traffic", "cost": "x 2",
     "example": "2 x 3,500 ≈ 7,000 tweets/s", "cite": ["S0020"]},
    {"title": "Media per day", "phase": "storage", "body": "Only 10% of tweets have media, about 1 MB each.",
     "example": "300M x 10% = 30M files x 1 MB = 30 TB/day", "cite": ["S0020"]},
    {"title": "Media for 5 years", "phase": "storage",
     "example": "30 TB x 365 x 5 ≈ 30 TB x 1,800 ≈ 55 PB", "cite": ["S0020"]},
    {"title": "Tweet text for 5 years (extra)", "phase": "storage",
     "body": "Not in the source, but uses its sizes: 64 B ID + 140 B text ≈ 200 B.",
     "example": "300M x 200 B = 60 GB/day x 1,800 ≈ 110 TB", "cite": ["S0020"]},
    {"title": "Media upload bandwidth (extra)", "phase": "bandwidth",
     "example": "30 TB/day ÷ 10^5 s = 300 MB/s ≈ 2.4 Gbps average", "cite": ["S0020", "S0025"]}
  ]
}
```

> [!example] So what does this decide?
> - **55 PB of media** is far too big for any database. The files go to object storage, and the database keeps only small records that point to them [S0021] [S0023].
> - **~110 TB of tweet rows** is well past the "few TB" one Postgres instance handles comfortably, so the tweet table must be spread over several machines [S0015].
> - **~7,000 writes/s at peak** is just under the ~10K/s write trigger [S0015]. Writes alone are not the problem; storage size is.
> - Reading timelines will be many times the write load (we did not estimate it here), which points to caching. Later nodes cover how.

## Worked example 2: product reviews in a shopping app

This one is closer to what you build: a Next.js product page that calls a NestJS API, which reads reviews from Postgres. The numbers below are my own assumptions, chosen to be realistic for a mid-size app. The method is the one from [S0021] and [S0025].

| Assumption | Value |
|---|---|
| DAU | 2 million |
| Product pages viewed per user per day | 10 (each page loads one page of reviews) |
| Users who write a review on a given day | 1% |
| One review on disk | ~1 KB (3 IDs x 8 B + rating + ~500 B text + timestamps ≈ 550 B, rounded up for indexes) |
| Reviews per API response | 20, so ~20 KB |
| Peak factor | 3x (a sale day could be 10x) |
| Kept | 5 years, 3 copies (primary + 2 replicas) |

```lk-steps
{
  "title": "Product reviews: every step",
  "steps": [
    {"title": "Reads per day", "phase": "traffic", "example": "2M users x 10 pages = 20M = 2 x 10^7 reads/day", "cite": ["S0025"]},
    {"title": "Average read QPS", "phase": "traffic", "cost": "÷ 10^5", "example": "2 x 10^7 ÷ 10^5 = 2 x 10^2 = 200 reads/s", "cite": ["S0021"]},
    {"title": "Peak read QPS", "phase": "traffic", "cost": "x 3", "example": "200 x 3 = 600 reads/s (sale day at 10x: 2,000)", "cite": ["S0021", "S0025"]},
    {"title": "Writes", "phase": "traffic", "body": "1% of 2M users write one review.", "example": "20,000 reviews/day ÷ 10^5 = 0.2 writes/s; peak 0.6/s. Read:write = 1,000:1", "cite": ["S0021"]},
    {"title": "Storage per day", "phase": "storage", "example": "20,000 x 1 KB = 20 MB/day", "cite": ["S0025"]},
    {"title": "Storage for 5 years", "phase": "storage", "example": "20 MB x 1,800 days ≈ 36 GB ≈ 40 GB\nx 3 copies ≈ 120 GB", "cite": ["S0021", "S0025"]},
    {"title": "Outgoing bandwidth at peak", "phase": "bandwidth", "example": "600 x 20 KB = 12 MB/s x 8 ≈ 100 Mbps", "cite": ["S0021", "S0025"]},
    {"title": "Compare with yardsticks", "phase": "decide", "example": "600 reads/s vs DB up to ~50K/s\n40 GB vs a few TB per DB, 512 GiB RAM per server\n100 Mbps vs a 25 Gbps network card", "cite": ["S0015", "S0023"]}
  ]
}
```

> [!example] So what does this decide?
> - **One Postgres is plenty.** 600 reads/s is far under what one database can do [S0015]. Even the old, cautious figure of ~1,000 QPS for MySQL covers it [S0022].
> - **No sharding.** 40 GB (120 GB with replicas) is far below "a few TB" per instance [S0015]. The whole table would even fit in one server's RAM [S0023].
> - **A cache is optional, not forced.** Add one for speed if you like, but the numbers do not require it [S0015].
> - **Run two NestJS instances anyway**, for availability, not for capacity: two in parallel beat one [S0001].
> - **What would change the answer?** At 20M DAU and a 10x sale peak, reads reach 20,000/s. Now a cache and read replicas are worth discussing [S0021] [S0015]. The method is the same; only one input changed.

## How it works: the numbers drive the boxes

The same method gives a very different architecture depending on the result. Compare the reviews feature with the photo-sharing service from [S0021].

```lk-arch
{
  "title": "Same method, two outcomes",
  "panels": [
    {"title": "Reviews: 600 reads/s, 40 GB", "flow": [
      {"box": "Browser", "sub": "Next.js product page", "kind": "client"},
      {"arrow": "GET /products/:id/reviews"},
      {"box": "NestJS API", "sub": "2 instances, for availability", "kind": "app"},
      {"arrow": "SQL"},
      {"box": "Postgres", "sub": "one primary + replica", "kind": "data"}],
     "pros": ["every number far under one box's limit", "simple to run"],
     "cons": ["recheck if DAU grows 10x"]},
    {"title": "Photo app: 30K reads/s, 30 PB", "flow": [
      {"box": "Users", "kind": "client"},
      {"arrow": "15 GB/s out at peak"},
      {"box": "CDN", "sub": "serves popular photo bytes (~1 TB)", "kind": "edge"},
      {"arrow": ""},
      {"box": "App servers", "kind": "app"},
      {"arrow": ""},
      {"box": "Cache", "sub": "photo records, ~2 GB", "kind": "cache"},
      {"arrow": ""},
      {"box": "DB + object storage", "sub": "records in DB, bytes in object storage", "kind": "data"}],
     "pros": ["each box is justified by one number"],
     "cons": ["more parts to run"]}
  ]
}
```

In the photo app each number forces one box [S0021]. In the reviews feature, no number forces anything [S0015].

## Trade-offs: when to estimate, and how much

| Situation | Do this |
|---|---|
| The number could change a decision (one box or many, fits in memory or not) | Estimate it, then say the decision out loud [S0004] [S0015] |
| The interviewer asks "how many servers?" | Walk through peak QPS ÷ per-server capacity, plus headroom [S0015] |
| The answer is obviously "huge" and changes nothing | Skip it or say it in one line [S0004] |
| Not sure what the interviewer wants | Ask whether they want estimates now [S0014] |

Spend your minutes on the numbers that matter: usually peak QPS, read:write ratio and total storage [S0020] [S0004].

## Common misconceptions

> [!misconception] "I need the exact answer."
> You need the right order of magnitude. Rounding "99,987 / 9.1" to "100,000 / 10" is expected [S0020]. Carrying three significant figures through guesses gives a false sense of precision [S0021].

> [!misconception] "Average QPS is what I design for."
> Systems fall over at the peak. Always multiply by a peak factor (2x-10x) and say which one you chose [S0025] [S0020].

> [!misconception] "MB/s and Mbps are the same."
> One byte is 8 bits [S0020]. 500 MB/s is about 4 Gbps [S0025]. Mixing them up gives an 8x error.

> [!misconception] "Storage is size x items. Done."
> Also multiply by retention and copies: 10 PB of photos becomes 30 PB with three copies [S0021] [S0025]. But do not apply the peak factor to storage [S0021].

> [!misconception] "A database tops out around 1,000 QPS, so shard early."
> That is an old number. A well-tuned database handles tens of thousands of queries per second, and Redis hundreds of thousands [S0015] [S0023]. Sharding at 500 GB adds complexity for no reason [S0015].

> [!misconception] "Estimation is the goal."
> The estimate is a tool to make a decision. "Ok, so it's a lot" earns nothing [S0004]. "So the bytes go to object storage" does [S0021].

## Where sources disagree or are unclear

- **Estimate upfront or only when needed?** Hello Interview says skip the upfront estimate and calculate only when it changes a decision [S0004] [S0015]. ByteByteGo puts it in step 2 to check the design fits the scale, while also saying to ask first [S0014] [S0020]. The System Design Handbook says interviewers often ask for it early, and that it becomes "non-negotiable" from mid-level up [S0025]. Safe path: ask, then estimate only the numbers that drive a decision.
- **How strong is one database?** About 1,000 QPS for MySQL in one course [S0022], against up to ~50K transactions/s in another [S0015]. The second is newer and explicitly warns about outdated numbers [S0023]. Use the modern figure, but know the old one exists.
- **Latency inside a datacenter.** 500 µs in the classic table [S0019] [S0001]; sub-1 ms within one zone and 1-2 ms across zones [S0023]; "1-10 ms" for network calls in another source [S0015]. All agree it is about 1,000x RAM and far below cross-region.
- **Small differences in the latency table.** Mutex lock/unlock is 100 ns in one copy [S0020] and 25 ns in another [S0019]. Reading 1 MB from disk is 30 ms [S0020] [S0001] or 20 ms [S0019]. One course gives very different values, such as 2 ms for 1 MB from disk [S0022]. Use the ranking, not the digits.
- **Peak factor.** 2x [S0020], 3x [S0021], 3-10x [S0025]. There is no single right value.
- **Partial snapshots.** The "Numbers to Know" page stops before its component table [S0023], so the capacity table uses its summary [S0015]. The Jeff Dean slides are not readable as text [S0024]; the definition is quoted via [S0020].

## Further reading

- [S0020] ByteByteGo: the definition, three reference tables, the Twitter example.
- [S0021] DesignGurus: a full photo-app estimate ending in four design decisions.
- [S0025] System Design Handbook: the step-by-step method and unit cheat sheets.
- [S0019] / [S0001] The latency, powers-of-two and availability tables.
- [S0015] / [S0023] Hello Interview: modern capacity numbers and scale triggers.
- [S0004] / [S0014] Two interview frameworks with different views on when to estimate.
- [S0022] Educative: BOTEC basics, with older rate numbers.
- [S0024] Jeff Dean's Stanford talk on building large distributed systems.

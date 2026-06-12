# Real-World Systems: How They're Built and How They Fail — Video Summary & Mini-Lesson

> **Videos:** [Cloudflare, AWS… ¿Por qué se cae medio Internet?](https://www.youtube.com/watch?v=zkSSoEpL7zU) and [¿Cómo funciona la infraestructura de Twitter? Detrás de la red social de Elon Musk](https://www.youtube.com/watch?v=waHk1grLR5U) — BettaTech
> *("Cloudflare, AWS… Why is half the internet going down?" and "How does Twitter's infrastructure work? Behind Elon Musk's social network")*

---

## 📋 Brief Summary

Two case studies that ground abstract system design in the real world: **how a large system is actually architected** and **how internet-scale infrastructure actually fails**. The Twitter video reverse-engineers (from Twitter's own engineering blog) how Twitter search works — Elasticsearch fed through a proxy for reads and a Kafka-based ingestion service with batching workers for writes, landing naturally on the **read/write separation** pattern. The outages video, recorded mid-Cloudflare-outage and a month after the big AWS one, explains *why* a single company breaking takes down X, ChatGPT, and even McDonald's ordering screens: CDNs and proxies sit **in front of** everyone's servers, AWS's own services cascade through a DynamoDB-backed DNS layer, and the internet's compute has **centralized into a handful of providers** — creating critical single points of failure. The shared message: a feature that looks trivial ("a search bar", "serving a web page") hides serious infrastructure — and that infrastructure is worth understanding both to build it and to survive its failures.

---

## 🎓 Mini-Lesson: The Concepts

### Part A — Inside Twitter's search infrastructure

#### 1. "Why does Twitter need thousands of developers?"

- After Musk's acquisition and the wave of developer layoffs, a common reaction was: *"how did Twitter have so many developers when the app barely changes?"* To the user's eye, Twitter doesn't visibly evolve — so what do thousands of engineers do?
- The presenter dug into **Twitter's engineering blog** for the answer, focusing on one "simple" feature: **search** (searching tweets and users). Twitter has an **entire infrastructure team dedicated just to search**.

#### 2. The engine: Elasticsearch (on Lucene)

- Twitter's search engine is **Elasticsearch** — a document database specialized in search.
- Underneath, Elasticsearch uses **Apache Lucene**, a product focused on **indexing text content** (books, paragraphs, articles, whatever): you type a small phrase and it finds which texts contain it, fast. That makes it the perfect base for a search database: index all the content, then query it.
- You can deploy Elasticsearch on your own infrastructure and just throw content at it: every time a tweet is written → boom, send it to Elasticsearch, which eats and indexes it.

#### 3. The naive setup breaks: too many connections, too much ingest

- Direct model: every user connects straight to Elasticsearch. One user, one connection — fine. But **every new user is another connection**, and how many users does Twitter have? The more users, the worse Elasticsearch behaves — it's eating all the connections directly and becomes **unstable**.
- Same on the write side: the more tweets created per minute, the "fatter" the ingest line attacking the cluster. Add ever-more users and you have a swarm of arrows hammering the Elasticsearch cluster — your search foundation is **constantly dying**.

#### 4. Fix #1 — a proxy in front of reads

- Twitter's move is "the infrastructure secret" seen across the channel: **don't connect users directly** — put a **proxy** in between. Users connect to the proxy; the proxy connects to the Elasticsearch cluster.
- The win is a **reduction by n**: thousands of users can attack the same proxy while it keeps **one connection** to Elasticsearch — per ~1,000 users, one ES connection. The proxy itself can be scaled, but the user count can grow much faster than the proxy count needs to. Elasticsearch's load gets dramatically lighter.

#### 5. Fix #2 — an ingestion service in front of writes

- Writes were still on fire: the thing inserting tweets into Elasticsearch was the **tweet-creation API itself** — not a special service. Every change to how tweets enter ES meant reimplementing it there.
- Solution: add an **ingestion service** — *"almost everything is solved by adding services in between; the key is knowing **when**, and **what** the service should do."* Same idea as the proxy: the ingestion service becomes **the only service allowed to write into Elasticsearch**.
- It carries real logic: **queueing**, **slowing down requests**, and holding tweets **in memory** until Elasticsearch signals it can accept more — all of which moves out of tweet-creation and into one place.

#### 6. Batching — because real-time is dangerous

- Per the blog, instead of inserting each tweet one-by-one in real time, the ingestion service builds **batches**: e.g., collect 50 tweets, send all 50 in a single Elasticsearch request; next 50, another request.
- This slashes the number of communications: instead of, say, **400 requests per second**, Elasticsearch receives **20 bigger ones**. Some services prefer many small payloads; others — like Elasticsearch — work better with **fewer, larger packets**.
- Batching is a recurring technique in real-time data work, because **real-time is dangerous: you can drown a service**.

#### 7. Inside the ingestion service: Kafka + an army of workers

- The proxy is simple (user makes an HTTP call, proxy makes an HTTP call to ES). The ingestion service — queues, batches — is built on **Kafka**: an open-source system for **distributed event/streaming workloads**, heavily used for analytics, user-behavior tracking, and Big Data.
- Flow: each tweet creation **emits an event** to a Kafka **topic** (its name for a queue) — something like a JSON `{type: tweet_created, user: bettatech, content: ...}` (the presenter's guess at the shape, not Twitter's actual schema). Kafka **accumulates** the events; the queue fills up.
- Then an **"army of workers"** — a replicable, scalable set of services — **consumes the Kafka queue**: grab, say, 7 tweets, process/convert them to the format Elasticsearch needs, and send them over in one request.
- Scalability is now dial-able: Kafka is built to absorb huge data volumes, and **the more items in the queue, the more workers you spin up** (if you want every tweet searchable within *x* minutes) — while still respecting Elasticsearch's ingest limit.

#### 8. What you've accidentally built: read/write separation

- Step back and the architecture splits cleanly: **reads** (user → proxy → Elasticsearch queries: "give me all tweets matching this") and **writes** (tweet creation → Kafka → workers → Elasticsearch).
- This is the classic **read/write separation** pattern that appears whenever you scale services handling large data volumes: separate the responsibility of reading from the responsibility of writing, so search capacity stays high **independently** of whether ingestion is keeping up — you control both sides at once.
- Reads are normally **cheaper** than writes: here reads barely need extra scaling, while writes need queues and workers.

#### 9. The reflection: don't scale prematurely

- A problem that sounds trivial — *"a search bar where I type text and get the list of things containing it"* — can genuinely require a dedicated infrastructure team, and this wasn't built in a day, a week, or a month: Twitter's solution **evolved** as growth surfaced problems that didn't exist before.
- But **don't scale from day one**: with 100 users creating tweets, a plain database query handles search just fine. Only when growth makes your search misbehave is it time to weigh investing the **time and money** for a solution like this.

### Part B — Why half the internet goes down

#### 10. The incidents

- **A month earlier**, Amazon Web Services went down, *"paralyzing practically the whole internet."* **Now Cloudflare** goes down too — breaking access to **X (Twitter)**, **ChatGPT**, and piles of websites that use it for different reasons. The presenter even saw photos of **McDonald's ordering screens** that couldn't take orders, because the kiosk interface is basically a web page.
- The puzzle: outages hit even products that **don't use Cloudflare as their web server**. So why did they fall?

#### 11. Reason 1 — Cloudflare as CDN: the middleman fails

- Your hosting may be **physically far** from your user: user in Spain, server in Ireland (the eu-west-1 case on AWS — Ireland or even London) or, worse, in the US. Every request pays the physical travel time of the signal.
- A **CDN** is a proxy/small server that **caches your server's responses in a location near the user**: first request from Spain isn't cached → the CDN fetches it from the origin, caches it, returns it. From then on, everyone matching that CDN location gets the cached version — massively speeding up global distribution of pages and files.
- **But if the CDN itself goes down, it stops proxying** — the communication is simply cut, and the page fails. That's exactly what hit most products using Cloudflare as a CDN: their servers, hosted somewhere else entirely, were fine. Hence the error page in effect saying: *the browser works, the host works, but the intermediate Cloudflare CDN/proxy failed*.

#### 12. Reason 2 — Cloudflare as protective proxy

- The second big use: Cloudflare as a **filtering proxy** — same intermediary concept as a CDN, different goal. Instead of distributing content efficiently, these intermediate servers act as a **filter**: a horde of bots launches a **DDoS attack** at your server, and Cloudflare blocks the malicious traffic while letting normal traffic through.
- CDN + proxy are among Cloudflare's most-used products across a huge number of companies — which is why **a large part of the internet** went down with it.

#### 13. What the status page showed (no postmortem yet)

- Checking Cloudflare's status page live: problems started around **12:00 UTC**, suspiciously aligned with scheduled **maintenance windows** (one in Tahiti also started around 12) — *"may or may not be related; we don't know, the postmortem isn't out yet."* A fix had been announced about an hour earlier, but mitigation was still ongoing.
- *(Dated note: Cloudflare's later official postmortem attributed the November 2025 outage to an oversized, auto-generated bot-management configuration file — not the maintenance windows. The video predates it and is explicit that it's speculating.)*

#### 14. The AWS outage of October 19: a DNS cascade

The AWS failure, unlike Cloudflare's, already had a **detailed postmortem**, which the video walks through:

- The root failure was in **DynamoDB** — AWS's key-value database, designed for performance and query speed, used by **many AWS services internally**, notably to implement **DNS services**.
- **DNS recap:** it translates a domain (e.g., `www.youtube.com`) into an IP. That intermediary table is what lets you **change hosts and scale horizontally** while keeping the same domain — "YouTube is at this IP now; I scale, add more IPs, and distribute load across them." (His "roughly speaking" version of DNS.) In AWS, this is built on DynamoDB.
- Errors started appearing in DynamoDB in **us-east-1 (North Virginia)** — one of AWS's biggest zones — and then **failures cascaded**:
  - **EC2** (the actual hosts where your Node or Python servers live) uses DNS for server management → **EC2 fell** in the zone.
  - **Load balancers** use DNS to know which host to route load to → they broke too.
  - In total, "a ton and a half" of services were affected — **queues, Kinesis**, and many more — causing an hours-long outage of most internet services using AWS, specifically in Virginia. **Europe was less affected**, since European workloads commonly live in Ireland.

#### 15. The real diagnosis: centralization

- Internet distribution has ended up **centralized in a few companies** that make these services easy to consume: **Google Cloud, AWS, Cloudflare, DigitalOcean…** — and some of these products **use each other under the hood**. Your CDN contracted from provider X may run on Cloudflare underneath, so you got hit anyway.
- Running data centers and global CDNs takes **enormous money and infrastructure** — so only a few companies can support what the internet needs to stay up. Putting CDNs, filters and proxies *in front of* everything creates a **critical point of failure**: when that point falls, **everything behind it falls too**.

#### 16. The advice: design for your provider's bad day

- **Make the middleman toggleable:** if you use Cloudflare (or similar) as CDN/proxy, have a simple way to **activate/deactivate it quickly**. Disable it during an outage and your own servers eat much more load — but at least users get in.
- **Keep alternative CDN providers** you can switch to quickly, so a Cloudflare outage doesn't equal *your* outage.
- **On AWS, think multi-region:** the presenter's own academy survived the AWS outage because everything ran in **Ireland** — had it been in the US (or had Ireland fallen), it would have been a very big problem. You can keep a **minimal infrastructure in another AWS region**, ready to activate fast, and migrate DNS records during the outage.
- **Only if it's worth it:** big outages aren't that frequent — AWS has maybe **one or two big falls a year** (and Cloudflare, having just fallen, "hopefully" won't again for months). If your product can tolerate waiting for the fix, doing nothing is also a valid choice.

---

## 🧭 The Big Picture (mental map)

```
 PART A: BUILDING ONE (Twitter search)          PART B: WHEN THE FOUNDATIONS FAIL
 ─────────────────────────────────────          ──────────────────────────────────
            users                                    users
         ┌────┴────┐                                   │
       READS     WRITES                         Cloudflare in the middle
         │         │                            ┌──────┴───────┐
         ▼         ▼                            ▼              ▼
      [Proxy]  tweet-creation API             CDN          filtering PROXY
   (1000s of      │ emits event            (cache near     (blocks DDoS /
    users →       ▼                         the user)       bot traffic)
    1 ES conn) [Kafka topic]  ──┐               │              │
         │        ▼             │ ingestion     └──── falls ───┘
         │   [army of workers]  │ service              │
         │    batch ~50 tweets ─┘            origin server is FINE,
         │    (400 req/s → 20 big ones)      but the middleman is gone
         ▼         ▼                         → "half the internet" errors
      [ Elasticsearch (Lucene) ]
         indexes text → search               AWS (Oct 19): DynamoDB ─→ DNS
                                                 │ cascades in us-east-1
  = READ/WRITE SEPARATION pattern                ▼
  (reads cheap; writes need                  EC2 hosts → load balancers
   queues + workers)                          → queues, Kinesis, ...

      WHY IT'S SYSTEMIC: compute is CENTRALIZED in a few providers
      (Google Cloud, AWS, Cloudflare, DigitalOcean — some built on
       each other) → critical single points of failure.
      MITIGATE: toggleable CDN/proxy · backup CDN provider ·
      minimal standby in a 2nd AWS region + DNS switch · or just wait.
```

---

## 💡 Key Takeaways

1. **"Simple" features hide whole teams:** Twitter's search bar needs Elasticsearch, a proxy, Kafka, and an army of batching workers — built incrementally as growth surfaced new problems.
2. **Almost everything is solved by adding a service in between** — a proxy collapses thousands of user connections into one; an ingestion service makes itself the only writer and absorbs the chaos. The skill is knowing *when* and *what for*.
3. **Batch instead of real-time when you can:** real-time is dangerous — sending 20 big requests instead of 400/s keeps services like Elasticsearch alive. Separate reads from writes and scale each side on its own.
4. **The internet has centralized into a few providers** (AWS, Cloudflare, Google Cloud, DigitalOcean — some stacked on each other), so CDNs and proxies *in front of* everyone's servers become critical single points of failure — and failures cascade internally too (DynamoDB → DNS → EC2 → load balancers).
5. **Plan for your provider's outage in proportion to your stakes:** quick CDN on/off switch, a backup CDN, a minimal standby region with a DNS flip — or consciously accept the downtime, since giant outages happen only about once or twice a year.

---

*Related summaries: [System Design Thinking](system-design-thinking-video-summary.md), [DevOps Concepts](devops-concepts-video-summary.md), [Building Atlassian's Edge Platform](atlassian-edge-platform-video-summary.md) (a first-person account of building exactly this kind of proxy/edge infrastructure)*

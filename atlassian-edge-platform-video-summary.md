# Building Atlassian's Edge Platform: 8 Years in Retrospect — Video Summary & Mini-Lesson

> **Video:** [I was laid off by Atlassian](https://www.youtube.com/watch?v=55pTFVoclvE) — Vasilios Syrakis (40:06)

---

## 📋 Brief Summary

After being laid off in Atlassian's layoffs, the author looks back on his **eight years** there and walks through what he built — mostly the **edge/load-balancing platform**: a self-service provisioning broker, a dynamic Envoy control plane (open-sourced as **Sovereign**), the CloudFormation + Packer/SaltStack infrastructure behind a fleet of **~2,000 proxies across ~13 regions**, and the centralization of cross-cutting concerns (auth, DDoS protection, rate limiting, access logs) at the edge so a thousand dev teams don't each reimplement them. He closes with the non-technical lessons: maintenance and codebase **churn as a smell**, onboarding people for on-call, diplomacy and personality conflicts, and why mentoring an intern felt harder than teaching colleagues. The core message: a platform team creates leverage by turning "talk to us to get a load balancer" into validated self-service configuration, and by solving shared concerns **once, early in the request chain**, instead of a bazillion times in backend services.

---

## 🎓 Mini-Lesson: The Concepts

### 1. The interview — and the question that defined the job

- Process (8 years ago): a **HackerRank coding quiz** (full marks) → a technical interview where they handed him a **Cloudflare white paper about custom domains**, left the room for ~10 minutes, and asked him to articulate it back, plus questions on microservices, containers, architecture → a **troubleshooting interview** replaying a real Atlassian incident (an application problem that led to a denial of service), where the candidate must *prompt the interviewer* for information → a values interview.
- One question he got wrong-but-acceptably: how **latency-based DNS** works. He reasoned from first principles (Route 53 triangulating on actual client latency); in reality it's more likely done with a **geolocation database**.
- His own best question to the interviewers: *"Imagine 12 months from now, looking back — what would I have to achieve for you to say hiring me was a good decision?"* The answer: build an internal app for **self-service load balancers** (like AWS ALBs, but for Atlassian's internal developers). He'd never used the framework involved, but was confident from building Python web apps — and they hired him on that confidence.

### 2. First build: an Open Service Broker (async provisioning pattern)

- Joining Atlassian is "**drinking from the fire hose**." His self-assigned first task: build the app he'd promised — an **Open Service Broker (OSB)**: a web app + API that facilitates **provisioning of resources for a platform** (the spec is public on GitHub, including an OpenAPI document; it's built for a Kubernetes-style world of resources binding to pods/instances, with a **catalog endpoint** listing services and plans, plus provision/update/delete endpoints).
- Tech evolution: first built with **Connexion** (a Python library that generates API route handlers from an OpenAPI document) → migrated to **pure Flask** → eventually **FastAPI** (still the stack today, as far as he knows). At Atlassian, provisioning wasn't console-clicking — config files committed to **version control**, uploaded from a build server on deploy.
- Architecture: **FastAPI web app + SQS + worker + DynamoDB**. The client asks "please provision X"; the web tier doesn't do the work itself — it **drops task details into SQS**; the **worker** executes the provisioning task asynchronously (creating DNS records, a CloudFront distribution, API calls…) and writes the result to the database; the client **polls** ("is it ready?") until the web tier reads the status and answers done (or errored).

### 3. Replacing enterprise load balancers with Envoy

- An architect's idea: replace Atlassian's **enterprise load balancers** (with licensing costs) with an **open-source, cloud-native commodity proxy** — they chose **Envoy** (similar role to Nginx, but more modern), and make load balancing **self-service** so devs don't need to talk to the team.
- The key Envoy feature: an **API for dynamic configuration** — config can be reloaded at runtime. So you deploy a fleet of proxies that sit running all the time, and when someone needs different configuration for their service, the change **flows to the proxies** without redeploying them.

### 4. Sovereign — the Envoy control plane (templates + context)

- He built the **Envoy management server** ("Envoy control plane"), open-sourced as **Sovereign** (public repo on Bitbucket, "at least for now"). Also a FastAPI app.
- The model: Sovereign loads **templates** (per Envoy resource type: **clusters, routes, listeners**…) and **context** (dynamic data), and exposes the rendered results as APIs that the proxies request. **Context comes from polling multiple sources** — the broker's database (the provisioning state) and other sources like an **S3 bucket** whose data changes over time.
- End-to-end flow: client → broker → worker writes new data to the database → Sovereign polls that data (plus other sources) → feeds it into templates → renders new Envoy configuration → the proxy picks it up **and starts behaving differently**. Developer input is reduced to "a little bit of JSON": simple parameters, **validated**, then fed into template logic that produces valid Envoy resources.
- Where the real engineering concentrated: **validation and abstraction**. Envoy config is huge (virtual hosts, domain matching, routing, direct responses, redirects, header manipulation…), and a route action can send traffic to **any cluster on the proxy** — with a thousand services each owning a cluster, the platform must validate parameters so the templates can only render safe, valid resources. It looks easy to him now, but he suspects that's the **curse of knowledge**.

### 5. The fleet: CloudFormation + the AMI pipeline

- The proxies themselves: provisioned by a **CloudFormation template** (infrastructure as code) — VPC, subnet, internet gateway, security group, key pair, IAM role, **auto scaling group** creating the EC2 instances, an **NLB** (layer-4) in front, **ACM** for certificates, some Route 53 records. Scale: roughly **2,000 proxies in ~13 regions**.
- The ASG needs an **AMI** — referenced by the template, not produced by it. The image pipeline: **HashiCorp Packer** (EC2 provisioner: spin up an instance in a dev account, provision it, snapshot it into an image) + **SaltStack** configuration (config management à la Puppet/Ansible/Chef — declarative install-packages/put-files/run-services in a particular order).
- Baked into the AMI: install & configure **Envoy**, **logging agents**, **security/hardening**, **network tuning**, **containers**, and an **observability agent** (logging, tracing, metrics). At boot, CloudFormation **parameters** pass in secrets and keys; the proxies grab their resources, configure themselves, and accept traffic. That was roughly **the first two years** — the foundation of the team's product: centralized load balancing, with all customer-facing features living in template logic.

### 6. The migration: platform leverage and secure-by-default

- Next phase, two big parts: making the **larger products** able to use the platform, and **migrating all Atlassian microservices** onto it.
- The microservice migration was the easier one **because the platform could enforce it**: the platform's old, very basic load balancing stopped allowing public exposure, so to be publicly accessible you had to go through the centralized edge **and explicitly configure it** — turning public exposure into a deliberate, signaled intention rather than something accidental and poorly protected.
- The big products took a couple of years of feature-building for their special cases on what is effectively a **generic multi-tenanted platform** — but **Jira, Confluence, Bitbucket, Statuspage** and many others ended up behind this edge infrastructure.

### 7. Centralizing cross-cutting concerns at the edge

- The groundwork created **opportunity**: handle concerns **early in the request chain**. Customer traffic flows: customer → NLB → proxies → backend services. Anything dealt with at the proxy is solved **before** it reaches "a bazillion" backend services — saving money, time, and feature velocity (imagine a thousand dev teams each implementing auth, DDoS protection, rate limiting, and access logs themselves).
- How each concern landed:
  - **DDoS protection** — provided via **CloudFront** in front (an effort spearheaded by a smart, conscientious colleague).
  - **Access logs** — **natively in Envoy**, via network filters (the **HTTP connection manager**), all configured through the same dynamic templates.
  - **Authentication, authorization, rate limiting** — too complex for native config: a **sidecar model**, with Envoy calling out to services running locally on the proxy (as containers), installed and configured by the same AMI pipeline. He wrote the **authentication sidecar in Rust** ("the Lord's language"); authorization and rate limiting came from **other teams** contributing sidecars. Sidecars also receive **dynamic configuration over the wire**, making the proxy even more programmable.
- After that came **compliance** requirements — no new building, just making sure everything met the rules: "very boring checklist-ticking work."

### 8. Maintenance lessons: churn is a smell

- Building something new comes with a known up-front cost: docs, onboarding, training people for **on-call** — what log messages mean, which metrics to check, what an AWS outage does to you ("what if SQS stops working and provisioning halts? what if a proxy receives *valid* configuration that destroys traffic?").
- The harder part is **time**: people leave, new people bring new opinions and change things, and **churn concentrates predictably** in certain areas of the codebase. Recurring churn is a **smell** — a signal that that part will keep growing in size or complexity and something must be done before it becomes a mess.
- His sharpest line: **"Building something is easy. Changing it — and making sure you can still change it over time — is difficult."** Things slowly couple; a change in one area affects another, and you inherit a detangling task. He wonders how **vibe-coded / AI-assisted apps** will fare when maintenance burdens appear and their owners aren't really familiar with what they created — maybe an LLM can do the detangling ("if we can do that, that's fantastic — but I don't want to be too optimistic").

### 9. People lessons: diplomacy, conflict, mentoring

- Eight years exposed him to many manager/colleague personality types → real growth in **diplomacy, conflict avoidance/resolution, persuading, proposing, teaching**. Some personality mismatches are near-inevitable even among people you respect; the only lever is **self-awareness plus awareness of the other person**, anticipating the conflict and taking responsibility for making the relationship work. Conflicts caused him real stress and at times **affected his performance** — which is why he took the lesson seriously.
- **Mentoring vs. teaching:** explaining systems, breaking complex things into simple terms, being available to help — that was his "bread and butter" (and the consistent feedback from colleagues). But true *mentoring* — his intern last year — felt different and difficult: balancing **how much time to give and what it consists of**, not handing out answers but not letting the mentee hit frustrating dead-ends. The intern got the **highest possible rating** (effectively guaranteeing a return offer) with help from several subject-matter experts — but he's honest that he can't fully attribute the success to himself, and that having **never been mentored**, he lacks a reference for what good mentoring looks like.

---

## 🧭 The Big Picture (mental map)

```
                      SELF-SERVICE PROVISIONING (control plane)
 dev's JSON params
       │ "provision a load balancer"
       ▼
 [Open Service Broker]  FastAPI ──► SQS ──► worker ──► DynamoDB
  (Connexion→Flask→FastAPI)         async tasks: DNS records,
       ▲ client polls "ready?"      CloudFront, API calls…
       │                                      │ state
       │                                      ▼
       │            [Sovereign]  ◄── polls ── DB + S3 + other sources
       │       Envoy control plane
       │       templates (clusters/routes/listeners) + context
       │       validate params → render config dynamically
       │                  │ config over the wire (runtime reload)
       ▼                  ▼
 ┌────────────────────────────────────────────────────────────┐
 │  ENVOY FLEET  (~2,000 proxies, ~13 regions)                │
 │  CloudFormation: VPC/subnet/SG/IAM/ASG/EC2 + NLB + ACM     │
 │  AMI pipeline: Packer + SaltStack (Envoy, logging,         │
 │   hardening, net tuning, containers, observability agent)  │
 │  sidecars: authn (his, Rust) · authz · rate limiting       │
 └────────────────────────────────────────────────────────────┘
                          ▲
                          │ DATA PLANE
 customer ──► CloudFront (DDoS) ──► NLB ──► Envoy ──► backend services
              concerns solved HERE (auth, rate limit, access logs)
              instead of in a bazillion backends
              → Jira, Confluence, Bitbucket, Statuspage behind it
```

---

## 💡 Key Takeaways

1. **Platform teams sell leverage:** turn "file a ticket for a load balancer" into validated self-service — devs send a little JSON, templates render safe Envoy config, and the whole edge reacts dynamically without redeploys.
2. **Solve cross-cutting concerns once, early in the request chain:** auth, DDoS protection, rate limiting and access logs at the proxy (native filters or sidecars) instead of in every one of a thousand backend services.
3. **Async provisioning is a reusable shape:** web tier → queue (SQS) → worker → database, with the client polling for completion — the web server never does the slow work itself.
4. **Enforce migrations through the platform:** making explicit edge configuration the *only* way to be publicly accessible both completed the migration and made public exposure a deliberate, signaled act instead of an accident.
5. **Building is easy; staying changeable is hard.** Watch where churn concentrates — it's a smell predicting complexity growth — and budget for onboarding, on-call training, and detangling. (Open question he raises: who maintains vibe-coded apps when the burdens appear?)

---

*Related summaries: [Real-World Systems: How They're Built and How They Fail](real-world-systems-video-summary.md) (proxies/CDNs in front of everything, centralization as single point of failure) · [DevOps Concepts](devops-concepts-video-summary.md) (IaC, observability, on-call fundamentals).*

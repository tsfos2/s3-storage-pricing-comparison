# s3 storage pricing comparison: what 1 TB really costs on AWS, Google Cloud, Wasabi, Backblaze and Vultr, plus the fees that decide your real bill

List prices for S3-compatible object storage run from **$2.63 to $23 per TB per month**. Almost nobody's invoice matches those numbers. The storage rate is the one fee providers put on the front page; egress, per-request charges and minimum storage durations are what the bill is actually made of, and those vary far more between providers than storage rates do.

One example before any tables. A terabyte on AWS S3 Standard costs $23 a month. Serve five terabytes out to the internet in that same month and the transfer alone adds about $441 at AWS's tiered rates. The same 5 TB of egress costs roughly $50 at Vultr's listed $10/TB, and $0 at Cloudflare R2 or Wasabi. That's why an S3 storage pricing comparison that only lines up storage rates will steer you wrong about half the time.

> **In a hurry?**
>
> - Cold backups you rarely restore: Wasabi ($6.99/TB, 90-day minimum retention), Backblaze B2 ($6/TB, free egress up to 3× stored data) or iDrive E2 ($4.95/TB) run roughly 70–80% below S3 Standard.
> - Heavy downloads or CDN origin storage: the egress column decides it. Cloudflare R2 and Wasabi list $0/TB out.
> - General-purpose S3-compatible storage with moderate read traffic: Vultr Object Storage lists **$18/TB** storage, $10/TB egress and $0.01 per 1,000 requests, 1 TB minimum, no lock-in — 👉 [check Vultr's current Object Storage pricing](https://www.vultr.com/products/object-storage/?ref=7268283).

## The headline numbers: storage, egress and requests side by side

On list price alone, standard-tier storage spans from $2.63 per TB (MEGA S4) to $23 (AWS S3 Standard). The egress column is where providers really diverge: $0 at four of them, $120 at Google Cloud. Here are the current listed rates:

| Provider and plan | Storage per TB/mo | Egress per TB | Reads per 1,000 | Fine print worth knowing |
| --- | --- | --- | --- | --- |
| AWS S3 Standard | $23.00 | $90.00 | $0.0004 | Volume discounts only above 50 TB; deepest archive options in the industry |
| Google Cloud Storage Standard | $20.00 | $120.00 | $0.0004 | Highest egress of the majors; 99.95% SLA |
| Azure Blob Storage (Hot) | $20.48 | $87.00 | — | Comparable storage to AWS, slightly lower egress |
| Cloudflare R2 | $15.00 | $0 | $0.00036 | Zero egress fees; per-op charges still apply |
| Backblaze B2 | $6.00 | $10.00 (free up to 3× stored) | $0.001 | US, Netherlands, Germany only |
| Wasabi Hot Cloud Storage | $6.99 | $0 | free | 90-day minimum retention on stored objects |
| iDrive E2 | $4.95 | $10.00 (free up to 3× stored) | free | Cheapest list price in this set |
| Hetzner Object Storage | $12.30 | $1.20 | free | Germany and Finland locations only |
| Contabo Object Storage | $10.96 | $0 | free | 95% SLA, the lowest in this table |
| MEGA S4 | $2.63 | $0 | free | 3 TB minimum to start |
| **Vultr Object Storage** | **$18.00** | **$10.00** | **$0.01** | 1 TB minimum; US, Netherlands and Singapore; 99.9% SLA; no minimum term |

Three things to notice when you read this table. Uploads are $0 at every provider listed — only downloads get billed. "Free" egress usually comes with a string attached: Wasabi's 90-day retention rule, or the 3× multiplier cap at Backblaze and iDrive E2. And per-read pricing ranges from $0.00036 to $0.01 per 1,000 requests, a spread that only matters once you're serving millions of small objects — more on that below.

## Three fees that decide your real bill

Egress, request pricing, and minimum-duration rules. AWS alone bills across eight separate dimensions — storage, requests, data transfer, management features, replication, query features, plus its newer vector and table storage — but most surprise charges trace back to the first three.

### Egress: the fee that scales with your traffic

Data leaving the provider's network is the expensive direction, everywhere. AWS charges $0.09/GB after the first 100 GB free each month, stepping down to $0.05/GB only past 150 TB. Google Cloud's effective rate works out to $120/TB, Azure's to $87/TB.

The worked example from earlier deserves a second look, because it changes provider choice outright. A site serving 5 TB/month of images pays $441 delivering straight from S3. Route the same traffic through CloudFront — the S3-to-CloudFront hop is free, and CloudFront's first 1 TB of outbound transfer is free — and it drops to about $340. On Vultr's listed rates, 5 TB out costs around $50. On R2 or Wasabi, $0. At AWS prices, 5 TB of monthly egress costs more than 19 TB of storage. Any comparison that ignores this column is comparing the wrong numbers.

### Request pricing: where small files punish you

Every GET, PUT, LIST and lifecycle transition is metered. AWS charges $0.005 per 1,000 PUTs and $0.0004 per 1,000 GETs on Standard. Vultr lists a flat $0.01 per 1,000 for both reads and writes. R2 sits at $0.0045 writes and $0.00036 reads.

The classic mistake here: moving millions of small files to a cheaper storage class to save on storage. Take 1 TB of small files hit with 10 million GETs a month. On S3 Standard that's $23 storage plus $4 in requests. Move it to Standard-IA for the lower rate and the same workload costs $32.50 all-in, because IA adds higher per-op charges plus a $0.01/GB retrieval fee. The "cheaper" class costs more.

There's a genuinely useful break-even hiding in these rates. Vultr saves you $5 per TB on storage versus AWS but charges about $9.60 more per million reads. So below roughly half a million reads per stored TB per month, Vultr's cheaper storage wins; above that line, AWS's cheap GETs claw the difference back. If you run write-heavy ingest pipelines pushing millions of tiny objects, note that Vultr's write rate is 2× AWS's — usually small change, but worth a quick calculation first.

### Minimums, retention rules and metadata overhead

The small print is where per-GB rates earn their asterisks:

- **Standard-IA and One Zone-IA**: 30-day minimum storage duration and a 128 KB minimum billable object size. Delete or overwrite earlier and you're charged for the remaining days anyway.
- **Glacier tiers**: 90-day minimum for Instant Retrieval and Flexible Retrieval, 180 days for Deep Archive. Deep Archive restores take 12–48 hours.
- **Glacier metadata tax**: every object in Glacier Flexible or Deep Archive carries an extra 40 KB of billed metadata — 8 KB at Standard rates, 32 KB at Glacier rates. Harmless for large archives, real money for billions of tiny objects.
- **Intelligent-Tiering**: auto-tiering only applies to objects of 128 KB or more; smaller objects stay at Frequent Access rates. The archive tiers need to be enabled explicitly.
- **Outside AWS**: Wasabi enforces 90-day minimum retention, Contabo's SLA sits at 95%, Hetzner only operates in Germany and Finland, and the "free egress" at Backblaze and iDrive E2 caps out at 3× your stored data.

A September 2025 Crayon survey found 94% of IT leaders still struggling to optimize cloud costs, and metered storage with this kind of fine print is a typical reason why.

## Vultr Object Storage: what you actually pay

Vultr lists one pay-as-you-go product rather than tiered packages: $18/TB/month for storage, $10/TB for egress, $0.01 per 1,000 API requests, free uploads, with a 1 TB minimum. Locations cover the US, Netherlands and Singapore, the SLA is 99.9%, and there's no minimum commitment period.

| What you pay for | Listed price | Details | Buy |
| --- | --- | --- | --- |
| Object Storage | **$18/TB/month** | S3-compatible API, 1 TB minimum, US / Netherlands / Singapore, no long-term commitment | [see current Object Storage pricing](https://www.vultr.com/products/object-storage/?ref=7268283) |
| Data transfer out | $10/TB | Uploads (ingress) are free | — |
| API requests | $0.01 per 1,000 | Same rate for reads and writes | — |

Converted into realistic workloads (estimates from the listed rates):

| Workload | Storage | Egress | Requests | Rough monthly total |
| --- | --- | --- | --- | --- |
| 1 TB backup vault, restored once a month | $18 | $10 | under $1 | **~$29** |
| 5 TB app data, 1 TB out, 200K reads | $90 | $10 | $2 | **~$102** |
| 10 TB write-once archive, rarely touched | $180 | ~$0 | ~$0 | **~$180** |

The honest trade-offs: three regions instead of a global footprint, read operations at 25× AWS's rate, and no archive-class tier anywhere near Deep Archive's $0.99/TB. The honest strengths: the bill has exactly three line items, no retrieval fees, no tier-monitoring charges, no 30-day deletion penalties, and no lock-in — you can walk away after a month. For general app storage, backup vaults you actually restore from time to time, and media libraries with moderate delivery, that simplicity counts for something. 👉 [see Vultr's Object Storage plans and region options](https://www.vultr.com/products/object-storage/?ref=7268283).

## Which provider fits which job

Match the workload to the pricing model and the decision mostly makes itself:

- **Backups and archives, a few restores a year**: Wasabi at $6.99/TB with zero egress, or Backblaze B2 at $6/TB if restore volumes are unpredictable (the 3× free egress buffer absorbs spiky months). Watch Wasabi's 90-day retention clock if your backups churn.
- **Tightest possible list price**: iDrive E2 at $4.95/TB, or MEGA S4 at $2.63/TB if 3 TB minimum and its regions suit you.
- **Public downloads and static assets**: Cloudflare R2. Zero egress beats every storage discount once traffic grows — the $15/TB storage rate is secondary to the $0 transfer column.
- **Millions of small, hot objects**: AWS Standard's $0.0004/1K GETs are hard to beat. Run the request math before chasing a lower storage rate.
- **Compliance archives read once a decade**: AWS Glacier Deep Archive at $0.99/TB, 96% below Standard. Nothing in the budget tier matches it.
- **A general-purpose bucket with moderate reads, US/EU/Asia coverage and one simple bill**: Vultr at $18/TB plus $10/TB egress fits that middle ground, especially if you're already running workloads on Vultr. 👉 [open Vultr's Object Storage pricing page](https://www.vultr.com/products/object-storage/?ref=7268283) and compare it against your current bill.

## Frequently asked questions

**What does 1 TB of S3-compatible storage cost per month?**
Between $2.63 and $23 on list price for standard-tier storage, before egress and request fees. Within AWS itself the spread runs from $0.023/GB on Standard down to $0.00099/GB on Deep Archive — a 23× difference between classes of the same product.

**Why do Wasabi and Cloudflare R2 list $0 egress — what's the catch?**
Wasabi's trade-off is a 90-day minimum retention policy on stored objects. R2's zero-egress model is exactly as listed; it still charges per operation ($0.0045 per 1,000 writes, $0.00036 per 1,000 reads) and $15/TB for storage.

**Did AWS's free tier change recently?**
Yes. Accounts created after July 15, 2025 get $200 in credits valid for six months across all services, replacing the old 5 GB-for-12-months free tier. Accounts created before that date keep the original terms.

**When does more expensive storage end up cheaper?**
When objects are small and read often. A terabyte of files hit with 10 million monthly reads costs $27 all-in on S3 Standard versus $32.50 on Standard-IA, and around $118 on Vultr's rates. Past roughly 500,000 reads per stored TB per month, AWS's request pricing offsets its storage premium.

**How do I estimate my own bill before committing?**
Three multiplications: terabytes stored × storage rate, terabytes transferred out × egress rate, and expected operations × the per-1,000 rate. AWS publishes an official pricing calculator that handles its tiered structure; for the other providers, the table above contains everything the math needs.

## Before you sign up

Two quick checks. First, model your egress before your storage — it's the number most likely to dwarf everything else, and it's the one column where providers differ by infinity (literally $0 to $120/TB). Second, re-confirm rates on the provider's own pricing page the day you sign up; object storage pricing moves more often than compute pricing, and no comparison table stays current forever.

If your workload is read-light, everything in the budget tier — Wasabi, Backblaze, iDrive E2, Vultr — will run 20–80% below the hyperscalers, so pick by region coverage and egress terms rather than by the storage headline. For a middle-ground setup with straightforward pay-as-you-go billing, 👉 [start with 1 TB of Vultr Object Storage](https://www.vultr.com/products/object-storage/?ref=7268283) and scale from there.

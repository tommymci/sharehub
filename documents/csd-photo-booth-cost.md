---
title: "CSD Photo Booth — Cost per Poster and Event Budget"
date: 2026-09-23
---

# CSD Photo Booth — Cost per Poster and Event Budget

Every visitor poster is produced by two calls to Google's Gemini image API.
That API is effectively the whole cost of running the booth; everything else
rounds to nothing.

## Cost per poster

The quality setting is switchable at the booth, so this can be chosen on the
day — faster and cheaper while a queue is forming, higher quality when it is
quiet.

| Setting | Time per poster | HKD | USD |
|:---|---:|---:|---:|
| **Fast · 1K** — current default | ~22 sec | **$0.54** | $0.069 |
| Fast · 2K | ~25 sec | $0.80 | $0.103 |
| Best quality (Pro) | ~43 sec | $1.10 | $0.141 |

## Event budget

All figures in HKD. The final column is the one to budget against: it is the
default setting plus the 30% buffer explained below.

| Posters | Fast · 1K | Fast · 2K | Best quality | **Budget (1K + 30%)** |
|---:|---:|---:|---:|---:|
| 100 | $54 | $80 | $110 | **$70** |
| 500 | $270 | $400 | $550 | **$350** |
| 1,000 | $540 | $800 | $1,100 | **$700** |
| 5,000 | $2,700 | $4,000 | $5,500 | **$3,500** |
| 10,000 | $5,400 | $8,000 | $11,000 | **$7,000** |

A full recruitment day of roughly 500 posters should be budgeted at about
**HK$350**.

### Why the 30% buffer

Visitors retake photos, and a small share of renders are rejected by the check
that protects the CSD crest and wording. Each of those costs a full render
again, so the number of renders is always higher than the number of posters
handed out.

## Monthly API spend

The booth has no subscription and no minimum — the API is billed purely on
what is generated, so a month with no events costs nothing. These are monthly
totals at different levels of use, in HKD.

| Level of use | Posters / month | Fast · 1K | Fast · 2K | Best quality | **Budget (1K + 30%)** |
|:---|---:|---:|---:|---:|---:|
| Occasional — a few event days | 500 | $270 | $400 | $550 | **$350** |
| Regular — roughly weekly | 2,000 | $1,080 | $1,600 | $2,200 | **$1,400** |
| Busy — most weekdays | 5,000 | $2,690 | $4,020 | $5,500 | **$3,500** |
| Continuous — a booth running daily | 10,000 | $5,380 | $8,030 | $11,000 | **$7,000** |
| Peak campaign | 30,000 | $16,150 | $24,100 | $33,000 | **$21,000** |

For scale: 10,000 posters a month is roughly 330 a day, every day — more than
one staffed booth can physically serve at 22 seconds each. Realistic recurring
use for a recruitment campaign sits in the first two rows, so **HK$300 to
HK$1,400 a month** is the range to plan around.

Two things that do not change with volume: there is **no standing cost** in a
quiet month, and there are **no volume discounts** at these levels — the cost
is simply linear in posters produced.

## Where the money goes

| Item | Per poster | Share |
|:---|---:|---:|
| Gemini image API | $0.069 | ~97% |
| Cloud Run compute (~22 sec) | $0.0013 | ~2% |
| Storage and download traffic | under $0.0001 | under 1% |

Posters are deleted within 24 hours, so storage never accumulates. The service
scales to zero between events, so there is **no standing monthly cost** — an
idle month costs nothing at all.

## Which setting to use

**Fast · 1K is the right default.** It is both the cheapest option and the
quickest. Best quality triples the cost and doubles the wait for a difference
most visitors will not notice in a poster viewed on a phone.

| | Fast · 1K | Best quality |
|:---|---:|---:|
| Cost per poster | $0.54 | $1.10 |
| Wait per visitor | ~22 sec | ~43 sec |
| Posters per hour, one booth | ~160 | ~84 |

The throughput figure matters as much as the price: at a busy stand the slower
setting nearly halves how many people can be served.

## Basis for these figures

Calculated from Google's published Gemini API pricing, September 2026 — image
output at $0.067 (1K) and $0.101 (2K) for the Fast model and $0.134 for Pro,
plus input charges of roughly $0.002 and $0.007 respectively. Converted at
HK$7.8 to the US dollar.

These are **calculated figures, not invoiced amounts.** Treat them as a
planning estimate and confirm against an actual bill after the first event.
Google may also revise its API pricing.

---
title: "CSD Photo Booth — Cost per Poster and Event Budget"
date: 2026-09-23
---

# CSD Photo Booth — Cost per Poster and Event Budget

Every visitor poster is produced by two calls to Google's Gemini image API.
That API is effectively the whole cost of running the booth; everything else
rounds to nothing.

Two models are available, and the booth can switch between them on the day:

| Model | API name | Role |
|:---|:---|:---|
| **Gemini 3.1 Flash Image** | `gemini-3.1-flash-image` | The default — faster and cheaper |
| Gemini 3 Pro Image | `gemini-3-pro-image` | Higher quality, slower and dearer |

Output resolution is a separate choice: **1K** or **2K**. It affects price on
Flash but not on Pro, which Google charges the same for either.

## Cost per poster

| Model and resolution | Time per poster | HKD | USD |
|:---|---:|---:|---:|
| **Flash 3.1 · 1K** — current default | ~22 sec | **$0.54** | $0.069 |
| Flash 3.1 · 2K | ~25 sec | $0.80 | $0.103 |
| Pro 3 · 1K or 2K | ~43 sec | $1.10 | $0.141 |

## Event budget

All figures in HKD. The final column is the one to budget against: the default
model plus the 30% buffer explained below.

| Posters | Flash 3.1 · 1K | Flash 3.1 · 2K | Pro 3 | **Budget (Flash 1K + 30%)** |
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

The booth has no subscription and no minimum — the API is billed purely on what
is generated, so a month with no events costs nothing. Monthly totals in HKD:

| Level of use | Posters / month | Flash 3.1 · 1K | Flash 3.1 · 2K | Pro 3 | **Budget (Flash 1K + 30%)** |
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
quiet month, and there are **no volume discounts** at these levels — cost is
simply linear in posters produced.

## Where the money goes

| Item | Per poster | Share |
|:---|---:|---:|
| Gemini image API (Flash 3.1 · 1K) | $0.069 | ~97% |
| Cloud Run compute (~22 sec) | $0.0013 | ~2% |
| Storage and download traffic | under $0.0001 | under 1% |

Posters are deleted within 24 hours, so storage never accumulates. The service
scales to zero between events, so there is **no standing monthly cost** — an
idle month costs nothing at all.

## Which model to use

**Flash 3.1 at 1K is the right default.** It is both the cheapest option and
the quickest. Pro 3 doubles the cost and the wait for a difference most
visitors will not notice in a poster viewed on a phone.

| | Flash 3.1 · 1K | Pro 3 |
|:---|---:|---:|
| Cost per poster | $0.54 | $1.10 |
| Wait per visitor | ~22 sec | ~43 sec |
| Posters per hour, one booth | ~160 | ~84 |

Throughput matters as much as price here: at a busy stand the slower model
nearly halves how many people can be served.

Pro 3 is the stronger model at rendering text, but that advantage does not
apply to this booth. The poster's crest and wording are never taken from the
model's output — they are preserved from the original artwork — so the only
thing Pro is being paid for is the figure itself.

## Basis for these figures

Calculated from Google's published Gemini API pricing, September 2026:

| Model | Image output | Input |
|:---|:---|:---|
| `gemini-3.1-flash-image` | $0.067 per 1K image, $0.101 per 2K | $0.50 per million tokens |
| `gemini-3-pro-image` | $0.134 per 1K or 2K image | $2.00 per million tokens |

Input charges add roughly $0.002 per poster on Flash and $0.007 on Pro, across
both API calls. Converted at HK$7.8 to the US dollar.

These are **calculated figures, not invoiced amounts.** Treat them as a
planning estimate and confirm against an actual bill after the first event.
Google may also revise its API pricing.

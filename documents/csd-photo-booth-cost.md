---
title: "CSD Photo Booth — Cost per Poster and Event Budget"
date: 2026-09-23
---

<style>
/* Scoped to this page. ShareHub's layout ships no table styling, so tables
   render with browser defaults and the columns collide. Colours are written
   against currentColor so this stays legible on the light theme and in a
   dark-mode browser alike. */
.doc-body { max-width: 62rem; }
.doc-body table {
  width: 100%;
  border-collapse: collapse;
  margin: 1.4rem 0 1.8rem;
  font-variant-numeric: tabular-nums;
  font-size: 0.97rem;
}
.doc-body thead th {
  text-align: left;
  font-weight: 700;
  padding: 0.7rem 1rem;
  border-bottom: 2px solid rgba(128, 128, 128, 0.45);
  white-space: nowrap;
}
.doc-body tbody td {
  padding: 0.62rem 1rem;
  border-bottom: 1px solid rgba(128, 128, 128, 0.22);
  vertical-align: top;
}
.doc-body tbody tr:nth-child(odd) { background: rgba(128, 128, 128, 0.07); }
.doc-body th[align="right"], .doc-body td[align="right"] { text-align: right; }
.doc-body tbody tr td:last-child { font-weight: 600; }
/* A wide table should scroll rather than crush its columns on a phone. */
.doc-scroll { overflow-x: auto; -webkit-overflow-scrolling: touch; }

.doc-key {
  display: flex; flex-wrap: wrap; gap: 1.6rem;
  padding: 1.2rem 1.4rem; margin: 1.6rem 0 2rem;
  border: 1px solid rgba(128, 128, 128, 0.3);
  border-left: 4px solid #667eea;
  border-radius: 8px;
  background: rgba(128, 128, 128, 0.06);
}
.doc-key div { min-width: 9rem; }
.doc-key b { display: block; font-size: 1.5rem; line-height: 1.1; }
.doc-key span { font-size: 0.85rem; opacity: 0.75; }
.doc-body h2 {
  margin-top: 2.4rem; padding-top: 1.1rem;
  border-top: 1px solid rgba(128, 128, 128, 0.25);
}
.doc-note {
  padding: 0.9rem 1.2rem; margin: 1.5rem 0;
  border: 1px solid rgba(128, 128, 128, 0.3); border-radius: 8px;
  background: rgba(128, 128, 128, 0.06); font-size: 0.93rem;
}
</style>

<div class="doc-body" markdown="1">

# CSD Photo Booth — Cost per Poster and Event Budget

Every visitor poster is produced by two calls to Google's Gemini image API.
That API is effectively the whole cost of running the booth; everything else
rounds to nothing.

<div class="doc-key">
  <div><b>HK$0.54</b><span>per poster, default setting</span></div>
  <div><b>HK$350</b><span>a 500-poster event day</span></div>
  <div><b>HK$0</b><span>a month with no events</span></div>
  <div><b>~160</b><span>posters per hour, one booth</span></div>
</div>

Currency is marked on every figure: **HK$** for Hong Kong dollars, **US$** for
United States dollars. Google bills in US dollars; HK$ figures are converted at
**7.8**.

## The two models

<div class="doc-scroll" markdown="1">

| Model | API name | Role |
|:---|:---|:---|
| **Gemini 3.1 Flash Image** | `gemini-3.1-flash-image` | The default — faster and cheaper |
| Gemini 3 Pro Image | `gemini-3-pro-image` | Higher quality, slower and dearer |

</div>

Output resolution is a separate choice, **1K** or **2K**. It changes the price
on Flash but not on Pro, which Google charges the same for either.

## Cost per poster

<div class="doc-scroll" markdown="1">

| Model and resolution | Time per poster | Cost (HK$) | Cost (US$) |
|:---|---:|---:|---:|
| **Flash 3.1 · 1K** — current default | ~22 sec | **HK$0.54** | US$0.069 |
| Flash 3.1 · 2K | ~25 sec | HK$0.80 | US$0.103 |
| Pro 3 · 1K or 2K | ~43 sec | HK$1.10 | US$0.141 |

</div>

The setting is switchable at the booth, so it can be chosen on the day —
cheaper and faster while a queue is forming, higher quality when it is quiet.

## Event budget

<div class="doc-scroll" markdown="1">

| Posters | Flash 3.1 · 1K | Flash 3.1 · 2K | Pro 3 | Budget (Flash 1K + 30%) |
|---:|---:|---:|---:|---:|
| 100 | HK$54 | HK$80 | HK$110 | **HK$70** |
| 500 | HK$270 | HK$400 | HK$550 | **HK$350** |
| 1,000 | HK$540 | HK$800 | HK$1,100 | **HK$700** |
| 5,000 | HK$2,700 | HK$4,000 | HK$5,500 | **HK$3,500** |
| 10,000 | HK$5,400 | HK$8,000 | HK$11,000 | **HK$7,000** |

</div>

The last column is the one to budget against. **A full recruitment day of
roughly 500 posters should be budgeted at about HK$350.**

<div class="doc-note" markdown="1">
**Why the 30% buffer.** Visitors retake photos, and a small share of renders
are rejected by the check that protects the CSD crest and wording. Each of
those costs a full render again, so renders always exceed posters handed out.
</div>

## Monthly API spend

There is no subscription and no minimum. The API is billed purely on what is
generated, so a month with no events costs nothing.

<div class="doc-scroll" markdown="1">

| Level of use | Posters / month | Flash 3.1 · 1K | Flash 3.1 · 2K | Pro 3 | Budget (Flash 1K + 30%) |
|:---|---:|---:|---:|---:|---:|
| Occasional — a few event days | 500 | HK$270 | HK$400 | HK$550 | **HK$350** |
| Regular — roughly weekly | 2,000 | HK$1,080 | HK$1,600 | HK$2,200 | **HK$1,400** |
| Busy — most weekdays | 5,000 | HK$2,690 | HK$4,020 | HK$5,500 | **HK$3,500** |
| Continuous — daily booth | 10,000 | HK$5,380 | HK$8,030 | HK$11,000 | **HK$7,000** |
| Peak campaign | 30,000 | HK$16,150 | HK$24,100 | HK$33,000 | **HK$21,000** |

</div>

For scale, 10,000 posters a month is roughly 330 a day, every day — more than
one staffed booth can physically serve at 22 seconds each, so those rows imply
multiple booths. Realistic recurring use for a recruitment campaign sits in the
first two rows, making **HK$300 to HK$1,400 a month** the range to plan around.

Two things that do not change with volume: there is **no standing cost** in a
quiet month, and there are **no volume discounts** at these levels — cost is
simply linear in posters produced.

## Where the money goes

<div class="doc-scroll" markdown="1">

| Item | Per poster (US$) | Share of cost |
|:---|---:|---:|
| Gemini image API (Flash 3.1 · 1K) | US$0.069 | ~97% |
| Cloud Run compute (~22 sec) | US$0.0013 | ~2% |
| Storage and download traffic | under US$0.0001 | under 1% |

</div>

Posters are deleted within 24 hours, so storage never accumulates, and the
service scales to zero between events.

## Which model to use

**Flash 3.1 at 1K is the right default** — both the cheapest option and the
quickest.

<div class="doc-scroll" markdown="1">

| | Flash 3.1 · 1K | Pro 3 |
|:---|---:|---:|
| Cost per poster | HK$0.54 | HK$1.10 |
| Wait per visitor | ~22 sec | ~43 sec |
| Posters per hour, one booth | ~160 | ~84 |

</div>

Throughput matters as much as price: at a busy stand the slower model nearly
halves how many people can be served.

Pro 3 is the stronger model at rendering text, but that advantage does not
apply here. The poster's crest and wording are never taken from the model's
output — they are preserved from the original artwork — so the only thing Pro
is being paid for is the figure itself.

## Basis for these figures

Calculated from Google's published Gemini API pricing, September 2026, which
Google publishes and bills in US dollars:

<div class="doc-scroll" markdown="1">

| Model | Image output | Input |
|:---|:---|:---|
| `gemini-3.1-flash-image` | US$0.067 per 1K image, US$0.101 per 2K | US$0.50 per million tokens |
| `gemini-3-pro-image` | US$0.134 per 1K or 2K image | US$2.00 per million tokens |

</div>

Input charges add roughly US$0.002 per poster on Flash and US$0.007 on Pro,
across both API calls. Hong Kong dollar figures are converted at HK$7.8 to
US$1, so they move with the exchange rate.

<div class="doc-note" markdown="1">
These are **calculated figures, not invoiced amounts.** Treat them as a
planning estimate and confirm against an actual bill after the first event.
Google may also revise its API pricing.
</div>

</div>

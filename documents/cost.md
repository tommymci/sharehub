---
title: "CSD Photo Booth — Running Cost"
date: 2026-09-23
---

# CSD Photo Booth — Running Cost

Each visitor poster is produced by two calls to Google's Gemini image API. That
API is essentially the entire cost of the service; everything else rounds to
nothing.

## Cost per poster

| Quality setting | Per poster (USD) | Per poster (HKD) |
|---|---|---|
| **Fast · 1K — current default** | **$0.069** | **$0.54** |
| Fast · 2K | $0.103 | $0.80 |
| Best quality (Pro) | $0.141 | $1.10 |

The three settings are switchable at the booth, so the figure can be chosen on
the day: faster and cheaper when there is a queue, higher quality when there
is not.

## Cost at scale

| Posters | Fast · 1K | Fast · 2K | Best quality |
|---|---|---|---|
| 100 | $7 / HK$54 | $10 / HK$80 | $14 / HK$110 |
| 500 | $35 / HK$269 | $52 / HK$402 | $71 / HK$550 |
| 1,000 | $69 / HK$538 | $103 / HK$803 | $141 / HK$1,100 |
| 5,000 | $345 / HK$2,691 | $515 / HK$4,017 | $705 / HK$5,499 |
| 10,000 | $690 / HK$5,382 | $1,030 / HK$8,034 | $1,410 / HK$10,998 |

A full recruitment day of roughly 500 posters costs about **HK$270** on the
default setting.

## What to budget

Add about **30%** to whatever poster count is expected. Visitors retake photos,
and a small number of renders are rejected by the safety check that protects
the CSD crest and wording — each of those costs a full render again. So 500
expected posters is better budgeted as 650, or roughly **HK$350**.

## Why the other costs do not matter

| Item | Cost per poster |
|---|---|
| Gemini image API | ~$0.069 |
| Cloud Run compute (~25 seconds) | ~$0.0013 |
| Storage and download traffic | under $0.0001 |

Posters are deleted within 24 hours, so storage never accumulates. The service
scales to zero between events, so there is **no standing monthly cost** — an
idle month costs nothing.

## Choosing a setting

**Fast · 1K is the right default.** It is the cheapest option and also the
quickest at roughly 22 seconds per poster. Moving to Best quality triples the
cost and nearly doubles the wait, for a difference most visitors will not see
in a poster viewed on a phone.

## Basis for these figures

Calculated from Google's published Gemini API pricing as of September 2026:
image output at $0.067 (1K) and $0.101 (2K) for the Fast model and $0.134 for
Pro, plus input charges of roughly $0.002 and $0.007 respectively. HKD shown at
7.8 to the US dollar.

These are **calculated figures, not invoiced amounts.** They should be treated
as a planning estimate and confirmed against an actual bill after the first
event. Google may also revise API pricing.

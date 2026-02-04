# Google POC Credit Application - Shave for Hope

---

## 1. Customer Use Case / Business Justification

> *Please lay out the customer use case and how Google will be solving the customer's problem through this POC.*

**Customer:** Children's Cancer Foundation (CCF) Hong Kong - a registered charity supporting children with cancer and their families since 1989.

**Use Case:** "Shave for Hope" (剃亮希望) is an annual fundraising campaign where supporters symbolically shave their heads in solidarity with children undergoing cancer treatment.

**Problem Being Solved:**
- **Engagement barrier:** Traditional fundraising requires physical commitment (shaving head), limiting participation
- **Viral potential untapped:** Current campaign lacks shareable digital content for social media amplification
- **Personalization gap:** Supporters want to visualize themselves before committing to shave

**Google's Solution:**
Using Google Cloud's Generative AI capabilities, we're building a web application that:
1. **AI Photo Transformation (Gemini 2.5 Flash):** Users upload a photo and AI transforms it to show them with a shaved head, letting them "preview" their support
2. **AI Video Generation (Veo 3.1):** Transformed photos become personalized 5-second animated videos for maximum social media impact
3. **Viral Fundraising Loop:** Users share their AI-generated content with personal fundraising pages, driving donations

This lowers the barrier to participation while increasing viral reach - supporters can share their "shaved" image/video without physical commitment first, then convert to actual head-shaving participants.

---

## 2. POC Timeline

> *Provide a proposed timeframe for the Proof of Concept (POC), including a start date and an end date. Timeline refers to how long the credit will last. Credits cannot be pre-approved or start in the future. The maximum duration for POC credit is 3 months with a max of two extensions. Ex: November 4 - December 27, 2024. It is acceptable to state, "Start date upon issuance of credit". End date or duration of the credit (in months), is required.*

**Start Date:** Upon issuance of credit (Expected: February 3, 2026)

**End Date:** April 30, 2026 (approximately 3 months)

**Key Milestones:**

| Date | Milestone |
|------|-----------|
| Feb 3 | GCP credit activated, Video generation development begins |
| Feb 5 | Video generation feature complete |
| Mar 1 | Official public launch |
| Mar 14 | Head Shaving Day event (Central Market, Hong Kong) |
| Apr 30 | POC evaluation complete |

---

## 3. Deliverables

> *Please document all the deliverables to be completed as part of this POC. At a minimum, there should be 2 deliverables that you will pass off to the customer that will prove that the POC worked.*

**Deliverable 1: AI Photo Transformation System**
- Web application allowing users to upload photos
- Gemini 2.5 Flash transforms photos to show shaved head appearance
- Automatic frame overlay with campaign branding
- Shareable output with tracking for fundraising attribution
- **Proof:** Live production system at shaveforhope.ccf.org.hk with usage analytics

**Deliverable 2: AI Video Generation Pipeline**
- Integration of Veo 3.1 for video generation
- Transformed photos converted to 5-second animated videos
- Optimized for social media sharing (Instagram, Facebook, WhatsApp)
- Entitlement system (donation-gated access to video feature)
- **Proof:** Video generation logs, sample outputs, and conversion metrics

**Deliverable 3: Campaign Analytics Dashboard**
- Real-time metrics: transformations, shares, donations
- AI usage tracking and cost analysis
- ROI measurement for Google Cloud services
- **Proof:** Analytics report comparing AI-driven vs traditional campaign performance

---

## 4. Success Criteria

> *Please explain what success will look like after this POC is complete. What are the expected outcomes that will convince the customer to close their deal with Google? Expected outcomes should be measurable, (%, $ increases, # increase with volume of traffic, expected % of increased performance, etc).*

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Photo Transformations** | 10,000+ | Number of AI transformations completed |
| **Video Generations** | 2,000+ | Number of Veo-generated videos |
| **Social Shares** | 5,000+ | Tracked shares via platform analytics |
| **Donation Conversion** | 15%+ | % of users who donate after transformation |
| **Total Fundraising** | HK$500,000+ (~US$65,000) | Direct donations through platform |
| **New Donors Acquired** | 1,000+ | First-time donors to CCF |
| **AI Cost Efficiency** | <HK$1/transformation | Cost per AI operation |
| **User Satisfaction** | 4.5+/5 | Post-transformation survey rating |

**Expected Outcome:** Demonstrate that AI-powered engagement increases fundraising participation by 3-5x compared to traditional campaign methods, proving the value of Google Cloud AI for nonprofit digital transformation.

---

## 5. 15X Return on Investment

> *How will Google get 15X ROI with this POC during the 12 months following completion. This number should align with the Revenue Expected figure provided during submission. Examples: Will the customer sign a commit worth $X? Will the customer move additional workloads upon a successful POC and what is the value of them?*

**POC Credit Requested:** US$5,000 (estimated)

**Expected 15X ROI Path (US$75,000+ in 12 months):**

### Immediate Value (Campaign Period - Mar 2026)
- Campaign fundraising target: HK$500,000+ (~US$65,000)
- Platform demonstrates Google Cloud AI capabilities to 10,000+ users
- Case study potential for Google Cloud nonprofit marketing

### Post-POC Expansion (12 months following)

1. **CCF Annual Adoption:** CCF plans to use this platform for future annual campaigns (2027, 2028), representing ongoing GCP consumption of ~US$3,000-5,000/year

2. **Template for Other Nonprofits:** Success case can be replicated for other Hong Kong charities (est. 5-10 organizations), each representing US$2,000-5,000 annual GCP spend

3. **HKMCI (Development Partner) Commitment:**
   - HKMCI (development partner) will recommend Google Cloud for future nonprofit projects
   - Pipeline of 3-5 similar projects annually at ~US$5,000-10,000 each

4. **Enterprise Expansion:**
   - HKMCI's corporate clients exposed to Gemini/Veo capabilities
   - Potential enterprise AI adoption from demonstration effect

### Conservative 12-Month Revenue Projection

| Source | Value |
|--------|-------|
| CCF ongoing usage | US$5,000 |
| 3 nonprofit replications | US$15,000 |
| HKMCI project pipeline | US$30,000 |
| Enterprise referrals | US$25,000+ |
| **Total** | **US$75,000+** |

---

## 6. Products Included in the POC

> *Compile a list of the products that will be tested during the POC. This should align to what was submitted on the intake form (go/sales-credit). NOTE: If products leverage TPU/GPU SKU Groups, please send a comment to cloud-credit-program@google.com for your submission to be updated.*

| Product | Purpose | Usage |
|---------|---------|-------|
| **Gemini 2.5 Flash** | Photo-to-shaved-head AI transformation | ~10,000+ API calls |
| **Veo 3.1** | Video generation from transformed images | ~2,000+ video generations |
| **Firebase** | Authentication, Firestore database, Storage, Hosting | Full application backend |
| **Cloud Storage** | Store original/transformed images and videos | ~50GB estimated |
| **Cloud Run** (if needed) | Serverless compute for AI flow orchestration | On-demand |
| **Vertex AI** | Model serving and management | API access |

**Note:** Video generation (Veo 3.1) requires GPU/TPU resources. Please update submission accordingly.

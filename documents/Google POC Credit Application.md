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

### AI Usage Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Photo Transformations** | 3,000+ | Number of AI transformations completed |
| **Video Generations** | 500+ | Number of Veo-generated videos (donation-gated) |
| **Social Shares** | 2,000+ | Tracked shares via platform analytics |
| **AI Cost Efficiency** | <HK$2/transformation | Cost per AI operation |
| **User Satisfaction** | 4.0+/5 | Post-transformation survey rating |

### Fundraising & Donation Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Total Fundraising** | HK$300,000+ (~US$38,000) | All donations through platform |
| **Online Donations** | HK$150,000+ | Direct online payment (AlipayHK) |
| **Offline/Bank Transfers** | HK$150,000+ | Bank transfers & cheques |
| **Average Donation** | HK$200+ | Per donor average |
| **New Donors Acquired** | 500+ | First-time donors to CCF |
| **Donation Conversion Rate** | 10%+ | % of users who donate after transformation |
| **Recurring Donors** | 50+ | Users who donate multiple times |

### Donation Flow Details
1. **Free Tier:** Users can transform photos for free (quota-limited)
2. **Video Unlock:** Donation of any amount unlocks video generation feature
3. **Fundraiser Model:** Users create personal fundraising pages, share with friends/family
4. **Payment Methods:** AlipayHK (primary), bank transfer, cheque
5. **Attribution Tracking:** All donations linked to referrer for leaderboard

**Expected Outcome:** Demonstrate that AI-powered engagement increases fundraising participation by 2-3x compared to traditional campaign methods, proving the value of Google Cloud AI for nonprofit digital transformation.

---

## 5. 15X Return on Investment

> *How will Google get 15X ROI with this POC during the 12 months following completion. This number should align with the Revenue Expected figure provided during submission. Examples: Will the customer sign a commit worth $X? Will the customer move additional workloads upon a successful POC and what is the value of them?*

**POC Credit Requested:** US$3,000 (estimated)

**Expected 15X ROI Path (US$45,000+ in 12 months):**

### Immediate Value (Campaign Period - Mar 2026)
- Campaign fundraising target: HK$300,000+ (~US$38,000)
- Platform demonstrates Google Cloud AI capabilities to 3,000+ users
- Case study potential for Google Cloud nonprofit marketing

### Post-POC Expansion (12 months following)

1. **CCF Annual Adoption:** CCF plans to use this platform for future annual campaigns (2027, 2028), representing ongoing GCP consumption of ~US$2,000-3,000/year

2. **Template for Other Nonprofits:** Success case can be replicated for other Hong Kong charities (est. 3-5 organizations), each representing US$1,500-3,000 annual GCP spend

3. **Master Concept (Development Partner) Commitment:**
   - Master Concept (development partner) will recommend Google Cloud for future nonprofit projects
   - Pipeline of 2-3 similar projects annually at ~US$3,000-5,000 each

4. **Enterprise Expansion:**
   - Master Concept's corporate clients exposed to Gemini/Veo capabilities
   - Potential enterprise AI adoption from demonstration effect

### Conservative 12-Month Revenue Projection

| Source | Value |
|--------|-------|
| CCF ongoing usage | US$3,000 |
| 3 nonprofit replications | US$9,000 |
| Master Concept project pipeline | US$15,000 |
| Enterprise referrals | US$18,000+ |
| **Total** | **US$45,000+** |

---

## 6. Products Included in the POC

> *Compile a list of the products that will be tested during the POC. This should align to what was submitted on the intake form (go/sales-credit). NOTE: If products leverage TPU/GPU SKU Groups, please send a comment to cloud-credit-program@google.com for your submission to be updated.*

| Product | Purpose | Usage |
|---------|---------|-------|
| **Gemini 2.5 Flash** | Photo-to-shaved-head AI transformation | ~3,000+ API calls |
| **Veo 3.1** | Video generation from transformed images | ~500+ video generations |
| **Firebase** | Authentication, Firestore database, Storage, Hosting | Full application backend |
| **Cloud Storage** | Store original/transformed images and videos | ~20GB estimated |
| **Cloud Run** (if needed) | Serverless compute for AI flow orchestration | On-demand |
| **Vertex AI** | Model serving and management | API access |

**Note:** Video generation (Veo 3.1) requires GPU/TPU resources. Please update submission accordingly.

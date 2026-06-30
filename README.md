# OrthoNow — Digital Marketing & Analytics Implementation

A production-ready technical assignment for **OrthoNow**, a chain of 9 orthopaedic clinics across Bengaluru, Hyderabad, and Chennai. This project delivers a complete GTM event schema, a high-converting landing page, and an integration architecture design.

---

## 📋 Project Overview

OrthoNow's current digital infrastructure has significant gaps:
- 5-year-old WordPress website with no GTM implementation
- GA4 tracking limited to page views only
- No CRM integration
- Landing page conversion rate of **2.1%** (vs. 6–8% industry benchmark)

This assignment addresses all three areas with production-ready deliverables:

| Deliverable | Description |
|---|---|
| **Task 1 — GTM Event Schema** | Complete event tracking plan with 20 events, booking funnel deep-dive, and Google Ads conversion strategy |
| **Task 2 — Landing Page** | CRO-optimized single-file landing page targeting Bengaluru professionals with back/knee pain |
| **Task 3 — Integration Design** | End-to-end architecture: Landing Page → HubSpot CRM → Karix WhatsApp API → Google Ads |

---

## 🛠 Technologies Used

| Technology | Usage |
|---|---|
| **HTML5** | Semantic markup for the landing page |
| **CSS3** | Custom properties, Grid, Flexbox, animations — no frameworks |
| **Vanilla JavaScript** | Form validation, dataLayer integration, state management |
| **Google Tag Manager** | Event schema design and trigger configuration |
| **Google Analytics 4** | Event tracking, funnel exploration, audience building |
| **Google Ads** | Conversion import strategy |
| **HubSpot CRM** | Contacts API v3 integration design |
| **Karix API** | WhatsApp Business messaging integration design |
| **Markdown** | Documentation and technical specifications |

---

## 📁 Folder Structure

```
Namoza Developer Assignment/
│
├── README.md                          ← You are here
│
├── task-1/                            ← GTM & Analytics
│   ├── gtm-schema.md                 ← Complete event schema (20 events)
│   └── booking-funnel.md             ← Deep-dive: triggers, dataLayer, GA4 funnel
│
├── task-2/                            ← Landing Page
│   ├── index.html                    ← Production-ready single-file landing page
│   └── pagespeed-notes.md            ← Performance optimizations & CRO rationale
│
├── task-3/                            ← Integration Architecture
│   └── integration-design.md         ← HubSpot + Karix + Google Ads flow
│
├── assets/                            ← Static assets (currently empty — SVGs are inline)
│   └── README.md                     ← Asset conventions documentation
│
└── loom-walkthrough-script.md         ← 8-minute video walkthrough script
```

---

## 📝 Implementation Notes

### Task 1 — GTM Event Schema
- **20 events** covering all specified interactions: booking funnel (6 events), call buttons (3 events), WhatsApp widget (1 event), patient guide download (2 events), clinic pages (2 events), blog scroll depth (4 events), and additional recommended events (2 events)
- All events follow `snake_case` naming convention aligned with GA4 standards
- Booking funnel documentation includes **real JSON** (not pseudocode) for all 6 dataLayer pushes
- Frontend developer implementation guide written as a handoff document
- Google Ads conversion strategy selects `booking_completed` as the sole primary conversion with detailed reasoning

### Task 2 — Landing Page
- **Single HTML file** — zero external dependencies (no Bootstrap, Tailwind, or CDN)
- **2-field form** (Name + Phone) with Indian phone number validation (`/^[6-9]\d{9}$/`)
- **`window.dataLayer.push()`** fires exactly on successful form submission (not page load)
- **Thank You state** renders without page reload
- **Duplicate submission prevention** via `isSubmitting` and `hasSubmitted` flags
- **Mobile responsive** — tested against 320px to 1440px viewports
- **Target PageSpeed Mobile: 90+** — achieved through zero images, inline resources, and minimal DOM

### Task 3 — Integration Design
- **Chosen method:** HubSpot Contacts API v3 (not Forms API, not Zapier)
- **Phone-based deduplication** using HubSpot's search API before create/update
- **WhatsApp SLA:** 2-minute delivery via async queue with exponential backoff retries
- **Dead-letter queue** and SMS fallback for failed WhatsApp deliveries
- **Monitoring:** Centralized logging, failure rate alerting, p95 latency tracking

---

## 🚀 Running Instructions

### Landing Page (Task 2)

**Option 1 — Direct file open:**
```bash
# Simply open the HTML file in any browser
# Windows
start task-2/index.html

# macOS
open task-2/index.html

# Linux
xdg-open task-2/index.html
```

**Option 2 — Local server (recommended for testing):**
```bash
# Using Python
cd task-2
python -m http.server 8080
# Open http://localhost:8080

# Using Node.js
npx serve task-2
# Open the URL shown in terminal
```

**Option 3 — VS Code Live Server:**
1. Install the "Live Server" extension in VS Code
2. Right-click `task-2/index.html`
3. Select "Open with Live Server"

---

## 🧪 Testing Instructions

### Landing Page Functional Tests

| Test Case | Steps | Expected Result |
|---|---|---|
| **Empty form submission** | Click "Book My Free Consultation" without entering anything | Both fields show error messages. Button is not disabled. |
| **Invalid phone number** | Enter "1234567890" (starts with 1) | Phone error: "Enter a valid 10-digit Indian mobile number" |
| **Short phone number** | Enter "98765" (only 5 digits) | Phone error appears on blur |
| **Valid submission** | Enter "Ravi Kumar" + "9876543210" | Loading spinner → Thank You state appears. No page reload. |
| **Duplicate prevention** | Submit form, then try to submit again | Second submission is prevented (hasSubmitted flag) |
| **dataLayer push** | Open browser console before submitting | After submission, `window.dataLayer` contains `consultation_form_submitted` event |
| **Mobile responsive** | Resize browser to 375px width | Form stacks below hero content. All text readable. |
| **WhatsApp widget** | Click green floating button | Opens WhatsApp with pre-filled message |
| **Call button** | Click header phone number | Triggers `tel:` link |

### Verifying dataLayer Push
```javascript
// Open browser console (F12) and run:
console.log(window.dataLayer);

// After form submission, the last entry should be:
// { event: "consultation_form_submitted", form_name: "consultation_landing_page", ... }
```

### GTM Preview/Debug Mode
1. Open GTM → Preview mode
2. Navigate to the landing page URL
3. Submit the form
4. In GTM debug panel, verify `consultation_form_submitted` event appears
5. Check all parameters are populated (not `undefined`)

---

## 📸 Screenshots Section

> Screenshots should be captured after deploying the landing page:
>
> 1. **Desktop Hero** — Full hero section with form visible above the fold
> 2. **Mobile Hero** — Mobile view showing headline → form stack
> 3. **Form Validation** — Error states on both fields
> 4. **Thank You State** — Post-submission confirmation view
> 5. **Trust Section** — Doctor credentials and patient reviews
> 6. **GTM Debug Panel** — dataLayer event firing in GTM Preview mode
> 7. **PageSpeed Score** — Lighthouse mobile score screenshot

---

## 🔮 Future Improvements

1. **A/B Testing Framework** — Implement Google Optimize (or VWO) to test:
   - Headline variations ("Don't let pain hold you back" vs. "Get back to living pain-free")
   - CTA copy ("Book My Free Consultation" vs. "Talk to a Specialist Now")
   - Form position (right-side card vs. centered modal)

2. **Progressive Form Enhancement** — Add an optional "Select Condition" dropdown (Back Pain / Knee Pain / Other) as a third field to improve lead qualification without significantly impacting conversion rate.

3. **Clinic Selector** — Add a "Preferred Clinic" dropdown populated from the 9 clinic locations, enabling direct routing to the correct clinic team in HubSpot.

4. **Multi-Language Support** — Add Kannada, Telugu, and Tamil language toggles for Bengaluru, Hyderabad, and Chennai audiences respectively.

5. **Real-Time Slot Availability** — Integrate with the clinic's scheduling system to show available time slots on the landing page, converting the 2-step process (submit form → wait for call) into a 1-step process (book a specific slot).

6. **Enhanced GA4 Tracking** — Add `form_field_interaction` events for field-level analytics, scroll depth tracking on the landing page, and time-on-page metrics.

7. **Server-Side Tagging** — Migrate from client-side GTM to server-side GTM for improved data accuracy, reduced page weight, and better privacy compliance.

8. **Offline Conversion Import** — Upload CRM closed-deal data back to Google Ads to optimize Smart Bidding for actual patients (not just form leads).

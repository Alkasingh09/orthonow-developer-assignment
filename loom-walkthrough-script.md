# Loom Walkthrough Script — 8 Minutes

## Pre-Recording Setup
- Open the landing page in Chrome (full-screen)
- Open GTM Preview mode in a second tab
- Open the browser console (F12) in a third tab
- Have the markdown files open in VS Code
- Timer: Use a phone timer set to track 2:00, 5:00, 8:00 marks

---

## PART 1 — GTM Event Schema (0:00 – 2:00)

### [0:00 – 0:20] Introduction

> "Hi, I'm [Name], and this is my walkthrough of the OrthoNow technical assignment. OrthoNow operates 9 orthopaedic clinics across Bengaluru, Hyderabad, and Chennai. Their current setup has a 5-year-old WordPress site, no GTM, GA4 tracking only page views, no CRM integration, and a landing page converting at just 2.1% — well below the 6 to 8 percent industry benchmark. Let me walk you through how I'm addressing each of these gaps."

### [0:20 – 0:50] Event Schema Overview

*[Show gtm-schema.md in VS Code]*

> "Starting with Task 1 — the GTM Event Schema. I've designed 20 events covering every critical interaction on the OrthoNow website. Let me scroll through the schema table. You can see each event has a defined trigger type, trigger condition, GA4 event name, minimum 3 parameters, the GA4 report where it appears, the audience it creates, whether it should be a Google Ads conversion, and the business reason explaining why this event matters.

> The events are organized into 7 categories: booking funnel events, call now buttons across 3 page sections, WhatsApp widget, patient guide downloads, clinic location pages, blog scroll depth at 4 thresholds, and two additional recommended events for form field interactions and consultation submissions."

### [0:50 – 1:30] Booking Funnel Deep-Dive

*[Switch to booking-funnel.md]*

> "The booking funnel is the most critical section. I've documented exactly why GTM cannot automatically detect multi-step forms — the steps are rendered as show-hide divs, no page load occurs, no form submission fires between steps, and GTM can't know if validation passed on a button click. The solution is custom dataLayer pushes written by the frontend developer.

> Scrolling down, here are the real JSON examples — not pseudocode. Step 1 Completed includes clinic location, specialty, timestamp, session ID. Step 2 adds preferred date. Booking Completed adds the booking ID. I've also included Booking Failed and Booking Abandoned events with realistic error messages and time-on-form metrics.

> I've written a developer implementation guide — this is exactly what I would hand to the frontend team. It specifies where to place the push call, how to prevent duplicates, and how to handle the abandoned event using navigator.sendBeacon."

### [1:30 – 2:00] Google Ads Conversion Strategy

> "For Google Ads, I'm importing only `booking_completed` as the primary conversion. Not Step 1, not Step 2, not Step 3 — only the final server-confirmed booking. Why? Because Smart Bidding trains on primary conversions. If you import Step 1 and Step 2 as primary, the algorithm inflates conversion counts 3x and destroys your CPA targets. Call clicks and WhatsApp clicks are imported as secondary conversions for observation only.

> Let's move to the landing page."

---

## PART 2 — Landing Page (2:00 – 5:00)

### [2:00 – 2:30] Page Overview

*[Switch to browser with landing page open]*

> "Task 2 — the landing page. This is a single HTML file with everything inlined — HTML, CSS, and JavaScript. No frameworks, no Bootstrap, no Tailwind, no CDN dependencies. Let me walk through it from a user's perspective.

> At the top, you have a sticky header with the OrthoNow logo and a call button. The hero section has a clear headline — 'Don't let pain hold you back' — targeting working professionals in Bengaluru dealing with back pain and knee pain. Below the headline are pain-point tags that match the Google Ads keywords this traffic would come from."

### [2:30 – 3:00] Form & CRO Design

> "The form is the heart of this page. It's a minimal 2-field form — just Name and Phone. I deliberately excluded email because OrthoNow's audience in India responds better via phone and WhatsApp. The +91 prefix is pre-filled to reduce typing. The CTA says 'Book My Free Consultation' — not a generic 'Submit'. Above the form, there's an urgency signal: '200+ consultations booked this week.'

> The amber CTA button uses high-contrast color against the white card — it's the most visually prominent element on the page. This isn't accidental. Every design decision here is a conversion optimization decision."

### [3:00 – 3:30] Form Validation & dataLayer Demo

*[Click submit with empty fields to show validation]*

> "Let me demonstrate the validation. Clicking submit with empty fields shows inline errors. Let me enter an invalid phone number — '1234567890'. The validator catches that Indian mobile numbers must start with 6, 7, 8, or 9.

> Now let me open the console and submit a valid form."

*[Enter valid data, open console, submit]*

> "Watch the dataLayer. There — `consultation_form_submitted` event with `form_name`, `patient_name`, `patient_phone` with +91 prefix, `clinic_preference`, `timestamp`, `traffic_source`, `campaign`, and `session_id`. This push fires exactly on successful submission — not on page load, not on button click, only after validation passes.

> Notice the thank-you state appeared without a page reload. It shows clear next steps: 'Our team calls you', 'WhatsApp confirmation', 'Visit the clinic.' And if I try to submit again — nothing happens. Duplicate prevention is active."

### [3:30 – 4:15] Trust & Content Sections

*[Scroll down through the page]*

> "Below the fold, we have the trust section. Doctor credentials — Dr. Rajesh Malhotra with MBBS, MS Ortho, AIIMS Fellowship, 22 years of experience. This establishes clinical authority. Then three patient reviews with specific conditions, specific outcomes, and specific Bengaluru locations. Generic reviews don't convert — specific ones do.

> The conditions section targets the exact search terms: lower back pain, knee pain, sports injuries, joint stiffness. Each card speaks directly to working professionals — 'long hours at a desk', 'morning run', 'weekend sports.'

> At the bottom, a high-contrast CTA section catches scroll-past users and anchors them back to the form. And the WhatsApp floating widget is always visible for users who prefer messaging."

### [4:15 – 5:00] Performance & Mobile

> "On performance — this page targets a 90+ mobile PageSpeed score. How? Zero external images — all visuals are inline SVGs averaging 200 bytes each. Total page weight is under 25KB. One HTTP request loads the entire page. No render-blocking resources. The DOM has about 120 elements — well under the 1,500 warning threshold.

> The page is fully mobile responsive. On mobile, the form stacks below the headline so it's immediately accessible. All touch targets are appropriately sized. The WhatsApp widget stays fixed in the corner.

> This design should convert significantly higher than 2.1% because it addresses every conversion barrier: minimal form friction, above-the-fold CTA, specific trust signals, urgency elements, and mobile-first layout.

> Let's look at the integration design."

---

## PART 3 — Integration Design (5:00 – 8:00)

### [5:00 – 5:30] Architecture Overview

*[Switch to integration-design.md in VS Code]*

> "Task 3 is the integration architecture. The flow is: Landing Page form submission → HubSpot CRM → Karix WhatsApp API → Google Ads Conversion. Let me walk through each connection.

> I chose the HubSpot Contacts API v3 as the integration method — not the Forms API, not Zapier, not Make. Why? Because this form collects phone only, no email. HubSpot's Forms API and native forms use email as the primary deduplication key. Without email, they create orphaned contacts. The Contacts API lets us explicitly use phone as the lookup property."

### [5:30 – 6:15] HubSpot Integration Details

> "The request flow works like this: The browser sends a POST to our backend endpoint. The backend calls HubSpot's Contacts API with firstname, phone, clinic preference, lead status set to 'New Enquiry', and source set to 'Google Ads Consultation Landing Page.'

> Authentication uses a Private App Access Token — a Bearer token stored as an environment variable, never in client-side code.

> Now, the deduplication challenge. HubSpot deduplicates on email by default, but we only have phone. So before creating a new contact, the backend calls HubSpot's search API with a phone filter. If a match exists, we update instead of create. If two different users submit the same phone number with different names, the system updates the name to the most recent submission and appends a timeline note flagging the conflict for the sales team."

### [6:15 – 7:15] WhatsApp & Error Handling

> "For WhatsApp, the Karix API must send a confirmation message within 2 minutes of form submission. This is processed asynchronously through a message queue — SQS or Redis — to handle traffic spikes without blocking the main request.

> I've addressed every failure mode specified in the assignment. Queue delays are mitigated by auto-scaling consumers. API failures trigger 3 retries with exponential backoff — 1 second, 4 seconds, 16 seconds. Network timeouts are set at 5 seconds with cross-zone retry. Rate limits are managed by a token bucket rate limiter on the queue consumer.

> If all retries fail, the message goes to a dead-letter queue and an SMS is sent via a fallback provider like MSG91. Every API call is logged with request ID, timestamp, response status, and latency to CloudWatch or Stackdriver. If the failure rate exceeds 5% in a 15-minute window, PagerDuty alerts the engineering team."

### [7:15 – 7:45] Google Ads Conversion

> "The Google Ads conversion — `consultation_form_submitted` — fires in the browser via `dataLayer.push` before the backend call begins. This is intentional. Even if the backend fails, the conversion is recorded in GA4 because it happened client-side. GA4 is linked to Google Ads, and this event is imported as a secondary conversion for landing page campaign optimization.

> The primary conversion in Google Ads remains `booking_completed` from the main website. This landing page event complements it by measuring the top-of-funnel lead capture specific to paid campaigns."

### [7:45 – 8:00] Closing

> "To summarize: Task 1 delivers a 20-event GTM schema with real dataLayer JSON and a detailed frontend developer handoff guide. Task 2 delivers a production-ready landing page designed to significantly outperform the current 2.1% conversion rate through minimal form friction, specific trust signals, and mobile-first design. Task 3 delivers an integration architecture with phone-based deduplication, 2-minute WhatsApp SLA, comprehensive error handling, and proper conversion attribution.

> Thank you for reviewing. I'm happy to answer any questions about the implementation."

---

## Post-Recording Checklist

- [ ] Video is under 8 minutes 30 seconds
- [ ] Audio is clear (no background noise)
- [ ] Screen recording shows: landing page, console, VS Code with docs
- [ ] All 3 tasks are covered proportionally (2:00 / 3:00 / 3:00)
- [ ] Upload to Loom, set privacy to "Anyone with the link"
- [ ] Copy Loom URL into submission email

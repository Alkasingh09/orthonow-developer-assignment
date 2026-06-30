# Integration Design — End-to-End Architecture

## Landing Page → HubSpot CRM → Karix WhatsApp API → Google Ads Conversion

---

### Chosen Integration Method: HubSpot Contacts API (v3, RESTful)

**Why the Contacts API over the alternatives:**

The HubSpot Contacts API (v3) is the correct choice because this form collects **phone only — no email**. HubSpot's native Forms API and Native Embedded Forms both require email as the primary deduplication key and will create orphaned contacts without one. The Contacts API allows us to explicitly define phone as the lookup property using the `idProperty` parameter, enabling phone-based deduplication at the API level. Zapier and Make are rejected because they introduce third-party latency (500ms–3s per execution), have per-task pricing that scales poorly with lead volume, and create an opaque failure layer that cannot be monitored with our own alerting. A direct API call from a lightweight Node.js backend (or serverless function on AWS Lambda / Google Cloud Functions) gives us full control over error handling, retry logic, and response time SLAs.

---

### Request Flow

When a user submits the landing page form, the browser sends a POST request to our backend endpoint (e.g., `POST /api/leads`). The backend executes three operations sequentially with a parallel fallback: **(1)** Create or update the contact in HubSpot CRM, **(2)** Send a WhatsApp confirmation via Karix API, **(3)** Confirm the Google Ads conversion event via the frontend dataLayer push (which already fired in the browser before the backend call).

**Authentication:** The backend authenticates with HubSpot using a Private App Access Token (Bearer token in the `Authorization` header), stored as an environment variable — never in client-side code. Karix API uses an API key + account SID passed in request headers. Both tokens are rotated quarterly and stored in a secrets manager (AWS Secrets Manager or GCP Secret Manager).

**HubSpot Payload:** The backend calls `POST https://api.hubapi.com/crm/v3/objects/contacts` with a JSON body containing `properties: { firstname, phone, clinic_preference, hs_lead_status: "New Enquiry", lead_source: "Google Ads Consultation Landing Page" }`. If HubSpot returns a `409 Conflict` (contact already exists), the backend falls back to `PATCH /crm/v3/objects/contacts/{contactId}` to update the existing record. This handles **phone-based deduplication**: before creating, we call `GET /crm/v3/objects/contacts/search` with a filter on the `phone` property. If a match is found, we update rather than create. If two users submit the same phone number with different names, the system updates the existing contact's name to the most recent submission and appends a timeline note documenting the conflict — this preserves data integrity while flagging the discrepancy for the sales team to resolve manually.

**Response Handling:** On a successful HubSpot response (HTTP 200/201), the backend proceeds to the Karix WhatsApp call. On failure, the lead is written to a PostgreSQL fallback table and enqueued in a dead-letter queue (AWS SQS or Redis) for retry.

---

### WhatsApp via Karix API — 2-Minute SLA

The Karix WhatsApp Business API call (`POST https://api.karix.io/message/`) sends a pre-approved HSM template message (e.g., "Hi {{name}}, your consultation at OrthoNow is confirmed. Our team will call you shortly.") within the 2-minute window. **Possible failures and mitigations:**

**Queue delays:** The backend processes the WhatsApp call asynchronously using a message queue (SQS/Redis). If the queue backs up under high traffic (e.g., campaign launch spike), a dedicated consumer auto-scales to maintain the 2-minute SLA. **API failures:** Karix returns HTTP 4xx/5xx — the system retries 3 times with exponential backoff (1s, 4s, 16s). After 3 failures, the message is routed to a dead-letter queue and an alert fires to the engineering Slack channel via PagerDuty. **Network failures:** Timeout set at 5 seconds; on timeout, the request is retried from a different availability zone. **Rate limits:** Karix enforces per-second rate limits — the queue consumer implements a token bucket rate limiter to stay within bounds. **Fallback:** If WhatsApp delivery fails after all retries, an SMS is sent via a secondary provider (e.g., Twilio or MSG91) as a guaranteed fallback. **Monitoring:** Every Karix API call is logged with request ID, timestamp, response status, and latency to a centralized logging system (CloudWatch / Stackdriver). A dashboard tracks delivery success rate, p95 latency, and failure rate. If the failure rate exceeds 5% in any 15-minute window, an automated alert triggers.

---

### Google Ads Conversion

The `consultation_form_submitted` event fires in the browser via `window.dataLayer.push()` at the moment of successful form submission — **before** the backend API call begins. This ensures the Google Ads conversion is recorded even if the backend subsequently fails. GTM captures this event and sends it to GA4, which is linked to Google Ads for conversion import. This event is imported as a **secondary conversion** in Google Ads (the primary conversion remains `booking_completed` from the main website). The conversion fires **when** the user clicks submit and validation passes, **how** via the dataLayer → GTM → GA4 → Google Ads pipeline, and **why** because it measures the landing page's direct contribution to lead generation, enabling campaign-level CPA optimization and ROAS calculation specific to this consultation funnel.

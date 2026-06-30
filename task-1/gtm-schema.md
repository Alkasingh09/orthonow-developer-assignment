# GTM Event Schema — OrthoNow

## Naming Convention

All event names follow `snake_case` using the pattern:

```
{category}_{action}_{qualifier}
```

**Examples:**
- `booking_step_completed`
- `call_now_clicked`
- `whatsapp_widget_clicked`

**Rationale:** This convention aligns with GA4's recommended naming format, ensures consistency across all tracking, and makes events immediately readable in GA4 DebugView, BigQuery exports, and Google Ads conversion imports.

---

## Variables Required in GTM

Before creating tags and triggers, the following GTM variables must be configured:

| Variable Name | Variable Type | Purpose |
|---|---|---|
| `DLV - event` | Data Layer Variable | Captures the `event` key from dataLayer pushes |
| `DLV - step_number` | Data Layer Variable | Captures the booking funnel step number |
| `DLV - step_name` | Data Layer Variable | Captures the booking funnel step name |
| `DLV - clinic_location` | Data Layer Variable | Captures selected clinic location |
| `DLV - specialty` | Data Layer Variable | Captures selected medical specialty |
| `DLV - booking_id` | Data Layer Variable | Captures booking confirmation ID |
| `DLV - preferred_date` | Data Layer Variable | Captures user's preferred appointment date |
| `DLV - form_name` | Data Layer Variable | Captures form name for lead forms |
| `DLV - user_name` | Data Layer Variable | Captures user-submitted name |
| `DLV - user_phone` | Data Layer Variable | Captures user-submitted phone |
| `DLV - guide_name` | Data Layer Variable | Captures patient guide PDF name |
| `DLV - scroll_depth` | Built-in Variable | Scroll depth percentage threshold |
| `DLV - user_type` | Data Layer Variable | New vs. returning visitor |
| `DLV - traffic_source` | Data Layer Variable | Source of traffic |
| `DLV - campaign` | Data Layer Variable | Campaign name |
| `DLV - session_id` | Data Layer Variable | Session identifier |
| `Click URL` | Built-in Variable | URL of clicked element |
| `Click Text` | Built-in Variable | Text of clicked element |
| `Click Classes` | Built-in Variable | CSS classes of clicked element |
| `Page Path` | Built-in Variable | Current page path |
| `Page URL` | Built-in Variable | Full page URL |
| `Page Title` | Built-in Variable | Current page title |
| `Referrer` | Built-in Variable | Referrer URL |

---

## Complete Event Schema

### 1. Booking Funnel Events

| Event Name | Purpose | Trigger Type | Trigger Condition | GA4 Event Name | Parameters (Minimum 3) | GA4 Report | GA4 Audience | Google Ads Conversion | Business Reason |
|---|---|---|---|---|---|---|---|---|---|
| `booking_step1_completed` | Track when user selects clinic location & specialty | Custom Event | dataLayer event = `booking_step1_completed` | `booking_step1_completed` | `step_number`, `step_name`, `clinic_location`, `specialty`, `page_location`, `page_title`, `timestamp`, `user_type`, `traffic_source`, `campaign`, `session_id` | Funnel Exploration → Step 1 entry count | Users who completed Step 1 but not Step 2 (Abandoners Segment) | No | Measures top-of-funnel intent. Identifies which clinic locations and specialties generate the most interest. High volume here with low Step 2 completion signals UX friction on the name/phone form. |
| `booking_step2_completed` | Track when user enters name, phone, and preferred date | Custom Event | dataLayer event = `booking_step2_completed` | `booking_step2_completed` | `step_number`, `step_name`, `clinic_location`, `specialty`, `preferred_date`, `page_location`, `page_title`, `timestamp`, `user_type`, `traffic_source`, `campaign`, `session_id` | Funnel Exploration → Step 2 completion rate | Users who completed Step 2 but not Step 3 (High-intent Abandoners) | No | Captures personal information submission. A drop-off here indicates trust issues (users reluctant to share phone number) or date availability friction. |
| `booking_step3_completed` | Track when user clicks "Confirm Booking" | Custom Event | dataLayer event = `booking_step3_completed` | `booking_step3_completed` | `step_number`, `step_name`, `clinic_location`, `specialty`, `preferred_date`, `booking_id`, `page_location`, `page_title`, `timestamp`, `user_type`, `traffic_source`, `campaign`, `session_id` | Funnel Exploration → Step 3 confirmation rate | Users who confirmed booking (Converted Patients) | No | Measures confirmation intent. Drop-off here means user reviewed details but chose not to confirm — possible pricing concern or scheduling conflict. |
| `booking_completed` | Track successful booking confirmation (server response) | Custom Event | dataLayer event = `booking_completed` | `booking_completed` | `booking_id`, `clinic_location`, `specialty`, `preferred_date`, `page_location`, `page_title`, `timestamp`, `user_type`, `traffic_source`, `campaign`, `session_id` | Conversions report; Acquisition → Traffic acquisition with conversion filter | Converted Patients audience for Google Ads remarketing exclusion | **Yes** | **Primary conversion event.** This is the ultimate business outcome — a confirmed appointment. Used for Google Ads Smart Bidding optimization (Target CPA / Maximize Conversions). |
| `booking_failed` | Track booking submission failures (API error, timeout) | Custom Event | dataLayer event = `booking_failed` | `booking_failed` | `error_message`, `clinic_location`, `specialty`, `page_location`, `page_title`, `timestamp`, `user_type`, `session_id` | Custom report: Failed Bookings by Clinic & Error Type | Users who experienced booking failure (for remarketing with apology offer) | No | Identifies technical failures losing revenue. A booking failure is a lost patient. Monitoring this event alerts engineering to API issues and lets marketing retarget these users. |
| `booking_abandoned` | Track when user starts but does not complete booking within session | Custom Event | dataLayer event = `booking_abandoned` | `booking_abandoned` | `last_step_completed`, `clinic_location`, `specialty`, `page_location`, `page_title`, `timestamp`, `user_type`, `traffic_source`, `campaign`, `session_id` | Funnel Exploration → Abandonment analysis | Booking Abandoners audience (for remarketing campaigns) | No | Identifies the exact funnel step where users drop. Enables remarketing to high-intent users who started but didn't finish booking. Critical for reducing the gap between 2.1% and the 6–8% benchmark. |

---

### 2. Call Now Button Events

| Event Name | Purpose | Trigger Type | Trigger Condition | GA4 Event Name | Parameters (Minimum 3) | GA4 Report | GA4 Audience | Google Ads Conversion | Business Reason |
|---|---|---|---|---|---|---|---|---|---|
| `call_now_clicked_homepage` | Track Call Now clicks on homepage | Click — All Elements | Click URL contains `tel:` AND Page Path = `/` | `call_now_clicked` | `page_section: homepage`, `click_url`, `page_location`, `page_title`, `timestamp`, `user_type`, `traffic_source`, `campaign`, `session_id` | Events report filtered by `page_section = homepage` | Users who clicked Call Now on homepage (Phone Lead audience) | Yes | Phone calls are high-intent conversions. Tracking by page section reveals which page drives the most call leads. Homepage calls indicate brand-aware users. |
| `call_now_clicked_clinic` | Track Call Now clicks on clinic pages | Click — All Elements | Click URL contains `tel:` AND Page Path matches `/clinic/*` | `call_now_clicked` | `page_section: clinic_page`, `clinic_location`, `click_url`, `page_location`, `page_title`, `timestamp`, `user_type`, `traffic_source`, `campaign`, `session_id` | Events report filtered by `page_section = clinic_page`, broken down by `clinic_location` | Users who clicked Call Now on specific clinic page | Yes | Clinic-page call clicks indicate users comparing locations. Tracking per clinic reveals which locations generate phone inquiries vs. online bookings. |
| `call_now_clicked_landing` | Track Call Now clicks on landing pages | Click — All Elements | Click URL contains `tel:` AND Page Path matches `/lp/*` or campaign landing page URL | `call_now_clicked` | `page_section: landing_page`, `click_url`, `page_location`, `page_title`, `timestamp`, `user_type`, `traffic_source`, `campaign`, `session_id` | Events report filtered by `page_section = landing_page` with campaign dimension | Users who clicked Call Now on landing page (Campaign Responders audience) | Yes | Landing page call clicks represent campaign-driven phone leads. Essential for calculating true campaign ROI (form submissions + phone calls). Without this, you undercount conversions by 30–40%. |

---

### 3. WhatsApp Widget Event

| Event Name | Purpose | Trigger Type | Trigger Condition | GA4 Event Name | Parameters (Minimum 3) | GA4 Report | GA4 Audience | Google Ads Conversion | Business Reason |
|---|---|---|---|---|---|---|---|---|---|
| `whatsapp_widget_clicked` | Track floating WhatsApp widget clicks | Click — All Elements | Click Element matches CSS selector for WhatsApp widget (e.g., `.whatsapp-float`, `a[href*="wa.me"]`, or `a[href*="whatsapp"]`) | `whatsapp_widget_clicked` | `page_location`, `page_title`, `click_url`, `timestamp`, `user_type`, `traffic_source`, `campaign`, `session_id` | Events report → WhatsApp clicks by page, device, and source | Users who prefer WhatsApp communication (WhatsApp Engagers audience) | Yes | WhatsApp is the dominant patient communication channel in India. Tracking this reveals the share of users preferring messaging over calls/forms. High WhatsApp clicks with low form submissions signals the need for WhatsApp-native booking flows. |

---

### 4. Patient Guide Download Events

| Event Name | Purpose | Trigger Type | Trigger Condition | GA4 Event Name | Parameters (Minimum 3) | GA4 Report | GA4 Audience | Google Ads Conversion | Business Reason |
|---|---|---|---|---|---|---|---|---|---|
| `guide_form_submitted` | Track when user submits name + phone to access the patient guide | Custom Event | dataLayer event = `guide_form_submitted` | `guide_form_submitted` | `guide_name`, `form_name`, `user_name`, `user_phone`, `page_location`, `page_title`, `timestamp`, `user_type`, `traffic_source`, `campaign`, `session_id` | Events report → Guide form submissions by guide name | Lead Magnet Responders audience (for nurture email/WhatsApp campaigns) | No | Captures contact information from users not yet ready to book. These are mid-funnel leads requiring nurture. The guide download is a micro-conversion indicating clinical interest. |
| `guide_downloaded` | Track actual PDF download after form submission | Custom Event | dataLayer event = `guide_downloaded` | `guide_downloaded` | `guide_name`, `file_url`, `page_location`, `page_title`, `timestamp`, `user_type`, `traffic_source`, `campaign`, `session_id` | Events report → Downloads by guide name and traffic source | Guide Downloaders audience (for remarketing with booking offers) | No | Confirms the user actually downloaded the PDF (not just submitted the form). Discrepancy between form submission and download counts reveals UX issues (broken links, slow downloads). |

---

### 5. Clinic Location Page Events

| Event Name | Purpose | Trigger Type | Trigger Condition | GA4 Event Name | Parameters (Minimum 3) | GA4 Report | GA4 Audience | Google Ads Conversion | Business Reason |
|---|---|---|---|---|---|---|---|---|---|
| `clinic_page_viewed` | Track views of each of the 9 clinic location pages | Page View (History Change or standard) | Page Path matches `/clinic/koramangala`, `/clinic/indiranagar`, `/clinic/hsr-layout`, `/clinic/whitefield`, `/clinic/electronic-city`, `/clinic/hitech-city`, `/clinic/gachibowli`, `/clinic/anna-nagar`, `/clinic/t-nagar` (all 9 clinics) | `clinic_page_viewed` | `clinic_location`, `city`, `page_location`, `page_title`, `timestamp`, `user_type`, `traffic_source`, `campaign`, `session_id` | Pages and screens report filtered by clinic pages; custom report by `clinic_location` and `city` | Users who viewed specific clinic pages (Location-intent audience) | No | Reveals geographic demand distribution across the 9 clinics in Bengaluru, Hyderabad, and Chennai. Informs Google Ads location bid adjustments and clinic-specific budget allocation. A clinic page with high views but low bookings needs UX investigation. |
| `clinic_directions_clicked` | Track when user clicks "Get Directions" / Google Maps link on clinic page | Click — All Elements | Click URL contains `maps.google.com` or `goo.gl/maps` AND Page Path matches `/clinic/*` | `clinic_directions_clicked` | `clinic_location`, `city`, `click_url`, `page_location`, `page_title`, `timestamp`, `user_type`, `traffic_source`, `session_id` | Events report → Directions clicks by clinic | Users who clicked directions (High Walk-in Intent audience) | No | Direction clicks are a strong signal of walk-in intent. These users are physically planning to visit. High direction clicks with no subsequent booking may indicate walk-in patients not captured in digital conversion data. |

---

### 6. Blog Article Events

| Event Name | Purpose | Trigger Type | Trigger Condition | GA4 Event Name | Parameters (Minimum 3) | GA4 Report | GA4 Audience | Google Ads Conversion | Business Reason |
|---|---|---|---|---|---|---|---|---|---|
| `blog_scroll_25` | Track 25% scroll depth on blog articles | Scroll Depth | Vertical Scroll Depth ≥ 25% AND Page Path matches `/blog/*` | `blog_scroll_depth` | `scroll_threshold: 25`, `page_location`, `page_title`, `timestamp`, `user_type`, `traffic_source`, `session_id` | Custom report: Blog engagement by scroll depth | Casual Readers audience (25% scroll, content browsers) | No | Baseline engagement metric. Users scrolling 25% have at minimum read the headline and introduction. Low 25% rates indicate poor headline-content match or slow page load. |
| `blog_scroll_50` | Track 50% scroll depth on blog articles | Scroll Depth | Vertical Scroll Depth ≥ 50% AND Page Path matches `/blog/*` | `blog_scroll_depth` | `scroll_threshold: 50`, `page_location`, `page_title`, `timestamp`, `user_type`, `traffic_source`, `session_id` | Custom report: Blog engagement by scroll depth | Engaged Readers audience (50%+ scroll) | No | Indicates genuine content engagement. Users have consumed half the article. Comparing 50% to 25% rates reveals content quality — a steep drop signals content fails to hold attention past the intro. |
| `blog_scroll_75` | Track 75% scroll depth on blog articles | Scroll Depth | Vertical Scroll Depth ≥ 75% AND Page Path matches `/blog/*` | `blog_scroll_depth` | `scroll_threshold: 75`, `page_location`, `page_title`, `timestamp`, `user_type`, `traffic_source`, `session_id` | Custom report: Blog engagement by scroll depth | Highly Engaged Readers audience (75%+ scroll, for remarketing) | No | High engagement signal. These users are consuming nearly the full article, indicating strong clinical interest. Ideal audience for remarketing with booking CTAs — they have educated themselves and may be ready to convert. |
| `blog_scroll_100` | Track 100% scroll depth on blog articles | Scroll Depth | Vertical Scroll Depth = 100% AND Page Path matches `/blog/*` | `blog_scroll_depth` | `scroll_threshold: 100`, `page_location`, `page_title`, `timestamp`, `user_type`, `traffic_source`, `session_id` | Custom report: Blog completions by article | Article Completers audience (for remarketing with specific specialty offers) | No | Full article consumption. These users are the most informed prospects. Tracking which articles get 100% scroll reveals the most compelling content topics — use these topics for ad copy and future content strategy. |

---

### 7. Additional Recommended Events

| Event Name | Purpose | Trigger Type | Trigger Condition | GA4 Event Name | Parameters (Minimum 3) | GA4 Report | GA4 Audience | Google Ads Conversion | Business Reason |
|---|---|---|---|---|---|---|---|---|---|
| `consultation_form_submitted` | Track landing page lead form submission | Custom Event | dataLayer event = `consultation_form_submitted` | `consultation_form_submitted` | `form_name`, `clinic_preference`, `page_location`, `page_title`, `timestamp`, `user_type`, `traffic_source`, `campaign`, `session_id` | Conversions report; Landing page conversion rate | Landing Page Converters audience | **Yes** | Primary landing page conversion. Directly measures campaign effectiveness and landing page performance. Used for Google Ads conversion optimization and CPA bidding. |
| `form_field_interaction` | Track when user first interacts with a form field | Custom Event | dataLayer event = `form_field_interaction` | `form_field_interaction` | `field_name`, `form_name`, `page_location`, `page_title`, `timestamp`, `user_type`, `session_id` | Custom report: Form field interaction rates | Form Starters (for remarketing to users who started but didn't submit) | No | Reveals form friction at the field level. If 80% of users interact with the Name field but only 40% interact with the Phone field, it signals phone number collection is causing drop-off. Actionable UX insight. |

---

## Summary Statistics

- **Total Events:** 20
- **Google Ads Conversions:** 5 (booking_completed, call_now × 3 sections, whatsapp_widget_clicked, consultation_form_submitted)
- **Booking Funnel Events:** 6
- **Engagement Events:** 8
- **Lead Capture Events:** 3
- **Location Events:** 2

---

## GTM Tag Configuration Summary

For each event above, a corresponding **GA4 Event Tag** must be created in GTM:

1. **Tag Type:** Google Analytics: GA4 Event
2. **Measurement ID:** `G-XXXXXXXXXX` (OrthoNow's GA4 property)
3. **Event Name:** As specified in the "GA4 Event Name" column
4. **Event Parameters:** As specified, using the GTM variables defined above
5. **Trigger:** As specified in the "Trigger Type" and "Trigger Condition" columns

### Trigger Setup Pattern for Custom Events (dataLayer-based):

```
Trigger Type: Custom Event
Event Name: {{event name from dataLayer push}}
This trigger fires on: All Custom Events (matching the event name)
```

### Trigger Setup Pattern for Click Events:

```
Trigger Type: Click — All Elements
This trigger fires on: Some Clicks
Conditions: Click URL contains "tel:" / "wa.me" / "whatsapp" AND Page Path matches pattern
```

### Trigger Setup Pattern for Scroll Events:

```
Trigger Type: Scroll Depth
Vertical Scroll Depths: 25, 50, 75, 100
This trigger fires on: Some Scroll Depth Events
Conditions: Page Path matches "/blog/*"
```

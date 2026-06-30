# Booking Funnel — Deep Technical Documentation

## Table of Contents

1. [Why GTM Cannot Automatically Detect Multi-Step Forms](#1-why-gtm-cannot-automatically-detect-multi-step-forms)
2. [The Solution: Custom dataLayer Pushes](#2-the-solution-custom-datalayer-pushes)
3. [Who Writes the dataLayer Push Code](#3-who-writes-the-datalayer-push-code)
4. [Frontend Developer Implementation Guide](#4-frontend-developer-implementation-guide)
5. [Real dataLayer Push JSON Examples](#5-real-datalayer-push-json-examples)
6. [How GTM Listens for dataLayer Pushes](#6-how-gtm-listens-for-datalayer-pushes)
7. [How GA4 Receives the Events](#7-how-ga4-receives-the-events)
8. [GA4 Funnel Exploration Configuration](#8-ga4-funnel-exploration-configuration)
9. [Step Drop-Off Calculation](#9-step-drop-off-calculation)
10. [Google Ads Conversion Import Strategy](#10-google-ads-conversion-import-strategy)

---

## 1. Why GTM Cannot Automatically Detect Multi-Step Forms

Google Tag Manager is designed to track standard browser interactions: page loads, element clicks, form submissions, scroll depth, and element visibility. These are observable through the DOM.

**Multi-step forms present a fundamental detection problem:**

OrthoNow's booking form uses a **single-page, multi-step UI pattern** — the user progresses through Step 1 (Select Clinic + Specialty) → Step 2 (Enter Name, Phone, Preferred Date) → Step 3 (Confirm Booking) — all **without a full page reload**. The "steps" are implemented as:

- **Showing/hiding `<div>` sections** via CSS (`display: none` / `display: block`)
- **Conditional rendering** in a JavaScript framework (React, Vue, or vanilla JS state management)
- **CSS transitions** between panels

From GTM's perspective, **nothing observable happens** when a user moves from Step 1 to Step 2:

| What GTM Can Detect | What Happens in a Multi-Step Form |
|---|---|
| Full page load (`gtm.js`) | ❌ No page load occurs |
| HTML `<form>` submission (`gtm.formSubmit`) | ❌ No `<form>` is submitted between steps |
| URL change (`gtm.historyChange`) | ❌ URL does not change (usually) |
| Specific element click (`gtm.click`) | ⚠️ GTM can detect a "Next" button click, but it **cannot know** whether the step was **completed successfully** (validation passed, data was saved) |

**The critical distinction:** GTM can detect that a "Next" button was *clicked*, but it cannot determine:

1. Whether form validation passed
2. Whether the data was saved to application state
3. Whether the step transition actually occurred
4. What specific data the user entered (clinic, specialty, date)

A click on "Next" that fails validation is **not** a step completion. Only the application's JavaScript knows whether the step truly completed.

**Therefore, GTM requires explicit signals from the frontend application** — these signals are `window.dataLayer.push()` calls.

---

## 2. The Solution: Custom dataLayer Pushes

The `dataLayer` is a JavaScript array that serves as GTM's **communication interface** with the website. It works as follows:

```
Frontend Code → window.dataLayer.push({...}) → GTM Listener → GA4 Tag → GA4 Property
```

1. The **frontend developer** adds `window.dataLayer.push()` calls at the exact moment a business event occurs
2. **GTM** has a built-in listener on the `dataLayer` array — it intercepts every push in real time
3. GTM evaluates the push against its **Custom Event triggers**
4. If a trigger matches, GTM fires the corresponding **GA4 Event Tag**
5. The GA4 tag sends the event with all parameters to **Google Analytics 4**

This is the **only reliable method** to track multi-step form interactions.

---

## 3. Who Writes the dataLayer Push Code

> **The Frontend Developer is responsible for implementing all `window.dataLayer.push()` calls.**

GTM cannot inject these pushes — they must be part of the application's source code. The GTM specialist (analytics engineer) provides the **exact specification** of what events to push, what parameters to include, and when each push should fire. The frontend developer implements this specification within the application's existing JavaScript codebase.

### Responsibility Matrix

| Role | Responsibility |
|---|---|
| **GTM / Analytics Engineer** | Defines the event schema, parameter requirements, naming conventions, and trigger conditions. Creates GTM tags and triggers. Validates implementation in GTM Preview/Debug mode. |
| **Frontend Developer** | Implements `window.dataLayer.push()` calls in the application code. Ensures pushes fire at the correct moment (after validation, after state change). Includes all required parameters with correct data types. |
| **QA Engineer** | Validates that events fire correctly across browsers, devices, and user flows. Tests edge cases (validation failures, API errors, back-button navigation). |

---

## 4. Frontend Developer Implementation Guide

### Instructions for the Frontend Team

This section is written as a **developer implementation document** — hand this directly to your frontend team.

---

#### Dear Frontend Developer,

We need you to implement the following `window.dataLayer.push()` calls in the booking form component. These pushes enable our analytics tracking through Google Tag Manager.

#### Prerequisites

1. **Ensure the GTM container snippet is installed** in the `<head>` of every page. You should already have this:
   ```html
   <!-- Google Tag Manager -->
   <script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
   new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
   j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
   'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
   })(window,document,'script','dataLayer','GTM-XXXXXXX');</script>
   <!-- End Google Tag Manager -->
   ```

2. **Initialize the dataLayer** before the GTM snippet (if not already done):
   ```html
   <script>
     window.dataLayer = window.dataLayer || [];
   </script>
   ```

#### Implementation Rules

1. **Push ONLY after successful validation and state transition.** Do NOT push on button click — push after you confirm the step completed successfully.
2. **Push ONCE per step completion.** Prevent duplicate pushes (e.g., if user clicks "Next" rapidly). Use a flag variable.
3. **Include ALL parameters** listed in the JSON specifications below. Missing parameters will cause incomplete analytics data.
4. **Use exact event names.** Case-sensitive. No spaces. Exactly as specified.
5. **Generate `session_id`** using your existing session management, or create one: `Date.now().toString(36) + Math.random().toString(36).substr(2)`
6. **`timestamp`** must be ISO 8601 format: `new Date().toISOString()`
7. **`user_type`** should be `"new"` or `"returning"` based on your session/cookie logic
8. **`traffic_source`** and **`campaign`** should be extracted from UTM parameters in the URL

#### Where to Place the Push

```javascript
// ❌ WRONG — fires on click, regardless of validation
nextButton.addEventListener('click', function() {
  window.dataLayer.push({ event: 'booking_step1_completed', ... });
});

// ✅ CORRECT — fires only after successful validation and state change
function onStep1Complete(formData) {
  // Your existing validation logic
  if (isValid) {
    // Your existing state transition logic
    showStep2();

    // THEN push to dataLayer
    window.dataLayer.push({
      event: 'booking_step1_completed',
      step_number: 1,
      step_name: 'select_clinic_specialty',
      clinic_location: formData.clinicLocation,
      specialty: formData.specialty,
      // ... all other parameters
    });
  }
}
```

#### Handling the Abandoned Event

The `booking_abandoned` event requires special handling. It should fire when:

1. User has completed at least Step 1
2. User leaves the page (closes tab, navigates away, or session times out)

Implement using the `beforeunload` event:

```javascript
let bookingStarted = false;
let bookingCompleted = false;
let lastCompletedStep = 0;

// Set bookingStarted = true when user completes Step 1
// Set bookingCompleted = true when booking is confirmed

window.addEventListener('beforeunload', function() {
  if (bookingStarted && !bookingCompleted) {
    window.dataLayer.push({
      event: 'booking_abandoned',
      last_step_completed: lastCompletedStep,
      // ... other parameters
    });
    // Use navigator.sendBeacon for reliable delivery on page unload
    navigator.sendBeacon('/analytics/beacon', JSON.stringify({
      event: 'booking_abandoned',
      last_step_completed: lastCompletedStep
    }));
  }
});
```

> **Note:** `navigator.sendBeacon()` is recommended for the abandoned event because standard `dataLayer.push()` may not complete before the page unloads. The beacon provides a reliable backup.

---

## 5. Real dataLayer Push JSON Examples

### Step 1 Completed — Clinic & Specialty Selected

```json
{
  "event": "booking_step1_completed",
  "step_number": 1,
  "step_name": "select_clinic_specialty",
  "clinic_location": "Koramangala, Bengaluru",
  "specialty": "Spine & Back Pain",
  "page_location": "https://www.orthonow.in/book-appointment",
  "page_title": "Book Appointment | OrthoNow",
  "timestamp": "2026-06-30T14:22:35.847Z",
  "user_type": "new",
  "traffic_source": "google_ads",
  "campaign": "blr_back_pain_june2026",
  "session_id": "lxk8f2m9_a7b3c1d5"
}
```

### Step 2 Completed — Personal Details Entered

```json
{
  "event": "booking_step2_completed",
  "step_number": 2,
  "step_name": "enter_personal_details",
  "clinic_location": "Koramangala, Bengaluru",
  "specialty": "Spine & Back Pain",
  "preferred_date": "2026-07-04",
  "page_location": "https://www.orthonow.in/book-appointment",
  "page_title": "Book Appointment | OrthoNow",
  "timestamp": "2026-06-30T14:23:48.192Z",
  "user_type": "new",
  "traffic_source": "google_ads",
  "campaign": "blr_back_pain_june2026",
  "session_id": "lxk8f2m9_a7b3c1d5"
}
```

### Step 3 Completed — Booking Confirmed by User (Click)

```json
{
  "event": "booking_step3_completed",
  "step_number": 3,
  "step_name": "confirm_booking",
  "clinic_location": "Koramangala, Bengaluru",
  "specialty": "Spine & Back Pain",
  "preferred_date": "2026-07-04",
  "booking_id": "ON-BLR-KRM-20260630-0847",
  "page_location": "https://www.orthonow.in/book-appointment",
  "page_title": "Book Appointment | OrthoNow",
  "timestamp": "2026-06-30T14:24:12.005Z",
  "user_type": "new",
  "traffic_source": "google_ads",
  "campaign": "blr_back_pain_june2026",
  "session_id": "lxk8f2m9_a7b3c1d5"
}
```

### Booking Completed — Server Confirmation Received

```json
{
  "event": "booking_completed",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "clinic_location": "Koramangala, Bengaluru",
  "specialty": "Spine & Back Pain",
  "preferred_date": "2026-07-04",
  "booking_id": "ON-BLR-KRM-20260630-0847",
  "booking_status": "confirmed",
  "page_location": "https://www.orthonow.in/book-appointment",
  "page_title": "Book Appointment | OrthoNow",
  "timestamp": "2026-06-30T14:24:15.331Z",
  "user_type": "new",
  "traffic_source": "google_ads",
  "campaign": "blr_back_pain_june2026",
  "session_id": "lxk8f2m9_a7b3c1d5"
}
```

### Booking Failed — API Error

```json
{
  "event": "booking_failed",
  "step_number": 3,
  "step_name": "booking_confirmation_failed",
  "clinic_location": "Koramangala, Bengaluru",
  "specialty": "Spine & Back Pain",
  "preferred_date": "2026-07-04",
  "error_message": "API_TIMEOUT: Booking service did not respond within 10 seconds",
  "error_code": "503",
  "page_location": "https://www.orthonow.in/book-appointment",
  "page_title": "Book Appointment | OrthoNow",
  "timestamp": "2026-06-30T14:24:25.998Z",
  "user_type": "new",
  "traffic_source": "google_ads",
  "campaign": "blr_back_pain_june2026",
  "session_id": "lxk8f2m9_a7b3c1d5"
}
```

### Booking Abandoned — User Left Without Completing

```json
{
  "event": "booking_abandoned",
  "last_step_completed": 2,
  "step_name": "abandoned_after_personal_details",
  "clinic_location": "Koramangala, Bengaluru",
  "specialty": "Spine & Back Pain",
  "preferred_date": "2026-07-04",
  "page_location": "https://www.orthonow.in/book-appointment",
  "page_title": "Book Appointment | OrthoNow",
  "timestamp": "2026-06-30T14:28:55.102Z",
  "user_type": "new",
  "traffic_source": "google_ads",
  "campaign": "blr_back_pain_june2026",
  "session_id": "lxk8f2m9_a7b3c1d5",
  "time_on_form_seconds": 260
}
```

---

## 6. How GTM Listens for dataLayer Pushes

When the GTM container loads, it replaces the native `Array.push()` method on the `window.dataLayer` array with a **custom push function**. This custom function:

1. **Adds the pushed object** to the dataLayer array (maintaining it as a data store)
2. **Evaluates the pushed object** against all configured triggers
3. **Fires matching triggers** immediately and synchronously

### GTM Trigger Configuration for Booking Events

For each booking event, create a **Custom Event trigger** in GTM:

#### Example: Trigger for `booking_step1_completed`

```
Trigger Name: CE — Booking Step 1 Completed
Trigger Type: Custom Event
Event Name: booking_step1_completed
This trigger fires on: All Custom Events
```

When `window.dataLayer.push({ event: 'booking_step1_completed', ... })` executes:

1. GTM's custom push interceptor detects the `event` key
2. GTM evaluates the value `"booking_step1_completed"` against all Custom Event triggers
3. The trigger `CE — Booking Step 1 Completed` matches
4. GTM fires the associated **GA4 Event Tag**

### GTM Variable Extraction

When the trigger fires, GTM simultaneously makes all pushed key-value pairs available as **Data Layer Variables**. The GA4 Event Tag is configured to read these variables:

```
Tag Name: GA4 Event — Booking Step 1 Completed
Tag Type: Google Analytics: GA4 Event
Measurement ID: G-XXXXXXXXXX
Event Name: booking_step1_completed
Event Parameters:
  step_number    = {{DLV - step_number}}
  step_name      = {{DLV - step_name}}
  clinic_location = {{DLV - clinic_location}}
  specialty      = {{DLV - specialty}}
  page_location  = {{DLV - page_location}}
  ... (all parameters)
Trigger: CE — Booking Step 1 Completed
```

---

## 7. How GA4 Receives the Events

When the GA4 Event Tag fires, GTM sends an HTTP request to GA4's collection endpoint:

```
POST https://www.google-analytics.com/g/collect
```

The request includes:

- **Measurement ID** (`G-XXXXXXXXXX`)
- **Event name** (`booking_step1_completed`)
- **All event parameters** as key-value pairs
- **Automatically collected parameters** (client_id, session_id, page_location, etc.)

GA4 processes this event in **real-time** (visible in DebugView within seconds) and in **standard reporting** within 24–48 hours.

### GA4 Event Configuration

In the GA4 property, the following events should be **marked as conversions**:

| Event Name | Mark as Conversion |
|---|---|
| `booking_completed` | ✅ Yes |
| `consultation_form_submitted` | ✅ Yes |
| `call_now_clicked` | ✅ Yes |
| `whatsapp_widget_clicked` | ✅ Yes |

All other events remain as standard events for analysis.

### Custom Dimensions Registration

Register the following **custom dimensions** in GA4 (Admin → Custom definitions):

| Dimension Name | Scope | Event Parameter |
|---|---|---|
| Clinic Location | Event | `clinic_location` |
| Specialty | Event | `specialty` |
| Step Name | Event | `step_name` |
| Step Number | Event | `step_number` |
| Booking ID | Event | `booking_id` |
| Guide Name | Event | `guide_name` |
| City | Event | `city` |
| Page Section | Event | `page_section` |
| Scroll Threshold | Event | `scroll_threshold` |
| Form Name | Event | `form_name` |
| Error Message | Event | `error_message` |
| Last Step Completed | Event | `last_step_completed` |
| Traffic Source (custom) | Event | `traffic_source` |
| Campaign (custom) | Event | `campaign` |

---

## 8. GA4 Funnel Exploration Configuration

### Setup Steps

1. Navigate to **GA4 → Explore → Funnel Exploration**
2. Click **"+"** to create a new exploration
3. Name it: **"OrthoNow Booking Funnel"**

### Funnel Steps Configuration

| Step | Event Name | Step Name (Display) |
|---|---|---|
| Step 1 | `booking_step1_completed` | Clinic & Specialty Selected |
| Step 2 | `booking_step2_completed` | Personal Details Entered |
| Step 3 | `booking_step3_completed` | Booking Confirmed (Click) |
| Step 4 | `booking_completed` | Booking Confirmed (Server) |

### Funnel Settings

| Setting | Value | Reason |
|---|---|---|
| **Open/Closed Funnel** | Closed | Users must enter at Step 1. This prevents users who directly complete Step 3 (impossible in our UX) from skewing the data. |
| **Elapsed Time** | Optional — enable to see average time between steps | Identifies steps where users hesitate. |
| **Make Open Funnel** | No | An open funnel would count users entering at any step, which doesn't match our linear booking flow. |

### Breakdown Dimensions

Add these **breakdowns** to segment funnel performance:

- `clinic_location` — Which clinic has the best/worst funnel completion rate?
- `specialty` — Which specialty converts best?
- `traffic_source` — Do Google Ads users convert better than organic users?
- `device_category` — Mobile vs. desktop funnel performance
- `campaign` — Which campaign drives the highest quality traffic?

### Segment Comparisons

Create these **segments** for comparison:

1. **Bengaluru Users** vs. **Hyderabad Users** vs. **Chennai Users**
2. **Google Ads Traffic** vs. **Organic Traffic**
3. **Mobile Users** vs. **Desktop Users**

---

## 9. Step Drop-Off Calculation

### Drop-Off Formula

```
Drop-off Rate (Step N) = ((Users at Step N) - (Users at Step N+1)) / (Users at Step N) × 100
```

### Example Calculation (Hypothetical Monthly Data)

| Step | Users | Completion Rate | Drop-Off Rate | Drop-Off Count |
|---|---|---|---|---|
| Step 1: Clinic & Specialty Selected | 5,000 | 100% (entry) | — | — |
| Step 2: Personal Details Entered | 3,200 | 64.0% | **36.0%** | 1,800 |
| Step 3: Booking Confirmed (Click) | 2,400 | 75.0% | **25.0%** | 800 |
| Step 4: Booking Confirmed (Server) | 2,280 | 95.0% | **5.0%** | 120 |

### Interpretation

- **Step 1 → Step 2 (36% drop-off):** 1,800 users selected a clinic and specialty but abandoned when asked for personal details. **Action:** This is the highest drop-off point. Investigate:
  - Is the form visually overwhelming? Consider progressive disclosure.
  - Is phone number mandatory? Test making it optional.
  - Add trust signals ("Your information is secure. We will only call to confirm your appointment.")

- **Step 2 → Step 3 (25% drop-off):** 800 users entered their details but did not click "Confirm." **Action:** Investigate:
  - Is the confirmation screen clearly showing what they're booking?
  - Is there a surprise (unexpected cost, no available dates)?
  - Add a reassurance: "Free consultation. No payment required."

- **Step 3 → Step 4 (5% drop-off):** 120 users clicked "Confirm" but the server did not confirm the booking. **Action:** This is a **technical issue**. Investigate:
  - API timeouts
  - Server errors
  - Monitor `booking_failed` event for error details

### Where to View Drop-Off in GA4

1. **Funnel Exploration** → The visualization automatically shows a bar chart with drop-off at each step
2. **Right side panel** → Click any step to see the exact count and percentage
3. **Abandonment analysis** → Enable "Show elapsed time" to see how long users spend at each step before dropping off

---

## 10. Google Ads Conversion Import Strategy

### Chosen Primary Conversion: `booking_completed`

**Only `booking_completed` should be imported as a PRIMARY conversion in Google Ads.**

### Why `booking_completed` and Not Other Events

| Event | Import? | Reason |
|---|---|---|
| `booking_completed` | ✅ **Primary** | This is the actual business outcome — a confirmed appointment. Google Ads Smart Bidding (Target CPA, Maximize Conversions) should optimize for real bookings, not intermediate steps. |
| `booking_step1_completed` | ❌ No | Too early in the funnel. Optimizing for Step 1 completions would bring low-quality traffic that selects a clinic but never books. Smart Bidding would learn to find "clickers," not "bookers." |
| `booking_step2_completed` | ❌ No | Still not a conversion. Users who enter their name and phone but don't confirm are not patients. |
| `booking_step3_completed` | ❌ No | Close, but not confirmed. There's a 5% gap between Step 3 and actual confirmation. Optimizing for Step 3 introduces noise. |
| `call_now_clicked` | ⚠️ Secondary | Import as **SECONDARY** (observation only). Call clicks are valuable but Google cannot verify if the call was answered or led to a booking. Use Google's Call Conversion tracking for actual calls instead. |
| `whatsapp_widget_clicked` | ⚠️ Secondary | Import as **SECONDARY**. WhatsApp clicks are engagement signals but not confirmed conversions. |
| `consultation_form_submitted` | ⚠️ Secondary | Import as **SECONDARY** for landing pages. Useful for landing page campaigns where the main site booking flow isn't the conversion action. Can be promoted to PRIMARY for specific landing page campaigns. |

### Google Ads Configuration

1. **Link GA4 to Google Ads** (Admin → Google Ads Links)
2. In Google Ads: **Tools → Conversions → Import → Google Analytics 4**
3. Select `booking_completed`
4. Set as **Primary** conversion action
5. **Conversion window:** 30 days (patient may book after researching)
6. **Attribution model:** Data-driven (Google's recommended model)
7. **Count:** One per click (each click should count only one booking)

### Why Only One Primary Conversion

Google Ads Smart Bidding uses the **primary conversion** to train its machine learning model. If you import multiple primary conversions (e.g., Step 1 + Step 2 + Booking), the algorithm counts each funnel step as a separate conversion, **inflating conversion counts 3×** and destroying your CPA targets.

**Rule:** One business outcome = One primary conversion. All other events are secondary (observation-only) or not imported.

---

## Appendix: QA Checklist for Frontend Developer

After implementing all dataLayer pushes, verify:

- [ ] All 6 booking events fire in GTM Preview/Debug mode
- [ ] Events fire ONLY after successful validation (not on click)
- [ ] No duplicate events fire on rapid clicking
- [ ] All parameters contain correct values (not `undefined` or `null`)
- [ ] `booking_abandoned` fires on page close (test with Chrome DevTools → Network → `sendBeacon`)
- [ ] Events fire correctly on mobile devices (iOS Safari + Android Chrome)
- [ ] Events fire correctly when user navigates backward in the form
- [ ] `booking_failed` fires with correct `error_message` when API returns an error
- [ ] `timestamp` is in ISO 8601 format
- [ ] `session_id` persists across all steps within the same session

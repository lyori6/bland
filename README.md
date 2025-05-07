# Bland AI: Enhanced Support Engineer Recruiting Pathway 🚀

**Objective:** This project showcases a significantly enhanced version of Bland AI's "Support Engineer Recruiting Pathway." We've moved beyond baseline functionality to deliver a robust, user-friendly, and operationally transparent system (the "120% solution"), focusing on reliability, key integrations, and smart product decisions.

🔗 **Live Demo & Resources:**
*   **Loom Video Walkthrough:** `(Your Loom Video Link Here)`
*   **Live Bland Pathway (Enhanced):** [View Pathway](https://app.bland.ai/dashboard/convo-pathways?id=a06ee867-afbb-4736-b580-4125768c3899)
*   **Original Pathway (Reference):** [View Original](https://app.bland.ai/dashboard/convo-pathways?id=22e38484-e3b3-4870-99fe-3347f7a87537)
*   **Cloudflare Worker Code:** [View Worker Repo](https://github.com/lyori6/bland-cloudflare-clean)

---

## Key Enhancements

This pathway now delivers a superior experience for applicants and enhanced visibility for the recruiting team:

*   **For the Applicant:**
    *   Automated Calendar Invites: Applicants instantly receive an `.ics` email invite.
*   **For Operations:**
    *   Real-Time Slack Alerts: Immediate notifications for bookings and critical errors.
    *   Pathway Tags: Clear tagging (e.g., `booked_and_completed`, `error_lookup_failed`) for analytics.
*   **For Pathway Reliability:**
    *   Robust Error Handling & Retry Logic for API calls.
    *   Refined "Bland Tone" Prompts for clearer agent interactions and data formatting.
    *   Graceful Exits: Global nodes for user-initiated call endings and job queries.

---

## Pathway Journey: A High-Level Overview

The applicant's journey through the enhanced pathway:

1.  **Greeting & Availability (`Start: Greet & Check Availability`)**: Intro and availability check.
2.  **Name Collection (`Collect Full Name`)**: Captures and lowercases name.
3.  **User Info Retrieval (`Webhook: Get User Info`)**: Fetches applicant data, with retry and Slack alerts for failures.
4.  **Details Confirmation (`Confirm Customer Details`)**: Verifies job title and application date.
5.  **Salary Expectations (`Collect Salary Expectation`)**: Directs to transfer for high salary or proceeds.
6.  **Interview Scheduling (`Collect Interview Date & Time (PT)`)**: Gathers preferred slot (DD-MM-YYYY, HH:MM PT).
7.  **(Optional) Legacy API Booking (`Webhook: Book via Render API`)**: Integration point for existing systems, with failure alerts.
8.  **ICS Email Invite (`Webhook: Send Email & ICS Invite (CF Worker)`)**: Cloudflare Worker handles `.ics` generation (ApyHub) & email (SendGrid), plus internal Slack alerts.
9.  **Call Conclusion**: Defined end states for success or pathway errors.
10. **Global Handlers**: Manages user requests to end call or ask job-specific questions.

---

## Technical Implementation Highlights

### Cloudflare Worker: Integration Hub
A TypeScript Cloudflare Worker manages external communications:

*   **`/book-email` Endpoint:**
    *   **Input:** `email`, `interview_date` (DD-MM-YYYY), `interview_time` (HH:MM, PT).
    *   **Actions:** Parses PT to UTC, generates `.ics` (ApyHub, aware of `America/Los_Angeles`), sends email (SendGrid, displays PT), triggers internal Slack alerts.
*   **`/slack-event` Endpoint:**
    *   Internal route for non-blocking Slack messages (Block Kit) to configured channels.
*   **Key Tech:** `@sendgrid/mail`, `date-fns-tz`.
*   **Secrets:** `APY_TOKEN`, `SENDGRID_KEY`, `SLACK_BOOKINGS_URL`, `SLACK_ERRORS_URL`.

### Dual-Layer Slack Alerts
*   **Worker-Side:** For status of the email/ICS process.
*   **Pathway-Side:** For critical Bland pathway failures (e.g., API call failures within the pathway itself).

### Smart Prompting & Logic
*   **Bland Tone:** Consistent agent voice via global and refined node prompts.
*   **Structured Data:** Prompts guide LLM to format dates/times before worker ingestion.

---

## Testing & Validation

Five key unit test scenarios ensure pathway robustness:
1.  Happy Path Booking Success
2.  High Salary Transfer
3.  API Error Recovery (Get User Info)
4.  Webhook Failure (Book Appointment/Email)
5.  Caller Not Ready

---

## Assumptions & Key Decisions

*   **Date/Time Authority:** Bland pathway is prompted to format date/time inputs. Worker ensures PT display & correct UTC.
*   **Focus:** Prioritized backend integrations and pathway logic.
*   **Available Tools:** Leveraged standard Bland features for self-sufficient problem-solving.
*   **Latency:** Non-critical notifications use `ctx.waitUntil` for efficiency.

---

## Future Enhancements

*   Full HMAC security for worker endpoints.
*   Richer, interactive Slack messages.
*   Enhanced DST handling for worker input parsing (if required).
*   General "Oops, I misheard" global node.

---

This project delivers a polished, reliable, and operationally aware recruiting pathway.

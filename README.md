# ✨ Bland AI: Enhanced Support Engineer Recruiting Pathway ✨

**Objective:** This project showcases a significantly enhanced version of Bland AI's "Support Engineer Recruiting Pathway." We've moved beyond baseline functionality to deliver a robust, user-friendly, and operationally transparent system (the "120% solution"), focusing on reliability, key integrations, and smart product decisions.

🔗 **Live Demo & Resources:**
*   **Loom Video Walkthrough:** `(Link to your Loom Video will go here)`
*   **Live Bland Pathway (Enhanced):** [View Pathway](https://app.bland.ai/dashboard/convo-pathways?id=a06ee867-afbb-4736-b580-4125768c3899)
*   **Original Pathway (For Reference):** [View Original](https://app.bland.ai/dashboard/convo-pathways?id=22e38484-e3b3-4870-99fe-3347f7a87537)
*   **Cloudflare Worker Code:** [View Worker Repo](https://github.com/lyori6/bland-cloudflare-clean)

---

## 🚀 Key Enhancements at a Glance

This pathway now delivers a superior experience for applicants and enhanced visibility for the recruiting team:

*    Applicant Experience:
    *   🗓️ **Automated Calendar Invites:** Applicants instantly receive an `.ics` email invite upon booking.
*   ⚙️ Operational Excellence:
    *   📢 **Real-Time Slack Alerts:** Immediate notifications for successful bookings and critical pathway errors.
    *   🏷️ **Pathway Tags:** Clear tagging (e.g., `booked_and_completed`, `error_lookup_failed`) for streamlined analytics and log filtering.
*   💪 Pathway Reliability & Intelligence:
    *   🔄 **Robust Error Handling:** Includes retry logic for API calls and graceful failure management.
    *   🗣️ **Refined "Bland Tone" Prompts:** Clearer, more concise agent interactions and intelligent date/time formatting via prompt engineering.
    *   🚪 **Graceful Exits:** Global nodes handle user requests to end calls or ask job-specific questions.

---

## Pathway Journey: A High-Level Overview

*(This section aligns with your Loom video's walkthrough of the pathway)*

The applicant's journey is now more intuitive and reliable:

1.  👋 **Greeting & Availability (`Start: Greet & Check Availability`)**: Friendly intro, checks if applicant is free.
2.  📝 **Name Collection (`Collect Full Name`)**: Captures and lowercases name.
3.  🔍 **User Info Retrieval (`Webhook: Get User Info`)**: Fetches applicant data. Includes retry logic and Slack alerts (`Alert: User Info Lookup Failed (Slack)`) on persistent failure.
4.  ✅ **Details Confirmation (`Confirm Customer Details`)**: Verifies job title and application date.
5.  💰 **Salary Expectations (`Collect Salary Expectation`)**:
    *   Directs to `Transfer: High Salary (> $500k)` if expectations are high.
    *   Otherwise, proceeds to scheduling.
6.  🗓️ **Interview Scheduling (`Collect Interview Date & Time (PT)`)**: Gathers preferred slot, with prompts guiding structured DD-MM-YYYY & HH:MM (PT) input.
7.  🔗 **(Optional) Legacy API Booking (`Webhook: Book via Render API`)**: Can still integrate with existing booking systems. Failures trigger `Alert: API Booking Failed (Slack)`.
8.  📨 **ICS Email Invite (`Webhook: Send Email & ICS Invite (CF Worker)`)**: The core enhancement! Our Cloudflare Worker takes over:
    *   Generates `.ics` file (ApyHub).
    *   Sends confirmation email (SendGrid).
    *   Internally triggers Slack alerts for this specific booking/emailing process.
9.  🏁 **Call Conclusion**: Clear end states for success (`End Call Success: Invite Sent`) or pathway issues (`End Call - Pathway Error`).
10. 🌐 **Global Handlers**:
    *   `Global End Call: User Request`: For user-initiated call endings.
    *   `Answer Position Questions (KB)`: For job-related inquiries.

---

## 🛠️ Technical Deep Dive (The "How")

### Cloudflare Worker: The Integration Powerhouse ⚡
A dedicated Cloudflare Worker (TypeScript) manages external communications, keeping the Bland pathway lean.

*   **`/book-email` Endpoint:**
    *   **Input:** `email`, `interview_date` (DD-MM-YYYY), `interview_time` (HH:MM, PT).
    *   **Core Actions:**
        1.  Parses Pacific Time, converts to UTC.
        2.  **ApyHub:** Generates `.ics` (requests event in `America/Los_Angeles` timezone).
        3.  **SendGrid:** Emails `.ics` (displays time explicitly in PT using `date-fns-tz`).
        4.  **Internal Slack Trigger:** Non-blockingly calls `/slack-event` for its own status.
*   **`/slack-event` Endpoint:**
    *   Internal route for posting formatted Block Kit messages to Slack (`SLACK_BOOKINGS_URL` / `SLACK_ERRORS_URL` - currently unified).
*   **Key Tech:** `@sendgrid/mail`, `date-fns-tz`.
*   **Secrets Used:** `APY_TOKEN`, `SENDGRID_KEY`, `SLACK_BOOKINGS_URL`, `SLACK_ERRORS_URL`. (HMAC planned).

### Dual-Layer Slack Alerts 🛡️
*   **Worker-Side:** For success/failure of the email & ICS generation process itself (via `/slack-event`).
*   **Pathway-Side:** Direct Slack webhooks for critical failures *within the Bland pathway* (e.g., `/get-user-info` or legacy booking API calls failing).

### Smart Prompting & Logic 🧠
*   **Bland Tone:** Maintained via a detailed global prompt and refined node-specific instructions.
*   **Structured Data:** Variable extraction prompts for dates/times ensure they are formatted (DD-MM-YYYY, HH:MM) by Bland *before* hitting the worker.

---

## ✅ Testing & Validation

A suite of 5 unit tests covers key scenarios, ensuring pathway robustness:
1.  `happy_path_booking_success`
2.  `high_salary_transfer`
3.  `api_error_recovery_get_user_info` (conceptual)
4.  `webhook_failure_book_appointment`
5.  `caller_not_ready`

---

## 💡 Assumptions & Key Decisions

*   **Date/Time Authority:** Bland pathway is prompted to format date/time inputs before sending to the worker. Worker ensures PT display & correct UTC for backend.
*   **Focus:** Prioritized backend integrations and pathway logic over extensive live call simulations for this phase.
*   **Available Tools:** Leveraged standard Bland features, demonstrating self-sufficient problem-solving.
*   **Latency:** Non-critical notifications (like worker-side Slack alerts) use `ctx.waitUntil` to avoid impacting primary response times.

---

## 🔮 Future Enhancements

*   Implement full HMAC security for worker endpoints.
*   Richer, more interactive Slack messages using advanced Block Kit.
*   Robust PST/PDT handling for worker *input parsing* using `date-fns-tz` if needed.
*   General "Oops, I misheard" global node for enhanced conversation repair.

---

This project delivers a polished, reliable, and operationally aware recruiting pathway, ready to make a great impression!

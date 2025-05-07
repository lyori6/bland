# Bland AI Support Engineer Recruiting Pathway - Enhancement Showcase

**Objective:** To elevate the baseline "Support Engineer Recruiting Pathway" from a 100% functional state to a 120% showcase level, demonstrating enhanced reliability, valuable integrations (ICS email invites, Slack alerts), and thoughtful product management trade-offs, all while adhering to Bland AI's principles.

**(Link to your Loom Video will go here when ready)**
**(Link to the live Bland Pathway (if shareable) or JSON files in this repo)**

---

## 1. Project Overview & Enhancements

This project significantly enhances the Support Engineer Recruiting Pathway by:

*   **Improving Applicant Experience:** Automated ICS calendar invites via email ensure applicants never miss their interview.
*   **Increasing Operational Visibility:** Real-time Slack notifications for successful bookings and critical pathway errors.
*   **Enhancing Reliability:**
    *   Robust error handling for external API calls.
    *   Clear retry logic for candidate information lookup.
    *   Graceful handling of user-initiated call endings.
*   **Streamlining Pathway Logic:** Clearer node naming, refined prompts, and pathway tags for better analytics.
*   **Adhering to Bland Principles:** Maintaining a "Bland Tone," focusing on low-latency interactions, and making pragmatic PM trade-offs.

---

## 2. The Enhanced Pathway Flow (High-Level for Loom)

*(This section mirrors what you'll show in the Loom video at a high level. You can reference specific node names from your "new renamed pathway.json" here. No complex diagram needed, just a narrative flow.)*

The enhanced pathway guides the applicant through the following key stages:

1.  **Greeting & Availability (Node: `Start: Greet & Check Availability`)**:
    *   Agent introduces self and checks if the applicant is free.
    *   Handles "not available" gracefully (Tag: `ended_user_unavailable`).
2.  **Name Collection (Node: `Collect Full Name`)**:
    *   Captures first and last name, stored in lowercase.
3.  **User Info Retrieval (Node: `Webhook: Get User Info`)**:
    *   POSTs name to `/get-user-info` (Render API).
    *   **Retry Logic:** If lookup fails (e.g., name not found), routes to `Retry: Collect Full Name` and `Webhook: Retry Get User Info`.
    *   **Failure Alert:** If retry also fails, triggers `Alert: User Info Lookup Failed (Slack)` (Tag: `error_lookup_failed`) before ending the call.
4.  **Application Details Confirmation (Node: `Confirm Customer Details`)**:
    *   Confirms `job_title` and `date_applied` with the applicant.
5.  **Salary Expectation (Node: `Collect Salary Expectation`)**:
    *   Collects and confirms salary.
    *   **Branching Logic:**
        *   If > $500k, routes to `Transfer: High Salary (> $500k)` (Tag: `transfer_high_salary`).
        *   If <= $500k, proceeds to scheduling.
6.  **Interview Scheduling (Node: `Collect Interview Date & Time (PT)`)**:
    *   Collects preferred date/time (PT), formatted as DD-MM-YYYY and HH:MM by Bland's extraction.
7.  **Legacy Booking (Optional - Node: `Webhook: Book via Render API`)**:
    *   Pathway can still call the original Render API for booking.
    *   Failure triggers `Alert: API Booking Failed (Slack)` (Tag: `error_booking_failed`).
8.  **ICS Email Invite & Final Booking (Node: `Webhook: Send Email & ICS Invite (CF Worker)`)**:
    *   Calls our Cloudflare Worker (`/book-email`).
    *   Worker handles ApyHub ICS generation & SendGrid email.
    *   Worker internally triggers Slack success/failure alerts for *its own process*.
9.  **Call Conclusion:**
    *   Successful booking: `End Call Success: Invite Sent` (Tag: `booked_and_completed`).
    *   Pathway errors: `End Call - Pathway Error` (Tag: `ended_with_error`).
10. **Global Nodes:**
    *   `Global End Call: User Request`: Handles user requests to end call (Tag: `Call Ended - User Requested`).
    *   `Answer Position Questions (KB)`: Handles job-specific queries using the knowledge base.

---

## 3. Key Technical Implementations

### 3.1. Cloudflare Worker: The Integration Hub

A Cloudflare Worker (TypeScript) was developed to handle external integrations seamlessly, keeping the Bland pathway focused on conversational logic.

*   **Endpoint: `/book-email`**
    *   Receives `email`, `interview_date` (DD-MM-YYYY), `interview_time` (HH:MM, assumed PT).
    *   **Parses & Converts Time:** Converts input Pacific Time to UTC ISO 8601, robustly handling date parsing (future: enhance with `date-fns-tz` for full DST accuracy if needed for input).
    *   **ApyHub Integration:** Calls ApyHub API to generate an `.ics` calendar file. The ApyHub request specifies `time_zone: 'America/Los_Angeles'` to ensure the event is correctly defined for PT.
    *   **SendGrid Integration:** Sends an email to the applicant with the `.ics` file attached. Email text displays time formatted explicitly in Pacific Time (using `date-fns-tz`). Handles CC logic carefully to avoid duplicate recipient errors.
    *   **Internal Slack Trigger:** Non-blockingly calls its own `/slack-event` endpoint for success/failure of this email process.
*   **Endpoint: `/slack-event`**
    *   Receives an internal payload (e.g., `{type: 'booking_success', data: {...}}`).
    *   Formats a Slack message using Block Kit.
    *   Posts to the appropriate Slack channel (`SLACK_BOOKINGS_URL` or `SLACK_ERRORS_URL` - currently configured to a single alerts channel).
*   **Key Libraries Used:** `@sendgrid/mail`, `date-fns-tz` (for display formatting).
*   **Environment Variables (Secrets):**
    *   `APY_TOKEN`: For ApyHub.
    *   `SENDGRID_KEY`: For SendGrid.
    *   `SLACK_BOOKINGS_URL`: For Slack success notifications.
    *   `SLACK_ERRORS_URL`: For Slack error notifications.
    *   `HMAC_SECRET`: (Provisioned for future HMAC request verification from Bland).

### 3.2. Slack Integration - Dual Approach

*   **Worker-Side Alerts:** The Cloudflare worker's `/book-email` route triggers internal calls to `/slack-event` for real-time status of the email/ICS generation process (success or failure). This uses `ctx.waitUntil()` for non-blocking execution.
*   **Pathway-Side Alerts:** For critical pathway failures *before* the Cloudflare worker is called (e.g., if the initial `/get-user-info` Render API fails or the legacy booking API fails), the Bland pathway has dedicated webhook nodes (`Alert: User Info Lookup Failed (Slack)`, `Alert: API Booking Failed (Slack)`) that directly post detailed error messages to Slack. This ensures visibility even if the CF Worker isn't reached or its specific task isn't initiated.

### 3.3. Prompt Engineering & Pathway Logic

*   **Refined Node Prompts:** Prompts for key nodes like "Start: Greet & Check Availability" and "Collect Full Name" were updated for clarity, Bland Tone, and to guide the LLM effectively.
*   **Structured Variable Extraction:** Descriptions for `chosen_date` and `chosen_time` variables in the "Collect Interview Date & Time (PT)" node explicitly instruct the LLM to format outputs as DD-MM-YYYY and HH:MM.
*   **Global Prompts:** Utilized the pathway's global prompt for consistent tone and behavior. Added a "Global End Call: User Request" node for graceful exits.
*   **Pathway Tags:** Implemented for analytics and operational insight (e.g., `booked_and_completed`, `error_lookup_failed`, `transfer_high_salary`, `ended_user_unavailable`, `Call Ended - User Requested`).

---

## 4. Testing Strategy

Five key unit test scenarios were defined and (conceptually, or if UI allows, physically) configured within Bland's Pathway › Tests tab:

1.  **`happy_path_booking_success`**: Standard flow resulting in successful booking and notifications.
2.  **`high_salary_transfer`**: Applicant salary > $500k, routes to transfer.
3.  **`api_error_recovery_get_user_info`**: Simulates `/get-user-info` failing once then succeeding on retry.
4.  **`webhook_failure_book_appointment`**: Simulates the final booking/email worker call failing, triggering failure alerts.
5.  **`caller_not_ready`**: Applicant is not available at the start.

---

## 5. Assumptions & Key Decisions

*   **Date/Time Formatting:** Assumed Bland AI pathway's variable extraction is capable of formatting date/time into DD-MM-YYYY and HH:MM (Pacific Time) based on detailed prompts before sending to the worker. The worker then handles UTC conversion and PT display.
*   **Case Sensitivity (Lowercase):** Name collection prompts instruct for lowercase storage. Assumed downstream systems (APIs, email) are generally case-insensitive for identifiers like email addresses, so explicit worker-side lowercasing for *all* fields was not prioritized over core logic.
*   **Focus of Assignment:** Prioritized backend integrations (API calls, ICS, email, Slack), pathway logic, error handling, and prompt engineering. Did not extend to live multi-turn phone call simulations beyond Bland's "Test Pathway" UI.
*   **Bland Feature Scope:** Utilized features available within the standard Bland AI account. Advanced features like "Send SMS" or "Custom Code" nodes were acknowledged but not used, focusing on self-sufficient problem-solving with provided tools.
*   **Simplicity for Slack Webhooks:** Used Slack's "Incoming Webhooks" for directness. Configured to post all alerts to a single channel for this iteration, though the worker logic supports separate channels.

---

## 6. Future Considerations

*   **Full HMAC Security:** Implement robust HMAC signature verification on all Cloudflare Worker endpoints receiving requests from Bland.
*   **Richer Slack Messages:** Further enhance Slack messages using more advanced Block Kit layouts for better readability and potential action buttons.
*   **Enhanced DST Handling for Input:** If Bland's formatting isn't sufficient, integrate `date-fns-tz` into the worker's input parsing in `index.ts` for fully robust Pacific Time (PST/PDT) to UTC conversion.
*   **Global 'Oops' Node:** Implement a general misunderstanding/repeat request handler in the pathway.
*   **Comprehensive Unit Test Mocking:** Explore deeper mocking capabilities for webhook responses within Bland's testing framework if available.

---

This enhanced pathway provides a reliable, user-friendly, and operationally transparent solution for the initial stages of Support Engineer recruiting.

**(Link to JSON files for the pathway if included in the repo)**
*   [Original Unsolved Pathway](./pathway_json/unsolved%20-%20pathway%20(2).json)
*   [Current Enhanced Pathway](./pathway_json/new%20renamed%20pathway.json)

**(Link to Cloudflare Worker Code if included in the repo)**
*   [Cloudflare Worker Code](./cloudflare_worker_code/)

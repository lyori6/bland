# Enhanced Support Engineer Recruiting Pathway ✨

**Objective:** This project upgrades Bland AI's "Support Engineer Recruiting Pathway" I aimed for a solution that's not just functional but also reliable, user-friendly, and gives clear operational insights—the "120% solution"

🔗 **Live Demo & Resources**
*   **Project Documentation & Overview:** [https://lyori6.github.io/bland/](https://lyori6.github.io/bland/) (This README)
*   **Loom Video Walkthrough:** `(Your Loom Video Link Here)`
*   **Live Bland Pathway (Enhanced):** [View Pathway](https://app.bland.ai/dashboard/convo-pathways?id=a06ee867-afbb-4736-b580-4125768c3899)
*   **Original Pathway (Reference):** [View Original](https://app.bland.ai/dashboard/convo-pathways?id=22e38484-e3b3-4870-99fe-3347f7a87537)
*   **Cloudflare Worker Code:** [View Worker Repo](https://github.com/lyori6/bland-cloudflare-clean)

---

## 🚀 Key Enhancements

This pathway now offers a better experience and clearer operational tracking:

*   **For the Applicant**
    *   Automated Calendar Invites: Applicants instantly get an `.ics` email invite
*   **For Operations**
    *   Real-Time Slack Alerts: Quick updates on bookings and critical errors
    *   Pathway Tags: Easy filtering of call logs (eg, `booked_and_completed`, `error_lookup_failed`)
*   **For Pathway Performance**
    *   Improved Error Handling & API Retry Logic
    *   Clear "Bland Tone" Prompts for agent interactions and data formatting
    *   Flexible Data Collection: Handles cases where applicants decline to share salary, proceeding gracefully
    *   Smooth Call Endings: Global nodes manage user requests to end calls or ask job questions

---

## 🗺️ Pathway Journey: A Quick Look

The applicant's experience is now more direct and reliable:

1.  **Greeting & Availability (`Start: Greet & Check Availability`)**: Agent intro and availability check
2.  **Name Collection (`Collect Full Name`)**: Captures and lowercases name
3.  **User Info Retrieval (`Webhook: Get User Info`)**: Fetches applicant data Includes retries and Slack alerts for persistent lookup failures
4.  **Details Confirmation (`Confirm Customer Details`)**: Verifies job title and application date
5.  **Salary Expectations (`Collect Salary Expectation`)**: If > $500k, transfers call If applicant declines to state, the pathway politely acknowledges and proceeds, assuming a standard salary range
6.  **Interview Scheduling (`Collect Interview Date & Time (PT)`)**: Gathers preferred slot (DD-MM-YYYY, HH:MM PT)
7.  **(Optional) Legacy API Booking (`Webhook: Book via Render API`)**: Connects to existing systems, with failure alerts Failures trigger a specific Slack alert, but the process *continues* to attempt the Email/ICS invite
8.  **ICS Email Invite (`Webhook: Send Email & ICS Invite (CF Worker)`)**: Cloudflare Worker handles `.ics` generation (ApyHub) & email (SendGrid), plus internal Slack alerts
9.  **Call Conclusion**: Defined end states for success or pathway issues
10. **Global Handlers**: Manages user requests to end calls or ask job-specific questions

---

## 🛠️ Technical Setup

### Cloudflare Worker: Integration Helper
A TypeScript Cloudflare Worker manages key external tasks:

*   **`/book-email` Endpoint**
    *   **Input:** `email`, `interview_date` (DD-MM-YYYY), `interview_time` (HH:MM, PT)
    *   **Actions:** Parses PT to UTC, generates `.ics` (ApyHub, specifies event in `America/Los_Angeles`), sends email (SendGrid, displays PT via `date-fns-tz`), triggers internal Slack alerts
*   **`/slack-event` Endpoint**
    *   Internal route for sending Slack messages (Block Kit)
*   **Key Tech:** `@sendgrid/mail`, `date-fns-tz`
*   **Secrets:** `APY_TOKEN`, `SENDGRID_KEY`, `SLACK_BOOKINGS_URL`, `SLACK_ERRORS_URL`

### Slack Alerts
*   **Worker-Side:** For success/failure of the email & ICS process
*   **Pathway-Side:** For critical Bland pathway failures (eg, user info lookup failure)

### Smart Prompting
*   **Bland Tone:** Consistent agent voice via global and node-specific prompts
*   **Data Formatting:** Prompts the LLM to format dates/times correctly *within Bland*

---

## ✅ Testing Our Work

I defined these key test scenarios in Bland's Pathway Test tab:

1.  **Happy Path:** Booking confirmed, calendar invite sent
2.  **High Salary:** Salary > $500k, call transferred to recruiter
3.  **Candidate Lookup Fail:** `/get-user-info` fails, Slack alert sent, call ends gracefully
4.  **Lookup Retry Success:** `/get-user-info` fails once, succeeds on retry, flow continues
5.  **User Requested End Call:** Global node handles user's request to hang up
6.  **Caller Not Free:** Call ends politely if user is unavailable at start

---

## 💡 Our Approach & Decisions

*   **Date/Time (Simplified for MVP):** For this iteration, the Bland pathway is prompted to format date/time inputs as DD-MM-YYYY and HH:MM, assumed to be Pacific Time (PT) The Cloudflare Worker converts this to UTC for backend systems, ensuring email displays and calendar invites are PT-aware **Constraint:** This simplified model assumes bookings don't occur late enough in PT to shift the UTC date to the next day in a way that causes discrepancies with user expectations before full timezone conversion logic is implemented This is an area for future refinement to robustly handle various user timezones and Daylight Saving Time intricacies more directly
*   **Salary Collection Flexibility:** If an applicant declines to provide salary information, the pathway acknowledges this and proceeds, defaulting to the standard salary path, ensuring the conversation continues smoothly
*   **Project Focus:** Prioritized backend integrations and pathway logic, per assignment
*   **Tooling:** Used standard Bland features for self-sufficient problem-solving
#*   **Efficiency:** Non-critical notifications use `ctx.waitUntil` to keep main responses fast

---

## 🔮 Next Steps & Future Ideas

*   **Enhanced Security:** Add request verification (eg, HMAC signatures) to worker endpoints
*   **Advanced Timezone Management:** Implement more comprehensive timezone handling in the Cloudflare Worker to accurately parse and manage interview times from users across *various* input timezones, including full Daylight Saving Time (DST) logic This would address the current constraint around evening bookings and potential date shifts
*   **Deeper Call Analysis:** More listening to live call patterns to further refine pathway flow and agent responses
*   **Richer Slack Messages:** Use more advanced Slack Block Kit features for more interactive or detailed alerts

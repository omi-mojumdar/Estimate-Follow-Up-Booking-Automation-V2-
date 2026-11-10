# Estimate Follow-Up & Booking Automation
An n8n automation system for residential cleaning businesses that manages the estimate lifecycle—from a customer submitting a cleaning estimate request to follow-ups, customer replies, booking status updates, error notifications, and live reporting.
Portfolio project: This README describes the six-workflow version (Workflows 2.1–2.6). The system is demonstrated with the CleanFlow example business. A portfolio demonstration is not evidence of a live client deployment.

Overview
Cleaning businesses can lose potential jobs when estimates are not tracked, customers are not followed up with, replies are missed, or booking statuses are not updated consistently.

This project connects six n8n workflows to help manage that process:
- Capture estimate requests submitted through a website form.
- Generate a unique Estimate ID and calculate the estimate amount in a JavaScript Code node.
- Store estimate details and status information in Google Sheets.
- Send an estimate confirmation email and a capped sequence of follow-up emails through Gmail.
- Detect replies from known customers and notify the business owner with action buttons.
- Update booking status, log email failures, and provide a live HTML dashboard.
  
System Architecture
<img width="2614" height="3016" alt="mermaid-diagram" src="https://github.com/user-attachments/assets/4289072d-ca14-4dc2-ab60-4f96c04fdefc" />

⚙️ Workflows — How It Works

2.1 — Customer Estimate Submission & Confirmation
Trigger: A customer submits the CleanFlow website estimate form, which sends a POST request to an n8n webhook.

The form collects:
- Customer name and email
- Service type: Standard, Deep, Move-In, or Post-Construction
- Number of bedrooms and bathrooms
- ZIP Code
The workflow then:
1. Generates a unique Estimate ID (for example, EST-1794234565969) and timestamps using a Code node.
2. Calculates the estimate amount in the Code node using the project's configured pricing logic.
3. Saves the estimate record in the Google Sheets Estimates tab.
4. Sets the initial status to Pending and the follow-up count to 0.
5. Sends a formatted HTML confirmation email through Gmail. The email displays the customer's estimate details, calculated amount, and a Confirm My Booking button.
The project overview uses $190 as an example estimate amount. The specific pricing rates and formula are configured in the workflow and are not reproduced in this README.

2.2 — Daily Follow-Up Checker
Trigger: An n8n Schedule trigger runs every morning at 9:00 AM.

The workflow:
1. Reads estimate records from Google Sheets.
2. Filters for estimates with Pending status that are due for follow-up.
3. Checks whether the follow-up count is below three.
4. Sends a follow-up email through Gmail when the estimate is eligible.
5. Calculates the next follow-up date.
6. Updates the follow-up count and follow-up date in Google Sheets.
7. When the three-follow-up limit has been reached, marks the estimate as Follow-up Complete and stops the sequence.
This creates a capped, three-touch follow-up sequence rather than sending unlimited reminders.

2.3 — Customer Reply Detection & Owner Notification
Trigger: A Gmail Trigger detects an incoming customer reply.

The workflow:
1. Looks up the sender's email address in Google Sheets.
2. Matches the reply to the corresponding Estimate ID using a Code node.
3. Uses an IF node to determine whether the sender is a known customer.
4. For a known customer, updates the estimate status to Reply Received.
5. Sends the business owner a Customer Replied — Action Needed email containing the customer's message and three action buttons:
   - Mark as Booked
   - Reply to Customer
   - Mark as Opted Out
6. If the sender is unknown, the workflow silently ends.
The owner can choose an action from the notification email. The documented booking action is handled by Workflow 2.4.

2.4 — Booking Confirmation Handler
Trigger: A GET webhook.

The webhook is triggered when the customer clicks Confirm My Booking in the estimate email or when the owner clicks Mark as Booked in the reply notification.

The workflow:
1. Receives the booking update.
2. Updates the corresponding estimate's status to Booked in Google Sheets.
3. Returns a booking confirmation page to the browser using a Webhook Response node.
This workflow updates booking status and displays a confirmation page. A calendar-based appointment scheduler or cleaner-assignment system is not part of the documented scope.

2.5 — Automatic Email Error Handling
Trigger: An n8n Error Trigger catches a qualifying failure from a Gmail send node in the project.

The workflow:
1. Captures the email failure.
2. Logs it in the Google Sheets Errors tab.
3. Records the timestamp, Estimate ID, error message, and workflow name.
4. Sends an error notification to the business owner through Gmail.
The project overview includes an example error captured during testing:
The 'Column to Match On' parameter is required

2.6 — Live Business Dashboard
Trigger: The business owner clicks the Open Dashboard hyperlink in the Google Sheets Dashboard tab.

The workflow:
1. Fetches estimate records from Google Sheets.
2. Uses a Code node to build a JSON summary and count estimates by status.
3. Returns a live HTML dashboard to the browser.
The dashboard displays six summary cards:
- Total Estimates
- Pending
- Booked
- Opted Out
- Customer Replied
- Follow-up Complete
It also displays an estimates table with status highlighting (Pending = orange; Booked = green) and a Refresh button to reload the data.
Google Sheets Structure
The project overview describes three tabs:
Tab	Purpose	Documented fields or contents
Estimates	Stores estimate records and follow-up state	Estimate ID, customer name, email, service type, bedrooms, bathrooms, ZIP Code, estimated amount, date, status, follow-up count, and follow-up date information
Errors	Stores email failure records	Timestamp, Estimate ID, error message, workflow name
Dashboard	Gives the owner access to the live dashboard	Open Dashboard hyperlink to the dashboard webhook

Main Status Values
The documented estimate statuses include:
Status	Meaning
Pending	Estimate has been created and is awaiting the customer's decision.
Reply Received	A reply from a known customer has been detected.
Booked	The booking confirmation handler has updated the estimate.
Follow-up Complete	The estimate has reached the three-follow-up limit.
Opted Out	An opt-out action is presented to the owner in the reply notification.

The dashboard's Customer Replied card is a reporting label; the reply workflow describes the corresponding estimate status as Reply Received.
Technology Stack
- n8n — Workflow orchestration, webhooks, schedule trigger, Gmail trigger, Error Trigger, and webhook responses.
- JavaScript — Code-node logic for Estimate ID generation, estimate price calculation, timestamps, matching, and dashboard summary generation.
- Google Sheets — Estimate records, error logs, and dashboard entry point.
- Gmail — Customer confirmation emails, follow-up emails, owner notifications, and error alerts.
- CleanFlow website form — Customer-facing estimate request form.

- 📸 Screenshots
<img width="1682" height="796" alt="2c5b3e35-a88a-4e73-bc85-12b03240fe5e" src="https://github.com/user-attachments/assets/2d829c2d-53eb-4ff2-9d29-4f77fcea4f91" />
<img width="1919" height="805" alt="7133ebf1-e5f8-40ef-845c-a7439027502b" src="https://github.com/user-attachments/assets/f1ee3375-c7b2-4f7b-9cd7-526c0090e7e1" />
<img width="1919" height="811" alt="66032ff0-5b8e-4479-9454-65c0cf0d023e" src="https://github.com/user-attachments/assets/944fe893-12fd-402c-9e0a-3edb6aeafdb9" />
<img width="1919" height="815" alt="862f9f70-16fb-4c7d-a067-ccd34f607f57" src="https://github.com/user-attachments/assets/2df9b3e5-29db-4275-a3b3-aab791648f4b" />
<img width="1919" height="822" alt="c785f9fb-3d32-4660-9e79-c987d3d1b0e4" src="https://github.com/user-attachments/assets/5476fd0b-2d8d-4bff-a9dc-d6ff6a60e346" />
<img width="1919" height="810" alt="e3e00bb9-fd1f-450f-9891-ad04f73eb860" src="https://github.com/user-attachments/assets/9c14d8b0-8355-4573-9d01-fa3135f4da5c" />
<img width="1919" height="811" alt="673ebdf5-b1a7-4d5f-9b93-f4e0e42e9996" src="https://github.com/user-attachments/assets/537b9568-9a7f-4a43-b6e2-52309ec54a17" />

- Video Link: (https://drive.google.com/file/d/1QdisLA-jqIdJ7CeSUXFiUd9aUuxTRUj0/view?usp=drive_link)
  
- Key Features
Automated 3-touch follow-up sequence — capped to avoid over-emailing
Action buttons inside notification emails — owner can Mark as Booked, Reply, or Opt Out directly from inbox
Live webhook-powered dashboard — no manual sheet refresh needed
Error monitoring — every failed email is logged and owner is alerted immediately
Fully modular — each workflow is independent and can be extended or replaced
🎯 Target Use Case

Built for US residential cleaning companies looking to:

Automate their estimate-to-booking pipeline
Reduce manual follow-up work
Get a simple CRM without paying for expensive software
Track leads and conversions in one place
👤 Author

Omi Mojumdar AI Automation Developer 📧 omimojumdar88@gmail.com

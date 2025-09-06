# Automated Employee Time-Off Management System

A robust, no-code solution designed to automate the entire lifecycle of employee leave requests, from submission to balance tracking.

`#Airtable` `#Zapier` `#NoCode` `#Automation` `#HR` `#Workflow`

## 📋 Project Overview

This project replaces inefficient, manual leave tracking (via spreadsheets and emails) with a fully automated workflow. Using Airtable as a smart database and Zapier and Airtable Automations as the logic engines, this system provides a centralized, error-proof, and transparent process for managing employee Paid Time Off (PTO).

The system handles request submissions, validates them against available balances, routes valid requests to managers for approval, and keeps all stakeholders notified in real-time.

## ✨ Key Features

-   **Centralized Request Management:** All leave requests are submitted and tracked in one place.
-   **Real-Time PTO Balance Calculation:** Employee PTO balances are automatically updated upon approval of a request.
-   **Automated Annual Reset:** On January 1st of each year, all employee PTO balances automatically reset to the annual allowance with zero manual intervention.
-   **Intelligent Request Validation:** The system automatically checks if a request exceeds the employee's available balance. Invalid requests are instantly rejected and the employee is notified.
-   **Seamless Notification Workflow:** Managers are notified of valid pending requests, and employees are notified of the final decision (approved or rejected).
-   **Professional Email Sender:** All automated emails are sent from a custom email address (via Gmail/Outlook integration) instead of a generic no-reply address.

## 🛠️ Tech Stack

-   **Database & Backend Logic:** [Airtable](https://www.airtable.com/)
-   **Workflow Automation & Integration:** [Zapier](https://zapier.com/) & Airtable Automations

## ⚙️ System Architecture & Workflow
Employee Submits Leave Request (via Airtable Form)
|
'-> Record is created in Airtable's 'Leave Requests' table
|
'-> Is the request VALID? (Days Requested <= PTO Balance)
|
+--> YES: The record appears in the "Valid Pending Requests" view.
|    |
|    '-> ZAPIER TRIGGER fires:
|         |
|         1. Finds the employee's manager details.
|         2. Sends an email notification to the manager with an approval link.
|
+--> NO: The request is INVALID.
|
'-> AIRTABLE AUTOMATION fires:
|
1. Updates the request status to "Rejected".
2. Sends an email to the EMPLOYEE explaining why.
3. Sends an FYI email to the MANAGER.

## 🚀 Step-by-Step Setup Guide

### Part 1: Airtable Base Configuration

1.  **Create a New Base:** Start with a new, empty base in Airtable named `Employee Time-Off Management`.
2.  **Create the `Employees` Table:** This table holds all employee data. Set it up with the fields listed in the configuration details below. **Crucially, ensure the `Employee Name` field is the Primary Field (the first column).**
3.  **Create the `Leave Requests` Table:** This table will capture all submissions. Set it up with the fields listed in the configuration details. This includes several "helper" fields (`Is Valid Request?`, `Rollup`, `Lookups`, etc.) that are essential for the automations to work.
4.  **Create the Submission Form:** In the `Leave Requests` table, create a new **Form** view. Drag and drop the necessary fields (`Employee`, `Start Date`, `End Date`, `Notes`) for the user to fill out. Make the key fields required.
5.  **Create the "Valid Pending Requests" View:** In the `Leave Requests` table, create a new **Grid** view named `Valid Pending Requests`. This view is critical for the Zapier trigger. Add the filters detailed below.

### Part 2: Airtable Automations Configuration

You will build two main automations directly within Airtable.

1.  **Automation #1: "Flag & Reject Invalid PTO Request"**
    -   **Trigger:** `When a record is created` in the `Leave Requests` table.
    -   **Logic:** Use a **Conditional Logic** block that only runs IF the `Employee PTO Balance Rollup` is less than the `Number of Days` requested.
    -   **Actions (inside the "If" block):**
        1.  **Update Record:** Change the request's `Status` to `Rejected` and add an automatic `Manager Comment`.
        2.  **Send Email (Gmail/Outlook):** Notify the **employee** that their request was automatically rejected and why.
        3.  **Send Email (Gmail/Outlook):** Notify the **manager** that an invalid request was submitted and handled.

2.  **Automation #2: "Notify Employee of Approval/Rejection"**
    -   This can be one or two automations.
    -   **Trigger:** `When a record matches conditions` (e.g., `Status` is `Approved`).
    -   **Action:** **Send Email (Gmail/Outlook)** to the employee confirming the status of their request.

### Part 3: Zapier Workflow Configuration

Create a new Zap to handle notifications for valid requests.

1.  **Trigger: "New Record in a View" in Airtable**
    -   Connect your Airtable account.
    -   Select your base and the `Leave Requests` table.
    -   **Crucially, set the "Limit to View" option to your `Valid Pending Requests` view.** This ensures the Zap only runs for valid requests.

2.  **Action: "Find Record" in Airtable**
    -   This step is necessary to get the manager's details.
    -   Configure it to search the `Employees` table using the employee's name from the trigger step.

3.  **Action: "Send Email" in Gmail/Outlook**
    -   **To:** Use the `Manager Email` from the "Find Record" step.
    - **Body:** Include all relevant details from the trigger step.
    - **Pro-Tip:** Add the `Airtable Record URL` from the trigger data into the email body to give managers a direct link to the record for quick approval.

---

## 🔧 Configuration Details

<details>
<summary><strong>Click to view `Employees` Table Configuration</strong></summary>

| Field Name | Field Type | Notes |
| :--- | :--- | :--- |
| **`Employee Name`** | Single line text | **(Primary Field 🔑)** |
| **`Employee Email`** | Email | |
| **`Manager Name`** | Single line text | |
| **`Manager Email`** | Email | |
| **`Leave Requests`** | Link to another record | Links to the `Leave Requests` table (Allow multiple records) |
| **`Approved Days (Current Year)`**| Rollup | Rolls up `Number of Days` from `Leave Requests` with `SUM(values)` where `Is Current Year?` is `1` AND `Status` is `Approved` |
| **`PTO Balance`** | Formula | Formula: `22 - {Approved Days (Current Year)}` |

</details>

<br>

<details>
<summary><strong>Click to view `Leave Requests` Table Configuration</strong></summary>

| Field Name | Field Type | Notes |
| :--- | :--- | :--- |
| **`Request ID`** | Autonumber | **(Primary Field 🔑)** |
| **`Employee`** | Link to another record | Links to `Employees` table (Allow multiple records: **OFF**) |
| **`Start Date`** | Date | |
| **`End Date`** | Date | |
| **`Number of Days`** | Formula | Formula: `IF(AND({Start Date}, {End Date}), WORKDAY_DIFF({Start Date}, {End Date}) + 1, 0)` |
| **`Request Year`** | Formula | Formula: `IF({Start Date}, YEAR({Start Date}), BLANK())` |
| **`Notes`** | Long Text | **Optional notes from the employee.** |
| **`Submission Date`** | Created time | **Automatically records when the request was submitted.** |
| **`Status`** | Single select | Options: `Pending`, `Approved`, `Rejected` |
| **`Manager's Comment`** | Long Text | **Optional comments from the manager during approval/rejection.** |
| **`Is Current Year?`** | Formula | **For annual reset.** Formula: `YEAR({Start Date}) = YEAR(TODAY())` |
| **`Zapier Name`** | Formula | **For name lookup in zapier.** Formula: `{Employees}` |
| **`Employee PTO Balance Rollup`** | Rollup | **For validation.** Rolls up `PTO Balance` from `Employees` with `SUM(values)` |
| **`Is Valid Request?`** | Formula | **For filtering.** Formula: `IF({Employee PTO Balance Rollup} >= {Number of Days}, 1, 0)` |
| **`Employee Email Lookup`** | Lookup | **For notifications.** Looks up `Employee Email` from the `Employee` link. |
| **`Manager Email Lookup`** | Lookup | **For notifications.** Looks up `Manager Email` from the `Employee` link. |

</details>

<br>

<details>
<summary><strong>Click to view "Valid Pending Requests" View Filters</strong></summary>

This view in the `Leave Requests` table should have the following filters:
- `Where` **`Status`** `is` **`Pending`**
- `AND`
- `Where` **`Is Valid Request?`** `is` **`1`**

</details>

---

## 📈 Future Improvements

-   **Calendar Integration:** The Zapier workflow could be extended to automatically create an event on a shared company calendar when a request is approved.
-   **Different Leave Types:** The system could be enhanced by adding a "Leave Type" field (e.g., Sick Leave, Unpaid Leave) and creating separate balance calculation logic for each.
-   **Slack Notifications:** A new Zapier action could be added to notify managers via Slack in addition to email for faster response times.

---

## ✍️ Author

-   **Ayomide Abiola**
-   [LinkedIn Profile](https://www.linkedin.com/in/ayomide-abiola-77381a262/)

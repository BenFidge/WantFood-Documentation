# Monday.com Setup and Bug Reporting Guide

## Purpose

This guide explains how to set up your Monday.com board for the WantFood UAT project and how to use it to report, track, and manage bugs throughout the testing process.

---

## Why Monday.com?

Monday.com provides:

- **Centralized bug tracking**—all issues in one place
- **Visual status tracking**—see what's pending, in progress, fixed, or verified
- **Team collaboration**—developers, testers, and project managers work together
- **Audit trail**—complete history of each bug from discovery to resolution

---

## Step 1: Create Your UAT Board

### 1.1 Create a New Board

1. Log in to Monday.com with the credentials provided by your project manager
2. Click **"Create board"** in the top right
3. Choose **"Blank board"** (do not use a template)
4. Name the board: **"WantFood UAT — [Your Name]"** (e.g., "WantFood UAT — Jane Smith")
5. Click **"Create board"**

### 1.2 Set Board Permissions

1. Click the three-dot menu in the top right of the board
2. Select **"Board settings"** → **"Permissions"**
3. Ensure the following people have access:
	- Your project manager (Owner)
	- Development team lead (Edit)
	- All UAT testers (Edit)
4. Save changes

---

## Step 2: Set Up Groups (Test Areas)

Groups organize bugs by the area of the platform they affect. Create the following groups:

### Recommended Group Structure

| Group Name | Purpose |
|------------|---------|
| **System Admin** | Bugs found in the System Admin portal |
| **Vendor Admin** | Bugs found in the Vendor Admin portal |
| **Driver Portal** | Bugs found in the Driver Portal |
| **Customer Front-end** | Bugs found in the Customer-facing website |
| **Order Saga** | Bugs affecting the end-to-end order flow across multiple portals |
| **Automation & Jobs** | Bugs related to background processes or scheduled jobs |
| **Cross-Portal Impact** | Bugs found in TC-XP-* tests covering behaviour that spans multiple portals |
| **Permissions & Accounts** | Bugs found in TC-AC-* tests covering role-based access and account management |
| **Resilience & Edge Cases** | Bugs found in TC-RE-* tests covering error handling, timeouts, and boundary conditions |
| **General / Cross-Cutting** | Bugs that don't fit into a specific portal (e.g., authentication, performance) |

### How to Create Groups

1. Click **"+ Add Group"** at the bottom of the board
2. Enter the group name (e.g., "System Admin")
3. Repeat for all groups listed above

---

## Step 3: Configure Columns (Bug Item Template)

Each bug (called an "item" in Monday.com) should capture the following information. Set up these columns:

### Required Columns

| Column Name | Column Type | Purpose |
|-------------|-------------|---------|
| **Bug Title** | Text | Short, clear summary of the bug |
| **Steps to Reproduce** | Long Text | Numbered list of steps to reproduce the bug |
| **Expected Result** | Long Text | What should happen |
| **Actual Result** | Long Text | What actually happened |
| **Severity** | Status | Critical / High / Medium / Low |
| **Browser/Device** | Text | Browser and OS (e.g., "Chrome 120 / Windows 11") |
| **Test Case Reference** | Text | Link to the test case that found this bug (e.g., "02-system-admin-uat.md#vendor-management") |
| **Screenshot/Recording** | File | Attach screenshots or screen recordings |
| **Assignee** | People | Who is responsible for fixing this |
| **Status** | Status | Pending / In Progress / Fixed / Verified / Reopened / Accepted |
| **Date Reported** | Date | When the bug was first logged |
| **Date Resolved** | Date | When the bug was marked as Fixed |

### How to Add Columns

1. Click the **"+"** icon to the right of the existing columns
2. Select the column type (Text, Long Text, Status, etc.)
3. Name the column (e.g., "Severity")
4. For **Status** columns, define the allowed values:
	- **Severity**: Critical, High, Medium, Low
	- **Status**: Pending, In Progress, Fixed, Verified, Reopened, Accepted
5. Repeat for all columns listed above

### Optional Columns (Nice to Have)

| Column Name | Column Type | Purpose |
|-------------|-------------|---------|
| **Tester** | People | Who found the bug |
| **Build/Version** | Text | The UAT build version where the bug was found |
| **Workaround** | Long Text | Temporary workaround if one exists |
| **Related Bugs** | Link to Item | Link to other related bugs |

---

## Step 4: Define Status Workflow

Each bug goes through a lifecycle. Here's the recommended status workflow:

```
Pending → In Progress → Fixed → Verified → (Closed)
			↓
	 Reopened (if verification fails) → In Progress → Fixed → Verified
```

### Status Definitions

| Status | Definition | Who sets it |
|--------|------------|-------------|
| **Pending** | Bug is reported and waiting for a developer to pick it up | Tester |
| **In Progress** | A developer is actively working on the fix | Developer |
| **Fixed** | Developer believes the bug is resolved and ready for retesting | Developer |
| **Verified** | Tester has confirmed the bug is resolved | Tester |
| **Reopened** | Tester retested and the bug still exists | Tester |
| **Accepted** | Bug is known but will not be fixed before go-live (e.g., low priority cosmetic issue) | Project Manager |

---

## Step 5: How to Report a Bug

### 5.0 Quick bug report checklist

Use this quick checklist every time you log a bug. If any box is unticked, the report is not ready to submit.

- [ ] The title follows this format: `[Feature] - Short description of what broke`
- [ ] The steps are numbered and include exact clicks, field values, and test account used
- [ ] The expected result describes what should happen in plain language
- [ ] The actual result describes what happened instead, including any exact error message text
- [ ] The environment is recorded (browser, browser version, operating system, and device type)
- [ ] The test case reference is linked (for example `03-vendor-admin-uat.md#menu-management`)
- [ ] Evidence is attached (screenshot or recording)
- [ ] Severity is selected and justified with one sentence

### 5.0.1 Copy and paste template

Paste this into the item description or comment field when needed:

```text
Bug title:
[Feature] - Short description

Environment:
Browser + version, OS, device

Test account:
email@wantfood.com

Precondition:
What state the system was in before step 1

Steps to reproduce:
1.
2.
3.

Expected result:
-
-

Actual result:
-
-

Evidence:
- Screenshot(s):
- Screen recording:

Severity:
Critical | High | Medium | Low

Severity reason:
One sentence explaining business impact
```

### 5.1 Create a New Item

1. Navigate to the appropriate **group** (e.g., "System Admin" if the bug is in System Admin)
2. Click **"+ Add"** at the bottom of the group
3. Enter a short, clear **Bug Title** in the item name field

### 5.2 Fill in the Bug Template

Click on the newly created item to open its details, then fill in all columns:

#### Bug Title

**Format**: `[Feature] - Short description`

**Examples**:
- `[Vendor Management] - Cannot activate vendor`
- `[Checkout] - Payment fails with valid card`
- `[Order Kanban] - Orders not refreshing automatically`

#### Steps to Reproduce

Write a **numbered list** of exact steps to reproduce the bug. Be specific.

**Example**:
```
1. Log in to System Admin as admin.uat@wantfood.com
2. Navigate to Vendors → Vendors List
3. Find vendor "Test Restaurant 1" (ID: 123)
4. Click the "Activate" button
5. Observe the result
```

#### Expected Result

What **should** happen if the feature is working correctly.

**Example**:
```
- The vendor status should change to "Active"
- A success message should appear: "Vendor activated successfully"
- The "Activate" button should change to "Deactivate"
```

#### Actual Result

What **actually** happened.

**Example**:
```
- The page reloads but the vendor status remains "Inactive"
- No success message appears
- The "Activate" button does not change
- An error message appears in the browser console: "403 Forbidden"
```

#### Severity

Choose the appropriate severity level (see [00-introduction.md](00-introduction.md#severity-definitions) for guidance):

- **Critical**: System is unusable, blocks testing
- **High**: Major feature broken, difficult workaround
- **Medium**: Feature partially broken, but usable
- **Low**: Minor cosmetic issue or typo

#### Browser/Device

Record the browser, version, and operating system where you found the bug.

**Examples**:
- `Chrome 120 / Windows 11`
- `Safari 17 / macOS Sonoma`
- `Chrome Mobile / Android 13`
- `Safari / iOS 17 (iPhone 14)`

#### Test Case Reference

Link to the test case document and section that found this bug.

**Examples**:
- `02-system-admin-uat.md#vendor-management`
- `05-customer-frontend-uat.md#checkout-with-card`
- `06-order-saga-uat.md#happy-path-card-payment`

#### Screenshot/Recording

Attach a screenshot or screen recording showing the bug.

**Tools**:
- **Windows**: Snipping Tool, Windows + Shift + S
- **macOS**: Command + Shift + 4 (screenshot), Command + Shift + 5 (screen recording)
- **Browser extensions**: Loom, Awesome Screenshot

**Best practices**:
- Capture the entire browser window (not just the error message)
- Include the URL bar so the page is clear
- Circle or highlight the problem area if needed
- For complex bugs, use a screen recording tool (Loom, OBS)

**What to capture for high-quality evidence**:

1. **Before and after state** when possible:
	- Screenshot A: just before the action
	- Screenshot B: immediately after the failure
2. **Full window context**:
	- Include navigation, page title, and URL bar
	- Keep browser zoom at 100% unless zoom is part of the bug
3. **Error details**:
	- Capture visible toast, validation, or page error text
	- If there is no visible error, capture the silent failure state
4. **Time context**:
	- Leave system clock visible if possible, or add time in the bug comment
	- This helps developers correlate logs
5. **Cross-portal proof** (for TC-XP tests):
	- Capture the admin save/publish success state
	- Capture the customer or driver portal still showing the wrong state

#### Assignee

Assign the bug to **your project manager** by default. They will reassign it to the appropriate developer.

#### Status

Set the status to **"Pending"** when you first report the bug.

#### Date Reported

Set this to **today's date** (Monday.com may auto-populate this).

### 5.3 Example Completed Bug Item

| Field | Value |
|-------|-------|
| **Bug Title** | `[Vendor Management] - Cannot activate vendor` |
| **Steps to Reproduce** | 1. Log in to System Admin as admin.uat@wantfood.com<br>2. Navigate to Vendors → Vendors List<br>3. Find vendor "Test Restaurant 1" (ID: 123)<br>4. Click the "Activate" button<br>5. Observe the result |
| **Expected Result** | - The vendor status should change to "Active"<br>- A success message should appear: "Vendor activated successfully"<br>- The "Activate" button should change to "Deactivate" |
| **Actual Result** | - The page reloads but the vendor status remains "Inactive"<br>- No success message appears<br>- The "Activate" button does not change<br>- An error message appears in the browser console: "403 Forbidden" |
| **Severity** | High |
| **Browser/Device** | Chrome 120 / Windows 11 |
| **Test Case Reference** | 02-system-admin-uat.md#vendor-management |
| **Screenshot/Recording** | (screenshot attached) |
| **Assignee** | Project Manager |
| **Status** | Pending |
| **Date Reported** | 2025-01-30 |

### 5.4 Severity decision guide

If you are unsure between two levels, choose the higher one and add a short reason.

| If this is true... | Use severity |
|---|---|
| Testing cannot continue, or data/payment risk is severe with no safe workaround | **Critical** |
| A key flow is broken (checkout, order acceptance, login, delivery update) and workaround is poor | **High** |
| Feature still works but is partly wrong, confusing, or unreliable | **Medium** |
| Cosmetic issue, wording, spacing, minor non-blocking inconsistency | **Low** |

### 5.5 Final check before submitting

Before you click away from the item, confirm:

1. The steps reproduce the issue from a fresh session.
2. Expected and actual results are clearly different.
3. The screenshot or recording proves the issue without extra explanation.
4. Severity matches business impact, not frustration level.
5. Status is **Pending** and assignee is your project manager.
6. You can reproduce in at least one supported browser from [00-introduction.md](00-introduction.md#supported-browsers-and-devices), unless the bug is explicitly browser-specific.

---

## Step 6: Bug Lifecycle and Retesting

### 6.1 When a Bug is Fixed

The developer will:

1. Change the **Status** to **"Fixed"**
2. Add a comment explaining the fix
3. Tag you in the comment to notify you

You will receive a **notification** from Monday.com (via email or in-app).

### 6.2 How to Retest a Fixed Bug

1. Open the bug item in Monday.com
2. Read the developer's comment to understand what was fixed
3. Re-run the **exact steps from "Steps to Reproduce"**
4. Check if the **Expected Result** now occurs

### 6.3 If the Bug is Resolved

1. Change the **Status** to **"Verified"**
2. Add a comment: "Retested and confirmed fixed. Verified."
3. Set the **Date Resolved** to today

### 6.4 If the Bug Still Exists

1. Change the **Status** to **"Reopened"**
2. Add a comment explaining what still doesn't work
3. Attach a new screenshot or recording if needed
4. Tag the developer and project manager

### 6.5 If a Bug is Accepted (Not Fixed)

Sometimes a bug will not be fixed before go-live (e.g., a low-priority cosmetic issue). The project manager will:

1. Change the **Status** to **"Accepted"**
2. Add a comment explaining why it won't be fixed
3. Move it to a "Post-Launch Backlog" group (if applicable)

---

## Step 7: Linking Bugs to Test Cases

When reporting a bug, always include a **Test Case Reference** so developers can see which test case found the issue.

### Format

Use this format:

```
[Document Name]#[Section Heading]
```

### Examples

| Test Case | Reference |
|-----------|-----------|
| System Admin: Vendor Management → Activate Vendor | `02-system-admin-uat.md#vendor-management` |
| Customer Front-end: Checkout with Card | `05-customer-frontend-uat.md#checkout-with-card` |
| Cross-Portal Impact: Portal Consistency Check | `09-cross-portal-impact-uat.md#portal-consistency` |
| Permissions and Accounts: Role Access Validation | `10-permissions-and-account-uat.md#role-access-validation` |
| Resilience and Edge Cases: Timeout Handling | `11-resilience-and-edge-cases-uat.md#timeout-handling` |
| Order Saga: Happy Path (Card Payment) | `06-order-saga-uat.md#happy-path-card-payment` |

### Why Link Test Cases?

- Developers can quickly understand the context
- Project managers can see which areas of the platform have the most bugs
- You can easily find the test case to retest after a fix

---

## Step 8: Using Monday.com Features Effectively

### 8.1 Filters

Use **filters** to view specific subsets of bugs:

- **Show only my bugs**: Filter by "Assignee = Me"
- **Show only Critical/High bugs**: Filter by "Severity = Critical OR High"
- **Show only Pending bugs**: Filter by "Status = Pending"
- **Show only bugs I need to retest**: Filter by "Status = Fixed" and "Assignee = Me"

**How to create a filter**:
1. Click the **funnel icon** in the top right of the board
2. Select **"Add filter"**
3. Choose the column, condition, and value
4. Save the filter with a name (e.g., "My Bugs")

### 8.2 Board Views

Create custom **views** for different purposes:

- **Main Table**: Default view showing all bugs
- **Kanban**: Drag-and-drop view grouped by Status
- **Timeline**: View bugs on a calendar based on Date Reported
- **My Bugs**: Filtered view showing only bugs assigned to you

**How to create a view**:
1. Click **"Add view"** at the top of the board
2. Choose the view type (Kanban, Timeline, etc.)
3. Configure the view settings
4. Name the view (e.g., "My Bugs")

### 8.3 Notifications

Enable notifications so you're alerted when:

- A bug is assigned to you
- A developer comments on your bug
- A bug's status changes

**How to enable notifications**:
1. Click your profile icon in the top right
2. Select **"Notifications & settings"**
3. Enable email and/or in-app notifications for "Assigned to me" and "Comments"

### 8.4 Comments and Mentions

Use **comments** to:

- Ask clarifying questions about a bug
- Provide additional information
- Update stakeholders on retest results

**Mention someone** using `@` followed by their name:
- `@ProjectManager` — Can you prioritize this bug?
- `@Developer` — I retested and it's still broken, see attached screenshot

---

## Step 9: Best Practices for Bug Reporting

### Do:

✅ **Be specific**—include exact steps, error messages, and conditions  
✅ **Attach screenshots or recordings**—visual evidence is invaluable  
✅ **Report bugs immediately**—don't batch them at the end of the day  
✅ **Link to test cases**—helps developers understand context  
✅ **Retest thoroughly**—re-run the exact steps to confirm a fix  
✅ **Keep comments professional**—focus on facts, not opinions

### Don't:

❌ **Don't assume a bug is a duplicate**—report it anyway, the project manager will merge duplicates  
❌ **Don't under-report severity**—if you think it's High, mark it High  
❌ **Don't report "bugs" that are actually questions**—ask questions in Slack or email instead  
❌ **Don't edit another tester's bugs**—comment instead  
❌ **Don't wait for a fix before logging other bugs**—keep testing and logging

---

## Step 10: Example Monday.com Board Structure

Here's what your board should look like after setup:

```
┌─────────────────────────────────────────────────────────────────────────┐
│ WantFood UAT — Jane Smith                                              │
├─────────────────────────────────────────────────────────────────────────┤
│ Group: System Admin                                                     │
│ ├─ [Vendor Management] - Cannot activate vendor (High, Pending)         │
│ ├─ [Orders List] - Filter by date not working (Medium, Fixed)           │
│ └─ [Commissions] - Tier calculation shows incorrect rate (Critical, In Progress) │
├─────────────────────────────────────────────────────────────────────────┤
│ Group: Vendor Admin                                                     │
│ ├─ [Menu Builder] - Cannot reorder dishes (High, Verified)              │
│ ├─ [Order Kanban] - Orders not refreshing (Critical, Reopened)          │
│ └─ [Promotions] - Promo code validation fails (Medium, Fixed)           │
├─────────────────────────────────────────────────────────────────────────┤
│ Group: Driver Portal                                                    │
│ ├─ [Deliveries] - Cannot mark delivered (Critical, Fixed)               │
│ └─ [Shift Management] - Start shift button disabled (Medium, Pending)   │
├─────────────────────────────────────────────────────────────────────────┤
│ Group: Customer Front-end                                               │
│ ├─ [Checkout] - Payment fails with valid card (High, In Progress)       │
│ ├─ [Order Tracking] - Live tracking not updating (Medium, Fixed)        │
│ └─ [Reviews] - Cannot submit dish review (Low, Accepted)                │
├─────────────────────────────────────────────────────────────────────────┤
│ Group: Order Saga                                                       │
│ ├─ [End-to-End] - Order stuck in "Pending" state (Critical, Fixed)      │
│ └─ [Refunds] - Partial refund amount incorrect (High, Pending)          │
├─────────────────────────────────────────────────────────────────────────┤
│ Group: Automation & Jobs                                                │
│ └─ [Nightly Tiers] - Tier calculation job not running (Medium, Fixed)   │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Step 11: Daily UAT Routine

### Morning Routine

1. **Open Monday.com** and check for bugs assigned to you with Status = "Fixed"
2. **Filter** the board to show only "Fixed" bugs assigned to you
3. **Prioritize** Critical and High severity bugs for retesting first
4. **Retest** each Fixed bug using the original "Steps to Reproduce"
5. **Update status** to "Verified" (if fixed) or "Reopened" (if still broken)

### During Testing

1. **Log bugs immediately** when you find them—don't wait
2. **Take screenshots** as you test—easier to attach them right away
3. **Check for duplicates** before creating a new item (use the search bar)
4. **Comment on existing bugs** if you have additional information

### End of Day Routine

1. **Review your bugs**—ensure all have screenshots and complete information
2. **Check for any unfinished items**—fill in missing columns
3. **Post a summary** in the project Slack channel or email:
	- Number of new bugs reported today
	- Number of bugs verified today
	- Any blocking issues

---

## Questions or Issues?

If you have questions about Monday.com setup or need help with the board:

**Contact your Project Manager:**

- Email: `pm@wantfood.com` (example)
- Slack: `#uat-wantfood` (example)

---

## Next Steps

Now that your Monday.com board is set up, you're ready to start testing!

📄 **Begin testing with:**

1. **[System Admin UAT](02-system-admin-uat.md)**
2. **[Vendor Admin UAT](03-vendor-admin-uat.md)**
3. **[Driver Portal UAT](04-driver-portal-uat.md)**
4. **[Customer Front-end UAT](05-customer-frontend-uat.md)**
5. **[Order Saga UAT](06-order-saga-uat.md)**
6. **[Automation and Jobs UAT](07-automation-jobs-uat.md)**
7. **[Cross-Portal Impact UAT](09-cross-portal-impact-uat.md)**
8. **[Permissions and Accounts UAT](10-permissions-and-account-uat.md)**
9. **[Resilience and Edge Cases UAT](11-resilience-and-edge-cases-uat.md)**

---

**Happy Bug Hunting! 🐛**

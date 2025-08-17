# Mobile App Requirements: Field Officer

This document outlines the user stories and acceptance criteria for the Field Officer mobile application.

**Technology Stack:**
- **Frontend:** Flutter with Riverpod state management
- **Backend:** FastAPI with Supabase for Auth and Database

---

## 1. Login & Security

This section covers the authentication and security features for the Field Officer.

### Story 1.1: User Registration

**As a** Field Officer,
**I want to** register for an account using my company email ID and a secure password,
**So that** I can gain access to the application.

#### Acceptance Criteria

**Frontend (Flutter/Riverpod):**
- [ ] A registration screen shall be available from the main login screen.
- [ ] The screen shall have input fields for Email Address and Password.
- [ ] The email field must validate that the input is a valid email format.
- [ ] The password field must enforce a minimum of 8 alphanumeric characters.
- [ ] Real-time validation feedback should be provided to the user (e.g., "Password must be at least 8 characters").
- [ ] A "Register" button will be disabled until all validation rules are met.
- [ ] On successful registration, the user is automatically logged in and redirected to the Dashboard.
- [ ] On failure (e.g., email already exists, invalid domain), a clear error message is displayed to the user.
- [ ] The state of the registration process (e.g., loading, error, success) shall be managed by Riverpod.

**Backend (FastAPI/Supabase):**
- [ ] An endpoint `/auth/register` shall be created.
- [ ] The endpoint accepts `email` and `password` in the request body.
- [ ] It must validate that the email domain is from a list of approved company domains (e.g., `@yourcompany.com`). If not, return a 400 Bad Request error.
- [ ] The password must be hashed securely before being stored.
- [ ] A new user record is created in the Supabase `auth.users` table.
- [ ] A corresponding profile entry should be created in a public `profiles` table, linking to the `auth.users` record via UUID, and setting a default role of 'field_officer'.
- [ ] Upon successful creation, a JWT token is generated and returned to the client.
- [ ] If the user already exists, a 409 Conflict error should be returned.

---

### Story 1.2: User Login

**As a** Field Officer,
**I want to** log in using my email and password,
**So that** I can access my dashboard and daily tasks.

#### Acceptance Criteria

**Frontend (Flutter/Riverpod):**
- [ ] The login screen shall have input fields for Email and Password, and a "Login" button.
- [ ] The "Login" button triggers the authentication process.
- [ ] A loading indicator is shown while the authentication request is in progress.
- [ ] On successful login, the received JWT is securely stored on the device (e.g., using `flutter_secure_storage`).
- [ ] The user is redirected to the Dashboard screen upon successful login.
- [ ] If login fails (e.g., incorrect credentials), a user-friendly error message is displayed.
- [ ] The authentication state (logged in/out) shall be managed globally using a Riverpod provider.

**Backend (FastAPI/Supabase):**
- [ ] An endpoint `/auth/login` shall be created.
- [ ] The endpoint accepts `email` and `password`.
- [ ] It authenticates the user against the Supabase `auth.users` table.
- [ ] On successful authentication, a new JWT is generated and returned.
- [ ] On failure, a 401 Unauthorized error is returned.

---

### Story 1.3: Secure PIN Login

**As a** Field Officer,
**I want to** set up and use a secure PIN for subsequent logins,
**So that** I can access the app more quickly and securely without entering my full password every time.

#### Acceptance Criteria

**Frontend (Flutter/Riverpod):**
- [ ] After the first successful password login, the user shall be prompted to set up a 4 or 6-digit PIN.
- [ ] The user must enter the PIN twice to confirm.
- [ ] The PIN shall be securely stored on the device (e.g., in `flutter_secure_storage`). It should NOT be sent to the server.
- [ ] On subsequent app launches, if a PIN is set, the user is shown a PIN entry screen instead of the password login.
- [ ] Entering the correct PIN grants access to the app. The app should use the stored JWT for API calls.
- [ ] After 3 incorrect PIN attempts, the user is logged out, the locally stored PIN and JWT are deleted, and they are forced to log in again with their full email and password.
- [ ] A "Forgot PIN?" or "Logout" option should be available on the PIN screen to allow the user to return to the main password login page.

**Backend (FastAPI/Supabase):**
- [ ] No backend changes are required for this feature as the PIN is a device-local authentication mechanism. The backend will continue to honor the JWT provided by the client.

---

### Story 1.4: Biometric Login

**As a** Field Officer,
**I want to** use my device's biometric capabilities (fingerprint/face ID) to log in,
**So that** I can have the most seamless and secure access to the app.

#### Acceptance Criteria

**Frontend (Flutter/Riverpod):**
- [ ] On the PIN setup screen or in the app's settings, the user should be given the option to enable Biometric login.
- [ ] The app must check if the device supports biometrics using a package like `local_auth`.
- [ ] If enabled, on app launch, the OS-level biometric prompt will be shown.
- [ ] Successful biometric authentication should grant access to the app (similar to a correct PIN entry).
- [ ] If biometric authentication fails or is cancelled, the app should fall back to the PIN entry screen.
- [ ] The app settings should have a toggle to enable/disable biometric login.

**Backend (FastAPI/Supabase):**
- [ ] No backend changes are required. This is a client-side feature that unlocks the locally stored JWT.

---

### Story 1.5: Secure Access and Token Management

**As a** Field Officer,
**I want** the app to manage my session securely,
**So that** my data is protected.

#### Acceptance Criteria

**Frontend (Flutter/Riverpod):**
- [ ] The JWT received from the backend must be included in the Authorization header for all protected API requests (e.g., `Authorization: Bearer <token>`).
- [ ] A network layer (e.g., using Dio interceptors) should be implemented to automatically add the token to requests.
- [ ] The interceptor must handle 401 Unauthorized responses from the server. Upon receiving a 401, the app should attempt to use a refresh token (if implemented) or log the user out and clear all stored credentials.
- [ ] The app should proactively log the user out if the JWT expires and cannot be refreshed.

**Backend (FastAPI/Supabase):**
- [ ] All endpoints beyond login/register must be protected and require a valid JWT.
- [ ] A dependency injection system in FastAPI should be used to validate the JWT from the `Authorization` header on incoming requests.
- [ ] The JWT validation must check the signature, expiration date, and issuer.
- [ ] (Future Consideration - MFA) If Multi-Factor Authentication is implemented, the login flow will need to be adjusted to handle the second factor (e.g., TOTP code). The JWT could contain a claim indicating the authentication level.
- [ ] Implement token refresh logic. Supabase handles this by default. The client should be able to request a new access token using a refresh token.

---

## 2. Dashboard

This section outlines the features of the main dashboard screen, which is the first screen the user sees after logging in.

### Story 2.1: View Visit Schedules

**As a** Field Officer,
**I want to** see my assigned client visits organized by timeframes (Today, This Week, This Fortnight, This Month),
**So that** I can easily prioritize and manage my work.

#### Acceptance Criteria

**Frontend (Flutter/Riverpod):**
- [ ] The dashboard shall be the default screen after login.
- [ ] The screen should display a tabbed or sectioned layout for "Today's Visits," "This Week's Visits," "This Fortnight's Visits," and "This Month's Visits."
- [ ] Each section will display a list of client names (CRMs) assigned to the officer for that period.
- [ ] If a list is empty, a user-friendly message like "No visits scheduled for today" should be shown.
- [ ] The app will fetch this data from the backend upon loading the dashboard.
- [ ] A Riverpod provider will manage the state of the visit lists (loading, data, error).
- [ ] Tapping on a client name in the list will navigate the user to the Audit Start screen (see Section 3).

**Backend (FastAPI/Supabase):**
- [ ] A single endpoint, e.g., `/visits`, shall be created to provide all visit information for the logged-in Field Officer.
- [ ] The endpoint should be protected, requiring a valid JWT.
- [ ] The endpoint should return a JSON object structured by period, e.g., `{ "today": [...], "week": [...], "fortnight": [...], "month": [...] }`.
- [ ] Each item in the list should contain `client_id`, `client_name`, `client_location`, and `audit_status`.
- [ ] The logic needs to query a `visits` or `assignments` table, filtering by the `field_officer_id` (extracted from the JWT) and the date range for each period.
    - **Today:** `date == today()`
    - **This Week:** `date >= start_of_week AND date <= end_of_week`
    - **This Fortnight:** `date >= start_of_fortnight AND date <= end_of_fortnight`
    - **This Month:** `date >= start_of_month AND date <= end_of_month`
- [ ] The database schema should support associating clients with visit frequencies (daily, weekly, fortnightly, monthly) and assigning them to Field Officers.

---

### Story 2.2: Visual Cue for Completed Audits

**As a** Field Officer,
**I want to** see a clear visual indicator next to clients whose audits I have already completed for the current period,
**So that** I don't perform the same audit twice and can track my progress.

#### Acceptance Criteria

**Frontend (Flutter/Riverpod):**
- [ ] In the visit lists on the dashboard, any client whose audit for the current period is complete shall be marked with a green checkmark icon.
- [ ] The `audit_status` field from the backend response will be used to determine if the checkmark should be displayed.
- [ ] The visual state should update automatically after an audit is successfully submitted, without requiring a manual refresh (the Riverpod state should be updated).

**Backend (FastAPI/Supabase):**
- [ ] The `/visits` endpoint response for each client needs to include a status field, e.g., `audit_status: "completed" | "pending"`.
- [ ] The logic for this endpoint must check if an audit record exists for the given `client_id`, `field_officer_id`, and the relevant time period. For example, for a weekly visit, it checks if a completed audit exists in the current week.

---

## 3. Audit Workflow: Starting an Audit

This section describes the process of initiating a site audit after selecting a client from the dashboard.

### Story 3.1: Site Validation via Scanner

**As a** Field Officer,
**I want to** scan a site-specific RFID tag, Barcode, or QR code when I arrive at a client's location,
**So that** the app can verify I am at the correct site before I begin my audit.

#### Acceptance Criteria

**Frontend (Flutter/Riverpod):**
- [ ] Tapping a client name on the dashboard navigates to a scanning screen.
- [ ] The device's camera will be activated to scan for a QR code or barcode. The app should use a package like `mobile_scanner` or `qr_code_scanner`.
- [ ] (If RFID is pursued) Integration with device-specific RFID hardware/APIs will be required. This may need a separate proof-of-concept. For Phase 1, QR/Barcode is sufficient.
- [ ] A clear UI overlay should instruct the user what to do (e.g., "Scan Site QR Code").
- [ ] Once a code is detected, its value is sent to the backend for validation. A loading indicator should be shown.
- [ ] **On successful validation:**
    - The app navigates to the "Audit Checklist" screen.
    - Client data received from the backend (Client Name, Location, etc.) is passed to the audit screen and stored in a Riverpod provider for the duration of the audit.
- [ ] **On failed validation:**
    - An alert dialog is shown to the user with a clear error message (e.g., "Incorrect Site Code Scanned").
    - Tapping "OK" or "Accept" on the alert dialog dismisses it and navigates the user back to the Dashboard screen.

**Backend (FastAPI/Supabase):**
- [ ] An endpoint, e.g., `/audits/validate-site`, shall be created. It should accept the `scanned_code` and the `client_id` (which the app knows from the dashboard selection).
- [ ] The endpoint must be protected and require a valid JWT.
- [ ] The logic will look up the `client_id` in a `clients` table and compare the `scanned_code` with the stored `site_code` (which could be an RFID tag ID, barcode value, etc.).
- [ ] **If validation succeeds:**
    - The server should return a 200 OK status.
    - The response body should include all necessary data to pre-populate the audit, such as: `client_name`, `location`, `key_contacts`, `allocated_staff_count`, and a list of `assets`.
    - A new, temporary token for this specific audit session should be generated and returned. This token (`audit_entry_token`) will be used to submit the final audit data, preventing duplicate submissions.
- [ ] **If validation fails:**
    - The server should return a 400 Bad Request or 404 Not Found status with an error message.

---

## 4. Audit Checklist

This section details the audit form that the Field Officer fills out after successful site validation.

### Story 4.1: Complete a Site Audit

**As a** Field Officer,
**I want to** fill out a comprehensive checklist about the site's status,
**So that** I can record a complete and accurate picture of the site's operational health.

#### General Acceptance Criteria

**Frontend (Flutter/Riverpod):**
- [ ] The audit checklist screen shall be presented as a single, scrollable form.
- [ ] The form should be logically divided into the sections outlined below (e.g., Attendance, SOP Compliance, etc.).
- [ ] Data passed from the site validation step (Client Name, Location) should be displayed as read-only information at the top of the screen.
- [ ] The user's name and the current timestamp should be automatically captured and displayed.
- [ ] The state of the entire audit form will be managed by a Riverpod provider. The state should be preserved if the user navigates away from the app temporarily.
- [ ] A "Submit Audit" button shall be present at the bottom of the form. It should be enabled only after all mandatory fields are filled.
- [ ] The app should handle file/image attachments, allowing the user to take a photo with the camera or select one from the gallery. A package like `image_picker` should be used.

**Backend (FastAPI/Supabase):**
- [ ] An endpoint, e.g., `/audits`, shall be created to accept the submission of the completed audit form (HTTP POST).
- [ ] The endpoint must be protected and require a valid JWT.
- [ ] The request body will contain the `audit_entry_token` obtained during site validation, and a JSON payload with all the checklist data.
- [ ] The server must validate the `audit_entry_token` to ensure the audit is valid and hasn't already been submitted.
- [ ] Upon successful submission, the server will invalidate the `audit_entry_token`.
- [ ] The endpoint will parse the incoming JSON and save the data to a new record in an `audits` table. The record should be linked to the `client_id` and `field_officer_id`.
- [ ] For any attached files (images/screenshots), the endpoint should handle multipart/form-data uploads. Files should be saved to Supabase Storage, and the URLs stored in the `audits` table record.

---

#### 4.1.1: Attendance Verification

**Frontend (Flutter/Riverpod):**
- [ ] A "Yes/No" toggle or radio button for "Attendance Verified".
- [ ] A button to attach a screenshot/photo of the attendance list. Display a thumbnail of the selected image.
- [ ] Numeric input fields for "Allocated Staff" vs. "Actual Staff".
- [ ] A multi-line text area for "Special Instructions/Observations".

**Backend (FastAPI/Supabase):**
- [ ] The `audits` table schema should have columns for:
    - `attendance_verified` (boolean)
    - `attendance_attachment_url` (text)
    - `staff_allocated_count` (integer)
    - `staff_actual_count` (integer)
    - `attendance_observations` (text)

---

#### 4.1.2: Uniform & Grooming

**Frontend (Flutter/Riverpod):**
- [ ] A choice-based input (e.g., dropdown or radio buttons) with options: "All in order", "Issues noted".
- [ ] If "Issues noted" is selected, a text area for details becomes visible and required.
- [ ] An optional button to attach a picture.

**Backend (FastAPI/Supabase):**
- [ ] `uniform_grooming_status` (text, e.g., 'all_in_order', 'issues_noted')
- [ ] `uniform_grooming_issues` (text, nullable)
- [ ] `uniform_grooming_attachment_url` (text, nullable)

---

#### 4.1.3: Patrol Logs

**Frontend (Flutter/Riverpod):**
- [ ] A choice-based input: "Compliant", "Issues noted".
- [ ] If "Issues noted", a text area for details becomes visible and required.
- [ ] An optional button to attach screenshots of patrol logs.

**Backend (FastAPI/Supabase):**
- [ ] `patrol_logs_status` (text, e.g., 'compliant', 'issues_noted')
- [ ] `patrol_logs_issues` (text, nullable)
- [ ] `patrol_logs_attachment_url` (text, nullable)

---

#### 4.1.4: Incident Reports Review (Phase 1)

**Frontend (Flutter/Riverpod):**
- [ ] A "Yes/No" toggle for "Incidents Reviewed".
- [ ] A multi-line text area to summarize findings or list incident IDs.

**Backend (FastAPI/Supabase):**
- [ ] `incident_reports_reviewed` (boolean)
- [ ] `incident_reports_summary` (text, nullable)
- [ ] *Note for Phase 2: This will be replaced by a list of incidents from a master table.*

---

#### 4.1.5: Assets Review (Phase 1)

**Frontend (Flutter/Riverpod):**
- [ ] The UI should allow for a dynamic list of assets. The user can add/remove assets to the list.
- [ ] For each asset, the following fields should be present:
    - `Asset Name` (text input)
    - `Asset Owner` (text input)
    - `Asset Status` (Choice: "Verified Good", "Verified - Issues Identified")
    - `Remarks` (text area, visible if status is "Verified - Issues Identified")
- [ ] *Note for Phase 3: This will be a pre-populated list from master data.*

**Backend (FastAPI/Supabase):**
- [ ] The audit data should accept a JSON array for assets, e.g., `assets_review: [{...}]`.
- [ ] The `audits` table could have a JSONB column to store this data.
- [ ] Each object in the array will contain `asset_name`, `asset_owner`, `asset_status`, `asset_remarks`.

---

#### 4.1.6: SOP Compliance

**Frontend (Flutter/Riverpod):**
- [ ] This section will contain a list of SOP items.
- [ ] For each item, there will be a consistent UI component (e.g., a card).
- [ ] Each item will have a title (e.g., "Shift Scheduling") and a choice-based input (e.g., "Compliant" / "Not Compliant" or "In Place" / "Missing").
- [ ] A text area for "Remarks/Issues" will be available for each item.

**SOP Items:**
1.  **Shift Scheduling:** [Compliant / Not Compliant], Remarks
2.  **Shiftwise Trainings:** [Being Conducted / Not Conducted], Remarks
3.  **Night Shift Guard Rotation:** [In Place / Missing], Remarks
4.  **Scenario-based Drills:** [Being Conducted / Not Conducted], Remarks
5.  **Occurrence Register:** [Maintained / Not Maintained], Remarks
6.  **Hotspots Review:** [Safe & Secure / Having Issues], Remarks
7.  **CCTV Coverage & Condition:** [Working Well / Not Working Well], Remarks
8.  **Shift Handover Process:** [Maintained / Not Maintained], Remarks

**Backend (FastAPI/Supabase):**
- [ ] The `audits` table should have a JSONB column `sop_compliance` to store this structured data, or have individual columns for each SOP item's status and remarks. A JSONB column is more flexible for future changes.
- [ ] Example structure: `sop_compliance: { "shift_scheduling": { "status": "compliant", "remarks": "..." }, ... }`

---

#### 4.1.7: Emergency Readiness

**Frontend (Flutter/Riverpod):**
- [ ] A choice-based input: "Satisfactory", "Needs Improvement".
- [ ] A text area for "Remarks/Issues".

**Backend (FastAPI/Supabase):**
- [ ] `emergency_readiness_status` (text)
- [ ] `emergency_readiness_remarks` (text, nullable)

---

#### 4.1.8: Client Meeting Status

**Frontend (Flutter/Riverpod):**
- [ ] A choice-based input: "Met", "Not Met".
- [ ] If "Met" is selected, a text input for "Client Contact Name" becomes visible and required.
- [ ] A multi-line text area for "Summary of feedback".

**Backend (FastAPI/Supabase):**
- [ ] `client_meeting_status` (text)
- [ ] `client_contact_name` (text, nullable)
- [ ] `client_feedback_summary` (text, nullable)

---

#### 4.1.9: Payments Status

**Frontend (Flutter/Riverpod):**
- [ ] A choice-based input: "All Clear", "Outstanding".
- [ ] A text area for "Remarks" if "Outstanding".

**Backend (FastAPI/Supabase):**
- [ ] `payments_status` (text)
- [ ] `payments_remarks` (text, nullable)

---

#### 4.1.10: Closing Recommendations

**Frontend (Flutter/Riverpod):**
- [ ] An optional multi-line text area for "Closing Recommendations".

**Backend (FastAPI/Supabase):**
- [ ] `closing_recommendations` (text, nullable)

---

## 5. Submission and Completion

This section covers the final steps of the audit process.

### Story 5.1: Submit the Audit and Get Confirmation

**As a** Field Officer,
**I want to** submit my completed audit form,
**So that** the results are saved permanently and I can move on to my next task.

#### Acceptance Criteria

**Frontend (Flutter/Riverpod):**
- [ ] Pressing the "Submit Audit" button at the bottom of the checklist form initiates the submission process.
- [ ] A loading overlay is displayed to prevent further interaction and indicate that the submission is in progress.
- [ ] All form data, including any attached images, is sent to the backend `/audits` endpoint.
- [ ] **On successful submission:**
    - A success message or toast is displayed to the user (e.g., "Audit submitted successfully!").
    - The app automatically navigates the user back to the Dashboard screen.
    - The dashboard state should be refreshed to show the completed status for the audit that was just submitted.
- [ ] **On submission failure (e.g., network error, server error):**
    - An error dialog is shown with a clear message (e.g., "Submission Failed. Please try again.").
    - The user remains on the audit form with all their entered data intact, so they can retry the submission.

**Backend (FastAPI/Supabase):**
- [ ] The `/audits` POST endpoint handles the data submission as described in section 4.
- [ ] After successfully saving the audit record and uploading any files to the database and storage...
- [ ] **Trigger an email notification:**
    - An email should be sent to a predefined list of recipients (e.g., the Operations Manager).
    - The email should contain a summary of the audit, such as Client Name, Field Officer Name, Time of Audit, and a summary of any issues found. A direct link to a web view of the full audit report could be included.
    - This can be implemented using Supabase Edge Functions triggered by a new entry in the `audits` table, or by using a service like SendGrid/Postmark called from the FastAPI endpoint.
- [ ] The endpoint returns a 201 Created status on success, or an appropriate error code (4xx, 5xx) on failure.

---

## 6. Operations Manager Web Interface/App (Future Scope)

This section is a placeholder for the Operations Manager web interface. The features below are high-level suggestions based on the functionality of the Field Officer app and will need to be expanded into detailed user stories and acceptance criteria in a separate requirements phase.

**Technology Stack Suggestion:**
- **Frontend:** React/Vue/Angular or a Python-based framework like Dash/Streamlit.
- **Backend:** Can utilize the same FastAPI backend, but with a different set of role-protected endpoints.

---

### Potential High-Level Features / Stories:

#### Story 6.1: Dashboard & Analytics
**As an** Operations Manager,
**I want to** see a high-level dashboard with key metrics (e.g., audits completed today, open issues, officer activity),
**So that** I can quickly assess the overall status of field operations.

---

#### Story 6.2: View and Review Audits
**As an** Operations Manager,
**I want to** view a list of all submitted audits, with filtering and search capabilities,
**So that** I can review the work being done by Field Officers.
- *Details needed: What filters are required (date, officer, client, status)? What does a detailed audit view look like?*

---

#### Story 6.3: User Management
**As an** Operations Manager,
**I want to** manage the accounts of Field Officers (create, update, deactivate),
**So that** I can control who has access to the mobile application.

---

#### Story 6.4: Client & Site Management
**As an** Operations Manager,
**I want to** manage client information, site details (including site codes for scanning), and visit schedules,
**So that** I can assign work to Field Officers and keep site information up to date.

---

#### Story 6.5: Master Data Management
**As an** Operations Manager,
**I want to** manage master data lists (e.g., Assets for Phase 3, Incident types for Phase 2),
**So that** I can ensure consistency in the audit checklists.

# Phase E — Workflows

**Generated:** 2026  
**Repository:** AB3 Medical Management System  
**Purpose:** End-to-end user journeys with step-by-step API calls and state transitions

---

## WORKFLOW 1: User Authentication & Login

**Name:** User Login with Optional 2FA  
**Actors:** SUPER_ADMIN (System + Org), ADMIN (System + Org), MODERATOR (System + Org), MEDIC, EXTERNAL  
**Trigger:** User submits login form

**UI Components:**

- **Page:** `/authentication/login` → `LoginStep.tsx`
- **Page (2FA):** `/authentication/otp-verifications` → OTP verification form
- **Redirect:** `/dashboard` (after successful login)

**Steps:**

1. **UI:** User (any type: SUPER_ADMIN, ADMIN, MODERATOR, MEDIC, EXTERNAL) enters email/password on login page
2. **API:** `POST /api/v1/users/login` → `UserService.loginUser()`
3. **Backend:**
   - Validate email/password
   - Check if user exists and is not deleted
   - Check if 2FA is enabled
   - If 2FA enabled: Generate 6-digit code, save to user, send email, return `twoFactorEnabled: true`
   - If 2FA disabled: Generate JWT token, create session, return token
4. **UI:**
   - If 2FA: Redirect to `/authentication/otp-verifications` (OTP verification page)
   - If no 2FA: Set cookie, redirect to `/dashboard`
5. **API (if 2FA):** `POST /api/v1/users/verify2FAOTP` → Verify code, generate token, create session
6. **UI:** Set cookie, redirect to `/dashboard` (Dashboard page with role-based content)

**State Transitions:**

- User status: `PENDING` → `ACTIVE` (on first login after invite)
- Session created in database

**Notifications:**

- Email sent with 2FA code (if enabled)

**Error States:**

- Invalid credentials → 401
- Account deleted → 403
- Expired 2FA code → 400
- Invalid OTP → 400

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/User/user.services.ts:293-386`
- `ab3-medical-api-v2/src/app/modules/User/user.controller.ts:62-98`
- `ab3-medical-new/src/components/Login/LoginStep.tsx`
- `ab3-medical-new/src/app/authentication/login/page.tsx`
- `ab3-medical-new/src/app/authentication/otp-verifications/page.tsx`

---

## WORKFLOW 2: Password Reset (Forgot Password)

**Name:** Reset Password via Email Link  
**Actors:** All users  
**Trigger:** User requests password reset

**Steps:**

1. **UI:** Upon Clicking Forget Password On Login Screen, Navigate to `/authentication/reset-password`, enter email
2. **API:** `POST /api/v1/users/reset-link` → `UserService.resetLink()`
3. **Backend:**
   - Find user by email
   - Generate reset token
   - Save token to user
   - Send password reset email with link
4. **Email:** User receives email with link to `/authentication/create-new-password?token=...`
5. **UI:** User clicks link, enters new password
6. **API:** `POST /api/v1/users/reset-password` → `UserService.resetPassword()`
7. **Backend:**
   - Verify token
   - Hash new password
   - Update user password
   - Clear reset token
8. **UI:** Redirect to login

**State Transitions:**

- User password updated
- Reset token cleared

**Notifications:**

- Password reset email sent

**Error States:**

- User not found → 404
- Invalid/expired token → 400

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/User/user.services.ts:502-536`
- `ab3-medical-new/src/app/authentication/reset-password/page.tsx`
- `ab3-medical-new/src/app/authentication/create-new-password/page.tsx`

---

## WORKFLOW 3: Change Password

**Name:** User Changes Password  
**Actors:** All authenticated users  
**Trigger:** User changes password in settings

**Steps:**

1. **UI:** Navigate to `/setting`, open "Change Password" modal
2. **UI:** Enter old password, new password, confirm password
3. **API:** `POST /api/v1/users/change-password` → `UserService.changePassword()`
4. **Backend:**
   - Fetch user with password field
   - Verify old password matches
   - Hash new password
   - Update user password
5. **UI:** Show success, close modal

**State Transitions:**

- User password updated

**Error States:**

- Old password incorrect → 401
- User has no password → 400
- User not found → 404

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/User/user.services.ts:538-565`
- `ab3-medical-new/src/components/Modals/ChangePasswordModal.tsx`

---

## WORKFLOW 4: User Logout

**Name:** Logout and Session Management  
**Actors:** All authenticated users  
**Trigger:** User clicks logout

**Steps:**

1. **UI:** User clicks logout button in navbar
2. **API:** `POST /api/v1/users/logout` → `UserController.logoutUser()`
3. **Backend:**
   - Find session by user ID and token
   - Delete session from database
4. **UI:**
   - Clear cookies (accessToken, isAuth)
   - Clear localStorage (rememberMe)
   - Clear sessionStorage (sessionActive)
   - Redirect to `/authentication/login`

**State Transitions:**

- Session deleted from database
- Cookies cleared

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/User/user.controller.ts:173-210`
- `ab3-medical-new/src/components/Shared/Navbar.tsx:65-87`

---

## WORKFLOW 5: Organization Creation & Setup

**Name:** Create Organization and Invite Super Admin  
**Actors:** SYSTEM_ADMIN  
**Trigger:** SYSTEM_ADMIN creates new organization

**Steps:**

1. **UI:** SYSTEM_ADMIN navigates to `/organizations`, clicks "Create Organization"
2. **UI:** Fills organization form (name, email, type, sport, address, super admin details)
3. **API:** `POST /api/v1/organizations` → `OrganizationService.createOrganization()`
4. **Backend:**
   - Start MongoDB transaction
   - Create Organization document with status `Pending`
   - Generate invite token for super admin
   - Create User with status `INVITED`, userType `SUPER_ADMIN`
   - Create Profile for super admin
   - Create OrganizationUser with status `INVITED`
   - Send invitation email with setup link
   - Commit transaction
5. **Email:** Super admin receives email with link to `/setup-organization?token=...`
6. **UI:** Super admin clicks link, completes setup form (password, organization details)
7. **API:** `POST /api/v1/organizations/setup-organization-super-admin` → Set password, activate user
8. **API:** `POST /api/v1/organizations/setup-organization` → Update organization status to `Active`
9. **UI:** Redirect to dashboard

**State Transitions:**

- Organization: Created with status `Pending` → `Active` (after setup)
- User: `INVITED` → `ACTIVE`
- OrganizationUser: `INVITED` → `ACTIVE`

**Notifications:**

- Invitation email sent to super admin

**Error States:**

- User already exists → 409
- Invalid token → 400
- Organization already setup → 409

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Organization/organization.services.ts:45-194`
- `ab3-medical-api-v2/src/app/modules/Organization/organization.services.ts:628-712`
- `ab3-medical-api-v2/src/app/modules/Organization/organization.services.ts:762-823`

---

## WORKFLOW 6: View Organization List & Update Organization, Status Change, Delete Organization

**Name:** View Organization List & Update  
**Actors:** SYSTEM_ADMIN, SUPER_ADMIN, ADMIN, MODERATOR  
**Trigger:** Admin navigates to organization list page

**UI Components:**

- **Page:** `/organizations` → `OrganizationsPageContent.tsx`
- **Stats:** `OrganizationStats.tsx`
- **Search:** `EnhancedSearchInput.tsx`
- **Advanced Filters:** `OrganizationAdvancedSearch.tsx`
- **Table:** `OrganizationRecords.tsx` → `OrganizationTableRow.tsx`
- **Modals:**
  - `InviteOrganizationModal.tsx` (to create/invite new organization)
  - `EditOrganizationModal.tsx` (to update existing organization)

**Steps:**

1. **UI:** Admin navigates to `/organizations`.
2. **UI:** `OrganizationsPageContent.tsx` displays stats, search inputs, and the organization records table.
3. **API:** `GET /api/v1/organizations` → `getAllOrganization()` (with search, status, type, sport, and date filters).
4. **UI:** Admin can use:
   - **Search:** Filter by name, email, city, etc.
   - **Advanced Filters:** Filter by Type, Sport, Member Count, Date Range.

**Actions (Per-Organization):**

1. **Update / Edit Organization:**
   - **Trigger:** Select "Edit" from row dropdown.
   - **UI:** `EditOrganizationModal.tsx` opens with current data.
   - **Modifiable Fields:** Name, Image, Address, City, State, Country, Phone, Organization Type, Organization Sport, and **Organization Status** (Active, Inactive, Pending, Deleted).
   - **API:** `PUT /api/v1/organizations/:id` → `updateOrganization()`
   - **Backend:** `OrganizationService.updateOrganization()` (updates the organization document).
   - **UI:** Show success toast, refresh list, invalidate stats.
2. **Delete Organization (Soft Delete):**
   - **Trigger:** Select "Delete" from row dropdown.
   - **Confirm:** SweetAlert confirmation dialog.
   - **API:** `DELETE /api/v1/organizations/:id` → `deleteOrganization()`
   - **Backend:** `OrganizationService.deleteOrganization()` (sets `organizationStatus: 'Deleted'`).
   - **UI:** Show success toast, refresh list.

**State Transitions:**

- Organization Status: `Pending/Active/Inactive` → `Deleted`
- Organization Details: Fields updated based on form submission.

**Evidence:**

- `ab3-medical-new/src/components/organizations/OrganizationsPageContent.tsx`
- `ab3-medical-new/src/components/Tables/OrganizationTableRow.tsx`
- `ab3-medical-new/src/components/Modals/EditOrganizationModal.tsx`
- `ab3-medical-api-v2/src/app/modules/Organization/organization.services.ts`
- `ab3-medical-api-v2/src/app/modules/Organization/organization.route.ts`

---

## WORKFLOW 7: View Organization Details

**Name:** View Organization Information and Records  
**Actors:** SYSTEM_ADMIN, SUPER_ADMIN, ADMIN, MODERATOR  
**Trigger:** SYSTEM_ADMIN, SUPER_ADMIN, ADMIN, or MODERATOR navigates to organization detail page

**UI Components:**

- **Page:** `/organizations/[id]` → `OrganizationDetailsPage`
- **Views:**
  - `OrganizationInformation.tsx` (organization details: name, email, type, sport, address, contact info)
  - `OrganizationRecords.tsx` (tabbed view: Patients, Medics, Admins, Teams, Externals)
- **Sub-Views / Shared Tables:**
  - `OrganizationPatients.tsx` (reuses the same patient table patterns as the global patients list workflow)
  - `OrganizationMedics.tsx` (reuses `MedicsPageContent`/`MedicRecords`/`MedicTableRow` table behavior from **WORKFLOW 32: Manage Medics**)
  - `OrganizationAdmins.tsx` (reuses `UsersPage`/`UserRecords`/`UserTableRow` table behavior from **WORKFLOW 33: Manage Organization Users**)
  - `OrganizationTeams.tsx` (teams table with `TeamTableHeader.tsx` and `TeamTableRow.tsx`) (See **WORKFLOW 25: Create Team** and **WORKFLOW 29: View Team Members**)
  - `OrganizationExternals.tsx` (reuses `ExternalPage`/`ExternalRecord`/`ExternalTableRow` table behavior from **WORKFLOW 35: Manage External Users**)

**Steps:**

1. **UI:** SYSTEM_ADMIN, SUPER_ADMIN, ADMIN, or MODERATOR navigates to `/organizations/[id]`
2. **UI:** `OrganizationDetailsPage` displays:
   - `OrganizationInformation.tsx`: Organization details (read-only)
   - `OrganizationRecords.tsx`: Tabbed records view
3. **API:**
   - `GET /api/v1/organizations/:id` → Get organization details
   - `GET /api/v1/teams/organization/:organizationId` → Get teams
   - `GET /api/v1/organizations/:id/medics` → Get medics
   - `GET /api/v1/organizations/:id/patients` → Get patients
4. **UI:** Display organization data, teams (in table), medics (in table), and patients (in table)

**State Transitions:**

- None (read-only view)

**Evidence:**

- `ab3-medical-new/src/app/(dashboard)/organizations/[id]/page.tsx`
- `ab3-medical-new/src/components/organizations/[id]/OrganizationInformation.tsx`
- `ab3-medical-new/src/components/organizations/[id]/OrganizationRecords.tsx`
- `ab3-medical-new/src/components/organizations/[id]/OrganizationTeams.tsx`
- `ab3-medical-new/src/components/organizations/[id]/OrganizationMedics.tsx`
- `ab3-medical-new/src/components/organizations/[id]/OrganizationPatients.tsx`

---

**Error States:**

- Organization not found → 404
- Duplicate name/email → 400

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Organization/organization.services.ts:825-841`
- `ab3-medical-api-v2/src/app/modules/Organization/organization.route.ts:26`

---

## WORKFLOW 8: User Invitation & Onboarding

**Name:** Invite User to Organization  
**Actors:** SUPER_ADMIN, ADMIN (within organization)  
**Trigger:** SUPER_ADMIN or ADMIN invites user via invite modal

**UI Components:**

- **Modals:**
  - `InviteMedicModal.tsx` (for inviting medics)
  - `InviteNewPatientModal.tsx` (for inviting new patients)
  - `InviteOldPatientModal.tsx` (for inviting existing patients)
  - `InviteOrganizationUserModal.tsx` (for inviting org users)
  - `InviteExternalModal.tsx` (for inviting external users)
- **Page:** `/onboarding-user?token=...` → `OnBoardingUser.tsx`
- **Redirect:** `/authentication/login` or `/dashboard` (after onboarding)

**Steps:**

1. **UI:** SUPER_ADMIN or ADMIN navigates to:
   - `/patients` → Opens `InviteNewPatientModal.tsx` or `InviteOldPatientModal.tsx`
   - `/medics` → Opens `InviteMedicModal.tsx`
   - `/users` → Opens `InviteOrganizationUserModal.tsx`
   - `/external` → Opens `InviteExternalModal.tsx`
2. **UI:** Admin fills invite form (email, name, role, teams) in modal
3. **API:** `POST /api/v1/invites/invite-org-user` or `POST /api/v1/invites/invite-patient` or `POST /api/v1/invites/invite-external`
4. **Backend:** `InviteService.inviteOrgUser()` or `invitePatient()` or `inviteExternal()`
   - Check authorization (inviter must be SUPER_ADMIN or ADMIN of org)
   - Generate invite token
   - Create User with status `INVITED`
   - Create OrganizationUser with status `INVITED`
   - Create TeamUser entries for each team (if provided)
   - Send invitation email with token link
5. **Email:** Invited user (MEDIC, PATIENT, EXTERNAL, or org user) receives email with link to `/onboarding-user?token=...`
6. **UI:** User clicks link, lands on onboarding page (`OnBoardingUser.tsx`)
7. **API:** `GET /api/v1/organizations/get-organization-by-token` (verify token)
8. **UI:** User completes onboarding form (password, name) on onboarding page
9. **API:** `POST /api/v1/users/signupForInvite`
10. **Backend:** `UserService.completeSignUpForInvite()`
    - Verify token and expiration
    - Hash password
    - Create Profile
    - Update User: set password, status to `ACTIVE`, clear token
    - Update OrganizationUser: status to `ACTIVE`, set `joinedAt`
    - Commit transaction
11. **UI:** Redirect to `/authentication/login` or `/dashboard`

**State Transitions:**

- User: `INVITED` → `ACTIVE`
- OrganizationUser: `INVITED` → `ACTIVE`
- TeamUser: `INVITED` → `ACTIVE` (if teams assigned)

**Notifications:**

- Invitation email sent to user

**Error States:**

- Invalid/expired token → 400
- User already exists → 409
- Unauthorized inviter → 403

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/RoleBasedInvite/invite.services.ts:368-554`
- `ab3-medical-api-v2/src/app/modules/User/user.services.ts:136-203`
- `ab3-medical-new/src/app/onboarding-user/page.tsx`
- `ab3-medical-new/src/components/Modals/InviteMedicModal.tsx`
- `ab3-medical-new/src/components/Modals/InviteNewPatientModal.tsx`
- `ab3-medical-new/src/components/Modals/InviteOldPatientModal.tsx`

---

## WORKFLOW 9: Update Own Profile

**Name:** User Updates Own Profile  
**Actors:** SYSTEM_ADMIN, SUPER_ADMIN, ADMIN, MODERATOR, MEDIC, PATIENT, EXTERNAL  
**Trigger:** Any authenticated user updates own profile

**UI Components:**

- **Page:** `/profile` → `ProfilePage`
- **Components:**
  - Profile picture upload section
  - Tabs: "Basic Profile Info", "Account Information"
  - Form fields: firstName, lastName, phoneNumber, homeAddress, DOB, spokenLanguages
  - Account info (read-only): userRole, organization, userId, accountStatus
- **Buttons:** "Edit Profile", "Save", "Cancel"

**Steps:**

1. **UI:** Any authenticated user (SYSTEM_ADMIN, SUPER_ADMIN, ADMIN, MODERATOR, MEDIC, PATIENT, EXTERNAL) navigates to `/profile`
2. **UI:** `ProfilePage` displays:
   - Profile picture (with edit overlay)
   - Basic Information section with tabs
   - Account Information tab (read-only)
3. **UI:** User clicks "Edit Profile" button
4. **UI:** Form becomes editable, user modifies:
   - First Name, Last Name
   - Phone Number
   - Home Address
   - Date of Birth
   - Spoken Languages
5. **UI:** User can upload profile picture (optional) via file input
6. **API:** `PUT /api/v1/users/me` → `UserService.updateMyProfile()`
7. **Backend:**
   - Update Profile fields
   - If profile photo uploaded: Upload to Cloudinary/S3, update `profilePhoto_url`
8. **UI:** Show success, refresh profile view, form returns to read-only mode

**State Transitions:**

- Profile updated
- Profile photo updated (if changed)

**Error States:**

- Invalid data → 400
- File upload failed → 400

**Evidence:**

- `ab3-medical-new/src/app/(dashboard)/profile/page.tsx`
- `ab3-medical-api-v2/src/app/modules/User/user.services.ts:522-560`
- `ab3-medical-api-v2/src/app/modules/User/user.route.ts:47`

---

## WORKFLOW 10: View Patient List & Update Patient Status, Delete Patient

**Name:** View and Search Patient List  
**Actors:** MEDIC, ADMIN, MODERATOR, SUPER_ADMIN, SYSTEM_ADMIN, EXTERNAL  
**Trigger:** User navigates to patient list page

**UI Components:**

- **Page:** `/patients` → `PatientsPageContent.tsx`
- **Components:**
  - `PatientStats.tsx` (patient statistics cards)
  - `PatientRecords.tsx` (patient table with pagination)
  - `TextInputV2.tsx` (search input)
  - `FilterButtonWithDropdown.tsx` (status/sort filters)
- **Table Components:**
  - `PatientTableRow.tsx` (for organization admins/medics)
  - `PatientTableRowSuperAdmin.tsx` (for system admins)
- **Modals:**
  - `InviteNewPatientModal.tsx` (invite new patient form)

**Steps:**

1. **UI:** User navigates to `/patients`
2. **UI:** `PatientsPageContent.tsx` displays stats, search/filter bar, and `PatientRecords.tsx` table.
3. **API:**
   - **Non-External Users:** `GET /api/v1/organization-users` → `GetAllUsersByOrganization()`
   - **External Users:** `GET /api/v1/problem/assigned-external-user/:id` (to filter patients)
   - **System Admins:** `GET /api/v1/users/patients-with-organizations`
4. **UI:** Display patients for organization (or system-wide for SYSTEM_ADMIN). External users see filtered list.

**Per‑Patient Actions:**

1. **Change Status (Activate / Deactivate / Restore):**
   - **Trigger:** User selects status action from row dropdown menu.
   - **API (ORG roles):** `PATCH /api/v1/organization-users/:id/status` → `updateOrganizationUserStatus()`
   - **API (SYSTEM_ADMIN):** `PATCH /api/v1/users/:id` → `updateUser()`
   - **Backend:** Update status (`ACTIVE`, `INACTIVE`, `DELETED` → `ACTIVE`).
   - **UI:** Show success toast, refresh patient list, invalidate stats.
2. **Delete Patient:**
   - **Trigger:** User selects "Delete" from row dropdown menu (only if status is not `DELETED`).
   - **Confirm:** SweetAlert confirmation dialog appears.
   - **API (ORG roles):** `DELETE /api/v1/organization-users/:id` → `deleteOrganizationUser()`
   - **API (SYSTEM_ADMIN):** `DELETE /api/v1/users/:id` → `deleteUser()`
   - **Backend:** Soft or hard delete depending on implementation.
   - **UI:** Show success toast, refresh list, invalidate stats.

**State Transitions:**

- OrganizationUser/User status: `INVITED/PENDING/ACTIVE` ↔ `INACTIVE` / `DELETED`

**Evidence:**

- `ab3-medical-new/src/components/patients/PatientsPageContent.tsx`
- `ab3-medical-new/src/components/Tables/PatientTableRow.tsx`
- `ab3-medical-new/src/components/Tables/PatientTableRowSuperAdmin.tsx`
- `ab3-medical-api-v2/src/app/modules/OrganizationUser/organizationUser.services.ts`
- `ab3-medical-api-v2/src/app/modules/User/user.services.ts`

## WORKFLOW 11: View Patient Profile

**Name:** View Patient Summary and Details  
**Actors:** MEDIC, ADMIN, MODERATOR, SUPER_ADMIN, SYSTEM_ADMIN, EXTERNAL  
**Trigger:** User navigates to patient summary page

**UI Components:**

- **Page:** `/patients/[id]/summary` → `PatientSummaryPage`
- **Views:**
  - `PatientInformation.tsx` (Basic Profile Info, Emergency Contact, NHS Info tabs)
  - `ClinicalSummary.tsx` (for ORGANIZATION role only - shows stats)
  - `PatientRecords.tsx` (tabbed view: Organizations, Problems, Records, Files, Forms)
- **Sub-Views:**
  - `PatientOrganizationRecords.tsx` (organizations table)
  - `PatientProblemRecords.tsx` (problems table)
  - `PatientRecordRecords.tsx` (records/notes table)
  - `PatientFileRecords.tsx` (files table)
  - `PatientFormRecords.tsx` (forms table)

**Steps:**

1. **UI:** MEDIC, ADMIN, MODERATOR, SUPER_ADMIN, SYSTEM_ADMIN, or EXTERNAL navigates to `/patients/[id]/summary`
2. **UI:** Page displays:
   - `PatientInformation.tsx`: Patient basic info, emergency contact, NHS info (tabs)
   - `ClinicalSummary.tsx`: Clinical statistics (only for ORGANIZATION role)
   - `PatientRecords.tsx`: Tabbed records view
3. **API:**
   - `GET /api/v1/users/:id` → Get patient with organizations
   - `GET /api/v1/problems/patient/:patientId` → Get all problems for patient
   - `GET /api/v1/stats/clinical-summary` → Get clinical statistics (if ORGANIZATION role)
4. **UI:** Display patient data, active problems, clinical stats, and records in respective views

**State Transitions:**

- None (read-only view)

**Evidence:**

- `ab3-medical-new/src/app/(dashboard)/patients/[id]/summary/page.tsx`
- `ab3-medical-new/src/components/patients/summary/PatientInformation.tsx`
- `ab3-medical-new/src/components/patients/summary/ClinicalSummary.tsx`
- `ab3-medical-new/src/components/patients/summary/PatientRecords.tsx`

---

## WORKFLOW 12: Update Fitness Status

**Name:** Update Patient Fitness Status  
**Actors:** ADMIN, MODERATOR, SUPER_ADMIN, MEDIC  
**Trigger:** User updates patient fitness status

**Steps:**

1. **UI:** On patient summary page, change fitness status dropdown
2. **UI:** Confirm change in dialog
3. **API:** `PUT /api/v1/users/patient/:id` → `UserService.updatePatient()`
4. **Backend:**
   - Update Profile.fitnessStatus (Fit, In Recovery, Injured)
   - May also update OrganizationUser.fitnessStatus
5. **UI:** Show success, refresh patient data

**State Transitions:**

- Profile.fitnessStatus updated
- OrganizationUser.fitnessStatus may be updated

**Evidence:**

- `ab3-medical-new/src/components/patients/summary/PatientSummary.tsx:52-83`
- `ab3-medical-api-v2/src/app/modules/User/user.services.ts:1154-1394`

---

## WORKFLOW 13: Create Problem (Medical Record)

**Name:** Create Problem for Patient  
**Actors:** MEDIC, ADMIN, MODERATOR, SUPER_ADMIN  
**Trigger:** MEDIC, ADMIN, MODERATOR, or SUPER_ADMIN creates new problem for patient

**UI Components:**

- **Pages:**
  - `/patients/[id]/problems/add-problem` → `AddPatientProblem.tsx`
  - `/patients/add-new-problem` → `AddNewProblemContent.tsx`
- **Components:**
  - `AddPatientProblem.tsx` (problem creation form with tabs)
  - `AddNewProblemContent.tsx` (alternative problem creation flow)
  - `DiagnosisForm.tsx` (diagnosis selection)
  - `SeverityForm.tsx` (severity selection)
  - `VisibilityForm.tsx` (visibility scope selection)
  - `AssignmentForm.tsx` (user/team assignment)
  - `InjuryDetails.tsx`, `IllnessDetails.tsx`, `SurgeryDetails.tsx`, `ScreeningDetails.tsx`, `AllergyDetails.tsx` (type-specific forms)
  - `SearchDiagnosisModal.tsx` (diagnosis search modal)
- **Redirect:** `/patients/[id]/summary` or `/patients/[id]/problems/[problemId]`

**Steps:**

1. **UI:** MEDIC, ADMIN, MODERATOR, or SUPER_ADMIN navigates to:
   - `/patients/[id]/problems/add-problem` (from patient detail page)
   - `/patients/add-new-problem` (standalone problem creation)
2. **UI:** User fills problem form via `AddPatientProblem.tsx`:
   - Problem type selection (Injury, Illness, Surgery, Screening, Allergy, Other)
   - Title, onset date
   - Diagnosis (via `SearchDiagnosisModal.tsx`)
   - Severity (via `SeverityForm.tsx`)
   - Visibility scope (via `VisibilityForm.tsx`)
   - Type-specific details (via `InjuryDetails.tsx`, `IllnessDetails.tsx`, etc.)
   - Assignments (users/teams via `AssignmentForm.tsx`)
   - Initial assessment (optional)
3. **API:** `POST /api/v1/problem` → `ProblemService.createProblem()`
4. **Backend:**
   - Start MongoDB transaction
   - If `visibilityScope === "Public"`: Auto-assign patient's teams to `assignedTeams`
   - Create Problem document
   - If `initialAssessment` provided: Create Note with `isInitialAssessment: true`
   - Commit transaction
   - Send emails to assigned users/teams (async, non-blocking)
5. **UI:** Redirect to `/patients/[id]/summary` or `/patients/[id]/problems/[problemId]` (problem detail page)
6. **Notifications:** Email sent to assigned users about new problem

**State Transitions:**

- Problem created with status `Active`
- Note created (if assessment provided)

**Notifications:**

- Email notifications to assigned users and team members

**Error States:**

- Missing required fields → 400
- Invalid patient/organization → 400
- Transaction rollback on error

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Problem/problem.services.ts:199-272`
- `ab3-medical-new/src/components/patients/problem/AddPatientProblem.tsx`
- `ab3-medical-new/src/components/new-problem/AddNewProblemContent.tsx`
- `ab3-medical-new/src/components/patients/problem/DiagnosisForm.tsx`
- `ab3-medical-new/src/components/Modals/SearchDiagnosisModal.tsx`

---

## WORKFLOW 14: Update Problem Assignments

**Name:** Update Problem Assignments (Users/Teams)  
**Actors:** MEDIC, ADMIN, MODERATOR, SUPER_ADMIN  
**Trigger:** User updates problem assignments

**Steps:**

1. **UI:** Navigate to problem detail page (`/patients/[id]/problems/[problemId]`)
2. **UI:** Modify assignments via `AssignedUser`, `AssignedTeams`, or `AssignedExternalUser` components
3. **API:** `PUT /api/v1/problem/:id` → `ProblemService.updateProblem()`
4. **Backend:**
   - Fetch existing problem
   - Update problem document (only assignment fields: `assignedUsers`, `assignedTeams`, `externalUsers`)
   - Detect newly assigned users/teams
   - Send emails to newly assigned users/teams (async)
5. **UI:** Show success, refresh problem view

**Note:** Full problem update (title, status, problemType, etc.) is NOT available in the frontend. Only assignment updates are supported. Problem status update functionality is commented out in `ViewInitialProblemDetails.tsx`.

**State Transitions:**

- Problem assignments updated

**Notifications:**

- Email sent to newly assigned users/teams

**Error States:**

- Problem not found → 404
- Invalid data → 400

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Problem/problem.services.ts:382-441`
- `ab3-medical-new/src/components/patients/problem/single/AssignedUser.tsx`
- `ab3-medical-new/src/components/patients/problem/single/AssignedTeams.tsx`
- `ab3-medical-new/src/components/patients/problem/single/AssignedExternalUser.tsx`
- `ab3-medical-new/src/components/patients/problem/single/ViewInitialProblemDetails.tsx` (commented out status update)

---

## WORKFLOW 15: Delete Problem

**Name:** Delete Problem  
**Actors:** MEDIC, ADMIN, MODERATOR, SUPER_ADMIN  
**Trigger:** User deletes problem

**Steps:**

1. **UI:** Open menu of problem tables, click "Delete"
2. **API:** `DELETE /api/v1/problem/:id` → `ProblemService.deleteProblem()`
3. **Backend:**
   - Delete Problem document
   - Associated Notes may remain (check cascade rules)
4. **UI:** Show success, redirect to problems list

**State Transitions:**

- Problem deleted

**Error States:**

- Problem not found → 404
- Unauthorized → 403

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Problem/problem.route.ts:27`

---

## WORKFLOW 16: Add Record (Note) to Problem

**Name:** Create Clinical Record (Note)  
**Actors:** MEDIC, ADMIN, MODERATOR, SUPER_ADMIN  
**Trigger:** MEDIC, ADMIN, MODERATOR, or SUPER_ADMIN adds record to problem

**UI Components:**

- **Page:** `/patients/[id]/problems/[problemId]` → Problem detail page
- **View:** `ProblemNotes.tsx` (shows "Records" heading and "Add Record" button)
- **Modal:** `AddNoteModal.tsx` (record creation form with rich text editor)
- **Components:**
  - Rich text editor for note details
  - File upload for attachments
  - User mention functionality
  - Visibility toggles (`isDoctorOnly`, `isVisibleToPatient`)

**Steps:**

1. **UI:** MEDIC, ADMIN, MODERATOR, or SUPER_ADMIN opens problem detail page (`/patients/[id]/problems/[problemId]`)
2. **UI:** User views `ProblemNotes.tsx` component, clicks "Add Record" button
3. **UI:** Modal opens (`AddNoteModal.tsx`), user fills record form:
   - Note details (rich text editor)
   - Attachments (file upload)
   - Mentions (tag users)
   - Visibility settings
4. **API:** `POST /api/v1/notes` → `NoteService.createNote()`
5. **Backend:**
   - Create Note document
   - Link to problem, patient, organization
   - Set `seenBy` to creator
   - If mentions provided: Process mentions (send emails)
6. **UI:** Close modal, refresh records list in `ProblemNotes.tsx` (shown as "Records" section)
7. **Notifications:** Email sent to mentioned users

**Terminology Note:** In the frontend UI, "notes" are consistently referred to as "records" (e.g., "Add Record" button, "Records" heading, "Patient Records" tab). The backend API and models use "Note" terminology.

**State Transitions:**

- Note (record) created
- `seenBy` array initialized with creator

**Notifications:**

- Mention emails sent to mentioned users

**Error States:**

- Missing note details → 400
- Invalid problem/patient → 400

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Note/note.services.ts:51-812`
- `ab3-medical-new/src/components/Modals/AddNoteModal.tsx`
- `ab3-medical-new/src/components/patients/problem/single/ProblemNotes.tsx` (shows "Records" heading and "Add Record" button)

---

## WORKFLOW 17: Reply to Record (Note)

**Name:** Reply to Clinical Record (Note)  
**Actors:** MEDIC, ADMIN, MODERATOR, SUPER_ADMIN  
**Trigger:** User replies to a record

**Steps:**

1. **UI:** User clicks reply on record (note), `NoteReplyEditor` opens
2. **UI:** User types reply, can mention users
3. **API:** `POST /api/v1/note-replies` → `NoteReplyService.createNoteReply()`
4. **Backend:**
   - Create NoteReply document
   - Link to note, set `replyTo` user
   - Set `seenBy` to creator
   - Process mentions (send emails)
5. **UI:** Close editor, refresh record replies

**Terminology Note:** In the frontend UI, "notes" are referred to as "records", but replies are still called "replies" in the UI.

**State Transitions:**

- NoteReply created
- `seenBy` array initialized

**Notifications:**

- Mention emails sent

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/NoteReply/noteReply.services.ts`
- `ab3-medical-new/src/components/TextEditor/NoteReplyEditor.tsx`

---

## WORKFLOW 18: Update Record (Note)

**Name:** Update Clinical Record (Note)  
**Actors:** Record creator  
**Trigger:** User updates own record

**Steps:**

1. **UI:** Open record (referred to as "record" in frontend UI), click "Edit"
2. **UI:** Modify record details, attachments, mentions via `EditNoteEditor`
3. **API:** `PUT /api/v1/notes/:noteId` → `NoteService.updateNote()`
4. **Backend:**
   - Verify user is note creator (via `verifyNoteCreator` middleware)
   - Update Note document
   - Process new mentions (send emails)
5. **UI:** Show success, refresh records list

**Terminology Note:** In the frontend, "notes" are referred to as "records" in the UI (e.g., "Add Record", "Patient Records", `/patients/[id]/records/[recordId]`).

**State Transitions:**

- Note updated

**Notifications:**

- Email sent to newly mentioned users

**Error States:**

- Note not found → 404
- Not creator → 403
- Invalid data → 400

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Note/note.route.ts:17`
- `ab3-medical-api-v2/src/app/middlewar/verifyNoteCreator.ts`
- `ab3-medical-new/src/components/TextEditor/EditNoteEditor.tsx`
- `ab3-medical-new/src/components/patients/problem/single/ProblemNotes.tsx` (shows "Records" heading)
- `ab3-medical-new/src/app/(dashboard)/patients/[id]/records/[recordId]/page.tsx`

---

## WORKFLOW 19: Mark Note as Seen

**Name:** Mark Note as Read/Seen  
**Actors:** Users viewing notes  
**Trigger:** User views note

**Steps:**

1. **UI:** User opens note (problem detail page, patient records)
2. **API:** `PUT /api/v1/notes/:noteId/seen` → `NoteService.updateSeen()`
3. **Backend:**
   - Add user ID to Note `seenBy` array (if not already present)
   - Update Note document
4. **UI:** Note marked as seen (visual indicator)

**State Transitions:**

- User added to `Note.seenBy[]` array

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Note/note.route.ts:18`
- `ab3-medical-api-v2/src/app/modules/Note/note.services.ts`

---

## WORKFLOW 20: Delete Record (Note)

**Name:** Delete Record  
**Actors:** Note creator only  
**Trigger:** User deletes own record

**Steps:**

1. **UI:** Open Menu from three dot of the record, click "Delete"
2. **API:** `DELETE /api/v1/notes/:noteId` → `NoteService.deleteNote()`
3. **Backend:**
   - Verify user is note creator (via `verifyNoteCreator` middleware)
   - Delete Note document
   - Delete associated NoteReplies (cascade)
4. **UI:** Show success, refresh notes list

**State Transitions:**

- Note deleted
- NoteReplies deleted (cascade)

**Error States:**

- Note not found → 404
- Not creator → 403

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Note/note.route.ts:19`
- `ab3-medical-api-v2/src/app/middlewar/verifyNoteCreator.ts`

---

## WORKFLOW 21: View Patient Records (Notes) (Records Tab -> View Record)

**Name:** View Patient Clinical Records (Notes)  
**Actors:** MEDIC, ADMIN, MODERATOR, SUPER_ADMIN, SYSTEM_ADMIN, EXTERNAL  
**Trigger:** MEDIC, ADMIN, MODERATOR, SUPER_ADMIN, SYSTEM_ADMIN, or EXTERNAL views patient records

**UI Components:**

- **Page:** `/patients/[id]/summary` → Patient summary page
- **View:** `PatientRecordRecords.tsx` (records table)
- **Table Components:**
  - `PatientRecordsTableHeader.tsx` (table header)
  - `PatientRecordsTableRow.tsx` (table rows)
  - `TableScrollGrabber.tsx` (horizontal scroll)
  - `Pagination.tsx` (pagination controls)
- **Page (Detail):** `/patients/[id]/records/[recordId]` → `RecordDetailsPage`
- **Component:** `NoteCard.tsx` (displays record details)

**Steps:**

1. **UI:** MEDIC, ADMIN, MODERATOR, SUPER_ADMIN, SYSTEM_ADMIN, or EXTERNAL navigates to `/patients/[id]/summary`, clicks "Records" tab
2. **UI:** `PatientRecordRecords.tsx` displays records table with:
   - Search bar (search by type, problem)
   - Columns: Consultation Date, Type/Category, Problem (linked), Created By, Actions
   - "Add New Record" button
3. **API:** `GET /api/v1/notes/patient/:patientId` → `NoteService.getAllNotesByPatient()`
4. **Backend:**
   - Fetch all notes for patient
   - Apply filters (search, sort, pagination)
   - Return notes with populated fields
5. **UI:** Display records in table (`PatientRecordsTableRow.tsx`), allow filtering and pagination
6. **UI:** User clicks record row → Navigate to `/patients/[id]/records/[recordId]`
7. **API:** `GET /api/v1/notes/:noteId` → Get single note
8. **UI:** `RecordDetailsPage` displays record details via `NoteCard.tsx` component

**Terminology Note:** In the frontend UI, "notes" are consistently referred to as "records" (e.g., "Add Record", "Patient Records", "Loading records...").

**State Transitions:**

- None (read-only view)

**Evidence:**

- `ab3-medical-new/src/components/patients/summary/PatientRecordRecords.tsx`
- `ab3-medical-new/src/app/(dashboard)/patients/[id]/records/[recordId]/page.tsx`
- `ab3-medical-new/src/components/patients/problem/single/ProblemNotes.tsx` (shows "Records" heading)
- `ab3-medical-new/src/components/Tables/PatientRecordsTableHeader.tsx`
- `ab3-medical-new/src/components/Tables/PatientRecordsTableRow.tsx`
- `ab3-medical-new/src/components/Cards/NoteCard.tsx`
- `ab3-medical-api-v2/src/app/modules/Note/note.services.ts:201-230`

---

## WORKFLOW 22: File Upload (Files Tab)

**Name:** Upload File (Note Attachment or Standalone)  
**Actors:** Authenticated users  
**Trigger:** User uploads file

**Steps:**

1. **UI:** User selects file (via `UploadInput.tsx` or file upload button)
2. **API:** `POST /api/v1/uploads/upload` (Cloudinary) or `POST /api/v1/aws-uploads/single` (S3)
3. **Backend:**
   - Validate file (size, type)
   - Upload to Cloudinary or S3
   - If S3: Create AwsUpload record in DB
   - Return file URL
4. **UI:** File URL stored in form field or note attachment array

**Storage Options:**

- Cloudinary: Direct upload, returns URL
- AWS S3: Direct upload or presigned URL upload

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Upload/upload.services.ts`
- `ab3-medical-api-v2/src/app/modules/Upload/aws-upload.services.ts`

---

## WORKFLOW 23: View SCAT6 Assessment (Forms Tab)

**Name:** View SCAT6 Concussion Assessment  
**Actors:** Authenticated users (with access)  
**Trigger:** User views SCAT6 form

**Steps:**

1. **UI:** Navigate to `/patients/[id]/forms/view-scat-6/[formId]`
2. **API:** `GET /api/v1/scat6/:id` → `Scat6Service.getScat6()`
3. **Backend:**
   - Fetch Scat6 document
   - All encrypted fields automatically decrypted via Mongoose getters
4. **UI:** Display SCAT6 form with all steps (1-6, 15)
5. **UI:** Show encrypted data (decrypted for display)

**Data Displayed:**

- Athlete information (name, DOB, etc.)
- Step 1: Background
- Step 2: Symptom Evaluation
- Step 3: Cognitive Assessment
- Step 4: Coordination & Balance
- Step 5: Delayed Recall
- Step 6: Decision
- Step 15: Observable Signs & GCS

**Evidence:**

- `ab3-medical-new/src/app/(dashboard)/patients/[id]/forms/view-scat-6/[formId]/page.tsx`
- `ab3-medical-api-v2/src/app/modules/Scat6/scat6.model.ts` (decryption via getters)

---

## WORKFLOW 24: Create SCAT6 Assessment

**Name:** Complete SCAT6 Concussion Assessment  
**Actors:** MEDIC, ADMIN, MODERATOR, SUPER_ADMIN  
**Trigger:** User creates SCAT6 form for patient

**Steps:**

1. **UI:** Navigate to `/patients/[id]/forms/add-scat-6`
2. **UI:** Multi-step form (Steps 1-6, Step 15)
3. **API:** `POST /api/v1/scat6` → `Scat6Service.createScat6()`
4. **Backend:**
   - Create Scat6 document
   - All fields encrypted at rest (via Mongoose getters/setters)
   - Store step data as JSON
5. **UI:** Redirect to view SCAT6 or patient forms list

**State Transitions:**

- SCAT6 document created

**Data Encryption:**

- All athlete information encrypted (name, DOB, scores, etc.)

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Scat6/scat6.model.ts:330-687`
- `ab3-medical-new/src/app/(dashboard)/patients/[id]/forms/add-scat-6/page.tsx`

---

## WORKFLOW 25: View Team List

**Name:** View Team List  
**Actors:** SUPER_ADMIN, ADMIN, MODERATOR, MEDIC  
**Trigger:** User navigates to teams page or organization teams tab

**UI Components:**

- **Page:** `/teams` → `TeamsPageContent.tsx`
- **Stats:** `TeamStats.tsx`
- **Table:** `TeamRecords.tsx` → `TeamTableRow.tsx`

**Steps:**

1. **UI:** Admin/Medic navigates to `/teams` (Global) or `/organizations/[id]` (Teams tab).
2. **UI:** `TeamsPageContent.tsx` or `OrganizationTeams.tsx` displays stats, search inputs, and the teams table.
3. **API:**
   - Global (SYSTEM_ADMIN): `GET /api/v1/teams/all-teams` → `getAllTeams()`
   - Organization: `GET /api/v1/teams/org-teams` → `getOrgTeams()`
   - My Teams (MEDIC): `GET /api/v1/teams/my-teams` → `getUserTeams()`
4. **UI:** User can use search bar and filter by status to view specific teams.

**State Transitions:**

- None (read-only view)

**Evidence:**

- `ab3-medical-new/src/components/teams/TeamsPageContent.tsx`
- `ab3-medical-new/src/components/teams/TeamRecords.tsx`
- `ab3-medical-api-v2/src/app/modules/Team/team.services.ts`

---

## WORKFLOW 26: Create Team

**Name:** Create Team in Organization  
**Actors:** SUPER_ADMIN, ADMIN  
**Trigger:** Admin creates team

**Steps:**

1. **UI:** Navigate to `/teams` or organization detail, click "Create Team"
2. **UI:** Fill team form (name, logo)
3. **API:** `POST /api/v1/teams` → `TeamService.createTeam()` or `createOrgTeam()`
4. **Backend:**
   - Check if team name already exists in organization
   - Create Team document with status `ACTIVE`
   - Set `createdBy` to current user
5. **UI:** Show success, refresh teams list

**State Transitions:**

- Team created with status `ACTIVE`

**Error States:**

- Team name already exists → 400/409
- Unauthorized → 403

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Team/team.services.ts:14-80`
- `ab3-medical-new/src/components/Modals/CreateOrganizationTeamModal.tsx`

---

## WORKFLOW 27: Update Team

**Name:** Update Team Details  
**Actors:** SUPER_ADMIN  
**Trigger:** SUPER_ADMIN updates team

**Steps:**

1. **UI:** Navigate to team detail, click "Edit"
2. **UI:** Modify team fields (name, logo)
3. **API:** `PUT /api/v1/teams/:id` → `TeamService.updateTeam()`
4. **Backend:**
   - Verify team exists
   - Check authorization (SUPER_ADMIN only)
   - Update Team document
5. **UI:** Show success, refresh team view

**State Transitions:**

- Team updated

**Error States:**

- Team not found → 404
- Unauthorized → 403
- Duplicate name → 400

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Team/team.services.ts:159-175`

---

## WORKFLOW 28: Change Team Status

**Name:** Activate/Deactivate Team  
**Actors:** SUPER_ADMIN  
**Trigger:** SUPER_ADMIN changes team status

**Steps:**

1. **UI:** Navigate to team, change status dropdown
2. **API:** `PUT /api/v1/teams/:id/status` → `TeamService.changeTeamStatus()`
3. **Backend:**
   - Verify team exists
   - Check authorization (SUPER_ADMIN only)
   - Update Team.status (`ACTIVE` ↔ `INACTIVE`)
4. **UI:** Show success, refresh team view

**State Transitions:**

- Team status: `ACTIVE` ↔ `INACTIVE`

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Team/team.services.ts:187-214`

---

## WORKFLOW 29: Delete Team

**Name:** Delete Team  
**Actors:** SUPER_ADMIN  
**Trigger:** SUPER_ADMIN deletes team

**Steps:**

1. **UI:** Navigate to team, click "Delete"
2. **API:** `DELETE /api/v1/teams/:id` → `TeamService.deleteTeam()`
3. **Backend:**
   - Start MongoDB transaction
   - Verify team exists and belongs to organization
   - Delete all TeamUser documents for this team
   - Delete Team document
   - Commit transaction
4. **UI:** Show success, redirect to teams list

**State Transitions:**

- Team deleted
- All TeamUser entries deleted (cascade)

**Error States:**

- Team not found → 404
- Unauthorized → 403

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Team/team.services.ts:216-241`

---

## WORKFLOW 30: View Team Members

**Name:** View Team Members List  
**Actors:** Authenticated users  
**Trigger:** User navigates to team members page

**Steps:**

1. **UI:** Navigate to `/teams/[teamId]/members`
2. **API:** `GET /api/v1/teams/:id/members` → `TeamService.getTeamMembers()`
3. **Backend:**
   - Fetch Team document
   - Query TeamUser with filters (userType, status, search)
   - Populate User and Profile
   - Apply pagination, sorting
4. **UI:** Display team members table with filters

**Filters Available:**

- userType (MEDIC, PATIENT, etc.)
- status (ACTIVE, INACTIVE, PENDING)
- search (name, email)
- sortBy, sortOrder
- pagination

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Team/team.services.ts:245-433`
- `ab3-medical-new/src/app/(dashboard)/teams/[teamId]/members/page.tsx`

---

## WORKFLOW 31: Assign Member to Team

**Name:** Add User to Team  
**Actors:** SUPER_ADMIN, ADMIN  
**Trigger:** Admin assigns user to team

**Steps:**

1. **UI:** Navigate to `/teams/[teamId]/members`, click "Add Member"
2. **UI:** Select user from organization
3. **API:** `POST /api/v1/teams/assign-member` → `TeamService.assignMemberToTeam()`
4. **Backend:**
   - Start MongoDB transaction
   - Verify user exists
   - Verify OrganizationUser exists for this org
   - Verify team belongs to organization
   - Check if TeamUser already exists
   - Create TeamUser with status `ACTIVE`
   - Commit transaction
5. **UI:** Show success, refresh team members list

**State Transitions:**

- TeamUser created with status `ACTIVE`

**Error States:**

- User not found → 404
- User already in team → 409
- Team not in organization → 400

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Team/team.services.ts:435-507`

---

## WORKFLOW 32: Remove Team Member

**Name:** Remove User from Team  
**Actors:** SUPER_ADMIN, ADMIN  
**Trigger:** Admin removes user from team

**Steps:**

1. **UI:** Navigate to `/teams/[teamId]/members`, click "Remove" on member
2. **API:** `DELETE /api/v1/teams/members/:teamMemberId` → `TeamService.removeMemberFromTeam()`
3. **Backend:**
   - Delete TeamUser document
4. **UI:** Show success, refresh team members list

**State Transitions:**

- TeamUser deleted

**Error States:**

- TeamMember not found → 400

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Team/team.services.ts:531-545`

---

## WORKFLOW 33: Manage Medics (List, Invite, Edit, Status & Delete)

**Name:** Manage Medics — List, Invite, Edit, Change Status, Delete  
**Actors:** SYSTEM_ADMIN, SUPER_ADMIN, ADMIN, MODERATOR  
**Trigger:** User navigates to medics page, invites a new medic, edits an existing medic, or changes medic status

**UI Components:**

- **Page:** `/medics` → `MedicsPageContent.tsx`
- **Table Components:**
  - `MedicRecords.tsx` (main table wrapper)
  - `MedicTableHeader.tsx` (table header)
  - `MedicTableRow.tsx` (table rows with per-row action menu)
  - `TableScrollGrabber.tsx` (horizontal scroll)
  - `Pagination.tsx` (pagination controls)
- **Stats:** `MedicStats.tsx` (top stats cards)
- **Modals:**
  - `InviteMedicModal.tsx` (invite medic)
  - `MedicDetailsModal.tsx` (view medic details)
  - `EditMedicModal.tsx` (edit medic)
- **Filters:** Search, status filter, sort by, sort order
- **Status Badge & Actions:**
  - `MedicsStatusBadge.tsx` (shows ACTIVE / INACTIVE / INVITED / DELETED)
  - `DropDownMenu.tsx` inside `MedicTableRow.tsx` (View, Edit, Change Status, Delete)
  - **Also used in:** Organization details → `OrganizationRecords.tsx` → **“Medics”** tab (via `OrganizationMedics.tsx`)

**Steps:**

1. **UI:** SYSTEM_ADMIN, SUPER_ADMIN, ADMIN, or MODERATOR navigates to `/medics`
2. **UI:** `MedicsPageContent.tsx` renders:
   - Stats cards via `MedicStats.tsx`
   - Search input for medics
   - Status and sort filters (`FilterButtonWithDropdown`)
   - "Invite New Medic" button (opens `InviteMedicModal.tsx`)
3. **Invite New Medic (from header button):**
   - **UI:** Click "Invite New Medic" → `InviteMedicModal.tsx` opens
   - **UI:** Fill medic details (name, email, role, teams) and submit
   - **API:** `POST /api/v1/invites/invite-org-user` with `role = MEDIC`
   - **Backend:** Creates invited medic user and organizationUser, optionally attaches to teams, sends invite email
   - **UI:** Close modal and refetch medics list (`refetchMedics` / `refetchAllMedics`)
4. **List & Load:**
   - **API (SYSTEM_ADMIN):** `useGetAllUsersQuery({ role: MEDIC, ... })` → fetch all medics
   - **API (ORG roles):** `useGetAllUsersByOrganizationQuery({ organizationId, role: MEDIC, ... })` → fetch medics for organization
   - **UI:** `MedicRecords.tsx` renders rows using `MedicTableRow.tsx`
5. **Per‑Medic Actions (from action menu on each row):**
   - **View Medic (opens details modal):**
     - **UI:** Open row action menu → click "View"
     - **UI:** `MedicTableRow.tsx` constructs `IMedicData` from row props and calls `onViewMedic(medicData)`
     - **UI:** `MedicsPageContent.tsx` sets `selectedMedic` and opens `MedicDetailsModal.tsx`
     - **UI:** Modal shows full medic profile: name, email, phone, DOB, address, license, medic type, status
   - **Edit Medic (via modal from action menu):**
     - **UI:** Open row action menu → click "Edit"
     - **UI:** `MedicTableRow.tsx` constructs `IMedicData` (using `userId`) and calls `onEditMedic(medicData)`
     - **UI:** `MedicsPageContent.tsx` sets `selectedMedic` and opens `EditMedicModal.tsx`
     - **UI:** In `EditMedicModal.tsx`, user edits fields (name, email, phone, address, medic type, license) and submits
     - **API:** `PATCH /api/v1/users/:id` (via `useUpdateUserMutation`) and/or org-user update depending on accessRole
     - **UI:** Close modal, refetch medics list, invalidate stats via `statsApi.util.invalidateTags(["stats"])`
   - **Change Status (enable / disable / restore):**
     - **UI:** Open row action menu → click status-related item (derived from current status)
     - **UI:** `MedicTableRow.tsx` calls `handleStatusChange()`
     - **Logic:**
       - `ACTIVE` → `INACTIVE` (disable medic)
       - `INACTIVE` / `PENDING` / `INVITED` → `ACTIVE` (enable medic)
       - `DELETED` → `ACTIVE` (restore medic)
     - **API (SYSTEM_ADMIN):** `updateUser({ id: medicId, body: { status: newStatus } })`
     - **API (ORG roles):** `updateOrganizationUserStatus({ id: medicId, status: newStatus })`
     - **UI:** Show toast ("Medic activated/deactivated/restored successfully!"), update status badge, invalidate stats and `organizationUsers` / `organizations` caches
   - **Delete Medic (from action menu):**
     - **UI:** Open row action menu → click "Delete"
     - **UI:** SweetAlert confirmation dialog appears ("Are you sure? You won't be able to revert this!")
     - **API (SYSTEM_ADMIN):** `deleteUser(medicId)` via `useDeleteUserMutation`
     - **API (ORG roles):** `deleteOrganizationUser(medicId)` via `useDeleteOrganizationUserMutation`
     - **Backend:** Deletes or soft-deletes medic record (implementation in User / OrganizationUser services)
     - **UI:** On success, toast "Medic deleted successfully!", invalidate stats and organization user caches to refresh list
6. **UI:** User can search, filter by status, sort, and paginate through medics

**State Transitions:**

- Medic created in INVITED/PENDING/ACTIVE state (via invite)
- Medic status updated: `ACTIVE` ↔ `INACTIVE` / `INVITED` / `PENDING` / `DELETED`
- Medic deleted (soft or hard delete depending on role/API)

**Evidence:**

- `ab3-medical-new/src/app/(dashboard)/medics/page.tsx`
- `ab3-medical-new/src/components/medics/MedicsPageContent.tsx`
- `ab3-medical-new/src/components/medics/MedicRecords.tsx`
- `ab3-medical-new/src/components/Tables/MedicTableHeader.tsx`
- `ab3-medical-new/src/components/Tables/MedicTableRow.tsx`
- `ab3-medical-new/src/components/Modals/InviteMedicModal.tsx`
- `ab3-medical-new/src/components/Modals/MedicDetailsModal.tsx`
- `ab3-medical-new/src/components/Modals/EditMedicModal.tsx`

---

## WORKFLOW 34: Manage Organization Users (List, Invite, Status & Delete)

**Name:** Manage Organization Users — List, Invite, Change Status, Delete  
**Actors:** SUPER_ADMIN, ADMIN  
**Trigger:** SUPER_ADMIN or ADMIN navigates to org users page, invites a new org user, or changes org user status

**UI Components:**

- **Page:** `/users` → `UsersPage.tsx`
- **Table Components:**
  - `UserRecords.tsx` (main table wrapper)
  - `UserTableHeader.tsx` (table header)
  - `UserTableRow.tsx` (table rows)
  - `TableScrollGrabber.tsx` (horizontal scroll)
- **Stats:** `UserStats.tsx` (top stats cards)
- **Modals:**
  - `InviteUserModal.tsx` (invite new org user)
- **Filters:** Search, status filter, sort by, sort order
- **Status Badge & Actions:**
  - `UsersStatusBadge.tsx` (shows org user status)
  - `DropDownMenu.tsx` inside `UserTableRow.tsx` (Change Status, Delete)
  - **Also used in:** Organization details → `OrganizationRecords.tsx` → **“Admins”** tab (via `OrganizationAdmins.tsx`)

**Steps:**

1. **UI:** SUPER_ADMIN or ADMIN navigates to `/users`
2. **UI:** `UsersPage.tsx` renders:
   - Stats cards via `UserStats.tsx`
   - Search input for org users
   - Status and sort filters (`FilterButtonWithDropdown`)
   - "Invite New Org User" button (opens `InviteUserModal.tsx`)
3. **Invite New Org User:**
   - **UI:** Click "Invite New Org User" → `InviteUserModal.tsx` opens
   - **UI:** Fill org user details (name, email, role = ADMIN/MODERATOR) and submit
   - **API:** `POST /api/v1/invites/invite-org-user` with `role` in `[ADMIN, MODERATOR]`
   - **Backend:** Creates invited user and `OrganizationUser` entry, sends invite email
   - **UI:** Close modal and refetch org users (`useGetAllUsersByOrganizationQuery` invalidated via `organizationUserApi.util.invalidateTags(["organizationUsers"])`)
4. **List & Load:**
   - **API:** `useGetAllUsersByOrganizationQuery({ organizationId, roles: [ADMIN, MODERATOR], ... })` → fetch org users for organization
   - **UI:** `UserRecords.tsx` displays table:
     - Columns: Name, User Type (ADMIN/MODERATOR), Status, Last Updated
     - Rows rendered via `UserTableRow.tsx`
5. **Per‑Org User Actions (from action menu):**
   - **Change Status (enable / disable / restore):**
     - **UI:** Open row action menu → item text dynamically shows "Activate", "Deactivate", or "Restore"
     - **UI:** `UserTableRow.tsx` calls `handleStatusChange()`
     - **Logic:**
       - `ACTIVE` → `INACTIVE`
       - `INACTIVE` / `PENDING` / `INVITED` → `ACTIVE`
       - `DELETED` → `ACTIVE`
     - **API:** `updateOrganizationUserStatus({ id: orgUserId, status: newStatus })`
     - **UI:** Toast shows "Medic activated/deactivated/restored successfully!" (label bug in code), invalidate `organizationUsers`, `organizations`, and `stats` caches
   - **Delete Org User:**
     - **UI:** Open row action menu → click "Delete"
     - **UI:** SweetAlert confirmation dialog appears ("You won't be able to revert this!")
     - **API:**
       - `DELETE /api/v1/users/:id` via `deleteUser(userId)`
       - `DELETE /api/v1/organization-users/:id` via `deleteOrganizationUser(orgUserId)`
     - **UI:** On success, toast "User deleted successfully!", invalidate `organizationUsers` and `stats` tags to refresh list
6. **UI:** User can search, filter by status, and sort org users

**State Transitions:**

- Org user created in INVITED/PENDING/ACTIVE state (via invite)
- Org user status updated: `ACTIVE` ↔ `INACTIVE` / `INVITED` / `PENDING` / `DELETED`
- Org user deleted (both `User` and `OrganizationUser` removed)

**Evidence:**

- `ab3-medical-new/src/app/(dashboard)/users/page.tsx`
- `ab3-medical-new/src/components/users/UserRecords.tsx`
- `ab3-medical-new/src/components/Tables/UserTableHeader.tsx`
- `ab3-medical-new/src/components/Tables/UserTableRow.tsx`
- `ab3-medical-new/src/components/Modals/InviteUserModal.tsx`

---

## WORKFLOW 35: Manage System Users (List, Create, Invite, Status & Delete)

**Name:** Manage System Users — List, Create, Invite, Change Status, Delete  
**Actors:** SYSTEM_ADMIN  
**Trigger:** SYSTEM_ADMIN navigates to system users page, creates a new system user, invites a system user, or changes system user status

**UI Components:**

- **Page:** `/system-users` → `SystemUsersPage.tsx`
- **Table Components:**
  - `SystemUserRecords.tsx` (main table wrapper)
  - `SystemUserTableHeader.tsx` (table header)
  - `SystemUserTableRow.tsx` (table rows)
  - `TableScrollGrabber.tsx` (horizontal scroll)
- **Stats:** `SystemUserStats.tsx` (top stats cards)
- **Modals:**
  - `CreateSystemUserModal.tsx` (create new system user)
  - `InviteSystemUserModal.tsx` (invite system user)
- **Filters:** Search, status filter, sort by, sort order
- **Status Badge & Actions:**
  - `UsersStatusBadge.tsx` (shows system user status)
  - `DropDownMenu.tsx` inside `SystemUserTableRow.tsx` (Activate, Deactivate, Restore, Delete)

**Steps:**

1. **UI:** SYSTEM_ADMIN navigates to `/system-users`
2. **UI:** `SystemUsersPage.tsx` renders:
   - Stats cards via `SystemUserStats.tsx`
   - Search input for system users
   - Status and sort filters (`FilterButtonWithDropdown`)
   - "Create New System User" button (opens `CreateSystemUserModal.tsx`)
   - "Invite System User" button (opens `InviteSystemUserModal.tsx`)
3. **Create System User:**
   - **UI:** Click "Create New System User" → `CreateSystemUserModal.tsx` opens
   - **UI:** Fill fields (email, name, role, password) and submit
   - **API:** `useCreateSystemUserMutation` → creates active system-level user (ADMIN/MODERATOR) with `accessRole = SYSTEM_ADMIN`
   - **UI:** Close modal, refetch system users, invalidate `users` and `stats` tags
4. **Invite System User:**
   - **UI:** Click "Invite System User" → `InviteSystemUserModal.tsx` opens
   - **UI:** Fill user details (email, name, role) and submit
   - **API:** `useCreateSystemUserMutation` / invite endpoint to create INVITED system user
   - **UI:** Close modal, refetch system users
5. **List & Load:**
   - **API:** `useGetAllUsersQuery({ roles: [ADMIN, MODERATOR], accessRole: SYSTEM_ADMIN, ... })` → fetch all system users
   - **UI:** `SystemUserRecords.tsx` displays table:
     - Columns: Name, User Type, Status, Last Updated, Actions
     - Rows rendered via `SystemUserTableRow.tsx`
6. **Per‑System User Actions (from action menu):**
   - **Deactivate:**
     - **UI:** If status is `ACTIVE`, menu shows "Deactivate"
     - **UI:** `handleDeactivateUser()` updates status to `INACTIVE`
     - **API:** `updateUser({ id: userId, body: { status: "INACTIVE" } })`
     - **UI:** Toast "User deactivated successfully!", invalidate `users` and `stats` caches
   - **Activate:**
     - **UI:** If status is `INACTIVE`, menu shows "Activate"
     - **UI:** `handleActivateUser()` sets status to `ACTIVE`
     - **API:** `updateUser({ id: userId, body: { status: "ACTIVE" } })`
     - **UI:** Toast "User activated successfully!", invalidate caches
   - **Restore:**
     - **UI:** If status is `DELETED`, menu shows "Restore"
     - **UI:** `handleRestoreUser()` sets status to `ACTIVE`
     - **API:** `updateUser({ id: userId, body: { status: "ACTIVE" } })`
     - **UI:** Toast "User restored successfully!", invalidate caches
   - **Delete:**
     - **UI:** Menu shows "Delete" when status is not `DELETED`
     - **UI:** SweetAlert confirmation ("You want to delete this user?")
     - **API:** `deleteUser(userId)` via `useDeleteUserMutation`
     - **UI:** Toast "User deleted successfully!", invalidate `users` and `stats` tags
7. **UI:** SYSTEM_ADMIN can search, filter, and sort system users

**State Transitions:**

- System user created as `ACTIVE` (create) or `INVITED` (invite)
- System user status updated: `ACTIVE` ↔ `INACTIVE` ↔ `DELETED`

**Evidence:**

- `ab3-medical-new/src/app/(dashboard)/system-users/page.tsx`
- `ab3-medical-new/src/components/users/SystemUserRecords.tsx`
- `ab3-medical-new/src/components/Tables/SystemUserTableHeader.tsx`
- `ab3-medical-new/src/components/Tables/SystemUserTableRow.tsx`
- `ab3-medical-new/src/components/Modals/CreateSystemUserModal.tsx`
- `ab3-medical-new/src/components/Modals/InviteSystemUserModal.tsx`

---

## WORKFLOW 36: Manage External Users (List, Invite, Edit, Status & Delete)

**Name:** Manage External Users — List, Invite, View, Edit, Change Status, Delete  
**Actors:** SYSTEM_ADMIN, SUPER_ADMIN, ADMIN, MODERATOR  
**Trigger:** SYSTEM_ADMIN or organization admins navigate to external users page, invite an external user, or view/edit/delete an external user

**UI Components:**

- **Page:** `/external` → `ExternalPage.tsx`
- **Table Components:**
  - `ExternalRecord.tsx` (main table wrapper)
  - `ExternalTableHeader.tsx` (table header)
  - `ExternalTableRow.tsx` (table rows)
  - `TableScrollGrabber.tsx` (horizontal scroll)
  - `Pagination.tsx` (pagination controls)
- **Stats:** `StatCardV2.tsx` cards on top of page
- **Modals:**
  - `InviteExternalModal.tsx` (invite external user)
  - `ExternalDetailsModal.tsx` (view external user details)
  - `EditExternalModal.tsx` (edit external user)
- **Filters:** Search, status filter, sort by, sort order
- **Actions:** `ExternalTableRow.tsx` includes View, Edit, Delete and uses a status badge pattern (via `MedicsStatusBadge` import)
  - **Also used in:** Organization details → `OrganizationRecords.tsx` → **“Externals”** tab (via `OrganizationExternals.tsx`)

**Steps:**

1. **UI:** SYSTEM_ADMIN, SUPER_ADMIN, ADMIN, or MODERATOR navigates to `/external`
2. **UI:** `ExternalPage.tsx` renders:
   - Stat cards via `StatCardV2`
   - Search input for external users
   - Status and sort filters (`FilterButtonWithDropdown`)
   - "Invite New External" button (opens `InviteExternalModal.tsx`)
3. **Invite External User:**
   - **UI:** Click "Invite New External" → `InviteExternalModal.tsx` opens
   - **UI:** Fill external user details (email, name, organization) and submit
   - **API:** `POST /api/v1/invites/invite-org-user` or dedicated external invite endpoint with `role = EXTERNAL`
   - **Backend:** Creates invited external user and `OrganizationUser` entry, sends invite email
   - **UI:** Close modal and refetch externals list
4. **List & Load:**
   - **API (SYSTEM_ADMIN):** `useGetAllUsersQuery({ role: EXTERNAL, ... })` → fetch all external users
   - **API (ORG roles):** `useGetAllUsersByOrganizationQuery({ organizationId, role: EXTERNAL, ... })` → fetch externals for organization
   - **UI:** `ExternalRecord.tsx` displays table:
     - Columns: Name, Email, Join Date, Status, Last Updated, Actions
     - Rows rendered via `ExternalTableRow.tsx`
5. **Per‑External User Actions (from each row):**
   - **View External Details:**
     - **UI:** Click "View" action in `ExternalTableRow.tsx`
     - **UI:** `onViewExternal(externalData)` is called with normalized data (name, email, phone, profile fields)
     - **UI:** `ExternalDetailsModal.tsx` opens to show full details
   - **Edit External (via modal):**
     - **UI:** Click "Edit" action in `ExternalTableRow.tsx`
     - **UI:** `onEditExternal(externalData)` is called with normalized data
     - **UI:** `EditExternalModal.tsx` opens to allow editing profile fields; on submit, underlying user/org-user records are updated
   - **Change Status (enable / disable / restore):**
     - **UI:** Status label and available actions follow the same pattern as medics/org users (Active/Inactive/Invited/Deleted)
     - **API:** Uses `deleteUser`, `deleteOrganizationUser`, or status update endpoints for `User` / `OrganizationUser`
     - **UI:** Toasts on success, list refreshed
   - **Delete External:**
     - **UI:** "Delete" option triggers SweetAlert confirmation and then:
     - **API:** `deleteUser(externalId)` or `deleteOrganizationUser(externalId)` depending on role
     - **UI:** Toast on success and refetch of externals list
6. **UI:** User can search, filter by status, sort, and paginate external users

**State Transitions:**

- External user created in INVITED/PENDING/ACTIVE state
- External user status updated: `ACTIVE` ↔ `INACTIVE` / `INVITED` / `DELETED`
- External user deleted (user + org-user entries)

**Evidence:**

- `ab3-medical-new/src/app/(dashboard)/external/page.tsx`
- `ab3-medical-new/src/components/external/ExternalRecord.tsx`
- `ab3-medical-new/src/components/Tables/ExternalTableHeader.tsx`
- `ab3-medical-new/src/components/Tables/ExternalTableRow.tsx`
- `ab3-medical-new/src/components/Modals/InviteExternalModal.tsx`
- `ab3-medical-new/src/components/Modals/ExternalDetailsModal.tsx`
- `ab3-medical-new/src/components/Modals/EditExternalModal.tsx`

---

## WORKFLOW 37: Session Management

**Name:** View and Delete Sessions  
**Actors:** All authenticated users  
**Trigger:** User views sessions in settings

**Steps:**

1. **UI:** Navigate to `/setting`, open "Session Management"
2. **API:** `GET /api/v1/sessions/my-sessions` → Get user's sessions
3. **Backend:**
   - Fetch all Session documents for user
   - Decrypt device info, IP address for display
4. **UI:** Display sessions list (device, browser, IP, last login)
5. **UI:** User can delete session (except current)
6. **API:** `DELETE /api/v1/sessions/:sessionId` → Delete session
7. **Backend:**
   - Delete Session document
   - If current session: User will be logged out on next request
8. **UI:** If current session deleted, redirect to login

**State Transitions:**

- Session deleted from database

**Evidence:**

- `ab3-medical-new/src/components/Settings/SessionManagement.tsx`
- `ab3-medical-api-v2/src/app/modules/Session/session.controller.ts`

---

## WORKFLOW 38: Create/Update/Delete Diagnosis

**Name:** Manage Diagnosis Codes  
**Actors:** SYSTEM_ADMIN  
**Trigger:** SYSTEM_ADMIN manages diagnosis codes

**Steps:**

1. **UI:** Navigate to `/diagnosis`, view diagnosis list
2. **Create:**
   - Click "Add Diagnosis"
   - Fill form (diagnosisName, diagnosisCode)
   - **API:** `POST /api/v1/diagnosis` → `DiagnosisService.createDiagnosis()`
   - Backend: Create Diagnosis with encrypted fields, status `ACTIVE`
3. **Update:**
   - Click "Edit" on diagnosis
   - Modify fields
   - **API:** `PUT /api/v1/diagnosis/:id` → `DiagnosisService.editDiagnosis()`
   - Backend: Update Diagnosis (encrypted fields)
4. **Delete:**
   - Click "Delete" on diagnosis
   - **API:** `DELETE /api/v1/diagnosis/:id` → `DiagnosisService.deleteDiagnosis()`
   - Backend: Delete Diagnosis

**Data Encryption:**

- `diagnosisName` and `diagnosisCode` encrypted at rest

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Diagnosis/Diagnosis.services.ts`
- `ab3-medical-api-v2/src/app/modules/Diagnosis/Diagnosis.route.ts`
- `ab3-medical-new/src/app/(dashboard)/diagnosis/page.tsx`

---

## WORKFLOW 39: Create/Update/Delete Organization Types

**Name:** Manage Organization Types  
**Actors:** SYSTEM_ADMIN  
**Trigger:** Admin navigates to organization types page

**UI Components:**

- **Page:** `/organization-types` → `OrganizationTypesPage.tsx`
- **Table:** `OrganizationTypeTableRow.tsx`
- **Modals:**
  - `AddOrganizationTypeModal.tsx`
  - `EditOrganizationTypeModal.tsx`

**Steps:**

1. **List & Search:**
   - **UI:** SYSTEM_ADMIN navigates to `/organization-types`.
   - **API:** `GET /api/v1/organization-types` → `OrganizationTypeService.getAllOrganizationType()`
   - **UI:** Results displayed in a table with search filtering by name.
2. **Create:**
   - **UI:** Click "Add New Organization Type".
   - **UI:** Fill name in `AddOrganizationTypeModal`.
   - **API:** `POST /api/v1/organization-types` → `OrganizationTypeService.createOrganizationType()`
3. **Edit & Status Change:**
   - **UI:** Click "Edit" in row or select "Activate/Deactivate" from action menu.
   - **API:** `PUT /api/v1/organization-types/:id` → `OrganizationTypeService.editOrganizationType()`
   - **Backend:** Updates `organizationTypeName` or `status`.
4. **Delete:**
   - **UI:** Click "Delete" in row.
   - **Backend API:** `DELETE /api/v1/organization-types/:id` → `OrganizationTypeService.deleteOrganizationType()`
   - **Note:** Frontend restricts deletion if connected to organizations.

**State Transitions:**

- Status toggle: `ACTIVE` ↔ `INACTIVE`
- Deletion: Document removed.

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/OrganizationType/OrganizationType.services.ts`
- `ab3-medical-new/src/app/(dashboard)/organization-types/page.tsx`

---

## WORKFLOW 40: Create/Update/Delete Organization Sports

**Name:** Manage Organization Sports  
**Actors:** SYSTEM_ADMIN  
**Trigger:** Admin navigates to organization sports page

**UI Components:**

- **Page:** `/organization-sports` → `OrganizationSportsPage.tsx`
- **Table:** `OrganizationSportTableRow.tsx`
- **Modals:**
  - `AddOrganizationSportModal.tsx`
  - `EditOrganizationSportModal.tsx`

**Steps:**

1. **List & Search:**
   - **UI:** SYSTEM_ADMIN navigates to `/organization-sports`.
   - **API:** `GET /api/v1/organization-sports` → `OrganizationSportService.getAllOrganizationSport()`
   - **UI:** Results displayed in a table with search filtering by name.
2. **Create:**
   - **UI:** Click "Add New Organization Sport".
   - **UI:** Fill name in `AddOrganizationSportModal`.
   - **API:** `POST /api/v1/organization-sports` → `OrganizationSportService.createOrganizationSport()`
3. **Edit & Status Change:**
   - **UI:** Click "Edit" in row or select "Activate/Deactivate" from action menu.
   - **API:** `PUT /api/v1/organization-sports/:id` → `OrganizationSportService.editOrganizationSport()`
   - **Backend:** Updates `organizationSportName` or `status`.
4. **Delete:**
   - **UI:** Click "Delete" in row.
   - **Backend API:** `DELETE /api/v1/organization-sports/:id` → `OrganizationSportService.deleteOrganizationSport()`
   - **Note:** Frontend restricts deletion if connected to organizations.

**State Transitions:**

- Status toggle: `ACTIVE` ↔ `INACTIVE`
- Deletion: Document removed.

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/OrganizationSport/OrganizationSport.services.ts`
- `ab3-medical-new/src/app/(dashboard)/organization-sports/page.tsx`

## WORKFLOW 41: Create/Edit Form

**Name:** Form Builder - Create/Edit Dynamic Form  
**Actors:** Authenticated users  
**Trigger:** User creates or edits form

**Steps:**

1. **Create:**
   - Navigate to `/forms/create`
   - Use FormBuilder component to add pages, fields
   - Set form title and description
   - Click "Save"
   - **API:** `POST /api/v1/forms` → `FormService.createForm()`
   - Backend: Create Form with `formData` (JSON structure), status `ACTIVE`
2. **Edit:**
   - Navigate to `/forms/edit/[id]`
   - Load existing form data into FormBuilder
   - Modify form structure
   - Click "Save"
   - **API:** `PUT /api/v1/forms/:id` → `FormService.updateForm()`
   - Backend: Update Form `formData`
3. **View:**
   - Navigate to `/forms/view/[id]`
   - **API:** `GET /api/v1/forms/:id` → Get form
   - Display form in view mode

**Form Data Structure:**

- `formData` contains: `state` (array of pages with inputs), `formTitle`, `formDescription`, `formId`

**Evidence:**

- `ab3-medical-new/src/app/(dashboard)/forms/create/page.tsx`
- `ab3-medical-new/src/app/(dashboard)/forms/edit/[id]/page.tsx`
- `ab3-medical-api-v2/src/app/modules/Form/form.services.ts`

---

## WORKFLOW 42: Account Settings Management

**Name:** Manage Account Settings (Password, Sessions, Delete Account)  
**Actors:** SYSTEM_ADMIN, SUPER_ADMIN, ADMIN, MODERATOR, MEDIC, PATIENT, EXTERNAL  
**Trigger:** Any authenticated user accesses settings page

**UI Components:**

- **Page:** `/setting` → `SettingsPage`
- **Components:**
  - `SettingsTabs.tsx` (tabbed settings interface)
  - `SessionManagement.tsx` (active sessions list with delete functionality)
- **Modals:**
  - `ChangePasswordModal.tsx` (change password form)
  - `DeleteAccountModal.tsx` (delete account confirmation)

**Steps:**

1. **UI:** Any authenticated user (SYSTEM_ADMIN, SUPER_ADMIN, ADMIN, MODERATOR, MEDIC, PATIENT, EXTERNAL) navigates to `/setting`
2. **UI:** `SettingsPage` displays `SettingsTabs.tsx` with tabs:
   - **Security Settings:**
     - Change Password button (opens `ChangePasswordModal.tsx`)
     - Session Management section (shows `SessionManagement.tsx` component)
   - **Account Settings:**
     - Delete Account button (opens `DeleteAccountModal.tsx`)
3. **Change Password:**
   - **UI:** User clicks "Change Password" → `ChangePasswordModal.tsx` opens
   - **UI:** User enters old password, new password, confirm password in modal form
   - **API:** `POST /api/v1/users/change-password` → `UserService.changePassword()`
   - **Backend:** Verify old password, hash new password, update user
   - **UI:** Show success toast, close modal
4. **Session Management:**
   - **UI:** `SessionManagement.tsx` displays active sessions list (device, IP, last active)
   - **API:** `GET /api/v1/sessions/me` → `SessionService.getMySessions()`
   - **UI:** User clicks "Delete" button on a session row
   - **API:** `DELETE /api/v1/sessions/:sessionId` → `SessionService.deleteSession()`
   - **Backend:** Delete session from database
   - **UI:** Show success toast, refresh sessions list
5. **Delete Account:**
   - **UI:** User clicks "Delete Account" → `DeleteAccountModal.tsx` opens with confirmation
   - **UI:** User confirms deletion in modal
   - **API:** `DELETE /api/v1/users/me` → `UserService.deleteMyAccount()`
   - **Backend:** Soft delete user (set `isDeleted: true`)
   - **UI:** Clear localStorage, sessionStorage, cookies, redirect to `/authentication/login`

**State Transitions:**

- Password updated
- Session deleted
- Account deleted (soft delete)

**Error States:**

- Invalid old password → 400
- Session not found → 404
- Account deletion failed → 400

**Evidence:**

- `ab3-medical-new/src/app/(dashboard)/setting/page.tsx`
- `ab3-medical-new/src/components/setting/SettingsTabs.tsx`
- `ab3-medical-new/src/components/Modals/ChangePasswordModal.tsx`
- `ab3-medical-new/src/components/Settings/SessionManagement.tsx`
- `ab3-medical-new/src/components/Modals/DeleteAccountModal.tsx`
- `ab3-medical-api-v2/src/app/modules/User/user.services.ts:562-600`
- `ab3-medical-api-v2/src/app/modules/Session/session.services.ts:50-80`

---

**Document End**

## WORKFLOW 43: Submit Support Request

**Name:** Submit Support/Contact Form  
**Actors:** SYSTEM_ADMIN, SUPER_ADMIN, ADMIN, MODERATOR, MEDIC, PATIENT, EXTERNAL  
**Trigger:** Any authenticated user submits support form

**Steps:**

1. **UI:** Navigate to `/support`, fill support form (firstName, lastName, email, subject, message)
2. **API:** `POST /api/v1/support` → `SupportService.sendSupportRequest()`
3. **Backend:**
   - Validate all required fields
   - Validate email format
   - Send support email via `sendSupportEmail()` helper
   - Include user ID and userType in email (if authenticated)
4. **UI:** Show success message, reset form

**Notifications:**

- Support email sent to system administrators

**Error States:**

- Missing required fields → 400
- Invalid email format → 400

**Evidence:**

- `ab3-medical-api-v2/src/app/modules/Support/support.services.ts`
- `ab3-medical-new/src/app/(dashboard)/support/page.tsx`

---

## WORKFLOW 44: System Settings (Modal)

**Name:** System Settings Centralized Modal  
**Actors:** SYSTEM_ADMIN  
**Trigger:** Admin clicks "System Settings" link in sidebar

**UI Components:**

- **Trigger:** `Sidebar.tsx` (System Settings button)
- **Modal:** `SystemSettingsModal.tsx`
- **Tabs:**
  - `SystemSettingsOrganizationTypes.tsx`
  - `SystemSettingsOrganizationSports.tsx`
  - `SystemSettingsDiagnosis.tsx`

**Steps:**

1. **UI:** SYSTEM_ADMIN clicks "System Settings" at the bottom of the sidebar.
2. **UI:** `SystemSettingsModal.tsx` opens as a full-screen overlay with a side navigation menu.
3. **Sections:**
   - **Organisation:**
     - **Tab: Organisation Types:** Managed via `SystemSettingsOrganizationTypes.tsx` (See [WORKFLOW 39](#workflow-39-createupdatedelete-organization-types)).
     - **Tab: Sports / Disciplines:** Managed via `SystemSettingsOrganizationSports.tsx` (See [WORKFLOW 40](#workflow-40-createupdatedelete-organization-sports)).
   - **Clinical Standards:**
     - **Diagnosis / Classification Systems:** Managed via `SystemSettingsDiagnosis.tsx` (See [WORKFLOW 38](#workflow-38-createupdatedelete-diagnosis)).
   - **Records:** (Placeholder for future settings)
   - **Roles & Permissions:** (Placeholder for future settings)
4. **Closing:** Admin clicks the "Close" button with `CrossIcon` to exit the modal.

**Note:** This modal centralizes management for System Admins, providing a more streamlined experience than navigating to individual pages.

**Evidence:**

- `ab3-medical-new/src/components/Shared/Sidebar.tsx`
- `ab3-medical-new/src/components/Modals/SystemSettingsModal.tsx`
- `ab3-medical-new/src/components/system-settings/`

---

## WORKFLOW 45: Dashboard Overview

**Name:** Dashboard Stats View
**Actors:** Authenticated Users (Content varies by role)
**Trigger:** User logs in or navigates to `/dashboard`

**Steps:**

1. **UI:** User navigates to `/dashboard`.
2. **API:** `GET /api/v1/stats/*` (Endpoints vary based on user role and dashboard configuration).
3. **Backend:** Aggregates statistics (e.g., active patients, recent problems, team status).
4. **UI:** Displays dashboard widgets and charts.

**Evidence:**

- `ab3-medical-new/src/app/(dashboard)/dashboard/page.tsx`
- `ab3-medical-api-v2/src/app/modules/Stats/stats.services.ts` (implied)

---

## WORKFLOW 46: My Problems (Medic)

**Name:** View Assigned Problems
**Actors:** MEDIC
**Trigger:** User navigates to `/my-problems`

**Steps:**

1. **UI:** Medic clicks "My Problems" in the sidebar.
2. **API:** `GET /api/v1/problem/user/:id` (Fetches problems assigned to or associated with the current user).
3. **UI:** Displays a list of problems.
4. **UI:** User can filter or search the list.
5. **UI:** User clicks a problem to view details (navigates to `/patients/[id]/problems/[problemId]`).

**Evidence:**

- `ab3-medical-new/src/app/(dashboard)/my-problems/page.tsx`
- `ab3-medical-api-v2/src/app/modules/Problem/problem.services.ts`

---

## WORKFLOW 47: Organization Problems List

**Name:** View Organization Problems
**Actors:** ADMIN, MODERATOR, SUPER_ADMIN
**Trigger:** User navigates to `/problems`

**Steps:**

1. **UI:** User clicks "Problems" in the sidebar.
2. **API:** `GET /api/v1/problem/organization/:organizationId`
3. **UI:** Displays a paginated list of all problems in the organization.
4. **UI:** User applies filters (e.g., status, injury type) or searches.

**Evidence:**

- `ab3-medical-new/src/app/(dashboard)/problems/page.tsx`
- `ab3-medical-api-v2/src/app/modules/Problem/problem.services.ts`

---

## WORKFLOW 48: External User Problem Access

**Name:** View Assigned Problems (External)
**Actors:** EXTERNAL
**Trigger:** User navigates to `/problem`

**Steps:**

1. **UI:** External User clicks "Problem" in the sidebar.
2. **API:** `GET /api/v1/problem/external-user/:externalUserId`
3. **UI:** Displays list of problems assigned to the external user.
4. **UI:** User can view problem details (read-only access to specific fields).

**Evidence:**

- `ab3-medical-new/src/app/(dashboard)/problem/page.tsx`
- `ab3-medical-new/src/store/apis/Problem/index.ts`

---

## WORKFLOW 49: Create Problem (Standalone)

**Name:** Add New Problem (Standalone)
**Actors:** MEDIC, ADMIN, MODERATOR, SUPER_ADMIN
**Trigger:** User clicks "Add New Problem" from sidebar.

**Steps:**

1. **UI:** Navigate to `/patients/add-new-problem`.
2. **UI:** User searches for and selects a patient.
3. **UI:** User fills in problem details:
   - status (open/closed)
   - type (Injury/Illness/etc.)
   - body part / location
   - date of onset
   - severity
4. **API:** `POST /api/v1/problem` → `ProblemService.createProblem()`
5. **Backend:**
   - Creates problem record.
   - Associates with selected patient and organization.
6. **UI:** Redirects to the Problem Details page or Patient Summary.

**Evidence:**

- `ab3-medical-new/src/app/(dashboard)/patients/add-new-problem/page.tsx`
- `ab3-medical-api-v2/src/app/modules/Problem/problem.services.ts`

---

## WORKFLOW 50: View and Manage Problem Details

**Name:** View Problem Details
**Actors:** Authenticated Users (with access to the patient)
**Trigger:** User clicks on a problem from a list.

**Steps:**

1. **UI:** Navigate to `/patients/[id]/problems/[problemId]`.
2. **API:**
   - `GET /api/v1/problem/:id` (Fetch problem details)
   - `GET /api/v1/notes/problem/:problemId` (Fetch associated notes)
3. **UI:** Displays problem info, SOAP notes, and attachment options.

**Evidence:**

- `ab3-medical-new/src/app/(dashboard)/patients/[id]/problems/[problemId]/page.tsx`
- `ab3-medical-new/src/app/(dashboard)/patients/[id]/problems/ProblemNotes.tsx` (Component)
- `ab3-medical-api-v2/src/app/modules/Problem/problem.services.ts`
- `ab3-medical-api-v2/src/app/modules/Note/note.services.ts`

---

## Note: All the patient, medic, organization user, system user, external users, teams, team members, organizations routes/page have filter, search of their own.

**Document End**

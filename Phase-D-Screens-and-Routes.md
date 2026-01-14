# Phase D — Screens/Routes and Functions

**Generated:** 2026
**Repository:** AB3 Medical Management System  
**Purpose:** Complete catalog of all screens, routes, UI components, and API calls

---

## SCREENS Catalog

| ScreenName                | Route/Path                                  | Primary Purpose                    | Key UI Components                   | Functions Available                           | API Calls Invoked                                                     | Permissions Required                         | Evidence                                                                                |
| ------------------------- | ------------------------------------------- | ---------------------------------- | ----------------------------------- | --------------------------------------------- | --------------------------------------------------------------------- | -------------------------------------------- | --------------------------------------------------------------------------------------- |
| **Login**                 | `/authentication/login`                     | User authentication                | `LoginStep.tsx`                     | Login (with/without 2FA), password reset link | `POST /api/v1/users/login`, `POST /api/v1/users/login-without-2fa`    | None (public)                                | `ab3-medical-new/src/app/authentication/login/page.tsx`                                 |
| **OTP Verification**      | `/authentication/otp-verifications`         | 2FA code verification              | `OtpVerification.tsx`               | Verify OTP, resend OTP                        | `POST /api/v1/users/verify2FAOTP`, `POST /api/v1/users/resend2FAOTP`  | None (public)                                | `ab3-medical-new/src/app/authentication/otp-verifications/page.tsx`                     |
| **Password Reset**        | `/authentication/reset-password`            | Password reset                     | Reset password form                 | Reset password                                | `POST /api/v1/users/reset-password`                                   | None (public)                                | `ab3-medical-new/src/app/authentication/reset-password/page.tsx`                        |
| **User Onboarding**       | `/onboarding-user`                          | Complete invitation signup         | `OnBoardingUserStep1-4.tsx`         | Complete signup, set password                 | `POST /api/v1/users/signupForInvite`                                  | None (public, token-based)                   | `ab3-medical-new/src/app/onboarding-user/page.tsx`                                      |
| **Setup Organization**    | `/setup-organization`                       | Organization setup                 | Organization setup form             | Create organization super admin               | `POST /api/v1/organizations/setup-organization-super-admin`           | None (public, token-based)                   | `ab3-medical-new/src/app/setup-organization/page.tsx`                                   |
| **Dashboard**             | `/dashboard`                                | Main dashboard                     | Dashboard content                   | View dashboard stats                          | `GET /api/v1/stats/*`                                                 | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/dashboard/page.tsx`                                |
| **Patients List**         | `/patients`                                 | View all patients                  | `PatientsPageContent.tsx`           | List patients, filter, search                 | `GET /api/v1/users/patients-with-organizations`                       | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/patients/page.tsx`                                 |
| **Patient Detail**        | `/patients/[id]/summary`                    | Patient profile view               | Patient summary, problems, notes    | View patient details                          | `GET /api/v1/users/patient-with-organizations/:userId`                | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/patients/[id]/summary/page.tsx`                    |
| **Add Patient Problem**   | `/patients/[id]/problems/add-problem`       | Create new problem for patient     | `AddPatientProblem.tsx`             | Create problem                                | `POST /api/v1/problem`                                                | MEDIC, ADMIN, MODERATOR, SUPER_ADMIN         | `ab3-medical-new/src/app/(dashboard)/patients/[id]/problems/add-problem/page.tsx`       |
| **View Patient Problem**  | `/patients/[id]/problems/[problemId]`       | View problem details               | `ProblemNotes.tsx`, problem details | View problem, add notes                       | `GET /api/v1/problem/:id`, `GET /api/v1/notes/problem/:problemId`     | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/patients/[id]/problems/[problemId]/page.tsx`       |
| **Add SCAT6 Form**        | `/patients/[id]/forms/add-scat-6`           | Create SCAT6 concussion assessment | SCAT6 form steps                    | Create SCAT6                                  | `POST /api/v1/scat6`                                                  | MEDIC, ADMIN, MODERATOR, SUPER_ADMIN         | `ab3-medical-new/src/app/(dashboard)/patients/[id]/forms/add-scat-6/page.tsx`           |
| **View SCAT6 Form**       | `/patients/[id]/forms/view-scat-6/[formId]` | View SCAT6 assessment              | SCAT6 view component                | View SCAT6                                    | `GET /api/v1/scat6/:id`                                               | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/patients/[id]/forms/view-scat-6/[formId]/page.tsx` |
| **Problems List**         | `/problems`                                 | View all problems                  | `ProblemsPageContent.tsx`           | List problems, filter by status               | `GET /api/v1/problem/organization/:organizationId`                    | ADMIN, MODERATOR, SUPER_ADMIN                | `ab3-medical-new/src/app/(dashboard)/problems/page.tsx`                                 |
| **My Problems**           | `/my-problems`                              | Patient's own problems             | Problems list                       | View own problems                             | `GET /api/v1/problem/user/:id`                                        | MEDIC                                        | `ab3-medical-new/src/app/(dashboard)/my-problems/page.tsx`                              |
| **Add New Problem**       | `/patients/add-new-problem`                 | Create problem (standalone)        | `AddNewProblemContent.tsx`          | Create problem                                | `POST /api/v1/problem`                                                | MEDIC, ADMIN, MODERATOR, SUPER_ADMIN         | `ab3-medical-new/src/app/(dashboard)/patients/add-new-problem/page.tsx`                 |
| **Organizations List**    | `/organizations`                            | View all organizations             | Organizations list                  | List organizations                            | `GET /api/v1/organizations`                                           | SYSTEM_ADMIN (Super_Admin, Admin, Moderator) | `ab3-medical-new/src/app/(dashboard)/organizations/page.tsx`                            |
| **Organization Detail**   | `/organizations/[id]`                       | Organization details               | Organization view                   | View organization                             | `GET /api/v1/organizations/:id`                                       | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/organizations/[id]/page.tsx`                       |
| **Teams List**            | `/teams`                                    | View all teams                     | Teams list                          | List teams                                    | `GET /api/v1/teams`                                                   | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/teams/page.tsx`                                    |
| **Team Members**          | `/teams/[teamId]/members`                   | Team member management             | Team members list                   | View team members                             | `GET /api/v1/teams/:id`                                               | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/teams/[teamId]/members/page.tsx`                   |
| **Forms List**            | `/forms`                                    | View all forms                     | Forms list                          | List forms                                    | `GET /api/v1/forms`                                                   | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/forms/page.tsx`                                    |
| **Create Form**           | `/forms/create`                             | Create new form                    | Form builder                        | Create form                                   | `POST /api/v1/forms`                                                  | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/forms/create/page.tsx`                             |
| **Edit Form**             | `/forms/edit/[id]`                          | Edit form                          | Form builder                        | Update form                                   | `PUT /api/v1/forms/:id`                                               | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/forms/edit/[id]/page.tsx`                          |
| **View Form**             | `/forms/view/[id]`                          | View form                          | Form viewer                         | View form                                     | `GET /api/v1/forms/:id`                                               | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/forms/view/[id]/page.tsx`                          |
| **Diagnosis List**        | `/diagnosis`                                | View diagnoses                     | Diagnosis list                      | List diagnoses                                | `GET /api/v1/diagnosis`                                               | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/diagnosis/page.tsx`                                |
| **Profile**               | `/profile`                                  | User profile                       | Profile form                        | View/update own profile                       | `GET /api/v1/users/me`, `PUT /api/v1/users/me`                        | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/profile/page.tsx`                                  |
| **Settings**              | `/setting`                                  | User settings                      | Settings form                       | Update settings                               | Various                                                               | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/setting/page.tsx`                                  |
| **Support**               | `/support`                                  | Support/contact                    | Support form                        | Submit support request                        | `POST /api/v1/support`                                                | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/support/page.tsx`                                  |
| **System Users**          | `/system-users`                             | System user management             | Users list                          | List system users                             | `GET /api/v1/users`                                                   | SYSTEM_ADMIN (Super_Admin, Admin, Moderator) | `ab3-medical-new/src/app/(dashboard)/system-users/page.tsx`                             |
| **Medics**                | `/medics`                                   | Medic user list                    | Medics list                         | List medics                                   | `GET /api/v1/organization-users` (filtered)                           | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/medics/page.tsx`                                   |
| **Organization Types**    | `/organization-types`                       | Manage org types                   | Org types list                      | CRUD org types                                | `GET /api/v1/organization-types`, `POST /api/v1/organization-types`   | SYSTEM_ADMIN (Super_Admin, Admin, Moderator) | `ab3-medical-new/src/app/(dashboard)/organization-types/page.tsx`                       |
| **Organization Sports**   | `/organization-sports`                      | Manage org sports                  | Org sports list                     | CRUD org sports                               | `GET /api/v1/organization-sports`, `POST /api/v1/organization-sports` | SYSTEM_ADMIN (Super_Admin, Admin, Moderator) | `ab3-medical-new/src/app/(dashboard)/organization-sports/page.tsx`                      |
| **External Users**        | `/external`                                 | External user management           | External users list                 | Manage external users                         | Various                                                               | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/external/page.tsx`                                 |
| **Organization Problems** | `/problems`                                 | organization's problem list        | organization's problem list         | List organization's problems                  | Get /api/v1/problem/organization/:organizationId                      | Authenticated                                | `ab3-medical-new/src/app/(dashboard)/external/page.tsx`                                 |

| **External User's Problems** | `/problem` | external user's problem list | external user's problem list | List external user's problems | Get /api/v1/problem/external-user/:externalUserId | Authenticated | `ab3-medical-new/src/app/(dashboard)/external/page.tsx` |

---

## Route Groups

### Public Routes (No Authentication)

- `/authentication/login`
- `/authentication/otp-verifications`
- `/authentication/reset-password`
- `/authentication/reset-password-email-sent`
- `/authentication/password-reset-success`
- `/onboarding-user` (token-based)
- `/setup-organization` (token-based)

### Protected Routes (Requires Authentication)

All routes under `/dashboard` require authentication via `authGuard()` middleware. And then they are protected by role-based access control. (System Admin ( Super Admin, Admin, Moderator), Organization (Super Admin, Admin), Medic(Organization Medic))

---

## API Endpoint Summary (This is just summary of API endpoints, backend has more endpoints)

### User Endpoints

- `POST /api/v1/users/login` - Login
- `POST /api/v1/users/signupForInvite` - Complete signup
- `GET /api/v1/users/me` - Get current user
- `GET /api/v1/users/patients-with-organizations` - List patients

### Problem Endpoints

- `GET /api/v1/problem/organization/:organizationId` - List org problems
- `GET /api/v1/problem/user/:id` - List user problems
- `POST /api/v1/problem` - Create problem
- `PUT /api/v1/problem/:id` - Update problem
- `DELETE /api/v1/problem/:id` - Delete problem

### Note Endpoints

- `GET /api/v1/notes/problem/:problemId` - List problem notes
- `POST /api/v1/notes` - Create note
- `PUT /api/v1/notes/:noteId` - Update note
- `DELETE /api/v1/notes/:noteId` - Delete note

### Organization Endpoints

- `GET /api/v1/organizations` - List organizations (SYSTEM_ADMIN (Super_Admin, Admin, Moderator))
- `POST /api/v1/organizations` - Create organization (SYSTEM_ADMIN (Super_Admin, Admin, Moderator))
- `GET /api/v1/organizations/:id` - Get organization

**Evidence:**

- `ab3-medical-api-v2/src/app/routes/routes.ts`

---

**Document End**

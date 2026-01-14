# Phase G — System Reality Summary

**Generated:** 2026  
**Repository:** AB3 Medical Management System  
**Purpose:** High-level system overview, architectural patterns, data lifecycle, integrations, and gaps

---

## System Overview

AB3 Medical is a **medical management system for sports organizations** built with a **Next.js frontend** and **Express.js/TypeScript backend** using **MongoDB**. The system manages:

- **Multi-tenant organizations** (clubs, academies, hospitals, etc.)
- **User hierarchy**: SYSTEM_ADMIN ( SUPER_ADMIN, ADMIN, MODERATOR) → SUPER_ADMIN (Organization level) → ADMIN (Organization level) → MODERATOR (Organization level) → MEDIC (Organization level) → PATIENT (Organization level) → EXTERNAL (Organization level)
- **Medical records**: Problems (injuries, illnesses, surgeries, screenings, allergies), Notes (Records), SCAT6 concussion assessments
- **Team management**: Organizations contain teams, users (patient and medic) belong to teams
- **Invitation system**: Token-based user onboarding
- **File storage**: AWS S3 (For future not in use now) and Cloudinary for attachments
- **Field-level encryption**: Sensitive medical data encrypted at rest

---

## Key Architectural Patterns

1. **MVC Pattern**: Each module has `model.ts`, `controller.ts`, `services.ts`, `route.ts`

   - **Evidence**: `ab3-medical-api-v2/src/app/modules/` structure

2. **Middleware Chain**: `authGuard()` → `routeGuard()` → controller

   - **Evidence**: `ab3-medical-api-v2/src/app/middlewar/authGuard.ts`, `routeGuard.ts`

3. **Transaction Safety**: Critical operations use MongoDB transactions

   - **Evidence**: `ab3-medical-api-v2/src/app/modules/Problem/problem.services.ts:200-271`

4. **Async Notifications**: Email sending happens after DB commits (non-blocking)

   - **Evidence**: `ab3-medical-api-v2/src/app/modules/Problem/problem.services.ts:246-261`

5. **Encryption**: Field-level encryption via Mongoose getters/setters
   - **Evidence**: `ab3-medical-api-v2/src/helper/encrypt.ts`, `decrypt.ts`

---

## Data Lifecycle

### Create Rules

- **Problems**: Created by MEDIC/ADMIN/MODERATOR/SUPER_ADMIN
  - **Evidence**: `ab3-medical-api-v2/src/app/modules/Problem/problem.route.ts:23`
- **Notes (Records)**: Created by authenticated users
  - **Evidence**: `ab3-medical-api-v2/src/app/modules/Note/note.route.ts:16`
- **Users**: Created via invitation system
  - **Evidence**: `ab3-medical-api-v2/src/app/modules/RoleBasedInvite/invite.services.ts`
- **Organizations**: Created by SYSTEM_ADMIN (SUPER_ADMIN, ADMIN, MODERATOR)
  - **Evidence**: `ab3-medical-api-v2/src/app/modules/Organization/organization.route.ts:10`

### Update Rules

- **Problems**: Updated by creator or MEDIC/ADMIN/MODERATOR/SUPER_ADMIN
  - **Evidence**: `ab3-medical-api-v2/src/app/modules/Problem/problem.route.ts:25`
- **Notes (Records)**: Updated by creator (verified via `verifyNoteCreator` middleware)
  - **Evidence**: `ab3-medical-api-v2/src/app/modules/Note/note.route.ts:17`
- **Users**: Updated by self (profile) or ADMIN/MODERATOR/SUPER_ADMIN
  - **Evidence**: `ab3-medical-api-v2/src/app/modules/User/user.route.ts:49-52`

### Delete Rules

- **Problems**: Deleted by MEDIC/ADMIN/MODERATOR/SUPER_ADMIN
  - **Evidence**: `ab3-medical-api-v2/src/app/modules/Problem/problem.route.ts:27`
- **Notes**: Deleted by creator only
  - **Evidence**: `ab3-medical-api-v2/src/app/modules/Note/note.route.ts:19`
- **Users**: Soft delete (status → DELETED)
  - **Evidence**: `ab3-medical-api-v2/src/app/modules/User/user.model.ts:52-56`

### Draft/Publish States

- **Problems**:
  - `visibilityScope` = `Draft` (not visible, only to creator), `Internal` (org only), `Public` (accross all organizations)
  - `status` = `Active`, `Resolved`, `Archived`
  - **Evidence**: `ab3-medical-api-v2/src/app/modules/Problem/problem.model.ts:29-44`

### Audit Logging

- `createdAt`, `updatedAt` timestamps on all models (Mongoose timestamps)
- `createdBy` field on Problems, Notes (Record), NoteReplies (Record replies)
- Session tracking with device info, IP address (encrypted)
- **Evidence**: Model files use `{ timestamps: true }` option

---

## Integrations

| Integration                             | Purpose                    | Implementation                  | Evidence                                                           |
| --------------------------------------- | -------------------------- | ------------------------------- | ------------------------------------------------------------------ |
| **AWS S3 (Not in use now, for future)** | File storage               | Direct upload or presigned URLs | `ab3-medical-api-v2/src/app/modules/Upload/aws-upload.services.ts` |
| **Cloudinary**                          | File storage (alternative) | Direct upload                   | `ab3-medical-api-v2/src/app/modules/Upload/upload.services.ts`     |
| **Nodemailer**                          | Email sending              | SMTP email service              | `ab3-medical-api-v2/src/helper/sendEmail.ts`                       |
| **JWT**                                 | Authentication             | Token-based auth                | `ab3-medical-api-v2/src/helper/jwtHelper.ts`                       |
| **MongoDB**                             | Database                   | Document store                  | `ab3-medical-api-v2/src/app.ts:27-44`                              |

---

## Gaps & Unknowns

1. Patient/ User transfer between organizations
2. Surgery, Refferal, Vaccination type records clarification.
3. Shound changing Status from System level affect all organization level users.
4. Where to change problem status in new view problem screen.
5. How to stop showing old organization newly created problem after transfer.
6. Should we show initial assestemnt record in records tab.
7. What to do with referral tabs of the patient summary page.

---

## Database Schema Reference

The complete database schema is documented in:

- `ab3-medical-api-v2/database_schema.dbml`

This DBML file can be imported into [dbdiagram.io](https://dbdiagram.io) to generate ER diagrams.

---

## Key Files Reference

### Backend Entry Points

- Server: `ab3-medical-api-v2/src/server.ts`
- App: `ab3-medical-api-v2/src/app.ts`
- Routes: `ab3-medical-api-v2/src/app/routes/routes.ts`

### Frontend Entry Points

- Root Layout: `ab3-medical-new/src/app/layout.tsx`
- Dashboard Layout: `ab3-medical-new/src/app/(dashboard)/layout.tsx`

### Core Models

- User: `ab3-medical-api-v2/src/app/modules/User/user.model.ts`
- Problem: `ab3-medical-api-v2/src/app/modules/Problem/problem.model.ts`
- Note (Record): `ab3-medical-api-v2/src/app/modules/Note/note.model.ts`
- Organization: `ab3-medical-api-v2/src/app/modules/Organization/organization.model.ts`

### Security

- Auth Guard: `ab3-medical-api-v2/src/app/middlewar/authGuard.ts`
- Route Guard: `ab3-medical-api-v2/src/app/middlewar/routeGuard.ts`
- Encryption: `ab3-medical-api-v2/src/helper/encrypt.ts`

---

**Document End**

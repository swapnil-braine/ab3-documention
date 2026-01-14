# Phase F — RBAC / Security / Visibility

**Generated:** 2026  
**Repository:** AB3 Medical Management System  
**Purpose:** Complete documentation of roles, permissions, access rules, and security mechanisms

---

## PERMISSIONS List

| Permission Key                                          | What It Gates                                                                                                          | Evidence                                                                   |
| ------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **SYSTEM_ADMIN access (SUPER_ADMIN, ADMIN, MODERATOR)** | Create organizations, view all users, invite users to all organizations, all things org can do like problem, note, etc | `ab3-medical-api-v2/src/app/modules/Organization/organization.route.ts:10` |
| **SUPER_ADMIN/ADMIN/MODERATOR (ORGANIZATION)**          | Create/update/delete problems, view all user (organization level), invite users to own organization                    | `ab3-medical-api-v2/src/app/modules/Problem/problem.route.ts:23-27`        |
| **MEDIC**                                               | Create problems, notes, SCAT6                                                                                          | `ab3-medical-api-v2/src/app/modules/Problem/problem.route.ts:23`           |
| **MEDIC**                                               | View problems assigned to medic                                                                                        | `ab3-medical-api-v2/src/app/modules/Problem/problem.route.ts:15`           |
| **EXTERNAL**                                            | View problems assigned to external user                                                                                | `ab3-medical-api-v2/src/app/modules/Problem/problem.route.ts:17`           |

---

## PERMISSION_MATRIX

| Role                                             | Create Problem | Update Problem | Delete Problem | Create Note | Delete Note | Invite Users | View All Problems  | Manage Organizations |
| ------------------------------------------------ | -------------- | -------------- | -------------- | ----------- | ----------- | ------------ | ------------------ | -------------------- |
| **SYSTEM_ADMIN (SUPER_ADMIN, ADMIN, MODERATOR)** | ✅             | ✅             | ✅             | ✅          | ✅ (own)    | ✅           | ✅                 | ✅                   |
| **SUPER_ADMIN, ADMIN, MODERATOR (Organization)** | ✅             | ✅             | ✅             | ✅          | ✅ (own)    | ✅           | ✅ (org)           | ❌                   |
| **MEDIC**                                        | ✅             | ✅             | ✅             | ✅          | ✅ (own)    | ❌           | ❌ (assigned only) | ❌                   |
| **EXTERNAL**                                     | ❌             | ❌             | ❌             | ✅          | ✅ (own)    | ❌           | ❌ (assigned only) | ❌                   |

---

## Security Mechanisms

| Mechanism              | Implementation                                                                              | Evidence                                                                               |
| ---------------------- | ------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| **Authentication**     | JWT token in HTTP-only cookie                                                               | `ab3-medical-api-v2/src/app/middlewar/authGuard.ts:11`                                 |
| **Session Validation** | Token verified against Session collection                                                   | `ab3-medical-api-v2/src/app/middlewar/authGuard.ts:33-44`                              |
| **Route Guards**       | `routeGuard()` middleware checks `accessRole` or `roleType`                                 | `ab3-medical-api-v2/src/app/middlewar/routeGuard.ts:22-26`                             |
| **Field Encryption**   | Sensitive fields encrypted at rest (Diagnosis, MedicalIdentifiers, EmergencyContact, Scat6) | `ab3-medical-api-v2/src/helper/encrypt.ts`, `ab3-medical-api-v2/src/helper/decrypt.ts` |
| **Password Hashing**   | bcrypt with salt rounds                                                                     | `ab3-medical-api-v2/src/helper/bcryptHelper.ts`                                        |
| **2FA**                | Email-based OTP, 6-digit code, 5-minute expiration                                          | `ab3-medical-api-v2/src/app/modules/User/user.services.ts:324-344`                     |

---

## Access Role vs User Type

The system uses two role concepts:

1. **accessRole** (`ENUM_ACCESS_ROLE`): SYSTEM_ADMIN, ORGANIZATION
2. **userType** (`ENUM_USER_TYPE`): SUPER_ADMIN, ADMIN, MODERATOR, MEDIC, PATIENT, EXTERNAL

**Route guards check both:**

- `routeGuard()` checks if user has required `accessRole` OR `roleType`
- This allows flexibility in permission checking

**Evidence:**

- `ab3-medical-api-v2/src/app/middlewar/routeGuard.ts:22-26`

---

**Document End**

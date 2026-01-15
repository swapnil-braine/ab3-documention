# Phase C — Relationships

**Generated:** 2026  
**Repository:** AB3 Medical Management System  
**Purpose:** Complete mapping of relationships between domain objects

---

## RELATIONSHIPS Table

| FromObject           | ToObject               | RelationshipType        | Mechanism                                                                                  | Enforcement Location        | Evidence                                                             |
| -------------------- | ---------------------- | ----------------------- | ------------------------------------------------------------------------------------------ | --------------------------- | -------------------------------------------------------------------- |
| **User**             | **Profile**            | 1:1                     | `User.profile` → `Profile._id`, `Profile.user` → `User._id`                                | Both (bidirectional ref)    | `user.model.ts:13-16`, `profile.model.ts:7-10`                       |
| **User**             | **EmergencyContact**   | 1:1                     | `Profile.emergencyContact` → `EmergencyContact._id`, `EmergencyContact.user` → `User._id`  | Both                        | `profile.model.ts:90-93`, `emergencyContact.model.ts:70-74`          |
| **User**             | **MedicalIdentifiers** | 1:1                     | `Profile.NHSNO` → `MedicalIdentifiers._id`, `MedicalIdentifiers.user` → `User._id`         | Both                        | `profile.model.ts:94-97`, `medicalIdentifiers.model.ts:106-110`      |
| **User**             | **Session**            | 1:M                     | `Session.user` → `User._id`                                                                | DB (indexed)                | `session.model.ts:8-12`                                              |
| **User**             | **OrganizationUser**   | 1:M                     | `OrganizationUser.user` → `User._id`, `OrganizationUser.organization` → `Organization._id` | DB (unique composite index) | `organizationUser.model.ts:18-22`, `organizationUser.model.ts:78-81` |
| **User**             | **TeamUser**           | 1:M                     | `TeamUser.user` → `User._id`, `TeamUser.team` → `Team._id`                                 | DB (unique composite index) | `teamUser.model.ts:12-16`, `teamUser.model.ts:47-50`                 |
| **User**             | **Problem**            | 1:M (as patient)        | `Problem.patient` → `User._id`                                                             | DB (indexed)                | `problem.model.ts:6-10`                                              |
| **User**             | **Problem**            | 1:M (as creator)        | `Problem.createdBy` → `User._id`                                                           | DB (indexed)                | `problem.model.ts:48-51`                                             |
| **User**             | **Problem**            | M:M (assigned)          | `Problem.assignedUsers[]` → `User._id[]`                                                   | App logic                   | `problem.model.ts:74-77`                                             |
| **User**             | **Problem**            | M:M (external users)    | `Problem.externalUsers[]` → `User._id[]`                                                   | App logic                   | `problem.model.ts:82-85`                                             |
| **User**             | **Note**               | 1:M (as patient)        | `Note.patient` → `User._id`                                                                | DB                          | `note.model.ts:41-45`                                                |
| **User**             | **Note**               | 1:M (as creator)        | `Note.createdBy` → `User._id`                                                              | DB (indexed)                | `note.model.ts:59-62`                                                |
| **User**             | **Note**               | 1:M (as consultant)     | `Note.consultant` → `User._id`                                                             | DB                          | `note.model.ts:53-57`                                                |
| **User**             | **Note**               | M:M (mentions)          | `Note.mentions[]` → `User._id[]`                                                           | App logic                   | `note.model.ts:64-69`                                                |
| **User**             | **Note**               | M:M (seenBy)            | `Note.seenBy[]` → `User._id[]`                                                             | App logic                   | `note.model.ts:71-76`                                                |
| **User**             | **Note**               | M:M (likedBy)           | `Note.likedBy[]` → `User._id[]`                                                            | App logic                   | `note.model.ts:77-82`                                                |
| **User**             | **NoteReply**          | 1:M (as creator)        | `NoteReply.createdBy` → `User._id`                                                         | DB                          | `noteReply.model.ts:24-27`                                           |
| **User**             | **NoteReply**          | 1:M (as replyTo target) | `NoteReply.replyTo` → `User._id`                                                           | DB                          | `noteReply.model.ts:28-32`                                           |
| **User**             | **NoteReply**          | M:M (mentions)          | `NoteReply.mentions[]` → `User._id[]`                                                      | App logic                   | `noteReply.model.ts:33-38`                                           |
| **User**             | **NoteReply**          | M:M (seenBy)            | `NoteReply.seenBy[]` → `User._id[]`                                                        | App logic                   | `noteReply.model.ts:39-44`                                           |
| **User**             | **NoteReply**          | M:M (likedBy)           | `NoteReply.likedBy[]` → `User._id[]`                                                       | App logic                   | `noteReply.model.ts:45-50`                                           |
| **Organization**     | **OrganizationUser**   | 1:M                     | `OrganizationUser.organization` → `Organization._id`                                       | DB (indexed)                | `organizationUser.model.ts:13-17`                                    |
| **Organization**     | **Team**               | 1:M                     | `Team.organization` → `Organization._id`                                                   | DB (indexed)                | `team.model.ts:15-19`                                                |
| **Organization**     | **Problem**            | 1:M                     | `Problem.organization` → `Organization._id`                                                | DB (indexed)                | `problem.model.ts:11-15`                                             |
| **Organization**     | **Note**               | 1:M                     | `Note.organization` → `Organization._id`                                                   | DB (indexed)                | `note.model.ts:47-51`                                                |
| **Organization**     | **OrganizationType**   | M:1                     | `Organization.organizationType` → `OrganizationType._id`                                   | DB                          | `organization.model.ts:50-54`                                        |
| **Organization**     | **OrganizationSport**  | M:1                     | `Organization.organizationSport` → `OrganizationSport._id`                                 | DB                          | `organization.model.ts:55-59`                                        |
| **Organization**     | **Scat6**              | 1:M                     | `Scat6.organization` → `Organization._id`                                                  | DB                          | `scat6.model.ts:336-339`                                             |
| **Team**             | **TeamUser**           | 1:M                     | `TeamUser.team` → `Team._id`                                                               | DB (indexed)                | `teamUser.model.ts:7-11`                                             |
| **Team**             | **Problem**            | M:M (assigned)          | `Problem.assignedTeams[]` → `Team._id[]`                                                   | App logic                   | `problem.model.ts:78-81`                                             |
| **Problem**          | **Note**               | 1:M                     | `Note.problem` → `Problem._id`                                                             | DB                          | `note.model.ts:35-39`                                                |
| **Problem**          | **Diagnosis**          | M:1                     | `Problem.diagnosis` → `Diagnosis._id`                                                      | DB (indexed)                | `problem.model.ts:52-57`                                             |
| **Note**             | **NoteReply**          | 1:M                     | `NoteReply.note` → `Note._id`                                                              | DB                          | `noteReply.model.ts:6-9`                                             |
| **User**             | **User**               | 1:M (invitedBy)         | `User.invitedBy` → `User._id`                                                              | DB                          | `user.model.ts:44-47`                                                |
| **OrganizationUser** | **User**               | M:1 (invitedBy)         | `OrganizationUser.invitedBy` → `User._id`                                                  | DB                          | `organizationUser.model.ts:45-48`                                    |
| **TeamUser**         | **User**               | M:1 (invitedBy)         | `TeamUser.invitedBy` → `User._id`                                                          | DB                          | `teamUser.model.ts:27-30`                                            |
| **User**             | **Scat6**              | 1:M (as patient)        | `Scat6.patient` → `User._id`                                                               | DB                          | `scat6.model.ts:332-335`                                             |
| **User**             | **Scat6**              | 1:M (as creator)        | `Scat6.createdBy` → `User._id`                                                             | DB                          | `scat6.model.ts:340-343`                                             |
| **User**             | **Form**               | 1:M (as creator)        | `Form.createdBy` → `User._id` (string id)                                                  | App logic                   | `form.model.ts:32-35`                                                |

---

## Relationship Patterns

### One-to-One (1:1)

- User ↔ Profile (bidirectional)
- User ↔ EmergencyContact (via Profile)
- User ↔ MedicalIdentifiers (via Profile)

### One-to-Many (1:M)

- User → Sessions
- User → Problems (as patient)
- User → Problems (as creator)
- User → Notes (as patient/creator/consultant)
- User → NoteReplies (as creator/replyTo)
- User → Scat6 (as patient)
- User → Scat6 (as creator)
- User → Forms (as creator)
- Organization → Teams
- Organization → Problems
- Organization → Notes
- Organization → Scat6
- Organization → OrganizationType
- Organization → OrganizationSport
- Team → TeamUsers
- Problem → Notes
- Note → NoteReplies

### Many-to-Many (M:M)

- User ↔ Organization (via OrganizationUser join table)
- User ↔ Team (via TeamUser join table)
- User ↔ Problem (via `assignedUsers` array)
- User ↔ Problem (via `externalUsers` array)
- Team ↔ Problem (via `assignedTeams` array)
- User ↔ Note (via `mentions` / `seenBy` / `likedBy` arrays)
- User ↔ NoteReply (via `mentions` / `seenBy` / `likedBy` arrays)

### Self-Referential

- User → User (invitedBy relationship)

---

## Indexes for Performance

The following relationships have database indexes for query performance:

- `User.email` (unique)
- `User.profile` (indexed)
- `User.status`, `User.userType`, `User.accessRole` (single-field indexes)
- `Organization.name`, `Organization.email` (unique)
- `Organization.organizationStatus`, `Organization.organizationType` (indexes)
- `OrganizationUser(user, organization)` (unique composite)
- `OrganizationUser(organization, userType, status)` (composite)
- `TeamUser(user, team)` (unique composite)
- `Team(name, organization)` (unique composite)
- `Problem(patient, organization)` (composite)
- `Problem(organization, status)` (composite)
- `Problem.diagnosis`, `Problem.createdBy`, `Problem.onsetDate` (single-field indexes)
- `Note(organization, consultationDate)` (composite)
- `Note.createdBy`, `Note.noteStatus`, `Note.category` (single-field indexes)
- `Session(user, lastLogin)` (composite)
- `Session.token`, `Session.deviceId`, `Session.lastLogin` (single-field indexes)

**Evidence:**

- Model files contain index definitions

---

**Document End**

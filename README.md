# AB3 Medical System - Product Reality Extraction Documentation

**Generated:** 2026  
**Repository:** AB3 Medical Management System  
**Purpose:** Code-evidenced understanding of domain objects, relationships, screens, workflows, and permissions

---

## This is the overall architecture of the system/project

- From here you can go to specific phases to get more details

## Some Clarifications:

- Workflows gives an end-to-end journey of features in this project
- Er Diagram gives an overall visualaizion of backend schemas and relationships.
- To Understand the frontend, Workflows is a recommended and realible place.
- To Understand the backend, Er Diagram along with md files is a recommended and realible place.
- Note is considered as Record in UI level.
- SYSTEM_ADMIN means SUPER_ADMIN, ADMIN, MODERATOR of system level.
- ORGANIZATION means SUPER_ADMIN, ADMIN, MODERATOR of organization level.
- MEDIC means MEDIC of organization level.
- EXTERNAL means EXTERNAL of organization level.
- As the project got worked so many times with logics, its expected to see legacy fields in schems which are kept for old data compability and will be removed in future on database gets reset.

## ER Diagram (As relation, domain objects are extracted using AI, some of them might not be correct, hence mix of Er and md files is good way to explore)

[ER Diagram](https://dbdiagram.io/d/DB_design-69665365d6e030a024ea484f)

## Documentation Structure

This documentation is organized into phases as specified in the product reality extraction requirements:

### [Phase A — Stack and Repository Layout](./Phase-A-Stack-and-Repository-Layout.md)

- Technology stack identification
- Repository structure
- Key folders and their purposes
- Entry points

### [Phase B — Domain Object Inventory](./Phase-B-Domain-Object-Inventory.md)

- Complete OBJECTS table with 18 domain models
- Primary identifiers and key fields
- File path evidence for each object

### [Phase C — Relationships](./Phase-C-Relationships.md)

- RELATIONSHIPS table mapping all object relationships
- Relationship types (1:1, 1:M, M:M)
- Enforcement mechanisms and evidence

### [Phase D — Screens/Routes and Functions](./Phase-D-Screens-and-Routes.md)

- SCREENS catalog with 30+ screens
- Routes, UI components, API calls
- Permissions required for each screen

### [Phase E — Workflows](./Phase-E-Workflows.md)

- End-to-end workflows
- Step-by-step user journeys
- API calls, state transitions, notifications
- Error states and edge cases

### [Phase F — RBAC / Security / Visibility](./Phase-F-RBAC-and-Security.md)

- PERMISSIONS list
- PERMISSION_MATRIX
- Security mechanisms

### [Phase G — System Reality Summary](./Phase-G-System-Reality-Summary.md)

- System overview
- Architectural patterns
- Data lifecycle rules
- Integrations
- Gaps & Unknowns

---

## Quick Reference

### Key Technologies

- **Backend**: Express.js, TypeScript, MongoDB, Mongoose
- **Frontend**: Next.js 15, React 19, TypeScript
- **Storage**: Cloudinary, AWS S3 (for future, not used now)
- **Auth**: JWT, Cookie-based sessions

### Core Domain Objects

- User, Profile, Organization, OrganizationUser
- Team, TeamUser
- Problem, Note, NoteReply
- Diagnosis, EmergencyContact, MedicalIdentifiers
- Session, Form, Scat6

### User Roles

- SYSTEM_ADMIN ( SUPER_ADMIN, ADMIN, MODERATOR) → SUPER_ADMIN (Organization level) → ADMIN (Organization level) → MODERATOR (Organization level) → MEDIC (Organization level) → PATIENT (Organization level) → EXTERNAL (Organization level)

---

## Evidence-Based Documentation

All statements in this documentation are backed by file path references to the actual codebase. Every table, workflow, and rule includes evidence paths pointing to the source code.

---

**Last Updated:** 2026

# Phase A — Stack and Repository Layout

**Generated:** 2026 
**Repository:** AB3 Medical Management System  
**Purpose:** Identify frameworks, languages, entry points, and folder structure

---

## Technology Stack

### Backend (ab3-medical-api-v2)

- **Framework:** Express.js 5.1.0
- **Language:** TypeScript 5.9.2
- **Database:** MongoDB 6.18.0 (via Mongoose 8.17.1)
- **Authentication:** JWT (jsonwebtoken 9.0.2), Cookie-based sessions
- **File Storage:** AWS S3 (via @aws-sdk/client-s3) + Cloudinary
- **Email:** Nodemailer 7.0.6
- **Validation:** Zod 4.0.17
- **Encryption:** crypto-js 4.2.0 (for field-level encryption)

**Evidence:**
- `ab3-medical-api-v2/package.json`

### Frontend (ab3-medical-new)

- **Framework:** Next.js 15.5.9 (App Router)
- **Language:** TypeScript 5
- **UI Library:** React 19.2.3
- **State Management:** Redux Toolkit 2.8.2, Zustand 5.0.8
- **UI Components:** Radix UI primitives
- **Styling:** Tailwind CSS 4, SCSS
- **Forms:** React Hook Form 7.62.0
- **Rich Text:** Tiptap 3.x
- **HTTP Client:** Axios 1.11.0

**Evidence:**
- `ab3-medical-new/package.json`

---

## Repository Structure

```
ab3-medical-api-v2/
├── src/
│   ├── app/
│   │   ├── modules/          # Feature modules (User, Problem, Note, etc.)
│   │   ├── routes/            # Route aggregator
│   │   └── middlewar/         # Auth guards, error handlers
│   ├── config/                # Configuration (DB, JWT, AWS, etc.)
│   ├── helper/                # Utilities (encrypt, decrypt, email, JWT)
│   ├── errors/                # Custom error classes
│   ├── interfaces/            # TypeScript interfaces
│   ├── shared/                # Shared utilities (enums, catchAsync)
│   └── server.ts              # Server entry point
├── database_schema.dbml       # Database schema documentation
└── package.json

ab3-medical-new/
├── src/
│   ├── app/                   # Next.js App Router pages
│   │   ├── (dashboard)/       # Protected dashboard routes
│   │   ├── authentication/   # Login, OTP, password reset
│   │   └── api/               # API route handlers
│   ├── components/            # React components (692 files)
│   ├── store/                # Redux store and API slices
│   ├── types/                 # TypeScript type definitions
│   ├── hooks/                 # Custom React hooks
│   ├── utility/               # Helper functions
│   └── formbuilder/           # Form builder system
└── package.json
```

---

## Key Folders

| Folder | Purpose | Evidence |
|--------|---------|----------|
| `src/app/modules/` | Feature modules with MVC pattern (model, controller, service, route) | `ab3-medical-api-v2/src/app/modules/` |
| `src/app/middlewar/` | Authentication guards, route guards, error handlers | `ab3-medical-api-v2/src/app/middlewar/authGuard.ts` |
| `src/helper/` | Encryption/decryption, email sending, JWT helpers | `ab3-medical-api-v2/src/helper/encrypt.ts` |
| `src/app/(dashboard)/` | Protected dashboard pages (patients, problems, notes, etc.) | `ab3-medical-new/src/app/(dashboard)/` |
| `src/components/` | Reusable React components | `ab3-medical-new/src/components/` |
| `src/store/apis/` | RTK Query API definitions | `ab3-medical-new/src/store/apis/` |

---

## Entry Points

**Backend:**
- Server entry: `ab3-medical-api-v2/src/server.ts`
- App entry: `ab3-medical-api-v2/src/app.ts`
- Routes: `ab3-medical-api-v2/src/app/routes/routes.ts`

**Frontend:**
- Root layout: `ab3-medical-new/src/app/layout.tsx`
- Dashboard layout: `ab3-medical-new/src/app/(dashboard)/layout.tsx`
- Home page: `ab3-medical-new/src/app/page.tsx`

---

**Document End**


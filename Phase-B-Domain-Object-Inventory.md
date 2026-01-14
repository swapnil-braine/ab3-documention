# Phase B — Domain Object Inventory

**Generated:** 2026 
**Repository:** AB3 Medical Management System  
**Purpose:** Complete inventory of all domain objects with their types, fields, and usage

---

## OBJECTS Table

| ObjectName | Type | Primary Identifiers | Fields (All Model Fields) | Where Defined | Where Used |
|------------|------|---------------------|---------------------------|---------------|-----------|
| **User** | MongoDB Model | `_id` | `profile`, `email`, `password`, `accessRole`, `userType`, `medicType`, `invitedBy`, `inviteToken`, `expiresAt`, `isActive`, `status`, `twoFactorCode`, `twoFactorEnabled`, `twoFactorMethod`, `twoFactorSecret`, `twoFactorCodeExpiresAt`, `faceIdOrTouchId`, `passcode`, `passcodeBoolean`, `faceIdOrtouchIdBoolean`, `smsAlert`, `pushnotifications`, `emailnotifications`, `medicalLicense`, `createdAt`, `updatedAt` | `ab3-medical-api-v2/src/app/modules/User/user.model.ts` | Authentication, authorization, all user types (system, org, medic, patient, external) |
| **Profile** | MongoDB Model | `_id` | `user`, `firstName`, `lastName`, `age`, `gender`, `profilePhoto_url`, `addressLine1`, `addressLine2`, `city`, `state`, `postcode`, `country`, `phoneNumber`, `uiLanguage`, `spokenLanguages[]`, `accessibilityNeeds`, `hideSensitivePreviews`, `dateOfBirth`, `allergies[]`, `fitnessStatus`, `emergencyContact`, `NHSNO`, `termsAndConditions`, `createdAt`, `updatedAt` | `ab3-medical-api-v2/src/app/modules/Profile/profile.model.ts` | User profile data, patient demographics, clinical summary |
| **Organization** | MongoDB Model | `_id` | `name`, `email`, `image`, `phoneNumber`, `addressLine1`, `addressLine2`, `city`, `state`, `postCode`, `country`, `timeZone`, `countryAndTimezone`, `organizationType`, `organizationSport`, `organizationStatus`, `createdBy`, `joinedAt`, `createdAt`, `updatedAt` | `ab3-medical-api-v2/src/app/modules/Organization/organization.model.ts` | Organization management, scoping users, teams, problems |
| **OrganizationUser** | MongoDB Model | `_id` | `organization`, `user`, `status`, `userType`, `fitnessStatus`, `condition`, `invitedBy`, `joinedAt`, `lastLogin`, `isActive`, `createdAt`, `updatedAt` | `ab3-medical-api-v2/src/app/modules/OrganizationUser/organizationUser.model.ts` | User–organization membership, role and fitness status within org |
| **Team** | MongoDB Model | `_id` | `name`, `teamLogo`, `organization`, `createdBy`, `status`, `createdAt`, `updatedAt` | `ab3-medical-api-v2/src/app/modules/Team/team.model.ts` | Teams inside organizations; assignment targets for problems and invites |
| **TeamUser** | MongoDB Model | `_id` | `team`, `user`, `status`, `userType`, `invitedBy`, `createdAt`, `updatedAt` | `ab3-medical-api-v2/src/app/modules/TeamUser/teamUser.model.ts` | User–team membership; which medics/patients belong to which team |
| **Problem** | MongoDB Model | `_id` | `patient`, `organization`, `title`, `problemType`, `onsetDate`, `status`, `isFlagged`, `visibilityScope`, `resolvedAt`, `createdBy`, `diagnosis`, `initialAssessment`, `attachments[]`, `severity`, `isVisibleToPatient`, `assignedUsers[]`, `assignedTeams[]`, `externalUsers[]`, `injuryLocations[{ bodyRegion, laterality }]`, `illnessSeverity`, `surgeryProcedureName`, `surgeryLocations[{ bodyRegion, laterality }]`, `screeningType`, `allergyAllergen`, `allergyCategory`, `reactionType`, `allergyReactionSeverity`, `createdAt`, `updatedAt` | `ab3-medical-api-v2/src/app/modules/Problem/problem.model.ts` | Core clinical problems / episodes; drive records, assignments, and workflows |
| **Note (Record)** | MongoDB Model | `_id` | `consultationDate`, `category`, `noteDetails`, `labels[]`, `attachments[]`, `playerCondition`, `problem`, `patient`, `organization`, `consultant`, `createdBy`, `mentions[]`, `seenBy[]`, `likedBy[]`, `noteStatus`, `isAddToCoachReport`, `isTreatmentObtained`, `isDoctorOnly`, `isInitialAssessment`, `isVisibleToPatient`, `createdAt`, `updatedAt` | `ab3-medical-api-v2/src/app/modules/Note/note.model.ts` | Clinical notes **(records in UI)** linked to problems and patients |
| **NoteReply** | MongoDB Model | `_id` | `note`, `noteReplyDetails`, `isDoctorOnly`, `attachments[]`, `createdBy`, `replyTo`, `mentions[]`, `seenBy[]`, `likedBy[]`, `createdAt`, `updatedAt` | `ab3-medical-api-v2/src/app/modules/NoteReply/noteReply.model.ts` | Threaded replies/discussion on records (notes) |
| **Diagnosis** | MongoDB Model | `_id` | `diagnosisName` (encrypted), `diagnosisCode` (encrypted), `status`, `createdAt`, `updatedAt` | `ab3-medical-api-v2/src/app/modules/Diagnosis/Diagnosis.model.ts` | Diagnosis code dictionary used in problems |
| **EmergencyContact** | MongoDB Model | `_id` | `emergencyContactName`, `emergencyContactRelationship`, `emergencyContactPhoneE164` (encrypted), `emergencyContactEmail`, `emergencyContactAddress`, `emergencyContactAddressLine1`, `emergencyContactAddressLine2`, `emergencyContactCity`, `emergencyContactState`, `emergencyContactPostcode`, `emergencyContactCountry`, `user`, `createdAt`, `updatedAt` | `ab3-medical-api-v2/src/app/modules/EmergencyContact/emergencyContact.model.ts` | Stored emergency contacts for a user/patient |
| **MedicalIdentifiers** | MongoDB Model | `_id` | `nhsNumber` (encrypted), `gpName` (encrypted), `gpFirstName` (encrypted), `gpLastName` (encrypted), `gpPhone` (encrypted), `surgeryName` (encrypted), `nationalHealthId` (encrypted), `gpAddress` (encrypted), `gpAddressLine1` (encrypted), `gpAddressLine2` (encrypted), `gpCity` (encrypted), `gpState` (encrypted), `gpPostcode` (encrypted), `gpCountry` (encrypted), `user`, `createdAt`, `updatedAt` | `ab3-medical-api-v2/src/app/modules/MedicalIdentifiers/medicalIdentifiers.model.ts` | NHS/GP identifiers and contact details for patients |
| **Session** | MongoDB Model | `_id` | `user`, `deviceName`, `deviceId` (encrypted), `browserName`, `ipAddress` (encrypted), `lastLogin`, `token` (encrypted), `authToken` (encrypted), `createdAt`, `updatedAt` | `ab3-medical-api-v2/src/app/modules/Session/session.model.ts` | Login sessions, device and token tracking |
| **Form** | MongoDB Model | `_id` | `formName`, `formDescription`, `status`, `formData` (JSON/mixed), `createdBy`, `createdAt`, `updatedAt` | `ab3-medical-api-v2/src/app/modules/Form/form.model.ts` | Dynamic form builder definitions |
| **Scat6** | MongoDB Model | `_id` | Top-level: `patient`, `organization`, `createdBy`, `athleteName`, `idNumber`, `dateOfBirth`, `dateOfExamination`, `timeOfInjury`, `sex`, `dominantHand`, `sport`, `currentYear`, `completedYearsOfEducation`, `firstLanguage`, `preferredLanguage`, `examinerName`, `pastDiagnosisNumber`, `lastConcussionDate`, `primarySymptoms`, `recentRecoveryTime`, `isBaseLine`; `step1`: `hospitalized`, `migraine`, `dyslexia`, `adhd`, `psychologicalDisorder`, `notes`, `currentMedications`; `step2`: all symptom scores and metadata – `headaches`, `pressureInHead`, `neckPain`, `nauseaOrVomiting`, `dizziness`, `blurredVision`, `balanceProblems`, `sensitivityToLight`, `sensitivityToNoise`, `feelingSlowedDown`, `feelingLikeInAFog`, `dontFeelRight`, `difficultyConcentrating`, `difficultyRemembering`, `fatigueOrLowEnergy`, `confusion`, `drowsiness`, `moreEmotional`, `irritability`, `sadness`, `nervousOrAnxious`, `troubleFallingAsleep`, `isBaseLine`, `timeElapsed`, `timeElapsedUnit`, `worseWithPhysicalActivity`, `worseWithMentalActivity`, `percentOfNormal`, `whyNot100Percent`, `totalSymptoms`, `symptomSeverityScore`; `step3`: cognitive & orientation fields – `wordList`, `digitList`, `timeLastTrialCompleted`, `monthsTimeTaken`, `monthsErrors`, `monthsScore`, `concentrationScore`, `orientationScore`, `immediateMemoryScore`, `digitsScore`, `trial1Total`, `trial2Total`, `trial3Total`, all individual `trialXY` flags, orientation answers (`month`, `date`, `day`, `year`, `time`), reverse month selections (`month1`–`month12`), digit flags (`digitList1`–`digitList4`); `step4`: coordination/balance – `footTested`, `surface`, `footwear`, `notCompletedReason`, `bessFirm{double,tandem,single,total,timeCompleted}`, `bessFoam{double,tandem,single,total,timeCompleted}`, `gait{trial1,trial2,trial3,average,fastest}`, and `dualTask.practice` / `dualTask.trial1` / `dualTask.trial2` / `dualTask.trial3` each with `time`, `errors` and their number fields; `step5`: delayed recall – `delayedRecallWordList`, `delayedRecallTimeStarted`, `delayedRecall1`–`delayedRecall10`, `delayedRecallScore`, `totalCognitiveScore`, `athleteDifferentFromUsualSelf`, `neurologicalExam`; `step6`: decision – `concussionDiagnosed`, `hcpName`, `hcpTitleSpeciality`, `hcpRegistrationLicense`, `hcpDate`, `additionalClinicalNotes`, `clinicalNotesRegistrationLicense`; `step15`: observable signs & GCS – nested `step1` observational flags, `step2` GCS fields (`eyeResponse*`, `verbalResponse*`, `motorResponse*`, `gcsScore*`), `step3` neck/limb checks, `step4` coordination/eye movement checks, `step5` Maddocks questions and `maddocksScore`; plus `createdAt`, `updatedAt` | `ab3-medical-api-v2/src/app/modules/Scat6/scat6.model.ts` | Full SCAT6 concussion assessment payload |
| **OrganizationType** | MongoDB Model | `_id` | `organizationTypeName` (encrypted, unique), `status`, `createdAt`, `updatedAt` | `ab3-medical-api-v2/src/app/modules/OrganizationType/OrganizationType.model.ts` | Reference data for organization categories |
| **OrganizationSport** | MongoDB Model | `_id` | `organizationSportName` (encrypted, unique), `status`, `createdAt`, `updatedAt` | `ab3-medical-api-v2/src/app/modules/OrganizationSport/OrganizationSport.model.ts` | Reference data for organization sports/disciplines |

---

## Field Encryption Notes

The following objects have encrypted fields (encrypted at rest using Mongoose getters/setters):

- **Diagnosis**: `diagnosisName`, `diagnosisCode`
- **MedicalIdentifiers**: All fields (NHS number, GP info, etc.)
- **EmergencyContact**: `emergencyContactPhoneE164`
- **Scat6**: All athlete information fields (name, DOB, scores, etc.)
- **Session**: `deviceId`, `ipAddress`, `token`
- **OrganizationType**: `organizationTypeName`
- **OrganizationSport**: `organizationSportName`

**Evidence:**
- `ab3-medical-api-v2/src/helper/encrypt.ts`
- `ab3-medical-api-v2/src/helper/decrypt.ts`

---

**Document End**


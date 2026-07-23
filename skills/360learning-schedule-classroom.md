---
name: Schedule a classroom and register learners
description: Create an instructor-led classroom, add a scheduled slot, and register learners to it on the 360Learning platform.
api: openapi/360learning-core-api.json
operations:
  - v2.classrooms.CreateClassroomController_createClassroom
  - v2.classrooms.CreateClassroomSlotController_createClassroomSlot
  - v2.classrooms.AddSlotRegistrationController_addSlotRegistration
  - v2.classrooms.CreateAttendanceSheetController_createAttendanceSheet
---

# Schedule a classroom and register learners

Use this flow to stand up instructor-led training (virtual or physical): a
classroom, a scheduled slot (session), and learner registrations, then capture
attendance.

## Prerequisites
- OAuth 2.0 client credentials with scopes `classrooms:write` and `attendanceSheets:write`.
- Bearer token from `POST /api/v2/oauth2/token`; headers `Authorization: Bearer <token>` and `360-api-version: v2.0`.

## Steps
1. **Create the classroom** — `v2.classrooms.CreateClassroomController_createClassroom` (`POST /api/v2/classrooms`). Requires `classrooms:write`.
2. **Add a slot (session)** — `v2.classrooms.CreateClassroomSlotController_createClassroomSlot` (`POST /api/v2/classrooms/{classroomId}/slots`). Requires `classrooms:write`. Set the date/time and capacity.
3. **Register a learner** — `v2.classrooms.AddSlotRegistrationController_addSlotRegistration` (`POST /api/v2/classroom-slots/{classroomSlotId}/registrations/{userId}`). Requires `classrooms:write`. Repeat per learner; if a slot is full, users go to the waitlist via the waitlist endpoints.
4. **Record attendance** — `v2.classrooms.CreateAttendanceSheetController_createAttendanceSheet` (`POST` under the classroom slot). Requires `attendanceSheets:write`.

## Rules
- Registration and attendance list endpoints are paginated (Link header, rel="next").
- No idempotency key — check existing registrations (`GetSlotRegistrationsController`) before re-adding.
- Errors return `{code, message}`; a `409` indicates a conflict (e.g. already registered).

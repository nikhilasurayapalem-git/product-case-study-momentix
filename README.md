# Product Case Study: Group Program and League Management for Momentix

Nikhila Surayapalem

---

## About This Case Study

Momentix is a fictional, multi tenant SaaS platform for service based businesses, gyms, studios, clinics, and clubs, to manage scheduling, memberships, and recurring programs. This case study is a self directed exercise built to demonstrate core product management competencies: market and competitive analysis, prioritization under real constraints, requirements engineering, and technical fluency across backend and frontend design. The scenario, company, and every technical detail below are original and fictional. The underlying skills, extracting market signal quickly, sequencing work by risk and value rather than convenience, and writing development tickets precise enough to remove ambiguity for an engineering team, are the same skills I apply across product domains, not specific to any one industry.

**The scenario:** Momentix is considering a new feature area, Group Program and League Management, to let any service business run recurring group sessions (a fitness studio's weekly class series, a clinic's recurring group therapy sessions, a club's recurring league play) inside the same platform members already use for individual bookings.

---

## Part 1: Market Landscape Analysis

Rather than analyze two named competitors, this section compares two common architectural philosophies I have observed across scheduling and membership software, since the tradeoff between them is the more durable insight for a portfolio piece than any single company's current feature set.

### Approach A: Enterprise Event Platform Model

Positioning and core value prop: Products in this category are built for large scale, formally organized events, think multi day tournaments, certification courses, or large enrollment programs, with a strong emphasis on structured registration, credentialing, and reporting for the organizing body. They tend to serve professional or semi professional organizers rather than individual location managers.

Strengths:
- Deep support for structured competitive or credentialing formats, including seeding, multi round progression, and official results tracking.
- Strong reporting and compliance features built for organizations managing many simultaneous events.
- Live, real time status updates during an event, which builds trust with participants and organizers alike.

Weaknesses:
- Heavy setup overhead, more configuration than a single location manager typically needs for a weekly recurring program.
- Interfaces optimized for expert organizers rather than a first time program lead at a single business location.
- Pricing and packaging generally assume an organization running many events per year, not a single location running one recurring program.

Representative feature areas: Event and Program Setup, Registration and Credentialing, Structured Competition or Progression Logic, Reporting and Compliance.

### Approach B: Lightweight Community Scheduling Model

Positioning and core value prop: Products in this category prioritize speed and simplicity for a single location running informal, recurring group activity, optimized for a program lead who wants to get a session running in minutes, not hours.

Strengths:
- Extremely fast setup flow, often reducible to a handful of fields before a session is live.
- Automatic recurring scheduling once a program is configured, removing manual week to week setup work entirely.
- Flexible participant invitation paths that adapt to whether registration happens inside the platform or is still managed informally by the location (sign in sheets, phone lists).

Weaknesses:
- Monetization and access models are often decided once at setup and are difficult to change later without disrupting an active program.
- Minimal identity verification for participants added outside the platform, which can create data quality problems in reporting until those participants create real accounts.
- Program discovery tends to live in a separate part of the product from program creation, so a participant browsing nearby activity and a location manager creating a new program do not share the same context.

Representative feature areas: Rapid Program Setup, Recurring Session Auto Scheduling, Participant Invitation and Check In, Program Discovery.

---

## Part 2: Decomposition and Sequencing

**Closest analog:** Approach B, the lightweight community scheduling model. Momentix's target user for this feature is a single location manager running one recurring program, not an organization managing a portfolio of large formal events. Approach A's depth (credentialing, multi round competitive progression, heavy reporting) is a mismatch for that user's actual job to be done on day one.

**Deliberate scoping:** Approach B's full maturity, multiple session format templates, automatic long horizon scheduling, a locked in monetization choice made at setup, is more than a first release needs. This sequence intentionally scopes down to a single fixed session format, one program at a time rather than an auto generated season, and no monetization decision yet, so the location manager gets a working core loop first. Format templates, long horizon auto scheduling, and monetization model are natural fast follows once that loop is proven. This same underlying platform concept, treating the recurring activity type as a configurable field rather than hardcoding one vertical, also means the sequence extends to other service business categories (fitness, clinical, hobby, professional) without structural rework.

**Success metrics to validate the sequence:** time from program creation to first published session, percentage of programs that reach a second scheduled session (a proxy for whether the core loop is actually usable), and support ticket volume tied to scheduling confusion in the first 30 days.

**Sequenced tickets:**

1. **Location manager creates a new recurring program.** Basic program creation: name, activity category, format, date, time, location, participant limit. Rationale: nothing downstream can exist until a program record exists. Full stack, needs a location facing creation form and a backend record.

2. **Member views and registers for a program.** Registration flow enforcing whether membership is required before registering, per location configuration. Rationale: registration is the next gating step before any pairing or scheduling logic matters. Full stack, registration UI plus backend enforcement of gating rules.

3. **Waitlist and automatic promotion for full programs.** Enforce participant limit, maintain a waitlist, auto promote the next member when someone drops. Rationale: location managers need a reliable, trustworthy roster before any session structure can be generated. Full stack, waitlist UI state plus backend promotion logic.

4. **Session pairing or grouping generation and schedule display.** Backend logic generates a fair grouping or pairing structure from the confirmed roster, with location manager and member facing views of the resulting schedule. Rationale: this is the technical core of the feature and depends on having a stable, confirmed roster. Full stack, grouping algorithm plus schedule display UI.

5. **Session result or attendance entry.** Member and location facing entry for attendance or results per session. Rationale: participants need a way to act on the structure generated in the prior ticket. Full stack, entry UI plus backend storage.

6. **Program standings or progress dashboard.** Aggregate attendance or results into a simple dashboard for members and location managers. Rationale: this is the payoff feature once session data exists, and depends on the prior ticket. Full stack, dashboard display driven by backend aggregation.

**Stakeholder alignment for this sequence (RACI):**

| Decision | Responsible | Accountable | Consulted | Informed |
|---|---|---|---|---|
| Feature scope and sequencing | Product Manager | Head of Product | Engineering Lead, Design Lead | Location Manager Advisory Panel |
| Data model and API design | Engineering Lead | Engineering Lead | Product Manager | QA Lead |
| Grouping and pairing fairness logic | Engineering Lead | Product Manager | Data Science (if available) | Support Lead |
| Monetization model (deferred) | Product Manager | Head of Product | Finance, Legal | Engineering Lead |

---

## Part 3: Two Tickets at Different Granularities

Both tickets follow a consistent format I use across ticket writing: a header block, a plain language background section, a goal statement, a user journey for anything with a user facing surface, an explicit in scope and out of scope split, a technical approach with real schema and API shape, binary acceptance criteria, edge cases, and open questions with a named owner and resolution target.

### Ticket A (small, focused)

**Ticket ID:** PROG-001
**Title:** Location manager creates a new recurring program
**Surface:** Location Admin
**Status:** Draft
**Layer:** Full stack
**Dependencies:** None
**Blocks:** PROG-002 (Member registration for programs)

**Background**

**Program:** A scheduled recurring activity hosted by a location, such as a weekly class series, a recurring clinical group session, or a recurring league. A Program is distinct from a Membership, which is a standing relationship between a member and a location.

**Activity Category:** The type of activity associated with a Program, for example fitness, clinical, or recreational. Each location may support multiple Activity Categories depending on its offerings.

**Recurring Format:** A Program format in which the same group of participants meets on a repeating schedule, as distinct from a single one time event. For example, a Program with a weekly recurring format generates a new session each week for the same enrolled group.

**Goal**

Allow a location manager to create a new recurring Program with basic identifying information, so that members have something to discover and register for later in the sequence.

**User Journey**

1. Location manager navigates to the Programs section of the Location Admin surface and selects Create Program.
2. Location manager enters the Program name, selects an Activity Category from the location's configured list, selects the recurrence format, and enters the start date, time, and location or room.
3. Location manager enters a maximum participant count for the Program.
4. Location manager selects Save Draft or Publish. Save Draft stores the Program without making it visible to members. Publish makes the Program visible to members per the membership requirement configured for that Activity Category.
5. Location manager sees the new Program listed in the location's Programs list with its current status, Draft or Published.

**Scope**

In scope:
- Creating a Program record with name, Activity Category, recurrence format, start date, time, location, and maximum participant count.
- Draft and Published status for a Program.
- Displaying the created Program in the location manager's Programs list.

Out of scope:
- Generating session groupings or schedules.
- Member registration flow.
- Payment or fee collection for the Program.
- Multiple simultaneous recurrence patterns; this ticket covers a single weekly or set cadence per Program.

**Technical Approach**

```sql
CREATE TABLE programs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    location_id UUID NOT NULL REFERENCES locations(id),
    activity_category_id UUID NOT NULL REFERENCES activity_categories(id),
    name TEXT NOT NULL,
    recurrence_format TEXT NOT NULL CHECK (recurrence_format IN ('weekly', 'biweekly', 'custom')),
    start_date DATE NOT NULL,
    start_time TIME NOT NULL,
    room_or_location TEXT,
    max_participants INTEGER NOT NULL CHECK (max_participants > 0),
    status TEXT NOT NULL DEFAULT 'draft' CHECK (status IN ('draft', 'published')),
    created_by UUID NOT NULL REFERENCES staff(id),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_programs_location_status ON programs (location_id, status);
```

```
POST /api/location/:locationId/programs
Request body:
{
  "name": string,
  "activityCategoryId": uuid,
  "recurrenceFormat": "weekly" | "biweekly" | "custom",
  "startDate": "YYYY-MM-DD",
  "startTime": "HH:MM",
  "roomOrLocation": string or null,
  "maxParticipants": integer,
  "status": "draft" | "published"
}

Response 201:
{
  "id": uuid,
  "status": "draft" | "published",
  "createdAt": timestamp
}
```

Pseudocode for the create handler:

```
function createProgram(locationId, payload, actingStaff):
    validate actingStaff has permission to manage Programs for locationId
    validate payload.maxParticipants > 0
    validate payload.activityCategoryId belongs to locationId's configured categories
    program = insert into programs (...)
    return program
```

**Acceptance Criteria**

- A location manager with permission to manage Programs can create a Program with name, Activity Category, recurrence format, date, time, location, and max participants.
- A Program saved as Draft does not appear in the member facing Program list for that location.
- A Program saved as Published appears in the location manager's Programs list with status Published visible on screen.
- Attempting to create a Program with max participants of 0 or less returns a 400 error and the form displays a validation message without navigating away.
- Attempting to select an Activity Category not configured for the location returns a 400 error.

**Edge Cases**

- A location with no configured Activity Categories should not be able to reach the Create Program form. The Activity Category selector should be empty and prompt the manager to configure categories first.
- Changing a Program from Published back to Draft after members have already registered is not addressed by this ticket and should be tracked separately if needed.

### Ticket B (larger, multi component)

**Ticket ID:** PROG-004
**Title:** Session grouping generation and schedule display
**Surface:** Location Admin, Member
**Status:** Draft
**Layer:** Full stack
**Dependencies:** PROG-001 (Program creation), PROG-002 (Member registration), PROG-003 (Waitlist and roster confirmation)
**Blocks:** PROG-005 (Attendance or result entry), PROG-006 (Program standings dashboard)

**Background**

**Roster:** The confirmed list of members registered for a specific Program, excluding anyone still on the waitlist.

**Session:** A single scheduled occurrence within a recurring Program in which every active participant on the Roster is grouped for that meeting.

**Grouping:** The assignment of participants into pairs, teams, or subgroups for a specific Session, depending on the activity type.

**Unassigned Slot:** A Session in which a participant has no group assignment, typically because the Roster has an odd number of participants relative to the grouping size required.

**Goal**

Generate a fair grouping structure from a Program's confirmed Roster for each Session, and display the resulting schedule to both the location manager and registered members.

**User Journey**

1. Location manager opens a Published Program whose registration period has closed and selects Generate Schedule.
2. System generates Session groupings from the confirmed Roster and displays the full schedule to the location manager for review, including any Unassigned Slots.
3. Location manager selects Confirm Schedule to make it visible to members, or selects Regenerate to produce a new grouping set before confirming.
4. Member opens the Program from the Member surface and sees their personal session schedule: group or partner assignment, session date, and approximate time.
5. Member can also view the full Program schedule, not only their own sessions.

**Scope**

In scope:
- Generating Session groupings for a recurring Program from the confirmed Roster.
- Assigning an Unassigned Slot when the Roster does not divide evenly, rotating who receives it across sessions where more than one session exists.
- Displaying the full schedule to the location manager with a Regenerate option before confirmation.
- Displaying confirmed schedules to members, both their personal sessions and the full Program schedule.

Out of scope:
- Attendance or result entry.
- Formats requiring elimination or bracket style progression; this ticket covers recurring, non elimination formats only.
- Room or equipment assignment per session; this ticket generates participant groupings and session dates only, not physical resource scheduling.
- Automatic rescheduling if a member cancels after the schedule is confirmed.

**Technical Approach**

```sql
CREATE TABLE program_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    program_id UUID NOT NULL REFERENCES programs(id),
    session_number INTEGER NOT NULL,
    UNIQUE (program_id, session_number)
);

CREATE TABLE session_groupings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    program_session_id UUID NOT NULL REFERENCES program_sessions(id),
    participant_one_id UUID NOT NULL REFERENCES program_registrations(id),
    participant_two_id UUID REFERENCES program_registrations(id),
    is_unassigned BOOLEAN NOT NULL DEFAULT false,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    CHECK (
        (is_unassigned = true AND participant_two_id IS NULL)
        OR (is_unassigned = false AND participant_two_id IS NOT NULL)
    )
);

CREATE UNIQUE INDEX idx_groupings_no_duplicate_participant
    ON session_groupings (program_session_id, participant_one_id);
```

```
POST /api/location/:locationId/programs/:programId/schedule/generate
Response 200:
{
  "programId": uuid,
  "sessions": [
    {
      "sessionNumber": integer,
      "groupings": [
        { "participantOneId": uuid, "participantTwoId": uuid or null, "isUnassigned": boolean }
      ]
    }
  ]
}

POST /api/location/:locationId/programs/:programId/schedule/confirm
Response 200:
{ "programId": uuid, "status": "schedule_confirmed" }

GET /api/member/programs/:programId/schedule
Response 200:
{
  "mySessions": [ { "sessionNumber": integer, "partnerName": string, "approximateTime": string or null } ],
  "fullSchedule": [ same shape as generate response ]
}
```

Pseudocode for grouping generation:

```
function generateGroupings(roster):
    participants = roster.confirmedParticipants
    if len(participants) does not divide evenly:
        assign one participant an Unassigned Slot for this generation,
            rotating who received it in prior sessions if regenerating
    remaining = participants minus the unassigned participant
    shuffle remaining using a seeded random order so results are
        reproducible when compared across Regenerate attempts
    group remaining into participant pairs or subgroups
    avoid grouping together participants who were already grouped together
        in an earlier session of the same Program, where an alternate
        grouping exists that does not repeat
    return sessions with groupings
```

**Acceptance Criteria**

- Selecting Generate Schedule on a Program with a confirmed Roster produces one grouping per participant per session, with no participant appearing twice in the same session.
- A Program with a Roster that does not divide evenly assigns exactly one Unassigned Slot per session.
- The location manager sees the generated schedule on screen before it is confirmed, including a visible Regenerate option.
- After the location manager selects Confirm Schedule, a member on the Roster can view their personal sessions on the Member surface, showing partner or group assignment and session number.
- A member can also view the full Program schedule, not only their own sessions.
- Regenerating the schedule before confirmation replaces the previously generated groupings rather than appending to them.

**Edge Cases**

- A Program with fewer than two confirmed participants cannot generate a schedule. The Generate Schedule action should be disabled with an explanatory message rather than failing silently.
- If a member cancels their registration after the schedule is confirmed, this ticket does not handle removing or reassigning their groupings. That gap should be tracked as a follow up ticket.
- Retention for session and grouping records follows the location's standard data retention period once a Program concludes. The exact retention duration is location configurable and is not defined by this ticket.

**Open Questions**

- OQ-1: Should repeat groupings, the same two participants grouped together twice, be strictly disallowed even when the Roster size makes it mathematically unavoidable across many sessions, or is a soft preference sufficient? Owner: Head of Product. Resolution target: before this ticket can be promoted to Ready for Development.
- OQ-2: This ticket stores and displays member linked personal data, partner or group assignments, in confirmed schedules. If a member's account is erased under applicable data privacy regulation after a schedule has been confirmed, should their existing grouping records be soft deleted, preserving the session structure with an anonymized placeholder, or hard erased, removing the grouping and leaving an Unassigned Slot in its place? Owner: Head of Product. Resolution target: before this ticket can be promoted to Ready for Development.

---

## What This Demonstrates

This exercise is meant to show a consistent product management approach that transfers across industries: extracting signal from a market quickly without getting lost in exhaustive detail, making and defending a deliberate scope cut rather than trying to build everything at once, and writing requirements precise enough that an engineer, a designer, and a QA lead could all pick up the same ticket and build the same thing without a clarifying meeting.

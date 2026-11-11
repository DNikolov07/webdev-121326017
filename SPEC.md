# <Study group · CoLearn>

<!-- Exercise 2. Write the non-goals first. Every acceptance criterion must be checkable in under a minute.
     Keep these headings exactly as they are: the self-checks look for them by name.
     The full target is on the course site under Resources → The course project. -->

## Project option
<!-- One line: lab booking, study groups, club events, or the own project the lecturer approved. -->
Study groups

## Goals
- A student browses available study groups by course code, filters by study style or location, and joins a group.
- A group creator sets up a study group with a maximum member limit, sets a topic or a schedule and tracks who has signed up.
- Members create or vote on meeting times and the app automatically generates a calendar invite with a Zoom link or campus location.

## Non-goals
- No payments: every event is free.
- No email or push notifications: members check the site.
- No mobile app: one web UI that also works on a phone screen.
- No Dating or Casual Social Matching: all group discovery is grounded strictly in subjects, courses, or shared learning goals.
- No Official Academic Record Keeping: will not track attendance for official credit or generate reports for university administration or professors.
- No Native Educational Content: will not create or host courses, lectures, or study materials.

## Constraints
- Node with Express, SQLite, plain HTML and JavaScript in the UI.
- Passwords are stored only as a hash, never as readable text.

## The model feature
<!-- Which model, which modality, and what it does for a user.
     Name three things or the check fails: the input it receives, the output it produces,
     and what happens when the model is slow, rate-limited or wrong. -->
- Model: Gemini 3.8 Flash called from `src/services/suggest.js` and nowhere else.
- Input: The title and the free-text notes the group owner types for a study session and nothing about any member.
- Output: A short plan of at most five bullet points for the session, shown in the session form for the owner to edit.
- When it fails: No key, a timeout or a 429 all leave the plan field empty, with the note "Suggestion unavailable, write the plan yourself". If the answer is empty or longer than five points, it is discarded. Nothing is saved until the owner presses "Save session".

## Auth
<!-- Local sign-in plus at least one provider. Say how an account signing in both ways stays one account. -->
- Local: e-mail and password, stored only as a bcrypt hash.
- Provider: Sign in with Google, the authorization-code flow on the server, with `state` checked.
- Linking: matched on Google's account id. If Google reports the e-mail as verified and it already belongs to a password account, Google is attached to that same account. An unverified Google e-mail is never linked, and no second row appears in `users`.

## MCP tools
<!-- At least three tools your own MCP server exposes over this project's data.
     Each tool calls the same service functions the HTTP API calls, so the same rules apply.
     A description shorter than 20 characters is not a description. -->
| Tool | What it does | Who may call it |
| `my_upcoming_sessions` | Lists the upcoming study sessions, with time and place, of every group the token's owner belongs to | the token's owner |
| `open_groups` | Lists open groups for a given course that a student can still join | anyone |
| `join_group` | Adds the token's owner to an open group, refusing a group they are already in | the token's owner |

## UI
<!-- Stack, and the three or more screens. The graded checklist lives in docs/UI.md. -->
Plain HTML, CSS and JavaScript served by Express. Three screens: sign in, my groups with their upcoming sessions, and one group with its members and a form to schedule a session (for the owner). Each screen handles loading, empty, error and success states. The checklist is in `docs/UI.md`.

## Architecture
<!-- Exercise 3. One request traced through every layer, naming the file for each one,
     plus one place where your own code does not match this layering. -->

## Acceptance criteria
<!-- Every criterion carries an id. A test proves it by naming that id in its title, for example
     test('AC-3 a booking that overlaps another is refused with 409', ...).
     Say who acts, what they see, and what must be true in the database. -->
- [AC-1] A signed-in member who opens `/groups` and filters by a course code sees only active study groups for that course.
- [AC-2] A member who signs up for a study group with no free places sees the message "Group Full" and no new row appears in the `memberships` table.
- [AC-3] An organizer who opens a study group details page sees the list of members signed up, while a non-organizer who opens the same group does not see that list.
- [AC-4] A member who is already signed up for a group and presses sign up again gets a 409 Conflict status code and no duplicate row is added to `memberships`.
- [AC-5] With no Gemini key set, an organizer who types a title and notes for a session sees an empty plan field with the note "Suggestion unavailable, write the plan yourself", and can still save the session.
- [AC-6] A search query submitted with an invalid or non-existent course code renders an empty list with the message "No study groups found for this course".
- [AC-7] Cancelling a study group membership frees the place, and the group immediately accepts a new registration.
- [AC-8] An organizer who deletes a scheduled session no longer sees it on the group page, no row for it remains in the `sessions` table, and it disappears from the member's `my_upcoming_sessions` result.
- [AC-9] The MCP tool `join_group` refuses a request for a full study group with the same message the API returns, "Group Full", and no new row appears in `memberships`.

### Slice 1: sign-in and active study groups list
- [AC-10] A visitor who opens `/groups` without signing in lands on `/login` instead.
- [AC-11] After signing in with a seeded account, `/groups` shows the seeded active study groups, each with its name, course code and number of free places, and no closed group.
- [AC-12] Deleting one group row in the database and reloading `/groups` removes that group from the screen.
- [AC-13] A visitor who signs in with a wrong password sees the message "Wrong e-mail or password" and gets no session cookie.

## Traceability
<!-- Filled in from Exercise 3 onward: each AC id, and the test that names it. -->
| Criterion | Test |
| --- | --- |
| AC-1 |  |

## Spec vs result
<!-- After the agent builds the slice: which criteria passed, which it missed or reinterpreted, and what you changed in this spec because of it. -->

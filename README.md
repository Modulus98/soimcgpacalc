# Student Marks, Consultation, Mentoring & Impian Quest

A static web app (no server of our own) on **Vercel**, backed by one **Firebase** project (Authentication + Firestore).
Lecturers work in `admin-hub.html`; students sign in with Google.

## Files

| File | Who uses it | What it is |
|---|---|---|
| `index.html` | everyone | Landing page - links to the pages below |
| `report.html` | students | Coursework results dashboard (Half-Semester, Semester Ongoing, End of Semester, Archived), what-if calculator, consultation booking |
| `mentoring.html` | mentees | Mentor-mentee goals and notes |
| `admin-hub.html` | lecturer | Reports, publishing, consultation booking, mentor-mentee, settings, **Game Questions** |
| `game.html` | students | **Impian Quest** - a cute open-world gacha game; correct answers earn wishes |
| `chars/` | game | 19 character images (`<id>.webp`). Must sit next to `game.html` |

## Deploying

1. Upload everything in this folder to the GitHub repo (keep `chars/` next to `game.html`). Vercel redeploys automatically.
2. Firebase console -> Firestore -> Rules: use the rules shown in the admin hub's "Firebase isn't configured yet" panel
   (swap in your admin email). They cover every collection below.
3. Firebase console -> Authentication: Google sign-in enabled (students), Email/Password enabled (admin), and the Vercel
   domain listed under Authorized domains.

## Firestore collections

| Collection | Purpose | Student access |
|---|---|---|
| `publishedResults/{email}/reports/{subject}_{mode}` | published marks reports | read own |
| `publishLog` | audit trail of publishes | none |
| `adminSettings/{dashboard,consultation}` | default tab, **current semester**, booking settings | read |
| `consultationRequests`, `reservedSlots` | consultation booking | create/read own |
| `adminWorkingState`, `mentorWorkingState` | lecturer drafts | none |
| `publishedMenteeData`, `menteeOwnContent` | mentoring | read/write own |
| `studentGameSaves/{email}` | a student's game save | read/write own |
| `gameQuestionBanks/{subjectId}` | a subject's quiz questions (no student data) | read |
| `gameEnrollments/{email}` | which subjects a student takes, per semester | read own |

## Impian Quest - how questions stay in scope

1. **Settings -> Current semester** (e.g. `Sem 1, 2026/2027`).
2. For each subject: fill in **Semester / Session**, load the marksheet with student emails.
3. **Report tab -> 8. Game Questions**: add questions (by hand, bulk paste, or AI draft within the "topics in scope"), then
   press **Publish game questions & enrol students**. Publish again whenever the roster or questions change.
4. A student is only asked questions from the subjects they are enrolled in for the current semester. One correct answer =
   one wish; each question pays once, and a wrong answer locks it for 24 hours. Students with no subject questions at all
   (and guests) get a small daily-capped General Maths practice.

Adding a character: put `chars/<id>.webp` in the folder and add one `C(...)` line to `CHAR_LIST` in `game.html`.

## Notes

- There is no server, so answer keys reach the student's browser with their questions. Treat the game as motivation, not assessment.
- Firebase web config values in the HTML are public by design; the Firestore rules are what protect the data.

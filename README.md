# PROG6212-Raceday-Assignment-POE
Part 3 – MVC application
How the MVC app is built
RaceDay.Web/
  Controllers/   Home, Account, Events, Participant, Organiser, Media
  Services/      RaceDayApi (typed HttpClient that calls the API), SessionUser (session state)
  Filters/       SessionAuthorize attribute (login + role check for pages)
  Models/        API data shapes and form view models (no business logic)
  Views/         Razor views, shared layout, partials
  wwwroot/       site.css (RaceDay theme), site.js
No database code and no business logic in the MVC project. All reads and writes are calls to the Part 2 API through RaceDayApi. Validation rules, ownership checks and role rules are enforced by the API.
Session management: after login the API's JWT, the role and the user's name are stored in server-side ASP.NET Core session. SessionAuthorize checks the session on every protected page: not logged in → redirected to the login page (and back after login); wrong role → "access denied" page; expired token → session ended and redirected to login. Log out clears the session.
Role-aware navigation: Organisers see Dashboard / My Events / Categories / Enrolments / Results / Profile. Participants see Events / My Enrolments / My Results / Profile. Guests see Events / Log in / Sign up.
Errors are friendly: form errors are shown next to the fields, API errors are turned into short sentences, and unexpected failures show a branded error page. Raw exceptions are never shown.
Organiser pages
Dashboard (events created, total enrolments per event, upcoming dates) · create/edit/delete events with banner upload · categories · enrolments (confirm or set pending) · results entry (finish time + position, published straight away).
Participant pages
Home with filters (search, type, date range, include past) · event detail with banner and categories · enrol with a category (recorded immediately, bib number issued) · My Enrolments · My Results (e.g. "47th out of 312 finishers") · Profile with picture upload.
Enrolment statuses are colour coded: green = Confirmed, amber = Pending. New enrolments start as Pending until the organiser confirms them.
Image uploads (wwwroot/uploads)
The API owns all file handling: POST /api/events/{id}/banner (owning Organiser) and POST /api/users/profile/picture (logged-in user). Only JPG/PNG/WEBP/GIF up to 5 MB are accepted, and the file contents are checked, not just the extension.
Images are saved in RaceDay.Api/wwwroot/uploads/events and .../profiles. The API serves them through GET /api/media/{name}.
The MVC app never reads the files itself: it uploads to the API and shows images by asking the API for them (through its own /media/... route).
Uploaded images are not committed to git (.gitignore); the empty uploads folder is kept with a .gitkeep file.
New API endpoints in Part 3
Method
Route
Role
GET
/api/events?search=&type=&from=&to=
Guest (filters added)
GET
/api/events/mine
Organiser – own events with enrolment counts
POST
/api/events/{id}/banner
Organiser (owner) – multipart upload
POST
/api/users/profile/picture
Any logged-in user – multipart upload
GET
/api/media/{name}
Guest – serves an image
PUT
/api/events/{id}/enrolments/{enrolmentId}/status
Organiser (owner) – Pending / Confirmed
Responses also gained: bannerImage, profilePicture, enrolment status and eventDate, and result eventDate and totalFinishers.

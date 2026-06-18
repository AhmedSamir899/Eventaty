# 📅 Eventaty — Event Scheduling & Planning System

<p align="center">
  <em>A multi-role event management platform built with C# Windows Forms and Oracle DB, supporting attendees, organizers, and venue owners through a single integrated desktop application.</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/C%23-.NET%204.7.2-green.svg" alt="C# .NET 4.7.2"/>
  <img src="https://img.shields.io/badge/UI-Windows%20Forms-blue.svg" alt="Windows Forms"/>
  <img src="https://img.shields.io/badge/DB-Oracle%2011g-red.svg" alt="Oracle 11g"/>
  <img src="https://img.shields.io/badge/Driver-ODP.NET-orange.svg" alt="ODP.NET"/>
  <img src="https://img.shields.io/badge/Status-In%20Progress-yellow.svg" alt="Status: In Progress"/>
</p>

---

## 📖 Overview

**Eventaty** is a desktop event-management application designed to bring attendees, organizers, and venue owners into a single coordinated workflow. Built on **C# Windows Forms** with **Oracle Database** as the persistence layer (accessed via **Oracle Data Provider for .NET / ODP.NET**), the project implements role-based access control, normalized relational schema design, and full CRUD operations across the complete event lifecycle — creation, scheduling, enrollment, and management reporting.

> ℹ️ **Current repo state:** This repository currently hosts the foundation scaffold — the ODP.NET↔Oracle connection is wired up and verified against the database, and the project structure is in place. The role-based UIs and business logic (described below) are in active development. See the [Roadmap](#-roadmap) section for the feature trajectory.

---

## ✨ Planned Features

| Feature                              | Description                                                                                                  |
|--------------------------------------|--------------------------------------------------------------------------------------------------------------|
| 👤 **Role-based access control**     | Three distinct user types — attendees, organizers, venue owners — each with separate login flows and permissions. |
| 🎫 **Event creation & scheduling**   | Organizers create events with date, time, capacity, and venue assignment; conflicts flagged automatically.   |
| 🏛 **Venue management**              | Venue owners list and manage their venues, set capacity, and approve/reject booking requests.                |
| 📝 **Enrollment workflow**           | Attendees browse upcoming events, register, and cancel; capacity and duplicate-attendance enforced at the DB layer. |
| 📊 **Management reporting**          | Per-event attendance, per-venue utilization, and per-organizer performance dashboards.                      |
| 🔐 **Authentication & sessions**     | Per-role credential validation; session state held in the application context.                              |
| 🗂 **Normalized relational schema**  | Tables for users, roles, events, venues, enrollments, and audit — with referential integrity and constraints. |
| 🖥 **Desktop-first UI**              | Windows Forms interfaces tailored per role — attendee dashboard, organizer console, venue-owner portal.      |

---

## 🏗 Architecture

### Layered design (intended)

```
┌─────────────────────────────────────────────────────────┐
│  Presentation Layer (Windows Forms)                    │
│  Per-role forms: AttendeeDashboard, OrganizerConsole,  │
│  VenueOwnerPortal, LoginForm, ReportViewer             │
└────────────────────────┬────────────────────────────────┘
                         │ calls
┌────────────────────────▼────────────────────────────────┐
│  Business Logic Layer                                   │
│  (planned) EventService, VenueService, EnrollmentService│
│  Validation, RBAC checks, conflict detection            │
└────────────────────────┬────────────────────────────────┘
                         │ uses
┌────────────────────────▼────────────────────────────────┐
│  Data Access Layer (ODP.NET)                            │
│  OracleConnection + OracleCommand + OracleDataReader    │
│  Hand-written parameterized SQL against the EVENTATY    │
│  schema                                                 │
└────────────────────────┬────────────────────────────────┘
                         │ TNS
┌────────────────────────▼────────────────────────────────┐
│  Oracle Database (orcl)                                 │
│  Schema: eventaty — users, roles, events, venues,       │
│          enrollments, audit_log                         │
└─────────────────────────────────────────────────────────┘
```

### Role-based separation

| Role          | Capabilities                                                                                |
|---------------|---------------------------------------------------------------------------------------------|
| **Attendee**  | Browse events · register · cancel registration · view own enrollment history                |
| **Organizer** | All attendee capabilities · create events · assign venues · view attendance · export reports |
| **Venue Owner** | All attendee capabilities · register venues · approve/reject bookings · view utilization   |

---

## 🗄 Database Schema (intended)

The application targets an Oracle 11g instance (TNS alias `orcl`). The full schema design includes:

| Table           | Purpose                                              | Key Columns                                                                                |
|-----------------|------------------------------------------------------|--------------------------------------------------------------------------------------------|
| `users`         | Registered users                                     | `user_id` (PK), `username`, `password_hash`, `email`, `role_id` (FK)                       |
| `roles`         | Role lookup                                          | `role_id` (PK), `role_name` (ATTENDEE / ORGANIZER / VENUE_OWNER)                            |
| `venues`        | Venues owned by venue-owner users                    | `venue_id` (PK), `owner_id` (FK→users), `name`, `capacity`, `location`                     |
| `events`        | Events created by organizers                         | `event_id` (PK), `organizer_id` (FK→users), `venue_id` (FK→venues), `title`, `start_dt`, `end_dt`, `capacity` |
| `enrollments`   | Attendee ↔ event many-to-many                        | `enrollment_id` (PK), `attendee_id` (FK→users), `event_id` (FK→events), `enrolled_at`, `status` |
| `audit_log`     | Audit trail for sensitive actions                    | `log_id` (PK), `user_id`, `action`, `entity_id`, `at`                                       |

**Constraints enforced at DB level:**
- `events.capacity` ≤ `venues.capacity`
- No two events at the same venue with overlapping time windows (exclusion constraint)
- Unique `(attendee_id, event_id)` in `enrollments` to prevent duplicate registration
- Cascade rules: deleting a user cascades to their enrollments; deleting an event cascades to its enrollments

---

## 🛠 Tech Stack

| Layer            | Technology                                                                                  |
|------------------|---------------------------------------------------------------------------------------------|
| Language         | C# (.NET Framework 4.7.2)                                                                   |
| UI framework     | Windows Forms (Visual Studio designer)                                                      |
| Database         | Oracle Database 11g (TNS alias: `orcl`)                                                     |
| DB driver        | Oracle Data Provider for .NET (ODP.NET) — `Oracle.DataAccess.dll` v2.112.1.0                |
| Build / IDE      | Visual Studio 2022 (solution: `WindowsFormsApp5.sln`)                                       |
| Target platform  | Windows desktop                                                                             |

---

## 📁 Project Structure

```
Eventaty/
├── WindowsFormsApp5.sln                    # Visual Studio solution
└── WindowsFormsApp5/
    ├── Program.cs                          # Entry point — launches Form1
    ├── Form1.cs                            # Main form (foundation: ODP.NET connection test)
    ├── Form1.Designer.cs                   # WinForms designer-generated layout
    ├── Form1.resx                          # Form resources
    ├── App.config                          # .NET runtime config
    ├── WindowsFormsApp5.csproj             # MSBuild project file
    ├── Properties/
    │   ├── AssemblyInfo.cs                 # Assembly metadata
    │   ├── Resources.Designer.cs           # Embedded resources
    │   └── Settings.Designer.cs            # Application settings
    ├── lib/                                # (planned) External DLLs
    └── bin/Debug/
        ├── WindowsFormsApp5.exe            # Built executable
        └── Oracle.DataAccess.dll           # ODP.NET driver (bundled)
```

---

## 🚀 Getting Started

### Prerequisites

- **Visual Studio 2022** (Community Edition is free and sufficient)
- **.NET Framework 4.7.2** targeting pack (included with VS 2022)
- **Oracle Database 11g** or later running locally or on a reachable host
- **ODP.NET** — Oracle.DataAccess.dll (already bundled in `bin/Debug/`; for fresh setups, install ODAC from Oracle's download page)

### 1. Set up the database

The current code connects using the classic Oracle sample schema:

```
Data Source = orcl
User Id     = scott
Password    = tiger
```

> ⚠️ Change these credentials in `Form1.cs` line 12 before running. For production use, create a dedicated Oracle user (`eventaty_app`) with grants only on the application schema.

Verify your TNSNAMES.ORA has an `orcl` alias pointing to your Oracle instance, e.g.:

```
ORCL =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
    (CONNECT_DATA = (SERVER = DEDICATED)(SERVICE_NAME = orcl))
  )
```

### 2. Open the project

1. Clone the repo:
   ```bash
   git clone https://github.com/AhmedSamir899/Eventaty.git
   ```
2. Open `WindowsFormsApp5.sln` in Visual Studio 2022.
3. Restore NuGet packages (if any are missing) — ODP.NET is referenced via a local `HintPath`, so ensure `Oracle.DataAccess.dll` is present in `bin/Debug/` or update the reference to point to your local ODP.NET install.

### 3. Run

Press `F5` (Debug) or `Ctrl+F5` (Run without debugging). The main form opens and, on load, executes `select JOB from EMP` against the Oracle `scott` schema to verify connectivity — the result populates the dropdown on the form.

> ✅ If you see job titles (CLERK, SALESMAN, PRESIDENT, etc.) in the dropdown, the ODP.NET↔Oracle pipeline is working. The next step is to replace this connectivity test with the actual `eventaty` schema and role-based UI.

---

## 🧪 Current functionality (foundation)

The initial commit establishes:

- ✅ **Visual Studio solution structure** for .NET Framework 4.7.2 WinForms
- ✅ **ODP.NET reference** wired into the `.csproj`
- ✅ **Oracle connection** opening on form load (`Form1_Load` event)
- ✅ **Sample query execution** against the `EMP` table — proves the full C# → ODP.NET → Oracle round trip works
- ✅ **Build artifact** — `WindowsFormsApp5.exe` checked into `bin/Debug/`

---

## 🗺 Roadmap

| Phase | Scope                                                                                          | Status |
|-------|------------------------------------------------------------------------------------------------|--------|
| **0** | Project scaffold + ODP.NET connectivity test                                                  | ✅ Done |
| **1** | Schema design — create `eventaty` schema with users, roles, events, venues, enrollments tables | ⏳ Next |
| **2** | Authentication form + role detection (login → role-based dashboard routing)                    | ⏳ Planned |
| **3** | Attendee dashboard — browse events, register, cancel registration                              | ⏳ Planned |
| **4** | Organizer console — create events, assign venues, view attendance                              | ⏳ Planned |
| **5** | Venue owner portal — register venues, approve/reject booking requests                          | ⏳ Planned |
| **6** | Reporting — per-event attendance, per-venue utilization dashboards                             | ⏳ Planned |
| **7** | Polish — input validation, error handling, transactional integrity, audit logging              | ⏳ Planned |

---

## 📜 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details (or feel free to reuse under standard MIT terms).

---

## 👤 Author

**Ahmed Samir**
- GitHub: [@AhmedSamir899](https://github.com/AhmedSamir899)
- LinkedIn: [ahmed-samir-b9503a233](https://www.linkedin.com/in/ahmed-samir-b9503a233)
- Email: AhmedSamir8456@gmail.com

---

## 🙏 Acknowledgements

- Built as part of coursework at **Ain Shams University**, Faculty of Computer & Information Science.
- Oracle sample schema (`scott/EMP`) used in the foundation scaffold — © Oracle Corporation.

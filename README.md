# Covid Centers Management System

**Desktop application for coordinating patients, COVID-19 tests, appointments, prescriptions, and pharmacy inventory at a care facility.**

A Java Swing client backed by MySQL, with role-based screens so admins, reception staff, doctors, and pharmacists each see the tools relevant to their work.

---

## Overview

Public-health crises put enormous pressure on clinics and testing centers: long queues, scattered records, and staff who need different views of the same data. This project models a **single-center workflow**—register patients, schedule doctor visits, log test results, record prescriptions, and track medical supplies—in one place.

It matters because it demonstrates how **operational data** (who was seen, when, and with what outcome) can be structured relationally and accessed through a **clear UI**, similar to ideas used in real hospital information systems—at a scale suitable for learning and portfolio demonstration.

---

## Features

- **Authentication** — Login with username, password, and role; session passes user context into the main dashboard.
- **Role-based dashboard** — Admin, Doctor, Receptionist, and Pharmacist roles hide or show actions (e.g., doctors do not manage users; pharmacists focus on inventory-related tasks).
- **Patient management** — Register and maintain patient demographics and visit-related fields.
- **COVID-19 testing** — Record test type, patient details, dates, and report outcomes.
- **Appointments** — Create appointments linking patients, doctors, rooms, and dates; view and manage scheduled visits.
- **Doctors** — Add doctors and view doctor listings (specialization, qualification, room, contact).
- **Prescriptions** — Create and view prescriptions tied to appointments and doctors.
- **Inventory** — Maintain pharmacy/medical **items** (ID, name, description, quantity).
- **User administration** — Add system users (where permitted by role).
- **Settings & profile** — Utilities for account/settings flows (`Set`, password change, user updates).
- **Quick reference** — Link to WHO COVID-19 information from the main screen.

---

## Tech Stack

| Layer | Technology |
|--------|------------|
| **Language** | Java (project targets **Java 15** per NetBeans configuration) |
| **UI** | **Java Swing** (JFrame forms, GroupLayout, Nimbus look and feel) |
| **Database** | **MySQL / MariaDB** (schema provided; tested with MariaDB 10.x) |
| **Connectivity** | **JDBC** via MySQL Connector/J (`com.mysql.jdbc.Driver`) |
| **Build / IDE** | **Apache Ant** (`build.xml`), **Apache NetBeans** project (`nbproject/`) |
| **Libraries** | MySQL Connector/J, SwingX (see `nbproject/project.properties` for classpath) |

---

## Architecture / System Design

This is a **two-tier desktop application**: the Swing client runs on the user’s machine and talks to MySQL **directly over JDBC** (no separate application server or REST layer).

```text
┌─────────────┐     JDBC (localhost:3306)     ┌──────────────────┐
│  Swing UI   │ ─────────────────────────────►│  MySQL / MariaDB │
│  (forms)    │     SQL (PreparedStatement)   │  database:       │
│  Login →    │                               │  `covidcenter`   │
│  Main +     │                               └──────────────────┘
│  modules    │
└─────────────┘
```

**Flow (typical):**

1. User opens the app → **`Login`** validates credentials against the `user` table.
2. On success → **`Main`** dashboard opens with **username**, **role**, and **date**; buttons are enabled/disabled by `utype`.
3. Each feature (Patient, Test, Appointment, etc.) opens its own JFrame, which opens a **connection** to `jdbc:mysql://localhost:3306/covidcenter` and reads/writes the corresponding tables (`patient`, `test`, `appoinment`, `doctor`, `prescription`, `item`, etc.).

**Data model (high level):**

- `user` — staff accounts and roles  
- `patient` — patient master data  
- `test` — COVID test records and reports  
- `appoinment` — scheduled visits (patient, doctor, room, date)  
- `doctor` — doctor directory  
- `prescription` — prescriptions linked to appointments  
- `item` — inventory lines  

---

## Project Structure

```
Covid-Centers-Management-System/
├── src/                    # Java sources and NetBeans .form files
│   ├── Login.java          # Entry point & DB connection bootstrap
│   ├── Main.java           # Main dashboard (role-based actions)
│   ├── Patient.java        # Patient module
│   ├── Test.java           # COVID test module
│   ├── Appoinment.java     # Appointments
│   ├── Viewappoinment.java
│   ├── Doctor.java, ViewDoc.java
│   ├── Prescription.java, ViewPrescription.java
│   ├── Item.java, Inventory.java
│   ├── User.java, changeU.java, ChangeP.java, DeleteA.java
│   ├── Set.java            # Settings
│   └── About.java
├── nbproject/              # NetBeans project metadata & build hooks
├── build.xml               # Ant build
├── covidcenter.sql         # Database schema + seed admin user
├── MyDB/covidcenter.sql    # Copy of SQL dump (same schema)
├── dist/                   # Built JAR output (after build)
├── build/                  # Compiled classes (generated)
└── manifest.mf
```

---

## Installation & Setup

### Prerequisites

- **JDK 15+** (match `javac.source` / `javac.target` in `nbproject/project.properties`, or adjust the project to your JDK).
- **MySQL or MariaDB** running locally (e.g. via **XAMPP**, **WAMP**, or a standalone server).
- **Apache NetBeans** (optional but recommended) or **Ant** for building.
- **MySQL Connector/J** JAR on the classpath — the NetBeans project references local paths; **clone authors should add** `mysql-connector-*.jar` (and other libs listed in `project.properties`) via NetBeans **Libraries** or by placing JARs under `dist/lib/` when running the packaged app.

### Database

1. Start your MySQL/MariaDB service.
2. Create a database named **`covidcenter`**.
3. Import **`covidcenter.sql`** (phpMyAdmin **Import** tab, or `mysql` CLI):

   ```bash
   mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS covidcenter CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
   mysql -u root -p covidcenter < covidcenter.sql
   ```

4. Default connection in code is:

   ```text
   jdbc:mysql://localhost:3306/covidcenter
   User: root
   Password: (empty string)
   ```

   If your root password is not empty, update the `DriverManager.getConnection(...)` calls in the source (or externalize credentials via configuration—recommended for production).

### Build (NetBeans)

1. Open the project folder as a NetBeans project.
2. Resolve **library paths** for MySQL Connector/J (and SwingX, etc.) to match your machine.
3. **Clean and Build** (F11). The runnable JAR is produced under **`dist/`** (see `dist/README.TXT`).

### Build (Ant)

From the project root:

```bash
ant jar
```

(Run `ant -p` to list targets if your NetBeans import exposes them.)

### Run from built JAR

After a successful build:

```bash
cd dist
java -jar Covid_Centers_Managment_System.jar
```

Distribute the **entire `dist/` folder** including **`lib/`** so all dependencies are on the classpath (per `dist/README.TXT`).

---

## Usage

### Default login (seed data)

| Field | Value |
|--------|--------|
| **Username** | `Admin` |
| **Password** | `Admin` |

*(Defined in `covidcenter.sql` in the `user` table.)*

### Example workflows

1. **Front desk** — Log in as a receptionist-level user → register **patients**, create **appointments**, **view appointments**.
2. **Clinical** — Log in as **Doctor** → use **View Doctors** / prescription flows as enabled; patient and admin actions stay restricted on the dashboard.
3. **Pharmacy** — Log in as **Pharmacist** → manage **items** / inventory-related screens; administrative patient/user actions are disabled on the main form.
4. **Admin** — Full access including **Add User**, **Add Doctor**, and other modules per UI rules.

Logout returns to the **Login** screen.

---

## Screenshots / Demo

> **Placeholder:** Add screenshots of the Login screen, main dashboard, and one module (e.g., Patient or Appointments) here.

```markdown
![Login](docs/screenshots/login.png)
![Dashboard](docs/screenshots/dashboard.png)
```

*(Create a `docs/screenshots/` folder and replace with your own images.)*

---

## API Endpoints

**Not applicable.** This application does **not** expose HTTP/REST endpoints. All business logic runs in the desktop process and communicates with the database through **JDBC** inside classes such as `Login`, `Patient`, `Appoinment`, etc.

If you extend the project with a web or mobile front end, a future layer could expose REST APIs that mirror the same relational model.

---

## Future Improvements

- **Externalize configuration** — JDBC URL, user, and password via properties file or environment variables.
- **Stronger security** — Password hashing (e.g., BCrypt), least-privilege DB user, remove default admin credentials from seed scripts in shared repos.
- **Input validation & UX** — Consistent date handling, form validation, and accessibility (keyboard navigation, larger fonts).
- **Reporting** — PDF exports for prescriptions and test reports; simple analytics dashboard.
- **Automated tests** — Unit tests for DAO-style logic; integration tests against a disposable MySQL instance.
- **Packaging** — jlink or installers for Windows/macOS/Linux with bundled JRE.
- **Optional web API** — Spring Boot backend reusing the same schema for remote access.

---

## Contributing

1. **Fork** the repository and create a feature branch from `main`.
2. **Keep changes focused** — one concern per pull request (e.g., “fix JDBC URL configuration” vs. large unrelated refactors).
3. **Match existing style** — Swing/NetBeans patterns, naming, and SQL style used in the project.
4. **Test locally** — Import DB, run the app, and verify your role still behaves correctly on the main dashboard.
5. Open a **Pull Request** with a short description of what changed and why.

---

## Why This Project Stands Out

- **End-to-end domain modeling** — Covers patients, tests, scheduling, clinical notes (prescriptions), staff, and inventory—not a single-table demo.
- **Role-based UX** — The main window adapts to `utype`, showing how permission concepts map to real front-desk vs. clinical vs. pharmacy workflows.
- **Classic enterprise stack** — Java + Swing + JDBC + MySQL is still widely understood in enterprises and is a strong talking point in interviews when explained clearly.
- **Portfolio-ready narrative** — You can discuss trade-offs (desktop vs. web, direct JDBC vs. API layer) and how you would harden or scale the same design.

---

## License

No `LICENSE` file is included in this repository. **Add a license** (e.g., MIT, Apache-2.0, or “All Rights Reserved”) to clarify how others may use or contribute to the code.


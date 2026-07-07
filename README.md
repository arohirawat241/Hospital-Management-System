# Hospital-Management-System
# LifeLineDB

A hospital management backend built with Spring Boot and MySQL. This was originally scoped as a DBMS project (normalized schema, ER modeling, triggers, views) and implemented here as a working REST API instead of the original Python/Tkinter plan.

Handles the core stuff a hospital system needs to track: patients, doctors, departments, appointments, room admissions, billing, and prescriptions.

## Stack

- Java 17
- Spring Boot 3.3.4 (Web, Data JPA, Validation)
- MySQL 8
- Maven
- Lombok

## Project layout

```
lifelinedb/
├── pom.xml
├── database/
│   └── schema.sql        # plain SQL version of the schema, with triggers + views
├── src/main/java/com/lifelinedb/
│   ├── entity/            # JPA entities
│   ├── repository/        # Spring Data repositories
│   ├── service/            # business logic
│   ├── controller/          # REST endpoints
│   ├── config/              # CORS config
│   └── exception/          # error handling
└── src/main/resources/
    └── application.properties
```

## Getting it running

You'll need Java 17+, Maven, and a MySQL server.

1. Clone the repo and open `src/main/resources/application.properties`. Set your MySQL username/password:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/lifelinedb?createDatabaseIfNotExist=true&useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=yourpassword
```

2. Run it:

```bash
mvn spring-boot:run
```

Hibernate will create the tables automatically on first run (`ddl-auto=update`). If you'd rather set up the schema by hand — including the triggers and views — run `database/schema.sql` in MySQL yourself and flip `ddl-auto` to `validate`.

3. Check it's alive:

```bash
curl http://localhost:8080/api/departments
```
Should return `[]`.

## API

All endpoints are under `/api`. Standard CRUD for every resource, plus a couple of extras:

- `GET /api/rooms/available` — only rooms not currently occupied
- `GET /api/admissions/current` — patients currently admitted (no discharge date yet)
- `PUT /api/admissions/{id}/discharge` — discharge a patient, frees up their room

| Resource | Base path |
|---|---|
| Departments | `/api/departments` |
| Doctors | `/api/doctors` |
| Patients | `/api/patients` |
| Rooms | `/api/rooms` |
| Appointments | `/api/appointments` |
| Admissions | `/api/admissions` |
| Billing | `/api/billing` |
| Prescriptions | `/api/prescriptions` |

Example — create a department:

```bash
curl -X POST http://localhost:8080/api/departments \
  -H "Content-Type: application/json" \
  -d '{"name":"Cardiology","description":"Heart and vascular care"}'
```

## Room status logic

Admitting a patient automatically flips their room to `OCCUPIED`; discharging flips it back to `AVAILABLE`. This is handled in `AdmissionService`, and there's an equivalent pair of MySQL triggers in `database/schema.sql` if you want it enforced at the database level too.

## Notes

- `database/schema.sql` also has three views (`vw_patient_billing_summary`, `vw_doctor_appointment_load`, `vw_current_admissions`) for basic reporting — query them directly if you're inspecting the DB with a MySQL client.
- There's no auth on any of this yet — it's meant as a backend/API layer, not something to expose publicly as-is.
- The original project brief called for a Tkinter/Flask GUI; that's swapped out here for a plain REST API so any frontend can sit on top of it.


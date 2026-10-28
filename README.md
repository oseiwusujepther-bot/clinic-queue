# Clinic Appointment Queue System

Patients book a slot, check in on arrival, and get SMS updates. Staff call the next
patient from a dashboard. A public board shows who is being served.

## Setup (XAMPP/WAMP or any PHP 8.1+ with MySQL)
1. Copy this folder to `htdocs/clinic-queue`.
2. Import `sql/schema.sql` (phpMyAdmin or `mysql -u root -p < sql/schema.sql`).
3. Edit `config/config.php` (DB credentials, `base_url`).
4. Create a staff login (choose your own password): `php tools/create_staff.php admin YOUR_PASSWORD "Clinic Admin"`.
   The schema does not create a default staff account. If you imported an older
   schema that included a placeholder `admin` account, rerun this command using
   the username you want; it replaces that account's password and reactivates it.
   With XAMPP on Windows, run it using `C:\xampp\php\php.exe tools\create_staff.php admin YOUR_PASSWORD "Clinic Admin"`.
5. Open `http://localhost/clinic-queue/public/`

## Record management
Log in as staff and open **Manage records**. Staff can manage patient accounts,
appointments, and queue actions; admins can also manage staff accounts, services,
and notification logs. Patients can update or deactivate their own profile, edit
or cancel booked appointments, and upload their passport photo. Deactivation,
appointment cancellation, queue status changes,
and notification archiving preserve related history instead of deleting records.
Staff accounts are deactivated rather than permanently deleted, and the current
admin cannot deactivate their own account.
Staff can generate an A4-ready patient report from the patient list and print it
using the browser's print dialog.
The staff dashboard also provides a daily served-patients report, with a date
selector and A4 print layout.
When staff mark a patient as served, the system sends a private, expiring
seven-day report link by SMS. Patients can open the link and use the browser's
View report button, then Print / Save as PDF; the confirmation step prevents
automatic SMS link scanners from opening patient data. Set `public_url` in `config/config.php` to the
application's public HTTPS URL before enabling SMS outside local development.

For an existing database, run these migrations once:
```sql
ALTER TABLE patients ADD COLUMN is_active TINYINT(1) NOT NULL DEFAULT 1;
ALTER TABLE patients ADD COLUMN location VARCHAR(150) NULL;
ALTER TABLE patients ADD COLUMN ghana_card_number VARCHAR(15) NULL;
ALTER TABLE services ADD COLUMN is_active TINYINT(1) NOT NULL DEFAULT 1;
ALTER TABLE notifications ADD COLUMN archived_at DATETIME NULL;
ALTER TABLE notifications MODIFY type ENUM('booking_confirmation','queue_update','your_turn','reminder','report_link') NOT NULL;
```
If you already added passport photo support, do not repeat its `passport_photo`
column migration.
Also create the `patient_report_links` table from `sql/schema.sql`.

## Patient passport photos
Patients can upload a JPG, PNG, or WebP photo (up to 5 MB) from their dashboard.
Photos are stored under `storage/patient-photos/`, outside the `public/` directory,
so the web server must be allowed to create and write to that folder and honor the
included `storage/.htaccess` access restriction. The photo endpoint requires the
patient to be logged in and only serves that patient's own image.
For an existing database, run once:
`ALTER TABLE patients ADD COLUMN passport_photo VARCHAR(80) NULL;`

## SMS
`driver => 'log'` records messages in the `notifications` table (good for dev and demos).
To send real sandbox SMS, create an Africa's Talking sandbox app, then set
`driver => 'africastalking'` and your `at_api_key`.

## Automated tests
`composer install` then `composer test` (PHPUnit, no database needed).

## Structure
- `src/QueueService.php`: check-in, call-next, position and wait logic
- `src/SmsService.php`: SMS sending and logging
- `public/`: pages (patient) and `public/staff/` (staff)
- `sql/schema.sql`: database
- `tests/`: automated tests

## Team workflow (for the "commits from every member" requirement)
One feature branch per person, e.g. `feature/booking`, `feature/staff-dashboard`,
`feature/sms`, `feature/tests`. Merge through pull requests with another member as reviewer.

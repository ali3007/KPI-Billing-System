# KPI Billing Evidence Management System

A modular PHP + MySQL application for daily KPI billing evidence. It runs under XAMPP/Apache/MySQL and has no cloud-service dependency at runtime.

## Evidence rules

The UI and server-side validation are driven by `kpis.evidence_type`:

- `pdf` → PDF upload only. Initial: Tonnage Report, Tracker Report.
- `images` → normal pictorial evidence, multiple JPG/JPEG/JFIF/PNG/WEBP/GIF.
- `before_after` → separate Before/After galleries. Initial: Bulk Waste Collection.

Changing a KPI evidence type in **KPI Management** changes the create/add-evidence UI without editing PHP logic.

## XAMPP installation

1. Copy the folder to `C:\xampp\htdocs\kpi_billing_system`.
2. Start Apache and MySQL from XAMPP.
3. Install Composer if it is not already available, then in this folder run:

   `composer install --no-dev --optimize-autoloader`

   This installs Dompdf, PHPWord and Bootstrap locally. No CDN is used by the running application.
4. In XAMPP `php.ini`, ensure these extensions are enabled: `pdo_mysql`, `fileinfo`, `gd`, `zip`, `mbstring`.
5. Browse to `http://localhost/kpi_billing_system/system-check.php` and resolve any missing requirement.
6. Browse to `http://localhost/kpi_billing_system/install.php`, create the admin login, then delete or rename `install.php`.
7. Sign in at `http://localhost/kpi_billing_system/login.php`.

MySQL defaults in `config.php` are `127.0.0.1`, database `kpi_billing`, user `root`, blank password. Change them if your XAMPP setup differs.

## Main modules

- Dashboard with daily progress and evidence KPIs
- KPI master management: add/edit/order/activate/deactivate/safe delete/evidence type
- Create daily billing record with duplicate warning (duplicates intentionally allowed)
- Normal images: bulk/individual selection, preview, remove, drag/reorder, add evidence
- Bulk Waste: separate Before and After galleries and configurable validation
- PDF evidence: strict extension + MIME + `%PDF-` structure checks and secure viewing/downloading
- Billing record search/filter/edit/delete
- Daily completion status using distinct KPI logic
- Individual PDF/Word generation
- Complete daily PDF/Word generation with pending-KPI sections
- Official signature/stamp footer settings
- Secure download handler, CSRF, prepared PDO, password hashing, role checks, upload hardening

## Storage

Evidence is stored under:

`uploads/kpi/YYYY/MM/DD/kpi-slug/`

Bulk Waste stores `before/` and `after/` separately. Random server-side filenames prevent collisions; original filenames remain in MySQL.

Generated reports are stored under:

`generated/YYYY/MM/DD/`

## PDF-only evidence in complete reports

Tonnage/Tracker PDFs remain the authoritative original evidence and are never converted into fake picture evidence. Individual PDF download returns the original PDF. Complete daily PDF/Word documents include a clearly labeled PDF-evidence reference section, while the original remains securely available from the billing record.

## Security notes

- Do not add `UNIQUE(kpi_id, record_date)`; same-day duplicates are supported by design.
- Uploaded executable extensions are blocked by Apache `.htaccess` and upload code accepts only configured image/PDF types.
- File downloads use database IDs and server-side path containment checks.
- Delete/rename `install.php` after setup.
- For LAN/production deployment, use HTTPS, a non-root MySQL account, strong passwords, backups, and move uploads outside the web root if required by your deployment policy.

## Validation performed in this package

All PHP source files were checked with `php -l`. The build environment did not include MySQL/PDO-MySQL/GD/ZIP, so database/report rendering must be exercised in the target XAMPP environment using `system-check.php` and the acceptance tests below.

## Acceptance tests

1. Tonnage: only PDF control appears; image controls stay hidden; upload/view/download works.
2. Tracker: same PDF-only behavior.
3. Container Clearance: multi-select plus repeated selection works; drag order saves; PDF/Word uses four images per page.
4. Bulk Waste: Before and After appear separately; both accept multiple images; report keeps the two groups separated.
5. Duplicate: create the same KPI twice on the same date; warning appears and Continue Anyway creates the second record.
6. Daily status: duplicates count as one completed KPI; complete reports follow admin KPI order and list missing KPIs when partial generation is enabled.


## Previous-Date KPI Management
- Admins can create KPI records for any valid previous date from Billing > Create KPI Record.
- Existing records can be edited to correct the record date, billing month, or billing year.
- Duplicate warnings are evaluated against the selected record date, not only today.
- Record changes, evidence additions, creations, and deletions are recorded in `audit_logs`.
- Non-admin users remain limited to creating records for the current date.

## Report Design
- Individual and daily PDF/Word reports use the Modern Premium visual style.
- Picture evidence remains 2x2 per page; Bulk Waste remains clearly separated into Before and After.
- Original PDF evidence remains authoritative and is never rasterized into fake picture evidence.

### Report export dependencies
- Word `.docx` exports use PHPWord Word2007 and are validated before download when the PHP ZIP extension is available.
- Legacy Word `.doc` exports use PHPWord's RTF writer in a Word-compatible `.doc` container.
- Complete daily PDF reports merge the generated KPI report with uploaded PDF evidence pages using FPDI/FPDF.
- After updating this version, run `composer update setasign/fpdf setasign/fpdi` from the project folder.
- For the complete PDF merge, FPDI is preferred. Ghostscript is supported as an optional fallback if it is installed and available in PATH.


## Commercial Licensing

This edition is prepared for commercial distribution as a licensed KPI Billing product.

- Each customer installation has a unique Installation ID.
- Licenses are cryptographically signed and verified with the public key shipped in `license/public.pem`.
- The private signing key is NOT part of the customer application.
- Users are unlimited; there is no `max_users` license restriction.
- A license may be bound to one or more hosts. This supports localhost and online deployments.
- An unlicensed/expired installation can still sign in to reach the License page, but billing/evidence/report pages remain protected.
- Company/vendor information is configurable from Settings and is used in reports.
- `sql/install.sql` is a clean schema with no sample KPIs, billing records or M&B operational data.

### Fresh customer installation

1. Copy the application folder to XAMPP or upload it to the online server.
2. Start/open `install.php`.
3. Enter the MySQL host, database name, user and password.
4. Enter the administrator and company/vendor details.
5. Install. The database is created with empty KPI/billing/evidence tables.
6. Sign in.
7. Copy the displayed Installation ID from License.
8. The software owner creates a signed license using the separate Owner License Generator.
9. Paste the license key into License → Activate.
10. Configure KPIs and report signature/stamp.

### Owner security

The separate Owner License Generator and its private RSA key are for the software owner only. Never upload or distribute them with the customer package.


## Incremental enhancement package (2026-10-10)
See `UPGRADE_NOTES_2026-10-10.md` before replacing an existing installation. Back up the database and evidence folders, then sign in as Admin and run `admin/migrate-enhancements.php` once before using the new modules. The migration preserves existing rows and creates recycle-bin/import-history metadata. New Admin navigation includes User Management, Recycle Bin, Bulk KPI Import, Folder Templates, and Audit Log.

The bulk importer supports date-wise folder staging in small requests and ZIP imports, per-date preview, existing KPI/date-rule validation, image/PDF previews, private pre-import manifests/backups, import history, CSV/XLSX history export, and selective undo of files added by an import. Duplicate categories can be skipped or updated by appending evidence without overwriting existing files. Per-file replacement comparison is not part of this increment.
#   K P I - B i l l i n g - S y s t e m  
 
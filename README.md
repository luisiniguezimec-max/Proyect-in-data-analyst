# Proyect-in-data-analyst
This project provides a repeatable pipeline to transform raw manufacturing/test row-data (CSV or Excel) into:

A human-friendly Excel report designed for engineering review, organized by test section (e.g., Seccion_1, Seccion_2, etc.) and ready to be filled with findings.

A full-fidelity CSV export containing all original columns (filtered to the selected error condition), suitable for:

BI / dashboards

SQL ingestion

automated monitoring

historical analysis

Hotspot analytics:

incidence count per FLAT_ID

incidence percentage per FLAT_ID

automatic blocklist generation for stations/testers that exceed a defined threshold (default: > 3 occurrences)

The script is built to be configurable and reusable: the error code filter is not hardcoded; it can be changed via command-line arguments to generate reports for different failure signatures.

Key Concepts
FLAT_ID as a “tester/station” identifier

The input dataset contains a column such as:

FLAT_ID (example formats: 1B-27, 2H-03, etc.)

FLAT_ID is treated as a station/tester identifier. Repeated occurrences of the same FLAT_ID for the selected error condition usually indicate a hot tester that should be reviewed with higher urgency.

Error filtering modes

The report can filter records by error code using different match strategies:

endswith: rows where ERROR_CODE ends with the requested value

equals: exact match

contains: substring match

regex: regular expression match (advanced use)

This allows the program to adapt to different data conventions (e.g., long error strings where the final digits represent the actual error code).

Inputs
Supported input formats

.csv

.xlsx, .xlsm, .xls

Required columns (auto-detected)

FLAT_ID (or variants like FLAT ID, FLAT-ID)

ERROR_CODE (or variants like ERROR CODE, ERROR-CODE)

If your dataset uses different names, you can override the column selection via CLI flags.

Outputs

The program generates multiple deliverables from a single run:

1) Excel report (XLSX)

The XLSX is designed for engineers to review and document findings.

Main sheets:

Resumen_<TAG>: Summary count of filtered failures per section

Detalle_<TAG>: Minimal view listing section, FLAT_ID, ERROR_CODE

Incidencias_FLAT_ID: Hotspot ranking (count, percent, priority)

Section sheets:

Seccion_1, Seccion_2, ...: engineering review sheets with only:

FLAT_ID

ERROR_CODE

TOM_B-STATUS

TOM_AL-STATUS

TOM_AR-STATUS

RAISER-STATUS

These are intentionally minimal to reduce noise and avoid confusion during troubleshooting.

Deduplication inside each section:
If the same FLAT_ID appears multiple times within a section, the program:

keeps only one row (removes duplicates)

highlights the remaining FLAT_ID in red to indicate it had repeated appearances

Highlighting rules:
The FLAT_ID cell is highlighted in red if:

the tester is in the blocklist (incidences > threshold), OR

the tester appeared duplicated within that section

2) Full CSV export (filtered but complete)

A CSV export is created that:

preserves all original columns

contains only rows matching the selected error condition

maintains consistent ordering by section/row/position for easier downstream processing

This CSV is ideal for:

feeding dashboards

importing into SQL (SQLite, MySQL, Postgres, etc.)

tracking trends over time

3) Hotspot analytics CSVs

Two additional CSV files are generated:

*_incidencias_flat_id.csv

A ranked list of testers/stations, including:

FLAT_ID

INCIDENCIAS (count)

PORCENTAJE (share of total filtered incidents)

PRIORIDAD (NORMAL, VIGILAR, URGENTE, BLOQUEAR)

*_blocklist_flat_id.csv

A list of testers/stations exceeding the incidence threshold, intended for:

proactive maintenance

escalation

station blocking policies

Sorting and sectioning logic

The program parses FLAT_ID values like 1B-27 into:

SECTION = 1

ROW_LETTER = B

POSITION = 27

Records are then sorted using:

SECTION ascending

custom row letter order (default: H, A, B, C, D, E, F, G)

numeric position ascending

This produces a consistent ordering aligned with how stations/testers are physically organized.

Blocklist policy

The blocklist threshold is configurable, with a default policy:

Block if incidences > 3

This is meant to support operational policies such as:

“If a tester generates more than 3 incidents of the same signature, escalate and/or block it.”

Typical use cases

Generate per-turn or daily engineering reports

Prioritize station inspections based on hotspot ranking

Automate tester blocking decisions based on failure frequency

Export clean datasets for BI dashboards and trend analysis

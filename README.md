# Concurrency Count

**Version:** 1.0.0  
**Released:** 1st July 2025  
**Author:** [20tele.com](https://20tele.com)  
**Licence:** GNU General Public License v3.0  
**Compatibility:** Asterisk 18, 20, and 22 (PJSIP only). Tested on FreePBX 16/17 and PBXact equivalents.

---

## Overview

**Concurrency Count** is a standalone Bash script that calculates the **maximum number of concurrent PJSIP calls** on a FreePBX or PBXact system over a specified time period.

It is designed for manual execution via SSH and supports diagnostics, capacity planning, and usage auditing. No integration with the FreePBX web interface is required.

### Supported Modes

- **Trunks mode** – Calculates concurrent calls per named PJSIP trunk  
- **Extensions mode** – Calculates concurrent calls per numeric extension  
- **Group mode** – Calculates overall system concurrency (all numeric call legs counted)

All results are based on second-by-second analysis of answered calls (`disposition = 'ANSWERED'`) from the Asterisk CDR database.

---

## Features

- Compatible with Asterisk 18, 20, and 22 (PJSIP only)
- Works on FreePBX 16/17 and PBXact equivalents
- Interactive and unattended (flag-based) operation
- Accepts flexible input formats:
  - `today`, `yesterday`
  - Named months (e.g. `June`)
  - Shorthand dates (e.g. `24`, `24-06`, `24-6-1`)
  - Custom full datetimes (e.g. `2024-05-10 14:30:00`)
- Rejects invalid or future dates (with 3 retry attempts)
- Automatically clamps call legs to 24 hours
- Identifies and warns about trunks with numeric names
- Runtime estimation with live progress output
- 60-minute runtime limit with optional override
- Debug mode for detailed visibility

---

## Requirements

- FreePBX or PBXact system using PJSIP (not `chan_sip`)
- MySQL table `asteriskcdrdb.cdr` must exist (standard FreePBX layout)
- Root SSH access
- MySQL credentials stored in `/root/.my.cnf`:

  ```ini
  [client]
  user=freepbxuser
  password=freepbxuser_password
  host=localhost
  ```

- Required system commands:
  - `bash`, `mysql`, `awk`, `grep`, `sed`, `date`, `sort`, `printf`, `cut`, `wc`

---

## Installation

Run the following one-liner via SSH:

```bash
wget https://raw.githubusercontent.com/20telecom/IN1CLICK/main/concurrency-count -O /tmp/IN1CLICK && chmod +x /tmp/IN1CLICK && /tmp/IN1CLICK
```

This installs the script to `/usr/local/bin/concurrency-count` and makes it executable.

Verify installation:

```bash
concurrency-count --status
```

---

## Usage

### Interactive Mode

```bash
concurrency-count
```

You will be prompted to:

1. Choose a mode: trunks, extensions, or group  
2. Enter a date range (month name, shorthand, or custom range)

### Flag-Based Mode

```bash
concurrency-count --extensions --debug
```

#### Available Flags

| Flag           | Description                            |
|----------------|----------------------------------------|
| `--trunks`     | Run in trunk mode                      |
| `--extensions` | Run in extension mode                  |
| `--group`      | Run in group (overall) mode            |
| `--debug`      | Enable verbose runtime debug output    |
| `--status`     | Display version and debug status       |
| `--help`       | Display usage instructions             |

---

## Output Examples

### Trunks or Extensions Mode

```
Maximum concurrent calls per PJSIP trunk between 2025-06-01 and 2025-06-30:

 INTRUNK1-DEMO              7
 OUTRUNK2-DEMO              5
```

### Group Mode

```
Peaks occurred at the following time ranges:

  2025-06-12 10:40:55 to 2025-06-12 10:41:17
  2025-06-17 11:27:21 to 2025-06-17 11:28:02

Maximum concurrent calls overall: 14
```

---

## Input Format Reference

| Input Example         | Interpreted As                               |
|------------------------|-----------------------------------------------|
| `today`                | From 00:00 to current time today              |
| `yesterday`            | Full previous day                             |
| `April` + `2024`       | 1st to 30th April 2024                        |
| `23`                   | Full year 2023                                |
| `24-06`                | June 2024                                     |
| `24-6-1`               | 1st June 2024                                 |
| `2024-05-10 14:30:00`  | Specific date and time                        |

The script allows up to three invalid input attempts before exiting.

---

## How It Works

1. Queries the `cdr` table for answered PJSIP calls in the selected time range.
2. Calculates each call leg's start and end time.
3. Tracks every second the call was active.
4. Records peak concurrency per extension, trunk, or in total.
5. Clamps any call leg over 24 hours to prevent inflated figures.

---

## Runtime Management

- Enforces a 60-minute maximum runtime
- If estimated time exceeds this, a warning is shown
- You may choose to continue or abort
- If no response is given, the script exits automatically

This protects the system and database under heavy load.

---

## Debug Mode

To enable debug output:

```bash
concurrency-count --debug
```

Debug mode displays:

- SQL query details
- Identified trunks or extensions
- Row-by-row progress
- Estimated completion time
- Runtime diagnostics

---

## Data Model & Assumptions

| Item                          | Detail                                                                 |
|-------------------------------|-------------------------------------------------------------------------|
| Call filtering                | Only `ANSWERED` calls are included                                     |
| Protocol                      | PJSIP only (`chan_sip` not supported)                                  |
| Extension detection           | Based on numeric `PJSIP/123-xxxxx` format                              |
| Trunk detection               | Names must contain at least one letter                                 |
| Long call handling            | Legs over 24 hours are clamped                                         |
| Call duplication              | Transfers/forks are counted individually                               |
| Billing, DID, direction       | Not evaluated                                                          |

---

## Known Limitations

- Only works with PJSIP (not compatible with `chan_sip`)
- Trunks named numerically (e.g. `24700020`) may be misidentified as extensions
- Requires root access (Asterisk CLI and MySQL credentials)
- Not intended for background automation (cron, GUI tools, etc.)

---

## Output Behaviour

- Results printed to standard output
- No data is saved or logged
- Designed for manual execution via SSH
- Not suitable for unattended environments

---

## Best Practice

Recommended uses:

- Audit SIP trunk/channel usage
- Investigate call path saturation
- Review historic concurrency during peak hours
- Assist with FreePBX/PBXact tuning or migrations

Avoid use in automated pipelines or system schedulers.

---

## Troubleshooting

| Problem                  | Solution                                                                  |
|--------------------------|---------------------------------------------------------------------------|
| Low or missing results   | Check for PJSIP usage, numeric trunk names, and populated CDR records     |
| Script exits prematurely | Runtime limit reached; try reducing the date range                        |
| No output                | Check MySQL access, CDR table existence, and `.my.cnf` configuration       |
| Group mode is slow       | Use a narrower date range or enable debug to monitor progress             |

---

## License

Licensed under the **GNU General Public License v3.0**.  
You are free to use, modify, and redistribute under the terms of the GPL.

Full licence text: [https://www.gnu.org/licenses/gpl-3.0.en.html](https://www.gnu.org/licenses/gpl-3.0.en.html)

---

## Contact

For support, queries, or feedback:

**Email:** [support@20tele.com](mailto:support@20tele.com)  
**Website:** [https://20tele.com](https://20tele.com)

---

## About

**Concurrency Count** was developed by [20tele.com](https://20tele.com) to provide visibility into real-time SIP channel usage on FreePBX and PBXact systems.

It is safe for production use, requires no external dependencies, and supports full user control. Contributions and forks are welcome.


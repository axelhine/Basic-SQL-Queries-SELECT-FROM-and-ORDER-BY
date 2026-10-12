# Basic SQL Queries: SELECT, FROM, and ORDER BY

> **Disclaimer:** This project was completed as part of the Google Cybersecurity Certificate (Coursera). The scenario and database come from the course. The queries and explanations are my own documentation of the work I performed.

## Project description
As a security analyst, I needed to find which employee devices required updates and check user login activity for anything unusual. Using SQL in a MariaDB shell, I queried the `machines` and `log_in_attempts` tables in the organization database to retrieve specific columns, scan for suspicious login locations and times, and sort results into a readable order. Pulling the right data from a database is a core analyst task for improving security and keeping data safe.

## Database Tables
**`machines`** (hardware information)

| Column | Description |
|---|---|
| `device_id` | Unique identifier for the device |
| `operating_system` | Operating system on the device |
| `email_client` | Email client on the device |
| `OS_patch_date` | Date the OS was last patched |
| `employee_id` | Employee who uses the device |

**`log_in_attempts`** (user activity)

| Column | Description |
|---|---|
| `event_id` | Unique identifier for the login event |
| `username` | Account that attempted to log in |
| `login_date` | Date of the attempt |
| `login_time` | Time of the attempt |
| `country` | Country the attempt came from |
| `ip_address` | IP address of the attempt |
| `success` | Whether the attempt succeeded |

## SQL Keywords Used
| Keyword | Purpose |
|---|---|
| `SELECT` | Choose which columns to return |
| `FROM` | Choose which table the data comes from |
| `*` | Return all columns in a table |
| `ORDER BY` | Sort the results by one or more columns |
| `;` | Ends the statement and runs the query |

A query reads like a sentence to the database: `SELECT` asks which columns I want to see, and `FROM` asks which table they are stored in. When selecting multiple columns, a comma separates each one, with no comma after the last column before `FROM`.

## Retrieve employee device data
Devices with old patch dates are the ones most likely to be vulnerable, so these queries help identify which machines need updates.

**All device information.** The asterisk (`*`) returns every column in the table:
```sql
SELECT *
FROM machines;
```
Output (first rows):
```
+--------------+------------------+----------------+---------------+-------------+
| device_id    | operating_system | email_client   | OS_patch_date | employee_id |
+--------------+------------------+----------------+---------------+-------------+
| a184b775c707 | OS 1             | Email Client 1 | 2021-09-01    |        1156 |
| a192b174c940 | OS 2             | Email Client 1 | 2021-06-01    |        1052 |
| a305b818c708 | OS 3             | Email Client 2 | 2021-06-01    |        1182 |
| a317b635c465 | OS 1             | Email Client 2 | 2021-03-01    |        1130 |
| a320b137c219 | OS 2             | Email Client 2 | 2021-03-01    |        1000 |
...
200 rows in set
```

**Email client on each device.** Selecting only the columns I need:
```sql
SELECT device_id, email_client
FROM machines;
```
Output (first rows):
```
+--------------+----------------+
| device_id    | email_client   |
+--------------+----------------+
| a184b775c707 | Email Client 1 |
| a192b174c940 | Email Client 1 |
| a305b818c708 | Email Client 2 |
| a317b635c465 | Email Client 2 |
| a320b137c219 | Email Client 2 |
...
```

**Operating system and last patch date.** This shows which devices are overdue for updates:
```sql
SELECT device_id, operating_system, OS_patch_date
FROM machines;
```
Output (first rows):
```
+--------------+------------------+---------------+
| device_id    | operating_system | OS_patch_date |
+--------------+------------------+---------------+
| a184b775c707 | OS 1             | 2021-09-01    |
| a192b174c940 | OS 2             | 2021-06-01    |
| a305b818c708 | OS 3             | 2021-06-01    |
| a317b635c465 | OS 1             | 2021-03-01    |
| a320b137c219 | OS 2             | 2021-03-01    |
...
```

## Investigate login activity
Here I scanned the `log_in_attempts` table for unusual activity.

**Login locations.** The organization operates in the United States, Canada, and Mexico, so a login from anywhere else is a red flag for possible credential theft:
```sql
SELECT event_id, country
FROM log_in_attempts;
```

**Login dates and times.** Attackers often use stolen credentials outside working hours, assuming no one is watching. A successful login at an unusual hour could mean a compromised account:
```sql
SELECT username, login_date, login_time
FROM log_in_attempts;
```

**Complete picture of all login attempts.** Selecting every column gives the full record, including IP address and whether each attempt succeeded:
```sql
SELECT *
FROM log_in_attempts;
```

Scanning rows by hand works for a small table, but it doesn't scale. With millions of entries, I would use a `WHERE` clause to filter for specific conditions, such as logins from outside the expected countries.

## Order login attempts data
`ORDER BY` turns an unsorted pile of records into an organized list.

**Sort by date:**
```sql
SELECT *
FROM log_in_attempts
ORDER BY login_date;
```

**Sort by date, then time.** SQL sorts by the first column listed, then uses the next column to order any rows that share the same value:
```sql
SELECT *
FROM log_in_attempts
ORDER BY login_date, login_time;
```
This produces a chronological timeline of login activity, which makes it easier to spot patterns or sequences of suspicious attempts.

## Findings
- **Email clients:** The third device in the `machines` table uses Email Client 2.
- **Patch dates:** The first device listed (`a184b775c707`) was last patched on 2021-09-01. Reviewing `OS_patch_date` across all 200 devices is how I would identify the machines most overdue for updates.
- **Login locations:** No login attempts came from Australia, so nothing in the country column raised a red flag for credential theft from outside North America.
- **Login times:** The fifth record returned in the username, date, and time query belongs to `jrafael`.
- **Sorted by date:** The earliest record in the table is `ivelasco` on 2022-05-08.
- **Sorted by date and time:** The first record is `bsand` at 00:19:11 on that earliest date, a login shortly after midnight. Early-morning attempts are worth checking against the `success` column, since attackers often use stolen credentials outside working hours.

## Full Query Sequence
```sql
SELECT * FROM machines;
SELECT device_id, email_client FROM machines;
SELECT device_id, operating_system, OS_patch_date FROM machines;
SELECT event_id, country FROM log_in_attempts;
SELECT username, login_date, login_time FROM log_in_attempts;
SELECT * FROM log_in_attempts;
SELECT * FROM log_in_attempts ORDER BY login_date;
SELECT * FROM log_in_attempts ORDER BY login_date, login_time;
```

## Summary
I used `SELECT` and `FROM` to pull device data and identify machines by operating system and patch date, then queried login attempts to look for unexpected countries and unusual login times. I used `ORDER BY` to sort the login data by date and time into a chronological view, which showed the earliest activity began at 00:19:11 on 2022-05-08 and confirmed no attempts came from Australia. These queries are the foundation for security investigations: finding outdated systems that need patching and spotting suspicious access that could indicate a compromised account.

## Key Takeaways
- `SELECT` chooses the columns and `FROM` chooses the table. Every query needs both.
- Select only the columns you need. It keeps results focused and easier to read than `SELECT *`.
- Old `OS_patch_date` values point to vulnerable machines that attackers could exploit.
- Unexpected login countries and off-hours logins are red flags for credential theft.
- `ORDER BY login_date, login_time` sorts by date first, then by time within each date.
- Table names are case-sensitive, and every query ends with a semicolon. Without one, the shell keeps waiting for more input.
- Manual scanning doesn't scale, which is why filtering with `WHERE` is the next step.

## Tools
- MariaDB shell
- SQL

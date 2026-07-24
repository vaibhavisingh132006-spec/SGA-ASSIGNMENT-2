# Question 1: Duplicate Submission Detection & Backup

## Design Overview

The `submission_manager.sh` script scans a directory containing student submissions and uses **MD5 checksums** to identify duplicate files based on their content rather than their filenames. Only unique submissions are copied to a backup directory, while a report summarizing the process is generated. Any errors encountered during execution are recorded separately in an error log.

## Commands Executed and Their Purpose

### `mkdir submissions`

Created a directory named **submissions** to store sample student assignment files for testing.

### Creating Sample Files

Five sample `.txt` files were placed inside the `submissions/` directory. Among these, two files intentionally contained the same content as other files but had different filenames. This allowed the script to verify that duplicate detection works using file content instead of filenames.

### `ln -s /nonexistent/missing_original.txt submissions/student_F_asg1.txt`

Created a dangling symbolic link that points to a non-existent file. This simulates a corrupted or incomplete student submission and tests whether the script correctly handles unreadable files by logging the error instead of terminating unexpectedly.

### `chmod +x submission_manager.sh`

Granted execute permission to the script so it could be run directly from the terminal.

### `./submission_manager.sh`

Executed the script. It calculated the MD5 hash for every readable submission, checked whether that hash had already been encountered, copied only unique files into the `backup_unique/` directory, and logged any unreadable files into `errors.log`.

### `cat report.txt`

Displayed the generated processing report. The report showed that **6 entries** were processed (5 valid submissions and 1 broken symlink), **2 duplicate files** were detected, and **3 unique files** were successfully backed up.

### `cat errors.log`

Displayed the contents of the error log. It contained an entry similar to:

`Error: cannot read submission (missing/broken/unreadable): ./submissions/student_F_asg1.txt`

This confirms that unreadable submissions are detected and logged separately instead of being ignored.

### `ls backup_unique/`

Verified that only the three unique readable files (`student_A_asg1.txt`, `student_C_asg1.txt`, and `student_E_asg1.txt`) were copied into the backup directory. Duplicate files and the broken submission were correctly excluded.

---

## Script Improvement

The initial implementation used the condition `[ -f "$file" ]` before processing files. Since `-f` follows symbolic links, broken symlinks failed this test and were skipped entirely, meaning corrupted submissions never reached the error-handling section.

To solve this issue, the condition was changed to:

`[ -e "$file" ] || [ -L "$file" ]`

This ensures that even broken symbolic links are considered during processing. The script then performs an explicit readability check using:

`[ ! -r "$file" ]`

Unreadable files are written to `errors.log` and skipped safely, ensuring that every submission is accounted for and that invalid files are properly reported.

---

## Justification of Commands, Redirection Operators, and Techniques

### File Existence and Readability Checks

Using `[ -e "$file" ] || [ -L "$file" ]` together with `[ ! -r "$file" ]` ensures that every item inside the submissions directory is examined. Broken symbolic links are detected, logged, and excluded from further processing instead of being silently ignored.

### `md5sum`

Generates a unique checksum based on file contents. This makes it possible to detect duplicate submissions even when students rename their files.

### Associative Array (`declare -A`)

Stores each checksum along with the first file that produced it. This provides fast duplicate detection with constant-time lookups instead of repeatedly comparing every file against every other file.

### `cp` Instead of `mv`

Copies unique submissions to the backup directory while leaving the original files unchanged. This preserves the original submissions, which is useful for record keeping and auditing.

### Output Redirection

* `>` creates or clears `report.txt` and `errors.log` at the beginning of each execution, ensuring that previous results are removed.
* `>>` appends new information to an existing file without overwriting earlier entries during the same run.
* `2>>` redirects only error messages (standard error) into `errors.log`, keeping normal program output separate from error information.

### `tee -a`

Displays the startup message on the terminal while simultaneously appending it to `report.txt`, eliminating the need to execute separate commands for display and logging.

---

## Screenshots

* **screenshots/01_setup_submissions.png** – Creating the `submissions` directory, adding sample files, and making the script executable.
* **screenshots/02_run_script.png** – Running `submission_manager.sh`.
* **screenshots/03_report_and_errors.png** – Viewing `report.txt` and `errors.log`.
* **screenshots/04_backup_verification.png** – Confirming that only unique submissions were copied to the backup directory.

---

## Files Included

* `submission_manager.sh` – Main Bash script.
* `submissions/` – Sample student submissions, including duplicate files and one broken symbolic link.
* `backup_unique/` – Directory containing the backed-up unique submissions.
* `report.txt` – Summary report generated after execution.
* `errors.log` – Log file containing any processing errors.
* `screenshots/` – Screenshots showing the execution steps and outputs.

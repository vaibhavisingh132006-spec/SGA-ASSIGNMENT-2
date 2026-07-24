# Question 5: **vi Recovery Mechanisms After a Crash**

## Scenario

A developer is editing an important configuration file using **vi/vim** when the system unexpectedly crashes before the changes are saved. This report examines the available recovery features in **vi/vim**, compares their effectiveness, and identifies the most dependable method for restoring unsaved work.

---

## Comparison of Recovery Mechanisms

| Recovery Mechanism                    | Description                                                                                                                                                                                                              | Reliability After a Crash                                                                                            |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------- |
| **Swap File (`.filename.swp`)**       | While editing, vi/vim periodically stores the current editing buffer in a hidden swap file. After a crash, reopening the file with `vi -r filename` (or simply `vi filename`) detects the swap file and offers recovery. | **Most reliable**, as it preserves recent unsaved edits that would otherwise be lost.                                |
| **Persistent Undo (`.filename.un~`)** | When `set undofile` is enabled, Vim stores the undo history on disk, allowing undo operations even after reopening the file.                                                                                             | Helpful for restoring previous edits across sessions, but only if persistent undo was enabled before the crash.      |
| **Registers**                         | Registers temporarily store copied, deleted, and yanked text during an editing session.                                                                                                                                  | Not suitable for crash recovery because register contents are normally lost when the editor terminates unexpectedly. |
| **Backup File (`filename~`)**         | If backup mode is enabled, Vim creates a backup of the original file before writing changes.                                                                                                                             | Protects against accidental overwrites but cannot recover edits that were never saved.                               |
| **Auto Recovery (`vi -r`)**           | Searches for an existing swap file and provides an option to recover the unsaved editing session.                                                                                                                        | Uses the swap file internally and is the standard recovery method after a crash.                                     |

---

## Commands Used for Demonstration

```bash
vi criticalconfig.conf
# Make changes without saving, then simulate a crash
# (for example: kill -9 <vi_pid> or close the terminal unexpectedly)

vi -r criticalconfig.conf

:recover

diff criticalconfig.conf.swp criticalconfig.conf
```

### Explanation

* The file is opened in **vi**, modified, and then an unexpected crash is simulated before executing `:w`.
* Running `vi -r criticalconfig.conf` causes Vim to detect the existing swap file and prompt for recovery.
* The `:recover` command can also be used from within Vim to restore the saved editing session.
* Finally, the recovered content can be compared with the previously saved file using `diff` to verify which changes were restored.

---

## Evaluation

Among the available recovery methods, the **swap file** is the most effective because it is specifically designed to preserve unsaved edits made during an editing session.

Other mechanisms have important limitations:

* **Registers** exist only in memory and disappear if the editor crashes.
* **Backup files** preserve only the last saved version of a file and cannot restore changes that were never written to disk.
* **Persistent undo** is useful for revisiting previous editing history but requires `set undofile` to have been enabled before the crash occurred.

Therefore, these mechanisms cannot replace the protection offered by swap files for recovering unsaved work.

---

## Recommended Recovery Strategy

The recommended recovery method is to use the **swap file** together with Vim's built-in recovery feature.

Suggested recovery procedure:

1. Reopen the file using `vi -r filename` (or simply `vi filename`, which automatically detects the swap file).
2. Recover the unsaved contents when prompted.
3. Compare the recovered file with the previously saved version using `diff` if verification is required.
4. Save the recovered file using `:w`.
5. Delete the old `.swp` file after successful recovery to prevent future swap-file warnings.

For improved long-term protection, it is also advisable to:

* Save work frequently using `:w`.
* Enable persistent undo by adding `set undofile` to the `.vimrc` configuration file.

Combining these practices provides protection against both unexpected crashes and accidental editing mistakes.

---

## Screenshots

* **screenshots/01_editing_before_crash.png** – Editing `criticalconfig.conf` in insert mode before the simulated crash.
* **screenshots/02_swap_file_recovery_prompt.png** – Recovery prompt displayed after reopening the file with `vi -r`, showing detection of the swap file.
* **screenshots/03_recovery_completed_and_cleanup.png** – Successfully recovered file, saving changes, and removing the stale `.swp` file.

---

## Files Included

* `explanation.md` – Documentation describing the recovery mechanisms and recommended workflow.
* `screenshots/` – Screenshots illustrating the editing, crash simulation, recovery process, and cleanup steps.

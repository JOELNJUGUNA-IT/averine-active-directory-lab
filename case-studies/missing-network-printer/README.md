# Missing HR Network Printer

**Type:** Simulated help-desk ticket

**Symptom:** The HR printer was missing from all apps on one Windows laptop, but a coworker could still print.

**Diagnosis:** The printer was absent from the user's Printers & scanners list but was available on the simulated print server at `\\PRINT01\HR-Printer`. The printer connection was missing from the user's Windows setup. Why it disappeared was not established.

**Proposed fix:** Reconnect the approved shared printer from `\\PRINT01`.

**Validation:** Confirm the printer reappears in Windows, then send a test page and verify it comes out of the correct printer. No physical print was performed in this simulation.

# Parkside Mower Data Point (DP) Summary

This document provides a concise overview of the key Tuya Data Points (DPs) used by the Parkside robot lawnmower and their roles in Home Assistant via LocalTuya.

| DP ID             | Code                | Type              | Purpose / Entity                                                                                                       |
| ----------------- | ------------------- | ----------------- | ---------------------------------------------------------------------------------------------------------------------- |
| 2                 | switch\_go          | bool              | *Switch* for Start/Stop (writes via DP 115)                                                                            |
| 3                 | mode                | enum              | *Select* for operational mode (`auto`, `manual`, `standby`)                                                            |
| 5                 | status              | enum              | *Sensor* for current mower status                                                                                      |
| 13                | battery\_percentage | value             | *Sensor* for battery level (%)                                                                                         |
| 101               | MachineStatus       | enum              | *Sensor* for detailed machine state                                                                                    |
| 102               | MachineError        | bitmap            | *Sensor* for error codes                                                                                               |
| 103               | MachineWarning      | enum              | *Sensor* for warning alerts (e.g., tilt, bump)                                                                         |
| 104               | MachineRainMode     | bool              | *Switch* for rain sensor override                                                                                      |
| 105               | MachineWorktime     | value             | *Sensor* for accumulated work time (hours)                                                                             |
| 106               | MachinePassword     | value             | *Number* for setting device PIN (4‑digit password)                                                                     |
| 115               | MachineControlCmd   | enum              | *Select* for drive commands:<br>PauseWork, CancelWork, ContinueWork, StartMowing, StartFixedMowing, StartReturnStation |
| 116               | MachineCover        | bool              | *Binary Sensor* for cover/dock state (open = mowing, closed = dock)                                                    |
| 110–113, 134, 139 | Reserved / Logs     | raw/string/bitmap | *Diagnostics*: appointments, error/work logs, firmware version                                                         |

---

**Usage Notes:**

* **DP 2** (`switch_go`) must be written through **DP 115** (`MachineControlCmd`) using `write_function: 115` in LocalTuya.
* **Enum** DPs require explicit `options:` definitions in YAML or UI.
* **Reserved** DPs (110–113, 139) can be added for advanced diagnostics but are optional.

Refer to the main `README.md` for full setup instructions and example automations.

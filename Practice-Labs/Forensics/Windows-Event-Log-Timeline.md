# 🧪 Practice Lab — Windows Event Log Timeline Analysis

> **Classification:** Self-created CTF-style practice scenario
>
> This is a synthetic lab designed for practice. It is **not** an official CTF challenge and does not represent a real incident, real flag, CVE, ranking, or personal achievement.

## Challenge / Context

A Windows workstation is suspected of running an unauthorized PowerShell command. You are given a small set of exported Windows event records and must reconstruct the activity timeline.

The objective is to determine:

1. Which account initiated the suspicious activity.
2. Which process started PowerShell.
3. What PowerShell command was executed.
4. How the events should be correlated to avoid relying on a single log entry.

The dataset is intentionally synthetic and contains no credentials or sensitive information.

## Reconnaissance / Analysis

Start by identifying the useful event sources and timestamps rather than immediately searching for a suspicious string.

Relevant Windows telemetry in this practice scenario:

| Event ID | Source | Purpose |
|---|---|---|
| 4624 | Security | Successful logon context |
| 4688 | Security | Process creation |
| 4104 | PowerShell | Script-block content |

Example filtering with PowerShell:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624,4688} |
    Select-Object TimeCreated, Id, Message
```

For PowerShell operational telemetry:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-PowerShell/Operational'; Id=4104} |
    Select-Object TimeCreated, Id, Message
```

## Timeline Construction

The key is to correlate the events by **time, account, host, and process relationship**.

A useful investigation table is:

| Time | Event | Account | Observation |
|---|---:|---|---|
| T1 | 4624 | `lab-user` | Interactive logon |
| T2 | 4688 | `lab-user` | `powershell.exe` created |
| T3 | 4104 | `lab-user` | Script block recorded |

The timestamps above are labels for the exercise, not a real incident timeline.

## Technique

A single PowerShell 4104 event is not sufficient by itself to establish the complete execution chain. Correlating it with process creation and authentication telemetry provides stronger evidence.

The investigation flow is:

```text
4624 successful logon
        ↓
4688 powershell.exe process creation
        ↓
4104 PowerShell script-block telemetry
        ↓
Correlate account + time + process
        ↓
Reconstruct activity
```

## Solution Steps

### 1. Locate the authentication event

Filter Event ID `4624` and identify the account associated with the relevant workstation activity.

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624} |
    Where-Object {$_.Message -match 'lab-user'}
```

### 2. Find the process creation event

Search Event ID `4688` for `powershell.exe`.

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4688} |
    Where-Object {$_.Message -match 'powershell.exe'}
```

Review the timestamp and account rather than treating the executable name alone as malicious.

### 3. Inspect PowerShell script-block logging

Event ID `4104` can contain the script block associated with PowerShell execution.

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-PowerShell/Operational'; Id=4104} |
    Select-Object TimeCreated, Message
```

Compare its timestamp with the `4688` event and verify that the same user and host context are involved.

### 4. Reconstruct the sequence

The expected lab reasoning is:

```text
User logon
  → powershell.exe process creation
  → PowerShell script block
```

This provides a defensible timeline instead of concluding compromise from an isolated PowerShell event.

## Optional Parsing Helper

For exported event text, a small Python helper can extract event IDs and timestamps for manual review:

```python
import re
from pathlib import Path

text = Path("events.txt").read_text(errors="ignore")

for match in re.finditer(r"Event ID:\s*(\d+).*?TimeCreated:\s*([^\n]+)", text, re.S):
    event_id, timestamp = match.groups()
    print(f"{timestamp.strip()} | Event {event_id}")
```

This helper is intentionally simple. In a real investigation, preserve the original evidence and validate parser assumptions before relying on automated extraction.

## Result

**Expected practice result:** the analyst should be able to reconstruct the synthetic execution chain by correlating the `4624`, `4688`, and `4104` records.

No real flag or real-world incident outcome is claimed by this lab.

## Lessons Learned

- Event correlation is more reliable than treating one event as conclusive evidence.
- `4624` provides authentication context, while `4688` provides process-creation context and `4104` provides PowerShell script-block visibility.
- Timestamps, account identity, host identity, and process relationships should be considered together.
- PowerShell telemetry can be valuable for SOC investigations, but logging availability and configuration affect visibility.
- Automated parsing should supplement, not replace, evidence validation.

## Defensive Notes

For a real Windows environment, defenders should evaluate appropriate auditing and PowerShell logging coverage, centralize relevant telemetry, establish normal administrative baselines, and correlate suspicious process activity with authentication and endpoint context.

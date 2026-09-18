# picoCTF — Log Hunt

**Category:** General Skills  
**Difficulty:** Easy  
**Technique:** Log analysis / fragment reconstruction  
**Author:** Tony_29

## Challenge / Context

The challenge provides a server log containing scattered pieces of a secret flag. Some fragments may be repeated, so the objective is to identify the relevant log entries and reconstruct the original flag.

The original solve notes document a manual log-analysis workflow: download the log, search for entries marked `INFO FLAGPART`, collect the fragments, reconstruct the flag, and submit the result.

## Reconnaissance / Analysis

I started by obtaining the supplied log file and opening it for inspection.

Rather than reading the entire log line by line, I looked for an identifying pattern associated with the flag fragments. The useful entries were marked:

```text
INFO FLAGPART
```

This provided a simple filter for separating relevant records from normal server activity.

## Technique

The challenge demonstrates **log analysis and data reconstruction**.

The key approach was:

1. Identify the marker associated with flag fragments.
2. Search the log for every occurrence of that marker.
3. Collect the corresponding fragments.
4. Account for repeated entries rather than treating every occurrence as a unique fragment.
5. Reconstruct the original flag in the required order.

## Solution

### 1. Obtain the log file

I downloaded the log file provided by the challenge and opened it locally.

### 2. Search for flag-related entries

Using the editor's search functionality, I searched for:

```text
INFO FLAGPART
```

The search returned the log entries containing the scattered flag fragments.

### 3. Collect the fragments

I recorded the relevant fragments and compared repeated entries so that duplicate log records did not result in duplicated portions of the flag.

### 4. Reconstruct the flag

The collected fragments were combined in their intended sequence to reconstruct the complete flag.

## Result

The reconstructed flag was:

```text
picoCTF{us3_y0urlinux_sk1lls_cedfa5fb}
```

The original solve notes confirm that the reconstructed flag was successfully submitted.

## Key Analysis Workflow

```text
Download Log
     ↓
Inspect Log Format
     ↓
Search for "INFO FLAGPART"
     ↓
Collect Relevant Fragments
     ↓
Handle Repeated Entries
     ↓
Reconstruct Flag
     ↓
Submit Result
```

## Lessons Learned

- Large log files should be filtered using distinctive indicators instead of reviewed entirely by hand.
- Log-analysis challenges often hide useful data among routine application events.
- Repeated log entries need to be considered when reconstructing data from fragmented records.
- Simple search and filtering techniques are valuable foundations for incident-response and SOC workflows.
- The same methodology applies beyond CTFs when investigating authentication events, suspicious requests, error patterns, or indicators distributed across logs.

## Key Takeaway

The challenge demonstrates a practical defensive-security skill: extracting meaningful indicators from noisy logs. Identifying a reliable marker such as `INFO FLAGPART` makes it possible to reduce a large dataset to the events relevant to the investigation.

## Source Notes

This writeup is based on the user's existing `picoCTF/Easy/General skills/Log Hunt.txt` solve notes in this repository. The challenge name, objective, `INFO FLAGPART` marker, fragment-reconstruction workflow, and recovered flag are taken directly from those notes.

> This writeup documents an authorized picoCTF challenge environment and is based on the user's original solve material.

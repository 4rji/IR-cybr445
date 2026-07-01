# Codex Instructions: Generate ICS-487 Incident Response Report

## Goal

You will receive an evidence document for an ICS-487 Incident Response assignment. Your task is to analyze the evidence, extract the relevant incident details, and generate two files:

1. `incident_report_draft.md` — a human-readable Markdown draft for review.
2. `incident_report.tex` — a LaTeX version of the same report.

The report must follow the ICS-487 incident report requirements and grading rubric. The final report should be concise, professional, and written as an Incident Response report.

Do **not** invent technical evidence that is not supported by the provided evidence document. Only invent administrative placeholders when the assignment explicitly allows it, such as organization names, incident number, responding company, and sponsor.

---

## Input Files

Assume the working folder contains:

- The evidence document provided by the user.
- `logo.png` for the cover page.
- Optional screenshots/images that may be added later.

If screenshots are referenced in the evidence document but not available as separate image files, insert clear placeholders in Markdown and LaTeX where screenshots should be added later.

Example placeholder:

```md
[SCREENSHOT PLACEHOLDER: Windows Event Log showing suspicious login activity]
```

For LaTeX:

```latex
\begin{figure}[h]
\centering
\fbox{\parbox{0.85\textwidth}{Screenshot placeholder: Windows Event Log showing suspicious login activity}}
\caption{Windows Event Log showing suspicious login activity}
\end{figure}
```

---

## Report Requirements

The report must include the following sections in this order:

1. Cover Page
2. Table of Contents
3. Executive Summary
4. Timeline
5. Findings
6. Investigative Questions
7. Systems / People Involved
8. Indicators of Compromise
9. Evidence Collected
10. Remediation
11. Recommendations
    - Short-Term Recommendations
    - Long-Term Recommendations
12. Lessons Learned
13. Appendices
14. References

The report should be at least 5 pages long when converted to PDF, excluding the cover page and any pictures. Avoid unnecessary filler. The writing should be concise and professional.

---

## Cover Page Requirements

The cover page must include:

- Logo: `logo.png`
- Affected Organization: create a realistic fictional company name
- Incident Name or Incident Number
- Date Published
- Organization that performed the investigation: create a realistic fictional IR/security company name
- The phrase: **Privileged and Confidential**

Use this default cover page information unless the evidence document gives better information:

```text
Affected Organization: Northbridge Financial Services
Incident Name: Unauthorized Access and Malware Execution Investigation
Incident Number: IR-2026-001
Date Published: [Use current date or user-provided date]
Investigation Performed By: Sentinel Ridge Cybersecurity
Classification: Privileged and Confidential
```

---

## Writing Style

Use a professional Incident Response tone.

The Executive Summary and Findings must be written for nontechnical executives.

Technical details should be placed in:

- Investigative Questions
- Evidence Collected
- Indicators of Compromise
- Remediation
- Appendices

Avoid copying long text directly from the evidence document. Retell the story in your own words.

Use IEEE citations only if external sources are used. Evidence from the provided document does not need to be cited unless the user specifically requests it.

---

## Section Instructions

## 1. Executive Summary

Write 2–3 paragraphs.

Must include:

- How the incident was discovered
- What was discovered
- Type of incident
- Response taken
- Goal of the investigation
- Duration of work
- Start and stop date
- Who sponsored/requested the work

Use nontechnical language.

If the evidence does not provide one of these items, create a reasonable administrative placeholder and mark it clearly.

Example:

```text
The investigation was sponsored by Northbridge Financial Services' executive leadership team.
```

---

## 2. Timeline

Create a table with major events in chronological order.

Columns:

| Date/Time UTC | Event | Source of Evidence |

Rules:

- Use UTC.
- Keep the timeline to about half a page.
- Summarize repeated actions.
- Do not include every small log entry.
- Include only events that help explain the incident story.

If a timestamp is local time and the timezone is unknown, write:

```text
[Timezone unknown; listed as shown in evidence]
```

---

## 3. Findings

Write no more than one page.

This section must summarize:

- Main incident story
- Investigative question answers
- Systems or users involved
- Key IOCs
- Evidence used
- Remediation taken

Use nontechnical language. This section should be understandable to executives.

---

## 4. Investigative Questions

Create 5–6 quality investigative questions based on the evidence.

Each question must be answerable using the evidence document.

Use this format:

```md
### Question 1: How was the incident initially identified?

**Answer:** ...
**Supporting Evidence:** ...
```

Possible question types:

- How was the incident discovered?
- What system was affected?
- What user account was involved?
- What malware, tool, command, or suspicious file was executed?
- Did the attacker establish persistence?
- Was there evidence of lateral movement, credential access, or exfiltration?
- What IOCs should be searched enterprise-wide?

Do not include weak or generic questions.

---

## 5. Systems / People Involved

Create a table.

Columns:

| Hostname / Person | IP Address | Role / Purpose | Date/Time of Pertinent Evidence | Compromise Found |

Include:

- Hostname
- IP address
- Username or person involved, if applicable
- System role
- Known compromise
- Relevant timestamp

If information is not available, use:

```text
Not identified in the provided evidence
```

Do not guess technical identifiers.

---

## 6. Indicators of Compromise

Create exactly 10 IOCs based on the evidence.

Use this table:

| # | IOC Type | IOC Value | Why It Matters | Suggested Enterprise Search |

IOC types may include:

- IP address
- Domain
- URL
- File path
- Filename
- File hash
- Registry key
- Scheduled task
- Process name
- Command line
- Username
- Service name
- Mutex
- Email address
- User agent

Rules:

- Every IOC must come from the evidence document.
- Do not invent IOCs.
- Choose IOCs that are useful for enterprise-wide hunting.
- If more than 10 exist, choose the strongest 10.
- If fewer than 10 exist, explain the limitation and include the strongest available evidence-supported indicators.

---

## 7. Evidence Collected

This is one of the most important sections.

For each evidence source, include:

- Evidence type
- Where it came from
- How it was collected or reviewed
- What facts it supports
- Why it matters
- How a third party could replicate the analysis

Use subsections like:

```md
### 7.1 Windows Event Logs

Description:
Collection / Review Method:
Relevant Facts Found:
How It Supports the Investigation:
Replication Steps:
Screenshot Placeholder:
```

Possible evidence categories:

- Windows Event Logs
- PowerShell logs
- Prefetch
- Registry
- Scheduled Tasks
- Browser history
- File system artifacts
- Network artifacts
- Running processes
- Services
- User accounts
- Logon events
- Installed programs
- Malware files
- Command history
- Memory evidence
- External storage evidence

Only include categories supported by the provided evidence.

For screenshots, insert placeholders instead of embedding images unless the image filenames are available.

---

## 8. Remediation

Describe exact steps taken to expel the attacker and restore normal operations.

Include actions such as:

- Isolated affected systems
- Disabled or reset compromised accounts
- Removed malicious files
- Removed persistence mechanisms
- Blocked malicious IPs/domains
- Patched exploited vulnerabilities
- Reimaged or restored affected hosts
- Verified no additional IOC hits existed
- Increased monitoring after containment

Do not be vague. Use exact actions tied to the evidence.

---

## 9. Recommendations

Split into two subsections.

### 9.1 Short-Term Recommendations

These should prevent the exact incident from happening again soon.

Examples:

- Patch the exploited vulnerability
- Reset affected passwords
- Enforce MFA
- Block IOCs
- Reimage affected host
- Remove malicious scheduled task or registry key
- Review similar hosts for the same IOCs

### 9.2 Long-Term Recommendations

These improve the company’s security posture.

Examples:

- Deploy EDR
- Centralize logs in a SIEM
- Improve phishing awareness training
- Implement vulnerability management
- Create an incident response playbook
- Enforce least privilege
- Improve asset inventory
- Implement regular threat hunting

Be specific. Avoid generic recommendations unless tied to findings.

---

## 10. Lessons Learned

Write 1–2 paragraphs.

Describe what was learned from this investigation and how it will be applied to future investigations.

Mention items such as:

- Building a timeline from multiple evidence sources
- Turning investigative leads into IOCs
- Using IOCs for enterprise-wide hunting
- Avoiding overloading the report with unnecessary technical detail
- Preserving evidence and documenting repeatable procedures

---

## 11. Appendices

Use appendices for long material that would interrupt the main report.

Possible appendices:

- Appendix A: Extended Logs
- Appendix B: Screenshot Placeholders
- Appendix C: File Listings
- Appendix D: Command Output

In the main report, reference appendices when needed.

Example:

```text
A longer excerpt of the relevant log entries is included in Appendix A.
```

---

## 12. References

If external sources are used, cite them in IEEE format.

Example:

```text
[1] Microsoft, "Windows Security Auditing Events," Microsoft Learn, 2026. [Online]. Available: ...
```

If no external sources are used, write:

```text
No external sources were used. The report was prepared from the provided evidence document.
```

---

## Markdown Output Requirements

Create `incident_report_draft.md`.

It must:

- Use clear headings
- Include all required sections
- Include tables where appropriate
- Include screenshot placeholders
- Be readable before conversion to LaTeX
- Avoid raw LaTeX except where necessary

---

## LaTeX Output Requirements

Create `incident_report.tex`.

Use:

- `article` class
- 11pt or 12pt font
- `graphicx`
- `geometry`
- `longtable`
- `booktabs`
- `hyperref`
- `float`
- `array`
- `xcolor` only if needed

The LaTeX must compile cleanly with `pdflatex` or `xelatex`.

The cover page must include `logo.png`.

Use this basic LaTeX structure:

```latex
\documentclass[12pt]{article}

\usepackage[margin=1in]{geometry}
\usepackage{graphicx}
\usepackage{longtable}
\usepackage{booktabs}
\usepackage{hyperref}
\usepackage{float}
\usepackage{array}

\begin{document}

% Cover page here

\tableofcontents
\newpage

% Report sections here

\end{document}
```

For screenshots that are not yet available, use placeholders.

---

## Quality Checklist

Before finishing, verify that the report includes:

- [ ] Cover page with affected organization
- [ ] Incident name or number
- [ ] Date published
- [ ] Investigation organization
- [ ] "Privileged and Confidential"
- [ ] Table of contents
- [ ] Executive summary in nontechnical language
- [ ] Start and stop date
- [ ] Sponsor of the work
- [ ] Timeline with UTC timestamps
- [ ] Findings section no longer than one page
- [ ] 5–6 investigative questions
- [ ] Systems / People involved table
- [ ] Exactly 10 IOCs, if evidence supports 10
- [ ] Evidence collected section with repeatable procedures
- [ ] Remediation with exact actions
- [ ] Short-term recommendations
- [ ] Long-term recommendations
- [ ] Lessons learned, 1–2 paragraphs
- [ ] Appendices
- [ ] References / IEEE citation section
- [ ] No unsupported technical claims
- [ ] No long copied paragraphs from the evidence document
- [ ] Markdown draft created
- [ ] LaTeX file created
- [ ] LaTeX compiles without errors

---

## Final Instruction to Codex

Analyze the provided evidence document carefully. Build the report around the evidence. Tell a clear Who, What, When, Where, Why, and How story without making the reader assemble the puzzle themselves. Use concise professional writing. Create both the Markdown draft and the LaTeX report.

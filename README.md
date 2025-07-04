📝 Project Title:

TryHackMe – Benign: Detecting LOLBIN Abuse in Host-Centric Logs via Splunk

🎯 Objective:

Analyze host-centric Windows process execution logs (Event ID 4688) ingested into Splunk to detect unauthorized user accounts, LOLBIN abuse, and malicious payload delivery. This investigation focuses on identifying post-compromise actions taken on an HR department host following suspicious IDS alerts.

🛠️ Tools Used:

Splunk (Search & Reporting)
Windows Event Logs (Event ID 4688)
LOLBIN Identification (bitsadmin, certutil, etc.)
Process Behavior Analysis
Base64 and Command Decoding Tools

❌ Skills Demonstrated:

Event 4688-based threat hunting
User and host behavior correlation
Detection of LOLBIN (Living Off the Land Binary) abuse
Payload delivery analysis and flag extraction
Incident triage across segmented networks

Project Overview
This investigation was conducted in the context of a simulated enterprise where the network is divided into IT, HR, and Marketing departments. Following an IDS alert about potentially suspicious process execution, logs from the affected host were ingested into Splunk under the win_eventlogs index.

Using Windows Event ID 4688, which records every process creation on an endpoint, we tracked indicators of compromise including malicious account creation, scheduled task usage, and payload downloads using system binaries (LOLBINs). The ultimate goal was to extract a malicious flag embedded in the downloaded file and identify the full URL used by the attacker.

Task Breakdown

✏️ Task 1: Count Logs from March 2022
⭕️ Objective: Determine how many logs were ingested from March 2022.
⭕️ Method:

Set time filter in Splunk to March 1–31, 2022.
Run:

spl
Copy
Edit
index=win_eventlogs EventCode=4688 | stats count

✏️ Task 2: Identify the Imposter User
⭕️ Objective: Detect the suspicious or unknown user account created by the attacker.
⭕️ Method:

spl
Copy
Edit
index=win_eventlogs EventCode=4688 | stats count by Account_Name
Compare usernames against known users in each department (HR: Haroon, Chris, Diana). Identify anomalies.

✏️ Task 3: HR User Running Scheduled Tasks
⭕️ Objective: Determine which HR user executed schtasks.exe.
⭕️ Method:

spl
Copy
Edit
index=win_eventlogs CommandLine="*schtasks.exe*"
Filter for HR usernames (Haroon, Chris, Diana). Look at Account_Name field in the event.

✏️ Task 4: HR User Executing LOLBIN to Download Payload
⭕️ Objective: Identify which HR user ran a system binary to fetch a payload.
⭕️ Method:

spl
Copy
Edit
index=win_eventlogs CommandLine="*http*"
Cross-reference Account_Name with HR users. Investigate who ran processes like bitsadmin.exe.

✏️ Task 5: Name the LOLBIN Used
⭕️ Objective: Name the system binary used to download the file.
⭕️ Method:

From Task 4 results, extract the binary used in New_Process_Name or CommandLine.

Common LOLBINs:

bitsadmin.exe

certutil.exe

powershell.exe

✏️ Task 6: Date of Execution
⭕️ Objective: Find the date the LOLBIN was executed.
⭕️ Method:

Check _time field of the log entry from Task 5.

Format the date as YYYY-MM-DD.

✏️ Task 7: Identify the File-Sharing Site Used
⭕️ Objective: Identify the third-party site used for hosting the payload.
⭕️ Method:

spl
Copy
Edit
index=win_eventlogs CommandLine="*http*"
Inspect CommandLine or ParentCommandLine for domains such as transfer.sh, pastebin, anonfiles, etc.

✏️ Task 8: Name of Downloaded File
⭕️ Objective: Extract the name of the file saved to disk.
⭕️ Method:

Look in the CommandLine for bitsadmin or certutil download commands.
Identify the file name after output, outfile, or at the end of the command.

✏️ Task 9: Extract Flag Pattern from Payload
⭕️ Objective: Reveal the malicious content pattern (e.g., THM{...}) in the downloaded file.
⭕️ Method:

Inspect the file contents (e.g., with type, cat, strings, or external analysis).
Look for string pattern: THM{.*}

✏️ Task 10: Reveal Full Malicious URL
⭕️ Objective: Identify the complete URL used to retrieve the malicious payload.
⭕️ Method:

Backtrack to the LOLBIN execution event from Task 5.
Extract full URL from the CommandLine field.

🔍 Analysis and Reflection

💡 Challenges Faced:

Separating legitimate admin activity from malicious impersonation

Tracing LOLBIN activity back to users in a segmented network

Identifying obfuscated or minimal command-line executions

💡 Lessons Learned:

Windows Event ID 4688 offers deep insight into attacker behavior

LOLBINs are commonly abused and often go unnoticed by AV/EDR

Context-aware filtering (user + department + time) improves detection accuracy

💡 Relevance to SOC Analyst Roles:

Reinforces log parsing and event reconstruction skills

Builds familiarity with post-exploitation techniques

Teaches how to enrich Splunk data with behavioral logic

💡 Relevance to Penetration Testing / Red Teaming:

Illustrates how attackers blend into normal activity

Shows how simple payload delivery via built-in tools can succeed

Emphasizes the importance of defense-in-depth and user monitoring

✅ Conclusion

💡 Summary:
Using Splunk and Windows Event ID 4688 logs, this investigation uncovered a suspicious user account, confirmed the use of bitsadmin.exe to download a payload, and identified a malicious file saved from a third-party file host. Through precise filtering and analysis, we extracted the malicious flag and the complete URL used in the attack campaign.

💡 Skills Gained:

Process execution analysis in Splunk

Detection of LOLBIN-based post-exploitation

Attribution of user actions across segmented environments

Extraction of flags and indicators from threat activity

💡 Next Steps:

Configure Splunk alerts for LOLBINs used with external URLs

Correlate Event IDs 4688 with file modification logs (Event ID 4663)

Simulate similar attacks in a lab environment to build detection playbooks



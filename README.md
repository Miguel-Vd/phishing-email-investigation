# Phishing Email Investigation

Investigation of a suspicious email from the TryHackMe **The Greenholt Phish** lab, focusing on email headers, attachment identification, and file reputation.

## Objective

Determine whether the email was malicious and document the evidence supporting the classification.

## Tools Used

- Mozilla Thunderbird — email and header inspection
- Linux `file` — identification of the attachment's actual format
- Linux `sha256sum` — SHA-256 calculation
- VirusTotal — hash reputation checks and archive relationships

## 1. Email Analysis

The email claimed that a SWIFT transfer had been completed and presented an attachment as a payment receipt.

The sender appeared as **Mr. James Jackson**.

<img width="463" height="409" alt="image" src="https://github.com/user-attachments/assets/e23d1297-a912-4b4b-bf26-ed8d99d40563" />

The Reply-To used a different domain from the From address, redirecting replies to a separate mailbox.

The headers recorded an SPF failure. This supported suspicion when combined with the other findings, but SPF failure alone would not prove phishing.

The DMARC result was inconclusive. The public IP was recorded as an investigative indicator, not independently confirmed as malicious.

## 2. Attachment Analysis

The attachment name contained `PDF` and ended in `.CAB`, suggesting a document or CAB archive.

I checked its actual format and calculated its hash:

~~~bash
file SWT*
sha256sum SWT*
~~~

The `file` command identified **RAR archive data, v5**. The actual format therefore differed from the filename extension.

<img width="449" height="143" alt="Captura de tela 2026-10-13 101034" src="https://github.com/user-attachments/assets/c07cec58-b734-46dd-908b-a3037da1206f" />

**Archive SHA-256:**

~~~text
2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f
~~~

I searched for this hash in VirusTotal without uploading or executing the attachment.

<img width="946" height="449" alt="Captura de tela 2026-10-13 101052" src="https://github.com/user-attachments/assets/c019df76-ded0-4f4b-ae29-c6cf9534a67f" />

## 3. Executable Inside the Archive

VirusTotal's **Relations → Bundled Files** section listed one Windows executable inside the archive.

Its filename contained `PDF` but ended in `.com`, which can be an executable extension on Windows. VirusTotal identified it as **Win32 EXE**.

<img width="905" height="241" alt="Captura de tela 2026-10-13 101225" src="https://github.com/user-attachments/assets/6a00c083-6550-4dd0-885d-b70adc6cb968" />

**Executable SHA-256:**

~~~text
05261f5a64f81a34fdde66cc82b573773e5dfa3bb5c3ccbfe2d0eef0e9d7b6c9
~~~

<img width="939" height="408" alt="Captura de tela 2026-10-13 101502" src="https://github.com/user-attachments/assets/60359fd7-ef90-4f8c-9418-67fe56d64dbe" />

The executable's report displayed a different filename. Filenames can vary between submissions; the hash identifies the file content.

Detection counts reflect the reports viewed during this investigation and may change. Vendor labels varied, so I did not attribute the sample to a specific malware family.

## Conclusion

**Classification: malicious phishing email with a malware attachment.**

The conclusion was supported by the combined evidence:

- Financial pretext encouraging the recipient to open a supposed receipt
- Different From and Reply-To domains
- SPF failure recorded in the headers
- Misleading attachment name and extension
- RAR archive containing a Windows executable
- Numerous malicious detections for both the archive and the executable

## Recommended SOC Actions

In a real environment, I would recommend:

- Quarantine the email and search for matching messages in other mailboxes.
- Block the confirmed malicious file hashes using available security controls.
- Check endpoint telemetry for extraction or execution of the payload.
- Escalate to incident response if execution or compromise is identified.
- Validate email addresses, domains, and infrastructure before broader blocking.

## Scope and Limitations

This was a training lab, not a production incident.

The attachment was not executed. The investigation used email inspection, static file identification, and existing VirusTotal reports.

No endpoint telemetry was available to establish execution or compromise. Malicious delivery does not by itself confirm infection.

## Key Takeaways

- Check the actual file format rather than trusting the extension.
- Combine authentication results with message and attachment evidence.
- Use hashes to correlate files across different names.
- Distinguish malicious delivery from confirmed endpoint compromise.

Only documentation and screenshots are included. Email and malware samples are excluded.

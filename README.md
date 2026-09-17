Hunt 23 - JadePuffer
Ticket Overview
Ticket Number: PW-2026-0730
Date/Time Reported: 2026-07-30 / approximately 19:21 UTC
Assigned Analyst: Khatab Aied
Customer/Environment: Flowforge Linux estate
Classification: Agentic Ransomware
Severity: High
Workspace: LAW-HuntPractice
Affected Hosts: ff-lf-01 (10.4.0.10), ff-minio-01 (10.4.0.20), ff-db-01 (10.4.0.30), ff-nacos-01 (10.4.0.40)
Incident Summary
The following incident started from an analytics alert on ff-lf-01 where the Langflow service account started a process it had not normally started before. The activity happened on July 30th 2026 and the main attacker chain ran for about 17 minutes. After investigating the process, web, network, audit, container and LLM agent telemetry i determined that this was not just a single exploit or a normal compromised user session. The attacker used the Langflow /api/v1/validate/code endpoint and CVE-2025-3248 to get remote code execution, then immediately started making decisions across the rest of the Flowforge environment.
From that first foothold the attacker established C2 to 45.131.66.106 on port 4444, created cron persistence under the langflow account, accessed credential material, scanned the internal 10.4.0.0/24 network, authenticated to MinIO with default credentials and retrieved credentials.json from the terraform-state bucket. It then used those credentials to reach Nacos, failed an initial privileged request, corrected the problem and successfully created a local svc_maint account. The chain ended with access to the database backend where 1,342 records were encrypted with AES_ENCRYPT, the config_info and history tables were dropped, and a README_RANSOM table was created with a Bitcoin payment address.
The most important part of this incident is what was driving the chain. LLMAgentLogs_CL shows one human instruction at the beginning of the malicious session, but after that the jadepuffer-agent planned and executed the attack itself. It also handled failures by changing what it was doing, for example when MinIO returned XML instead of JSON it adjusted the parser and refetched the object, and after the first Nacos admin creation failed it corrected the request and succeeded. Based on the session structure the best description is human-tasked rather than a person manually typing each step.

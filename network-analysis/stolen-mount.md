Stolen Mount — Network Forensics Case Study

Room: TryHackMe – Stolen Mount (analysis report, no solutions disclosed)
Discipline: Network Traffic Analysis / Exfiltration Detection
Tools Used: TShark, Zeek, Linux CLI
Objective: Identify suspicious SMB activity and determine whether unauthorized file access occurred.

Summary

This investigation focused on analyzing SMB traffic to determine whether an internal file share was accessed without authorization. The captured network data suggested reconnaissance behavior, followed by targeted access to shared resources that could indicate data theft attempts.

# Stolen Mount — Network Forensics Case Study

**Challenge:** Stolen Mount 
**Discipline:** Network Forensics / NFS Traffic Analysis  
**Primary Tool:** Wireshark
**Protocol:** NFS (Network File System) over TCP  
**Objective:** Analyze NFS traffic to identify unauthorized access, extract hidden archive, and assess exfiltration behavior

---

## Summary

A network packet capture containing NFS traffic was provided as evidence of a suspected breach. The task was to investigate the traffic to find traces of a stolen archive, validate unauthorized file access, and determine if sensitive data was exfiltrated. Through detailed packet analysis and stream reconstruction, suspicious NFS operations, file retrievals, and archive transfers were identified — consistent with a real-world data theft scenario.

---

## Investigation Methodology

### Step 1 – Filtering for NFS Traffic  
Loaded the PCAP file in Wireshark and applied the display filter:  
nfs

yaml
Copy code
to isolate NFS exports, lookups, directory enumeration, and file transfer packets.

### Step 2 – Identifying Export Access & Directory Enumeration  
Searched for NFS procedures such as `LOOKUP`, `READDIR`, and `READ` to uncover which directories and files were being accessed. The traffic revealed nonstandard directory traversal and repeated access to archive-related filenames (e.g. `hidden_stash.zip`, `secrets.png`), indicating targeted reconnaissance rather than normal system activity.

### Step 3 – Extracting File Data from Streams  
Followed relevant TCP streams associated with NFS operations. Exported the raw data corresponding to file transfers (from first magic bytes to end-of-file bytes) and saved it locally as a binary blob. Used ZIP-signature detection (magic bytes `PK`) to identify and extract the archived file.

### Step 4 – Handling Protected Archive & Flag Retrieval  
The extracted archive was password-protected. The packet stream contained a value consistent with an MD5 hash labeled “Archive Password.” After decrypting the hash using an MD5 lookup tool, the password was revealed. With it, the archive was unpacked, yielding a file (e.g. `secrets.png`) — which was a QR code. Scanning the QR code revealed the hidden flag.

---

## 🧠 MITRE ATT&CK Mapping

| Phase        | Technique                                                  |
|--------------|------------------------------------------------------------|
| Reconnaissance | **T1083 – File and Directory Discovery** (NFS share enumeration) |
| Collection     | **T1005 – Data from Local System** (retrieving files via NFS read) |
| Exfiltration   | **T1048 – Exfiltration Over Alternative Protocol** (NFS)        |

---

## Key Indicators & Observations

- NFS export access from unexpected source IP  
- Extensive use of `LOOKUP`, `READDIR`, and `READ` requests on archive-related filenames  
- Archive data transmitted over network instead of local file transfer  
- Presence of MD5 hash in stream labeled as “password”  
- Hidden archive containing a QR-encoded file: sign of staged attack or CTF payload

---

## Defensive Recommendations

1. Restrict NFS exports to authorized hosts/networks only (use `/etc/exports` accordingly)  
2. Enforce authentication and disable anonymous or public exports  
3. Monitor and log NFS procedure activity — flag unusual directory enumerations or bulk read operations  
4. Network-segment file share servers and isolate from untrusted networks  
5. Use intrusion detection rules to alert on NFS read traffic exceeding expected thresholds

---

## Conclusion

The packet capture analysis revealed a systematic NFS-based data theft operation: reconnaissance, archive transfer, and covert data exfiltration. This exercise showcases how malicious actors can leverage standard file sharing protocols like NFS for stealthy data theft. Proper export configuration, network segmentation, and monitoring are essential to defend against such attacks.

---

**Note:** This report does **not** contain flags, exact solution steps, or any proprietary or private content from TryHackMe. It solely documents a conceptual analysis and defensive-minded findings.

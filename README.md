# PCAP Threat Extractor

<p align="center">
  <img src="https://img.shields.io/badge/Security-Network%20Forensics-red?style=flat" alt="Security Domain">
  <img src="https://img.shields.io/badge/MITRE%20ATT%26CK-T1040%20%7C%20T1046%20%7C%20T1552-orange?style=flat" alt="MITRE ATT&CK">
  <img src="https://img.shields.io/badge/Python-3.10+-3776AB?style=flat&logo=python&logoColor=white" alt="Python">
</p>

A high-performance, zero-dependency network packet capture forensics and indicator of compromise (IOC) extraction engine. Inspects standard Libpcap captures (`.pcap`) without external Wireshark / tshark dependencies to harvest cleartext credentials, unauthorized protocol sessions, and stealth port scan activity.

---

## 🎯 The Problem It Solves

During incident response and threat hunting, security analysts must rapidly process multi-gigabyte packet captures on air-gapped or restricted forensic jump hosts where installing external tools like Wireshark, tshark, or heavy dependencies is impractical or prohibited.

**PCAP Threat Extractor** delivers a standalone, pure Python forensic engine that:
1. **Uncovers Exposed Credentials**: Decodes HTTP Basic Authentication headers, FTP plaintext user/pass strings, and Telnet sessions directly from raw byte streams.
2. **Detects Reconnaissance & Port Scans**: Tracks TCP flag patterns (SYN probes without ACK) across multiple destination ports to flag active port scanners (`T1046`).
3. **Zero External Binaries**: Unpacks Ethernet, IPv4, TCP, UDP, and ICMP frame headers using native binary struct decoders.
4. **Structured Incident Response Feeds**: Extracts IOCs (source/destination IP addresses, vulnerable endpoints, harvested tokens) into structured JSON feeds for immediate SIEM correlation.

---

## 🏗️ System Architecture & Workflow

```
[ Raw Network PCAP File ]
            |
            v
+-------------------------------+
|     1. Libpcap Binary Parser  | -> Validates magic numbers (0xD4C3B2A1), reads packet
|    (Frame & Endian Handling)  |    headers, timestamp epoch, and captured byte lengths
+---------------+---------------+
                |
                v
+-------------------------------+
|    2. Protocol Frame Unpacker | -> Extracts Ethernet (0x0800), IPv4 headers,
|    (IP / TCP / UDP / ICMP)    |    TCP control flags, and payload buffers
+---------------+---------------+
                |
                v
+-------------------------------+
|   3. Credential Decoder Engine| -> Searches for HTTP Basic Auth base64 strings,
|    (Plaintext Protocol Regex) |    FTP USER/PASS sequences, and Telnet login streams
+---------------+---------------+
                |
                v
+-------------------------------+
|     4. Reconnaissance Tracker | -> Identifies TCP SYN port scans across distinct
|    (SYN State Tracking & Port)|    port targets from individual source IP addresses
+---------------+---------------+
                |
                v
+-------------------------------+
|     5. Forensics Report & IOC | -> Compiles extracted credentials, threat timeline,
|      (DevSecOps JSON & Table) |    and MITRE ATT&CK indicators into structured report
+-------------------------------+
```

---

## 🚀 Technical Specifications & Capabilities

| Detection Category | Target Traffic | MITRE ATT&CK | Signature / Heuristic |
| :--- | :--- | :--- | :--- |
| **`CLEARTEXT_CREDENTIAL_TRANSMISSION`** | HTTP Port 80 / 8080 | [T1552.001](https://attack.mitre.org/techniques/T1552/001/) | Decodes `Authorization: Basic <base64>` header strings |
| **`CLEARTEXT_CREDENTIAL_TRANSMISSION`** | FTP Port 21 | [T1552.001](https://attack.mitre.org/techniques/T1552/001/) | Extracts `USER <name>` and `PASS <pwd>` cleartext commands |
| **`PORT_SCAN_RECONNAISSANCE`** | TCP SYN probes | [T1046](https://attack.mitre.org/techniques/T1046/) | $\ge 4$ distinct target ports probed with SYN flag (0x02) |
| **`UNENCRYPTED_DATA_STREAM`** | Telnet Port 23 | [T1040](https://attack.mitre.org/techniques/T1040/) | Cleartext command execution and terminal prompts |

---

## 💻 CLI Usage & Quickstart

### Generate a Synthetic Test PCAP Capture
```bash
python -m src.pcap_extractor.cli --generate-sample sample_traffic.pcap
```

### Analyze Forensic Packet Capture
```bash
python -m src.pcap_extractor.cli --pcap sample_traffic.pcap
```

### Export JSON Feed for SIEM / SOAR Ingestion
```bash
python -m src.pcap_extractor.cli --pcap sample_traffic.pcap --json
```

### Run Unit Tests
```bash
python -m unittest discover -s tests -p "test_*.py"
```

---

## 🔒 Production Code & Architecture Notice

* **Public Repository**: Outlines packet parsing specifications, threat signatures, and forensic detection rules.
* **Private Repository**: Maintained in `KarthikeyanSubramaniyam/pcap-threat-extractor` with live socket sniffing adapters, PCAP-NG block decoders, and distributed Zeek/Suricata log correlation engines.

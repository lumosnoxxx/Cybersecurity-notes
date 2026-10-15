# TLS Certificate Verification Using Nmap

## Overview

TLS certificates play an important role in securing communications between clients and servers. They help establish encrypted connections and authenticate the identity of services.

In enterprise environments, certificates may be issued by different Certificate Authorities (CAs), including internal PKI infrastructures and publicly trusted providers.

As part of security operations, it is important to regularly verify deployed certificates to ensure they comply with organizational security requirements.

This article documents a practical approach to **discovering, inspecting, and reporting TLS certificates across multiple servers using Nmap**.

## 1. Objective

The primary objective is to verify whether TLS certificates presented by network services are issued by an expected Certificate Authority.

The verification focuses on:

- Identifying accessible TCP services.
- Retrieving TLS certificates presented by those services.
- Identifying the certificate issuer.
- Reviewing certificate validity periods.
- Detecting certificates issued by unexpected CAs.
- Identifying services requiring further investigation.

The goal is to produce a consolidated certificate inventory that can support security reviews and remediation activities.

## 2. Tools Used

| Tool | Purpose |
|---|---|
| Nmap | Network scanning and TLS certificate discovery |
| Nmap Scripting Engine (NSE) | Certificate extraction using `ssl-cert` |
| OpenSSL | Additional TLS certificate and chain verification |
| Python | Parsing XML results and generating reports |
| CSV / HTML | Readable reporting and analysis |

## 3. Verification Procedure

### Step 1 — Prepare the Scope

Before scanning, identify the target servers and their associated TCP ports.

For example:

| Server | Ports |
|---|---|
| 192.0.2.10 | 5000–5005 |
| 192.0.2.20 | 443, 8443 |

*The IP addresses are documentation examples.*

Scans should be conducted only against authorized systems.

### Step 2 — Verify Network Connectivity

Before attempting certificate extraction, verify whether the target services are accessible.

Using Netcat:

```bash
nc -zv -w 3 192.0.2.10 5000
```

A successful TCP connection indicates that the target port is accepting connections.

Alternatively, use Nmap:

```bash
nmap -Pn -sT -p 5000-5005 192.0.2.10
```

The results can be interpreted as follows:

| State | Description |
|---|---|
| Open | A service is accepting TCP connections |
| Closed | The port is reachable but not accepting connections |
| Filtered | Network filtering or missing responses prevent a definitive determination |

A closed or filtered port does not necessarily indicate that the service is incorrectly configured.

### Step 3 — Retrieve TLS Certificates

Nmap provides the `ssl-cert` NSE script to retrieve certificate information from compatible TLS services.

```bash
nmap -Pn -sT -sV \
  -p 5000-5005 \
  --script ssl-cert \
  -v \
  192.0.2.10 192.0.2.20 \
  -oA certificate_scan
```

**Command explanation:**

| Option | Description |
|---|---|
| `-Pn` | Skip host discovery |
| `-sT` | Perform TCP connect scanning |
| `-sV` | Detect services and versions |
| `-p` | Specify target ports |
| `--script ssl-cert` | Retrieve TLS certificate information |
| `-v` | Enable verbose output |
| `-oA` | Export results into multiple formats |

The command generates:

```text
certificate_scan.nmap
certificate_scan.xml
certificate_scan.gnmap
```

The XML output is especially useful for automated reporting.

### Step 4 — Analyze Certificate Information

When a certificate is retrieved, Nmap can return information such as:

```text
Subject: CN=app.example.com
Issuer: O=Example Certificate Authority
Public Key type: rsa
Public Key bits: 2048
Signature Algorithm: sha256WithRSAEncryption
Not valid before: 2026-01-01
Not valid after: 2027-01-01
```

The most relevant fields are:

| Field | Purpose |
|---|---|
| Subject | Identifies the certificate subject |
| Subject Alternative Name | Identifies DNS names or IP addresses covered |
| Issuer | Identifies the issuing Certificate Authority |
| Not Before | Certificate validity start date |
| Not After | Certificate expiration date |
| Signature Algorithm | Identifies the certificate signature algorithm |
| Public Key | Identifies the public key type and size |

**The issuer field is the main starting point for identifying whether a certificate was issued by the expected CA.**

However, issuer information alone does not prove that a certificate is trusted. Complete validation requires checking the certificate chain, applicable trust anchors, hostname and validity.

### Step 5 — Generate a Consolidated Report

When multiple servers are scanned, analyzing the individual Nmap outputs can become inefficient.

The XML outputs can be parsed using Python and consolidated into CSV or HTML reports.

Example:

| Server | Port | State | Certificate Found | Expected CA |
|---|---|---|---|---|
| 192.0.2.10 | 5000 | Open | Yes | Yes |
| 192.0.2.10 | 5001 | Open | Yes | No |
| 192.0.2.10 | 5002 | Closed | No | Unknown |
| 192.0.2.20 | 8443 | Open | No | Unknown |

This report provides a clear overview of certificate discovery results and helps identify which services require attention.

## 4. Understanding the Results

Not every scan result allows a definitive conclusion.

Three principal situations should be distinguished.

**Case 1 — Port Open, Certificate Found**

A TLS certificate was successfully retrieved.

The certificate issuer can be compared against the expected CA.

If the issuer does not match the expected CA, the service may require certificate replacement or further validation.

**Case 2 — Port Open, No Certificate Found**

The TCP service is accessible, but Nmap could not retrieve a certificate.

Possible explanations include:

- The service does not use TLS.
- TLS is configured on another port.
- The service requires a protocol-specific TLS negotiation.
- The server requires a particular hostname through SNI.
- TLS is disabled or incorrectly configured.

Further investigation by the technical team is required before drawing a conclusion.

**Case 3 — Port Closed**

No TCP service is accepting connections on the tested port from the scanning machine.

The technical team should confirm whether the service is expected to be operational and whether the tested port is correct.

A closed port does not indicate certificate non-compliance by itself.

## 5. Additional Verification with OpenSSL

For services requiring closer inspection, OpenSSL can be used to retrieve the certificate and the chain presented by the server.

```bash
openssl s_client \
  -connect app.example.com:443 \
  -servername app.example.com \
  -showcerts </dev/null
```

The `-servername` parameter enables Server Name Indication (SNI).

This is especially important when several applications share the same IP address but present different certificates depending on the requested hostname.

The output can help investigate certificate chains and TLS negotiation issues. A separate trust validation is needed to establish whether the chain is trusted.

## 6. Findings and Remediation

After completing the verification, findings should be classified according to their status.

| Finding | Recommended Action |
|---|---|
| Expected certificate issuer detected | Validate the certificate and trust chain |
| Unexpected certificate issuer detected | Review certificate ownership and replacement requirements |
| Open port without certificate | Investigate service configuration and TLS requirements |
| Closed port | Confirm the expected operational status |
| Certificate expired | Prioritize certificate renewal |
| Certificate nearing expiration | Plan renewal before expiry |

Findings that cannot be resolved through remote inspection should be escalated to the responsible technical teams.

After corrective actions, perform a new verification to confirm the expected certificate is presented.

## 7. Limitations

Network-based TLS certificate discovery has several limitations:

- It identifies certificates **presented by accessible services**, not every certificate installed on a server.
- Applications behind load balancers or reverse proxies may present certificates different from those installed on backend systems.
- Services using SNI may require hostname-based scanning.
- Some application protocols require specific negotiation before TLS certificates can be retrieved.
- Identifying a certificate issuer does not automatically validate certificate trust.
- A successful scan does not prove that all certificate-related security requirements have been satisfied.

For comprehensive certificate inventory management, network scanning should be complemented by host-level inspection and certificate lifecycle management processes.

## 8. Key Takeaways

This approach demonstrates how basic network discovery tools can be used to support certificate compliance activities.

The main lessons are:

1. Validate network connectivity before attempting TLS certificate discovery.
2. Use Nmap and its `ssl-cert` NSE script to retrieve certificate metadata.
3. Export results into structured formats for easier analysis.
4. Distinguish confirmed certificate findings from inconclusive scan results.
5. Coordinate with technical teams to investigate services whose TLS status cannot be determined remotely.
6. Perform follow-up verification after certificate changes.

**Certificate discovery is an important first step in certificate lifecycle management, but it should not be confused with complete certificate trust validation.**

---

*Tools: Nmap · OpenSSL · Python*

*Category: Cybersecurity / Security Operations / TLS Security*

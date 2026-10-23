# PHP2Shell — Vulnerability Research & Lab Analysis

**Research topic:** PHP2Shell — CVE-2026-109618 and CVE-2026-109619  
**Project type:** Security research, proof-of-concept analysis, and defensive validation  
**Status:** Experimental — results depend on the PHP build and runtime configuration

## Overview

This repository contains a proof of concept (PoC) and supporting research notes for investigating PHP2Shell, a pair of PHP security vulnerabilities associated with out-of-memory (OOM) handling and stale request state in PHP's session upload-progress functionality.

The purpose of this project is to document the vulnerability mechanics, understand the heap-disclosure stage, examine the prerequisites for reproducibility, and support controlled testing in an isolated laboratory.

This project is for authorized security research and defensive validation. It is not intended for scanning or exploiting systems without explicit permission.

## Vulnerability Summary

The PHP2Shell research concerns two reported vulnerabilities:

- **CVE-2026-109618:** Remote code execution associated with out-of-memory corruption during request cleanup.
- **CVE-2026-109619:** Remote code execution associated with stale request-owned global state being reused by a subsequent request on the same worker.

The documented exploit chain relies on PHP request-processing behavior involving session upload progress. Depending on the affected build and configuration, the underlying memory-management issues may permit memory disclosure and potentially lead to code execution.

A PHP version banner alone does not prove exploitability. The actual PHP build, vendor patches, server API, architecture, and runtime configuration must be evaluated.

## Research Objectives

This project focuses on the following objectives:

1. Understand the relationship between OOM handling and stale PHP session state.
2. Explain the role of heap leaks in defeating address-space layout randomization (ASLR).
3. Identify the environmental conditions that influence reproducibility.
4. Compare observed behavior against published research and known laboratory results.
5. Document defensive controls, patch requirements, and detection opportunities.

## Technical Background

### 1. Heap Leak

A heap leak is an information-disclosure condition in which a program exposes data from dynamically allocated memory that should not be accessible to an external requester.

In exploitation research, a disclosed pointer may reveal information about the process's memory layout. This can help an attacker reason about randomized memory addresses, although a leak alone does not establish code execution.

### 2. Stale State and Out-of-Memory Handling

The published PHP2Shell research describes a request-processing sequence in which an allocation failure may interrupt normal cleanup while internal session state remains stale.

If a subsequent request handled by the same worker reuses that state incorrectly, the resulting behavior may expose memory or create conditions for further memory corruption.

### 3. Exploitation Stages

At a conceptual level, the investigated chain consists of:

1. **Heap disclosure:** Determine whether the vulnerable request lifecycle exposes useful heap information.
2. **Runtime address disclosure:** Determine whether memory disclosure reveals information about the PHP runtime's address space.
3. **Control-flow impact:** Assess whether the specific build and memory layout permit the reported corruption to influence program execution.

These stages are distinct. Success at one stage does not guarantee success at the next.

## Reproducibility Considerations

Exploit behavior can vary significantly between builds and configurations. Relevant factors include:

- Exact PHP release and vendor patch level
- Operating system, distribution, and CPU architecture
- PHP Server API (SAPI)
- Apache module and worker configuration
- `session.upload_progress` settings
- Error-reporting and memory-limit settings
- Per-build memory-layout offsets
- Whether successive requests reach the same worker

Published PoC profiles are specific to the builds for which they were developed and verified. A profile for one distribution or architecture should not be assumed to work on another.

A version identified as potentially affected should not automatically be described as successfully exploited.

## Laboratory Scope

Testing should be conducted exclusively in an isolated, disposable environment owned by the researcher or explicitly authorized for testing.

Recommended safeguards:

- Run the vulnerable application in a disposable VM or container.
- Bind laboratory services to loopback interfaces wherever possible.
- Do not expose vulnerable services to public networks.
- Use synthetic data and avoid production credentials.
- Monitor application logs and container health during testing.
- Expect memory-exhaustion tests to cause worker failures or service interruptions.
- Stop testing if the environment becomes unstable.

A local laboratory result is evidence about that particular configuration, not proof that another server is exploitable.

## Defensive Recommendations

### Patch Management

Prioritize vendor-supported fixed PHP releases or applicable vendor backports. According to the referenced NCSC-NL operational summary dated 21 October 2026, the published fixed releases include:

- PHP 8.2.35
- PHP 8.3.36
- PHP 8.4.27
- PHP 8.5.12

Confirm current release information and distribution-specific advisories before making remediation decisions.

### Configuration Hardening

- Disable `session.upload_progress` if the application does not require it.
- Keep `display_errors` disabled in production.
- Apply sensible request-body and PHP memory limits.
- Restrict oversized multipart requests where operationally appropriate.
- Run PHP workers with least privilege.
- Keep document roots and application files non-writable by the web-server account wherever possible.
- Restart affected workers after patching or applying mitigations, as recommended by the vendor.

Configuration changes are defense-in-depth measures; they are not substitutes for applying the appropriate security fixes.

### Monitoring

Investigate unusual combinations of:

- Repeated multipart requests with unusually large bodies or many fields
- Repeated out-of-memory errors and HTTP 500 responses
- Unexpected Apache child-process crashes or restarts
- Memory allocator corruption messages
- Unusual session-cookie behavior

These signals are not individually conclusive. Correlate them with application and infrastructure logs.

## Research References

The following resources provide background and supporting technical context:

1. **Recorded Future — PHP2Shell Vulnerability Exploitation**  
   https://www.recordedfuture.com/blog/php2shell-vulnerability-exploitation

2. **NCSC-NL — PHP2Shell Operational Information**  
   https://github.com/NCSC-NL/php2shell

3. **devploit — PHP2Shell Research Toolkit and Laboratory Documentation**  
   https://github.com/devploit/php2shell

4. **PHP Official Security Advisories**  
   https://www.php.net/security

The referenced projects and publications have different purposes and verification scopes. Consult their current documentation for updated findings, affected products, fixed builds, and known limitations.

## Research Limitations

- Findings may change as additional builds and configurations are tested.
- An unverified heap leak or memory-corruption condition is not equivalent to demonstrated remote code execution.
- A PoC that works in a laboratory may fail against a differently compiled or configured build.
- Version strings and HTTP banners can be inaccurate or incomplete.
- This repository does not claim original discovery of the underlying CVEs unless explicitly supported by documented evidence.

## Responsible Use

Use this material only on systems you own or for which you have explicit authorization.

Do not use the PoC against public targets, third-party infrastructure, or production systems without documented permission and an approved testing plan. Exploitation can cause process crashes, service disruption, and unintended access to sensitive information.

The researcher is responsible for complying with applicable laws, disclosure requirements, and the authorization terms of each engagement.

## Disclaimer

This repository is provided for educational purposes, vulnerability research, and defensive validation. It is supplied without guarantees of completeness, accuracy, or fitness for a particular purpose. No permission to test third-party systems is implied by the availability of this code.

---

**Project focus:** Understanding PHP2Shell's memory-disclosure mechanics, documenting reproducibility constraints, and improving defensive validation.

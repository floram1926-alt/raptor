---
name: solidity-threat-model-generation
description: Use when starting a security audit of a new target project or codebase, before searching for vulnerabilities. Generates a focused threat model from prior CVEs and architecture analysis.
---

# Threat Model Generation

Build the system context and threat model **before** prompting for vulnerabilities. This is the only scaffolding that consistently improves finding quality. Without it, audits degrade into breadth-first hallucination — long lists of generic CWE possibilities with no prioritization.

## Process

1. **Gather prior CVEs/advisories** for the target project. Search GitHub Security Advisories, NVD, and the project's own changelog/security policy.

2. **Feed CVE descriptions to identify plausible bug classes.** Prompt:
   > "Based on these previously disclosed CVEs for [project], generate a threat model identifying the most plausible bug classes and vulnerable subsystems. Group by root cause pattern, not by CVE ID."

3. **Map the attack surface:**
   - **Entry points**: HTTP routes, RPC handlers, CLI entrypoints, message consumers, scheduled jobs, plugin hooks
   - **Trust boundaries**: browser↔server, service↔service, plugin↔host, sandbox↔privileged, tenant↔tenant
   - **High-risk operations**: deserialization, templating, native bindings, authz checks, parsing untrusted input, cryptographic operations, file operations

4. **Define the attacker-victim model explicitly.** Pick one per audit pass:
   - Remote unauthenticated attacker
   - Remote authenticated low-privilege user
   - Cross-tenant / multi-tenant boundary
   - Local attacker with code execution
   - Malicious plugin/extension author

5. **Identify thin slices** — independent attack surfaces that can be audited separately (auth, session, parsing, file upload, sandbox boundary, etc.)

## Output

Save to `notes/<target>/threat-model.md`. Keep it under one page. This document becomes the **only context** passed to slice audit agents.

## What NOT to do

- Don't audit yet. This phase is recon only.
- Don't write a 20-page threat model. Compression is the point.
- Don't skip prior CVE research — it reveals what bug classes the project actually accepts as valid.

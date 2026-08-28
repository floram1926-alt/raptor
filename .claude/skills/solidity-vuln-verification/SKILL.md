---
name: solidity-vuln-verification
description: Use after a slice audit produces candidate vulnerabilities. Validates findings by building PoCs, writing reproducer tests, and classifying exploitability. Use before writing any report or advisory.
---

# Vulnerability Verification

Do not trust "the model says it's vulnerable." Every candidate finding from a slice audit must be validated with evidence before reporting.

## Process

1. **Reproduce the environment**: clone the target, install dependencies, build from source, start a local instance. If the target can't run locally, write unit tests against the vulnerable code path in isolation.

2. **Build the PoC**: turn the slice audit's PoC sketch into a runnable exploit.
   - Web targets: full HTTP request with `curl`, `requests`, or `httpx`
   - Binary targets: `pwntools` script with exact offsets
   - Auth/authz bugs: demonstrate the privilege boundary violation end-to-end
   - Logic bugs: write a test case that asserts the broken invariant

3. **Run and observe**: execute the PoC against the local instance. Capture the response, state change, or crash. If it doesn't work, determine whether the finding is a false positive or the PoC needs adjustment.

4. **Classify each finding**:

   | Classification | Criteria | Action |
   |---------------|----------|--------|
   | **Confirmed exploitable** | PoC succeeds, impact demonstrated | Write up for report |
   | **Likely valid, needs conditions** | Code path exists but requires specific config/state | Document preconditions, attempt to meet them |
   | **Theoretical** | Vulnerable pattern exists but unreachable or mitigated | Note for completeness, low priority |
   | **False positive** | Model hallucinated the vulnerability or misread the code | Discard, note the failure mode to improve future prompts |

5. **Write the finding**: for each confirmed vulnerability, produce:
   - Root cause (the actual bug, not the symptom)
   - Affected versions / code location
   - Attacker preconditions and capabilities required
   - Impact and CVSS estimate
   - Complete, runnable PoC
   - Suggested remediation

## Output

Save confirmed findings to `notes/<target>/verified/<vuln-name>.md`. Save the runnable PoC to `tools/<target>/<vuln-name>-poc.py` (or appropriate extension).

## Token Budget

Verification should consume ~20-30% of the total audit token budget. If you're spending more, the slice audit is producing too many false positives — tighten the threat model or narrow the slices.

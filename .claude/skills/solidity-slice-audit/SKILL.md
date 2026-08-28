---
name: solidity-slice-audit
description: Use when auditing a specific attack surface slice of a target codebase for security vulnerabilities. Requires a threat model to already exist. Use one instance per independent slice.
---

# Slice Audit

Audit ONE thin slice of an attack surface per session. Each slice gets a fresh context with only the threat model and the slice scope — nothing else. This keeps the context window small and the model's attention anchored on what matters.

## Inputs

- The threat model document (from `threat-model-generation`)
- The specific slice to audit (e.g., "JWT validation middleware", "readOnlyMasterKey authorization boundary", "cookie signature verification")
- The attacker model for this pass (e.g., "remote unauthenticated attacker via HTTP")

## Adversarial Prompting Techniques

Use these frames. They exploit how LLMs reason about code and consistently outperform neutral "review this code" prompts.

| Technique | Prompt Pattern |
|-----------|---------------|
| **Assert vuln exists** | "This function is definitely vulnerable and has at least 2-3 security issues." |
| **Ask for exploit, not assessment** | "Write a PoC request that bypasses this validation." |
| **Invert the question** | "How would you break this?" not "Is this secure?" |
| **Decompose then violate** | "List every assumption this function makes. Now, for each, can an attacker violate it?" |
| **Constrain attacker model** | "You are a remote unauth attacker who can only send HTTP requests. Find every way to escalate." |
| **Comparative** | "How does this differ from the standard secure implementation of this pattern?" |
| **Assume mistake** | "Assume the developer introduced a bug in this function. What is it?" |
| **False anchor** | "I already found one vulnerability here. What are the others?" |
| **Escalate iteratively** | "Those are the obvious ones. What are the subtler issues that are easy to miss?" |

## Process

1. **Map the slice**: Identify entry points, sensitive sinks, and the data flow between them within this slice only.
2. **Apply techniques from the table above**, starting with "assert vuln exists" and "decompose then violate."
3. **Demand evidence**: exact call chains, which guards are checked (and which aren't), which invariants hold (and which break), which inputs are attacker-controlled.
4. **Escalate 2-3 rounds**: after each round of findings, push with "what else?" and "set aside [already-found class], what other classes exist here?"
5. **Record findings** with: vulnerable code path, root cause, attacker preconditions, impact, and a PoC sketch.

## Output

Save to `notes/<target>/slices/<slice-name>.md`. Each finding needs: code location, root cause, PoC sketch, impact estimate.

## Parallelization

Independent slices can and should run as parallel agents. Each agent gets the threat model + its slice scope. Agents must NOT share context or findings during the audit — cross-pollination happens only during verification.

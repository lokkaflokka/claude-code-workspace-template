# Company LLM → Claude Handoff Template

Use this when you've done sensitive analysis in the company LLM and need Claude to help structure, generalize, or build on the output.

**Steps:**
1. Complete the sensitive analysis in the company LLM
2. Extract the abstracted findings (no PII, no raw data)
3. Fill in this template
4. Run the sensitivity lint (root CLAUDE.md → LLM Usage Policy) one final time
5. Paste into Claude session

---

**Goal:**
[What you're trying to accomplish]

**Constraints:**
[Technical, business, or timeline constraints]

**Key findings (abstracted — no identifiers):**
- [Finding 1]
- [Finding 2]
- [Finding 3]

**Decision needed:**
[What decision this analysis informs]

**Options considered:**
1. [Option A] — [pros/cons]
2. [Option B] — [pros/cons]

**What good looks like:**
[Success criteria for the output]

**Open questions:**
- [Question 1]
- [Question 2]

---

*Before pasting: verify no PII, production data, or secrets made it into the abstracted findings. Use the redaction standard (root CLAUDE.md) for any borderline content.*

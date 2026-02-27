---
description: Review current changes for correctness, security, and quality
---
Review all staged and unstaged changes in this repository:

1. **Summarise:** What was changed and why? (infer intent from commit messages and code)
2. **Logic check:** Are there any logic errors, off-by-one issues, or missed edge cases?
3. **Security scan:** Check for:
   - Hardcoded secrets or credentials
   - SQL injection or command injection vectors
   - Missing input validation
   - Overly permissive access controls
4. **Style check:** Do changes follow the coding standards in CLAUDE.md?
5. **Test coverage:** Are new code paths covered by tests? Identify gaps.
6. **Completeness:** Are there any TODO comments, partial implementations, or dead code?
7. **Verdict:** Provide a structured summary:
   - **Issues** (severity: critical / high / medium / low)
   - **Suggestions** (optional improvements)
   - **Questions** (things to clarify with the author)

Be thorough and critical. Ask at least 3 probing questions about the changes.

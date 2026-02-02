# /reviewthis

Description: Performs a hybrid v3.0 code review (Standard Plugin + Deep Arch Analysis) on a Pull Request.
Argument: pr_number - The PR number to review

---

## Phase 1: Context & Pre-flight

**Goal**: Gather deep context and ensure the PR is ready for review.

### Step 1.1: Eligibility Check
Run these commands to verify the PR is reviewable:
```bash
gh pr view <pr_number> --json state,isDraft,author,title
gh pr checks <pr_number>
```

**STOP the review if any of these conditions are true:**
- PR state is "CLOSED" or "MERGED"
- PR is a Draft (`isDraft: true`)
- Author is a bot (login contains `[bot]`)
- CI/Lint checks have failed

### Step 1.2: Context Mapping
1. Find all `CLAUDE.md` files in the repo root and any subdirectories touched by the PR
2. Fetch the full PR description: `gh pr view <pr_number>`
3. Get the list of modified files: `gh pr diff <pr_number> --name-only`
4. **Deep Read**: For every modified source file (`.ts`, `.py`, `.vue`, `.js`, `.tsx`, `.jsx`), use `Read` to get the **full content** (skip files > 50kb)

---

## Phase 2: The Multi-Agent Swarm

Spawn **6 parallel agents** using the Task tool. Each agent outputs a JSON list of issues.

### Agent A: SOLID Principles Analyzer (model: sonnet)
**Subagent type**: `general-purpose`
**Role**: Enforce project standards (`CLAUDE.md`) and SOLID compliance.
**Tasks**:
1. **CLAUDE.md Check**: Verify specific rules (e.g., "Use `willReturn().given()`", "No comments", TDA principle)
2. **Comment Policy**: Scan for `TODO`, `FIXME`, or implementation comments. These violate "No comments unless explaining why"
3. **Style**: Check for naming conventions and Primitive Obsession violations
4. **SOLID Violations**: Single Responsibility, Open/Closed, Liskov, Interface Segregation, Dependency Inversion

**Output format**:
```json
[{"file": "path", "line": N, "issue": "description", "rule": "CLAUDE.md quote or SOLID principle", "severity": "high|medium|low"}]
```

### Agent B: Architecture Pattern Advisor (model: sonnet)
**Subagent type**: `general-purpose`
**Role**: Validate architectural patterns and layer boundaries.
**Tasks**:
1. **Layer Violations**: Check Clean Architecture boundaries (domain -> application -> infrastructure -> api)
2. **Pattern Compliance**: Verify correct use of protocols, dependency injection, value objects
3. **Import Analysis**: Detect forbidden cross-layer imports

**Output format**:
```json
[{"file": "path", "line": N, "issue": "description", "pattern": "violated pattern", "severity": "high|medium|low"}]
```

### Agent C: Arch Critic (model: opus)
**Subagent type**: `general-purpose`
**Role**: Deep logic validation and historical analysis.
**Tasks**:
1. **Logic Challenge**: Does the PR description match the code? Is the feature logically sound?
2. **History Check**: Run `git blame` on modified lines to detect if breaking recent fixes. Search closed PRs for the modified files
3. **Data Flow**: Trace critical paths for race conditions or unhandled edge cases
4. **Regression Risk**: Identify changes that could break existing functionality

**Output format**:
```json
[{"file": "path", "line": N, "issue": "description", "evidence": "git blame or PR reference", "severity": "high|medium|low"}]
```

### Agent D: SRE Security Auditor (model: haiku)
**Subagent type**: `general-purpose`
**Role**: Security, reliability, and obvious bugs.
**Tasks**:
1. **Shallow Scan**: Look for obvious crashes (NPEs, null checks, off-by-one errors)
2. **Security (P0)**: Check for SQL Injection, XSS, Broken Auth, Hardcoded Secrets, SSRF
3. **Resilience**: What happens on timeout/disconnect? (WebSocket drops, API failures)

**Output format**:
```json
[{"file": "path", "line": N, "issue": "description", "category": "security|reliability|bug", "severity": "critical|high|medium"}]
```

### Agent E: Complexity Analyst (model: sonnet)
**Subagent type**: `general-purpose`
**Role**: Cognitive complexity and maintainability.
**Tasks**:
1. **Cognitive Complexity**: Identify deeply nested code, long functions, spaghetti code
2. **Over-abstraction**: Detect unnecessary abstractions or premature optimization
3. **Refactoring Needs**: Identify code that will be hard to maintain in 6 months

**Output format**:
```json
[{"file": "path", "line": N, "issue": "description", "metric": "complexity score or description", "severity": "high|medium|low"}]
```

### Agent F: Test Gap Analyst (model: haiku)
**Subagent type**: `general-purpose`
**Role**: Test coverage and quality.
**Tasks**:
1. **Test Gap**: Compare source changes vs test changes. Are new branches covered?
2. **Test Quality**: Are tests following AAA pattern? Using proper mocking (`willReturn().given()`)?
3. **Missing Edge Cases**: Identify untested error paths and boundary conditions

**Output format**:
```json
[{"file": "path", "line": N, "issue": "description", "missing_test": "what should be tested", "severity": "high|medium|low"}]
```

---

## Phase 3: The Judge (Verification)

**Goal**: Eliminate false positives.

Spawn a **haiku agent** to review the combined list of issues from Phase 2.

**Scoring Rubric (0-100)**:
- **0-25 (Ignore)**: False positive, pre-existing issue, or pedantic nitpick not backed by `CLAUDE.md`
- **26-50 (Minor)**: Real issue but low impact (style preference)
- **51-79 (Moderate)**: Worth mentioning but not blocking
- **80+ (Report)**: Explicit `CLAUDE.md` violation, Security Risk, Logic Bug, or Regression

**Instructions for Judge**:
1. Verify each issue against the specific text in `CLAUDE.md` if a rule is cited
2. Check if the issue is pre-existing (not introduced by this PR)
3. Discard anything with Score < 80
4. Merge duplicate issues
5. Rank remaining issues by severity

---

## Phase 4: Final Report

Post a comment on the PR using `gh pr comment <pr_number> --body "..."`.

**Format**:
```markdown
### 🛡️ Hybrid Code Review

Found **X** verified issues:

#### Critical/High Priority
1. **<Title>** (Score: <Score>)
   <Brief Description>
   *Reference*: `CLAUDE.md` says "<Quote>" (if applicable)
   📍 `<file>:<line>`

#### Medium Priority
2. ...

---

**♻️ Refactoring Suggestions** (Optional):
<Diff block if complexity analyst found major improvements>

---
🤖 Generated with `/reviewthis` (v3.0)
```

If no issues with score >= 80 are found, post:
```markdown
### ✅ Hybrid Code Review

No significant issues found. PR looks good to merge.

🤖 Generated with `/reviewthis` (v3.0)
```

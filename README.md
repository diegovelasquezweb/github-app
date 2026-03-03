# a11y GitHub App — Architecture

## Full Flow

| Step | Lives in | Action |
|------|----------|--------|
| 1 | GitHub | Someone comments `/a11y` on a PR |
| 2 | GitHub App | Detects the comment, extracts repo, branch, PR number, commit SHA |
| 3 | GitHub App | Clones the repo at the correct branch/commit |
| 4 | GitHub App | Installs dependencies and starts the app |
| 5 | GitHub App | Waits for localhost:3000 to respond |
| 6 | GitHub App → Claude Code Server | Sends audit job: `{ localPath, baseUrl }` |
| 7 | Claude Code Server | Runs audit: DOM scan + code scan + analyzer → generates remediation.md |
| 8 | Claude Code Server → GitHub App | Returns `{ jobId, findings }` |
| 9 | GitHub App | Posts comment on PR with issue list |
| 10 | GitHub | Reviewer replies with a fix command |
| 11 | GitHub App | Detects the command (e.g. `/a11y fix A1 A3`) |
| 12 | GitHub App | Resolves which issues to fix |
| 13 | GitHub App → Claude Code Server | Sends fix job: `{ jobId, fixes: [A1, A3] }` |
| 14 | Claude Code Server | Reads remediation.md, applies only the approved fixes |
| 15 | Claude Code Server | Runs re-scan to verify fixes |
| 16 | Claude Code Server → GitHub App | Returns `{ summary, changedFiles }` |
| 17 | GitHub App | Commits changes, pushes to new branch, opens fix PR |
| 18 | GitHub App | Posts final comment in original PR |

---

## Supported Commands

| Command | Action |
|---------|--------|
| `/a11y` | Run audit and post findings |
| `/a11y fix A1 A3` | Apply specific fixes by ID |
| `/a11y fix safe-only` | Apply all structural fixes |
| `/a11y ignore A2` | Exclude an issue from the fix queue |

---

## Issue Classification

| Type | Examples | Action |
|------|----------|--------|
| Safe | Missing form label, icon button without name, ARIA | Auto-applicable — goes in fix PR |
| Review | Color contrast, font-size, focus styles | Suggested only — comment in PR |

---

### 1. Scan Flow

```mermaid
%%{init: { 'theme': 'base', 'themeVariables': { 'primaryColor': '#3b5cd9', 'primaryTextColor': '#1e293b', 'primaryBorderColor': '#1e308a', 'lineColor': '#64748b', 'secondaryColor': '#f1f5f9', 'tertiaryColor': '#fff', 'mainBkg': '#fff', 'nodeBorder': '#e2e8f0', 'clusterBkg': '#f8fafc', 'clusterBorder': '#cbd5e1' } } }%%
flowchart LR
    A["<b>Developer</b><br/>comments /a11y on a PR"]
    B["<b>GitHub App</b><br/>Clone · Install · Start app"]
    C["<b>Claude Code Server</b><br/>Runs a11y skill"]
    D["<b>GitHub App</b><br/>Post issue list on PR"]

    A --> B --> C --> D

    linkStyle default stroke:#64748b,stroke-width:2px;
    classDef default font-family:Inter,sans-serif,font-size:14px,min-width:200px;
    classDef dev fill:#f0fdf4,stroke:#86efac,color:#166534;
    classDef app fill:#eff6ff,stroke:#93c5fd,color:#1e3a8a;
    classDef srv fill:#3b5cd9,color:#fff,stroke:#1e308a,stroke-width:2px;

    class A dev;
    class B,D app;
    class C srv;
```

---

### 2. Fix Flow

```mermaid
%%{init: { 'theme': 'base', 'themeVariables': { 'primaryColor': '#3b5cd9', 'primaryTextColor': '#1e293b', 'primaryBorderColor': '#1e308a', 'lineColor': '#64748b', 'secondaryColor': '#f1f5f9', 'tertiaryColor': '#fff', 'mainBkg': '#fff', 'nodeBorder': '#e2e8f0', 'clusterBkg': '#f8fafc', 'clusterBorder': '#cbd5e1' } } }%%
flowchart LR
    A["<b>Developer</b><br/>comments /a11y fix A1 A3"]
    B["<b>GitHub App</b><br/>Parse command · Send fix job"]
    C["<b>Claude Code Server</b><br/>Runs a11y skill"]
    D["<b>GitHub App</b><br/>Commit · Push · Open fix PR"]

    A --> B --> C --> D

    linkStyle default stroke:#64748b,stroke-width:2px;
    classDef default font-family:Inter,sans-serif,font-size:14px,min-width:200px;
    classDef dev fill:#f0fdf4,stroke:#86efac,color:#166534;
    classDef app fill:#eff6ff,stroke:#93c5fd,color:#1e3a8a;
    classDef srv fill:#3b5cd9,color:#fff,stroke:#1e308a,stroke-width:2px;

    class A dev;
    class B,D app;
    class C srv;
```

---

### PR Comment Format

```
## a11y Audit — 3 issues found

| ID | Issue | Severity | Type |
|----|-------|----------|------|
| A1 | Missing form label | Critical | safe |
| A2 | Modal missing accessible name | Serious | review |
| A3 | Icon button without name | Serious | safe |

**Reply with a command:**
- `/a11y fix A1 A3` — fix specific issues
- `/a11y fix safe-only` — fix all safe issues
- `/a11y ignore A2` — exclude an issue
```

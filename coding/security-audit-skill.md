# Security audit skill

Installed at `C:\Users\Faris\.cursor\skills\security-audit`.

Open a new Cursor chat in the repository you want reviewed, then name the mode in the first message. The skill stays in guidance mode unless you ask for a full audit.

## Guidance mode

Use this for a question, one area, or one suspected bug. The agent reads the relevant guide and answers in chat. It does not create an audit folder.

```text
Use the security-audit skill in guidance mode.
Review authorization on the chat endpoints.
Do not write audit artifacts and do not change source.
```

```text
Use the security-audit skill in guidance mode.
Is this file upload path a real trust-boundary failure? Show the source trace.
```

## Full audit mode

Use this when you want the six-phase audit and written reports. Say `full audit mode`, name the target, and name an output folder outside the repository.

```text
Use the security-audit skill in full audit mode.
Target: C:\path\to\repo
Profile: quick
Output: C:\Users\Faris\security-audit-skill\repo-name\run-1
Do not modify source. Do not probe live services.
```

Profiles:

| Profile | What it does |
|---|---|
| `quick` | One hunter wave, then one coverage check. Good first pass. |
| `standard` | Full hunting waves and a separate verifier for each candidate. Default. |
| `deep` | Finer coverage and a second pass on code already reviewed. |

Add a scope when you do not want the whole repository:

```text
Use the security-audit skill in full audit mode.
Target: C:\path\to\repo
Profile: standard
Scope: src/api and the auth middleware only
Output: C:\Users\Faris\security-audit-skill\repo-name\run-2
```

Add `Budget: 12 agent calls` when you want a hard stop. If that budget cannot cover reconnaissance plus the reserved critic and verifier calls, the run stops and is marked incomplete.

### Files a full run writes

All of these go in the output folder:

| Phase | Result |
|---|---|
| 1. Reconnaissance | `architecture.md`, `coverage-ledger.json` |
| 2. Hunting | Notes under `agents/<id>/scratch/` |
| 3. Validation | Each candidate checked by a fresh agent |
| 4. Findings | `findings.json` |
| 5. Verification | A second fresh agent checks every final claim |
| 6. Report | `REPORT.md`, `FINDINGS-DETAIL.md`, `NEEDS-VALIDATION.md` |

Verdicts in `findings.json`:

- `confirmed` — source trace and observed result are complete. This is the only verdict with a severity.
- `needs_validation` — one specific fact is still missing. No severity.
- `rejected` — the candidate was disproved.

After the ledger and after `findings.json`, the parent runs:

```bash
node "$HOME/.cursor/skills/security-audit/validate-coverage-ledger.cjs" <output-dir>/coverage-ledger.json
node "$HOME/.cursor/skills/security-audit/validate-findings.cjs" <output-dir>/findings.json
```

Run a full audit from WSL with Node installed there. The validators require POSIX file protections, so those two commands fail under Windows Node.

## Which mode to pick

| You want | Mode |
|---|---|
| An explanation or a look at one path | Guidance |
| A written report for a repo or a folder | Full audit |
| A fast first report | Full audit, profile `quick` |
| The normal complete pass | Full audit, profile `standard` |
| A high-stakes or large repository | Full audit, profile `deep` |

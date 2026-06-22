# Merge Readiness Report

Generated: 2026-06-22

## Open PRs

| # | Title | State | Draft | Mergeable | CI Checks | Review | Behind main | Blocker |
|---|-------|-------|-------|-----------|-----------|--------|-------------|---------|
| 159 | fix(frontend): remove duplicate exported AggregatedSession type alias | OPEN | no | MERGEABLE | none reported | required (0xSero) | 0 | needs approving review |
| 151 | feat(controller): add typed runtime failure reasons | OPEN | no | MERGEABLE | none reported | required (0xSero) | 0 | needs approving review |
| 152 | feat(controller): redact secrets in log API responses | OPEN | no | MERGEABLE | none reported | required (0xSero) | 0 | needs approving review |
| 157 | feat(frontend): add Readiness Matrix | OPEN | no | MERGEABLE | none reported | required (0xSero) | 0 | needs approving review |
| 160 | fix(controller): per-recipe crash-loop budget | OPEN | no | MERGEABLE | none reported | required (0xSero) | 0 | needs approving review |
| 161 | fix(controller): reject request-controlled runtime job commands | OPEN | no | MERGEABLE | none reported | required (0xSero) | 0 | needs approving review |
| 162 | fix(frontend): enforce registered agent filesystem roots | OPEN | no | MERGEABLE | none reported | required (0xSero) | 0 | needs approving review |

## Common Baseline Blockers

- **Review gate:** every PR has `mergeStateStatus: BLOCKED` and `reviewDecision: REVIEW_REQUIRED`; none can merge until 0xSero approves.
- **CI checks:** no GitHub Actions/status checks are reported on these branches. Verification in PR bodies was run locally.
- **`npm run check:contracts` baseline failure:** current `main` fails because `AggregatedSession` is exported from both `frontend/src/app/api/agent/sessions/all/route.ts` and `frontend/src/features/agent/session-contracts.ts`. PR #159 fixes this; after it merges, frontend PRs (#157, #162) can pass the contract check.
- **Controller integration baseline failure:** `observability-contracts.test.ts` expects `current_power_watts: 0`, but the workstation GPU reports a real draw. This is unrelated to any open PR.

## Recommended Merge Order

1. **#159** — contract export fix. Unblocks `check:contracts` for frontend PRs; no runtime impact.
2. **#151** — runtime failure taxonomy. Large controller change; touches `engine-coordinator.ts` and `engine-service.ts`.
3. **#152** — log redaction. Controller-only, independent of #151.
4. **#157** — readiness matrix. Frontend-only; depends on #159 for clean contract check.
5. **#160** — crash-loop budget. Controller-only; overlaps with #151 on `engine-coordinator.ts` and `engine-service.ts`, so expect a rebase/merge-conflict resolution after #151 lands.
6. **#161** / **#162** — private security hardening. Independent of each other; #161 is controller-only, #162 is frontend-only. Merge after the above to keep security fixes last in the batch.

## Per-PR Readiness

### #159 — fix(frontend): remove duplicate exported AggregatedSession type alias
- **Can merge now?** No — blocked only by missing review.
- **Exact blocker:** approving review required.
- **Dependencies:** none.
- **Verification after merge:** `npm run check:contracts`, `npm --prefix frontend run typecheck`, `npm --prefix frontend run lint`.

### #151 — feat(controller): add typed runtime failure reasons
- **Can merge now?** No — blocked only by missing review.
- **Exact blocker:** approving review required.
- **Dependencies:** soft dependency on #159 for a clean `check:contracts` run, but no code dependency.
- **Verification after merge:** `npm --prefix controller run typecheck`, `npm --prefix controller run lint`, `bun test tests/controller/integration/launch-failure-classifier.test.ts tests/controller/integration/observability-contracts.test.ts`.
- **Risk:** will likely require #160 to rebase because both touch `engine-coordinator.ts` / `engine-service.ts`.

### #152 — feat(controller): redact secrets in log API responses
- **Can merge now?** No — blocked only by missing review.
- **Exact blocker:** approving review required.
- **Dependencies:** none.
- **Verification after merge:** `npm --prefix controller run typecheck`, `npm --prefix controller run lint`, `bun test tests/controller/integration/log-redaction.test.ts`.

### #157 — feat(frontend): add Readiness Matrix
- **Can merge now?** No — blocked only by missing review.
- **Exact blocker:** approving review required.
- **Dependencies:** merge #159 first so `check:contracts` passes.
- **Verification after merge:** `npm --prefix frontend run typecheck`, `npm --prefix frontend run lint`, `npx tsx --test ../tests/frontend/e2e/readiness-matrix.test.ts`.

### #160 — fix(controller): per-recipe crash-loop budget
- **Can merge now?** No — blocked only by missing review.
- **Exact blocker:** approving review required.
- **Dependencies:** merge #151 first; expect rebase due to overlapping controller files.
- **Verification after merge:** `cd controller && bun run typecheck && bun run lint && bun test ../tests/controller/integration/engine-coordinator-crash-loop.test.ts`.

### #161 — fix(controller): reject request-controlled runtime job commands
- **Can merge now?** No — blocked only by missing review.
- **Exact blocker:** approving review required.
- **Dependencies:** merge #151/#160 first (route file overlap is small but safer after controller churn lands).
- **Verification after merge:** `cd controller && bun run typecheck && bun run lint && bun test ../tests/controller/integration/runtime-job-command-boundary.test.ts ../tests/controller/integration/runtime-upgrade-env-command.test.ts`.
- **Security impact:** removes arbitrary command/argv injection surface from runtime job APIs; operator-configured upgrade paths remain intact.

### #162 — fix(frontend): enforce registered agent filesystem roots
- **Can merge now?** No — blocked only by missing review.
- **Exact blocker:** approving review required.
- **Dependencies:** merge #159 first for clean contract check; otherwise independent.
- **Verification after merge:** `npm --prefix frontend run typecheck && npm --prefix frontend run lint && cd frontend && npx tsx --test ../tests/frontend/agent-fs-root-boundary.test.ts`.
- **Security impact:** restricts agent FS list/read to server-registered project roots and rejects traversal/symlink escapes.

## Commands Used for Verification

```bash
# PR status / mergeability / reviews
for pr in 159 151 152 157 160 161 162; do
  gh pr view $pr --repo sybil-solutions/vllm-studio \
    --json number,title,state,isDraft,mergeStateStatus,mergeable,reviewDecision,statusCheckRollup
  gh pr checks $pr --repo sybil-solutions/vllm-studio
  gh pr view $pr --repo sybil-solutions/vllm-studio --json reviewRequests,latestReviews
  head=$(gh pr view $pr --repo sybil-solutions/vllm-studio --json headRefName -q .headRefName)
  gh api repos/sybil-solutions/vllm-studio/compare/sybil-solutions:main...OnlyTerp:$head \
    --jq '[.status, .behind_by, .ahead_by] | @tsv'
done

# Changed files per PR
for pr in 159 151 152 157 160 161 162; do
  gh pr view $pr --repo sybil-solutions/vllm-studio --json files
done
```

## Notes

- Do not merge anything until 0xSero has reviewed and approved.
- After each merge, verify the next PR in the recommended order is still `MERGEABLE` before attempting it.

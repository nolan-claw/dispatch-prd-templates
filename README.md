# Dispatch PRD Templates

Structured PRD templates for the dotfiles-dispatch system. Write precise dispatch prompts that produce correct code on the first iteration.

## Why This Matters

A vague dispatch prompt ("RECONCILIATION") wastes hours as Ralph guesses what to do. A structured PRD with file paths, acceptance criteria, and verify commands ships in 1-2 iterations.

## PRD Schema

```json
{
  "project": "repo-name",
  "branchName": "feat/short-description",
  "description": "One paragraph: what exists, what to build, where the source reference lives",
  "userStories": [{
    "id": "XX-001",
    "title": "Short imperative title",
    "description": "WHAT to build, WHERE the reference code lives (full path), HOW to integrate",
    "acceptanceCriteria": [
      "Specific file exists and exports X",
      "pnpm build passes",
      "Existing tests still pass"
    ],
    "priority": 1
  }]
}
```

## Template: Feature Port

Use when porting a feature from a reference codebase (e.g., staging Python → monorepo TS).

```json
{
  "project": "matchpoint",
  "branchName": "feat/FEATURE-NAME-port",
  "description": "Port FEATURE from staging REFERENCE-CODEBASE into monorepo. The reference code is at /home/matchpoint/repositories/REFERENCE-PATH/src/. The target is apps/api/src/services/TARGET-PATH/. Reimplement in TypeScript — the reference code is behavioral spec only.",
  "userStories": [
    {
      "id": "FP-001",
      "title": "Port SERVICE-NAME from reference",
      "description": "Port SERVICE-NAME from /home/matchpoint/repositories/REFERENCE-PATH/src/SERVICE-FILE.py into apps/api/src/services/TARGET/service-name.service.ts. The service does BRIEF-DESCRIPTION. Use Zod schemas for input/output. Import types from @matchpoint/shared where they exist.",
      "acceptanceCriteria": [
        "service-name.service.ts exists and exports FUNCTION-NAMES",
        "Zod schemas for InputType and OutputType defined",
        "pnpm build passes from repo root",
        "Existing code imports and calls the new service"
      ],
      "priority": 1
    },
    {
      "id": "FP-002",
      "title": "Wire SERVICE-NAME into existing pipeline",
      "description": "Update apps/api/src/services/existing.service.ts to call the new service at the correct point in the pipeline. Add any needed imports and error handling.",
      "acceptanceCriteria": [
        "Existing service calls new service",
        "Error handling for service failures",
        "pnpm build && pnpm test pass"
      ],
      "priority": 2
    }
  ]
}
```

## Template: Bug Fix

Use for targeted fixes with clear reproduction steps.

```json
{
  "project": "matchpoint",
  "branchName": "fix/SHORT-DESCRIPTION",
  "description": "Fix BUG-DESCRIPTION. The bug occurs when STEPS-TO-REPRODUCE. Root cause is LIKELY-CAUSE.",
  "userStories": [
    {
      "id": "BF-001",
      "title": "Fix BUG-DESCRIPTION",
      "description": "In apps/api/src/routes/ROUTE-FILE.ts, the BUG-MANIFESTATION. Fix by CHANGES-NEEDED. The fix should handle EDGE-CASE.",
      "acceptanceCriteria": [
        "Specific behavior no longer occurs",
        "Edge case handled",
        "pnpm build && pnpm test pass",
        "No regression in related tests"
      ],
      "priority": 1
    }
  ]
}
```

## Template: New Package

Use when creating a new Turborepo package.

```json
{
  "project": "matchpoint",
  "branchName": "feat/PACKAGE-NAME-package",
  "description": "Create packages/PACKAGE-NAME/ as a new Turborepo package. PURPOSE. Add to turbo.json pipeline and pnpm-workspace.yaml.",
  "userStories": [
    {
      "id": "NP-001",
      "title": "Scaffold PACKAGE-NAME package",
      "description": "Create packages/PACKAGE-NAME/ with: package.json (name: @matchpoint/PACKAGE-NAME), tsconfig.json, src/index.ts (exports), SKILL.md (docs). Add to turbo.json build pipeline. Add to pnpm-workspace.yaml.",
      "acceptanceCriteria": [
        "Package directory exists with all config files",
        "turbo.json includes @matchpoint/PACKAGE-NAME in pipeline",
        "pnpm build passes from repo root",
        "Package is importable by apps/*"
      ],
      "priority": 1
    },
    {
      "id": "NP-002",
      "title": "Implement core PACKAGE-NAME logic",
      "description": "Implement the core functionality in packages/PACKAGE-NAME/src/. DETAILS. Export from index.ts.",
      "acceptanceCriteria": [
        "Core functions exported and tested",
        "Zod schemas for all public interfaces",
        "pnpm build && pnpm test pass"
      ],
      "priority": 2
    }
  ]
}
```

## Dispatching the PRD

```bash
# 1. Write PRD to dev user filesystem (NOT matchpoint user)
ssh dev@178.156.171.0 "cat > ~/tmp/prd.json << 'EOF'
YOUR_PRD_HERE
EOF"

# 2. Dispatch with PRD mode
ssh dev@178.156.171.0 /home/dev/dotfiles/bin/.local/bin/dotfiles-dispatch \
  matchpoint matchpoint --prd ~/tmp/prd.json --max 30

# 3. Monitor
ssh matchpoint@178.156.171.0 "tail -f ~/ralph-state/TASK-ID.log"
ssh dev@178.156.171.0 dotfiles-dispatch-status
```

## Pitfalls

- **PRD file must be on dev user filesystem**, not matchpoint. The script validates the file locally before SCPing.
- **Base branch defaults to wave0-foundation**. Always include `branchName` in PRD and verify PR targets `main`.
- **Verification runs on main checkout**, not the worktree. Missing node_modules causes false build failures. Ralph handles this by running `pnpm install` first.
- **Bash variable expansion mangles multiline strings** through SSH. Use heredoc (`<< 'EOF'`) with single-quoted delimiter to prevent expansion.
- **One story per concern**. Don't lump unrelated changes into one story. Ralph processes them sequentially.
- **2-4 stories per PRD is optimal**. More than 4 stories and Ralph starts hitting iteration limits.

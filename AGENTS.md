## Harness Engineering Rules

This repository uses Harness Engineering as the default delivery model for all future implementation work.

### Operating Model

- Design every change as the smallest reviewable delivery unit.
- Before substantial edits, state the entry points, affected audiences, rollout stage, validation gate, and rollback path.
- Prefer root-cause fixes over surface patches, but keep the blast radius narrow.
- Do not mix unrelated cleanup into documentation, template, or structure changes.
- Distinguish three outcomes in status reporting: implemented, verified, and residual risk.

### Release Safety For Documentation

- Treat documentation as an operational asset, not passive text.
- For changes that influence implementation, release, operations, or architecture decisions, call out which teams or repositories are affected.
- Prefer staged documentation updates for broad restructures: index first, body changes second, cross-links third.
- If a doc change revises behavior, requirements, or contracts, identify the impacted source-of-truth files and repositories.
- Keep rollback simple: revert the document delta and restore previous navigation or templates.

### Verification As A Gate

- Verification is mandatory. Each documentation change must have an explicit success signal.
- Required signals include successful site build, valid navigation, intact relative links, and consistency with referenced project behavior.
- If a document changes process, release, or API guidance, verify that examples and commands still match the current repositories.
- If a statement cannot be verified, mark it as an assumption or open item instead of presenting it as settled.

### Observability, Governance, And Source Of Truth

- Follow the Harness verification mindset: docs that affect delivery should make correctness inspectable.
- Prefer explicit checklists, examples, compatibility notes, and rollback instructions over vague prose.
- Treat templates, release checklists, operations runbooks, and architecture docs as governed assets.
- Keep structure declarative and reviewable. Prefer stable folder placement and predictable naming over ad hoc sprawl.

### Dependency And Supply Chain Rules

- Avoid introducing new doc tooling unless the current VitePress stack is insufficient.
- Any new dependency, plugin, or build step must be justified with source, purpose, risk, and minimum necessity.
- Preserve reproducibility and simple local preview workflows.

### Documentation-Specific Rules

- Every behavior-changing doc update should state scope, affected audience, validation method, and rollback path.
- For API, architecture, or operations docs, include compatibility constraints and known assumptions.
- When requirements or interfaces change, update the nearest index or navigation entry in the same delivery unit.
- Prefer concise, decision-oriented writing: what changed, why it matters, how to verify, how to reverse.

### Required Delivery Templates

- Requirement or product doc: scope -> affected users -> changed behavior -> dependent systems -> verification -> rollback.
- Architecture or platform doc: scope -> boundaries -> dependencies -> failure modes -> verification -> rollback.
- API doc: scope -> compatibility -> caller impact -> examples -> verification -> rollback.
- Operations doc: scope -> trigger conditions -> execution steps -> health signals -> rollback.

### Repository Verification Gates

- Minimum gate for documentation changes: `npm run docs:build`.
- For navigation, theme, or template changes, also confirm that the resulting site structure remains coherent.
- If build verification is blocked by unrelated issues, state the blocker and residual risk explicitly.

### Source Of Truth

- Primary reference: <https://developer.harness.io/docs>
- GitHub source reference: <https://github.com/harness/developer-hub>
- Most relevant Harness areas for this repository: Continuous Delivery verification, Feature Flags, Infrastructure as Code Management, Software Supply Chain Security, and documentation as a governed delivery artifact.
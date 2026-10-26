---
description: Enforce Planning Mode on all user requests before making changes or executing tasks.
---

# Always Plan First Rule

1. **Mandatory Planning Mode**:
   - Treat every user request, task, or workflow invocation as requiring a **Plan-First** approach.
   - Do NOT modify, create, or delete project code/files (outside of the planning artifact itself) without first presenting a detailed Implementation Plan artifact and obtaining explicit user approval.
   
2. **Implementation Plan Requirements**:
   - Write the plan artifact to `<Artifact Directory>/<plan_name>.md`.
   - Set `ArtifactMetadata.RequestFeedback = true` and `ArtifactMetadata.UserFacing = true`.
   - Provide a complete breakdown: Goal Description, User Review Required, Open Questions, Proposed Changes (with exact files demarcated), and Verification Plan.
   - Stop and wait for user review/approval before proceeding with execution.

3. **Exception**:
   - Pure informational queries, explanations, or questions that do not modify workspace state or execute tasks do not require an implementation plan unless specifically requested by the user.

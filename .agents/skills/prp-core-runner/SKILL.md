---
name: prp-core-runner
description: Orchestrate complete PRP workflow from feature request to pull request. Runs plan creation, implementation, commit, and PR in sequence. Use when implementing features using PRP methodology or when user requests full PRP workflow end-to-end.
---

# PRP Core Workflow Runner

## Instructions

When the user requests to implement a feature using the PRP workflow or wants end-to-end automation from idea to PR, execute the complete workflow by invoking the individual skills in sequence.

**Step-by-step execution:**

1. **Create the plan**: Use the `$prp-plan` skill with the feature description to generate a comprehensive implementation plan
2. **Execute the plan**: Use the `$prp-implement` skill with the generated plan file path to implement the changes
3. **Commit changes**: Use the `$prp-commit` skill to create an atomic git commit with a clear message
4. **Create pull request**: Use the `$prp-pr` skill to push and open a pull request

**Monitor progress**: Execute each step sequentially. Do NOT proceed to the next step if the current step fails.

**Handle failures**: If any step fails:
- Report which step failed and why
- Do NOT proceed to subsequent steps
- Provide actionable guidance for fixing the issue

**Report completion**: When all steps succeed, confirm the workflow completed and provide the PR URL.

**Error Handling:**

- Stop execution immediately if any validation fails
- Report the specific error clearly
- Guide the user on how to resolve the issue
- Do not attempt to auto-fix complex validation failures

## Workflow Detail

### Step 1: Plan

**Use the `$prp-plan` skill with the following prompt:**

```
{feature description from the user's request}
```

Wait for the plan to be created. Note the plan file path (e.g., `.claude/PRPs/plans/{feature-name}.plan.md`).

### Step 2: Implement

**Use the `$prp-implement` skill with the following prompt:**

```
Execute the plan at: {plan-file-path from step 1}
```

Wait for implementation to complete. Verify all validation checks pass.

### Step 3: Commit

**Use the `$prp-commit` skill** to stage and commit the changes with a descriptive message referencing the feature.

### Step 4: Pull Request

**Use the `$prp-pr` skill** to push the branch and create a pull request with a comprehensive description.

## Examples

**Example 1: Autonomous invocation**
```
User: "Can you implement user authentication using JWT with the PRP workflow?"
Assistant: I'll run the complete PRP workflow for implementing JWT authentication.
[Invokes $prp-plan → $prp-implement → $prp-commit → $prp-pr in sequence]
```

**Example 2: Feature request**
```
User: "I need to add a search API with Elasticsearch integration using PRP"
Assistant: I'll run the full PRP workflow to implement the search API with Elasticsearch.
[Invokes $prp-plan → $prp-implement → $prp-commit → $prp-pr in sequence]
```

**Example 3: Refactoring with PRP**
```
User: "Use the PRP methodology to refactor the database layer for better performance"
Assistant: I'll execute the PRP workflow for refactoring the database layer.
[Invokes $prp-plan → $prp-implement → $prp-commit → $prp-pr in sequence]
```

## When to Use

Use this skill when:
- User explicitly requests to "implement a feature using PRP"
- User asks to "run the full PRP workflow"
- User wants end-to-end automation from feature idea to pull request
- User mentions both "PRP" and a feature to implement
- User requests a complete workflow including branch, implementation, and PR

Do NOT use this skill when:
- User only wants to run a single PRP step (e.g., just create a plan)
- User is asking about PRP methodology (provide information instead)
- User wants to implement something without mentioning PRP workflow

# Command: Generate PRP (Product Requirements Prompt) 2.0

## Feature File

$ARGUMENTS

## Role

You are a senior project manager AI. Your goal is to create a comprehensive Product Requirements Prompt (PRP) by orchestrating a team of specialized sub-agents. You will delegate tasks, synthesize their findings, and produce a final, actionable implementation plan for a developer agent.

## Orchestration Process

1. **Delegate to Code Analyst**:

   - Invoke the `code-analyst` agent to perform an in-depth analysis of the existing codebase.
   - Provide it with the feature request file (`$ARGUMENTS`).
   - **Agent Command:**

     ```
     /agent ./.claude/agents/code-analyst.md "$ARGUMENTS"
     ```

   - Capture the output as `code_analysis_report`.

2. **Delegate to Web Researcher**:

   - Invoke the `web-researcher` agent to gather external context, documentation, and best practices.
   - Provide it with the feature request file (`$ARGUMENTS`).
   - **Agent Command:**

     ```
     /agent ./.claude/agents/web-researcher.md "$ARGUMENTS"
     ```

   - Capture the output as `web_research_report`.

3. **Synthesize and Generate PRP**:
   - Read the base PRP template from `PRPs/templates/prp_base.md`.
   - Carefully review the `code_analysis_report` and the `web_research_report`.
   - ULTRATHINK: Develop a comprehensive implementation blueprint by integrating the insights from both reports. Your plan should be clear, step-by-step, and include executable validation gates.
   - Populate the PRP template with the synthesized information, creating a complete and self-contained guide for the developer agent.

## Output

- **Save the final PRP** to a new file in the `PRPs/` directory. The filename should be based on the feature request (e.g., `PRPs/new-feature-name.md`).
- **Announce completion** and provide the path to the newly created PRP file.

## Quality Checklist

- [ ] Have both the `code-analyst` and `web-researcher` agents been called successfully?

* [ ] Are the key findings from both reports integrated into the final PRP?
* [ ] Is the implementation plan clear, logical, and broken down into actionable steps?
* [ ] Are the validation gates executable and relevant to the feature?
* [ ] Score the final PRP on a confidence level (1-10) for a successful one-pass implementation.

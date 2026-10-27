---
name: enrich-us
description: Analyze and enhance user stories with complete, implementation-ready technical detail from direct ticket input or Azure Devops.
author: LIDR.co
version: 1.0.0
---
# enrich-us Skill

Use it when this workflow is required in the project.

## Instructions

Please analyze and enrich the ticket: $ARGUMENTS.

Follow these steps:

1. Determine the ticket input source:
   - **Direct input mode (default when ticket text is provided):** Use the ticket content shared by the user in the prompt/chat.
   - **Azure Devops mode (optional):** If the user provides a Azure Devops id/key, or asks to use Azure Devops (including references like "the one in progress"), use Azure Devops MCP to fetch the ticket details.
2. Act as a product expert with technical knowledge.
3. Understand the problem described in the ticket.
4. Decide whether or not the User Story is completely detailed according to product best practices. Validate that it includes:
   - Full functionality description
   - Comprehensive list of fields to update
   - Required endpoints structure and URLs
   - Files/modules to modify according to architecture and best practices
   - Definition of done (implementation and delivery steps)
   - Documentation and unit test updates
   - Non-functional requirements (security, performance, observability, etc.)
5. If the story lacks enough technical detail for autonomous implementation, provide an improved version that is clearer, more specific, and concise, aligned with step 4. Use project technical context from `@documentation`. Return the result in markdown.
6. Output format must always include:
   - `## Original`
   - `## Enhanced`
7. Jira write-back is optional and only applies in Jira mode:
   - Update the Jira ticket by appending the enhanced content after the original content, with clear `h2` sections `[original]` and `[enhanced]` and readable formatting (lists/code snippets when useful).
   - If ticket status is `To refine`, move it to `Pending refinement validation`.

## Notes

- Do not require Jira when the user already provided full ticket content directly.
- If input is ambiguous (for example, user gives a short reference without content), ask whether to resolve via Jira or request the full ticket text.

## OpenSpec Traceability

When the enhanced user story corresponds to an OpenSpec change:

1. Identify the associated OpenSpec change. If its name was not specified, derive one in `kebab-case` from the story’s goal.
2. Save the complete story to `openspec/changes/<change-name>/historia.md`.
3. The file must include the `## Original` and `## Enhanced` sections, containing the input text and the enhanced user story.
4. `historia.md` is an additional traceability artifact; it does not replace or modify `proposal.md`, `spec.md`, `design.md`, or `tasks.md`.
5. Do not generate the other OpenSpec artifacts from this skill. Their creation belongs to the `openspec-propose` workflow.
6. Verify that `historia.md` exists in the correct change directory before reporting that enrichment is complete. If the file already exists, preserve it and update it with the current story.
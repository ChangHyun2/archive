How rules work
Large language models don't retain memory between completions. Rules provide persistent, reusable context at the prompt level.

When applied, rule contents are included at the start of the model context. This gives the AI consistent guidance for generating code, interpreting edits, or helping with workflows.

Project rules
Project rules live in .cursor/rules as markdown files and are version-controlled. They are scoped using path patterns, invoked manually, or included based on relevance.

Use project rules to:

Encode domain-specific knowledge about your codebase
Automate project-specific workflows or templates
Standardize style or architecture decisions
Rule file structure
Each rule is a markdown file that you can name anything you want. Cursor supports .md and .mdc extensions. Use .mdc files with frontmatter to specify description and globs for more control over when rules are applied.


.cursor/rules/
  react-patterns.mdc       # Rule with frontmatter (description, globs)
  api-guidelines.md        # Simple markdown rule
  frontend/                # Organize rules in folders
    components.md


Rule anatomy
Each rule is a markdown file with frontmatter metadata and content. Control how rules are applied from the type dropdown which changes properties description, globs, alwaysApply.

Rule Type	Description
Always Apply	Apply to every chat session
Apply Intelligently	When Agent decides it's relevant based on description
Apply to Specific Files	When file matches a specified pattern
Apply Manually	When @-mentioned in chat (e.g., @my-rule)

---
globs:
alwaysApply: false
---
- Use our internal RPC pattern when defining services
- Always use snake_case for service names.
@service-template.ts


Best practices
Good rules are focused, actionable, and scoped.

Keep rules under 500 lines
Split large rules into multiple, composable rules
Provide concrete examples or referenced files
Avoid vague guidance. Write rules like clear internal docs
Reuse rules when repeating prompts in chat
Reference files instead of copying their contents—this keeps rules short and prevents them from becoming stale as code changes
What to avoid in rules
Copying entire style guides: Use a linter instead. Agent already knows common style conventions.
Documenting every possible command: Agent knows common tools like npm, git, and pytest.
Adding instructions for edge cases that rarely apply: Keep rules focused on patterns you use frequently.
Duplicating what's already in your codebase: Point to canonical examples instead of copying code.
Start simple. Add rules only when you notice Agent making the same mistake repeatedly. Don't over-optimize before you understand your patterns.

Check your rules into git so your whole team benefits. When you see Agent make a mistake, update the rule. You can even tag @cursor on a GitHub issue or PR to have Agent update the rule for you.

Rule file format
Each rule is a markdown file with frontmatter metadata and content. The frontmatter metadata is used to control how the rule is applied. The content is the rule itself.


---
description: "This rule provides standards for frontend components and API validation"
alwaysApply: false
---
...rest of the rule content
If alwaysApply is true, the rule will be applied to every chat session. Otherwise, the description of the rule will be presented to the Cursor Agent to decide if it should be applied.

Examples

Standards for frontend components and API validation
This rule provides standards for frontend components:

When working in components directory:

Always use Tailwind for styling
Use Framer Motion for animations
Follow component naming conventions
This rule enforces validation for API endpoints:

In API directory:

Use zod for all validation
Define return types with zod schemas
Export types generated from schemas


Templates for Express services and React components
This rule provides a template for Express services:

Use this template when creating Express service:

Follow RESTful principles
Include error handling middleware
Set up proper logging
@express-service-template.ts

This rule defines React component structure:

React components should follow this layout:

Props interface at top
Component as named export
Styles at bottom
@component-template.tsx

Automating development workflows and documentation generation
This rule automates app analysis:

When asked to analyze the app:

Run dev server with npm run dev
Fetch logs from console
Suggest performance improvements
This rule helps generate documentation:

Help draft documentation by:

Extracting code comments
Analyzing README.md
Generating markdown documentation

Adding a new setting in Cursor
First create a property to toggle in @reactiveStorageTypes.ts.

Add default value in INIT_APPLICATION_USER_PERSISTENT_STORAGE in @reactiveStorageService.tsx.

For beta features, add toggle in @settingsBetaTab.tsx, otherwise add in @settingsGeneralTab.tsx. Toggles can be added as <SettingsSubSection> for general checkboxes. Look at the rest of the file for examples.


<SettingsSubSection
  label="Your feature name"
  description="Your feature description"
  value={
    vsContext.reactiveStorageService.applicationUserPersistentStorage
      .myNewProperty ?? false
  }
  onChange={(newVal) => {
    vsContext.reactiveStorageService.setApplicationUserPersistentStorage(
      "myNewProperty",
      newVal,
    );
  }}
/>
To use in the app, import reactiveStorageService and use the property:


const flagIsEnabled =
  vsContext.reactiveStorageService.applicationUserPersistentStorage
    .myNewProperty;

Examples are available from providers and frameworks. Community-contributed rules are found across crowdsourced collections and repositories online.
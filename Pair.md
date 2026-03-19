# playwright-component-coverage-agent-instructions.md

## Operating Principles
Always verify before concluding.  
Coverage claims must be tied to source evidence, config, test files, or executable output.

## Discovery Checklist
- Read `package.json`
- Read Playwright config
- Read component test config
- Read CI/test scripts if relevant
- Locate shared mount/test utilities
- Locate component folders
- Locate existing Playwright component tests
- Locate stories when useful for state discovery

## Component Inventory Rules
Treat as candidate components:
- exported React UI components
- feature components
- shared/ui components
- form components
- stateful widgets

Usually exclude or separately classify:
- type-only files
- index re-export files
- style-only files
- generated code
- deprecated components

## Coverage Evaluation Heuristics
A component is better covered when tests validate:
- user-visible rendering
- meaningful states
- interactions
- callbacks
- error/empty/loading branches
- provider-dependent behavior where relevant

A component is weakly covered when tests only:
- mount it
- assert one static label
- use snapshots without behavioral checks

## Priority Heuristics
Prioritize missing coverage for:
- business-critical components
- forms
- async/stateful components
- reusable shared components
- defect-prone or complex areas

## Reporting Rules
Always distinguish:
- exact evidence
- reasonable inference
- unknown or missing information

Never claim exact percentages unless coverage tooling provides them directly.

## Test Generation Guidance
When asked to generate tests:
- use existing repo conventions
- reuse shared helpers
- prefer accessible selectors
- keep tests deterministic and focused

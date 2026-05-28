---
name: playwright-generate
description: Generate, debug, and maintain Playwright end-to-end tests using the target repository's fixtures, helpers, page objects, setup conventions, and E2E command
---

Generate and maintain Playwright E2E tests for the target repository.

Do not assume a specific framework, folder structure, package manager, fixture layout, data setup method, or test command.

## Repository discovery

Before writing or editing tests, discover the repository's conventions:

- Read testing documentation if present, such as `docs/testing/e2e-context.md`, `docs/testing.md`, `CONTRIBUTING.md`, `README.md`, or files near the E2E tests.
- Inspect `package.json` scripts for E2E commands such as `test:e2e`, `e2e`, `playwright`, or project-specific wrappers.
- Inspect Playwright config files such as `playwright.config.ts`, `playwright.config.js`, or `playwright.config.mjs`.
- Identify E2E folders such as `e2e/`, `tests/e2e/`, `__e2e__/`, `playwright/`, or folders referenced by Playwright config.
- Inspect nearby specs in the same feature area.
- Inspect existing fixtures, helpers, page objects, factories, API clients, seed utilities, and authentication/storage setup.

Repository conventions are mandatory and take precedence over generic Playwright patterns or generated recorder-style code.

Before writing or editing tests:

1. Read the repository's testing documentation, if present.
2. Treat mandatory rules from that documentation as required implementation constraints.
3. Inspect the closest existing spec for the same feature area.
4. Inspect the relevant fixture, helper, page object, and setup files.
5. Reuse existing fixtures, helpers, setup utilities, and page object methods before introducing new patterns.

Core rules:

- Use repository fixtures instead of `@playwright/test` directly when applicable.
- Use the repository's existing setup utilities for entities, users, permissions, authentication, storage state, and feature flags.
- Create setup data through fixtures, API helpers, factories, seed utilities, GraphQL helpers, or shared utilities already used by the repository.
- Do not create data through the UI unless that creation flow is the behavior under test.
- Route UI interactions through page objects or helper abstractions when the repository uses them.
- Do not put reusable locators or reusable interaction logic directly in specs when the repository has an abstraction pattern for them.
- Extend existing page objects or helpers when possible.
- Create a new focused page object or helper only when the functionality represents a distinct reusable UI area or flow.
- Keep assertions visible in the spec body unless they are repeated low-level checks.
- Match naming, imports, structure, and style from nearby tests.

## Playwright CLI Skill Usage

When browser interaction, debugging, tracing, selector inspection, session management, or lightweight validation is needed, use the installed Playwright CLI skills and their reference guides.

Prefer Playwright CLI capabilities for:

- test generation drafts
- selector validation
- browser interaction
- tracing
- storage state inspection
- request mocking
- video recording
- session management
- lightweight flow validation
- debugging failing interactions

Prefer repository fixtures, page objects, helpers, and testing documentation for:

- final test architecture
- test organization
- setup strategy
- reusable interactions
- assertion placement
- long-term maintainability

Use `playwright-cli` only for exploration and debugging:

- discovering current UI behavior
- validating selectors
- checking screenshots or snapshots
- debugging failing flows

`playwright-cli` output is exploration material only.

Do not paste raw generated recorder-style code into specs. Refactor generated interactions into fixtures, helpers, and page object methods that follow repository conventions.

## Validation Rules

Use the lightest validation strategy possible.

Preferred validation order:

1. Typecheck or lint changed files if available.
2. Use Playwright CLI for fast interaction and selector validation.
3. Run the smallest relevant Playwright test scope in headless mode using the repository's E2E command.
4. Use headed mode only for explicit debugging.

Always avoid:

- unnecessary `--headed`
- full-suite execution
- validating unrelated specs

Before finishing, verify:

- repository testing documentation was followed, if present
- the closest existing spec was inspected
- the appropriate fixture, helper, or setup utility was used
- setup data uses repository-standard fixtures, APIs, factories, seed utilities, or helpers
- page object methods or equivalent repository abstractions are used or added where appropriate
- raw locators are kept out of specs where possible
- new interactions are reusable by future tests
- assertions remain visible in the spec body
- the generated code matches nearby repository examples
- the smallest relevant E2E command was run or clearly stated if not run

If generated code conflicts with repository testing documentation or nearby examples, revise the implementation before returning the result.

Use Playwright MCP only if `playwright-cli` is insufficient.

Use `output-template.md` for response formatting.

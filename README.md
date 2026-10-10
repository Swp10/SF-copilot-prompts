# Copilot Agents for Test Automation

AI agent prompts for GitHub Copilot that generate and audit tests for React microapps — Playwright component tests, coverage gap analysis, Cucumber acceptance tests with WireMock, and accessibility reviews.

Built and used on enterprise React microapps:

- **400+ Playwright component tests** generated across micro-apps
- **Acceptance test authoring cut from ~2 weeks to ~2 hours** by turning Jira acceptance criteria into Cucumber/Gherkin scenarios

## Why these agents are different

Most AI test generators are optimized to produce *more* tests. These are optimized to produce tests you can *trust*:

- **Evidence over assumption** — coverage findings must be tied to real files, config, or tool output. No invented coverage, no made-up percentages.
- **Behavior over implementation** — tests assert what users see and do (roles, labels, states), not internal state or brittle selectors.
- **Reuse over reinvention** — the acceptance agent maps scenarios to your team's *existing* shared step library and never creates new step definitions, so vocabulary stays consistent across teams.
- **Honest output** — every agent reports assumptions, uncertainty, and gaps alongside what it generated.

## The agents

| Agent | What it does | Use it when |
|---|---|---|
| [Playwright Component Test Generator](prompt.md) | Scans a microapp, understands component props/providers/mocks, and writes deterministic Playwright component tests that match local conventions | You need tests for a new or untested component |
| [Playwright Component Coverage Agent](Playwright_Coverage.md) | Inventories components and tests, maps them, classifies coverage (well / partial / minimal / indirect / uncovered / excluded) and ranks the highest-risk gaps | You want to know what's *actually* covered before writing more tests |
| [Coverage Agent Instructions](Pair.md) | Companion operating rules for the coverage agent (discovery checklist, heuristics, reporting rules) | Load alongside the coverage agent |
| [Cucumber Acceptance Test Agent](acceptance_test-agent.md) | Generates `.feature` files and WireMock stubs from acceptance criteria, reusing only shared step definitions, with a governance pass for duplicate and inconsistent steps | Turning Jira acceptance criteria into BDD scenarios |
| [Cucumber Acceptance Test Agent — Lite](acceptance_test_light-agent.md) | Shorter version of the above | Smaller models or tighter context windows |
| [WireMock + Cucumber Patterns](wiremock_agent.md) | Reference templates for happy path, not-found, loading, data-driven and error-boundary scenarios with WireMock stubs | Starting a new acceptance suite |
| [Accessibility Evaluation](ACCESSIBILITY%20EVALUATION) | Heuristic WCAG 2.2 AA review that produces a standalone HTML report for stakeholders | Early accessibility review of a page or flow |
| [Accessibility Audit — Quick](ADA_Audit) | One-shot prompt listing likely WCAG 2.2 AA issues and how to test each | Quick checklist before a manual a11y pass |

Also included: [Accelerating Testing with Copilot Agent](accelerating%20testing%20wiki) — a leadership overview of agent-driven test generation and how to pilot it.

## How to use

1. Open your microapp in VS Code with GitHub Copilot Chat.
2. Switch Copilot Chat to **Agent** mode.
3. Paste the agent prompt, or save it as a custom agent / prompt file in your repo.
4. Give it a task, for example:
   - `Generate Playwright component tests for src/components/LoginForm.tsx`
   - `Check Playwright component coverage for this microapp and list the top 5 gaps.`
   - `Create acceptance tests for this story: <paste acceptance criteria>.`
5. Review the output — each agent ends with assumptions and gaps to check before you commit.

**Before using the acceptance agents**, replace `<< INSERT SHARED STEP DEFINITIONS PATH HERE >>` with the path to your shared step library.

**Accessibility prompts are heuristic.** Pair them with automated scanning (for example [`@axe-core/playwright`](https://github.com/dequelabs/axe-core-npm)) and manual testing with assistive technology. They are not a substitute for a formal audit.

## Requirements

- GitHub Copilot with Agent mode in VS Code (Business or Enterprise plan for org repos)
- Playwright with component testing (`@playwright/experimental-ct-react`) for the Playwright agents
- Cucumber and WireMock for the acceptance agents

## Roadmap

- [ ] Move agents into `.github/agents/` and `.github/prompts/` so Copilot loads them automatically
- [ ] Merge overlapping prompts into clear full / lite pairs
- [ ] Add an `examples/` folder with a sample component, generated tests, and CI
- [ ] Add a CHANGELOG

## License

[MIT](LICENSE) — free to use and adapt. Feedback and pull requests welcome.

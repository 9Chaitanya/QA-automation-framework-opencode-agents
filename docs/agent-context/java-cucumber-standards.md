# Java + Cucumber + Playwright — Canonical Standards
This file is the BASELINE. Repo conventions (framework-conventions.md) override it ONLY where the repo has a clearly established pattern. Where the repo is silent, these rules apply.

This file must never be read as license to use an API surface newer than what is actually installed. Confirmed dependency versions in framework-conventions.md always win over any API/syntax implied here — see "Version compatibility" below.

## Gherkin / BDD
- Declarative business language. Never mention clicks, selectors, URLs, or waits in steps.
- Given = state, When = action, Then = observable outcome. Max one When→Then pair of intent per scenario.
- One behavior per scenario; scenarios are independent and order-agnostic.
- Scenario Outline + Examples for data variants; no copy-pasted scenarios.
- Background only for genuinely shared Givens (max 3-4 lines).
- Tags: @<XRAY-KEY> on every scenario, plus suite tags (@smoke/@regression) per repo convention.

## Step definitions (thin glue)
- Steps contain NO logic: no if/else, no loops, no try/catch swallowing. They delegate to page objects and assert.
- Assertions live in Then steps (or dedicated assertion helpers) — NEVER inside page objects.
- Share state between steps via a ScenarioContext/TestContext class injected with picocontainer (or repo's DI). NEVER static fields.
- Cucumber expressions over regex ({string}, {int}, custom @ParameterType for domain types).
- One step-definition class per functional area, not per feature file. Reuse before creating.
- No step calls another step.

## Page Object Model
- One class per page; one class per reusable widget/component (header, modal, grid) composed into pages.
- Constructor takes Playwright Page (plus context object if repo uses one). Extend BasePage for shared utilities.
- Locators: private final Locator fields initialized in the constructor (Playwright Locators are lazy — safe to pre-declare). Selector priority: getByRole(name) > getByLabel > getByTestId > getByPlaceholder > css#id > scoped css. Do NOT use XPath — this priority order is a hard ceiling, not a suggestion; if no selector in this list is available for an element, that element must be re-examined during UI exploration (e.g. via accessibility snapshot or vision-reader) rather than falling back to XPath.
- Public methods express user intent (login(user, pass), searchFor(term)) — not raw UI verbs (clickButton, typeText).
- Navigation methods return the next page object (fluent): `LoginPage.login(...)` returns `DashboardPage`.
- NO assertions in page objects. Pages may expose state getters (isErrorShown(), getToastText()) for Then steps to assert on.
- No sleeps, no manual waitForTimeout. Rely on Playwright auto-wait; for sync points use locator.waitFor() or expect-style assertions in steps.

## Playwright usage
- Assertions: PlaywrightAssertions.assertThat(locator) web-first assertions (auto-retrying), or AssertJ for non-UI values.
- One BrowserContext + Page per scenario (isolation); Browser shared per thread/run.
- Thread-safety for parallel runs: Playwright objects held in ThreadLocal or a DI-scoped manager — never static singletons.
- Trace/screenshot/video on failure only; attach to the Cucumber/Allure report in an @After hook. (Note: this is failure-artifact capture by the test run itself, a separate concern from how ui-explorer investigates the app pre-authoring — exploration uses accessibility snapshots as its default, not screenshots; see ui-explorer agent.)
- Base URL, browser, headless, timeouts come from config (properties/env), never hardcoded.
- Browser binary: tests must run against the system-installed Chrome (e.g. via Playwright's `channel` config set to chrome), not Playwright's bundled/auto-installed Chromium. Do not add or rely on a `playwright install` driver-download step in build/CI config under this baseline.

## Hooks & lifecycle
- @Before: create context+page (and tracing if enabled). @After: capture artifacts on failure, then close context. Order: artifact capture hook runs before teardown hook (@After(order)).
- Conditional hooks via tags (@Before("@api") etc.) instead of if-checks inside hooks.
- No business logic in hooks.

## Test data & config
- Test data separated from code: Examples tables for small variants; properties/JSON/builders for larger fixtures. No magic strings scattered in steps.
- Secrets via environment variables only — never committed.
- Unique data per run where the app requires it (timestamps/UUID suffix) to keep scenarios re-runnable.

## Version compatibility
- This baseline describes patterns, not pinned API versions. Whatever Cucumber/Playwright/Java/build-tool versions are confirmed in framework-conventions.md are authoritative for actual syntax (annotations, method signatures, imports) — if a pattern above implies an API shape that doesn't exist in the confirmed installed version, the confirmed version wins and the pattern must be adapted, not the other way around.
- Do not write code against an assumed "latest" version of any dependency. If framework-conventions.md has no confirmed version for a dependency you need, that is a blocker, not a guessing opportunity.

## Project hygiene
- Runner configured for parallel execution (cucumber.execution.parallel.enabled) with tag filtering via system property.
- cucumber.properties: snippet-type=camelcase, publish disabled, plugin for pretty + html/json (+ Allure if present).
- Generated code must compile with zero warnings under the repo's Java version; follow repo formatting (otherwise Google Java Style).
- Imports must be verified against the confirmed dependency versions before code is considered complete — see code-writer agent's mandatory import-verification step. Do not leave unverified or wildcard imports unless house style explicitly permits wildcards.

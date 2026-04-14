# WireMock Cucumber Copilot Agent

## Overview

A copilot agent that generates Cucumber feature files with WireMock HTTP stubs for acceptance testing a React microapp. Given a scenario description or endpoint, the agent produces ready-to-use `.feature` files, Java step definitions, and WireMock JSON mappings.

**Stack:** React microapp · Cucumber-JVM · WireMock · Selenium / Playwright

---

## Capabilities

| Action | Description |
|---|---|
| Generate feature file | Produces a `.feature` file for a given HTTP endpoint and scenario type |
| Generate step definitions | Outputs Java `@Given/@When/@Then` wiring WireMock `stubFor()` and `verify()` |
| Generate WireMock JSON stub | Produces standalone `mappings/` + `__files/` JSON for the WireMock file server |
| Scaffold scenario outline | Creates a data-driven `Scenario Outline` with an `Examples:` table |
| Stub error responses | Generates 4xx / 5xx fault stubs with React error boundary assertions |
| Stub delayed responses | Creates fixed-delay stubs to test loading skeletons and timeouts |

---

## System Prompt

```
You are a senior BDD practitioner and test automation expert. Your job is to
write living-documentation-style Cucumber feature files for a React microapp,
backed by WireMock HTTP stubs wired up invisibly in the step definitions.

Feature files must read as business behaviour, not technical implementation.
WireMock, HTTP status codes, JSON payloads, and API paths must NEVER appear
in the Gherkin — they belong exclusively in the Java step definitions.

When given a user story, feature area, or scenario description:
1. Write a .feature file using the role-goal-benefit format for the Feature
   narrative (As a / I want / So that).
2. Express all Given/When/Then steps in plain business language from the
   user's perspective (e.g. "the product catalogue is available",
   not "WireMock stubs GET /api/products to return 200").
3. Use a Background for shared user context (e.g. "Given I am a logged-in
   shopper on the products page").
4. Write Then steps as observable UI outcomes (e.g. "Then I should see 2
   products listed"), never as API assertions.
5. In the step definitions, map each business step to WireMock stubFor()
   calls and UI assertions — keeping all technical detail there.
6. If asked for a standalone stub, output valid WireMock JSON for
   mappings/ and __files/.

Always follow these conventions:
- Tags: @acceptance plus a domain tag (e.g. @products, @cart)
- Scenarios named from the user's perspective ("I can browse products")
- No HTTP verbs, status codes, JSON, or endpoint paths in the Gherkin
- Keep scenarios independent — step def @Before/@After resets WireMock state
```

---

## Templates

### Happy Path

```gherkin
@acceptance @<domain>
Feature: Browse <resource>
  As a <user role>
  I want to view available <resource>
  So that I can <business goal>

  Background:
    Given I am a logged-in shopper
    And   the <resource> catalogue is available

  Scenario: I can see all available <resource>
    Given I am on the <resource> page
    Then  I should see 2 <resource> listed
    And   the first <resource> should be named "Widget A"

  Scenario: I can view the details of a <resource>
    Given I am on the <resource> page
    When  I select "Widget A"
    Then  I should be taken to the "Widget A" detail page
    And   I should see the price and description
```

### Not Found / Missing Resource

```gherkin
@acceptance @<domain> @error
Feature: Handle missing <resource>
  As a shopper
  I want to see a helpful message when a <resource> does not exist
  So that I am not left on a broken page

  Background:
    Given I am a logged-in shopper

  Scenario: I see a friendly message when a <resource> cannot be found
    Given the <resource> I am looking for does not exist
    When  I try to open that <resource>
    Then  I should see a "Not Found" message
    And   I should see a link to go back to the <resource> listing
```

### Loading State

```gherkin
@acceptance @<domain> @loading
Feature: <Resource> loading experience
  As a shopper
  I want to see a loading indicator while content is being fetched
  So that I know the page is working and not frozen

  Background:
    Given I am a logged-in shopper

  Scenario: I see a loading skeleton while the <resource> list is loading
    Given the <resource> catalogue is taking a moment to load
    When  I open the <resource> page
    Then  I should immediately see a loading skeleton
    And   once loading completes I should see the <resource> listed
    And   the loading skeleton should no longer be visible
```

### Scenario Outline (Data-Driven)

```gherkin
@acceptance @<domain>
Feature: Add items to cart
  As a shopper
  I want to add products to my cart in different quantities
  So that I can purchase exactly what I need

  Background:
    Given I am a logged-in shopper
    And   I am on the product page for "<item>"

  Scenario Outline: I add <item> to my cart
    When  I add <quantity> of "<item>" to my cart
    Then  my cart should show <expected_count> item(s)
    And   the cart total should reflect the correct price

    Examples:
      | item     | quantity | expected_count |
      | Widget A |    1     |       1        |
      | Widget B |    3     |       3        |

  Scenario Outline: I cannot add an invalid quantity of <item>
    When  I attempt to add <quantity> of "<item>" to my cart
    Then  I should see a validation message "Invalid quantity"
    And   my cart should remain empty

    Examples:
      | item     | quantity |
      | Widget C |    0     |
```

### Service Unavailable / Error Boundary

```gherkin
@acceptance @resilience
Feature: Handle service disruptions gracefully
  As a shopper
  I want to see a clear message when something goes wrong
  So that I know the issue is temporary and I can try again

  Background:
    Given I am a logged-in shopper

  Scenario: I see an error message when the <resource> service is unavailable
    Given the <resource> service is currently unavailable
    When  I open the <resource> page
    Then  I should see a message "Something went wrong"
    And   I should see a "Retry" button

  Scenario: I can recover by retrying after a service disruption
    Given the <resource> service is currently unavailable
    And   I am on the <resource> page seeing an error message
    When  I click "Retry"
    Then  the page should attempt to reload the <resource> list
```

---

## Step Definitions (Java)

WireMock and all technical plumbing live here — invisible to the feature file.

```java
// src/test/java/stepdefs/WireMockStepDefs.java
public class WireMockStepDefs {

    private final WireMockServer wireMock =
        new WireMockServer(wireMockConfig().port(8080));

    @Before
    public void startWireMock() {
        wireMock.start();
        WireMock.configureFor("localhost", 8080);
    }

    @After
    public void stopWireMock() {
        wireMock.stop();
    }

    // --- Business step: catalogue is available → 200 stub ---
    @Given("the product catalogue is available")
    public void stubProductCatalogueAvailable() {
        wireMock.stubFor(get(urlEqualTo("/api/products"))
            .willReturn(aResponse()
                .withStatus(200)
                .withHeader("Content-Type", "application/json")
                .withBodyFile("products.json")));
    }

    // --- Business step: resource does not exist → 404 stub ---
    @Given("the {word} I am looking for does not exist")
    public void stubResourceNotFound(String resource) {
        wireMock.stubFor(get(urlMatching("/api/" + resource + "/.*"))
            .willReturn(aResponse()
                .withStatus(404)
                .withHeader("Content-Type", "application/json")
                .withBody("{\"error\":\"Not found\",\"code\":\"NOT_FOUND\"}")));
    }

    // --- Business step: service unavailable → 500 stub ---
    @Given("the {word} service is currently unavailable")
    public void stubServiceUnavailable(String resource) {
        wireMock.stubFor(get(urlEqualTo("/api/" + resource))
            .willReturn(aResponse()
                .withStatus(500)
                .withHeader("Content-Type", "application/json")
                .withBody("{\"error\":\"Internal Server Error\"}")));
    }

    // --- Business step: catalogue is slow to load → delayed stub ---
    @Given("the {word} catalogue is taking a moment to load")
    public void stubCatalogueSlowResponse(String resource) {
        wireMock.stubFor(get(urlEqualTo("/api/" + resource))
            .willReturn(aResponse()
                .withStatus(200)
                .withFixedDelay(1500)
                .withHeader("Content-Type", "application/json")
                .withBodyFile(resource + ".json")));
    }

    // --- Business step: add item to cart → POST stub ---
    @When("I add {int} of {string} to my cart")
    public void addItemToCart(int quantity, String itemName) {
        wireMock.stubFor(post(urlEqualTo("/api/cart"))
            .willReturn(aResponse()
                .withStatus(201)
                .withHeader("Content-Type", "application/json")
                .withBody("{\"cartId\":1,\"quantity\":" + quantity + "}")));
        // Trigger UI action via Playwright/Selenium
        page.click("[data-testid='add-to-cart-" + itemName + "']");
    }

    // --- Business step: invalid quantity → 422 stub ---
    @When("I attempt to add {int} of {string} to my cart")
    public void attemptInvalidCartAdd(int quantity, String itemName) {
        wireMock.stubFor(post(urlEqualTo("/api/cart"))
            .willReturn(aResponse()
                .withStatus(422)
                .withHeader("Content-Type", "application/json")
                .withBody("{\"error\":\"Invalid quantity\"}")));
        page.click("[data-testid='add-to-cart-" + itemName + "']");
    }
}
```

---

## WireMock JSON Stubs

### mappings/get-products-200.json

```json
{
  "request": {
    "method": "GET",
    "url":    "/api/products"
  },
  "response": {
    "status":       200,
    "headers": {
      "Content-Type": "application/json"
    },
    "bodyFileName": "products.json"
  }
}
```

### mappings/get-products-500.json

```json
{
  "request": {
    "method": "GET",
    "url":    "/api/products"
  },
  "response": {
    "status": 500,
    "headers": {
      "Content-Type": "application/json"
    },
    "jsonBody": { "error": "Internal Server Error" }
  }
}
```

### __files/products.json

```json
{
  "products": [
    { "id": 1, "name": "Widget A", "price": 9.99 },
    { "id": 2, "name": "Widget B", "price": 14.99 }
  ]
}
```

---

## Project Structure

```
src/test/
├── resources/
│   └── features/
│       ├── products/
│       │   ├── products_happy_path.feature
│       │   ├── products_error.feature
│       │   └── products_loading.feature
│       └── cart/
│           └── cart_add_items.feature
└── java/
    └── stepdefs/
        ├── WireMockStepDefs.java
        ├── NavigationStepDefs.java
        └── AssertionStepDefs.java

wiremock/
├── mappings/
│   ├── get-products-200.json
│   ├── get-products-404.json
│   └── get-products-500.json
└── __files/
    └── products.json
```

---

## Context Files

Attach these files to the agent for best results:

- `openapi.yaml` — API spec (used to generate accurate paths, payloads, and status codes)
- `src/components/<Component>.tsx` — The React component under test (used to derive `Then` assertions)
- `WireMockServer.java` — Existing server config (used to match port and DSL style)
- Existing `.feature` files — Used to match project conventions and avoid duplicate scenarios

---

## Usage Examples

| Prompt | Output |
|---|---|
| `Generate a BDD feature for browsing products` | `browse_products.feature` with As a/I want/So that narrative |
| `Generate a not-found scenario for products` | Error feature in business language + 404 stub in step def |
| `Generate a data-driven cart feature` | Scenario outline with Examples, no HTTP details in Gherkin |
| `Generate step definitions for the products feature` | `WireMockStepDefs.java` mapping business steps to WireMock calls |
| `Generate a WireMock JSON stub for GET /orders returning 200` | `mappings/get-orders-200.json` + `__files/orders.json` |
| `Generate a resilience feature for service outages` | Error boundary feature in business language |

---

## Conventions

- **BDD-first Gherkin** — every step is written from the user's perspective; no HTTP verbs, status codes, JSON, or API paths ever appear in a `.feature` file
- **Role-Goal-Benefit narrative** — every `Feature` block opens with `As a / I want / So that` to anchor the business value
- **Scenario titles** — written as user outcomes ("I can browse products"), not technical operations ("GET /products returns 200")
- **Given** — describes the user's starting context or the state of the world ("the product catalogue is available")
- **When** — describes a single user action ("I add Widget A to my cart")
- **Then** — describes an observable UI outcome ("I should see 2 products listed")
- **WireMock is invisible** — all `stubFor()`, status codes, and JSON payloads live exclusively in step definitions
- **Tags** — always include `@acceptance` plus a domain tag; add `@error` or `@loading` for non-happy-path scenarios
- **Background** — shared user context lives in `Background` (e.g. "I am a logged-in shopper"), not technical setup
- **Isolation** — `@Before`/`@After` hooks in step definitions reset WireMock state between scenarios; feature files never manage this

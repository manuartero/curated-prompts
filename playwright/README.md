## Goal

Set up a complete suite of **dockerized** Playwright tests.

## Working Directory

`webapp/tests`

## Docker Setup

Include a `Dockerfile` (or `docker-compose.yml`) that runs these tests in a containerized environment using the [official Playwright Docker image](https://playwright.dev/docs/docker). The container should:
- Install dependencies
- Run the full test suite
- Support passing Playwright CLI flags (e.g. `--grep`, `--project`)

## Structure

`webapp/tests/api-responses`: mocked API responses from the CMS (already included).

**What's remaining:** mock image responses from the API to support the Playwright tests.

> **Note:** Whenever you need a mocked image, create a solid single-color placeholder and save it to `webapp/tests/api-responses`. For example, to mock a 300×300 WebP image, create a file named `image300x300.webp` filled with a single solid color (e.g. `#efefef`).

`webapp/tests/integration`: Playwright code and setup

- **`global.setup.ts`** : any global setup needed. This file is called at module level before each `describe` block
- **`fixtures/`** : [Playwright fixtures](https://playwright.dev/docs/test-fixtures)
  - **`test.extend.ts`** : extend Playwright's `test` adding `apiRoutes`, `homePage` fixtures. Example:

```ts
type CustomFixtures = {
  apiRoutes: ApiRoutes;
  homePage: HomePage;
};

export const test = base.extend<CustomFixtures>({
  apiRoutes: [
    async ({ page }, use) => {
      const api = await ApiRoutes.initialize(page);
      await use(api);
    },
    { auto: true },
  ],
  homePage: async ({ page }, use) => {
    const homePage = new HomePage(page);
    await use(homePage);
  },
});
```

- **`home-page.ts`** : fixture for `homePage` exposing locators (e.g. `homePage.cinemaGrid`) and helper functions like `scrollTo('main-value-propositions')`, `scrollTo('faqs')`, etc.
- **`api-routes.ts`** : mock all CMS routing to return our mock responses (JSON and media/images). Export an `initialize(page)` function consumed by `test.extend.ts`. Register route handlers and **block any unregistered request** (abort anything not covered by our mocks to avoid unexpected behavior).

`webapp/tests/app/`: all `.spec` test case files.

## Test Case Conventions

File naming: `${page}.${feature}.spec.ts`

Examples:
- `home.hero-carousel.spec.ts`
- `home.featured-content.spec.ts`
- `home.showcase-rail.spec.ts`
- `home.error-cases.spec.ts`

Each file should contain one `describe('${page}: ${feature}', () => { ... })`.

Call `globalSetup()` before the `describe` block for initial setup.

Create **at least one `test()` (happy path)** per feature of the home page.

## Best Practices

Follow these Playwright guidelines:

1. [Make tests as isolated as possible](https://playwright.dev/docs/best-practices#make-tests-as-isolated-as-possible)
2. [Use locators](https://playwright.dev/docs/best-practices#use-locators)
3. [Use soft assertions](https://playwright.dev/docs/best-practices#use-soft-assertions)
4. **Do not snapshot the entire page.** Use snapshot assertions only on specific locators.

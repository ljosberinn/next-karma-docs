- Getting Started

  - Setting up a new project
    - .env
    - Using Vercel
    - GitHub Actions
    - Using Sentry
  - Authentication

    - Custom Authentication
    - Discord

      How to setup OAuth2 with Discord

      # Requirements

      - Discord Account

      # Steps

      1. go to https://discord.com/developers/applications
      2. create a new application
      3. choose a name
      4. copy `CLIENT ID` and `CLIENT SECRET` into `.env`
      5. select the `OAuth2` tab on the left
      6. enter http://localhost:3000/api/v1/auth/discord as well as any staging/production URLs you might need

      > If you need more profile access than just the basics, read up on how to [here](https://discord.com/developers/docs/topics/oauth2#shared-resources-oauth2-scopes).

    - Facebook

      How to setup OAuth2 with Facebook

      # Requirements

      - Facebook Account

      # Steps

      1. go to https://developers.facebook.com/apps/
      2. add a new app
      3. choose a name
      4. add `Facebook Login` from the list of products
      5. choose `Web`
      6. enter your site url (you may change this later too)
      7. in the left navigation, go to `Settings > Basic`
      8. copy `App ID` and `App Secret` fields over to the `.env` file
      9. in the left navigation, go to `Facebook Login > Settings`
      10. add any staging/production URLs to the field `Valid OAuth Redirect URIs`

    - GitHub

      How to setup OAuth2 with GitHub

      # Requirements

      - GitHub Account

      # Steps

      1. go to https://github.com/settings/developers
      2. create a `New OAuth App`
      3. copy `Client ID` and `Client Secret` over into `.env`
      4. set `Authorization callback URL` according to your current environment

      > GitHub does not allow multiple redirect URLs. For each environment, you will need a separate app.

    - Google

      How to setup OAuth2 with Google

      # Requirements

      - Google Account

      # Steps

      1. go to https://console.developers.google.com/apis/credentials
      2. click on `CREATE CREDENTIALS`
      3. select `OAuth client ID`
      4. select `Web Application` as `Application type`

  - Internationalization

    An overview over the react-i18n integration within next-karma.

    # react-i18next

    [i18next](https://www.i18next.com/) is a well established, well maintained internationalization framework.
    `next-karma` initializes with defaults which you may find in `karma/client/i18n.ts`.

    # Shoutout to [UnlyEd](https://github.com/UnlyEd)

    At the time of writing, none of the previously widely used internationalization libraries for Next.js are serverless compatible.
    UnlyEd and their project [next-right-now](https://github.com/UnlyEd/next-right-now) implemented a solution which `next-karma` draws inspiration from.

    # Language detection

    `next-karma` will automatically detect the language server-side based on (ordered by priority):

    - cookies (`react-i18next` sets a cookie to remember the language)
    - [HTTP Accept-Language header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Language) through an internal parser to avoid unnecessary dependencies
    - dev-defined `FALLBACK_LANGUAGE` (see `.env` chapter)

    # Additional Features

    ## Adjusting <html> attributes

    When a user changes the language, next-karma will adjust the

    - [html.dir attribute](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dir) using an allowlist
    - [html.lang attribute](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/lang)

  - CodeClimate

- Guides

  - API routes with next-connect

    - Using custom middlewares

      # authNSecurity

      A common requirement for API routes is to protect them from unauthorized access. The **authNSecurityMiddleware** ensures that only authenticated requests make it through your actual API route and on top decrypts the stored session, exposing it on the **request** object.

      ```js
      export default nextConnect().use(authNSecurityMiddleware).get(myApiRoute);
      ```

      > To be able to access the manipulated **request** property in TypeScript, you will need to extend the **RequestHandler** interface of **next-connect**.
      > An example can be found in the **/pages/api/v1/user/me.ts** route.

      ```js
      const SESSION_COOKIE_NAME = "session";

      interface AuthenticatedRequest {
        [SESSION_COOKIE_NAME]: User;
      }

      const meHandler: RequestHandler<AuthenticatedRequest> = (req, res) => {
        return res.json(req[SESSION_COOKIE_NAME]);
      };
      ```

      # expectJSONBody

      Many requests to the backend have a JSON body which has to be parsed before being able to use it. The **expectJSONBodyMiddleware** ensures that given a `req.body`, it contains a valid JSON, parses it and overwrites the original `req.body`. if not, it bails with status code 400 (BAD REQUEST).

      For further implementation detail, please inspect **/karma/server/middlewares/expectJSONBody.ts**.

      ```js
      export default nextConnect()
        .use(expectJSONBodyMiddleware)
        .post(myRouteExpectingJSON);
      ```

      > The request object supports this out of the box, so this is natively type-safe.

      # sentryMiddleware

      Just like on the frontend, we should know once something goes wrong in the backend. The **sentryMiddleware** will boot Sentry in an API route, attach metadata such as url, method, host, cookies, query, headers and body.

      ```js
      export default nextConnect()
        .use(sentryMiddleware)
        .post(myRouteThatMightError);
      ```

  - Authentication
    - Adding another provider
  - Internationalization
    - Adding a language
    - Adding a namespace

- Testing

  - Testing Components

    # @testing-library/react

    `next-karma` utilizes [jest](https://jestjs.io/) in combination with [@testing-library/react](https://testing-library.com/docs/react-testing-library/intro) for all component tests.

    If you are unfamiliar with [@testing-library](https://testing-library.com/docs/intro), I'd recommend to familiarize yourself with it first. The basics are easy to pick up.

    > It is highly recommended to use [Testing Playground](https://testing-playground.com/) when writing tests.

    # Custom Render

    `next-karma` comes with a predefined [`custom render function`](https://testing-library.com/docs/react-testing-library/setup#custom-render) for component tests. Its API extends the regular @testing-library `render` function:

    ```js
    render(<Component />, options);
    ```

    It ensures that all tested components are rendered within the following providers:

    - `ChakraProvider` with default theme
    - `I18NextProvider` with a static i18next instance with access to every translation
    - `AuthContextProvider` which defaults to no session

    # Testing components receiving react-i18next props

    Some components may receive properties of the result of `useTranslation` via props. To prevent mocking those all the time and having to resort to check for the existence of translation keys, you may use this:

    ```js
    render(<Component />, {
      i18n: {
        // may also be an array of strings; inherits from useTranslation types
        namespace: "namespace-to-use",
      },
    });
    ```

    To rename props you may pass an additional alias object.
    This will overwrite provided i18n props with the actual props while preserving type-safety.

    ```js
    render(
      <Component
        translate={jest.fn()}
        i18nInstance={jest.fn()}
        i18nReady={false}
      />,
      {
        i18n: {
          namespace: "namespace-to-use",
          alias: {
            t: "translate",
            i18n: "i18nInstance",
            ready: "i18nReady",
          },
        },
      }
    );
    ```

    # Testing with a session

    Some components may depend on an existing session. To allow those components to properly render in tests, you may pass a `session` to `render`. It defaults to null and if given, must match the implementation of `AuthContextDefinition['user']` from the `AuthContextProvider`.

    ```js
    render(<Component />, {
      session: mockGithubProfile,
    });
    ```

    # Accessibility testing

    To easen accessibility testing, a wrapper around [jest-axe](https://github.com/nickcolley/jest-axe) is included. From the same file where the custom render is defined, simply import `testA11Y`:

    ```js
    it("passes a11y test", async () => {
      await testA11Y(<Component />, options);
    });
    ```

    If you need to perform some kind of action, e.g. opening a Menu or Modal before testing for accessibility, you may also pass an element to `testA11Y`:

    ```js
    it("contains valid html when opened", () => {
      const { container } = render(<MyComponent />);

      const button = screen.getByRole('button');

      // actually open the menu as other elements are not rendered before
      userEvent.click(button);

      await testA11Y(container);
    });
    ```

    To disable certain html validation rules, e.g. when unavoidable for reasons beyond your control, you may pass an object `axeOptions` to `options`, which is a proxy for [`axe-core`](https://github.com/nickcolley/jest-axe#axe-configuration) configuration options.

    ```js
    it("contains valid html", async () => {
      await testA11Y(<Component />, {
        axeOptions: {
          rules: {
            "image-alt": { enabled: false },
          },
        },
      });
    });
    ```

    # HTML Validity testing

    To easen HTML validity testing, a wrapper around [html-validate](https://html-validate.org/) is included. From the same file where the custom render is defined, simply import `validateHtml`:

    ```js
    it("contains valid html", () => {
      validateHtml(<Component />, options);
    });
    ```

    To disable certain html validation rules, e.g. when unavoidable for reasons beyond your control, you may pass an object `htmlValidate` to `options`, which is a proxy for [html-validate](https://html-validate.org/rules/index.html) options:

    ```js
    it("contains valid html", () => {
      validateHtml(<Component />, {
        htmlValidate: {
          rules: {
            "attribute-boolean-style": "off",
          },
        },
      });
    });
    ```

    # Best practices

    - always use `describe('<ComponentName />, () => {})` for components
    - avoid naming tests with conjunctives:
      - do: `changes theme on click`
      - don't: `should change theme on click`
    - with required props, always import the type/interface of the components props and use it for your default props definition
    - always use the custom render function unless no render context needed
    - always use the accessibility `testA11Y` function
      - always name the a11y test `passes a11y test` | `...given default props` | `...when opened`
    - always use the html validation `validateHtml` function
      - always name the html test `contains valid html` | `...given default props` | `...when opened`

- Testing API Routes

- Best Practices
  - TypeScript: `WithChildren`, only type non-primitive types

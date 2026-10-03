# pa11y

[![Built with Ona](https://ona.com/build-with-ona.svg)](https://app.ona.com/#https://github.com/Interested-Deving-1896/pa11y) [![KDE Eco](https://img.shields.io/badge/KDE%20Eco-certified-brightgreen?logo=kde&logoColor=white&style=flat-square)](https://eco.kde.org/) [![Blue Angel](https://img.shields.io/badge/Blue%20Angel-DE--UZ%20215-0055a4?style=flat-square)](https://www.blauer-engel.de/en/certification/criteria)


<!-- AI:start:what-it-does -->
_Description pending._
<!-- AI:end:what-it-does -->

## Architecture

<!-- AI:start:architecture -->
_Architecture documentation pending._
<!-- AI:end:architecture -->

## Install

<!-- Add installation instructions here. This section is yours — the AI will not modify it. -->

```bash
git clone https://github.com/Interested-Deving-1896/pa11y.git
cd pa11y
```

## Usage

<!-- Add usage examples here. This section is yours — the AI will not modify it. -->

## Configuration


Pa11y has lots of options you can use to change the way Headless Chrome runs, or the way your page is loaded. Options can be set either as a parameter on the `pa11y` function or in a [JSON or JavaScript configuration file](#command-line-configuration). Some are also available directly as [command-line options](#command-line-interface).

Below is a reference of all the options that are available. Example [JSON](example/configs/pa11y.json) and [JavaScript](example/configs/pa11y.js) configuration files are available in `example/configs/`. Note that unlike [`pa11y-ci` configuration](https://github.com/pa11y/pa11y-ci?tab=readme-ov-file#default-configuration), there is no `defaults` property.

### `actions` (array)

Actions to be run before Pa11y tests the page. There are quite a few different actions available in Pa11y, the [Actions documentation](#actions) outlines each of them.

```js
pa11y(url, {
    actions: [
        'set field #username to exampleUser',
        'set field #password to password1234',
        'click element #submit',
        'wait for path to be /myaccount'
    ]
});
```

Defaults to an empty array.

### `browser` (Browser) and `page` (Page)

A [Puppeteer Browser instance][puppeteer-browser] which will be used in the test run. Optionally you may also supply a [Puppeteer Page instance][puppeteer-page], but this cannot be used between test runs as event listeners would be bound multiple times.

If either of these options are provided then there are several things you need to consider:

  1. Pa11y's `chromeLaunchConfig` option will be ignored, you'll need to pass this configuration in when you create your Browser instance
  2. Pa11y will not automatically close the Browser when the tests have finished running, you will need to do this yourself if you need the Node.js process to exit
  3. It's important that you use a version of Puppeteer that meets the range specified in Pa11y's `package.json`
  4. You _cannot_ reuse page instances between multiple test runs, doing so will result in an error. The page option allows you to do things like take screen-shots on a Pa11y failure or execute your own JavaScript before Pa11y

**Note:** This is an advanced option. If you're using this, please mention in any issues you open on Pa11y and double-check that the Puppeteer version you're using matches Pa11y's.

```js
const browser = await puppeteer.launch({
    ignoreHTTPSErrors: true
});

pa11y(url, {
    browser: browser
});

browser.close();
```

A more complete example can be found in the [puppeteer examples](#puppeteer-example).

Defaults to `null`.

### `chromeLaunchConfig` (object)

Launch options for the Headless Chrome instance. [See the Puppeteer documentation for more information][puppeteer-launch].

```js
pa11y(url, {
    chromeLaunchConfig: {
        executablePath: '/path/to/Chrome',
        ignoreHTTPSErrors: false
    }
});
```

Defaults to:

```js
{
    ignoreHTTPSErrors: true
}
```

### `headers` (object)

A key-value map of request headers to send when testing a web page.

```js
pa11y(url, {
    headers: {
        Cookie: 'foo=bar'
    }
});
```

Defaults to an empty object.

### `hideElements` (string)

A CSS selector to hide elements from testing, selectors can be comma separated. Elements matching this selector will be hidden from testing by styling them with `visibility: hidden`.

```js
pa11y(url, {
    hideElements: '.advert, #modal, div[aria-role=presentation]'
});
```

### `ignore` (array)

An array of result codes and types that you'd like to ignore. You can find the codes for each rule in the console output and the types are `error`, `warning`, and `notice`. Note: `warning` and `notice` messages are ignored by default.

```js
pa11y(url, {
    ignore: [
        'WCAG2AA.Principle3.Guideline3_1.3_1_1.H57.2'
    ]
});
```

Defaults to an empty array.

### `ignoreUrl` (boolean)

Whether to use the provided [Puppeteer Page instance][puppeteer-page] as is or use the provided url. Both the [Puppeteer Page instance][puppeteer-page] and the [Puppeteer Browser instance][puppeteer-browser] are required alongside `ignoreUrl`.

```js
const browser = await puppeteer.launch();
const page = await browser.newPage();

pa11y(url, {
    ignoreUrl: true,
    page: page,
    browser: browser
});
```

Defaults to `false`.

### `includeNotices` (boolean)

Whether to include results with a type of `notice` in the Pa11y report. Issues with a type of `notice` are not directly actionable and so they are excluded by default. You can include them by using this option:

```js
pa11y(url, {
    includeNotices: true
});
```

Defaults to `false`.

### `includeWarnings` (boolean)

Whether to include results with a type of `warning` in the Pa11y report. Issues with a type of `warning` are not directly actionable and so they are excluded by default. You can include them by using this option:

```js
pa11y(url, {
    includeWarnings: true
});
```

Defaults to `false`.

### `level` (string)

The level of issue which can fail the test (and cause it to exit with code 2) when running via the CLI. This should be one of `error` (the default), `warning`, or `notice`.

```json
{
    "level": "warning"
}
```

Defaults to `error`. Note this configuration is only available when using Pa11y on the command line, not via the JavaScript Interface.

### `levelCapWhenNeedsReview` (string)

Cap any issue requiring manual review to this level. This should be one of `error` (the default), `warning`, or `notice`. Only used by the axe runner.

```json
{
    "levelCapWhenNeedsReview": "warning"
}
```

### `log` (object)

An object which implements the methods `debug`, `error`, and `info` which will be used to report errors and test information.

```js
pa11y(url, {
    log: {
        debug: console.log,
        error: console.error,
        info: console.info
    }
});
```

Each of these defaults to an empty function.

### `method` (string)

The HTTP method to use when running Pa11y.

```js
pa11y(url, {
    method: 'POST'
});
```

Defaults to `GET`.

### `postData` (string)

The HTTP POST data to send when running Pa11y. This should be combined with a `Content-Type` header. E.g to send form data:

```js
pa11y(url, {
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded'
    },
    method: 'POST',
    postData: 'foo=bar&bar=baz'
});
```

Or to send JSON data:

```js
pa11y(url, {
    headers: {
        'Content-Type': 'application/json'
    },
    method: 'POST',
    postData: '{"foo": "bar", "bar": "baz"}'
});
```

Defaults to `null`.

### `reporter` (string)

The reporter to use while running the test via the CLI. [More about reporters](#reporters).

```json
{
    "reporter": "json"
}
```

Defaults to `cli`. Note this configuration is only available when using Pa11y on the command line, not via the JavaScript Interface.

### `rootElement` (element)

The root element for testing a subset of the page opposed to the full document.

```js
pa11y(url, {
    rootElement: '#main'
});
```

Defaults to `null`, meaning the full document will be tested. If the specified root element isn't found, the full document will be tested.

### `runners` (array)

An array of runner names which correspond to existing and installed [Pa11y runners](#runners). If a runner is not found then Pa11y will error.

```js
pa11y(url, {
    runners: [
        'axe',
        'htmlcs'
    ]
});
```

Defaults to:

```js
[
    'htmlcs'
]
```

### `rules` (array)

An array of WCAG 2.1 guidelines that you'd like to include to the current standard. You can find the codes for each guideline in the [HTML Code Sniffer WCAG2AAA ruleset][htmlcs-wcag2aaa-ruleset]. **Note:** only used by htmlcs runner.

```js
pa11y(url, {
    rules: [
        'Principle1.Guideline1_3.1_3_1_AAA'
    ]
});
```

### `screenCapture` (string)

A file path to save a screen capture of the tested page to. The screen will be captured immediately after the Pa11y tests have run so that you can verify that the expected page was tested.

```js
pa11y(url, {
    screenCapture: `${__dirname}/my-screen-capture.png`
});
```

Defaults to `null`, meaning the screen will not be captured. Note the directory part of this path must be an existing directory in the file system – Pa11y will not create this for you.

### `standard` (string)

The accessibility standard to use when testing pages. This should be one of:

- `WCAG2A`
- `WCAG2AA`
- `WCAG2AAA` (this level is currently used only by Pa11y's runner for HTML_CodeSniffer)

```js
pa11y(url, {
    standard: 'WCAG2A'
});
```

Defaults to `WCAG2AA`.

### `threshold` (number)

The number of errors, warnings, or notices to permit before the test is considered to have failed (with exit code 2) when running via the CLI.

```json
{
    "threshold": 9
}
```

Defaults to `0`. Note this configuration is only available when using Pa11y on the command line, not via the JavaScript Interface.

### `timeout` (number)

The time in milliseconds that a test should be allowed to run before calling back with a timeout error.

Please note that this is the timeout for the _entire_ test run (including time to initialise Chrome, load the page, and run the tests).

```js
pa11y(url, {
    timeout: 500
});
```

Defaults to `30000`.

### `userAgent` (string)

The `User-Agent` header to send with Pa11y requests. This is helpful to identify Pa11y in your logs.

```js
pa11y(url, {
    userAgent: 'A11Y TESTS'
});
```

Defaults to `pa11y/<version>`.

### `viewport` (object)

The viewport configuration. This can have any of the properties supported by the [puppeteer `setViewport` method][puppeteer-viewport].

```js
pa11y(url, {
    viewport: {
        width: 320,
        height: 480,
        deviceScaleFactor: 2,
        isMobile: true
    }
});
```

Defaults to:

```js
{
    width: 1280,
    height: 1024
}
```

### `wait` (number)

The time in milliseconds to wait before running HTML_CodeSniffer on the page.

```js
pa11y(url, {
    wait: 500
});
```

Defaults to `0`.

## CI

<!-- AI:start:ci -->
_CI documentation pending._
<!-- AI:end:ci -->

## Mirror chain

<!-- AI:start:mirror-chain -->
This repo is maintained in [`Interested-Deving-1896/pa11y`](https://github.com/Interested-Deving-1896/pa11y) and mirrored through:

```
Interested-Deving-1896/pa11y  ──►  OpenOS-Project-OSP/pa11y  ──►  OpenOS-Project-Ecosystem-OOC/pa11y
```

Changes flow downstream automatically via the hourly mirror chain in
[`fork-sync-all`](https://github.com/Interested-Deving-1896/fork-sync-all).
Direct commits to OSP or OOC are detected and opened as PRs back to `Interested-Deving-1896`.
<!-- AI:end:mirror-chain -->

## Contributors

<!-- AI:start:contributors -->
_Contributors pending._
<!-- AI:end:contributors -->

## Origins

<!-- AI:start:origins -->
_Original project — no upstream influences recorded._
<!-- AI:end:origins -->

## Resources

<!-- AI:start:resources -->
_No additional resource files found._
<!-- AI:end:resources -->

## Accessibility

<!-- AI:start:accessibility -->
This repo uses automated accessibility auditing via `check-accessibility.yml`.

Checks include: CODEOWNERS ownership coverage, README screen-reader compatibility,
WCAG 2.1 AA HTML compliance, audio overview (espeak-ng), and Braille output (liblouis).




Run the [Check Accessibility](https://github.com/Interested-Deving-1896/pa11y/actions/workflows/check-accessibility.yml)
workflow to generate the first report and accessibility artifacts.
See [DOCS/accessibility.md](https://github.com/Interested-Deving-1896/pa11y/blob/main/DOCS/accessibility.md) for the full reference.
<!-- AI:end:accessibility -->

## License

<!-- AI:start:license -->
[LGPL-3.0](https://github.com/Interested-Deving-1896/pa11y/blob/main/LICENSE) © 2026 [Interested-Deving-1896](https://github.com/Interested-Deving-1896)
<!-- AI:end:license -->

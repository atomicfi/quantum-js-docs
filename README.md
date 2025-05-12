# QuantumJS

Build user-permissioned connections into third party systems.

## Installation

```bash
$ npm i @atomicfi/quantum-js
```

## Example

```ts
import { Quantum, AuthStatus } from '@atomicfi/quantum-js'

// Authenticate into a website
const { page } = await Quantum.launch()
const startURL = 'https://example.com'

await page.show()

const status = await page.authenticate(startURL, async (page) => {
  const url = await page.url()
  return url?.includes('/dashboard')
})

if (status !== AuthStatus.Authenticated) {
  return
}

await page.hide()

// Use cookies and intercepted requests from the page to build an integration on your backend
const response = await axios.post('http://yourapi.com/endpoint', {
  cookies: await page.cookies('https://example.com'),
  headers: page.getRequestHeaders('https://api.com/endpoint')
})
```

# Quantum API

## Quantum.launch

### Parameters

Creates a quantum instance that includes a browser page.

| Parameter | Type          | Description                             |
| --------- | ------------- | --------------------------------------- |
| options   | LaunchOptions | Options for creating a Quantum instance |

### LaunchOptions

| Parameter         | Type    | Description                                                   |
| ----------------- | ------- | ------------------------------------------------------------- |
| interceptRequests | boolean | Whether to intercept page requests or not, defaults to `true` |

### Returns

`Promise<{ browser, page }>`

### Example

```ts
const { browser, page } = await Quantum.launch()

// Your interation logic with the `page`

// Frees up memory
await browser.close()
```

# Browser API

## browser.close

Closes the browser and frees up memory. Call this when you're done with your integration.

# Page API

## page.authenticate

Helper function to authenticate a website. Returns once the `authEvaluator` returns a truthy result, or throws an error if the user does not authenticate within the give time window.

| Parameter     | Type   | Description                                              |
| ------------- | ------ | -------------------------------------------------------- |
| startURL      | string | The URL to navigate to                                   |
| authEvaluator | Func   | A function that represents when a user is authenticated. |
| options       | Object | (Optional) Options for authenticating a user.            |

### AuthenticateOptions

| Parameter | Type   | Description                                                           |
| --------- | ------ | --------------------------------------------------------------------- |
| timeout   | number | (Optional) Timeout in seconds to wait for authentication to complete. |

### Returns

`Promise<AuthStatus>`

### Throws

`AuthError`

### Example

```ts
const status = await page.authenticate(
  startURL,
  async (page) => {
    const url = await page.url()
    return url?.includes('/dashboard')
  },
  {
    timeout: 90000
  }
)

if (status === AuthStatus.Authenticated) {
  // User is authenticated
}
```

## page.click

Click's an element on the page.

### Parameters

| Parameter | Type         | Description                                 |
| --------- | ------------ | ------------------------------------------- |
| selector  | string       | CSS selector to find the element            |
| options   | ClickOptions | (Optional) Options for clicking the element |

### ClickOptions

| Parameter         | Type    | Description                                                                  |
| ----------------- | ------- | ---------------------------------------------------------------------------- |
| simulateFramework | boolean | Whether to simulate a framework's click behavior or not, defaults to `false` |

### Example

```ts
await page.click('#login')
```

```ts
await page.click('#login', { simulateFramework: true })
```

## page.cookies

Returns cookies for the specified URL

### Parameters

| Parameter | Type   | Description                         |
| --------- | ------ | ----------------------------------- |
| url       | string | The domain to retrieve cookies from |

### Returns

`Promise<Cookie[]>`

### Example

```ts
const cookies = await page.cookies('https://example.com')
const cookieString = cookies
  .map((cookie) => `${cookie.name}=${cookie.value}`)
  .join('; ')
```

## page.evaluate

Evaluates a function in the page's context and returns the result.

### Parameters

| Parameter    | Type   | Description                              |
| ------------ | ------ | ---------------------------------------- |
| pageFunction | Func   | A function that is run within the page   |
| args         | Params | Parameters to send to the `pageFunction` |

### Returns

`Promise<unknown>`

### Example

```ts
const title = await page.evaluate(() => document.title)

const delayed = await page.evaluate(async () => {
  await new Promise((resolve) => setTimeout(resolve, 1000))
  return 'delayed'
})
```

```ts
const result = await page.evaluate((a, b) => a + b, [1, 2])
// `result` is `3`
```

## page.getRequests

Returns an array of all intercepted requests on the page.

### Parameters

None

### Returns

`Promise<Request[]>`

### Example

```ts
const requests = page.getRequests()
const authRequest = requests.find((request) => request.url.includes('/auth'))[0]

const {
  url,
  method,
  headers,
  data,
  response: { status, headers, data }
} = authRequest
```

## page.getRequestHeaders

Returns an object of merged request headers from the specified URL pattern. Note: if there are multiple requests made on the page that all use an `Authorization` header, then the most recent request's `Authorization` header would be returned in the object from this call.

### Parameters

| Parameter | Type                               | Description                                             |
| --------- | ---------------------------------- | ------------------------------------------------------- |
| url       | string \| (url: string) => boolean | Specific URL to match against or a URL matcher function |

### Returns

`Promise<RequestHeaders>`

### Example

```ts
// Specific URL match
const headers = page.getRequestHeaders('https://api.com/endpoint')
const authorizationToken = headers.authorization

// Request matcher function
const headers = page.getRequestHeaders((request: Request) =>
  request.url.includes('api.com')
)
const authorizationToken = headers.authorization
```

## page.goto

Navigates the page to the given url.

### Parameters

| Parameter | Type   | Description                                                                |
| --------- | ------ | -------------------------------------------------------------------------- |
| url       | string | URL to navigate the frame to. The URL should include scheme, e.g. https:// |

### Returns

`Promise<void>`

### Example

```ts
await page.goto('https://example.com')
```

## page.hide

Hide's the current page in the UI.

### Parameters

None

### Returns

`Promise<void>`

### Example

```ts
await page.hide()
```

## page.input

Input's text into a field.

## Parameters

| Parameter | Type         | Description                           |
| --------- | ------------ | ------------------------------------- |
| selector  | string       | CSS selector to find the input field  |
| text      | string       | Text to input into the field          |
| options   | InputOptions | (Optional) Options for inputting text |

### InputOptions

| Parameter         | Type    | Description                                                                  |
| ----------------- | ------- | ---------------------------------------------------------------------------- |
| simulateFramework | boolean | Whether to simulate a framework's input behavior or not, defaults to `false` |
| simulateTyping    | boolean | Whether to simulate typing or not, defaults to `true`                        |
| triggerEvents     | boolean | Whether to trigger events after inputting text or not, defaults to `true`    |

### Example

```ts
await page.input('#username', 'admin')
```

```ts
await page.input('#username', 'admin', { simulateFramework: true })
```

## page.request

Executes a native request on the user's device.

### Parameters

```javascript
{
    // `url` is the full URL that will be used for the request
    url: '/user',
    // `method` is the request method to be used when making the request
    method: 'get' // default
    // `data` is the data to be sent as the request body
    // Only applicable for request methods 'PUT', 'POST', 'DELETE', and 'PATCH'
    data: {
        firstName: 'Fred'
    },
    // `headers` are custom headers to be sent
    headers: {'X-Requested-With': 'XMLHttpRequest'},
    // `followRedirects` is whether to follow request redirects
    followRedirects: true // default
}
```

### Returns

`Promise<Request>`

### Example

```ts
const response = await page.request({
  url: 'https://example.com/endpoint',
  headers: { 'Content-Type': 'application/json' },
  data: { example: 'data' },
  method: 'post'
})

const { status, headers, data } = response

if (status === 200) {
  const contentType = headers['Content-Type']
  const dataField = data['data-field']
}

// `status` is the status of the response
// `headers` are the response headers of the request
// `data` is the response payload of the request
```

## page.screenshot

Takes a screenshot of the current page.

### Parameters

| Parameter | Type   | Description                          |
| --------- | ------ | ------------------------------------ |
| width     | number | (Optional) Width of the screenshot   |
| height    | number | (Optional) Height of the screenshot  |
| quality   | number | (Optional) Quality of the screenshot |

### Returns

`Promise<string | undefined>`

A base64 encoded string of the screenshot.

### Example

```ts
// Screenshot the entire viewport
const screenshot = await page.screenshot()

// Screenshot a specific width and height
const screenshot = await page.screenshot(100, 100)
```

## page.setUserAgent

Set's the user agent of the page

### Parameters

| Parameter | Type   | Description                             |
| --------- | ------ | --------------------------------------- |
| userAgent | string | Specific user agent to use in this page |

### Returns

`Promise<void>`

### Example

```ts
await page.setUserAgent(
  'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36'
)
```

## page.show

Display's the current page in the UI.

### Parameters

None

### Returns

`Promise<void>`

### Example

```ts
await page.show()
```

## page.url

The page's URL.

### Parameters

None

### Returns

`Promise<string>`

### Example

```ts
const url = await page.url()
```

## page.waitForAuthentication

Wait for the user to be authenticated.

### Parameters

| Parameter     | Type | Description                                              |
| ------------- | ---- | -------------------------------------------------------- |
| authEvaluator | Func | A function that represents when a user is authenticated. |

### Returns

`Promise<AuthStatus>`

### Example

```ts
const status = await page.waitForAuthentication(async (page) => {
  const url = await page.url()
  return url?.includes('/dashboard')
})

if (status === AuthStatus.Authenticated) {
  // User is authenticated
}
```

## page.waitForSelector

Wait for the `selector` to appear in page. If at the moment of calling the method the `selector` already exists, the method will return immediately. If the `selector` doesn't appear after the timeout milliseconds of waiting, the function will throw.

### Parameters

| Parameter | Type           | Description                            |
| --------- | -------------- | -------------------------------------- |
| selector  | string         | CSS selector to query the page for.    |
| options   | WaitForOptions | (Optional) Optional waiting parameters |

### Example

```ts
// Wait the default amount of time of 60 seconds for the CSS selector #login to be displayed
await page.waitForSelector('#login')
```

```ts
// Wait for the CSS selector #logout to be displayed. Attempt looking for this element 10 times with a 60ms delay between each check.
await page.waitForSelector('#logout', { times: 10, interval: 60 })
```

## page.waitForFunction

Waits for the provided function, `pageFunction`, to return a truthy value when evaluated in the page's context.

| Parameter    | Type           | Description                                                                  |
| ------------ | -------------- | ---------------------------------------------------------------------------- |
| pageFunction | Func \| string | Function to be evaluated in browser context until it returns a truthy value. |
| options      | WaitForOptions | (Optional) Optional waiting parameters                                       |
| args         | Params         | Parameters to send to the `pageFunction`                                     |

## Example

```ts
const timeout = 1000

// Will wait for a second before preceding
await page.waitForFunction(
  async (timeout) => {
    await new Promise((resolve) => setTimeout(resolve), timeout)
    return true
  },
  null,
  [timeout]
)
```

## page.waitForRequest

Wait for a request to be made on the page that matches the given criteria.

### Parameters

| Parameter      | Type                           | Description                                                       |
| -------------- | ------------------------------ | ----------------------------------------------------------------- |
| requestMatcher | string \| RequestMatchFunction | Specific URL to match against or a URL matcher function           |
| options        | WaitForOptions                 | (Optional) Options for waiting, defaults to DefaultWaitForOptions |

### Returns

`Promise<void>`

### Example

```ts
// Wait for specific URL
await page.waitForRequest('https://example.com/endpoint')

// Wait using matcher function
await page.waitForRequest((request: Request) => request.url.includes('api.com'))

// Wait with custom options
await page.waitForRequest('https://example.com/endpoint', {
  times: 10,
  interval: 100
})
```

## page.on

Listen for events happening on the page.

### Events

| Event          | Description                          |
| -------------- | ------------------------------------ |
| close          | WebChromeClient.onCloseWindow        |
| closed         | When page.close() is called          |
| dispatch       | Pass through custom event            |
| started        | WebViewClient.onPageStarted          |
| finished       | WebViewClient.onPageFinished         |
| locationchange | WebViewClient.doUpdateVisitedHistory |
| domchange      | JS document dom changing events      |
| hostblocked    | Host not in provided allow list      |

### Example

```ts
await page.on('finished', () => {
  // Page has finished loading
})
```

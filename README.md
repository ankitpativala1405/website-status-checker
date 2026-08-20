# website-status-checker

[![npm version](https://img.shields.io/npm/v/website-status-checker.svg)](https://www.npmjs.com/package/website-status-checker)
[![npm downloads](https://img.shields.io/npm/dm/website-status-checker.svg)](https://www.npmjs.com/package/website-status-checker)
[![License](https://img.shields.io/npm/l/website-status-checker.svg)](https://github.com/ankitpativala1405/website-status-checker/blob/main/LICENSE)

A lightweight Node.js and TypeScript package for checking website availability, HTTP status codes, response details, network timings, redirects, and SSL certificate information.

It supports one-time checks and continuous monitoring without an API key or external service.

## Benefits

- Check whether a website is up or down using configurable HTTP status codes.
- Measure DNS lookup, TCP connection, TLS handshake, TTFB, transfer, and total response times.
- Inspect response size, content type, content length, redirects, and errors.
- Optionally read HTTPS certificate dates, issuer, subject, fingerprint, serial number, and SAN data.
- Monitor websites continuously with a configurable interval and stop function.
- Use custom headers, `GET` or `HEAD` requests, timeouts, and redirect limits.
- Works with HTTP and HTTPS and provides predictable TypeScript types.
- Supports CommonJS, ES Modules, and TypeScript projects.

## Requirements

- Node.js 18 or newer

## Installation

```bash
npm install website-status-checker
```

## Basic Usage

### CommonJS

```js
const { checkWebsite } = require("website-status-checker");

async function main() {
  const result = await checkWebsite("https://example.com");

  console.log("Is website up?", result.is_up);
  console.log("Status code:", result.status.code);
  console.log("Total response time:", result.timings.total_time, "ms");
}

main();
```

### ES Modules

```js
import { checkWebsite } from "website-status-checker";

const result = await checkWebsite("https://example.com");

console.log(result);
```

### TypeScript

```ts
import { checkWebsite, WebsiteStatus } from "website-status-checker";

const result: WebsiteStatus = await checkWebsite("https://example.com", {
  expectedStatusCodes: [200, 204],
});

if (result.is_up) {
  console.log("Website is available");
} else {
  console.error(result.error?.message);
}
```

## SSL Certificate Information

Pass `ssl: true` for certificate information from an HTTPS request. The default is `false`.

```ts
import { checkWebsite } from "website-status-checker";

const result = await checkWebsite("https://example.com", {
  ssl: true,
});

if (result.ssl) {
  console.log("SSL starts:", result.ssl.start_date);
  console.log("SSL expires:", result.ssl.expire_date);
  console.log("Issuer:", result.ssl.issuer);
  console.log("Subject:", result.ssl.subject);
  console.log("Serial number:", result.ssl.serial_number);
  console.log("Fingerprint:", result.ssl.fingerprint);
  console.log("Subject alternative names:", result.ssl.subject_alt_name);
}
```

For HTTP URLs, requests without SSL enabled, and failed requests, `result.ssl` is `null`.

`ssl: true` only reads certificate information. Certificate validation is controlled separately with `rejectUnauthorized`, which defaults to `true`.

## Continuous Monitoring

`monitorWebsite` runs a check immediately and repeats it after each completed check. It returns a function that stops future checks.

```ts
import { monitorWebsite } from "website-status-checker";

const stop = monitorWebsite("https://example.com", {
  interval: 30_000,
  ssl: true,
  onResult: (result) => {
    console.log({
      isUp: result.is_up,
      statusCode: result.status.code,
      responseTime: result.timings.total_time,
      sslExpiry: result.ssl?.expire_date,
    });
  },
  onError: (error) => {
    console.error("Monitoring error:", error.message);
  },
});

// Call this when monitoring is no longer needed.
setTimeout(stop, 5 * 60 * 1000);
```

`onResult` is called for both successful and failed website checks. `onError` is reserved for unexpected monitoring errors.

## Options

The same check options can be passed to `checkWebsite` and `monitorWebsite`.

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `timeout` | `number` | `10000` | Request timeout in milliseconds. |
| `method` | `"GET" \| "HEAD"` | `"GET"` | HTTP method to use. |
| `headers` | `Record<string, string>` | `{}` | Custom request headers. |
| `expectedStatusCodes` | `number[]` | `[200]` | Status codes considered available. |
| `followRedirects` | `boolean` | `true` | Follow HTTP redirects. |
| `maxRedirects` | `number` | `5` | Maximum redirects to follow. |
| `maxResponseSize` | `number` | `10485760` | Maximum response size in bytes. |
| `rejectUnauthorized` | `boolean` | `true` | Reject invalid TLS certificates. |
| `ssl` | `boolean` | `false` | Include HTTPS certificate information. |
| `interval` | `number` | `1000` | Monitoring interval in milliseconds. `monitorWebsite` only. |

## Example Options

```ts
const result = await checkWebsite("https://api.example.com/health", {
  method: "HEAD",
  timeout: 5_000,
  headers: {
    Authorization: "Bearer token",
    "User-Agent": "my-monitor",
  },
  expectedStatusCodes: [200, 204],
  followRedirects: true,
  maxRedirects: 3,
  ssl: true,
});
```

## Result Structure

Every completed check returns a `WebsiteStatus` object:

```ts
{
  url: string;
  is_up: boolean;
  status: {
    code: number;
    text: string;
  };
  expected_status_codes: number[];
  timings: {
    lookup_time: number;
    connection_time: number;
    tls_time: number;
    ttfb: number;
    transfer_time: number;
    total_time: number;
  };
  response: {
    size: number;
    contentType: string | null;
    contentLength: number | null;
  };
  ssl: SSLInfo | null;
  redirects: string[];
  error: {
    code: string | null;
    message: string;
  } | null;
  timestamp: string;
}
```

Timing values are in milliseconds. A failed request is returned with `is_up: false` and details in `error` instead of throwing for normal network or HTTP request failures.

## SSLInfo Fields

When `ssl: true` is used with HTTPS, `result.ssl` contains:

| Field | Description |
| --- | --- |
| `start_date` | Certificate start date. |
| `expire_date` | Certificate expiry date. |
| `valid_from` | Certificate start date from Node.js TLS data. |
| `valid_to` | Certificate expiry date from Node.js TLS data. |
| `subject` | Certificate subject. |
| `issuer` | Certificate issuer. |
| `serial_number` | Certificate serial number. |
| `fingerprint` | Certificate fingerprint. |
| `subject_alt_name` | Certificate subject alternative names, if available. |

## License

MIT
# GitHub v3 REST API Swift SDK

GitHub's v3 REST API.

> Package `OctriGithub` · Version `1.1.4` · 1230 operations

## Installation

```swift
.package(path: "/path/to/generated-sdk")

// After this version is tagged
.package(url: "https://github.com/OctriDev/github-rest-swift-sdk.git", from: "1.1.4")
```

## Quickstart

The example calls `activityGetFeeds` (GET `/feeds`), a low-friction operation that requires no request arguments.

```swift
import Foundation
import OctriGithub

@main
struct Example {
    static func main() async throws {
        let config = ClientConfig(
            baseUrl: "https://api.github.com",
            auth: ClientAuthConfig(bearer: ProcessInfo.processInfo.environment["API_TOKEN"]!)
        )
        let client = GitHubV3REST(config: config)
        let result = try await client.activity.getFeeds()
        print(result)
    }
}
```

## Authentication

Keep credentials outside source control. The quickstart reads them from the environment and the client applies them to every request.

| Scheme | ClientAuthConfig field | Sent as |
| --- | --- | --- |
| SDK Studio bearer token | `bearer` | `Authorization: Bearer <token>` |

## Client behavior

- Base URL: `https://api.github.com`.
- Transport: URLSession.
- Timeout: 30,000 ms per attempt.
- Retries: up to 3 attempts for status codes `408`, `425`, `429`, `500`, `502`, `503`, `504`, with 500–8,000 ms backoff.
- Idempotency: disabled.
- Error telemetry is disabled by default, even when a reporting endpoint is baked into the build. Consumers must opt in explicitly.
- Telemetry PII filtering is enabled by default: common credentials and direct identifiers are recursively replaced with `[REDACTED]` before reports are sent. Disable it only through the generated logging config's `filterPii` (or language-native equivalent) for a trusted private sink.

High-level operation methods return the typed response body directly. The low-level request layer returns an `SdkResponse<T>` envelope containing data, status, headers, request ID, latency, and attempt count.

## Errors and response metadata

All failure paths use a small, predictable hierarchy:

| Error | Meaning |
| --- | --- |
| `SdkValidationError` | A request argument failed an OpenAPI constraint before network I/O. |
| `SdkHttpError` | The server returned a non-2xx response. |
| `SdkNetworkError` | DNS, connection, TLS, or socket failure. |
| `SdkTimeoutError` | The configured per-attempt timeout elapsed. |

HTTP errors expose `statusCode`, the response body and headers, plus `requestId` when the server supplies one. Preserve the request ID in support logs; it is the fastest way to correlate a failed SDK call with server-side traces.

## Project layout and API discovery

- Operation implementations are grouped under `Sources/OctriGithub/Methods/`.
- 975 component models are split by API domain under `Sources/OctriGithub/Models/<Domain>Models.swift` or `Models/<Tag Path>/<TagPath>Models.swift` in the same Swift module.
- Component schemas can choose a nested model folder with `x-octri-sdk-tags: ["Billing/Invoices"]`; the first tag owns the model and `/` creates nesting.
- [`sdk-manifest.json`](sdk-manifest.json) is the language-neutral public API index: operations, request/response modes, model properties, enum values, and generation settings.
- Public barrel/module exports are the compatibility boundary. Import public model names from those exports; internal domain filenames may evolve without changing model names.

## Links

- [Source repository](https://github.com/OctriDev/github-rest-swift-sdk)
- [Issue tracker](https://github.com/OctriDev/github-rest-swift-sdk/issues)
- [API documentation](https://docs.github.com/rest/)
- [Support](https://support.github.com/contact?tags=dotcom-rest-api)
- [Terms of service](https://docs.github.com/articles/github-terms-of-service)
- [MIT](https://spdx.org/licenses/MIT)

<!-- sdk-studio-mock-tests -->
## Local mock-server tests

Generated SDK includes schema-derived, zero-dependency mock server and network
contract suite. Node.js 20+ required. Contract probes use authored response
examples only; schema-synthesized routes remain available to the local server.

`./scripts/mock --port 4010` starts server. `./scripts/test` runs the mock contract suite, then native SDK tests. A zero-authored-example contract run succeeds with an explicit zero-test
summary; mismatches in authored examples still fail.

# From HTTP Response to Trustworthy Market Data

A price field arrives as the string `"187.42"`. Is it a current quote, a delayed quote, an error message disguised as JSON, or a number attached to the wrong symbol? Fetching market data is easy to demonstrate. Fetching data that a downstream system can interpret responsibly takes more care.

The section’s central lesson is that public APIs let software retrieve market information, often as JSON, and that a Rust application can combine HTTP requests, asynchronous execution, and JSON deserialization to process it. It also calls attention to network failures, API limits, and changing response formats. Those are useful starting points. In a real application, each is a boundary to validate, not a detail to assume away.

## An API response is a contract, not a promise

A public market-data API may provide quotes, historical observations, volumes, or broader financial datasets. The section names Alpha Vantage, IEX Cloud, Quandl, and Yahoo Finance as examples. Treat that list as illustrative, not as a guarantee that every provider currently offers the same endpoints, access terms, or data freshness. Providers change products and policies. Read the current provider documentation before choosing an endpoint.

An API contract describes what a request means and how its response is shaped. It may specify required parameters, authentication, units, timestamps, market coverage, and limits on calls. Those details matter as much as the JSON syntax. A response containing a price without its currency, observation time, or delay status may be parseable but unsuitable for a particular decision.

The section’s Alpha Vantage example requests `GLOBAL_QUOTE`, supplies a symbol and API key, and reads fields under `Global Quote`. It demonstrates the basic path: make an HTTP GET request, decode JSON, and extract a symbol and price. It does not establish that a response is real-time, that every request succeeds, or that the displayed field names can never change. Check the provider’s current semantics and response examples.

## Make the data path explicit

A dependable data path has distinct stages:

1. Build a request with validated parameters and protected credentials.
2. Send it over the network and handle transport failure.
3. Inspect the HTTP status and any provider-specific error information.
4. Decode the response body and validate its shape and values.
5. Attach context such as retrieval time, source, and freshness status.
6. Store or pass the observation in a form downstream code understands.

Separating these stages makes failures easier to diagnose. A DNS problem is not a JSON parsing problem. An HTTP success status is not proof that the body contains a quote: an API can return a successful HTTP response with a quota warning or an application-level error. Similarly, a valid decimal string is not proof that it is a fresh market price.

The section introduces Rust’s `reqwest` for HTTP, `tokio` for asynchronous execution, and `serde` with `serde_json` for JSON handling. These tools fit the workflow well. Asynchronous requests help an application wait on network I/O without blocking a thread for the entire wait. Serde can map JSON keys to Rust fields. Neither library validates the business meaning of the data automatically.

## Parse deliberately, then validate meaning

JSON keys do not always look like idiomatic Rust identifiers. A Serde rename attribute can connect a key such as `"01. symbol"` to a field named `symbol`. A typed struct makes expected fields visible to the compiler and helps detect missing or incompatible fields during deserialization.

The section also shows indexing a `serde_json::Value` and calling `unwrap()`. That is concise for a sketch, but `unwrap()` can panic if a key is missing or has a different type. For a service that should remain available, prefer typed deserialization with a handled error, or use checked accessors that return an error when a field is absent. Then validate values: is the symbol the requested one, is the price parseable as a finite decimal, and does the timestamp meet the application’s freshness requirement?

Prices are often represented as decimal strings. Keeping the original string can preserve the provider’s displayed precision, but calculations need an explicit numeric representation and rounding policy. Binary floating-point values cannot represent every decimal fraction exactly. For financial arithmetic, use a decimal type or integer units appropriate to the data, and document scale and rounding. Do not silently turn a missing field into zero.

## Worked example: a small, auditable polling budget

Suppose a research script needs one request for each of 18 symbols, and its provider’s current plan allows at most 5 requests per minute. Assume requests are made serially, there are no retries, and the limit is enforced as a rolling or otherwise strict per-minute maximum. A 12-second gap between request starts is a nominal pacing interval, not a safety margin for this specific limit.

The arithmetic is:

$$60\ \text{seconds per minute} \div 5\ \text{requests per minute} = 12\ \text{seconds per request}.$$

For 18 requests, there are 17 gaps between consecutive request starts. With a 12-second gap, the time from the first start to the last start is:

$$17 \times 12 = 204\ \text{seconds} = 3\ \text{minutes}\ 24\ \text{seconds}.$$

This does not include network latency, response processing, or a provider’s exact windowing rules. If the provider counts completed requests, applies burst limits, or imposes a daily quota as well, this schedule may not be sufficient. The section’s example uses a 12-second delay to illustrate pacing for a stated free-tier limit of 5 requests per minute. Verify current limits rather than treating that example as a universal or permanent policy.

A production poller should centralize rate-limit policy instead of scattering fixed sleeps through request code. It should also interpret provider responses such as throttling errors, use bounded retries where appropriate, and avoid retrying non-recoverable failures forever. Backoff can reduce repeated pressure, but it does not grant permission to exceed the provider’s terms.

## Errors need categories and useful context

The section correctly points to network issues, rate limits, and format changes as likely challenges, and to Rust’s `Result` and `Option` as tools for handling failure. In practice, preserve enough context to tell these failures apart. Record the endpoint or operation, requested symbol, HTTP status, and a safe summary of the provider response. Do not log API keys or other secrets.

Check the actual status returned by the HTTP client before interpreting the body as a normal quote. The sample error-handling sketch illustrates this intent, but its shown `reqwest::Error::new` call is not a generally available way to construct an arbitrary error in current `reqwest` APIs. Prefer returning an error type that can represent an HTTP status, such as a custom application error, or use the client’s status-checking helpers and propagate their errors. In other words, handle the failure with the APIs available in the version you use rather than copying that constructor literally.

Also consider timeouts, cancellation, malformed JSON, absent fields, invalid numbers, and provider-side messages embedded in an otherwise successful response. A useful error message should identify what failed without exposing credentials or dumping sensitive data.

## From a quote fetcher to a research workflow

For scientific or financial research, fetching is only the beginning. Persist the raw response or a carefully governed equivalent when auditability requires it, alongside retrieval time and source metadata. Normalize observations into a documented schema. Track whether timestamps denote exchange time, provider time, or local retrieval time. These distinctions help reproduce a dataset and explain why two sources disagree.

A build pipeline introduces a related integrity question: after generating normalized data files or analysis reports, how can a team inventory those artifacts and compare their content later? A deterministic SHA-256 file manifest can record paths, byte sizes, and content hashes without modifying the files. It does not prove that the market data is accurate, authorized, or complete. It can, however, help identify whether a file changed between pipeline stages or research runs.

For example, a team can place approved build outputs in an artifacts directory, generate a manifest, and retain the JSON output with its run metadata. The manifest provides a compact content inventory. A separate process must decide where to store it, compare it, sign it, or enforce acceptance rules. This is a practical bridge from API ingestion to supply-chain integrity, not a feature attributed to the book’s discussion of APIs.

## Assumptions, pitfalls, and sensible boundaries

A quote is an observation with a source and a time, not timeless truth. Market hours, exchange coverage, currency, corporate actions, provider delay, and licensing terms can affect how it should be interpreted. A successful fetch does not establish suitability for trading, valuation, or publication.

Keep API credentials out of source control. Validate user-provided symbols before constructing requests, and use URL-encoding mechanisms rather than inserting arbitrary input into a URL string. Reuse a configured HTTP client where appropriate, and set operational limits such as timeouts. Confirm the provider’s current authentication, usage, and redistribution requirements.

Finally, distinguish technical validity from financial interpretation. Parsing `"187.42"` into a decimal means only that the text represents a number under the chosen parsing rules. It does not establish the price’s market, currency, freshness, or correctness. Build those checks into the data model and tests.

## Exercises with short answers

1. **A response has HTTP status 200 but contains a quota notice instead of quote fields. Is it a successful quote?** No. HTTP success describes the transport-level response, not the application-level meaning. Inspect and classify the body.

2. **Why is a 12-second pause not a guarantee of compliance with every provider’s limits?** Limits can use different windows and may include burst, daily, or account-wide quotas. Follow the current provider rules and handle explicit throttling responses.

3. **A JSON field is missing. Should the parser substitute zero?** Usually not. Zero is a meaningful value and would hide a schema or data-quality problem. Return a clear validation error or represent absence explicitly.

4. **What does a SHA-256 manifest establish about an artifact?** It records a content digest for comparison. By itself it does not establish who created the file, whether it is trustworthy, or whether its contents are correct.

Reliable API work is less about getting a number onto the screen and more about preserving the meaning of that number as it travels through software. Rust’s types and error handling can make assumptions visible. Provider documentation, validation, careful metadata, and reproducible artifacts complete the job.

---

Published 2026-10-16.

**Source:** Rust Programming for your First Finance Job- A Comprehensive -- Publishing, Reactive & Van Der Post, Hayden, section **7.2 Fetching Market Data from Public APIs**.

This is an original explanatory note; the source book is not redistributed.

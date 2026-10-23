# Key concepts by sample

## 01 – Basic HTTP GET
**HTTP Request Defaults** hold the protocol, host and timeouts once so each sampler only carries a path. Two **Response Assertions** run on the response: one checks the status code is 200, the other checks the body contains a known substring. A sampler is marked failed only when an assertion fails, not just when it returns a non-2xx code. That is why assertions are the real definition of "correct".

## 02 – REST CRUD
Covers GET, POST, PUT, PATCH and DELETE with raw JSON bodies (`postBodyRaw`) and a shared **HTTP Header Manager** that sets `Content-Type: application/json`. The **JSON Assertion** (JSONPath) checks *values*, such as `$.title == perf-test`, not only status codes. Note: jsonplaceholder fakes writes, so the data is never persisted.

## 03 – Correlation
Correlation means capturing a dynamic value from one response and sending it in a later request. In real systems this is session tokens, CSRF tokens or account IDs. The **JSON Extractor** pulls `$.id`, and with match number `0` it picks a random post id from an array. The **Regular Expression Extractor** pulls `username` with `"username":\s*"([^"]+)"`, which is useful when the response is not JSON (HTML, headers, SOAP). Always set a default such as `NOT_FOUND` so a broken correlation is easy to see.

## 04 – CSV parameterisation
**CSV Data Set Config** reads `data/users.csv`, one row per iteration, and takes the variable names from its header row. A relative path is resolved against the `.jmx` folder. Use `-JcsvFile=/path/other.csv` to swap the data. Sharing mode "All threads" gives every thread a unique row, and Recycle on EOF loops the file. The CSV also holds the *expected* values, so the assertions are data-driven too.

## 05 – Think time and pacing
- **Uniform Random Timer** (500–1500 ms) adds realistic, varied user think time.
- **Constant Timer** adds a fixed delay.
- **Constant Throughput Timer** sets pacing in samples per *minute* (30/min, shared across all active threads). It controls the arrival rate no matter how fast the server responds.
- **Throughput Controller** (50%) sets the transaction mix, so only half the iterations open the "detail" page.

Timers apply to every sampler in their scope and run *before* the sampler.

## 06 – Properties-driven load
`${__P(name,default)}` reads a JMeter property, which you set with `-Jname=value`. One script can then serve smoke, load and soak runs from the command line or CI with no edits. Threads, ramp-up, duration (scheduler with infinite loops), host, protocol and think time are all properties. Values are stored as `stringProp` so the functions get resolved.

## 07 – Transactions + JSR223 Groovy
A **Transaction Controller** groups requests into a business transaction (TX01 login/view, TX02 payment). That gives end-to-end timings per business step, which is what SLAs are written against. **JSR223 + Groovy** is the recommended scripting option because it compiles and caches (`cacheKey`):
- the PreProcessor builds a unique `txnRef` and a JSON payload with `JsonOutput`
- the PostProcessor parses the response with `JsonSlurper`, stores `createdId` and fails the sample if the ref is not echoed back
- a JSR223 Assertion validates the extracted id

## 08 – Logic controllers
- **If Controller** with "interpret as variable expression" and `__groovy()` (faster than JavaScript evaluation).
- **Loop Controller** uses the built-in `__jm__<name>__idx` counter.
- **While Controller** paginates with a counter that a Groovy PostProcessor increments, until the condition becomes false.
- `SampleResult.setIgnore()` keeps the helper JSR223 Sampler out of the results.

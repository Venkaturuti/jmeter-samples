# JMeter 5.6.3 Learning Samples

[![JMeter samples](https://github.com/Venkaturuti/jmeter-samples/actions/workflows/jmeter.yml/badge.svg)](https://github.com/Venkaturuti/jmeter-samples/actions/workflows/jmeter.yml)

A curated set of runnable Apache JMeter 5.6.3 test plans that show the core techniques of performance test engineering: assertions, correlation, parameterisation, pacing, property-driven workload models, business transactions and Groovy scripting.

Every plan uses **only core JMeter components (no plugins)** and targets public demo APIs ([JSONPlaceholder](https://jsonplaceholder.typicode.com) and [httpbin](https://httpbin.org)). Loads are kept deliberately tiny (1–3 threads, short durations) out of respect for these free services. Every sample runs in CI on each push.

## Samples

| # | Test plan | What it teaches |
|---|-----------|-----------------|
| 01 | [`01_basic_http_get.jmx`](samples/01_basic_http_get.jmx) | HTTP Request Defaults, a GET sampler, Response Assertions on code and body |
| 02 | [`02_rest_crud.jmx`](samples/02_rest_crud.jmx) | REST GET/POST/PUT/PATCH/DELETE with JSON bodies, Header Manager, JSON Assertions |
| 03 | [`03_correlation.jmx`](samples/03_correlation.jmx) | Correlation with JSON Extractor and Regular Expression Extractor, chaining values across requests and hosts |
| 04 | [`04_csv_parameterisation.jmx`](samples/04_csv_parameterisation.jmx) | CSV Data Set Config ([`data/users.csv`](samples/data/users.csv)), data-driven requests and assertions |
| 05 | [`05_think_time_pacing.jmx`](samples/05_think_time_pacing.jmx) | Uniform Random and Constant Timers, Constant Throughput Timer pacing, Throughput Controller mix |
| 06 | [`06_properties_driven.jmx`](samples/06_properties_driven.jmx) | Workload model driven by `-J` properties: users, ramp-up, duration, host |
| 07 | [`07_transactions_groovy.jmx`](samples/07_transactions_groovy.jmx) | Transaction Controllers, JSR223 Groovy Pre/PostProcessor and Assertion |
| 08 | [`08_logic_controllers.jmx`](samples/08_logic_controllers.jmx) | If, Loop and While Controllers with `__groovy()` conditions |

Each concept is explained in more depth in [`docs/CONCEPTS.md`](docs/CONCEPTS.md).

## Running locally

**Prerequisites:** Java 8+ (17 recommended) and [Apache JMeter 5.6.3](https://jmeter.apache.org/download_jmeter.cgi), with `JMETER_HOME/bin` on your `PATH`.

### GUI mode (for building and debugging)

```bash
jmeter -t samples/01_basic_http_get.jmx
```

Add a *View Results Tree* listener while debugging. Remove or disable it before you run any load.

### Non-GUI mode (for running tests)

```bash
jmeter -n -t samples/02_rest_crud.jmx -l results/02.jtl -e -o report/02
```

| Flag | Meaning |
|------|---------|
| `-n` | non-GUI mode |
| `-t` | test plan |
| `-l` | results file (JTL) |
| `-j` | JMeter log file |
| `-e -o <dir>` | generate the HTML dashboard into an empty or non-existent folder |
| `-J<name>=<value>` | set a JMeter property, read by `${__P(name,default)}` |

### Properties-driven run (sample 06)

```bash
jmeter -n -t samples/06_properties_driven.jmx \
  -Jusers=3 -Jrampup=10 -Jduration=60 \
  -Jhost=jsonplaceholder.typicode.com -JthinkMs=1500 \
  -l results/06.jtl -e -o report/06
```

| Property | Default | Purpose |
|----------|---------|---------|
| `users` | 2 | number of threads |
| `rampup` | 5 | ramp-up in seconds |
| `duration` | 30 | test duration in seconds |
| `host` / `protocol` | jsonplaceholder.typicode.com / https | target system |
| `thinkMs` | 1000 | think time per request |

Sample 04 also accepts `-JcsvFile=<path>` to use a different data file.

On Windows, use `jmeter.bat` and replace the `\` line continuations with `^` (cmd) or a backtick (PowerShell). In Git Bash, the commands above work as shown.

## Continuous integration

[`.github/workflows/jmeter.yml`](.github/workflows/jmeter.yml) runs on every push to `main`, on every pull request and on demand (`workflow_dispatch`):

1. A **matrix** job is created per sample (at most 2 run in parallel).
2. `actions/checkout` and `actions/setup-java` (Temurin 17) prepare the runner. JMeter 5.6.3 is downloaded from `dlcdn.apache.org` (with an archive fallback) and cached.
3. Each plan runs non-GUI with small `-J` values.
4. The JTL is checked, and the job **fails if any sample failed**.
5. The HTML dashboard, JTL and log are uploaded as the `report-<sample>` artifact.

## Repository layout

```
.
├── .github/workflows/jmeter.yml   # CI: matrix run + HTML report artifacts
├── docs/CONCEPTS.md               # key concept behind each sample
├── samples/
│   ├── 01_basic_http_get.jmx
│   ├── ...
│   ├── 08_logic_controllers.jmx
│   └── data/users.csv             # test data for sample 04
├── .gitignore                     # excludes *.jtl, *.log, results/, report/
├── LICENSE                        # MIT
└── README.md
```

## Good practices shown

- Assertions on every request. A 200 response is not the same as a correct response.
- No listeners that are heavy on memory during load. Results go to a JTL and the HTML dashboard is built from it.
- Groovy for all scripting (JSR223 with caching), not BeanShell.
- Workload parameters set outside the script, so the same plan serves smoke, load and soak runs.
- Sensible connect and response timeouts in HTTP Request Defaults.

## Author

**Venkat Uruti**, AVP and Performance Testing Lead with 15+ years in banking and financial services performance engineering.

## License

[MIT](LICENSE)

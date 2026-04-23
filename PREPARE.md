# Evaluation Setup

This file is outside the editable surface. It defines how results are judged. Agents cannot modify the evaluator or the scoring logic — the evaluation is the trust boundary.

Consider defining more than one evaluation criterion. Optimizing for a single number makes it easy to overfit and silently break other things. A secondary metric or sanity check helps keep the process honest.

eval_cores: 1
eval_memory_gb: 1.0
prereq_command: pnpm run build

## Setup

Install dependencies and prepare the evaluation environment:

```bash
pnpm install
```

The project requires TypeScript compilation before benchmarking. The `prereq_command` is set to `pnpm run build`, which compiles TypeScript source files in `src/` to JavaScript in `dist/` using unbuild. The CLI runs this before the evaluation harness during recovery, ensuring it measures compiled output rather than stale artifacts.

The benchmark imports from the built artifacts in `dist/index.mjs`, so any source code changes in `src/` must be compiled before measurement.

## Run command

```bash
node test/benchmark.mjs
```

## Output format

The benchmark must print `METRIC=<number>` to stdout.

## Metric parsing

The CLI looks for `METRIC=<number>` or `ops_per_sec=<number>` in the output.

## Ground truth

The baseline metric represents the harmonic mean of operations per second across 12 core URL manipulation functions in the ufo library:

1. **parseURL** - Parse URL strings into structured objects
2. **stringifyParsedURL** - Stringify parsed URL objects back to strings
3. **parseQuery** - Parse query strings into objects
4. **stringifyQuery** - Stringify query objects to strings
5. **joinURL** - Join URL segments
6. **normalizeURL** - Normalize URL encoding and structure
7. **withQuery** - Add/replace query parameters
8. **hasProtocol** - Check for URL protocol
9. **cleanDoubleSlashes** - Remove double slashes
10. **withBase** - Add base path prefix
11. **withoutBase** - Remove base path prefix
12. **isEqual** - Compare URLs for equality

Each function is benchmarked with 100,000 iterations after a 1,000-iteration warmup. The benchmark uses a diverse set of test URLs covering different protocols, structures, and edge cases.

The harmonic mean is used as the composite metric because it better represents overall performance than arithmetic mean - it is more sensitive to slow operations, ensuring that improvements must be balanced across all functions rather than just optimizing the fastest ones.

**Baseline measurement**: ~807,000 ops/sec (harmonic mean) as measured on initial setup. Individual function performance ranges from ~213K ops/sec (normalizeURL, the slowest) to ~23M ops/sec (hasProtocol, the fastest).

## Secondary validation

All changes must pass the existing test suite:

```bash
pnpm test
```

This ensures functional correctness is maintained while pursuing performance improvements. The test suite includes comprehensive coverage of URL parsing, query handling, encoding, and edge cases.

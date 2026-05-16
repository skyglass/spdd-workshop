## Workshop Extension: Timestamp Placeholder Resolution

When resolving `@` file references, support workshop filenames that contain a
timestamp placeholder.

- If a referenced path contains `{timestamp}`, replace `{timestamp}` with `*`
  and treat the result as a file pattern rooted at the current project directory.
- Sort matching paths lexicographically and use the first matching file.
- If no files match, stop and report the unresolved input pattern.
- Apply this before reading referenced files or deriving API test scenarios.

Example:

```text
$spdd-api-test @./spdd/prompt/GGQPA-XXX-{timestamp}-[Feat]-api-token-usage-billing.md
```

If `./spdd/prompt/GGQPA-XXX-202603131758-[Feat]-api-token-usage-billing.md`
is the first matching file, use that concrete file as the input context.

## Workshop Extension: Idempotent API Test Scripts

Generate `scripts/test-api.sh` so it can be run repeatedly against the same
local application and database without changing expected results or failing
because of state left by a previous run.

Script requirements:

- Add an idempotency note near the top of the generated script that states the
  script cleans test data and is safe to run multiple times against the same
  local environment.
- Add a cleanup/reset function that removes data created by API test cases and
  restores mutable seed data used by those cases.
- Preserve initial data loaded by Flyway migrations. Do not truncate or delete
  baseline Flyway seed rows such as configured customers, plans, subscriptions,
  model pricing, quotas, or other reference data needed by the application.
- Cleanup may delete only records created by the API test script, such as bills,
  usage records, or other generated test resources.
- If a test mutates Flyway-provided seed rows, cleanup must restore those rows to
  their original Flyway values instead of removing them.
- Run cleanup before the first test case so leftovers from previous script runs
  cannot affect the current run.
- Run cleanup between test cases whenever one test mutates state that can affect
  another test, such as bills, usage totals, quotas, subscriptions, or generated
  resources.
- Run cleanup at the end of the script as best effort.
- Cleanup should use the most appropriate project mechanism available:
  dedicated reset/delete API endpoint, test-only cleanup endpoint, SQL cleanup
  command, fixture reload command, or Docker/Gradle database reset command.
- SQL cleanup must be scoped with stable predicates, such as test-created ids,
  timestamps, or known generated values, so Flyway baseline rows are preserved.
- If the project has no cleanup mechanism, generate one in the script using the
  available local environment instead of weakening assertions.
- It is valid to assert exact totals, counts, balances, and accumulated values
  when cleanup has reset the relevant data before the assertion.
- Do not require manual cleanup between runs.
- Repeated successful runs of the script must exit 0.

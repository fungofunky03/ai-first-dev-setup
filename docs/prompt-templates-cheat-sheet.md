# Prompt templates cheat sheet

Ready-to-use prompt scaffolds for common engineering tasks. Copy a block,
replace the bracketed `[like this]` sections, and paste it into your IDE
chat with the relevant files attached as context.

These are deliberately short. They are the quick-reference cousins of the
full [prompts library](../prompts/README.md): where a prompt file in
`prompts/<category>/` gives you a tested, opinionated prompt with a
worked example and personalisation tips, the templates here are bare
skeletons you fill in on the fly. Each section links to the fuller prompt
when one exists — reach for that when the quick template isn't enough.

> **How to read a template.** Every `[bracketed]` span is a slot you
> replace. Numbered lists are the steps you're asking the assistant to
> take; keep the ones that apply, delete the rest. The more specific your
> substitutions (exact file paths, real error messages, concrete
> acceptance criteria), the better the result.

## Bug fixes

For anything you can't immediately explain, prefer the hypothesis-driven
[`prompts/debug/investigate-bug.md`](../prompts/debug/investigate-bug.md)
over jumping straight to a fix. Use the template below when the cause is
already understood and you just want the fix plus a regression test.

### Fix a specific bug

```
Fix the bug where [describe the bug behavior].

Steps to reproduce:
1. [Step 1]
2. [Step 2]
3. [Step 3]

Expected behavior: [what should happen]
Actual behavior: [what actually happens]

Please:
1. Investigate the root cause in [relevant file/directory]
2. Implement a fix that addresses the root cause, not the symptom
3. Add a regression test that fails before the fix and passes after
4. Run the existing test suite to confirm no regressions
```

### Investigate a production issue

```
Users are reporting [describe the issue] in production.

Please:
1. Use the [Sentry/DataDog/log monitoring] MCP to pull recent errors and stack traces
2. Identify the root cause
3. Implement a fix
4. Add error handling to prevent similar failures
5. Add a regression test
6. Link the monitoring alert in the PR description
```

See also:
[`prompts/debug/root-cause-analysis.md`](../prompts/debug/root-cause-analysis.md).

## Feature implementation

For anything large enough to justify a written spec, run
[`workflows/spec-driven-development.md`](../workflows/spec-driven-development.md)
first. For small, well-scoped changes, use
[`workflows/feature-development.md`](../workflows/feature-development.md)
and the templates below.

### Add a new API endpoint

```
Create a new API endpoint [endpoint path] that [describe what it does].

Requirements:
- Method: [GET/POST/PUT/DELETE]
- Request body: [describe request structure]
- Response format: [describe response structure]
- Authentication: [describe auth requirements]

Please:
1. Reference the existing [similar endpoint file] for patterns
2. Implement the endpoint following our existing conventions
3. Add input validation and error handling
4. Write unit tests for the new endpoint
5. Update API documentation if applicable
6. Run the test suite to confirm everything passes
```

### Add a new UI component

```
Add a new [component type] component in [file/location].

Requirements:
- Component name: [ComponentName]
- Props: [list props and their types]
- Functionality: [describe what it should do]
- Styling: use [existing component/library] as a reference

Please:
1. Create the component following our existing patterns
2. Implement the required functionality
3. Add proper TypeScript types
4. Style it to match our design system
5. Add unit tests for the component
6. Integrate it into [parent component/page]
7. Test it manually by running the dev server
```

### Implement a feature from a design

```
Implement [feature name] from this design: [link to design].

Focus on the [specific frame/section] frame.

Requirements:
- Use our existing components from [component library path]
- Follow the styling in [design system file]
- Ensure responsive layout at [breakpoint 1] and [breakpoint 2]

Please:
1. Implement the feature following the design specifications
2. Reuse existing components where possible
3. Test at desktop (1440px) and mobile (375px) widths
4. Take screenshots to verify it matches the design
5. Do not open a PR until it visually matches the design
```

## Refactoring

Full prompts:
[`prompts/refactor/reduce-complexity.md`](../prompts/refactor/reduce-complexity.md),
[`prompts/refactor/extract-component.md`](../prompts/refactor/extract-component.md),
[`prompts/refactor/migrate-pattern.md`](../prompts/refactor/migrate-pattern.md).

### Refactor a module

```
Refactor [module/file name] to improve [maintainability/performance/readability].

Current issues:
- [Issue 1]
- [Issue 2]

Requirements:
- Keep all existing functionality intact
- Follow the patterns in [reference file]
- Improve [specific metric: cyclomatic complexity/latency/bundle size]

Please:
1. Analyze the current implementation
2. Refactor following best practices
3. Confirm all existing tests still pass
4. Add tests for any new functions introduced
5. Run the full test suite
6. Report the improvement against the metric above if measurable
```

### Convert to a new pattern

```
Convert [file/directory] to use [new pattern/library/framework].

Reference: [link to documentation or example file]

Requirements:
- Maintain all existing functionality
- Follow the conventions in [example file]
- Update any dependent code

Please:
1. Review the documentation and examples
2. Convert the code step by step
3. Update imports and dependencies
4. Confirm all tests pass
5. Run [build command] to verify no errors
6. Test the functionality manually
```

## Testing

Full prompts:
[`prompts/tests/unit-tests-from-fn.md`](../prompts/tests/unit-tests-from-fn.md),
[`prompts/tests/edge-case-finder.md`](../prompts/tests/edge-case-finder.md),
[`prompts/tests/e2e-scenarios.md`](../prompts/tests/e2e-scenarios.md).

### Add test coverage

```
Add test coverage for [file/module/function].

Current coverage: [current %]
Target coverage: [target %]

Please:
1. Analyze the code to identify edge cases
2. Write unit tests for all public methods
3. Add integration tests if applicable
4. Reference [existing test file] for testing patterns
5. Run the coverage report and confirm it meets the target
6. Confirm all tests pass
```

### Debug failing tests

```
Fix the failing tests in [test file or directory].

Failures:
- [Test name 1]: [error message]
- [Test name 2]: [error message]

Please:
1. Investigate why these tests are failing
2. Determine whether the tests or the implementation are wrong
3. Fix the root cause
4. Confirm all tests in the suite pass
5. Run the full test suite to check for regressions
```

## Documentation

Full prompts:
[`prompts/docs/api-docs.md`](../prompts/docs/api-docs.md),
[`prompts/docs/readme-generator.md`](../prompts/docs/readme-generator.md),
[`prompts/docs/changelog-from-commits.md`](../prompts/docs/changelog-from-commits.md).

### Document a module

```
Add documentation to [file/module].

Please:
1. Add JSDoc/TypeDoc comments to all public functions
2. Document parameters, return values, and thrown errors
3. Add usage examples for the non-obvious functions
4. Create a README if this is a new module
5. Follow the documentation style used in [reference file]
6. Update the main API documentation if applicable
```

### Update API documentation

```
Update the API documentation for [endpoint/function].

Changes made:
- [Change 1]
- [Change 2]

Please:
1. Update the [OpenAPI/Swagger] specification
2. Update any inline code comments
3. Add usage examples if the behavior changed
4. Update [documentation file]
5. Verify the documentation builds successfully
```

## Performance

Full prompt:
[`prompts/code-review/performance-review.md`](../prompts/code-review/performance-review.md).

### Optimize database queries

```
Optimize the database queries in [file/module].

Problems:
- [Specific query] is slow (takes [time])
- [Specific operation] causes N+1 queries

Please:
1. Analyze the query execution plans
2. Add appropriate indexes to [table/column]
3. Refactor N+1 access into joins or batched loads
4. Benchmark before and after
5. Confirm all tests still pass
6. Document the improvement
```

### Optimize frontend performance

```
Optimize the performance of [component/page].

Symptoms:
- Slow initial load
- Large bundle size
- Unnecessary re-renders

Please:
1. Analyze the bundle with [bundle analyzer]
2. Code-split [large module]
3. Add memoization where it measurably helps
4. Optimize images and assets
5. Lazy-load components below the fold
6. Measure the improvement with Lighthouse
7. Confirm functionality is unchanged
```

## Security

Full prompt:
[`prompts/code-review/security-audit.md`](../prompts/code-review/security-audit.md).

### Fix a security vulnerability

```
Fix the security vulnerability in [file/module].

Type: [e.g., SQL injection, XSS, CSRF]
Severity: [High/Medium/Low]

Please:
1. Review the advisory: [link]
2. Implement the recommended fix
3. Add input validation and sanitization
4. Add a security test to prevent regression
5. Run the security audit: [audit command]
6. Check whether the same class of issue exists elsewhere
```

### Add security headers

```
Add security headers to the [application/API].

Required headers:
- [Header 1]: [value]
- [Header 2]: [value]

Please:
1. Configure the headers in [config file]
2. Verify they are set using [tool/method]
3. Confirm existing functionality is not broken
4. Document the change
```

## Migration and upgrades

### Upgrade a dependency

```
Upgrade [package] from [old version] to [new version].

Please:
1. Review the changelog for breaking changes: [link]
2. Update the dependency in [package.json/requirements.txt/go.mod]
3. Update any deprecated API usage
4. Run the migration script if applicable: [command]
5. Run all tests to confirm compatibility
6. Test the application manually
7. Update documentation if APIs changed
```

### Migrate to a new service

```
Migrate from [old service] to [new service].

Reference: [link to new service docs]

Please:
1. Set up the new service following the documentation
2. Migrate existing data and configuration
3. Update all code to use the new service
4. Reference [example file] for implementation patterns
5. Run integration tests to verify functionality
6. Roll out gradually and monitor for issues
7. Decommission the old service after verification
```

## Code review

Full prompt:
[`prompts/code-review/pr-review.md`](../prompts/code-review/pr-review.md).

### Review a pull request

```
Review the pull request: [PR link or number].

Focus areas:
- Correctness and maintainability
- Performance implications
- Security considerations
- Test coverage
- Documentation

Please:
1. Review each changed file
2. Leave specific, actionable comments
3. Verify the changes match the PR description
4. Check for missed edge cases and error handling
5. Confirm the tests are adequate
6. Approve or request changes with clear reasoning
```

## General purpose

### Research and implement

```
I need to implement [feature/functionality] using [technology/library].

Please:
1. Research best practices for [technology/library]
2. Find and review the documentation: [expected doc sources]
3. Look at open-source examples if applicable
4. Propose an approach before implementing
5. Implement the solution following best practices
6. Add tests and documentation
7. Verify it works as expected
```

### Debug and fix

```
Something is wrong with [feature/component].

Symptoms:
- [Symptom 1]
- [Symptom 2]

Please:
1. Investigate the issue in [relevant files]
2. Add logging as needed
3. Identify the root cause
4. Implement a fix
5. Test the fix thoroughly
6. Remove any temporary debugging code
7. Confirm no regressions
```

---

**For recurring tasks**, don't retype these every time. Promote the ones
you use most into your own tested prompt files under
[`prompts/`](../prompts/README.md) (follow the schema in
[`prompts/README.md`](../prompts/README.md)), or into an end-to-end
[`workflow`](../workflows/) if the task spans several steps.

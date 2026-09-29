# Discard Rationale by Zone

Why each zone in the Infrastructure zone classification table in `candidate-criteria.md` is discarded. Read this when a discard decision is contested — do not re-derive the reasoning. Example strings per zone are in `examples.md`.

## Infrastructure / setup

If setup fails, tests cannot execute. A passing test run **is** the proof that setup worked, so automating setup verification spends effort to learn nothing.

**Migration runners are infrastructure code even when written in C#.** Their purpose is to prepare the environment for functional tests. A test suite that runs and passes proves schemas exist — if a schema were missing, every downstream test touching that schema would fail. A dedicated schema-presence test is redundant.

## File / asset presence

These verify deployment correctness, not application behaviour. They are infrastructure and DevOps concerns, not functional guarantees.

## Static analysis / linting

Static analysis tools already do this, and they do it on every file rather than the few a test would cover. Tests should verify behaviour, not re-run linters.

## Schema / type validation

The fact that code compiles and tests execute proves the schema is correct. If a field did not exist, the code would not compile.

## DI / composition

The DI container is third-party code. Verify that your domain logic uses the dependency correctly, not that the container resolves it. If resolution fails, the application will not start and every functional test fails loudly.

## Framework integration

These test third-party code, not your domain. Verify that **your** code uses the framework correctly — "aggregate is persisted correctly" is your code; "ORM can persist" is theirs. If framework usage is wrong, business logic tests will fail anyway.

## One-off data / seed operations

These are validation steps, not functional tests. If the seed or migration is wrong, it is caught when business logic that depends on it fails. If the business logic tests pass, the seed worked.

## Pattern repetition

Enumerating a pattern across N instances adds no additional confidence. If the implementation is correct for one instance, the shared code guarantees it is correct for all. The only instances worth testing separately are those with genuinely different behaviour.

## The underlying principle

All eight zones reduce to one test: if the test would still pass after replacing all your business logic with a stub that returns a hardcoded value, the test is not exercising your domain. Automation effort spent there buys confidence in the compiler, the container, the framework, or the deployment — none of which is what QA is accountable for.

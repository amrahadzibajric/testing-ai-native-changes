# Discard Examples by Zone

Concrete test-name and assertion examples for each zone in the Infrastructure zone classification table in `candidate-criteria.md`. Use these to recognise a candidate that belongs to a discard zone. The zone table is authoritative — this file only supplies examples.

## Infrastructure / setup

Initial setup, infrastructure prerequisites, and infrastructure code. Includes stories where code is written in a general-purpose language (C#, TypeScript) but whose purpose is environment setup rather than business behaviour delivery.

- "Application starts without error"
- "Database connection pool is initialized"
- "Message queue is accessible"
- "Data migrations complete successfully"
- "Migration runner creates expected schema"
- "Migration runner is idempotent when run twice"
- Any other "smoke test" that confirms the environment is ready

## File / asset presence

Verification that files exist in the right place.

- "Static stylesheet is deployed at expected path"
- "Configuration file is readable from expected location"
- "Health endpoint returns success status"
- "Required dependency is available on filesystem"

## Static analysis / linting

Checks that code meets syntactic or style standards.

- "All imports are used"
- "No undefined variables"
- "Naming conventions are followed"

## Schema / type validation

Checks that types and classes are defined as expected.

- "Handler can be resolved by dependency container"
- "Enum constant is defined in type"
- "Method implements expected interface correctly"
- "Required field is defined on entity"

## DI / composition

Checks that the dependency injection container works.

- "Component can be constructed by dependency container"
- "Service receives correct implementation instance"
- "Factory method produces correct type instance"

## Framework integration

Checks that third-party frameworks work as documented.

- "ORM can persist and retrieve an entity"
- "Message broker can publish and receive a message"
- "JSON serializer can convert objects to/from JSON"
- "Middleware processes requests in correct sequence"

Contrast: "aggregate is persisted correctly" is your code and in scope; "ORM can persist" is third-party code and out of scope.

## One-off data / seed operations

Operations that apply once and never repeat.

- "System administrator is created during initialization"
- "Reference data is seeded into database"
- "Default values are populated during setup"

Note: `candidate-criteria.md` Step 2 lists *one-off data migration correctness* as a **Manual** designation rather than a discard. Discard the schema/seed-presence assertion; if the migration's correctness itself must be checked once by a human, record it as a manual case with justification.

## Pattern repetition

N services, handlers, or runners that all follow the same structural pattern. Do not write N identical test cases.

- `VerifyContractsMigratorCreatesContractsSchema`, `VerifyNominationsMigratorCreatesNominationsSchema`, … (×4)
- `VerifyContractsHandlerValidatesRequest`, `VerifyNominationsHandlerValidatesRequest`, … (×N)

Use a parameterized test or test one representative case. Instances with genuinely different behaviour are worth testing separately; different names or connection strings are parameters, not behaviour differences.

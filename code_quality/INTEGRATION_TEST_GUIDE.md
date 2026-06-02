# Integration Test Guide

This guide defines integration-testing expectations for projects that contain
executable code. It is intentionally complementary to the unit test guide: use
it to shape tests that verify how components collaborate across real boundaries,
not as a replacement for unit or end-to-end testing strategy.

## Source of truth

Use this guide together with:

- [Unit Test Guide](./UNIT_TEST_GUIDE.md)
- [Code Quality Control Guide](./CODE_QUALITY_CONTROL_GUIDE.md)
- [Clean Architecture Guide](./CLEAN_ARCHITECTURE_GUIDE.md)
- [Low Coupling Guide](./LOW_COUPLING_GUIDE.md)

## Goal

Create tests that verify how two or more components collaborate across a real
boundary — database, message queue, filesystem, cache, or external adapter —
run in a repeatable environment, and fail with clear, localized signals.

## What Integration Testing Means

Integration testing verifies the behavior at the seam between components when
at least one real external boundary is involved.

An integration test is usually one of these:

1. a service and its real database executing a query or transaction
2. a repository and a live schema verifying reads, writes, and constraints
3. an HTTP client adapter sending requests to a test server or recorded fixture
4. a producer and consumer exchanging messages through a real queue or broker
5. two domain collaborators exercising a shared contract end-to-end within
   one context

The exact scope varies by project. What matters is the testing boundary:

- at least one real external dependency participates
- the test verifies behavior across the seam, not just inside one unit
- the environment is controlled and repeatable
- assertions describe observable outcomes at the boundary, not internal
  implementation details

## Integration Tests vs Other Test Types

| Test type | Scope | Speed | External dependencies | Main question |
| --- | --- | --- | --- | --- |
| Unit | One focused behavior | Fast | Replaced or isolated | "Does this unit behave correctly?" |
| Integration | Several collaborating components | Moderate | Some real boundaries | "Do these parts work together?" |
| End-to-end | Whole system or workflow | Slowest | Real runtime stack | "Does the user-visible flow work?" |

Good test suites use all three where appropriate. Integration tests close the
gap between fast unit feedback and slow end-to-end confidence.

## Why It Matters

1. It catches defects at the boundary where real dependencies meet, which unit
   tests cannot reach by design.
2. It validates that schemas, queries, and external contracts match what the
   code expects.
3. It exercises error paths that only appear when real I/O is involved, such as
   constraint violations, timeout behavior, and connection failures.
4. It provides a safety net for configuration drift, where environment changes
   break integrations that unit tests never exercise.
5. It gives more confidence than mocks alone when contracts with external
   systems are critical.

## Integration Testing and Code LLMs

Integration testing also improves the quality of LLM-assisted coding.

Code-focused models benefit from a clear separation between unit and integration
tests because it keeps the expected scope of any given test unambiguous. When
real boundaries are tested consistently and the environment setup is explicit,
a model editing integration-sensitive code can run the relevant suite and get
reliable feedback without accidentally treating integration coverage as unit
coverage or vice versa.

### Why LLMs Benefit

- Explicit real-boundary tests reveal which components are externally coupled.
- Controlled test environments reduce false positives from environment drift.
- Separate test suites let a model know which tests to run after touching an
  adapter vs. a domain rule.
- Seeded, reset state between tests prevents cascading failures that confuse
  root-cause analysis.
- Clear scope keeps test output focused on the actual integration failure.

### Where Weak Integration Tests Hurt LLMs

- Tests that mix real and mocked boundaries hide which layer actually failed.
- Shared mutable database state between tests causes non-deterministic failures
  that cannot be reliably reproduced or fixed.
- Slow integration tests bundled with unit tests cause developers (and models)
  to skip running them.
- Missing integration coverage lets incorrect assumptions about external
  contracts survive until production.

## Quality Gates

Every substantive code change that crosses a real boundary should satisfy these
gates.

### 1. Scope gate

- An integration test must involve at least two collaborating components and
  at least one real external boundary.
- Pure logic that can be isolated belongs in unit tests, not integration tests.
- Full end-to-end user flows belong in the end-to-end suite, not here.
- Name the test scope clearly so the reader knows which boundary is under test.

### 2. Real-boundary gate

- The boundary being tested must be real: a live database, a running service, a
  real queue, a real filesystem, or a live in-process adapter.
- The only acceptable substitutes are controlled test doubles that replicate the
  full protocol — for example, a local SMTP server, an in-memory SQLite
  instance, or a recorded HTTP fixture.
- Do not replace the boundary with a mock and call the test an integration test.

### 3. Determinism gate

- The same test with the same inputs should produce the same result every run
  regardless of execution order.
- Seed data must be explicit and scoped to the test or test group.
- State from previous tests must be cleaned up before each test starts.
- Do not rely on leftover state created by other tests.

### 4. Environment gate

- Integration tests should run against a controlled, local environment.
- Use containers, in-memory services, or embedded servers rather than shared
  staging or production resources.
- Configuration for the test environment must be explicit — environment
  variables, test config files, or container setup — not implicit from a
  developer's local configuration.
- The environment must be reproducible on any development machine and in CI.

### 5. Cleanup gate

- Every test that writes state must restore the system to a known state after
  it runs, whether it passes or fails.
- Prefer transaction rollback, container reset, or explicit teardown over
  leaving orphaned records.
- Cleanup must be unconditional, not only on success paths.

### 6. Contract gate

- Assert the full observable behavior at the boundary: return values, persisted
  state, emitted messages, error codes, and side effects the caller depends on.
- Do not limit assertions to the happy path when the error contract is equally
  load-bearing.
- Verify that schema, serialization, and encoding assumptions hold at the
  boundary, not just in internal types.

### 7. Error-path gate

- Test failure modes that only appear at the real boundary: constraint
  violations, duplicate keys, connection refusal, timeouts, malformed
  responses, and authentication failures.
- Verify that the integration layer translates these failures into the error
  types the domain layer expects.
- Do not assume the happy path is enough.

### 8. Execution gate

- Integration tests are expected to be slower than unit tests, but they must
  not be arbitrarily slow.
- Prefer containers that start once per suite over containers that restart per
  test.
- Keep the integration suite runnable locally within a reasonable time so
  developers actually run it before merging.
- Separate integration tests from unit tests in the project's test commands so
  each suite can be run independently.

## Positive Signals

- Each test names the boundary or collaborator it exercises.
- Test setup is explicit about what state the environment starts in.
- Failures in one test do not cascade into unrelated tests.
- The integration suite can be run in isolation with a single command.
- Error paths at the boundary are covered as deliberately as happy paths.
- Schema changes cause the relevant integration tests to fail immediately.
- Containers or embedded services start once and are shared within the suite
  for speed.

## Warning Signs

- An "integration" test mocks the external dependency it was written to verify.
- Tests share mutable database rows or queue state with no cleanup contract.
- Integration tests are mixed in the same run command as unit tests with no
  way to separate them.
- Tests pass locally but fail in CI because of environment-specific
  configuration.
- A test seeds data inline inside assertions rather than in explicit setup.
- The same integration test resets containers between every individual test
  case, causing the suite to be prohibitively slow.
- Integration tests verify internal implementation details of the external
  system rather than the behavior the code depends on.

## Test Structure Guidance

Adapt layout to the project's language and framework, but keep integration
tests clearly separated from unit tests and organized by the boundary they
cover.

### Preferred conventions

1. Keep integration tests in a directory or package that is clearly distinct
   from unit tests.
2. Name test groups after the boundary or integration surface being tested.
3. Share environment setup at the suite or package level, not per test.
4. Provide a single command to run the integration suite in isolation.
5. Document required environment setup (containers, env vars, seed scripts) at
   the top of the test module or in a co-located README.

### Typical layouts

```text
tests/
  unit/
    billing/
      calculate-total.test.ts
  integration/
    billing/
      order-repository.integration.test.ts
    notifications/
      email-sender.integration.test.ts
```

```text
tests/
  test_pricing.py            # unit
  integration/
    test_pricing_repository.py
```

```text
internal/
  repository/
    order_repo_test.go       # unit (with mock db)
integration/
  order_repo_integration_test.go
```

The exact layout can change. The requirement does not: unit and integration
tests must be clearly separated and independently runnable.

## Common Patterns

### Pattern 1: Database repository test

```javascript
describe('OrderRepository (integration)', () => {
  let db;

  beforeAll(async () => {
    db = await createTestDatabase(); // starts container or in-memory DB
    await db.migrate();
  });

  afterAll(() => db.close());

  beforeEach(() => db.seed({ orders: [] }));
  afterEach(() => db.truncate('orders'));

  test('persists a new order and retrieves it by id', async () => {
    const repo = new OrderRepository(db);
    const order = Order.create({ sku: 'BOOK-01', quantity: 2 });

    await repo.save(order);

    const found = await repo.findById(order.id);
    expect(found.sku).toBe('BOOK-01');
    expect(found.quantity).toBe(2);
  });

  test('raises OrderNotFoundError for an unknown id', async () => {
    const repo = new OrderRepository(db);
    await expect(repo.findById('unknown')).rejects.toThrow(OrderNotFoundError);
  });
});
```

### Pattern 2: HTTP adapter test with a recorded fixture server

```javascript
describe('PaymentGatewayAdapter (integration)', () => {
  let server;

  beforeAll(() => {
    server = startFixtureServer({
      'POST /charges': { status: 201, body: { id: 'ch_123', status: 'ok' } }
    });
  });

  afterAll(() => server.close());

  test('maps a successful charge response into a domain result', async () => {
    const adapter = new PaymentGatewayAdapter({ baseUrl: server.url });
    const result = await adapter.charge({ amount: 5000, currency: 'USD' });

    expect(result.chargeId).toBe('ch_123');
    expect(result.status).toBe('ok');
  });
});
```

### Pattern 3: Message queue producer–consumer test

```javascript
describe('NotificationQueue (integration)', () => {
  let queue;

  beforeAll(async () => {
    queue = await startTestQueue(); // local broker or in-memory
  });

  afterAll(() => queue.stop());
  afterEach(() => queue.purge());

  test('consumer receives the message published by the producer', async () => {
    const producer = new NotificationProducer(queue);
    const received = [];
    const consumer = new NotificationConsumer(queue, (msg) => received.push(msg));

    await consumer.start();
    await producer.publish({ type: 'welcome', userId: 'u1' });
    await queue.drain();

    expect(received).toHaveLength(1);
    expect(received[0].type).toBe('welcome');
  });
});
```

### Pattern 4: Constraint violation test

```javascript
test('rejects a duplicate order reference', async () => {
  const repo = new OrderRepository(db);
  const order = Order.create({ referenceId: 'REF-001', sku: 'BOOK-01' });

  await repo.save(order);

  await expect(
    repo.save(Order.create({ referenceId: 'REF-001', sku: 'BOOK-02' }))
  ).rejects.toThrow(DuplicateReferenceError);
});
```

## Review Heuristics

### Scope test

Is the test verifying a seam between components, or is it testing logic that
could be isolated in a unit test? If no real boundary participates, move it to
the unit suite.

### Real-boundary test

Does the test use the actual external system, an in-process equivalent, or a
full-protocol substitute? If the key dependency is mocked away, the test is not
exercising the integration.

### State-pollution test

If this test runs after any other test in the suite, will the outcome change?
Shared mutable state is the most common source of non-deterministic integration
failures.

### Environment-portability test

Would this test run unchanged on a colleague's machine and in CI with only the
documented environment setup? If it depends on a local service that is not
provisioned by the test infrastructure, it will break silently.

### Contract-coverage test

Does the test verify what the calling code actually depends on — the returned
shape, persisted fields, error type, and message — or only that no exception
was raised? Shallow assertions miss the contract.

### Boundary-choice test

Should this be an integration test, or is it a full end-to-end user flow? If
the test requires the full application stack, the HTTP layer, and a real
browser or client, move it to the end-to-end suite instead of inflating the
integration suite.

## Related Guides

- [UNIT_TEST_GUIDE.md](./UNIT_TEST_GUIDE.md) for fast, isolated tests of
  individual units — the complement to integration testing.
- [CODE_QUALITY_CONTROL_GUIDE.md](./CODE_QUALITY_CONTROL_GUIDE.md) for the
  broader quality gates that both unit and integration tests support.
- [CLEAN_ARCHITECTURE_GUIDE.md](./CLEAN_ARCHITECTURE_GUIDE.md) for keeping
  integration boundaries explicit and infrastructure adapters separate from
  domain logic.
- [LOW_COUPLING_GUIDE.md](./LOW_COUPLING_GUIDE.md) for designing boundaries
  that are easy to substitute in test environments.

## Summary Checklist

- [ ] Each integration test names the real boundary or collaborator it
      exercises.
- [ ] At least one real external dependency participates — no boundary is
      mocked away.
- [ ] Test state is seeded explicitly and cleaned up unconditionally.
- [ ] The integration suite is runnable in isolation with a single command.
- [ ] The test environment is reproducible locally and in CI.
- [ ] Error paths at the boundary are covered alongside the happy path.
- [ ] Containers or embedded services are shared at the suite level, not
      restarted per test.
- [ ] Integration tests are clearly separated from unit tests in layout and
      run commands.

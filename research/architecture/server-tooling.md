# TypeScript server and data tools for a 2027 WMS

**Trial Fastify, JSON Schema and node-postgres first** if the team can review SQL comfortably. This combination exposes the exact transaction that changes stock. **Compare Hono, Zod and Drizzle** if maintaining TypeScript definitions across a growing catalog and many endpoints is more costly. Both can run on Node.js and use PostgreSQL through the same driver, so the comparison can hold the database schema and business contract constant. These are implementation candidates. No benchmark has been run.

## What each combination buys

| Candidate | HTTP validation and OpenAPI | Data access and maintenance tradeoff |
| --- | --- | --- |
| Fastify + JSON Schema/Ajv + node-postgres (`pg`) | Declare request and response schemas on the route and use `@fastify/swagger` to generate OpenAPI. Test Fastify’s body coercion settings explicitly. Converting a numeric input into a string can defeat a decimal-string contract. [Fastify validation and serialization](https://fastify.dev/docs/latest/Reference/Validation-and-Serialization/) · [Fastify Swagger plugin](https://github.com/fastify/fastify-swagger) | Parameterized SQL and explicit transaction control make locks, predicates and rollback easy to inspect. The team owns query-result mapping, migration SQL and consistency between TypeScript and route schemas. Use the same checked-out client throughout a transaction and release it on every exit path. [node-postgres transactions](https://node-postgres.com/features/transactions) |
| Hono + Zod OpenAPI + Drizzle over `pg` | Hono’s Node adapter serves routes. `@hono/zod-openapi` uses Zod schemas to validate inputs and describe operations. Document runtime refinements and business errors that a generated schema cannot express. [Hono Node adapter](https://hono.dev/docs/getting-started/nodejs) · [Hono Zod OpenAPI example](https://hono.dev/examples/zod-openapi) | Drizzle supplies typed queries and a transaction callback with isolation settings. Its SQL template allows a guarded update to remain explicit. Inspect generated SQL and confirm every write uses the callback’s transaction handle. The Node adapter, validation bridge, ORM and migration tooling each introduce a compatibility check. [Drizzle PostgreSQL drivers](https://orm.drizzle.team/docs/get-started-postgresql) · [Drizzle transactions](https://orm.drizzle.team/docs/transactions) · [Drizzle SQL templates](https://orm.drizzle.team/docs/sql) |

## Database contract for both candidates

For the comparison, a receipt line identifies an authenticated merchant, command, item, lot if tracked, location, unit and exact quantity. Accept positive finite decimal text with an item-specific maximum scale. Store whole items as integral counts and measured quantities as exact decimals or scaled integers. Keep two crates and their 1.250 kg net weight as separate observations. Validate precision before casting to a constrained `numeric` column, since PostgreSQL can round on assignment. Return the documented string representation through both clients. [PostgreSQL numeric types](https://www.postgresql.org/docs/18/datatype-numeric.html)

For a client submission, use non-null fields and `UNIQUE (merchant_id, installation_id, operation_type, client_key)`. A manual entry uses the merchant’s internal source namespace. Authorize the current caller for the site and action before returning a stored outcome. Keep external order identity unique by merchant, installation and external order ID. Give a posted movement its own WMS-generated ID unique within the merchant. Scope references to a merchant with composite foreign keys. Protect every balance with a unique stock identity including location, variant, lot and disposition where applicable. If lot is optional, define how a missing lot participates in uniqueness, for example with `NULLS NOT DISTINCT`. Apply non-null and row-local check constraints to eligible and reserved quantities under the agreed stock policy. Cross-row invariants need transaction logic. [PostgreSQL constraints](https://www.postgresql.org/docs/18/ddl-constraints.html)

In one receipt transaction claim the command, record its normalized input and result, write the movement, update the balance and append any required outbox row. A repeat with identical input returns the original movement. A changed payload using the same key conflicts. A single-balance reservation can use the following parameterized statement inside the transaction, after checking quantity and eligibility. This example assumes that placing units on hold removes them from the eligible bucket under the same balance lock. If eligibility depends on a separate lot record, hold, reserve and dispatch must lock that record in a consistent order and recheck its disposition in the posting transaction. A zero-row update requires a domain outcome before reporting success.

```sql
UPDATE stock_balance
SET reserved = reserved + $3
WHERE merchant_id = $1 AND balance_id = $2
  AND on_hand - reserved >= $3
RETURNING reserved;
```

For multi-row business rules that use serializable isolation, retry the entire transaction after a serialization failure and preserve the caller’s command ID. Keep external delivery outside that transaction. [PostgreSQL transaction isolation](https://www.postgresql.org/docs/18/transaction-iso.html)

## A comparative implementation trial

Build the same two operations, `POST /receipts` and `POST /reservations`, in disposable branches. Feed both the same warehouse fixtures, PostgreSQL version, indexes, pool size, Node version and offered workload. Use the same OpenAPI dialect supported by both generators and the target merchant client. Publish an artifact locally, validate it, then generate a client that imports neither candidate’s server code. Check decimal strings, required fields, command identity, error responses and a replay from an earlier client. [OpenAPI specification](https://spec.openapis.org/oas/v3.0.3)

| Trial | Observable pass condition |
| --- | --- |
| Receipt and HTTP serialization | Receive two crates totaling 1.250 kg. Assert both quantities and their units on the wire, in the ledger and in the generated client. Reject numeric JSON input for a string field and 0.0005 kg for an item limited to grams. |
| Crash and replay | Fail after writing the movement but before commit, then retry. Separately lose the response after commit. In each case the final stock reflects one posting and retry returns the settled movement ID. |
| Contention and database serialization | Two sessions each reserve 0.750 kg from 1.250 kg. Coordinate execution with barriers. One reservation settles and 0.500 kg remains available. For a serializable multi-row case, seed two bins with one bag each and a trial rule that one bag must remain free across the site. Both sessions read both bins before claiming different bags. After the serialization retry, one reservation settles and one bag remains free. [Deterministic stock verification examples](inventory-verification.md) |
| Hold races | Apply a hold concurrently with a reservation, then with dispatch of a staged parcel. When the hold settles first, block new reservations and dispatch. If a reservation already settled, flag it and stop the pick or handover. If dispatch settled first, preserve the movement and include its recipient in the trace. |
| Upgrade and data compatibility | Add a new eight-bag pack alongside a historical ten-bag pack. Run both implementations against supported old receipt payloads, migrate twice and verify original unit evidence and quantities. [Historical unit migration](code-practices.md) |

Only compare speed after stock and contract checks pass. Record attempted and committed commands per second separately, median and 95th-percentile end-to-end latency, query count, lock waits and retry count for uncontended and hot-bin workloads. Repeat matched runs in alternating order. Have another developer change a receipt field and inspect the migration and generated client. Record the time, files touched and defects each reviewer caught. These are proposed measurements, with no target until a warehouse supplies its actual peak and response-time needs.

### Observed HTTP contract on two frameworks

On 26 November 2026, a temporary fixture sent **32 real HTTP requests** over loopback, covering eight single-line receipts under four configurations. Both strict configurations returned the same status and JSON for every case. Fastify’s baseline accepted the JSON number `1.25` and returned the string `"1.25"`. Hono with a plain Zod string rejected that number. Both baseline configurations accepted a misspelled or unknown field and omitted it from the accepted response. Write the desired input policy into the public contract and test it on the route. [Fastify validation and serialization](https://fastify.dev/docs/latest/Reference/Validation-and-Serialization/) · [Hono Zod OpenAPI route](https://hono.dev/examples/zod-openapi)

| Receipt variation | Fastify baseline | Hono baseline | Both strict |
| --- | --- | --- | --- |
| Nonempty item and pack, quantity `"1.250"` or `"9007199254740993.125"` | 201, exact original strings | 201, exact original strings | 201, exact original strings |
| Quantity supplied as JSON number `1.25` | 201, converted to string | 400, quantity | 400, quantity |
| Additional property `extra` | 201, discarded | 201, discarded | 400, extra |
| Quantity `"0.0005"` or `"0.000"` | 400, quantity | 400, quantity | 400, quantity |
| Missing pack or empty item | 400, respective field | 400, respective field | 400, respective field |

**What ran.** Node 24.22.3 on Linux arm64, Fastify 5.12.8, Ajv 8.20.0, fast-json-stringify 7.3.10, Fastify Swagger 9.9.5, Hono 4.13.22, Hono Node Server 2.1.11, Hono Zod OpenAPI 1.6.3 and Zod 4.6.7. Each route echoed the validated line and deliberately translated one field error to `{"code":"invalid_receipt","field":"…"}`. Fastify’s strict run disabled Ajv coercion and additional-property removal. Hono’s strict run used `z.strictObject`. Both generated OpenAPI 3.0.3 documents listing the POST operation. The fixture exercised wire validation and round-trip JSON. It did not test authentication, database persistence, generated clients, concurrent requests, or performance.

<details>
<summary>Repeat the HTTP trial in an empty temporary directory</summary>

With Node 24.22.3 available, save the JavaScript below as `trial.mjs`, then run these commands. The fixture opens ephemeral loopback ports, closes each server and exits on the first mismatch.

```sh
npm install --prefix . --cache ./cache --ignore-scripts --no-audit --no-fund --save-exact \
  fastify@5.12.8 @fastify/swagger@9.9.5 hono@4.13.22 @hono/node-server@2.1.11 \
  @hono/zod-openapi@1.6.3 zod@4.6.7 ajv@8.20.0 fast-json-stringify@7.3.10
node trial.mjs
```

```js
import assert from 'node:assert/strict';
import { once } from 'node:events';
import Fastify from 'fastify';
import swagger from '@fastify/swagger';
import { serve } from '@hono/node-server';
import { OpenAPIHono, createRoute, z } from '@hono/zod-openapi';

const pattern = /^(?!0(?:\.0{1,3})?$)(?:0|[1-9][0-9]*)(?:\.[0-9]{1,3})?$/;
const info = { title: 'Receipt fixture', version: '1' };
const body = { type: 'object', additionalProperties: false,
  required: ['item', 'pack', 'quantity'], properties: {
    item: { type: 'string', minLength: 1 }, pack: { type: 'string', minLength: 1 },
    quantity: { type: 'string', pattern: pattern.source } } };
const problem = field => ({ code: 'invalid_receipt', field });
async function fastify(strict) {
  const f = Fastify(strict ? { ajv: { customOptions: { coerceTypes: false, removeAdditional: false } } } : {});
  await f.register(swagger, { openapi: { openapi: '3.0.3', info } });
  f.setErrorHandler((error, req, reply) => {
    const issue = error.validation?.[0];
    if (!issue) return reply.send(error);
    const field = issue.params.missingProperty ?? issue.params.additionalProperty ?? issue.instancePath.slice(1);
    return reply.code(400).send(problem(field));
  });
  f.post('/receipts', { schema: { body, response: { 201: body } } },
    (req, reply) => reply.code(201).send(req.body));
  await f.listen({ host: '127.0.0.1', port: 0 });
  assert(f.swagger().paths['/receipts'].post);
  return [f.server.address().port, () => f.close()];
}
async function hono(strict) {
  const app = new OpenAPIHono({ defaultHook: (result, c) => {
    if (result.success) return;
    const issue = result.error.issues[0];
    return c.json(problem(issue.keys?.[0] ?? issue.path[0]), 400);
  } });
  const shape = { item: z.string().min(1), pack: z.string().min(1),
                  quantity: z.string().regex(pattern) };
  const schema = strict ? z.strictObject(shape) : z.object(shape);
  const content = { 'application/json': { schema } };
  app.openapi(createRoute({ method: 'post', path: '/receipts',
    request: { body: { required: true, content } },
    responses: { 201: { description: 'Accepted', content } } }),
    c => c.json(c.req.valid('json'), 201));
  assert(app.getOpenAPIDocument({ openapi: '3.0.3', info }).paths['/receipts'].post);
  const server = serve({ fetch: app.fetch, hostname: '127.0.0.1', port: 0 });
  if (!server.listening) await once(server, 'listening');
  return [server.address().port, () => new Promise((ok, fail) => server.close(e => e ? fail(e) : ok()))];
}

const valid = { item: 'tomato-42', pack: 'loose-kg', quantity: '1.250' };
const cases = [
  ['valid', valid, null], ['large', { ...valid, quantity: '9007199254740993.125' }, null],
  ['numeric', { ...valid, quantity: 1.25 }, 'quantity'],
  ['unknown', { ...valid, extra: 'unused' }, 'extra'],
  ['scale', { ...valid, quantity: '0.0005' }, 'quantity'],
  ['zero', { ...valid, quantity: '0.000' }, 'quantity'],
  ['missingPack', { item: valid.item, quantity: '1.250' }, 'pack'],
  ['emptyItem', { ...valid, item: '' }, 'item']
];
for (const build of [fastify, hono]) for (const strict of [false, true]) {
  const [port, close] = await build(strict);
  try {
    for (const [label, input, field] of cases) {
      const response = await fetch('http://127.0.0.1:' + port + '/receipts', {
        method: 'POST', headers: { 'content-type': 'application/json' }, body: JSON.stringify(input) });
      const actual = await response.json();
      const coerced = !strict && label === 'numeric' && build === fastify;
      const stripped = !strict && label === 'unknown';
      const expected = coerced ? { ...valid, quantity: '1.25' } : stripped ? valid : field ? problem(field) : input;
      assert.equal(response.status, field && !coerced && !stripped ? 400 : 201);
      assert.deepEqual(actual, expected);
      console.log(build.name, strict ? 'strict' : 'baseline', label, response.status, JSON.stringify(actual));
    }
  } finally { await close(); }
}
```

</details>

## Support check before selection

The documentation was checked on 26 November 2026. At selection, record the actual mutually supported Node, framework, validator, OpenAPI generator, driver and database versions. Review Fastify’s LTS and Swagger plugin compatibility tables. For Hono, verify the Node adapter and Zod OpenAPI peer dependencies together, and run the selected Drizzle release and migration-tool version against the chosen PostgreSQL major. Re-run the fixture suite after a representative dependency upgrade. Choose on demonstrated behavior and the team’s ability to maintain the combination through the intended product window. 2027 package availability and compatibility remain unverified. [Node release schedule](https://nodejs.org/en/about/previous-releases) · [Fastify LTS policy](https://fastify.dev/docs/latest/Reference/LTS/) · [Fastify Swagger compatibility](https://github.com/fastify/fastify-swagger)

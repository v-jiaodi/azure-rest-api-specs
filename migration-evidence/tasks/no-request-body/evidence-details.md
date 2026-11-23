# no-request-body supporting evidence

## Assessment boundary

This assessment describes the changes between merge base
`eacc4548e7e55dc8e464a93d20906712e7bc3a4f` and branch head
`acbe20a7c0a5f2b5b611fe3c3e694373dc03fa79`. The branch changes six files:

- [eng/typespec-rulesets/index.ts](../../../eng/typespec-rulesets/index.ts)
- [eng/tools/typespec-validation/test/compiler-cli.test.ts](../../../eng/tools/typespec-validation/test/compiler-cli.test.ts)
- [specification/machinelearningservices/data-plane/RunHistory/routes-runs.tsp](../../../specification/machinelearningservices/data-plane/RunHistory/routes-runs.tsp)
- [specification/purview/data-plane/datamap/client.tsp](../../../specification/purview/data-plane/datamap/client.tsp)
- [specification/purview/data-plane/datamap/routes.tsp](../../../specification/purview/data-plane/datamap/routes.tsp)
- [specification/search/resource-manager/Microsoft.Search/Search/client.tsp](../../../specification/search/resource-manager/Microsoft.Search/Search/client.tsp)

The branch contains no saved Swagger baseline, full-corpus TypeSpec diagnostic
artifact, successful/failed project inventory, API-version projection manifest, or
per-finding correspondence data. The evidence therefore supports an enablement and
focused-behavior assessment, not a full migration-equivalence conclusion.

## Focused profile validation

The repository-owned test compiles a synthetic fixture through both the
`data-plane` and `resource-manager` profiles, each also extending `client-sdk`.
Each run contains:

- a GET operation with an explicit body;
- a DELETE operation with an explicit body;
- a bodyless GET control;
- a bodyless DELETE control; and
- a POST operation with a body as a verb-scope control.

Validation command, run from `eng/tools/typespec-validation`:

```text
pnpm exec vitest run test/compiler-cli.test.ts --reporter=verbose
```

Result: exit code 0; 15 tests passed. Both named cases passed:

```text
enforces no-request-body through the data-plane and client-sdk profiles
enforces no-request-body through the resource-manager and client-sdk profiles
```

Each case asserts exactly two `no-request-body` diagnostics and checks the GET and
DELETE messages. The reader-facing TypeSpec count is therefore 4 across two
independent synthetic runs. It is not a corpus count and is not comparable with a
Swagger total that was not run.

## Compatibility suppressions

The branch adds 22 authored suppression directives in three TypeSpec projects.
These are migration-surface indicators, not captured diagnostic occurrences.

| Project | Suppression placement | GET directives | DELETE directives | Total directives |
| --- | --- | ---: | ---: | ---: |
| Machine Learning RunHistory | Individual operations | 0 | 3 | 3 |
| Purview Data Map | Individual operations | 0 | 6 | 6 |
| Purview Data Map | Client operation-group interfaces | 0 | 3 | 3 |
| Azure AI Search | Individual client customization operations | 6 | 4 | 10 |
| **Total** | | **6** | **16** | **22** |

The three Purview client suppressions are attached to interfaces rather than
individual operations. A directive can therefore cover more than one diagnostic.
No unsuppressed compile output was retained, so neither 22 nor the 19
operation-level directives may be treated as the TypeSpec diagnostic count.

### Representative GET compatibility case

Azure AI Search retains a client-only GET customization whose resolved request
options form a body. The branch suppresses `no-request-body` on the operation to
preserve the existing API/client shape. The corresponding emitted Swagger finding
was not captured in this assessment.

Source: [Search/client.tsp](../../../specification/search/resource-manager/Microsoft.Search/Search/client.tsp)

### Representative DELETE compatibility case

Machine Learning RunHistory retains DELETE operations with explicit body models,
including `deleteRunServices`. The branch suppresses `no-request-body` on each
operation because removing the body would change the existing wire contract. The
corresponding emitted Swagger finding was not captured in this assessment.

Source: [RunHistory/routes-runs.tsp](../../../specification/machinelearningservices/data-plane/RunHistory/routes-runs.tsp)

## Counting boundary

The assessed TypeSpec rule resolves the HTTP operation and reports once when a GET
or DELETE operation has a non-void body. It skips template declarations and checks
their concrete operations. Its body model includes explicit `@body`, `@bodyRoot`,
multipart, implicit, optional, and empty-model bodies.

The three Swagger rules inspect emitted OpenAPI 2 body parameters:

- ARM `GetMustNotHaveRequestBody` checks GET body parameters under `paths` and
  `x-ms-paths`.
- ARM `DeleteMustNotHaveRequestBody` checks DELETE body parameters under `paths`.
- Data-plane `RequestBodyNotAllowed` checks GET and DELETE body parameters under
  `paths`.

One TypeSpec source operation can emit into multiple files or API-version
projections. Conversely, the TypeSpec rule intentionally emits one diagnostic per
operation rather than one per emitted body parameter. Exact correspondence requires
an aligned version and emission policy plus retained raw diagnostics on both sides.

## Required evidence for completion

Before this report can support migration acceptance, a follow-up assessment must:

1. Pin the specs revision, TypeSpec rule revision, compiler/emitter closure, and
   Swagger baseline.
2. Record discovered, successfully compiled, failed, and excluded projects for the
   same ARM/data-plane and API-version population.
3. Run all three Swagger rules and the TypeSpec rule, retaining raw per-project
   diagnostics and captured suppressed diagnostics.
4. Compare Swagger-only, TypeSpec-only, and overlapping projects, then investigate
   every one-sided project.
5. Reconcile raw diagnostic differences using operation, HTTP verb, emitted file,
   JSON path, and TypeSpec source location without treating matching totals as
   finding identity.
6. Recompile the compatibility projects without only the task-owned suppressions to
   establish their actual TypeSpec diagnostic population and assess the associated
   Swagger findings.

# no-request-body migration

## 1. Scope

The TypeSpec linter
[`@azure-tools/typespec-azure-core/no-request-body`](https://github.com/Azure/typespec-azure/blob/cf06eebc9613ed64eb7ef893b4cc535fe9a0634d/packages/typespec-azure-core/src/rules/no-request-body.ts)
is assessed against Swagger linter rules
[`GetMustNotHaveRequestBody`](https://github.com/Azure/azure-openapi-validator/blob/main/docs/get-must-not-have-request-body.md),
[`DeleteMustNotHaveRequestBody`](https://github.com/Azure/azure-openapi-validator/blob/main/docs/delete-must-not-have-request-body.md),
and data-plane
[`RequestBodyNotAllowed`](https://github.com/Azure/azure-openapi-validator/blob/main/docs/request-body-not-allowed.md).
Original migration linter PR:
[Azure/typespec-azure#5986](https://github.com/Azure/typespec-azure/pull/5986).

## 2. Coverage

The current branch enables the TypeSpec rule for the repository's data-plane and
resource-manager profiles. A focused test assessed two synthetic compilations,
one per profile. Both compiled the same five-operation fixture and each reported
the expected two TypeSpec diagnostics. No Swagger linter run or full TypeSpec
repository corpus result is retained on this branch. Three existing TypeSpec
projects were updated with compatibility suppressions, but suppression directives
are not diagnostic occurrences and are excluded from the table.

| Count | Swagger linter | TypeSpec linter |
| --- | ---: | ---: |
| Reported diagnostics | Unassessed | 4 (focused synthetic runs only) |

**Analysis:** The focused evidence verifies repository profile activation and the
basic GET/DELETE behavior. It does not establish finding-by-finding correspondence
or a full-corpus migration result. The branch adds 22 suppression directives across
three projects: 19 target individual operations and three target Purview client
operation-group interfaces. Their unsuppressed diagnostic cardinality was not
captured. See [Why diagnostic counts differ](#3-why-diagnostic-counts-differ) and
the [supporting evidence](evidence-details.md).

**Assessed coverage:** Both repository profiles enforce the rule for the focused
GET and DELETE violations and allow the three controls. Full-corpus coverage,
Swagger correspondence, and compatibility-suppression diagnostic counts remain
unassessed.

## 3. Why diagnostic counts differ

The focused TypeSpec count uses one diagnostic per violating operation in two
synthetic profile runs. There is no aligned Swagger population on this branch.
The upstream TypeSpec rule resolves authored HTTP bodies and reports once per
operation, while the Swagger rules inspect emitted OpenAPI body parameters.

| Difference (with evidence) | Swagger diagnostics | TypeSpec diagnostics for the same check | analysis |
| --- | ---: | ---: | --- |
| [Focused profile activation](evidence-details.md#focused-profile-validation) | Unassessed | 4 | Two GET and two DELETE diagnostics were observed across the two independent profile runs. Run the three Swagger rules over the same emitted fixtures before comparing counts. |
| [Existing API compatibility surface](evidence-details.md#compatibility-suppressions) | Unassessed | Unassessed | The branch adds 22 suppression directives, including three interface-level suppressions. Re-run without task-owned suppressions and retain per-diagnostic output before assigning a TypeSpec count or claiming Swagger correspondence. |
| [Source-to-emission counting](evidence-details.md#counting-boundary) | Unassessed | Unassessed | TypeSpec reports once per resolved source operation; Swagger reports emitted body-parameter occurrences. Version projection and emission multiplicity have not been assessed for the affected projects. |

**Raw-count reconciliation:** Unassessed. The observed TypeSpec total of 4 belongs
only to synthetic profile tests, while no Swagger diagnostics were captured for
that population. The complete corpus difference therefore cannot be calculated
from this branch.

## 4. Function-level coverage

| Behavior | Result | analysis |
| --- | --- | --- |
| GET operation with a request body | Covered in focused repository test | Both data-plane and resource-manager profile runs report the expected GET diagnostic; see [focused profile validation](evidence-details.md#focused-profile-validation). |
| DELETE operation with a request body | Covered in focused repository test | Both profile runs report the expected DELETE diagnostic; see [focused profile validation](evidence-details.md#focused-profile-validation). |
| Bodyless GET and DELETE operations | Covered in focused repository test | The same fixture includes bodyless controls and asserts that only two diagnostics are emitted. |
| POST operation with a request body | Covered in focused repository test | The same fixture includes a POST body control and asserts that only GET and DELETE are reported. |
| Explicit, optional, implicit, `@bodyRoot`, multipart, and empty-model bodies | Covered by upstream rule tests; not rerun here | The [upstream tests](https://github.com/Azure/typespec-azure/blob/cf06eebc9613ed64eb7ef893b4cc535fe9a0634d/packages/typespec-azure-core/test/rules/no-request-body.test.ts) exercise these authored forms. This branch validates profile integration rather than repeating that matrix. |
| Standard ARM read and delete templates with and without customized bodies | Covered by upstream rule tests; not rerun here | The [ARM tests](https://github.com/Azure/typespec-azure/blob/cf06eebc9613ed64eb7ef893b4cc535fe9a0634d/packages/typespec-azure-resource-manager/test/rules/no-request-body.test.ts) cover standard templates and body customizations. |
| Existing GET/DELETE request bodies in repository projects | Compatibility suppressions added; diagnostic count unassessed | The affected Machine Learning, Purview, and Search declarations are inventoried in [compatibility suppressions](evidence-details.md#compatibility-suppressions). |
| Full mapping to all three Swagger rules across the repository corpus | Unassessed | No pinned Swagger baseline, aligned successful-project inventory, API-version projection, or per-finding correspondence artifact is present on this branch. |

## NOTES

- **Inputs:** Specs branch head
  [`acbe20a7c0a5f2b5b611fe3c3e694373dc03fa79`](https://github.com/v-jiaodi/azure-rest-api-specs/commit/acbe20a7c0a5f2b5b611fe3c3e694373dc03fa79),
  compared with merge base `eacc4548e7e55dc8e464a93d20906712e7bc3a4f`.
  The focused run used `@azure-tools/typespec-azure-core@0.74.0`; upstream source
  links are pinned to documentation snapshot
  [`cf06eebc9613ed64eb7ef893b4cc535fe9a0634d`](https://github.com/Azure/typespec-azure/commit/cf06eebc9613ed64eb7ef893b4cc535fe9a0634d).
- **Counting evidence:** Only the focused synthetic TypeSpec diagnostics were
  captured. Full-corpus Swagger and TypeSpec counts, suppression-inclusive counts,
  and member attribution remain unassessed.
- **Validation:**
  `pnpm exec vitest run test/compiler-cli.test.ts --reporter=verbose` passed all
  15 tests, including both profile cases. See the [validation receipt](evidence-details.md#focused-profile-validation).
- **Follow-ups:** A publishable migration-coverage conclusion requires an aligned,
  pinned Swagger baseline and TypeSpec corpus run, retained raw diagnostics,
  per-project set comparison, and investigation of every one-sided result.

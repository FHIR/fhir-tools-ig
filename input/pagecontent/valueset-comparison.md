This page describes the `$compare` operation, which determines the relationship between two value sets. It is intended for implementers of terminology clients and terminology servers.

### What it answers

Given two value sets, `$compare` answers a single question: **how do their memberships relate?** The comparison is over what the value sets *mean* &mdash; the set of codes each one includes &mdash; not over the structure of the two `ValueSet` resources. Two value sets that are written quite differently but include exactly the same codes are reported as the `same`.

The two value sets are referred to as **this** and **other**, and the result is always stated from the perspective of *this*. So `subset` means *this is a subset of other*, and `superset` means *this is a superset of other*.

### Invoking the operation

At the type level:

```
POST [base]/ValueSet/$compare
```

with a `Parameters` body that supplies each value set either inline or by canonical URL:

* `thisValueSet` / `otherValueSet` &mdash; the value set as an inline resource; or
* `thisUrl` / `otherUrl` &mdash; the canonical URL of a value set the server resolves (optionally with `valueSetVersion`).

At the instance level:

```
POST [base]/ValueSet/[id]/$compare
```

the value set addressed in the URL is *this*, and only *other* need be supplied (as `otherValueSet` or `otherUrl`).

If the value sets reference code systems the server does not already know, supply them as `tx-resource` parameters (or register them through the [terminology cache](terminology-caching.html)).

### The result

The operation returns a `Parameters` resource with a `result` code and a human-readable `message`. The `result` is one of:

| `result` | meaning (from the perspective of *this*) |
|----------|------------------------------------------|
| `same` | both value sets contain the same codes |
| `subset` | every code in *this* is in *other*, and *other* has more |
| `superset` | every code in *other* is in *this*, and *this* has more |
| `overlapping` | each value set contains codes the other does not, but they share some |
| `disjoint` | the value sets share no codes |
| `empty` | both value sets are empty |
| `indeterminate` | the server cannot determine the relationship |

`indeterminate` is a deliberate, honest answer, not a failure: the server returns it (with the reason in `message`) rather than guessing when it cannot soundly decide. Common causes are a value set with no `compose`, an expansion that is [unclosed](https://hl7.org/fhir/valueset-unclosed.html) or unbounded, an expansion that fails, or definitions that combine filters, imports, and code lists in ways the server cannot reason about exactly.

### How the server decides

A server **may** answer directly from the value set definitions where it can, and otherwise falls back to comparing expansions. Neither approach is required: a server is free to always expand, and a client cannot assume any particular strategy was used. What matters to the caller is the `result`; the strategy only affects cost and how often the answer is `indeterminate`.

**From the definitions.** A server that compares the `compose` definitions directly groups the includes and excludes by code system and compares them. This is cheap, exact, and needs no expansion. It can handle whole-system includes, enumerated code lists, and single filters &mdash; including reasoning about `is-a` filters through subsumption, so a value set bound to a concept and one bound to its ancestor can be related as `subset`/`superset`. How the comparison treats code system versions is described under [Versioning](#versioning) below.

**By expansion.** Where the definitions are too complex to compare directly (or the server simply chooses to), it expands both value sets and compares the resulting code lists. If either expansion is unclosed or unbounded, or fails, the result is `indeterminate`.

Whether the server expanded is reported in the diagnostics (see below); callers do not choose the strategy.

### Versioning

By default, `$compare` treats different versions of a code system as interchangeable: the same code is the same member whether it was drawn from version 1 or version 2 of its code system. Version becomes significant only where it has to &mdash; specifically, when the code system declares `versionNeeded = true`, or when a **filter** is involved, because a filter such as `is-a` can select different codes in different versions, so its result is inherently version-dependent.

A client can override this with the `version-policy` parameter, which takes one of three values:

* `always` &mdash; versions always matter. A code counts as shared only if its code system version matches on both sides.
* `as-needed` &mdash; the default described above: version matters where either code system declares `versionNeeded = true`, or where a filter makes it significant.
* `never` &mdash; versions never matter. Codes are compared by system and code alone, ignoring version. The caller takes responsibility for this, including where filters are present.

When versions are significant, two value sets that include the same codes but pin different code system versions can be reported as `disjoint` or `overlapping` rather than `same`; under `never` they would be reported as `same`.

### Diagnostics

If the request sets the `diagnostics` parameter to `true`, the server adds, where it can:

* `performed-expansion` &mdash; whether it had to expand the value sets (`false` means it decided from the definitions alone);
* `common-codes` &mdash; codes present in both value sets;
* `missing-codes` &mdash; codes in *this* but not *other*;
* `extra-codes` &mdash; codes in *other* but not *this*.

These are comma-separated code lists, and are best-effort: they describe the codes that drove the result, and may be omitted (for example, the lists are not produced when the value sets are identical, and may be limited for very large value sets).

### Examples

Comparing two value sets by URL, asking for diagnostics:

```json
{
  "resourceType": "Parameters",
  "parameter": [
    { "name": "thisUrl", "valueUri": "http://example.org/ValueSet/colours-primary" },
    { "name": "otherUrl", "valueUri": "http://example.org/ValueSet/colours-all" },
    { "name": "diagnostics", "valueBoolean": true }
  ]
}
```

A possible response &mdash; *this* is contained in *other*:

```json
{
  "resourceType": "Parameters",
  "parameter": [
    { "name": "result", "valueCode": "subset" },
    { "name": "message", "valueString": "The valueSet http://example.org/ValueSet/colours-primary is a sub-set of the valueSet http://example.org/ValueSet/colours-all" },
    { "name": "performed-expansion", "valueBoolean": false },
    { "name": "extra-codes", "valueString": "green,white,black" }
  ]
}
```

### Notes

* `$compare` does not change server state (`affectsState = false`); it is safe to call repeatedly.
* The result is symmetric in the sense that swapping *this* and *other* swaps `subset`/`superset` and `missing-codes`/`extra-codes`; `same`, `overlapping`, `disjoint`, `empty`, and `indeterminate` are unaffected.
* A server advertises support by declaring the `$compare` operation on `ValueSet` in its `CapabilityStatement`.

See the [$compare OperationDefinition](OperationDefinition-ValueSet-compare.html) for the formal operation definition.

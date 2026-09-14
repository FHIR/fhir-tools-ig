This page records what changed in each release of this implementation guide. Releases are listed newest first.

Every published version remains available at `http://hl7.org/fhir/tools/[version]`, and the full publication record — including technical corrections — is in the [publication history](http://hl7.org/fhir/tools/history.html).

### 1.2.0 (in preparation)

The theme of this release is terminology ecosystem support: two new operations, a substantially extended set of terminology issue types, and the documentation that goes with them. It also brings the additional binding purposes into line with the changes made for R6.

**New operations**

* `$cache-control` ([Terminology Cache Control](OperationDefinition-cache-control.html)) — manages a terminology client cache on the server. A client that repeatedly validates or expands against the same value sets and code systems registers them once under a server-issued cache-id, and then refers to them by url on subsequent calls rather than re-sending them. The operation defines the `start`, `check` and `end` modes, sealed and unsealed caches, and the idle timeout behaviour
* `$compare` on ValueSet ([ValueSet Comparison](OperationDefinition-ValueSet-compare.html)) — determines the set relationship between the memberships of two value sets: equivalent, subset, superset, overlapping or disjoint. The comparison is over what the value sets mean, not over the structure of the resources. The server works from the definitions where it can, and expands only where it must; where the relationship cannot be soundly established the result is `indeterminate` rather than a guess

**New extensions**

* [validator-version](StructureDefinition-validator-version.html) — the validation tool stamps its own version and build date into the OperationOutcome it produces, so a report carries a record of what produced it. Validation results depend on the version of the tool, and a report can easily outlive the build that made it
* [resource-tla](StructureDefinition-resource-tla.html) — a short mnemonic code, historically a three letter acronym, for a resource type. Used as a compact abbreviation, for example when constructing identifiers or short cross-references. The code is not required to be exactly three characters

**New terminology issue types**

Seven codes added to [tx-issue-type](CodeSystem-tx-issue-type.html), so that every issue a terminology server returns can carry a coded reason:

* `not-supported` — the server understood the request but does not implement what it asks for. Repeating it against this server will not produce a different answer; another server may be able to give one
* `too-costly` — the server can work out the answer, but the answer is too large or expensive to produce. The client should narrow the request with a filter, or page through it
* `version-error` — the problem lies in the versions of the resources involved rather than in the code or the value set
* `business-rule` — the server declines to process content because of a rule it enforces about what it will accept, rather than because the content is invalid
* `cannot-determine` — the server was unable to determine the answer and is declining to give one rather than give one that may be wrong. This arises where an operation's outcome codes have no value meaning "unknown" — `$subsumes`, for instance. Clients must not read it as a negative answer
* `cache-id-duplicate` — the request carried more than one cache-id; a server has no way to choose between them
* `cache-id-unknown` — the cache-id the client supplied is not known to the server: never created, expired, or released. Distinct from a value set or code system genuinely not being found, so that a stale cache does not masquerade as an authoring error

**New IG parameters**

New codes in [ig-parameters](CodeSystem-ig-parameters.html):

* `incubator-ig` — directs the publisher to use an incubator IG's definitions of its resources in place of the definitions in the base specification. Currently supports `hl7.fhir.uv.testing` (TestPlan, TestScript, TestReport)
* `page-heading-level` — the heading level that the top heading of every generated page is moved to
* `narrative-heading-level` — the heading level that a resource's narrative is placed at when rendered into a page (default 3)
* `wcag-conformant` — indicates that the IG should be WCAG conformant, or at least more so
* `signatures-using-r6-method` — pre-adopts the R6 method of signing Bundles
* `infer-resource-conformance` — infers the conformance level for a resource in a CapabilityStatement from the maximum conformance expectation of the interactions and search parameters used within it
* `tx-unload-early` — unloads the terminology context before the HTML inspection phase to reclaim memory earlier. Only for very large IGs that are memory starved; conformance statement rendering will not work when it is set
* `[r4|r4b|r5|r6]-inclusion` - directs the publisher to include resources for specific FHIR versions when creating multi-FHIR-version IGs.

**Changed content**

* [additional-binding-purpose](CodeSystem-additional-binding-purpose.html) has been reworked to align with the changes made in R6. `maximum` is now deprecated (it is equivalent to `required`); `current` is now displayed as "Current Binding (required)" and is joined by a new `current-extensible`; a new `best-practice` purpose is defined; and `preferred`, `ui`, `starter` and `component` are now children of a new abstract `open` purpose. The code system also now carries `notSelectable` and `status` concept properties. **Implementers should review any use of these codes** — the codes themselves are unchanged, but their hierarchy and status are not
* [type-operation](StructureDefinition-type-operation.html) — the context was wrong. It is corrected from `Extension` / `Extension.value` to `StructureDefinition`
* [view-hint](StructureDefinition-view-hint.html) — the definition is corrected: it is a complex extension, so `Extension.value[x]` is prohibited rather than required

**Documentation**

* New page: [Terminology Caching](terminology-caching.html) — the caching protocol, sealed and unsealed caches, keeping a cache alive, and the error handling. Documented here temporarily, pending a permanent home
* New page: [ValueSet Comparison](valueset-comparison.html) — what the comparison means, how servers should reason about it, and the diagnostics
* The [format extensions](format-extensions.html) page is corrected: the XML name, namespace and element-order extensions were moved into the tools canonical space some time ago, and the page still cited their old URLs. `json-primitive-choice`, `json-suppress-resourcetype`, `xml-choice-group`, `elementdefinition-date-rules`, `elementdefinition-string-format` and `implied-string-prefix` are now documented there
* The extension summary on the [home page](index.html) now covers `elementdefinition-string-format`, `resource-tla` and `validator-version`, and the usage contexts listed for `json-suppress-resourcetype` and `extension-style` are corrected

### 1.1.2 (2026-03-24)

Technical correction to 1.1.1.

* Correction to the representation of extension contexts in the `hl7.fhir.uv.tools.r4` package

### 1.1.1 (2026-03-22)

Technical correction to 1.1.0.

* Mark the CDS Hooks definitions as not experimental
* Tidy up conformance statements

### 1.1.0 (2026-03-03)

* Publish the `expansion-parameters` extension
* CDS Hooks QA corrections
* Add the `lang-pack` IG parameter

### 1.0.0 (2026-02-04)

* Add the `term-params-in-artifacts` parameter
* Add the R6 inter-version Requirements parameters
* Add `requirements-category-vs`
* Add the `strict-identifiers` and `toggle-changes` parameters

### 0.9.0 (2025-12-16)

* Support for Additional Resources
* Various documentation improvements

### 0.8.0 (2025-08-05)

* Further refinements to the CDS Hooks definitions

### 0.7.1 (2025-07-26)

* Regenerate the IG to include elements missing from the R3 and R4 snapshots for the CDS Hooks content

### 0.7.0 (2025-07-21)

* Fix the missing context on the `snapshot-source` extension
* Add the `type-profile-style` extension, which was missed in 0.6.0

### 0.6.0 (2025-07-20)

* UML diagrams
* More IG parameters
* Revise the CDS Hooks definitions
* TestPlan changes
* Add the snapshot extensions
* Deprecate the resources that moved to the Extensions Pack
* General QA

### 0.5.0 (2025-04-17)

* Fix the definition of additional bindings
* Add the `pin-canonicals` and `r5-bundle-relative-reference-policy` IG parameters
* Define parameterised value sets for testing

### 0.4.1 (2025-03-11)

Technical correction to 0.4.0.

* Fix the paths in the R4 sub-package

### 0.4.0 (2025-03-10)

* Routine update: new extensions and IG parameters
* Add the TestCases definition

### 0.3.0 (2024-10-27)

* Routine update, to publish the r4 sub-package

### 0.2.0 (2024-04-26)

* Routine update

### 0.1.0 (2023-12-19)

* First stable release

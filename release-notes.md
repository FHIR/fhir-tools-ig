* Add validator-version extension - the validation tool stamps its own version and build date into the OperationOutcome it produces, so a report shows what produced it
* Add `not-supported`, `too-costly`, `version-error` and `business-rule` to the terminology issue types - the four remaining ways a terminology request can fail that had no code of their own, so every issue a server returns can now carry a `tx-issue-type`
* Add `cannot-determine` to the terminology issue types - for when a server declines to answer rather than risk answering wrongly (e.g. $subsumes, which has no outcome code meaning "unknown")
* Add `cache-id-duplicate` to the terminology issue types - a request that carries more than one cache-id
* Document terminology caching (here temporarily)
* Add resource-tla extension for a resource type's short mnemonic / three-letter acronym
* Add new incubator-ig parameter code - directs the publisher to use the generated code for an incubator IG in place of the base specification definitions
* Document UML parameters + set up UML examples as demonstrations
* Rework CDSHooks definitions to validate CDSHooks content properly
* Add snapshot-source extension for tracking the source of the snapshot - preparing in advance for wildcard dependencies
* rework Test Plan for & following discussions in Madrid 
* Add new excludeflags parameter code
* add no-cibuild-issues parameter
* add elementdefinition-string-format extension
* Added openEHR view flag
* Add support for suppressing JSON, TTL, and XML files in the build via parameters
* Add pin-manifest and suppress-mappings
* Clarify what format should i18n-lang values should be provided in


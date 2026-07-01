This page describes the `$cache-control` operation and the terminology client caching protocol it supports. It is intended for implementers of terminology clients and terminology servers.

### The problem

A terminology client (such as a validator or an IG publisher) typically validates or expands many resources against the same set of value sets and code systems. Where those value sets and code systems are not already known to the server &mdash; for example, the ones being defined in the implementation guide that is currently being built &mdash; the client has to send them to the server. Sending the full definitions on every `$validate-code` or `$expand` call is wasteful: the same large resources are serialized and transmitted over and over.

The caching protocol lets a client send each value set or code system to the server **once**, and then refer to it by url on subsequent calls. The server holds the registered resources in a cache, keyed by a **cache-id**.

### How it works

The protocol is explicit and server-driven:

1. **Start.** The client calls `$cache-control` with `mode=start`. The server creates a cache and returns its identifier in the `cache-id` output parameter. The client may front-load resources into the cache in this call (as `tx-resource` parameters), and may set the [`sealed`](#sealed-and-unsealed-caches) parameter to control whether the cache is fixed at creation or is allowed to grow.

2. **Use.** On subsequent `$validate-code` and `$expand` requests, the client sends the cache-id as the **`X-Cache-Id` HTTP header**. The server makes the resources registered under that cache-id available to the operation. In an [unsealed cache](#sealed-and-unsealed-caches), resources are populated by sending them &mdash; as `tx-resource`, or as the primary `valueSet`/`codeSystem` being validated/expanded &mdash; the first time they are needed; thereafter the client refers to them by `url` (and `valueSetVersion`/`version`) alone, and the server resolves them from the cache. In a sealed cache (the default), the cache holds only what was front-loaded at `start`, and does not grow.

3. **End.** When the client is finished (for example, at the end of a build), it calls `$cache-control` with `mode=end`, sending the cache-id in the `X-Cache-Id` header. The server releases the cache. If the client does not call `end` (for example, because it crashed), the server releases the cache on an idle timeout instead.

The cache-id is carried as an HTTP header, rather than as an operation parameter, so that it is transport metadata rather than terminological content: it can be acted on by proxies and load balancers, and read by the server before the request body is parsed.

### Why the server issues the cache-id

It is essential that the **server** allocates the cache-id (at `mode=start`), not the client. Because the server owns the identifier, it always knows which cache-ids are live. This lets it distinguish two situations that are otherwise indistinguishable:

* a request that refers to a cache-id the server has (resolve the registered resources), versus
* a request that refers to a cache-id the server does **not** have &mdash; one that was never created, or has expired, or has already been released.

The second case is reported as an error. The request fails with HTTP status **`404 Not Found`**, and the returned `OperationOutcome` has an issue whose `details.coding` carries the code [`cache-id-unknown`](CodeSystem-tx-issue-type.html) from `http://hl7.org/fhir/tools/CodeSystem/tx-issue-type`. `404` is the appropriate status because the cache the request named is, as far as the server is concerned, simply not there. The coded issue is deliberately a **distinct, coded** signal, and it &mdash; not the status code alone &mdash; is what a client should key on: it means something different from a value set or code system genuinely not being found. A client that sees `cache-id-unknown` knows its cache is gone (and could, if it chose, start a new cache and replay its registrations), whereas an unknown value set is an authoring problem. Without this distinction, a lost cache masquerades as a content error, and implementers waste time hunting for an authoring mistake that does not exist.

### Capability negotiation

A server advertises support for this protocol by declaring the `$cache-control` operation at the system level in its `CapabilityStatement`. A client uses the protocol only against servers that advertise it; against any other server it simply inlines the resources on every request (correct, just not optimised). The operation declares `affectsState = true`; conformant clients invoke `start` and `end` with `POST`.

### Sealed and unsealed caches

The `start` call takes a `sealed` parameter (`valueBoolean`) that governs whether the cache may change after it is created:

* **`sealed = true` (the default).** The cache contains exactly the resources front-loaded in the `start` call, and nothing else. Resources sent inline on later `$validate-code`/`$expand` requests are used for that request but are **not** added to the cache; the cache never grows. This is the safe default: the contents of the cache are fixed and fully determined by the client at the moment it is created, so the server's behaviour for a given cache-id is stable and does not depend on the order or timing of subsequent requests.

* **`sealed = false`.** The cache grows: each resource the server sees under this cache-id &mdash; whether sent as `tx-resource` or as the primary `valueSet`/`codeSystem` &mdash; is added to the cache the first time it is seen, and is thereafter resolvable by reference. This is more convenient for a client that discovers the resources it needs incrementally rather than knowing them all up front, but it makes the cache a piece of mutable, shared server state.

A client that requests an **unsealed** cache takes on responsibility for the consequences of that shared, mutable state. In particular, it must ensure that its own requests do not race against each other in ways that produce unexpected results &mdash; for example, it should not issue overlapping requests that populate and read the same cache concurrently, because whether a given resource is present depends on whether the request that adds it has already been processed. Sealing the cache avoids this class of problem entirely, which is why it is the default.

### Versionless references and caching

Caching does not change *which* resource a reference resolves to, but it can make disagreements about resolution more visible and more damaging. When a client refers to a code system or value set **without a version** &mdash; either the primary resource or a dependency referenced from one cached resource to another &mdash; the server must decide which version is "the latest". If the client and the server do not resolve a versionless reference to the *same* version (for example, because they have different packages loaded, apply different "current version" rules, or were populated at different times), they will silently operate on different definitions.

Caching amplifies this in two ways. First, a resolution made once against the cache is reused for the life of the cache, so a single disagreement is "frozen in" and repeated across every request that uses that cache-id, rather than being re-evaluated (and possibly self-correcting) each time. Second, because the client refers to a cached resource by url alone after the first send, a versionless reference that the client believes points at one version may be resolved by the server to another, and the mismatch is invisible on the wire. Implementers should therefore **pin versions explicitly** on references between terminology resources wherever the exact version matters, rather than relying on client and server independently agreeing on what "latest" means; caching makes an existing versionless-resolution disagreement worse, not better.

### Implementation notes

* The cache holds resource **definitions**, not expansions. A value set sent with an inline `expansion` has that expansion cached as supplied.
* Caches are scoped to the endpoint (and FHIR version) they were started on; a cache-id from one endpoint is not valid on another.
* A request that carries an unknown cache-id (in the `X-Cache-Id` header) on `$validate-code`, `$expand`, or any other operation fails with HTTP `404` and the `cache-id-unknown` issue described above. By contrast, `mode=end` for a cache the server does not have is deliberately **not** an error: it returns `200`, because the client's intent &mdash; that the cache be gone &mdash; is already satisfied.
* `mode=check` is reserved for a future revision: it will report whether a cache is still valid and may return statistics about it.

See the [$cache-control OperationDefinition](OperationDefinition-cache-control.html) for the formal operation definition, and the [Terminology Issue Type code system](CodeSystem-tx-issue-type.html) for `cache-id-unknown` and the other terminology issue codes.

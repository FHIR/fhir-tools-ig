This page describes the `$cache-control` operation and the terminology client caching protocol it supports. It is intended for implementers of terminology clients and terminology servers.

### The problem

A terminology client (such as a validator or an IG publisher) typically validates or expands many resources against the same set of value sets and code systems. Where those value sets and code systems are not already known to the server &mdash; for example, the ones being defined in the implementation guide that is currently being built &mdash; the client has to send them to the server. Sending the full definitions on every `$validate-code` or `$expand` call is wasteful: the same large resources are serialized and transmitted over and over.

The caching protocol lets a client send each value set or code system to the server **once**, and then refer to it by url on subsequent calls. The server holds the registered resources in a cache, keyed by a **cache-id**.

### How it works

The protocol is explicit and server-driven:

1. **Start.** The client calls `$cache-control` with `mode=start`. The server creates a cache and returns its identifier in the `cache-id` output parameter. The client may front-load resources into the cache in this call (as `tx-resource` parameters).

2. **Use.** On subsequent `$validate-code` and `$expand` requests, the client sends the cache-id as the **`X-Cache-Id` HTTP header**. The server makes the resources registered under that cache-id available to the operation. Resources are populated into the cache by sending them &mdash; as `tx-resource`, or as the primary `valueSet`/`codeSystem` being validated/expanded &mdash; the first time they are needed; thereafter the client refers to them by `url` (and `valueSetVersion`/`version`) alone, and the server resolves them from the cache.

3. **End.** When the client is finished (for example, at the end of a build), it calls `$cache-control` with `mode=end`, sending the cache-id in the `X-Cache-Id` header. The server releases the cache. If the client does not call `end` (for example, because it crashed), the server releases the cache on an idle timeout instead.

The cache-id is carried as an HTTP header, rather than as an operation parameter, so that it is transport metadata rather than terminological content: it can be acted on by proxies and load balancers, and read by the server before the request body is parsed.

### Why the server issues the cache-id

It is essential that the **server** allocates the cache-id (at `mode=start`), not the client. Because the server owns the identifier, it always knows which cache-ids are live. This lets it distinguish two situations that are otherwise indistinguishable:

* a request that refers to a cache-id the server has (resolve the registered resources), versus
* a request that refers to a cache-id the server does **not** have &mdash; one that was never created, or has expired, or has already been released.

The second case is reported as an error whose `OperationOutcome.issue.details.coding` carries the code [`cache-id-unknown`](CodeSystem-tx-issue-type.html) from `http://hl7.org/fhir/tools/CodeSystem/tx-issue-type`. This is deliberately a **distinct, coded** signal: it is not the same as a value set or code system genuinely not being found. A client that sees `cache-id-unknown` knows its cache is gone (and could, if it chose, start a new cache and replay its registrations), whereas an unknown value set is an authoring problem. Without this distinction, a lost cache masquerades as a content error, and implementers waste time hunting for an authoring mistake that does not exist.

### Capability negotiation

A server advertises support for this protocol by declaring the `$cache-control` operation at the system level in its `CapabilityStatement`. A client uses the protocol only against servers that advertise it; against any other server it simply inlines the resources on every request (correct, just not optimised). The operation declares `affectsState = true`; conformant clients invoke `start` and `end` with `POST`.

### Implementation notes

* The cache holds resource **definitions**, not expansions. A value set sent with an inline `expansion` has that expansion cached as supplied.
* Caches are scoped to the endpoint (and FHIR version) they were started on; a cache-id from one endpoint is not valid on another.
* `mode=check` is reserved for a future revision: it will report whether a cache is still valid and may return statistics about it.

See the [$cache-control OperationDefinition](OperationDefinition-cache-control.html) for the formal operation definition, and the [Terminology Issue Type code system](CodeSystem-tx-issue-type.html) for `cache-id-unknown` and the other terminology issue codes.

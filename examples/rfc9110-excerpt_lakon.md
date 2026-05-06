---
source: /home/abribus/workspace/lakon/examples/rfc9110-excerpt.txt
translated_at: 2026-05-06T08:58:11Z
skill: skills/translate
version: "1.0"
---

evolution over time.

HTTP also designed as intermediation protocol: proxies + gateways translate non-HTTP info systems → generic interface.

Consequence: protocol ¬ definable by what occurs behind interface. Limited to: syntax of communication · intent of received communication · expected behavior of recipients. Communication in isolation → successful actions ought to reflect in observable interface changes at servers. Since multiple clients may act in parallel + cross-purposes, ¬ require such changes be observable beyond single response scope.

## 1.2. History and Evolution

HTTP primary info transfer protocol for World Wide Web since 1990. Began as trivial mechanism for low-latency requests — single method (GET) to request transfer of presumed hypertext doc identified by pathname. Web growth → HTTP extended to: enclose requests + responses in messages · transfer arbitrary data formats via MIME-like media types · route requests through intermediaries. Protocols eventually defined as HTTP/0.9 and HTTP/1.0 (see [HTTP/1.0]).

HTTP/1.1 designed to refine features while retaining compatibility with existing text-based messaging syntax, improving interoperability, scalability + robustness across Internet. Included: length-based data delimiters for fixed + dynamic (chunked) content · consistent framework for content negotiation · opaque validators for conditional requests · cache controls for better cache consistency · range requests for partial updates · default persistent connections. HTTP/1.1 introduced 1995, published Standards Track 1997 [RFC2068], revised 1999 [RFC2616], revised again 2014 ([RFC7230] through [RFC7235]).

HTTP/2 ([HTTP/2]) introduced multiplexed session layer atop existing TLS + TCP protocols for concurrent HTTP message exchange with efficient field compression + server push. HTTP/3 ([HTTP/3]) provides greater independence for concurrent messages via QUIC as secure multiplexed transport over UDP instead of TCP.

All three major HTTP versions rely on semantics defined by this doc. ¬ obsoleted each other — each has specific benefits + limitations depending on context. Implementations expected to choose most appropriate transport + messaging syntax for their context.

This revision separates: semantics definition (this doc) + caching ([CACHING]) from HTTP/1.1 messaging syntax ([HTTP/1.1]) → each major protocol version progresses independently while referencing same core semantics.

## 1.3. Core Semantics

HTTP provides uniform interface for interacting with resource (Section 3.1) — regardless of type, nature, or implementation — by sending messages that manipulate or transfer representations (Section 3.2).

Each message = request | response. Client constructs request messages communicating its intentions → routes toward identified origin server. Server listens for requests · parses each message · interprets semantics relative to identified target resource · responds with ≥1 response messages. Client examines responses to verify intentions carried out — determines next action from status codes + content received.

HTTP semantics include: intentions defined by each request method (Section 9) · extensions possibly described in request header fields · status codes describing response (Section 15) · other control data + resource metadata possibly in response fields.

Semantics also include: representation metadata describing how content intended to be interpreted by recipient · request header fields possibly influencing content selection · the various

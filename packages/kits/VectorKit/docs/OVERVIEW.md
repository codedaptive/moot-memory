---
doc: OVERVIEW
package: VectorKit
repo: moot-memory
authored_commit: ecbe2bc361c83a1e8bc636767d33d0c678f88bd7
authored_date: 2026-07-04
sources:
  - path: Sources/VectorKit/EmbeddingProvider.swift
    blob: ad2bf52732b46960b9357a01fea37254d1681561
  - path: Sources/VectorKit/Engine/BruteForceIndex.swift
    blob: da3bdac5a6d2b84ff73e4ec66057bcc2acd2b2cb
  - path: Sources/VectorKit/Engine/DenseHit.swift
    blob: 22289f57f49923647e4b99092c134dbc1910c15e
  - path: Sources/VectorKit/Engine/DenseIndex.swift
    blob: 010a51a54b6d62c971943070115e8822d9ffeafb
  - path: Sources/VectorKit/Engine/DenseMetric.swift
    blob: a28578e73ec36943a73d067e7768782311fa2005
  - path: Sources/VectorKit/Engine/FloatBruteForceIndex.swift
    blob: 888d5a4079c84939b2cfde93160ee6bc3851adeb
  - path: Sources/VectorKit/Engine/MaxSimScorer.swift
    blob: 91875a79b8f6eebf6a2fd0a3a9dde85311a50aae
  - path: Sources/VectorKit/Engine/MIHIndex.swift
    blob: 61e283122542218eaf1f057cd7b9f1022930956f
  - path: Sources/VectorKit/Engine/ResidentArrayStore.swift
    blob: 21c67979dfc05d761909edec9700849d7cad74a5
  - path: Sources/VectorKit/Engine/ResidentVectorArray.swift
    blob: 6e0f689702e4173388324b22fd828559ce0b1ab2
  - path: Sources/VectorKit/Engine/VectorPayload.swift
    blob: 9259b4db9380cf9d854abd84a1d5059a0fcff5ec
  - path: Sources/VectorKit/Engine/VectorRecordKey.swift
    blob: bb4fff18c74c37ccafc9b1eaa01a4ec86b80be20
  - path: Sources/VectorKit/FloatSimHashEmbeddingProvider.swift
    blob: efcb85396ceace1373e3017c0f799371a9a5c3bf
  - path: Sources/VectorKit/StoredVector.swift
    blob: 44702eaf3a0e28ef7f70031fa05751b48a8ecfbf
  - path: Sources/VectorKit/VectorKit.swift
    blob: 0a6eba27a0501601ee9ac015875de6d71bd4cf05
  - path: Sources/VectorKit/VectorKitError.swift
    blob: 89c486eba6992edd583649e37674a67ea95ee317
  - path: Sources/VectorKit/VectorMatch.swift
    blob: 24cc2c1bd25f71a7cef60a043c3a640df2368a23
  - path: Sources/VectorKit/VectorStore.swift
    blob: 3c7fe4a19eba1142ac82a993cee0e7660a4ffdce
---

# VectorKit Overview

## What This Library Does

VectorKit turns a piece of text into a vector — a fixed-size code that
stands in for the text's meaning — and stores that vector so a later query
can find the most similar ones. MOOTx01 is an on-device AI memory system.
It stores what an AI observes over time and helps the AI recall it later.
VectorKit is the part of MOOTx01 that answers "what stored memories are
most like this one?"

VectorKit stores two kinds of vector for the same piece of text. The first
is a 256-bit binary fingerprint called an `Engram`, defined by a sibling
library, EngramLib. A fingerprint is a short fixed-size code computed from
a piece of content; similar content produces similar fingerprints, so the
system compares things quickly without reading them in full. Two Engrams
are compared by Hamming distance — the number of bit positions where they
differ; smaller means more similar. The second kind is a dense float
vector: a list of several hundred decimal numbers produced directly by an
embedding model such as MiniLM. VectorKit keeps both because they serve
different needs, explained below.

## The Problem It Solves

An AI memory system needs to answer "find memories like this one" without
sending private text to a server. VectorKit runs entirely on the device
that captured the memory.

A 256-bit fingerprint is compact and fast to compare — comparing two
fingerprints is counting differing bits, pure integer arithmetic. Because
the comparison never uses floating-point math, it produces the exact same
answer on every device and every operating system. VectorKit calls this
property "four-way" determinism, and it never computes a fingerprint
comparison itself: every comparison is delegated to EngramLib, which in
turn delegates to a shared, conformance-gated kernel (a conformance
fixture is a recorded input/output pair both an original and a ported
implementation must reproduce exactly; EngramLib's kernel is checked this
way across four build configurations). This is spec I-7 in VectorKit's own
design documents: the kit performs no Hamming math of its own.

A fingerprint is compact, but compacting hundreds of numbers into 256 bits
throws information away. Some queries need the finer detail the original
float numbers carry — for example, telling a passage from its own echoed
question apart, which the fingerprint's collapsed representation cannot
always do. For those queries VectorKit also stores the float vector and
compares it with cosine distance, a measure of the angle between two
vectors. Float math is reproducible on one platform and one build, but it
is not guaranteed to produce byte-identical results on a different
platform. VectorKit documents this openly as a boundary, not a defect: the
binary fingerprint lane is the four-way-identical lane, and the float lane
is the "reproducible within one configuration" lane.

Every stored vector is tagged with the model identifier and model version
that produced it. Two different embedding models turn the same word into
different numbers for reasons that have nothing to do with meaning, so
comparing vectors from different models produces a meaningless answer.
VectorKit enforces this rule, called spec I-4, at multiple levels: storage
partitions vectors by model, and every search is scoped to one model.

Finally, an on-device memory store must stay fast as it grows. Scanning
every stored vector for every query works fine at a few thousand records
but becomes slow at scale. VectorKit solves this by keeping an in-memory
copy of all fingerprints (so no per-query database read is needed) and by
switching, once the count of live fingerprints crosses a threshold, from a
full scan to a sub-linear search structure that returns the identical
answer faster.

## How It Works

Writing a vector has two parts. `VectorStore.addPayload` first writes the
vector as one row to a `vectors` table, which is the durable source of
truth — nothing is ever considered stored until this write succeeds. It
then mirrors a binary vector into an in-memory packed array (or, for a
float vector, into a per-model in-memory array) so that later searches
never have to re-read the database. The in-memory array can optionally be
backed by an on-disk cache file, called a sidecar, so that reopening the
store does not require rebuilding the array from every database row.

Searching has two independent paths, one per vector kind. A binary search
(`findNearest`) compares the query fingerprint against the in-memory array
using Hamming distance. Below a configurable threshold of live fingerprints
(50,000 by default), the search does a full linear scan — always fast
enough at that scale. At or above the threshold, VectorKit switches to a
technique called Multi-Index Hashing, which slices each 256-bit fingerprint
into several shorter bands and uses per-band lookup tables to rule out most
of the collection without touching it. Multi-Index Hashing is provably
exact: it returns precisely the same neighbors the full scan would have
returned, just faster. A test suite checks this by running both searches
on the same random and adversarial inputs and requiring identical output.

A float search (`findNearestFloat`) compares the query's float vector
against a separate in-memory array holding only that model's float
vectors, using cosine, Euclidean, or dot-product distance. Because
different models produce vectors of different length, VectorKit keeps one
float array per model rather than mixing them.

A third comparison method, `MaxSimScorer`, serves models that produce many
small vectors per item instead of one — for example, one fingerprint per
word, an approach known as ColBERT-style late interaction. Rather than
compare one vector to one vector, it compares every query-word fingerprint
against every document-word fingerprint and keeps, for each query word,
the best match found in the document; the document's overall score is the
sum of those best matches. This scorer is exhaustive: it never skips a
candidate document, which makes it the correctness reference for any
faster method built later.

## How the Pieces Fit

Figure 1 shows the library's topology — its major parts and how data moves
between them.

![Figure 1. Topology of VectorKit](topology.svg)

*Figure 1. Topology of VectorKit. Text enters through an `EmbeddingProvider`
and becomes an Engram and, optionally, a float vector. `VectorStore` writes
both to the durable `vectors` table and mirrors them into in-memory
resident arrays. Reads dispatch through the `DenseIndex` seam to one of
three interchangeable search engines. The dashed regions mark the durable
storage boundary and the optional on-disk cache.*

`EmbeddingProvider` is the seam a host application implements: it supplies
whatever inference technique turns text into numbers (a CoreML model, for
instance). VectorKit's own concrete implementation,
`FloatSimHashEmbeddingProvider`, takes those numbers and projects them into
an Engram fingerprint using a shared substrate primitive, `FloatSimHash`,
so that every provider in the MOOTx01 kit graph produces fingerprints the
same deterministic way.

`VectorStore` is the actor every caller talks to. It owns the durable
`vectors` table (through a PersistenceKit `Storage` backend such as
SQLite) and the resident in-memory arrays. Three foundation types flow
through every layer beneath it: `VectorRecordKey` (which record this is),
`VectorPayload` (the raw typed bytes of one vector), and `DenseHit` (one
scored search result). All three are shared, additive-only types — no
search engine defines its own private version of them.

Underneath `VectorStore` sits the `DenseIndex` protocol, a pluggable engine
seam. Three concrete engines implement it: `BruteForceIndex` (the always-
correct binary linear scan and the conformance reference), `MIHIndex` (the
sub-linear binary search gated against `BruteForceIndex`), and
`FloatBruteForceIndex` (the float-lane linear scan, built once per model).
`VectorStore` decides which binary engine is active by comparing the live
fingerprint count against `mihThreshold`, and swaps between them without
rebuilding either — both are always kept in sync with every write.

`ResidentArrayStore` manages the optional `.vec` sidecar file: a packed,
fixed-format binary cache of the in-memory array. It is a regenerable
cache, never a second source of truth — if it is missing, stale, or
corrupted, VectorStore rebuilds it from the `vectors` table, the only
durable source.

## What Ships in the Package

The package ships the Swift sources listed above, a mirrored Rust
implementation in `rust/`, and no bundled data artifacts — VectorKit has
no fixed reference tables of its own; everything it stores comes from
caller-supplied text and caller-supplied embedding models.

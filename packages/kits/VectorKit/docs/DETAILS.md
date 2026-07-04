---
doc: DETAILS
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

# VectorKit Details

This document walks through every source file in the package. Read
`OVERVIEW.md` first for the big picture. Files appear here in pipeline
order: the module surface and errors, the embedding seam, the shared
engine foundation types, the three search engines, the resident-array
storage layer, and finally the storage-facing types and the `VectorStore`
actor that ties everything together.

## VectorKit.swift

This file provides the module surface: a short header comment naming the
kit's public pieces. It defines no types of its own.

The file explains a boundary worth restating here: VectorKit imports
EngramLib internally to use the `Engram` fingerprint type, but it does not
re-export that import. A caller that wants to construct an `Engram`
directly must `import EngramLib` itself. This keeps every package's import
list explicit, matching the convention used across the rest of the
substrate-dependent kits.

## VectorKitError.swift

This file provides `VectorKitError`, the single error type for every
VectorKit operation. Per MOOTx01 convention, errors are concrete named
cases rather than a generic failure plus a logged message, so a caller can
branch on exactly what went wrong.

`embeddingFailed(String)` reports an inference failure inside an
`EmbeddingProvider`. `modelUnavailable(String)` reports a model that is not
loaded on the current device. `storeUnavailable(String)` reports a failure
opening the backing storage. `notFound` is reserved for a future throwing
read path; today's read functions return `nil` instead. `invalidPayload`
covers a structurally broken `VectorPayload` — wrong kind, wrong byte
count, or a dimension mismatch — and is thrown by the payload's own
decode functions and by every search engine's input validation.
`decodingFailure` covers a malformed key or row that cannot be decoded.

Two cases record product decisions rather than plain bugs.
`int8QuantizationPolicyUndefined` is thrown whenever a caller tries to
write an `.int8` (quantized) vector: the rules for how to quantize and
later reverse the quantization have not been agreed on yet, so writing one
now would lock in behavior nobody has approved. The case documents that
this is deliberate and reversible — once a policy is ratified, the guards
that throw this error are meant to be removed. `embedFloatVocabMiss` is
thrown by embedding providers whose vocabulary is fixed in advance (for
example, statistical models trained on a specific word list) when none of
a query's words are in that vocabulary; it is distinct from
`embeddingFailed` because the provider is working correctly — the input
simply has nothing in common with what the provider knows.

## EmbeddingProvider.swift

This file provides the `EmbeddingProvider` protocol, the single seam
between "text" and "vector" that every embedding source in VectorKit's kit
graph implements.

The protocol is deliberately narrow: a `modelID` and `modelVersion` (the
tags every stored vector must carry, per spec I-4, so that vectors from
incompatible models are never compared), and three ways to turn text into
numbers. This narrowness is what lets VectorKit remain agnostic about
where inference happens — CoreML, ONNX, a hand-written statistical model —
and let a concrete provider such as `FloatSimHashEmbeddingProvider` or a
sibling package's MiniLM adapter fill in the details.

`embed(_:)` is the primary method: it returns the 256-bit binary Engram
for a piece of text, throwing `embeddingFailed` or `modelUnavailable` on
failure. Every conformer must return the substrate's canonical zero Engram
for an empty string, because this is the one input every provider is
guaranteed to agree on, and treating it specially lets empty records from
different providers land on the same "identical" partition instead of
colliding by coincidence.

`embedFloat(_:)` returns the dense float vector a provider computed on its
way to the Engram, before that vector was compressed into 256 bits. The
protocol's default implementation simply throws `embeddingFailed`, meaning
the float lane is opt-in: a provider that has nothing more precise to
offer than the fingerprint declines rather than fabricate numbers.
Providers that do real inference (MiniLM, mpnet, EmbeddingGemma, defined
in a sibling package) override this to return the vector they already
computed, at no extra inference cost. Empty input returns an empty array,
never a vector of zeros, because a vector of zeros would look like a
legitimate — and misleading — nearest neighbor to every genuinely empty
text.

`embedPair(_:)` is a convenience for callers that need both outputs from
one text. Its default implementation calls `embed` and then `embedFloat`
— two inference passes — swallowing a float-lane opt-out into an empty
array so the default matches historical two-call behavior. A provider that
computes both outputs from a single inference pass should override this to
avoid running its model twice.

`embedBatch(_:)` embeds several texts, defaulting to a sequential loop;
providers with genuinely batched inference should override it for
throughput. Output order always matches input order.

## FloatSimHashEmbeddingProvider.swift

This file provides `FloatSimHashEmbeddingProvider`, the one concrete
`EmbeddingProvider` VectorKit ships. It is a mirror of the Rust
`vectorkit::FloatSimHashEmbeddingProvider`.

The type is a thin wrapper: it holds a `modelID`, a `modelVersion`, a
`projectionSeed`, and an injected inference closure — `(String) async
throws -> [Float]` — supplied by whatever host owns the actual model.
VectorKit itself never loads a model or owns a tokenizer; concrete text
providers that do (MiniLM, mpnet, EmbeddingGemma) live in a different
package, CorpusKitProviders, and conform to this same protocol using this
type as their low-level building block.

`embed(_:)` short-circuits to the canonical zero Engram for empty input
before ever calling the inference closure, guaranteeing the empty-string
contract holds even if the closure itself would have produced something
non-zero. Otherwise it calls the inference closure and passes the result
through `SubstrateML.FloatSimHash.project(vector:seed:)` — a shared,
conformance-gated substrate primitive that projects an arbitrary-length
float vector down to a 256-bit fingerprint using a technique called
SimHash. The `projectionSeed` matters here: different seeds turn the same
float vector into different fingerprints, which is what keeps two
different models' outputs from accidentally landing in the same fingerprint
space and being wrongly compared (spec I-4's rule, enforced at the
projection layer as well as at storage).

`embedFloat(_:)` returns exactly the float vector the inference closure
produced — the same vector `embed(_:)` feeds into the projection — so a
caller using both lanes never pays for two inference passes.

## Engine/VectorRecordKey.swift

This file provides `VectorRecordKey`, the identifier every stored vector
carries inside the search engines. Note that this file lives under
`Engine/`, VectorKit's internal folder for the search-engine machinery
shared by every index implementation.

A key used to be a two-part (item, model) pair, which assumed one vector
per item per model. That assumption breaks for models like ColBERT that
produce one small vector per word instead of one vector per document, so
the key grew a third field. `itemID` names the owning record (a drawer or
a text chunk, as a UUID string). `vectorIndex` is the position of this
vector within its item's sequence — `0` for the ordinary single-vector
case, `0` through `N-1` for a multi-vector item. `modelID` and
`modelVersion` complete the tuple, because vectors from different model
versions are never comparable (spec I-4).

`VectorRecordKey` is `Comparable`, ordered lexicographically by
`(itemID, vectorIndex, modelID, modelVersion)`. This order is not
incidental: it is the tie-break every search result uses when two matches
land at the same distance, and it is the order the on-disk resident array
is built in, so both the search output and the storage layout agree on
what "sorted" means.

## Engine/VectorPayload.swift

This file provides `VectorPayload`, the one envelope every vector's raw
bytes travel in, plus `VectorKind` (the tag for which numeric family the
bytes represent) and `VectorPayloadInput` (a payload bundled with its
storage metadata, used for bulk writes).

`VectorKind` has three cases. `.binary` (raw value 0) is exactly the
32-byte Engram wire form. `.float32` (raw value 1) is `dim × 4` bytes of
IEEE-754 numbers. `.int8` (raw value 2) is reserved for a future quantized
representation; the case exists so that a future ratified quantization
policy does not require a new wire format, but `VectorStore` currently
rejects every `.int8` write (see `VectorKitError.int8QuantizationPolicyUndefined`
above). The raw values are stored on disk and must never be reordered.

`VectorPayload.init(engram:)` builds a binary payload directly from an
Engram's wire bytes — no conversion, no loss, so every existing binary
test still holds. `init(floats:)` serializes a `[Float]` to little-endian
IEEE-754 bytes explicitly, byte by byte, rather than relying on the
platform's native byte order. This is a deliberate choice, not an
oversight: it is what lets the on-disk `.vec` sidecar be read back
identically on an Apple device and on a Linux server, which may not share
the same native byte order convention. `asEngram()` and `asFloats()`
reverse these conversions, throwing `invalidPayload` when the payload's
kind or byte count does not match what was asked for.

## Engine/DenseHit.swift

This file provides `DenseHit`, the one result shape every search engine
returns, and `LaneTag`, an enum naming which retrieval technique produced
a given score (used by fusion and multi-technique retrieval code outside
this package).

A `DenseHit` carries the matched `key`, a `rawDistance`, and the `metric`
that produced it. The tricky design point is that `rawDistance` is a
single `Int32` field shared by very different kinds of number: an integer
Hamming distance for the binary lane, or the bit pattern of a `Float`
distance for the float lane. Two computed properties translate it back:
`hammingDistance` simply casts it to `Int` (safe because Hamming distances
are always 0–256), and `floatDistance` reconstructs the `Float` from its
stored bit pattern. Packing both families into one field, rather than
giving each lane its own result type, is what lets code elsewhere in the
kit graph handle a `[DenseHit]` without caring which lane produced it.

The file's header calls out an "additive-only" rule: any future field
added here must have a default value, so existing callers who build a
`DenseHit` with the current initializer keep compiling. This matters
because both a Swift and a Rust version of this type exist, and the two
must stay in lockstep.

## Engine/DenseMetric.swift

This file provides the metric vocabulary for the whole engine seam:
`BinaryMetric` (`.hamming`, `.jaccard`), `FloatMetric` (`.cosine`, `.l2`,
`.dot`), and `DenseMetric`, the umbrella enum wrapping either family so
that `DenseIndex.search` needs only one metric parameter regardless of
which lane it routes to.

The file's real content is its documentation of a determinism boundary
that recurs throughout this package: `.binary(.hamming)` is "four-way"
bit-identical, because it is pure integer arithmetic computed by a shared,
conformance-gated kernel; `.binary(.jaccard)` is bit-identical up through
its two integer counts, with one final IEEE-754 division that is itself
guaranteed identical because IEEE-754 mandates exact rounding for basic
operations; and every `.float(_)` metric is reproducible only within one
build and platform, never guaranteed identical between Swift and Rust.
This is stated as a documented property of floating-point math, not a
defect to be "fixed" — a warning aimed squarely at a future reviewer who
might otherwise try to force float parity that the underlying arithmetic
cannot honestly provide.

## Engine/DenseIndex.swift

This file provides the `DenseIndex` protocol — the single seam that lets
`VectorStore` treat three very different search engines
(`BruteForceIndex`, `MIHIndex`, `FloatBruteForceIndex`) as
interchangeable — along with three supporting types: `IndexKind` (a tag
naming which implementation is behind a given index, for tests that need
to pick the brute-force oracle deliberately), `SearchDirection`, and
`MetadataFilter`.

`SearchDirection` distinguishes `.nearest` (most similar first, the
default) from `.farthest` (most dissimilar first). Farthest search
supports an "find things unlike this" query. It is not a trick of negating
a nearest-neighbor list — the farthest items are not among the nearest
top-k at all — so the index has to scan and sort toward the opposite end
using the exact same distance calculation, just the opposite sort
direction.

`MetadataFilter` restricts a search to one `modelID` and, optionally, one
`modelVersion`. Its `accepts(_:)` method is the single predicate every
engine calls per-candidate; a `nil` field is a wildcard.

The protocol itself declares four operations — `build(from:)`,
`search(probe:metric:k:filter:)`, `add(key:vector:)`, and
`remove(key:)` — and documents the contract every conformer must honor:
results come back sorted by distance ascending, ties broken by the
matched key ascending, and `BruteForceIndex` is the correctness oracle
every other binary engine is measured against.

## Engine/BruteForceIndex.swift

This file provides `BruteForceIndex`, the exact linear-scan search engine
for binary (Hamming) vectors, and the conformance oracle every other
binary engine — currently just `MIHIndex` — is checked against.

The file is built around one hard rule, restated three times in its
comments: it performs zero Hamming arithmetic itself. Every distance is
computed by `EngramLib.distances`, which routes to a shared kernel
selected once per process (NEON on Apple silicon hardware, a scalar
fallback elsewhere) and checked for identical output across four build
configurations. Reimplementing a bitwise XOR-and-count here, even a
correct one, would bypass that check and risk silent divergence between
platforms — this is the file's version of spec I-7.

`search(probe:metric:k:filter:)` validates the probe (must be exactly 32
bytes of `.binary` kind) and the metric (only `.binary(.hamming)` is
supported; other requests throw `invalidPayload`), narrows the scan to one
model's slot range when a filter is present (an `O(log m)` lookup into a
sorted partition index, avoiding a full-array walk), collects the live,
un-tombstoned candidates in that range, and hands their Engrams to
`EngramLib.distances` in one batch call. It deliberately avoids
`EngramLib.findNearest`, which applies a different, insertion-order
tie-break; this file sorts the returned distances itself by
`(distance ascending, key ascending)`, because the engine's own contract
requires the full `VectorRecordKey` as the tie-break, not just the array's
insertion position — otherwise two records under the same item but
different model or vector index could be returned inconsistently.

`add(key:vector:)` implements upsert by tombstoning any existing slot with
the same key before appending the new bytes, then rebuilding the sorted
model-partition index from the updated key list. `remove(key:)` tombstones
every matching slot without touching the underlying storage bytes — actual
space reclamation is `ResidentArrayStore`'s job, not this type's.
`currentSnapshot()` returns a value-type copy of the live array so callers
outside the actor (chiefly `VectorStore`, when it needs to scan for
tombstoning) can read it safely.

## Engine/FloatBruteForceIndex.swift

This file provides `FloatBruteForceIndex`, the linear-scan search engine
for the float32 lane — cosine, Euclidean (`l2`), and dot-product distance
— and, unlike the binary lane, this is both the correctness reference and
the production search path: there is no separate accelerated float engine
in this package.

The file opens with an emphatic warning, repeated in this document because
it protects against a plausible but wrong "fix": float arithmetic here is
reproducible on one build and platform, but it is not, and cannot be
made to be, bit-identical between Swift and Rust or across different
hardware. This is a documented property of IEEE-754 arithmetic, not an
oversight; a reviewer must not try to force it to match the binary lane's
four-way guarantee.

`build(from:)` simply stores a reference to the supplied array — there is
no secondary structure to construct, so building is `O(1)`; the real cost
is whatever the caller paid to assemble the array. `search(probe:metric:
k:filter:)` validates that the probe is `.float32`, that the requested
metric is a float metric, and that the probe's byte count matches both its
own declared dimension and the array's fixed stride (a mismatch would
otherwise read past the end of a slot and throws instead). It then scans
every live, filter-passing slot, computing one of three distances per
candidate — cosine distance treats a zero vector as maximally distant
rather than crashing on a divide-by-zero; `l2` is the plain Euclidean
formula; `dot` is negated so that "smaller is nearer" holds for every
metric uniformly — and sorts ascending by distance, then by key.
`searchFarthest(probe:metric:k:filter:)` reuses the identical scan and
identical distance math, changing only the sort direction to descending,
which is what makes "find dissimilar items" a real bottom-of-the-list scan
rather than a negated top-of-the-list one.

`add(key:vector:)` establishes the array's dimension from the first vector
added and rejects any later vector of a different byte count, because a
mismatched stride would silently corrupt the flat storage buffer.
`remove(key:)` tombstones the matching slot; actual compaction happens the
next time `build(from:)` runs with a freshly assembled array.

## Engine/MaxSimScorer.swift

This file provides `MaxSimScorer` and `MaxSimHit`, the exhaustive
("Exact-A") implementation of ColBERT-style late-interaction scoring over
binary token fingerprints.

Some embedding techniques represent one document as many small vectors —
one per word or token — rather than one vector for the whole document.
Comparing two such documents means asking, for every word in the query,
"which word in this document is most like it?" and adding up those best
matches. That sum, `Σ (256 − minimum Hamming distance)` over every query
token, is the MaxSim score this file computes. Because it examines every
query token against every document token for every candidate document, it
never skips a candidate; this exhaustiveness is precisely what makes it
the correctness reference for any faster, pruned variant built later — the
file's header explicitly reserves the accelerated two-stage variant as out
of scope here.

`score(queryTokens:documents:k:)` iterates the supplied documents in
ascending itemID order — sorting the dictionary's keys explicitly, because
a Swift dictionary's own iteration order is not guaranteed and would make
results non-reproducible — computes each document's MaxSim score, sorts
the results `(score descending, itemID ascending)`, and truncates to `k`
only after the full sort, never before, so a document that would have
scored well is never cut for appearing late in an unsorted pass. Every
Hamming distance again goes through `EngramLib.Session.distances`,
constructed once per `MaxSimScorer` and reused for the whole call, so the
one-time cost of picking the fastest available kernel is paid once rather
than per comparison.

## Engine/ResidentVectorArray.swift

This file provides `ResidentVectorArray`, the packed in-memory data shape
every search engine reads from, and `ModelPartitionEntry`, one entry in
its per-model index. This is the shared contract underneath
`BruteForceIndex`, `MIHIndex`, and `FloatBruteForceIndex` — all three read
the identical layout, which is what lets `VectorStore` build the array
once and hand it to whichever engine is currently active.

The design reason is stated directly in the file's header comment:
measurement on the pre-existing code path showed that fetching and
decoding rows from the database consumed 87% of a search's latency, while
the actual distance kernel took 0.4%. A fixed-stride, contiguous byte
array removes the fetch-and-decode cost from every query after the first,
because the whole array is loaded once and then scanned as a flat block of
memory with no per-row allocation.

The type stores `kind` and `stride` (bytes per vector slot — 32 for
binary, `dim × 4` for float32), `count` (including tombstoned slots),
`storage` (the packed bytes themselves — on Apple platforms potentially
memory-mapped read-only from a sidecar file), a `keys` array parallel to
`storage`, a sorted `modelPartitions` index, and a `tombstones` bitmap.
`liveCount` walks the tombstone bitmap to compute how many slots are still
valid; `partitionRange(for:)` binary-searches the sorted partitions to
find one model's slot range in `O(log m)`; `isTombstoned(_:)` and
`vectorBytes(at:)` are the two per-slot accessors every engine's scan loop
calls.

## Engine/ResidentArrayStore.swift

This file provides `ResidentArrayStore`, the actor that owns the optional
on-disk `.vec` sidecar file: a packed binary cache of a
`ResidentVectorArray` that lets a reopened store skip rebuilding the array
from every database row.

The file documents its own on-disk format in full — a fixed header
(magic bytes, format version, vector kind, stride, count, a live-slot
count, and the tombstone bitmap), followed by the packed vector bytes,
then variable-length key records, then the model-partition index, with
every multi-byte integer explicitly little-endian so the same file reads
identically on an Apple device and on a Linux server. `writeSidecar`
writes to a temporary file and atomically renames it into place, so a
crash mid-write never leaves a half-written sidecar behind; `readSidecar`
memory-maps the file where the platform supports it (a load-time
optimization, not a difference in the bytes returned) and `parseSidecar`
does the actual decoding, checking every length field against the
remaining buffer size before trusting it, so a corrupted or hand-edited
file is rejected with a `decodingFailure` rather than crashing the
process.

The file is explicit about one policy: the `vectors` database table is
always the durable source of truth, and this sidecar is a regenerable
cache, never a second copy of record. `load()` reads the sidecar if
present; if it is missing or fails to parse, the store simply starts
empty and waits for `VectorStore` to rebuild it from the table.

Three write paths exist because a single write policy could not serve
both a low-latency single insert and a large bulk import well.
`append(key:bytes:)` is the eager path: it writes the sidecar
immediately after every single addition. `appendDeferred(key:bytes:)`
is the "write-behind" path a single insert uses in production: it updates
the in-memory array and marks the store dirty without touching disk,
trusting the caller to call `flush()` at a natural pause. This is safe
because the database row was already written durably before this call —
losing an unflushed sidecar only costs a rebuild on the next open, never
data. `appendBatch(records:)` is the bulk-import path: it extends storage,
keys, and the tombstone bitmap for the whole batch in one pass and writes
the sidecar exactly once, so importing a thousand vectors costs one disk
write instead of a thousand.

`compact()` rewrites the sidecar keeping only live (non-tombstoned)
records, sorted by key for a deterministic, reproducible layout, and is
triggered automatically whenever the tombstone ratio exceeds
`compactionThreshold` (25% by default) after any eager write.

## StoredVector.swift

This file provides `StoredVector`, the public, fully-decoded row shape
`VectorStore.vectors(forItemID:)` returns to callers who want the
convenient binary form rather than the raw typed payload.

Its fields mirror the `vectors` table's columns directly: a stable `id`
assigned on insert, the owning `itemID`, the `vectorIndex` position within
a multi-vector item, `modelID` and `modelVersion` (spec I-4's tags),
the decoded `engram`, and `filedAt`, the time the row was written,
round-tripped through the database's text-based ISO 8601 timestamp column
(which loses sub-millisecond precision — the file notes this explicitly so
a caller comparing timestamps at fine granularity is not surprised).
`StoredVector.engram` is non-nil only for binary rows; a float or int8 row
must be read through `VectorStore.getPayload` instead, since this
convenience type only round-trips the binary case.

## VectorMatch.swift

This file provides `VectorMatch`, the public search-result shape
`VectorStore.findNearest` and its float-lane counterparts return.

It carries `itemID` (the matched record), `distance` (Hamming distance for
the binary lane, an integer in 0…256; a scaled, quantized cosine distance
for the float lane, explained in `VectorStore`'s float-search functions),
and `modelID`, so a caller can confirm which model actually produced the
match. `VectorMatch` conforms to `Comparable`, ordered by `distance`
ascending with ties broken by `itemID` ascending — the same universal
tie-break rule used throughout the engine layer — so a sorted array of
matches reads nearest-to-farthest from front to back without a caller
having to know the sort convention.

## VectorStore.swift

This file provides `VectorStore`, the actor every consumer of VectorKit
actually talks to. It is the largest file in the package because it is
where every other piece — the durable table, the resident arrays, the
three search engines, and telemetry — is wired together into one
consistent API.

### Storage and schema

`VectorStore` wraps a PersistenceKit `Storage` backend (SQLite, an
in-memory backend for tests, or, in the future, PostgreSQL); the kit never
sees which backend is chosen — that decision belongs to the application.
`schemaDeclaration` is the static schema description passed to
`storage.open(schema:)` before the store is used: one `vectors` table,
version 3, whose `UNIQUE(item_id, vector_index, model_id)` constraint is
exactly `VectorRecordKey` minus `modelVersion` — the constraint that makes
an upsert on a changed model version a true replacement of the old row
rather than a duplicate.

### Two hot-path structures kept in sync

Every write updates three things together: the durable `vectors` table
row, the in-memory resident array (through `bruteForceIndex` and
`mihIndex`, which are both always kept current — only one is ever the
active `hotIndex`), and, when a sidecar was configured at construction,
the on-disk cache. `_selectIndex()` compares `liveBinaryCount` against
`mihThreshold` (50,000 by default) after every write that changes the
count, swapping `hotIndex` between `bruteForceIndex` and `mihIndex` with a
plain reference assignment — no rebuild is needed on promotion or
demotion, because both indexes were already current. `init(storage:
sidecarURL:mihThreshold:mihBandCount:deferredPendingLimit:)` allocates both
index actors up front so this swap never needs to construct anything at
query time.

### Write path

`addVector(itemID:engram:modelID:modelVersion:filedAt:)` is a convenience
wrapper for the common single binary vector case; it builds a
`VectorPayload` and delegates to `addPayload`. `addPayload(itemID:
vectorIndex:payload:modelID:modelVersion:filedAt:)` is the general write:
it rejects `.int8` payloads immediately (see `VectorKitError` above),
writes the row via an upsert keyed on the table's unique constraint, and
then — only for `.binary` and `.float32` kinds — mirrors the write into
the matching resident array. For a binary write, it first finds any
existing slot at the same logical position (`itemID`, `vectorIndex`,
`modelID`, ignoring `modelVersion`) so that a version change is treated as
a true replacement rather than leaving a stale duplicate slot behind; this
matching is deliberately looser than full key equality specifically to
catch that case. It then tombstones the stale slot (if any) and appends
the new one in both `bruteForceIndex` and `mihIndex`, updates
`liveBinaryCount` only when the write was genuinely new, and calls
`_selectIndex()`. Every write emits a `vectorkit.index.insert_latency_ms`
telemetry metric through IntellectusLib — a short-circuited no-op unless
monitoring has been explicitly turned on, so the cost on the default path
is one boolean check.

`addPayloads(_:)` is the bulk-import counterpart, and its whole reason for
existing is complexity: importing N vectors one at a time through
`addPayload` costs N sidecar rewrites and, without care, N index rebuilds.
This function upserts every row to the table (unavoidable — the table is
the durable source), then rebuilds both binary indexes exactly once from
the final merged array rather than once per row, cutting the amortized
cost from `O(N²)` bytes written to `O(N)`. `beginDeferredIndex()` and
`publishResidentIndex()` extend this further for very large or
multi-call bulk imports: while a deferred window is open, `addPayloads`
appends to storage but skips the index rebuild entirely, seeding an
in-memory tracked set of live keys so replacement detection stays cheap
across the whole window; `publishResidentIndex()` performs the single
rebuild the whole burst needed, once, when the caller signals the burst is
finished. `deferredPendingRecords`, capped at `deferredPendingLimit`
(50,000 by default), guards the memory-only variant of this path against
unbounded growth if a caller holds the window open indefinitely; crossing
the cap triggers `_flushDeferredPending()`, an internal intermediate merge
that keeps the deferred window open for the caller while bounding peak
memory.

### Search path

`findNearest(probe:modelID:limit:)` is the binary-lane search: it lazily
builds the resident array on first use (from the sidecar if one is
current, or from the table if not), then delegates entirely to
`hotIndex.search`, converting the returned `[DenseHit]` into
`[VectorMatch]` without re-sorting — the engine has already applied the
required `(distance ascending, itemID ascending)` order.
`findNearestFloat(probe:modelID:limit:)` is the float-lane equivalent: it
lazily builds a `FloatBruteForceIndex` per model (the map's presence is
the "already built" flag — there is no separate boolean), then searches
by cosine distance and quantizes the resulting `Float` distance to an
integer by multiplying by 10,000 and rounding, so results from different
languages' fixtures can be compared exactly rather than approximately.
`findFarthestFloat(probe:modelID:limit:)` is identical except it calls the
engine's farthest-ranking search, for an anti-similarity "find things
unlike this" query. `findByKeyword(_:limit:)` is a plain substring match
on `item_id`, explicitly documented as a quick pre-filter for hybrid
retrieval, not a real keyword search — full keyword scoring is a different
package's responsibility.

### Delete path

`deleteVector(itemID:modelID:)` deletes the row at `vectorIndex` 0 and
tombstones every matching resident slot. `deleteAllVectors(itemID:
modelID:)` deletes every vector index for an item and model, used when a
multi-vector item (all its ColBERT token vectors, for instance) needs
complete removal. Both first flush any in-flight deferred-index burst,
so a delete never races an unpublished bulk import. `destroyAllVectors()`
wipes the entire store — every row, both resident indexes, the sidecar,
and the per-model float indexes — as part of a coordinated estate
teardown; an estate is one user's complete memory store in MOOTx01.

### Coherence helpers

`_ensureIndexBuilt()` is the one-time (per process) function that
populates both binary resident indexes, either by trusting a sidecar whose
recorded live-slot count matches the table's live binary-row count, or, if
they disagree, by rebuilding from the table and rewriting the sidecar.
Comparing live-count to live-count, rather than the older approach of
comparing total slot counts, avoids a spurious full rebuild after ordinary
deletions leave tombstoned slots behind. `decodePayload(from:)` and
`storedVector(from:)` are the row-decoding functions every read path
shares; both explicitly guard every narrowing integer conversion (for
example, a negative `dim` or an out-of-range `kind` byte) so that a
hand-crafted or corrupted row is rejected with `nil` rather than crashing
the process on a Swift trap.

## Rust Port and Conformance

The `rust/` directory mirrors the Swift implementation file for file:
`vector_store.rs` alongside `engine/brute_force.rs`, `engine/mih.rs`,
`engine/float_brute_force.rs`, `engine/max_sim.rs`, `engine/resident.rs`,
`engine/resident_store.rs`, `engine/key.rs`, `engine/payload.rs`,
`engine/hit.rs`, `engine/metric.rs`, `engine/seam.rs`, plus
`embedding_provider.rs`, `simhash_embedding_provider.rs`, and `error.rs`.
The `.vec` sidecar format, the MIH band-hashing algorithm, and the MaxSim
scoring algorithm are all specified precisely enough in the Swift source
comments (colex enumeration order, the enumeration-budget guard's integer
arithmetic, the little-endian sidecar layout) that both ports are expected
to agree exactly on the binary lane. `rust/tests/` holds integration
suites for bulk ingest, the float lane, int8 rejection, the SimHash
provider, the vector store, and telemetry, exercising the same behaviors
described above. The package's own `MIHIndexTests.swift` gates `MIHIndex`
against `BruteForceIndex` directly within Swift; cross-language
conformance for the binary lane rests on both ports implementing the same
documented algorithm rather than on a single shared fixture file, unlike
LatticeLib's shared JSON fixtures. The float lane is, by design, exempt
from cross-language bit-identity (see `DenseMetric.swift` and
`FloatBruteForceIndex.swift` above); only within-platform reproducibility
and rank correctness are asserted for it, in both languages.

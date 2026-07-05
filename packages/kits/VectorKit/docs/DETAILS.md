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
order. First comes the module surface and errors. Then the embedding
seam. Then the shared engine foundation types. Then the three search
engines. Then the resident-array storage layer. Last comes the
storage-facing types and the `VectorStore` actor that ties everything
together.

## VectorKit.swift

This file provides the module surface. It is a short header comment
naming the kit's public pieces. It defines no types of its own.

The file explains a boundary worth restating here. VectorKit imports
EngramLib internally to use the `Engram` fingerprint type. It does not
re-export that import. A caller that wants to construct an `Engram`
directly must `import EngramLib` itself. This keeps every package's
import list explicit. It matches the convention used across the rest of
the substrate-dependent kits.

## VectorKitError.swift

This file provides `VectorKitError`, the single error type for every
VectorKit operation. Per MOOTx01 convention, errors are concrete named
cases. They are not a generic failure plus a logged message. A caller
can branch on exactly what went wrong.

`embeddingFailed(String)` reports an inference failure inside an
`EmbeddingProvider`. `modelUnavailable(String)` reports a model that is
not loaded on the current device. `storeUnavailable(String)` reports a
failure opening the backing storage. `notFound` is reserved for a future
throwing read path. Today's read functions return `nil` instead.
`invalidPayload` covers a structurally broken `VectorPayload`. This
means a wrong kind, a wrong byte count, or a dimension mismatch. The
payload's own decode functions throw it. So does every search engine's
input validation. `decodingFailure` covers a malformed key or row that
cannot be decoded.

Two cases record product decisions rather than plain bugs.
`int8QuantizationPolicyUndefined` is thrown whenever a caller tries to
write an `.int8`, or quantized, vector. The rules for how to quantize,
and later reverse the quantization, have not been agreed on yet. Writing
one now would lock in behavior nobody has approved. The case documents
that this is deliberate and reversible. Once a policy is ratified, the
guards that throw this error are meant to be removed.
`embedFloatVocabMiss` is thrown by embedding providers whose vocabulary
is fixed in advance. An example is a statistical model trained on a
specific word list. The error fires when none of a query's words are in
that vocabulary. It is distinct from `embeddingFailed`, because the
provider is working correctly here. The input simply has nothing in
common with what the provider knows.

## EmbeddingProvider.swift

This file provides the `EmbeddingProvider` protocol. It is the single
seam between text and vector that every embedding source in VectorKit's
kit graph implements.

The protocol is deliberately narrow. It defines a `modelID` and a
`modelVersion`. These are the tags every stored vector must carry, per
spec I-4. This tagging ensures that vectors from incompatible models
are never compared. The protocol also defines three ways to turn text
into numbers. This narrowness is what
lets VectorKit remain agnostic about where inference happens: CoreML,
ONNX, or a hand-written statistical model. A concrete provider, such as
`FloatSimHashEmbeddingProvider` or a sibling package's MiniLM adapter,
fills in the details.

`embed(_:)` is the primary method. It returns the 256-bit binary Engram
for a piece of text. It throws `embeddingFailed` or `modelUnavailable`
on failure. Every conformer must return the substrate's canonical zero
Engram for an empty string. This is the one input every provider is
guaranteed to agree on. Treating it specially lets empty records from
different providers land on the same identical partition, instead of
colliding by coincidence.

`embedFloat(_:)` returns the dense float vector a provider computed on
its way to the Engram. This is the vector before it was compressed into
256 bits. The protocol's default implementation simply throws
`embeddingFailed`. This means the float lane is opt-in. A provider with
nothing more precise to offer than the fingerprint declines, rather than
fabricate numbers. Providers that do real inference are defined in a
sibling package. Examples are MiniLM, mpnet, and EmbeddingGemma. They
override this method to return the vector they already computed, at no
extra inference cost. Empty input returns an empty array, never a
vector of zeros. A vector of zeros would look like a legitimate, and
misleading, nearest neighbor to every genuinely empty text.

`embedPair(_:)` is a convenience for callers that need both outputs from
one text. Its default implementation calls `embed` and then
`embedFloat`, two inference passes. It swallows a float-lane opt-out
into an empty array, so the default matches historical two-call
behavior. A provider might compute both outputs from a single inference
pass. That provider should override this method, to avoid running its
model twice.

`embedBatch(_:)` embeds several texts. It defaults to a sequential loop.
Providers with genuinely batched inference should override it for
throughput. Output order always matches input order.

## FloatSimHashEmbeddingProvider.swift

This file provides `FloatSimHashEmbeddingProvider`, the one concrete
`EmbeddingProvider` VectorKit ships. It is a mirror of the Rust
`vectorkit::FloatSimHashEmbeddingProvider`.

The type is a thin wrapper. It holds a `modelID`, a `modelVersion`, a
`projectionSeed`, and an injected inference closure. That closure has
the shape `(String) async throws -> [Float]`, and the host that owns the
actual model supplies it. VectorKit itself never loads a model or owns a
tokenizer. Concrete text providers that do live in a different package,
CorpusKitProviders. Examples are MiniLM, mpnet, and EmbeddingGemma. They
conform to this same protocol, using this type as their low-level
building block.

`embed(_:)` short-circuits to the canonical zero Engram for empty input.
It does this before ever calling the inference closure. This guarantees
the empty-string contract holds, even if the closure itself would have
produced something non-zero. Otherwise it calls the inference closure.
It then passes the result through
`SubstrateML.FloatSimHash.project(vector:seed:)`. This is a shared,
conformance-gated substrate primitive. It projects an arbitrary-length
float vector down to a 256-bit fingerprint using a technique called
SimHash. The `projectionSeed` matters here. Different seeds turn the
same float vector into different fingerprints. This is what keeps two
different models' outputs from accidentally landing in the same
fingerprint space and being wrongly compared. This is spec I-4's rule,
enforced at the projection layer as well as at storage.

`embedFloat(_:)` returns exactly the float vector the inference closure
produced. This is the same vector `embed(_:)` feeds into the projection.
So a caller using both lanes never pays for two inference passes.

## Engine/VectorRecordKey.swift

This file provides `VectorRecordKey`, the identifier every stored
vector carries inside the search engines. Note that this file lives
under `Engine/`, VectorKit's internal folder for the search-engine
machinery shared by every index implementation.

A key used to be a two-part pair of item and model. That design assumed
one vector per item per model. The assumption breaks for models like
ColBERT that produce one small vector per word instead of one vector
per document. So the key grew a third field. `itemID` names the owning
record, a drawer or a text chunk, as a UUID string. `vectorIndex` is the
position of this vector within its item's sequence. It is `0` for the
ordinary single-vector case, and `0` through `N-1` for a multi-vector
item. `modelID` and `modelVersion` complete the tuple. Vectors from
different model versions are never comparable, per spec I-4.

`VectorRecordKey` is `Comparable`. It is ordered lexicographically by
`(itemID, vectorIndex, modelID, modelVersion)`. This order is not
incidental. It is the tie-break every search result uses when two
matches land at the same distance. It is also the order the on-disk
resident array is built in. So both the search output and the storage
layout agree on what sorted means.

## Engine/VectorPayload.swift

This file provides `VectorPayload`, the one envelope every vector's raw
bytes travel in. It also provides `VectorKind` and `VectorPayloadInput`.
`VectorKind` tags which numeric family the bytes represent.
`VectorPayloadInput` bundles a payload with its storage metadata, for
bulk writes.

`VectorKind` has three cases. `.binary`, raw value `0`, is exactly the
32-byte Engram wire form. `.float32`, raw value `1`, is `dim × 4` bytes
of IEEE-754 numbers. `.int8`, raw value `2`, is reserved for a future
quantized representation. The case exists so that a future ratified
quantization policy does not require a new wire format. `VectorStore`
currently rejects every `.int8` write. See
`VectorKitError.int8QuantizationPolicyUndefined` above. The raw values
are stored on disk and must never be reordered.

`VectorPayload.init(engram:)` builds a binary payload directly from an
Engram's wire bytes. There is no conversion and no loss, so every
existing binary test still holds. `init(floats:)` serializes a `[Float]`
to little-endian IEEE-754 bytes explicitly, byte by byte, rather than
relying on the platform's native byte order. This is a deliberate
choice, and not an oversight. It is what lets the on-disk `.vec` sidecar
be read back identically on an Apple device and on a Linux server. The
two platforms may not share the same native byte order convention.
`asEngram()` and
`asFloats()` reverse these conversions. They throw `invalidPayload` when
the payload's kind or byte count does not match what was asked for.

## Engine/DenseHit.swift

This file provides `DenseHit`, the one result shape every search engine
returns. It also provides `LaneTag`. This is an enum naming which
retrieval technique produced a given score. Fusion and multi-technique
retrieval code outside this package uses it.

A `DenseHit` carries the matched `key`, a `rawDistance`, and the
`metric` that produced it. The tricky design point is that `rawDistance`
is a single `Int32` field shared by two very different kinds of number.
It can hold an integer Hamming distance for the binary lane. It can also
hold the bit pattern of a `Float` distance for the float lane. Two
computed properties translate it back. `hammingDistance` simply casts
it to `Int`. This is safe because Hamming distances are always in the
range 0 to 256. `floatDistance` reconstructs the `Float` from its stored
bit pattern. Packing both families into one field has one payoff. Code
elsewhere in the kit graph can handle a `[DenseHit]`. It need not care
which lane produced it. This design avoids giving each lane its own
result type.

The file's header calls out an additive-only rule. Any future field
added here must have a default value. This lets existing callers who
build a `DenseHit` with the current initializer keep compiling. This
matters because both a Swift and a Rust version of this type exist. The
two must stay in lockstep.

## Engine/DenseMetric.swift

This file provides the metric vocabulary for the whole engine seam. It
defines `BinaryMetric`, with cases `.hamming` and `.jaccard`. It defines
`FloatMetric`, with cases `.cosine`, `.l2`, and `.dot`. It defines
`DenseMetric`, the umbrella enum wrapping either family. This means
`DenseIndex.search` needs only one metric parameter, regardless of which
lane it routes to.

The file's real content is its documentation of a determinism boundary.
This boundary recurs throughout this package. `.binary(.hamming)` is
four-way bit-identical, because it is pure integer arithmetic computed
by a shared, conformance-gated kernel. `.binary(.jaccard)` is
bit-identical through its two integer counts. It ends with one final
IEEE-754 division, which is itself guaranteed identical, because
IEEE-754 mandates exact rounding for basic operations. Every `.float(_)`
metric is reproducible only within one build and platform. It is never
guaranteed identical between Swift and Rust. This is stated as a
documented property of floating-point math. It is not a defect to be
fixed. It is a warning aimed at a future reviewer. That reviewer might
otherwise try to force float parity that the underlying arithmetic
cannot honestly provide.

## Engine/DenseIndex.swift

This file provides the `DenseIndex` protocol. This is the single seam
that lets `VectorStore` treat three very different search engines,
`BruteForceIndex`, `MIHIndex`, and `FloatBruteForceIndex`, as
interchangeable. The file also provides three supporting types:
`IndexKind`, `SearchDirection`, and `MetadataFilter`. `IndexKind` is a
tag naming which implementation is behind a given index. Tests use it
to pick the brute-force oracle deliberately.

`SearchDirection` has two cases. `.nearest` is most similar first, and
is the default. `.farthest` is most dissimilar first. Farthest search
supports a query for things unlike this one. It is not a trick of
negating a nearest-neighbor list. The farthest items are not among the
nearest top-k at all. So the index has to scan and sort toward the
opposite end. It uses the exact same distance calculation, only the
opposite sort direction.

`MetadataFilter` restricts a search to one `modelID` and, optionally,
one `modelVersion`. Its `accepts(_:)` method is the single predicate
every engine calls per candidate. A `nil` field acts as a wildcard.

The protocol itself declares four operations: `build(from:)`,
`search(probe:metric:k:filter:)`, `add(key:vector:)`, and
`remove(key:)`. It documents the contract every conformer must honor.
Results come back sorted by distance ascending. Ties break by the
matched key ascending. `BruteForceIndex` is the correctness oracle every
other binary engine is measured against.

## Engine/BruteForceIndex.swift

This file provides `BruteForceIndex`. This is the exact linear-scan
search engine for binary vectors, using Hamming distance. It is also
the conformance oracle every other binary engine is checked against.
Today that is just `MIHIndex`.

The file is built around one hard rule, restated three times in its
comments. It performs zero Hamming arithmetic itself. Every distance is
computed by `EngramLib.distances`. This routes to a shared kernel,
selected once per process. The kernel uses NEON on Apple silicon
hardware, and a scalar fallback elsewhere. It is checked for identical
output across four build configurations. Reimplementing a bitwise
XOR-and-count here, even a correct one, would bypass that check. It
would risk silent divergence between platforms. This is the file's
version of spec I-7.

`search(probe:metric:k:filter:)` validates the probe, which must be
exactly 32 bytes of `.binary` kind, and the metric. Only
`.binary(.hamming)` is supported, and other requests throw
`invalidPayload`. It narrows the scan to one model's slot range when a
filter is present. This is an `O(log m)` lookup into a sorted partition
index. The lookup avoids a full-array walk. It collects the live,
un-tombstoned
candidates in that range and hands their Engrams to
`EngramLib.distances` in one batch call. It deliberately avoids
`EngramLib.findNearest`, which applies a different, insertion-order
tie-break. This file sorts the returned distances itself, by distance
ascending and then key ascending. The engine's own contract requires the
full `VectorRecordKey` as the tie-break, not just the array's insertion
position. Otherwise two records under the same item but different model
or vector index could be returned inconsistently.

`add(key:vector:)` implements upsert by tombstoning any existing slot
with the same key before appending the new bytes. It then rebuilds the
sorted model-partition index from the updated key list. `remove(key:)`
tombstones every matching slot without touching the underlying storage
bytes. Actual space reclamation is `ResidentArrayStore`'s job, not this
type's. `currentSnapshot()` returns a value-type copy of the live array.
This lets callers outside the actor, chiefly `VectorStore` when it needs
to scan for tombstoning, read it safely.

## Engine/FloatBruteForceIndex.swift

This file provides `FloatBruteForceIndex`. This is the linear-scan
search engine for the float32 lane. It supports three float metrics:
cosine, Euclidean, and dot-product distance. The code calls Euclidean
distance `l2`. Unlike the binary lane, this is both the correctness
reference and the production search path. There is no separate
accelerated float engine in this package.

The file opens with an emphatic warning, repeated in this document
because it protects against a plausible but wrong fix. Float arithmetic
here is reproducible on one build and platform. It is not, and cannot be
made to be, bit-identical between Swift and Rust or across different
hardware. This is a documented property of IEEE-754 arithmetic, not an
oversight. A reviewer must not try to force it to match the binary
lane's four-way guarantee.

`build(from:)` simply stores a reference to the supplied array. There is
no secondary structure to construct, so building is `O(1)`. The real
cost is whatever the caller paid to assemble the array.
`search(probe:metric:k:filter:)` validates three things. The probe must
be `.float32`. The requested metric must be a float metric. The probe's
byte count must match both its own declared dimension and the array's
fixed stride. A mismatch would otherwise read past the end of a slot, so
it throws instead. It then scans every live, filter-passing slot, computing
one of three distances per candidate. Cosine distance treats a zero
vector as maximally distant rather than crashing on a divide-by-zero.
`l2` is the plain Euclidean formula. `dot` is negated so that smaller is
nearer holds for every metric uniformly. The scan sorts ascending by
distance, then by key. `searchFarthest(probe:metric:k:filter:)` reuses
the identical scan and identical distance math. It changes only the sort
direction to descending. This is what makes finding dissimilar items a
real bottom-of-the-list scan, rather than a negated top-of-the-list one.

`add(key:vector:)` establishes the array's dimension from the first
vector added. It rejects any later vector of a different byte count. A
mismatched stride would silently corrupt the flat storage buffer.
`remove(key:)` tombstones the matching slot. Actual compaction
happens the next time `build(from:)` runs with a freshly assembled
array.

## Engine/MaxSimScorer.swift

This file provides `MaxSimScorer` and `MaxSimHit`. Together these form
the exhaustive, or Exact-A, implementation of ColBERT-style
late-interaction scoring over binary token fingerprints.

Some embedding techniques represent one document as many small vectors.
One vector exists per word or token, not one vector for the whole
document. Comparing two such documents means asking one question, for
every word in the query. Which word in this document is most like it.
The scorer then adds up
those best matches. That sum, `Σ (256 − minimum Hamming distance)` over
every query token, is the MaxSim score this file computes. It examines
every query token against every document token for every candidate
document. So it never skips a candidate. This exhaustiveness is
precisely what makes it the correctness reference for any faster, pruned
variant built later. The file's header explicitly reserves the
accelerated two-stage variant as out of scope here.

`score(queryTokens:documents:k:)` iterates the supplied documents in
ascending itemID order. It sorts the dictionary's keys explicitly. A
Swift dictionary's own iteration order is not guaranteed. Relying on it
would make results non-reproducible. It computes each document's MaxSim
score. It sorts the results by score descending, then itemID ascending.
It truncates to `k` only after the full sort, never before. So a
document that would have scored well is never cut for appearing late in
an unsorted pass. Every Hamming distance again goes through
`EngramLib.Session.distances`. This session is constructed once per
`MaxSimScorer` and reused for the whole call. The one-time cost of
picking the fastest available kernel is paid once, rather than per
comparison.

## Engine/ResidentVectorArray.swift

This file provides `ResidentVectorArray`, the packed in-memory data
shape every search engine reads from. It also provides
`ModelPartitionEntry`, one entry in its per-model index. This is the
shared contract underneath `BruteForceIndex`, `MIHIndex`, and
`FloatBruteForceIndex`. All three read the identical layout. This is
what lets `VectorStore` build the array once and hand it to whichever
engine is currently active.

The design reason is stated directly in the file's header comment.
Measurement on the pre-existing code path found a bottleneck. Fetching
and decoding rows from the database consumed eighty-seven percent of a
search's latency. The actual distance kernel took under one percent. A
fixed-stride, contiguous byte array removes the fetch-and-decode cost
from every query after the first. The whole array is loaded once. It is
then scanned as a flat block of memory with no per-row allocation.

The type stores `kind` and `stride`, the bytes per vector slot: thirty-two
for binary, `dim × 4` for float32. It stores `count`, including
tombstoned slots, and `storage`, the packed bytes themselves. On Apple
platforms, `storage` may be memory-mapped read-only from a sidecar file.
It also stores a `keys` array parallel to `storage`, a sorted
`modelPartitions` index, and a `tombstones` bitmap. `liveCount` walks the
tombstone bitmap to compute how many slots are still valid.
`partitionRange(for:)` binary-searches the sorted partitions to find one
model's slot range in `O(log m)`. `isTombstoned(_:)` and
`vectorBytes(at:)` are the two per-slot accessors every engine's scan
loop calls.

## Engine/ResidentArrayStore.swift

This file provides `ResidentArrayStore`, the actor that owns the
optional on-disk `.vec` sidecar file. This file is a packed binary cache
of a `ResidentVectorArray`. It lets a reopened store skip rebuilding the
array from every database row.

The file documents its own on-disk format in full. A fixed header holds
magic bytes, format version, vector kind, stride, count, a live-slot
count, and the tombstone bitmap. This header is followed by the packed
vector bytes, then variable-length key records, then the
model-partition index. Every multi-byte integer is explicitly
little-endian. So the same file reads identically on an Apple device
and on a Linux server. `writeSidecar` writes to a temporary file and
atomically renames it into place. So a crash mid-write never leaves a
half-written sidecar behind. `readSidecar` memory-maps the file where
the platform supports it. This is a load-time optimization, not a
difference in the bytes returned. `parseSidecar` does the actual
decoding. It
checks every length field against the remaining buffer size before
trusting it. So a corrupted or hand-edited file is rejected with a
`decodingFailure`, rather than crashing the process.

The file is explicit about one policy. The `vectors` database table is
always the durable source of truth. This sidecar is a regenerable cache,
and never a second copy of record. `load()` reads the sidecar if
present. If it is missing or fails to parse, the store simply starts
empty. It then waits for `VectorStore` to rebuild it from the table.

Three write paths exist. A single write policy could not serve both a
low-latency single insert and a large bulk import well.
`append(key:bytes:)` is the eager path. It writes the sidecar
immediately after every single addition. `appendDeferred(key:bytes:)` is
the write-behind path a single insert uses in production. It updates the
in-memory array and marks the store dirty, without touching disk. It
trusts the caller to call `flush()` at a natural pause. This is safe
because the database row was already written durably before this call.
Losing an unflushed sidecar only costs a rebuild on the next open, never
data. `appendBatch(records:)` is the bulk-import path. It extends
storage, keys, and the tombstone bitmap for the whole batch in one pass.
It writes the sidecar exactly once, so importing a thousand vectors
costs one disk write instead of a thousand.

`compact()` rewrites the sidecar, keeping only live, non-tombstoned,
records. It sorts them by key for a deterministic, reproducible layout.
This is triggered automatically whenever the tombstone ratio exceeds
`compactionThreshold`, twenty-five percent by default, after any eager
write.

## StoredVector.swift

This file provides `StoredVector`. This is the public, fully decoded row
shape that `VectorStore.vectors(forItemID:)` returns. Callers want this
convenient binary form, rather than the raw typed payload.

Its fields mirror the `vectors` table's columns directly. The fields
are `id`, `itemID`, `vectorIndex`, `modelID`, `modelVersion`, `engram`,
and `filedAt`. The `id` is a stable value assigned on insert. The
`itemID` is the owning record. The `vectorIndex` is the position within
a multi-vector item. The `modelID` and `modelVersion` are spec I-4's
tags. The `engram` is the decoded fingerprint. The `filedAt` field is
the time the row was written. This last field is round-tripped through
the database's text-based ISO 8601 timestamp column. That column loses
sub-millisecond precision. The file notes this explicitly, so a caller
comparing timestamps at fine granularity is not surprised.
`StoredVector.engram` is non-nil only for binary rows. A float or int8
row must be read through `VectorStore.getPayload` instead. This
convenience type only round-trips the binary case.

## VectorMatch.swift

This file provides `VectorMatch`, the public search-result shape
`VectorStore.findNearest` and its float-lane counterparts return.

It carries `itemID`, `distance`, and `modelID`. The `itemID` is the
matched record. The `modelID` lets a caller confirm which model
actually produced the match. `distance` is a Hamming distance for the
binary lane, an integer from zero to two hundred fifty-six. For the
float lane it is a scaled, quantized cosine distance, explained in
`VectorStore`'s float-search functions.
`VectorMatch` conforms to `Comparable`. It is ordered by `distance`
ascending, with ties broken by `itemID` ascending. This is the same
universal tie-break rule used throughout the engine layer. So a sorted
array of matches reads nearest to farthest, from front to back. A
caller need not know the sort convention.

## VectorStore.swift

This file provides `VectorStore`, the actor every consumer of VectorKit
actually talks to. It is the largest file in the package. Every other
piece is wired together here into one consistent API. This includes the
durable `vectors` table, the resident arrays, the three search engines,
and telemetry.

### Storage and schema

`VectorStore` wraps a PersistenceKit `Storage` backend. Supported
backends include SQLite and an in-memory test backend. A PostgreSQL
backend may be added in the future. The kit never sees which backend is
chosen. That decision belongs to the application. `schemaDeclaration` is
the static schema description passed
to `storage.open(schema:)` before the store is used. It describes one
`vectors` table, version three. Its `UNIQUE(item_id, vector_index,
model_id)` constraint is exactly `VectorRecordKey` minus `modelVersion`.
This constraint makes an upsert on a changed model version a true
replacement of the old row. It is not treated as a duplicate.

### Two hot-path structures kept in sync

Every write updates three things together: the durable `vectors` table
row, the in-memory resident array, and the on-disk cache. The resident
array updates through `bruteForceIndex` and `mihIndex`. The on-disk
cache updates only when a sidecar was configured at construction. Both
`bruteForceIndex` and `mihIndex` are always kept current. Only one is
ever the active `hotIndex`.
`_selectIndex()` compares `liveBinaryCount` against `mihThreshold`,
fifty thousand by default, after every write that changes the count. It
swaps `hotIndex` between `bruteForceIndex` and `mihIndex` with a plain
reference assignment. No rebuild is needed on promotion or demotion,
because both indexes were already current.
`init(storage:sidecarURL:mihThreshold:mihBandCount:deferredPendingLimit:)`
allocates both index actors up front. So this swap never needs to
construct anything at query time.

### Write path

`addVector(itemID:engram:modelID:modelVersion:filedAt:)` is a
convenience wrapper for the common single binary vector case. It builds
a `VectorPayload` and delegates to `addPayload`.
`addPayload(itemID:vectorIndex:payload:modelID:modelVersion:filedAt:)`
is the general write. It rejects `.int8` payloads immediately. See
`VectorKitError` above for that case. It writes the row through an
upsert keyed on the table's unique constraint. It then mirrors the
write into the matching resident array, but only for `.binary` and
`.float32` kinds. For a binary write, `addPayload` first finds any
existing slot at the same logical position. This position is defined by
`itemID`, `vectorIndex`, and `modelID`. The match ignores `modelVersion`.
This lets a version change be treated as a true replacement, rather
than leaving a stale duplicate slot behind. This matching is
deliberately looser than full key equality, specifically to catch that
case. It then tombstones the stale slot, if any, and appends the new
one in both
`bruteForceIndex` and `mihIndex`. It updates `liveBinaryCount` only when
the write was genuinely new, and calls `_selectIndex()`. Every write
emits a `vectorkit.index.insert_latency_ms` telemetry metric through
IntellectusLib. This metric is a short-circuited no-op unless monitoring
has been explicitly turned on. So the cost on the default path is one
boolean check.

`addPayloads(_:)` is the bulk-import counterpart. Its whole reason for
existing is complexity. Importing N vectors one at a time through
`addPayload` costs N sidecar rewrites and, without care, N index
rebuilds. This function upserts every row to the table, unavoidable
because the table is the durable source. It then rebuilds both binary
indexes exactly once, from the final merged array rather than once per
row. This cuts the amortized cost from `O(N²)` bytes written to `O(N)`.
`beginDeferredIndex()` and `publishResidentIndex()` extend this further
for very large or multi-call bulk imports. While a deferred window is
open, `addPayloads` appends to storage but skips the index rebuild
entirely. It seeds an in-memory tracked set of live keys, so replacement
detection stays cheap across the whole window. `publishResidentIndex()`
performs the single rebuild the whole burst needed, once, when the
caller signals the burst is finished. `deferredPendingRecords` is
capped at `deferredPendingLimit`, fifty thousand by default. This cap
guards the memory-only variant of this path against unbounded growth.
This applies if a caller holds the window open indefinitely. Crossing
the cap
triggers `_flushDeferredPending()`, an internal intermediate merge that
keeps the deferred window open for the caller, while bounding peak
memory.

### Search path

`findNearest(probe:modelID:limit:)` is the binary-lane search. It
lazily builds the resident array on first use. It builds this from the
sidecar if one is current, or from the table if not. It then delegates
entirely to `hotIndex.search`, converting the returned `[DenseHit]`
into `[VectorMatch]` without re-sorting. The engine has already applied
the required order of distance ascending, then itemID ascending.
`findNearestFloat(probe:modelID:limit:)` is the float-lane equivalent.
It lazily builds a `FloatBruteForceIndex` per model. The map's presence
is the already-built flag. There is no separate boolean. It then
searches by cosine distance. It quantizes the resulting `Float` distance
to an integer, by multiplying by ten thousand and rounding. So
results from different languages' fixtures can be compared exactly,
rather than approximately. `findFarthestFloat(probe:modelID:limit:)` is
identical, except it calls the engine's farthest-ranking search, for an
anti-similarity query for things unlike this one.
`findByKeyword(_:limit:)` is a plain substring match on `item_id`. It is
explicitly documented as a quick pre-filter for hybrid retrieval, and
not a real keyword search. Full keyword scoring is a different
package's responsibility.

### Delete path

`deleteVector(itemID:modelID:)` deletes the row at `vectorIndex` zero
and tombstones every matching resident slot.
`deleteAllVectors(itemID:modelID:)` deletes every vector index for an
item and model. This is used when a multi-vector item, such as all of a
ColBERT item's token vectors, needs complete removal. Both functions
first flush any in-flight deferred-index burst, so a delete never races
an unpublished bulk import. `destroyAllVectors()` wipes the entire
store: every row, both resident indexes, the sidecar, and the per-model
float indexes. This runs as part of a coordinated estate teardown. An
estate is one user's complete memory store in MOOTx01.

### Coherence helpers

`_ensureIndexBuilt()` is the one-time, per process, function that
populates both binary resident indexes. It trusts a sidecar when its
recorded live-slot count matches the table's live binary-row count. If
the counts disagree, it rebuilds from the table and rewrites the
sidecar. Comparing live count to live count avoids a spurious full
rebuild after ordinary deletions leave tombstoned slots behind. The
older approach compared total slot counts instead.
`decodePayload(from:)` and `storedVector(from:)` are the row-decoding
functions every read path shares. Both explicitly guard every narrowing
integer conversion, for example a negative `dim` or an out-of-range
`kind` byte. So a hand-crafted or corrupted row is rejected with `nil`,
rather than crashing the process on a Swift trap.

## Rust Port and Conformance

The `rust/` directory mirrors the Swift implementation file for file.
It provides `vector_store.rs`, alongside `engine/brute_force.rs`,
`engine/mih.rs`, `engine/float_brute_force.rs`, `engine/max_sim.rs`,
`engine/resident.rs`, `engine/resident_store.rs`, `engine/key.rs`,
`engine/payload.rs`, `engine/hit.rs`, `engine/metric.rs`,
`engine/seam.rs`, plus `embedding_provider.rs`,
`simhash_embedding_provider.rs`, and `error.rs`. Three things are
specified precisely enough in the Swift source comments: the `.vec`
sidecar format, the MIH band-hashing algorithm, and the MaxSim scoring
algorithm. This includes colex enumeration order, the enumeration-budget
guard's integer arithmetic, and the little-endian sidecar layout. Both
ports are expected to agree exactly on the binary lane. `rust/tests/`
holds integration suites for bulk ingest, the float lane, int8
rejection, the SimHash provider, the vector store, and telemetry. These
suites exercise the same behaviors described above. The package's own
`MIHIndexTests.swift` gates `MIHIndex` against `BruteForceIndex`
directly within Swift. Cross-language conformance for the binary lane
rests on both ports implementing the same documented algorithm. It does
not rest on a single shared fixture file. This differs from
LatticeLib's shared JSON fixtures. The float lane is, by design, exempt
from cross-language
bit-identity. See `DenseMetric.swift` and `FloatBruteForceIndex.swift`
above. Only within-platform reproducibility and rank correctness are
asserted for it, in both languages.

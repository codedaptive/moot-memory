---
doc: DETAILS
package: CorpusKit
repo: moot-memory
authored_commit: ecbe2bc361c83a1e8bc636767d33d0c678f88bd7
authored_date: 2026-07-04
sources:
  - path: Sources/CorpusKit/BasisStore.swift
    blob: 48850906faa7c2fe4aac2859a1c4e892cff32cab
  - path: Sources/CorpusKit/BM25Index.swift
    blob: 06fb90cd40e81f013e01a8a6c4c6f94e71bf33f3
  - path: Sources/CorpusKit/BundleStore.swift
    blob: 419b1c0609597cdd68bf623ed37bd40a0171597b
  - path: Sources/CorpusKit/Chunk.swift
    blob: d5a1be1bb08858f5f7bd59bb141a8a0ba6f1dfbe
  - path: Sources/CorpusKit/Chunker.swift
    blob: a2718e06d1715f539ff633e7037c70e10ecb7a2d
  - path: Sources/CorpusKit/CorpusIngestQueue.swift
    blob: 2c32133701ce728bc017d4ddad51b052cae990db
  - path: Sources/CorpusKit/CorpusKit.swift
    blob: 4518f15fdb798c3a203c4a9db949f4d6172f540d
  - path: Sources/CorpusKit/CorpusKitError.swift
    blob: 68ac8d0a248bc9c2dd1885b0bc531ac4ed9cb91d
  - path: Sources/CorpusKit/CorpusProviderCountsStore.swift
    blob: c92160041765cc8546501c5ac4d8a2b769656e93
  - path: Sources/CorpusKit/Engine/BM25Weighting.swift
    blob: 622f45870ab1118d7590cce1b379a063619b4714
  - path: Sources/CorpusKit/Engine/Fusion.swift
    blob: d128ed9bc206612fc7c2e849a2e77d03d8a6cafa
  - path: Sources/CorpusKit/Engine/InvertedIndex.swift
    blob: 1273adcb3794b1997c93488182fc5ed95b21f9ec
  - path: Sources/CorpusKit/Engine/InvertedIndexStore.swift
    blob: 242baf05e5c846c719c403599a9a407bba646f5b
  - path: Sources/CorpusKit/Engine/SparseTypes.swift
    blob: 54654e2c49b09d31f60c06503216a2b281939f87
  - path: Sources/CorpusKit/HybridRecall.swift
    blob: 21d9fb3415b699c6469a1e80d5e84da1e43981ca
  - path: Sources/CorpusKit/RemovedSourceStore.swift
    blob: 138a2a094ff369ee69eafa04eeba77206a58e5f4
  - path: Sources/CorpusKit/SyncManifest.swift
    blob: 39af591d1fbf1f213c93eb143c213258e41c6c4b
  - path: Sources/CorpusKit/Tokenizer.swift
    blob: 603028510f91b3c6d75cdda1cb0a1db1c59eee28
  - path: Sources/CorpusKit/TrainableEmbeddingBasis.swift
    blob: 4722def84980b3a8987adb090a1a702ac789f8ab
  - path: Sources/CorpusKitProviders/BasisCodec.swift
    blob: d107e1efd6341648fd8f717c7956a15b98c1b29f
  - path: Sources/CorpusKitProviders/DefaultEnsemble.swift
    blob: c58168f0991cc4e4c3ec490e2272bdc1a5a17be1
  - path: Sources/CorpusKitProviders/DeterministicTokenizer.swift
    blob: 0586b3a4ae93dc0b58ef8f62c0d104a81dcfefe3
  - path: Sources/CorpusKitProviders/EmbeddingGemmaProvider.swift
    blob: 593cd5952aad04fe0390e133e68cc07cad983d86
  - path: Sources/CorpusKitProviders/FdcProvider.swift
    blob: 96f3ffaf64c7b17e2f617f49ce06cd649e70a027
  - path: Sources/CorpusKitProviders/LsaProvider.swift
    blob: 3870f0d24659cb27ba13dc2cba9f94debb6b5c07
  - path: Sources/CorpusKitProviders/MiniLMTextProvider.swift
    blob: 35c739a37e9ef1a92098458b48c5f5a06f11050f
  - path: Sources/CorpusKitProviders/MPNetTextProvider.swift
    blob: 6f13fc6dcd4733460cad366e78273f542c65a844
  - path: Sources/CorpusKitProviders/NLContextualEmbeddingProvider.swift
    blob: 17d1acc363bab9c5d0bb807041e0b4c3d66fa0ee
  - path: Sources/CorpusKitProviders/NLEmbeddingProvider.swift
    blob: 2d714a051a1b6bd4dbde1ecd0e182a81f94b7008
  - path: Sources/CorpusKitProviders/NmfProvider.swift
    blob: 43f0e339a42426d68788a03e8a216890c73eb05a
  - path: Sources/CorpusKitProviders/PpmiProvider.swift
    blob: 282e2185cb7e3d066979ea23b74e096ee337545b
  - path: Sources/CorpusKitProviders/RandomIndexingProvider.swift
    blob: 552b55b1be93fb57b9e7daf123ecf3a73df7abef
  - path: Sources/CorpusKitProviders/ReducedVocab.swift
    blob: fb50a8566f9ef3b2a9c650102274b20894d6542d
  - path: Sources/CorpusKitProviders/TermDocumentCounts.swift
    blob: e72231cf3e50799b6bbeac6a165b80410dc40317
---

# CorpusKit Details

This document walks through every source file in the package. Read
`OVERVIEW.md` first for the big picture. Files appear in pipeline order:
the content types, the stores, the keyword engine, fusion and recall, the
trainable-basis machinery, the `Corpus` facade and its ingest queue, and
finally the providers target.

## Chunk.swift

This file provides `Chunk`, the fundamental retrievable unit, and
`ScoredChunk`, the result type recall returns.

A chunk's identity is content-addressed: its UUID is computed, not
assigned. `deriveID(sourceID:startOffset:text:)` builds an RFC 4122
version 5 UUID from a SHA-1 hash over the source identifier, the start
offset, and the exact text. The three fields are joined with the Unicode
unit separator so that no two different inputs can collide by ambiguous
concatenation. Content addressing is what makes the rest of the kit work:
re-ingesting the same source reproduces the same identities, so a repeat
insert is a harmless no-op, and two federated devices writing the same
chunk converge on one row instead of conflicting. The fixed namespace
bytes are permanent. Changing them would re-key every chunk in every
estate and break the join to existing vector rows, because the chunk id
doubles as the VectorKit item identifier.

Two initializers exist. The content-addressed one computes the id and is
the path normal ingestion uses. The explicit-id one reconstructs a chunk
whose id is already known, such as a decoded storage row; the caller must
ensure the id truly matches the content. The `hlc` field is a hybrid
logical clock stamp — a timestamp that also encodes causal order — and is
always caller-supplied. `ScoredChunk` pairs a chunk with its fused
`score` plus the raw `vectorScore` and `keywordScore`, kept separate so
callers can diagnose which lane produced a hit.

## Chunker.swift

This file provides `Chunker`, which splits raw source text into ordered
chunks with sentence-aware boundaries.

Chunks should not cut a sentence in half, because a half sentence embeds
poorly and reads worse. The chunker therefore segments the text into
sentences first, delegating to `EideticLib.sentences(_:)` so segmentation
logic lives in one shared place. It then fills a buffer greedily: sentences
accumulate until adding the next one would pass the target size, the buffer
flushes into a chunk, and the tail of the flushed chunk carries over as
overlap into the next buffer. Overlap means a match near a boundary still
brings its surrounding context along. Offsets into the original text are
tracked exactly, because the offset is part of each chunk's
content-addressed identity.

`ChunkerConfiguration` holds `targetChars` (default 800), `overlapChars`
(default 100), and `respectSentences` (default true). Its initializer
clamps nonsense values — overlap can never reach the target, which would
loop forever. `Chunker.chunk(text:sourceID:configuration:hlcGenerator:)`
is the single entry point. The HLC generator is passed `inout` because the
chunker is the sole authority for chunk order within one call: it stamps
each chunk in emission order. Changing the default sizes changes chunk
boundaries and therefore chunk identities for re-ingested content, so the
defaults are pinned to the substrate reference.

## Tokenizer.swift

This file provides the `Tokenizer` protocol and the single canonical
keyword tokenizer.

A tokenizer serves two different masters. An embedding model needs its own
vocabulary of integer token ids, so the protocol requires
`tokenize(_:) -> [Int32]` plus identity fields (`vocabID`, `maxTokens`,
pad and unknown ids). Keyword search needs plain words, so the protocol
also requires `keywordTokens(_:) -> [String]`. The protocol extension
supplies a default for the second, delegating to the free function
`defaultKeywordTokens(_:)`: lowercase the text, then keep runs of
alphabetic or ASCII-digit characters and split on everything else.

There is exactly one definition of keyword tokenization in the module for
a reason. BM25 and every distributional embedding signal must agree on
what a "term" is, or the keyword lane and the semantic lane would score
different vocabularies and hybrid recall would quietly degrade. The
function is also parity-critical: the Rust port implements the same rules,
and committed conformance vectors on both legs break if it changes. A
provider that overrides `keywordTokens` breaks this guarantee by
convention, not by compiler error.

## CorpusKitError.swift

This file provides `CorpusKitError`, the module's single error enum.

Seven flat cases cover the failure classes: `encodingFailure`,
`decodingFailure`, `tokenizerUnavailable`, `modelUnavailable`,
`embeddingFailed`, `storeUnavailable`, and `notTrainable`. Each carries a
plain message string, because callers mostly log or surface the message
rather than branch on structured data. `notTrainable` exists for
`EmbeddingModel.reconstruct(from:)`: providers without a trained basis
(the deterministic provider, the named neural models, the stateless FDC
provider) surface this error instead of silently substituting a wrong
provider. The enum is `Equatable` on its message strings, so tests that
compare errors must construct the exact message.

## BundleStore.swift

This file provides `BundleStore`, the persistence layer for chunks — the
content half of every content-plus-vector bundle. The vector half lives in
VectorKit, joined by the chunk's UUID string.

`BundleStore` is an actor wrapping `any Storage` from PersistenceKit, so
the application picks the backend (SQLite or in-memory) and the kit does
not. It owns schema version 3: a `chunks` table of ten columns plus a
`corpus_metadata` table, with indices on `source_id` and `hlc`. Inserts do
not go straight to the row store. They pass through a hashing decorator
that computes a content hash with `MerkleHash.leaf` on every write and
emits dirty-chain events. A Merkle hash chain lets the estate prove its
content has not drifted: each chunk hashes, each source's chunks combine
into a per-corpus root, and all corpus roots combine into one global root.
Because the hashing callback is synchronous, a small lock-guarded
`ParentChainCache` pre-stages each chunk's parent identifiers before the
insert. The corpus and root identifiers derive from fixed namespace
strings via SHA-256, so both language legs compute identical chains.

`insert(_:)` is idempotent by design. It attempts a plain insert per chunk
and treats a duplicate-key error as the documented no-op path — first
write wins, which is sound only because identities are content-addressed.
It returns only the chunks that were actually new, in input order, so
callers maintaining derived state never double-count a re-ingested chunk.
Read paths (`get`, `getMany`, `chunksForSource`, `allChunks`,
`allSourceIDs`, `count`, `chunkSourcePairs`) are thin query wrappers;
`chunkSourcePairs()` deliberately omits chunk bodies so opening a corpus
stays cheap. `scrubText(sourceID:)` is the hard-delete seam: it zeroes the
`text` column through a direct update, which is why the schema declares
the table `appendOnly: false` even though the API treats chunks as
immutable. Immutability here is a convention enforced by the surface, not
a database trigger; the sync layer's separate `appendOnly` conflict policy
should not be confused with this flag.

One decoding rule deserves emphasis. The SQLite backend round-trips UUIDs
as text and HLCs as packed integers, while the in-memory backend preserves
the semantic typed values. `decodeChunk` and its helpers accept both
forms. A past semantic-only decoder silently dropped every persisted chunk
on reopen, and in-memory tests never caught it. Any new decode path must
handle both forms.

## RemovedSourceStore.swift

This file provides `RemovedSourceStore`, the tombstone table that makes
source removal stick.

Chunk rows are never deleted, so "removing" a source can only delete its
vectors and keyword postings. Without a durable marker, any rebuild that
replays `allChunks()` — an explicit reindex, or the autonomic governor's
scheduled one — would re-embed the removed source and silently resurrect
it. The store is therefore the single source of truth every rebuild path
must consult. The presence of a row is the entire state: there is no
boolean column, per the fleet-wide schema rule. Reactivation is symmetric:
re-ingesting a source clears its tombstone, so ingestion itself is the
undo.

`markRemoved(_:now:)` upserts a tombstone with a caller-supplied
timestamp. `clearRemoved(_:)` deletes it. `removedIDs()` returns the full
set that rebuilds must subtract. `deleteAll()` supports index destruction.
Nothing enforces that a new rebuild path remembers to consult this store;
that is a blast-radius obligation on every future change.

## CorpusProviderCountsStore.swift

This file provides `CorpusProviderCountsStore`, persistence for the raw
statistics a trainable embedding signal accumulates between retrains.

The design splits cheap from heavy. The `counts` column is an opaque blob
the provider alone serializes; the store never decodes it and never
imports the providers target. Two small integer columns — document count
and vocabulary size — are lifted out of the blob so a staleness check can
ask "has the corpus grown enough to retrain?" with one tiny query instead
of deserializing a large blob. Rows are keyed by `(model_id,
model_version)`, matching how basis rows and vector rows are keyed,
because counts are only valid for the exact provider version that
accumulated them. `upsert(_:)` replaces the row whole; additive merging is
the provider's job before it calls in.

The file is candid that this is half a feature. Counts are persisted and
restored, but `Corpus.reindex` still retrains from raw chunk text rather
than from this table; the counts-backed retrain path and vector
re-projection are future work. Documentation should not present this store
as the current retrain mechanism.

## SyncManifest.swift

This file provides `CorpusKitSync.manifest(zoneIdentifier:)`, the
declarative sync contract for the `chunks` table.

The manifest declares one bidirectional synced table with primary key `id`
and conflict policy `appendOnly`. That policy is safe precisely because
chunks are content-addressed and immutable: two devices can never produce
conflicting edits to the same id, only identical re-derivations, so the
sync layer needs no merge strategy. The kit performs no sync itself; the
application hands this manifest and a storage instance to ConvergenceKit.
When VectorKit sync is also enabled, both tables should share one zone so
chunks and their vectors stay join-compatible on every device.

## Engine/SparseTypes.swift

This file provides the value types of the sparse retrieval lane:
`ImpactPosting`, `SparseHit`, `FusedHit`, and the `LaneTag` alias.

The load-bearing decision is that `ImpactPosting.impact` is an integer. A
float BM25 weight is quantized once at index build; from then on the whole
query path is integer arithmetic, which is what makes the Swift and Rust
legs bit-identical. `SparseHit` is the consumer surface: the integer score
divided back by the quantization scale. `FusedHit` carries the fused score
plus a `perLane` map of raw per-lane scores, preserved so later selection
stages can read lane signals without recomputation. `LaneTag` is a type
alias to VectorKit's enum rather than a second enum, because two identical
Swift enums are still distinct types and would make case names ambiguous
for consumers importing both kits. Posting lists are always sorted by item
id ascending — the WAND algorithm's pivoting invariant — and fused results
sort by score descending, then item id ascending.

## Engine/BM25Weighting.swift

This file provides BM25 as an impact-weighting scheme that feeds the
inverted index, plus the quantizer and query helpers.

BM25 scores a document for a term by combining the term's rarity (inverse
document frequency, IDF) with its frequency in the document, damped by
document length. The whole float computation happens exactly once, at
index build. `build(termFreqs:docLengths:parameters:)` evaluates the
classic formula per term and document, then quantizes each contribution
with `quantizeImpact(_:)` — multiply by 100 and round half to even. The
rounding mode is pinned deliberately: Swift's default rounding differs
from banker's rounding at exact halves, and both legs must agree. Term
strings map to dense integer ids in sorted order, so runs are
reproducible. `BM25Parameters` pins the defaults `k1 = 1.5` and
`b = 0.75`, tunable per estate. `queryPairs(queryTerms:termMapping:)`
turns query terms into (term id, weight 100) pairs, dropping unknown terms
and deduplicating repeats so each term contributes once.

## Engine/InvertedIndex.swift

This file provides the generic weighted inverted index with two exact
top-k algorithms: WAND and Block-Max WAND.

An inverted index maps each term to a posting list — the items containing
that term, each with a pre-quantized impact. Scoring an item for a query
is an integer dot product over shared terms. The naive approach scores
every candidate; WAND ("Weak AND") skips most of them. It keeps one cursor
per query term, sorted by current position, and computes a pivot: the
first point where the accumulated best-case impacts could beat the current
k-th best score. Items before the pivot cannot win and are skipped
wholesale. Block-Max WAND refines this with per-block maxima (block size
128): if even the tighter block bound cannot beat the threshold, the whole
block is skipped. Both algorithms are exact — they return precisely the
same top-k as a full scan — and `exhaustiveScan(query:k:)` ships as the
reference oracle for conformance tests.

The index is immutable after construction; mutation means rebuilding,
and serializing rebuilds is the wrapper's job. The internal bounded heap
implements the universal tie-break: equal scores resolve toward the
smaller item id, so results never depend on hash or insertion order. Item
ids compare as strings. The pinned constants `invertedIndexQuantScale`
(100) and `invertedIndexBlockSize` (128) are part of the cross-port
contract.

## Engine/InvertedIndexStore.swift

This file provides `InvertedIndexStore`, the persistent wrapper that lets
keyword state survive restarts without replaying chunk bodies.

The store persists only raw statistics: a term-frequency table and a
document-length table in two small SQLite tables. The weighted index
itself is derived. On demand, `buildIndex(parameters:)` runs the BM25
build over the in-memory mirrors and caches the result; every write
invalidates the cache. Persisting statistics instead of weighted postings
means changing `k1` or `b` never requires a data migration — only an
in-memory rebuild. `open()` loads all rows once, a cost proportional to
terms plus documents, never to chunk text.

`index(itemID:tokens:now:)` replaces a document's terms atomically and is
idempotent; empty tokens remove the item. `remove(itemID:)` and
`deleteAll()` complete the mutation surface, and
`topK(queryTerms:k:parameters:algorithm:)` is the one-call query path. The
actor serializes all mutation. The Rust twin owns a private database
connection with explicit batch methods; the Swift store instead shares the
estate's storage, which is why the facade manages transaction windows
around it during bulk ingest.

## BM25Index.swift

This file provides `BM25Index`, the original in-memory keyword index,
preserved as a public primitive.

The `Corpus` facade no longer uses it — durability required
`InvertedIndexStore` — but external callers that built on it keep a
working, chunk-typed surface. It holds term frequencies keyed by chunk
UUID string, tokenizes chunk text itself through an injected `Tokenizer`,
and delegates scoring to the same engine layer (BM25 weighting plus
Block-Max WAND), caching the built index between writes.
`index(_:)`, `remove(_:)`, `documentCount()`, and `topK(_:for:)` form the
surface; `topK` takes pre-tokenized terms, and the caller must tokenize
with the same vocabulary used at index time. Ties break by UUID string
order, which is not numeric UUID order but is identical on both legs.

## Engine/Fusion.swift

This file provides `Fusion`, the generalized weighted Reciprocal Rank
Fusion engine.

Reciprocal Rank Fusion (RRF) merges ranked lists without comparing their
raw scores, which live on incompatible scales. Each lane contributes
`weight × 1 / (rrfK + rank)` for every item it ranked; the sums decide the
final order. The constant `rrfK` (default 60, from the original RRF paper)
damps the advantage of rank one over rank two. The function deduplicates
within each lane — only an item's best rank counts, because a duplicate
would illegally double its contribution — and demands `rrfK > 0`, since
zero or negative values corrupt the formula. Output sorts by fused score
descending, then item id ascending.

Two overloads exist. `fuse(rankedLists:laneScores:weights:rrfK:)` takes
explicit ranks and optional raw scores to carry through into `perLane`.
`fuse(scoredLists:weights:rrfK:)` treats array position as rank; the
caller must pre-sort. The engine is a pure function over ranks and
weights — deterministic and reentrant. One caution: the configuration type
in `HybridRecall.swift` reserves an MMR field, but no diversification is
implemented here or anywhere on this path yet.

## HybridRecall.swift

This file provides `HybridRecall.recall(...)`, the canonical two-lane
retrieval pipeline.

The pipeline over-fetches a candidate window of `max(limit × 4, 32)` from
each lane, because fusion needs headroom: an item ranked eleventh in both
lanes can out-fuse an item ranked first in only one. The vector lane runs
VectorKit's nearest-neighbor search concurrently while the query is
tokenized and the keyword lane queries the inverted index. Both hit lists
become ranked lists keyed by canonical UUID strings. Canonicalization is a
deliberate security-review fix: a lowercase UUID written by the Rust leg
and an uppercase Swift keyword hit for the same item would otherwise never
fuse. `Fusion.fuse` merges the lanes with the configured weights, the
result truncates to the limit, and the winners hydrate from the bundle
store in fused order.

`HybridRecallConfiguration` pins the defaults: vector weight 0.6, keyword
weight 0.4, `rrfK` 60, and an `mmrLambda` slot that is currently declared
but never read. Score mapping is asymmetric on purpose: a vector score of
zero is a perfect Hamming match and is kept, while a keyword score of zero
means "did not match" and maps to nil. Telemetry (latency and per-lane
counts) fires at the operation boundary where it cannot affect results;
with monitoring off it costs one atomic load per metric.

## TrainableEmbeddingBasis.swift

This file provides the `TrainableEmbeddingBasis` protocol, the seam that
lets the core drive provider training without importing the providers
target.

Layering runs one way: providers depend on core, never the reverse. The
`Corpus` holds providers as type-erased values, so it needs a protocol to
ask "can you train, serialize, and reconstruct yourself?" Providers that
cannot — the deterministic provider, the named neural models, FDC, the
Apple NL providers — simply do not conform, and the facade surfaces
`CorpusKitError.notTrainable`. `reconstructBasis(from:)` is an instance
method rather than an initializer for exactly this reason: invoked on a
type-erased witness, it routes to the correct concrete type's
deserializing initializer.

The protocol has two halves. The basis half — `trainOnCorpus(texts:)`,
`serializeBasis()`, `reconstructBasis(from:)` — covers full training and
the round-trip law: a reconstructed provider embeds byte-identically to
the trained original. The counts half — `addToCounts(text:)`,
`serializeCounts()`, `restoreCounts(from:)`, `countsVocabularySize` —
maintains raw additive statistics incrementally, snapshotted at batch
boundaries because per-chunk serialization would be quadratic over an
import. Training must never read the wall clock; it is a pure function of
the texts and fixed seeds. The Rust port cannot cross-cast trait objects,
so there the embedding trait is a supertrait instead — a documented,
sanctioned divergence.

## BasisStore.swift

This file provides `BasisStore`, persistence for trained basis blobs, so a
reopened corpus embeds immediately instead of retraining.

One row per `(model_id, model_version)` in the `corpus_provider_basis`
table holds the opaque little-endian blob, a trained-at timestamp
(caller-supplied, stored as ISO 8601 text per the schema rules), and a
trained-chunk-count anchor reserved for a future auto-retrain policy.
The composite key matters: a blob trained for one provider must never load
into another, and the key matches how every vector row is keyed. Retrain
upserts in place, so exactly one row exists per provider — no history, no
orphans. Schema version 2 adds a nullable JSON `ext` column as a
forward-compatibility slot that version 1.0 writes as null and never
reads.

Like `BundleStore`, the decoder tolerates both typed-value forms — the
in-memory backend's semantic timestamps and SQLite's ISO text — because a
semantic-only reader would silently drop every row on reopen and semantic
recall would go dark on any restored estate. `upsert(_:)`,
`load(modelID:modelVersion:)`, and `deleteAll()` form the whole surface.

## CorpusKit.swift

This file provides the public entry point: the `Corpus` actor, the
`EmbeddingModel` selection enum, the `FloatLaneOutcome` result type, and
the `EncodeSpeed` quality-of-service knob. It is the largest file in the
package because it is the composition root: everything else exists so this
file can wire it together.

### The Corpus Actor and Its Provider Slots

A `Corpus` composes the bundle store, the persistent keyword index, the
vector store, the basis and counts stores, the tombstone store, and one
slot per configured embedding signal. It seals VectorKit behind its own
surface: no VectorKit type appears in a public signature except the
deliberate `sharedVectorStore` escape hatch, which lends the estate's one
vector store to the orchestrator so no second store is built over the same
table.

Each provider slot holds three things. First, the serving provider, which
embeds queries and chunks. Second, for trainable signals, a
`freshBasisBlob` — the serialized untrained basis captured at
construction. This is the from-scratch factory: training is additive, so
retraining a live provider would count the corpus twice; every retrain
instead reconstructs a fresh provider from this blob, which makes reindex
idempotent and canonical across ports. Third, a separate counts
accumulator, deliberately not the serving provider, because growing a
vocabulary in place would desync a factorized basis from its frozen
factors. Slot zero is the default signal: every single-signal entry point
delegates to it, so a one-model corpus behaves exactly like the old
single-provider design.

### Opening, Ingesting, Reindexing

`init(storage:models:)` migrates six schemas, resolves each slot (a
persisted basis reconstructs a trained provider; a corrupt blob throws
rather than serving untrained), opens the keyword index, and warm-loads a
chunk-to-source map from a body-free projection — the whole cold start
avoids reading chunk text. `ingest(_:sourceID:now:)` chunks the text,
clears any tombstone (re-ingest reactivates), inserts idempotently,
indexes keywords, folds counts, and embeds. Embedding is two-phase: any
trainable slot with no persisted basis triggers the one-and-only implicit
first-ingest training over the full corpus snapshot; all other slots fold
in under their frozen basis. Fold-in embeddings compute concurrently off
the actor — providers are `Sendable` values — and land in one batched
vector write. `ingestBatch(_:)` produces output identical to per-item
ingest but commits in windows of 512 items or 4,096 rows, long enough to
amortize disk syncs and short enough not to starve concurrent captures,
and fans embedding out in contiguous slices per core.

`reindex(now:)` is the explicit retrain trigger: reconstruct fresh, train
on all active chunks (tombstoned sources excluded), install, persist the
basis, and re-embed every active chunk under every slot. Only two train
triggers exist in the whole kit — first ingest and explicit reindex.

### Recall, Removal, Observation

`recall(_:limit:now:)` embeds the query on the default signal and
delegates to `HybridRecall`. `bm25TopKBySource(query:limit:)` is the pure
keyword lane aggregated to source granularity. The dense float lane —
`floatNearest`, `floatNearestPerSignal`, `floatFarthestPerSignal` — ranks
by true cosine similarity and never throws; unavailable states are typed
`FloatLaneOutcome` values (`.unavailableProviderOptOut`,
`.unavailableNoVocabHit`, `.unavailableNoFloatRows`, `.emptyQuery`,
`.storeError`), because a dark lane is an expected condition, not an
error. The farthest variant answers "what is unlike this?" and aggregates
by each source's closest chunk, so a source only counts as unlike when
even its best chunk is far.

`remove(sourceID:)` suppresses recall: it deletes keyword rows and every
model's vectors and writes the tombstone. `expunge(sourceID:)` scrubs the
verbatim text first, then removes, so content is destroyed even if a later
step fails. `destroyRecallIndex()` wipes every derived structure while
chunk rows survive. `count()`, `indexedSourceIDs()`,
`maintainedVocabAnchor()`, and the two Merkle root accessors round out the
observational surface.

### EmbeddingModel and the Small Types

`EmbeddingModel` names every signal the corpus can hold: `.deterministic`
(the permanent federation-grade baseline — a hash-based, lexical, fully
reproducible signal with a pinned seed), the three named neural models
(`.miniLM`, `.mpNet`, `.embeddingGemma`) that take a host-supplied
inference closure, the four trainable statistical signals
(`.randomIndexing`, `.ppmi`, `.lsa`, `.nmf`) that carry pre-built
providers, stateless `.fdc`, and the Apple-only `.nlEmbedding` and
`.nlContextualEmbedding`. `isTrainable` reports whether the carried
provider conforms to the training seam, and `reconstruct(from:)` routes a
persisted blob to the right concrete type. `EncodeSpeed` selects the
embed-concurrency cap: `.foreground` uses all cores, `.background` roughly
a quarter. A private `CorpusDefaultTokenizer` duplicates the providers'
deterministic tokenizer to avoid a circular dependency, and a private
`CorpusTextProvider` implements the tokenize-infer-project pipeline for
the named models, computing the pooled vector once per chunk for both the
engram and the float row.

## CorpusIngestQueue.swift

This file provides the asynchronous ingest pipeline as an extension on
`Corpus`: a durable queue, a background drain worker, and a single-drainer
lease. It exists so CorpusKit is a complete standalone substrate — any
consumer gets queued, multi-core encoding with no orchestrator.

`mountIngestQueue()` picks the backend by estate durability. A SQLite
estate gets a sibling `queue.sqlite` file derived deterministically from
the estate configuration — encrypted with the same key as the estate,
replacing an earlier plaintext directory queue that was a real security
hole beside an encrypted estate. An in-memory estate gets a transient
store under a fixed constant UUID, avoiding random-identity
nondeterminism. Because the physical queue can carry other streams, every
operation here is scoped to the `"encode"` stream; an unscoped wait would
deadlock on jobs this drainer never claims.

The drain loop coordinates through a `DrainLease`: one live drainer per
estate, with crash recovery on first acquisition that resets orphaned
in-flight jobs — safe precisely because the lease guarantees no other live
drainer holds them. A losing process becomes a warm standby that re-checks
every three seconds, bounded by the lease's fifteen-second staleness
window. Each drain pass claims the whole available batch, decodes jobs
(undecodable ones are terminally blocked, empty ones completed), runs
`ingestBatch` once for the batch, and retires the batch in one bulk reply.
While passes keep draining jobs, the loop spins without sleeping and
defers the vector-index publish until the burst ends — one index rebuild
per burst instead of one per pass, turning a quadratic bulk import linear.
Idle, it sleeps fifteen milliseconds, the near-realtime latency floor. A
failing item retries in place up to eight attempts before a terminal
blocked reply; in-place retry is sound only because ingest is idempotent.

`enqueueIngest(_:sourceID:now:)` and `enqueueIngestBatch(_:)` stamp jobs
with caller-supplied instants — never the wall clock — and the batch
variant wraps all inserts in one transaction, which removed the last
full-core bottleneck of bulk imports on encrypted SQLite.
`awaitIngestDrain(timeout:)` is the barrier importers use to know writes
are searchable. `setOnEncoded(_:)` installs the one callback CorpusKit
ever makes toward an orchestrator. The `IngestJob` wire format's JSON
field names are a pinned cross-port contract with the Rust twin.

## BasisCodec.swift

This file provides the shared binary codec every trainable provider uses
to serialize bases and counts.

The byte layout is the cross-port contract: the same trained state must
serialize to the same bytes on Swift and Rust, which rules out JSON (float
formatting, key order, and whitespace differ across ecosystems). The rules
are fixed: everything little-endian; floats written as raw IEEE-754 bit
patterns so negative zero and NaN round-trip exactly; strings
length-prefixed UTF-8; maps written with keys in ascending raw-byte order.
That byte-order sort exists because Swift's default string comparison is
Unicode-canonical while Rust's is byte order — the writer compares raw
UTF-8 to match. Every blob is framed with a four-byte magic tag and a
format version byte, currently 1.

`BasisWriter` is an append-only cursor with typed write methods;
`BasisReader` is a bounds-checked sequential reader whose
`expectMagic(_:)` and `expectVersion(_:)` reject wrong-provider or
future-format blobs with `CorpusKitError.decodingFailure` — never a crash,
never a silent misparse. Both are value types with no shared state.

## DeterministicTokenizer.swift

This file provides `DeterministicTokenizer`, the model-agnostic stand-in
tokenizer that ships as the version 1.0 default for the named neural
providers.

It is a hash, not a vocabulary. Words split by the canonical keyword
rules, then each word folds through FNV-1a into an id in the range two
through the vocabulary size; ids zero and one are reserved sentinels for
padding and unknown. Empty input returns a single pad token, never an
empty array. Because both legs fold through the same hash, conformance
harnesses get identical ids for identical input. The defaults (vocabulary
30,522, maximum 128 tokens) match the BERT family. The critical caveat:
feeding these ids into a real embedding model produces garbage, because
they have no relation to the model's true vocabulary. Real WordPiece and
SentencePiece tokenizers arrive with the version 1.1 model-bundle mission.

## TermDocumentCounts.swift

This file provides `TermDocumentCounts`, the shared count builder feeding
the LSA and NMF providers.

It owns three things: the vocabulary, built in encounter order (a term's
column index is fixed by the first document that mentions it, which keeps
matrix columns stable for a fixed document sequence); per-document term
frequencies; and per-term document frequencies. It deliberately does not
own weighting or factorization — those belong to the consuming providers,
which weight the same counts differently. `addDocument(_:)` is the full
training path. `addDocumentForCountsAnchor(_:)` is the lightweight
incremental path: it grows the vocabulary and the document count but keeps
no frequencies, because the heavy inputs are re-derived by re-tokenizing
the corpus at retrain time, bounding maintained state to the vocabulary's
size. The restored-vocabulary initializer likewise seeds a deserialized
provider with truthful metadata and empty frequency rows. The builder is
not thread-safe; all writes must finish before reads.

## ReducedVocab.swift

This file provides the shared vocabulary-reduction step for the dense
factorizations, LSA and NMF.

A dense matrix over tens of thousands of terms is unfactorizable on a
device — the comment estimates ten-to-the-fifteenth operations. The fix is
to keep only the most informative columns. Below the cap (512 by default)
the function is a strict no-op, so small estates and every conformance
fixture behave exactly as before reduction existed. Above the cap it drops
terms seen in only one document (pure noise), ranks the rest by document
frequency descending — terms that co-occur across many documents carry the
latent structure a factorization can find — and breaks ties by raw UTF-8
byte order, matching Rust's string ordering so both legs select identical
vocabularies. The selection is shared rather than per-provider because
informativeness is a corpus property, identical for both factorizations.
`ReducedVocabulary` freezes the kept terms, the projection map, and the
row-remapping table.

## RandomIndexingProvider.swift

This file provides `RandomIndexingProvider`, the first honest
distributional signal in the dense lane.

Random Indexing gives every term a deterministic sparse "index vector":
2,048 dimensions with exactly ten nonzero entries of plus or minus one.
The generator seeds a counter-based random stream from the FNV hash of
the lowercased term and draws exactly twenty values — ten positions, ten
signs — with collisions resolved last-wins rather than by rejection, so
the draw count is constant and the Swift and Rust streams stay aligned. A
term's meaning is then learned by addition: sliding a window of four over
training text, each term accumulates the index vectors of its neighbors
into a context vector. Terms that keep similar company converge, which is
genuine co-occurrence semantics at almost no computational cost, and the
accumulation is incremental by construction.

Embedding text sums the context vectors of its in-vocabulary terms and
normalizes to unit length. The float lane is honest about failure: an
untrained provider opts out with an empty vector, and a trained provider
whose query is entirely out of vocabulary throws a typed vocabulary-miss
error that the facade maps to the right dark-lane outcome. The basis blob
(magic `RIB1`) is the whole vocabulary map — Random Indexing has no
separate finalize step — and the counts blob (`RICT`) carries the same
payload under a distinct magic so a counts row can never be misread as a
basis row. The projection seed spells `RI_V1_MX`.

## PpmiProvider.swift

This file provides `PpmiProvider`, the co-occurrence signal weighted by
positive pointwise mutual information.

PPMI asks of each word pair: do these words co-occur more than chance
would predict? The score is the logarithm of the observed co-occurrence
probability over the product of the individual probabilities, floored at
zero. Frequent-but-meaningless neighbors (the "the" problem) score near
zero; genuinely associated pairs keep full weight. Training is two-phase.
Phase one counts: a sliding window of four accumulates pair counts, term
counts, and totals, additively across calls. Phase two, `finalize()`,
converts counts to context vectors: each term's vector is the
PPMI-weighted sum of its neighbors' Random Indexing index vectors, in the
same 2,048-dimension space with the same generator, so the two signals are
directly comparable. The file warns explicitly that this is mathematically
distinct from unweighted Random Indexing and must not be "simplified"
into it.

Embedding, opt-out, and vocabulary-miss behavior mirror the Random
Indexing provider. The basis blob (`PPB1`) persists only the derived
vectors; the count tables are training scratch. The counts blob (`PPMC`)
persists the raw additive state — including the nested pair-count map,
serialized with byte-sorted keys — so a retrain can resume counting
without re-tokenizing. Stored vectors are kept unnormalized;
normalization happens at embed time so tests can inspect raw sums. The
projection seed spells `PPMI_V1M` and must never equal the Random
Indexing seed, because the seed partitions vector storage by model.

## LsaProvider.swift

This file provides `LsaProvider`, the latent semantic analysis signal.

LSA finds hidden topic structure by factorizing a term-document matrix
with the singular value decomposition (SVD). Words that appear in similar
documents land near each other in the latent space, so synonyms co-locate
even when they never co-occur. Training builds on the shared count
builder: reduce the vocabulary (the ADR-022 step), weight each cell by
log term frequency times smoothed IDF (add-one smoothing on both sides,
so an unseen query term still gets positive weight), and decompose with
`JacobiSVD` from SubstrateML at rank 64. The Jacobi method is chosen
because its sweep count is pinned — thirty sweeps, exactly — making the
factorization a fixed computation rather than a convergence-dependent
one; changing the sweep count invalidates every conformance vector. Wide
matrices are transposed for the decomposition and the factors swapped
back, so downstream code sees one orientation.

New text embeds by the classical fold-in formula: project the query's
weighted term vector through the factor matrix, scaling by the inverse
singular values and skipping values below a small floor. Training
documents use the exact projection instead. The dark-lane contract
matches the other trainable providers. The basis blob (`LSB1`) persists
the reduced vocabulary, the IDF weights, and the raw factor matrices —
port-neutral, so each leg re-derives its own document vectors — and the
counts blob (`LSAC`) persists only the vocabulary and document-count
anchors, per the re-tokenize-at-retrain decision. The projection seed
spells `LSA_V1_M`.

## NmfProvider.swift

This file provides `NmfProvider`, the non-negative matrix factorization
signal.

NMF factorizes the term-document matrix into two non-negative factors, so
every latent dimension reads as an additive combination of terms — parts,
not contrasts. The matrix is oriented terms-by-documents, weighted by log
term frequency only: NMF requires non-negative input, and its update
steps are most stable with uniformly scaled entries, so IDF is deliberately
omitted. Factorization reuses SubstrateML's alternating least squares at
rank 32 with two pinned determinism devices: the convergence tolerance is
set to zero, which forces exactly one hundred iterations on every platform
regardless of floating-point convergence behavior, and factor
initialization is seeded with a fixed constant. Document embeddings are
the normalized factor columns, precomputed at finalize.

Queries fold in through a pseudo-inverse projection onto each factor
column, with a small epsilon guarding the denominator, then normalize.
Serialization follows the family pattern: the basis blob (`NMB1`) carries
configuration, the reduced vocabulary, and both raw factor matrices; the
counts blob (`NMFC`) carries anchors only. The projection seed spells
`NMF_V1_M`. The dark-lane contract matches the other trainable providers.

## FdcProvider.swift

This file provides `FDCProvider`, the taxonomic co-classification signal —
the one honest signal that needs no training at all.

The provider classifies text with LatticeLib's FDC engine, which lives in
the moot-semantics repository and is not reimplemented here. `FDC.encode`
returns a lattice code or nothing; `FDC.ancestors` returns the code's
chain up the classification hierarchy. The provider turns that chain into
geometry: each node in the chain gets a deterministic 256-dimension unit
vector generated from the hash of its code string, and the vectors sum
with weight one over depth-plus-one — shared roots give any two texts a
similarity floor, and shared deep ancestors add to it. Texts filed near
each other in the subject hierarchy therefore embed near each other,
regardless of surface wording. The node-vector generator is deliberately
a different pipeline from Random Indexing's, and the input domains cannot
collide.

Honesty governs the edges. Text the classifier cannot resolve returns
UNRESOLVED, and the provider opts out with an empty float vector and a
zero engram — unclassifiable text must not contribute false similarity.
The provider is stateless and `Sendable`; its determinism rests on
LatticeLib's own agreement property. The projection seed spells
`FDC_V1_P`, and the free function `fdcNodeVector(code:)` is public so
conformance tests can pin individual node vectors.

## DefaultEnsemble.swift

This file provides `CorpusEnsemble.defaultEnsemble()`, the single source
of truth for the production recall ensemble.

The factory returns the five honest signals in fixed order: Random
Indexing, PPMI, LSA, NMF, FDC. Order is load-bearing — the first element
becomes the corpus's default signal. It is a function rather than a shared
constant because the four trainable providers are reference types holding
mutable trained state; a shared array would alias one provider instance
across every estate. Fresh construction gives each estate its own
untrained providers, which the corpus lifecycle then trains and persists
under their own model identifiers. The factory lives in the providers
target because it names concrete types; the core's `EmbeddingModel` enum
never does.

## MiniLMTextProvider.swift, MPNetTextProvider.swift, EmbeddingGemmaProvider.swift

These three files provide the named neural embedding providers: MiniLM-L6
v2 (384 dimensions), mpnet-base-v2 (768), and EmbeddingGemma 300M (768).
They share one structure, so they are described together; each fact below
holds for all three unless noted.

Each provider runs the same pipeline: tokenize, run the host-injected
inference closure to get a pooled float vector, and project that vector to
a 256-bit engram with `FloatSimHash` from SubstrateML. An engram is a
fixed-size binary fingerprint; similar vectors produce similar engrams, so
the vector lane can compare chunks by fast Hamming distance. The inference
closure is the doctrine's model seam: the provider never holds a CoreML
model, which keeps it testable without a model bundle and leaves model
loading — different on iOS, macOS, and CI — to the host, composed once at
startup. Each provider pins its own projection seed so its fingerprints
are model-tagged: `MINLM_v1`, `MPNET_v1`, and `EMBGM_v1` in ASCII. Seeds
partition vector storage by model and must never change or collide.

Each exposes three surfaces. `embed` returns the engram; `embedFloat`
returns the raw pooled vector for the true-cosine float lane; `embedPair`
runs inference once and derives both, halving the cost of ingest, which
needs both per chunk. Empty input short-circuits to a zero engram and an
empty vector. All three default to the `DeterministicTokenizer` stand-in —
EmbeddingGemma's is sized for its SentencePiece vocabulary of 256,000 and
context of 2,048 tokens — so until real tokenizers ship, embedding values
are a property of the host's model bundle. What the kit itself owns, the
token-to-engram pipeline given a pooled vector, is bit-identical across
ports.

## NLEmbeddingProvider.swift

This file provides `NLEmbeddingProvider`, the Apple sentence-embedding
signal built on the operating system's bundled `NLEmbedding` model.

It is the cheap, immediate Apple-native lane: no download, no CoreML seam,
no training — the OS framework is the model, so there is nothing for a
host to inject. The provider looks up the OS sentence model for its
configured language, embeds the text, casts the result to floats, and
normalizes to unit length with the substrate's canonical vector
operations, keeping the float lane's cosine on a unit sphere like every
other provider. Absence is graceful: no model for the language, or text
the model cannot embed, returns an empty vector — a typed lane opt-out,
not an error. The projection seed spells `APNLEMB1`, defined here rather
than in the doctrine's CoreML seed table but under the same never-collide
rule. The file is compiled only where NaturalLanguage exists; there is no
Rust counterpart, a sanctioned Swift-only divergence recorded in ADR-019,
and recall fusion simply handles the absent lane elsewhere.

## NLContextualEmbeddingProvider.swift

This file provides `NLContextualEmbeddingProvider`, the higher-quality
Apple lane built on the on-device `NLContextualEmbedding` transformer.

The transformer needs a per-language downloadable asset that may be
absent, and the file's central rule is that an embed call must never
trigger a network fetch as a side effect. The provider checks asset
availability with a free, synchronous call and opts out with an empty
vector when the asset is missing; prefetching assets is the host
application's job, done before constructing the provider. When assets
exist, the provider runs the transformer, mean-pools the per-token vectors
(the conventional strategy for a transformer without a sentence-pooling
head, with a defensive dimension guard that skips malformed token
vectors), and normalizes. Every failure mode collapses to the empty-vector
opt-out, because a missing asset is an expected operational state. The
projection seed spells `APNLCTX1`, distinct from the sentence provider's
so the two lanes key to separate storage partitions. Apple-only, no Rust
counterpart, per ADR-019.

## Rust Port and Conformance

The `rust/` directory mirrors the core target and the `rust-providers/`
directory mirrors the providers target, matching the two-target split so
core consumers never depend on provider code. The core crate ships the
same chunk, chunker, tokenizer, store, engine, hybrid-recall, ingest-queue,
and `Corpus` types; concurrency uses locks where Swift uses actors, and
the queue and inverted-index store own private connections where Swift
shares the estate's storage. The providers crate ships the deterministic
tokenizer, the shared count and vocabulary-reduction code, the basis
codec, the five honest signals, and the three named neural providers over
the same host-injected inference seam. Neither crate bundles model
weights.

Three fixture families gate the ports. The shared canonical vectors in
`Tests/SharedVectors/` — BM25 impacts, per-provider embedding vectors, and
serialized basis blobs — are read by both legs and must reproduce byte for
byte. The Rust test suite additionally pins basis serialization
byte-for-byte for all four trainable providers and round-trips the counts
seam. The two Apple NaturalLanguage providers are the one sanctioned
divergence: they exist only in Swift, and the parity baseline is the
deterministic and classical providers. When you change either leg, run
both test suites; the fixtures are the contract.

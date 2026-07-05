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

This document walks through each source file in the package. Read
`OVERVIEW.md` first for the big picture. This document follows pipeline
order. It starts with the content types and the stores. Next comes the
keyword engine, plus fusion and recall. After that comes the
trainable-basis machinery. It ends with the `Corpus` facade, the ingest
queue, and the providers target.

## Chunk.swift

This file holds `Chunk`, the basic retrievable unit. It also
holds `ScoredChunk`, the result type recall returns.

A chunk's id is content-addressed. Its UUID is computed, never
assigned. The function `deriveID(sourceID:startOffset:text:)` builds an
RFC 4122 version 5 UUID. It hashes the source identifier, the start
offset, and the exact text with SHA-1. The three fields join through the
Unicode unit separator. That separator stops different inputs from
colliding through ambiguous concatenation.

Content addressing is what makes the rest of the kit work. Re-ingesting
the same source reproduces the same identities. A repeat insert becomes
a harmless no-op. Two federated devices that write the same
chunk converge on one row instead of conflicting. The fixed namespace
bytes are permanent. Changing them would re-key each chunk in each
estate. It would also break the join to existing vector rows, since
the chunk id doubles as the VectorKit item identifier.

Two initializers exist. The content-addressed one computes the id.
Normal ingestion always takes this path. The explicit-id one
reconstructs a chunk whose id is already known, such as a decoded
storage row. The caller must ensure that id truly matches the content.
The `hlc` field is a hybrid logical clock stamp, a timestamp that also
encodes causal order. It is always caller-supplied. `ScoredChunk` pairs
a chunk with its fused `score`. It also carries the raw `vectorScore`
and `keywordScore`. The kit keeps these two scores separate. A caller
can then diagnose which lane produced a hit.

## Chunker.swift

This file holds `Chunker`. It splits raw source text into ordered
chunks with sentence-aware boundaries.

A chunk should never cut a sentence in half. A half sentence embeds
poorly and reads worse. The chunker segments the text into sentences
first, for that reason. It delegates to `EideticLib.sentences(_:)`, so
segmentation logic lives in one shared place. It then fills a buffer
greedily. Sentences accumulate until adding the next one would pass the
target size. At that point the buffer flushes into a chunk. The tail of
the flushed chunk carries over as overlap into the next buffer. Overlap
means a match near a boundary still brings its surrounding context
along. Offsets into the original text are tracked exactly, since the
offset is part of each chunk's content-addressed id.

`ChunkerConfiguration` holds three settings: `targetChars` (default 800),
`overlapChars` (default 100), and `respectSentences` (default true). Its
initializer clamps nonsense values. Overlap can never reach the target,
since that would loop forever. `Chunker.chunk(text:sourceID:configuration:hlcGenerator:)`
is the single entry point. The HLC generator passes in as `inout`. The
chunker is the sole authority for chunk order within one call, and it
stamps each chunk in emission order. Changing the default sizes changes
chunk boundaries. That in turn changes chunk identities for re-ingested
content, so the defaults are pinned to the substrate reference.

## Tokenizer.swift

This file holds the `Tokenizer` protocol. It also holds the single
canonical keyword tokenizer.

A tokenizer serves two different masters. An embedding model needs its
own vocabulary of integer token ids. The protocol requires
`tokenize(_:) -> [Int32]` for that reason, plus id fields: `vocabID`, `maxTokens`,
and pad and unknown ids. Keyword search needs plain words instead. The
protocol also requires `keywordTokens(_:) -> [String]`. The protocol
extension supplies a default for that second method. It delegates to the
free function `defaultKeywordTokens(_:)`, which lowercases the text, then
keeps runs of alphabetic or ASCII-digit characters and splits on
everything else.

There is just one definition of keyword tokenization in the module,
for a reason. BM25 and each distributional embedding signal must agree
on what a "term" is. Otherwise the keyword lane and the semantic lane
would score different vocabularies. Hybrid recall would then quietly
degrade. The function is also parity-critical. The Rust port implements
the same rules, and committed conformance vectors on both legs break if
the rules change. A provider that overrides `keywordTokens` breaks this
guarantee by convention, not by compiler error.

## CorpusKitError.swift

This file holds `CorpusKitError`, the module's single error enum.

Seven flat cases cover the failure classes: `encodingFailure`,
`decodingFailure`, `tokenizerUnavailable`, `modelUnavailable`,
`embeddingFailed`, `storeUnavailable`, and `notTrainable`. Each case
carries a plain message string. Callers mostly log the message, or
surface it, rather than branch on structured data. The `notTrainable`
case exists for `EmbeddingModel.reconstruct(from:)`. Providers without a
trained basis surface this error instead of silently substituting a
wrong provider. Three provider families fall into this group: the
deterministic provider, the named neural models, and the stateless FDC
provider. The enum is `Equatable` on its message strings. Tests that
compare errors must so construct the exact message.

## BundleStore.swift

This file holds `BundleStore`, the persistence layer for chunks. It
is the content half of each content-plus-vector bundle. The vector half
lives in VectorKit, joined by the chunk's UUID string.

`BundleStore` is an actor. It wraps `any Storage` from PersistenceKit.
The application picks the backend, SQLite or in-memory. The kit itself
does not. It owns schema version 3: a `chunks` table of ten columns, plus a
`corpus_metadata` table, with indices on `source_id` and `hlc`. Inserts
do not go straight to the row store. They pass through a hashing
decorator first. That decorator computes a content hash with
`MerkleHash.leaf` on each write and emits dirty-chain events. A Merkle
hash chain lets the estate prove its content has not drifted. Each chunk
hashes. Each source's chunks combine into a per-corpus root. All corpus
roots combine into one global root. Since the hashing callback is
synchronous, a small lock-guarded `ParentChainCache` pre-stages each
chunk's parent identifiers before the insert. The corpus and root
identifiers derive from fixed namespace strings, through SHA-256, so
both language legs compute the same chains.

`insert(_:)` is idempotent by design. It attempts a plain insert per
chunk. It treats a duplicate-key error as the documented no-op path.
First write wins, and this is sound only since identities are
content-addressed. The method returns only the chunks that were actually
new, in input order. A caller that maintains derived state never
double-counts a re-ingested chunk this way. Seven methods form the read
surface: `get`, `getMany`, `chunksForSource`, `allChunks`,
`allSourceIDs`, `count`, and `chunkSourcePairs`. All are thin query
wrappers. `chunkSourcePairs()` omits chunk bodies on purpose, so opening
a corpus stays cheap.
`scrubText(sourceID:)` is the hard-delete seam. It zeroes the `text`
column through a direct update. This is why the schema declares the
table `appendOnly: false`, even though the API treats chunks as
immutable. Immutability here is a convention the surface enforces, not a
database trigger. The sync layer has its own separate `appendOnly`
conflict policy, and the two should not be confused.

One decoding rule deserves emphasis. The SQLite backend round-trips
UUIDs as text and HLCs as packed integers. The in-memory backend
preserves the semantic typed values instead. `decodeChunk` and its
helpers accept both forms. A past semantic-only decoder silently
dropped each persisted chunk on reopen, and in-memory tests never
caught it. Any new decode path must handle both forms.

## RemovedSourceStore.swift

This file holds `RemovedSourceStore`, the tombstone table that makes
source removal stick.

Chunk rows are never deleted. So "removing" a source can only delete its
vectors and its keyword postings. Without a durable marker, any rebuild
that replays `allChunks()` would re-embed the removed source. That
includes an explicit reindex, or the autonomic governor's scheduled one.
The source would then silently resurrect. The store is, for that
reason, the single source of truth each rebuild path must consult. The presence of
a row is the entire state. There is no boolean column, per the
fleet-wide schema rule. Reactivation is symmetric: re-ingesting a source
clears its tombstone, so ingestion itself is the undo.

`markRemoved(_:now:)` upserts a tombstone with a caller-supplied
timestamp. `clearRemoved(_:)` deletes it. `removedIDs()` returns the full
set that rebuilds must subtract. `deleteAll()` supports index
destruction. Nothing enforces that a new rebuild path remembers to
consult this store. That remains a blast-radius obligation on each
future change.

## CorpusProviderCountsStore.swift

This file holds `CorpusProviderCountsStore`. It persists the raw
statistics a trainable embedding signal builds up between retrains.

The design splits cheap state from heavy state. The `counts` column is
an opaque blob the provider alone serializes. The store never decodes
it, and never imports the providers target. Two small integer columns,
document count and vocabulary size, are lifted out of the blob. A
staleness check can then ask "has the corpus grown enough to retrain?"
with one tiny query, instead of deserializing a large blob. Rows key by
`(model_id, model_version)`. This matches how basis rows and vector rows
are keyed, since counts are only valid for the exact provider version
that accumulated them. `upsert(_:)` replaces the row whole. Additive
merging is the provider's job before it calls in.

The file is candid that this is half a feature. Counts are persisted and
restored. But `Corpus.reindex` still retrains from raw chunk text rather
than from this table. The counts-backed retrain path, and the matching
vector re-projection, are future work. Documentation should not present
this store as the current retrain mechanism.

## SyncManifest.swift

This file holds `CorpusKitSync.manifest(zoneIdentifier:)`, the
declarative sync contract for the `chunks` table.

The manifest declares one bidirectional synced table. Its primary key is
`id`, and its conflict policy is `appendOnly`. That policy is safe,
since chunks are content-addressed and immutable. Two devices can
never produce conflicting edits to the same id. They can only produce
the same re-derivations, so the sync layer needs no merge strategy. The kit performs no sync itself. The application hands this
manifest, and a storage instance, to ConvergenceKit. When VectorKit sync
is also enabled, both tables should share one zone. That keeps chunks
and their vectors join-compatible on each device.

## Engine/SparseTypes.swift

This file holds the value types of the sparse retrieval lane:
`ImpactPosting`, `SparseHit`, `FusedHit`, and the `LaneTag` alias.

The load-bearing decision is that `ImpactPosting.impact` is an integer.
A float BM25 weight quantizes once, at index build time. From then on
the whole query path runs on integer arithmetic. That is what makes the
Swift and Rust legs bit-identical. `SparseHit` is the consumer surface.
It divides the integer score back by the quantization scale.
`FusedHit` carries the fused score, plus a `perLane` map of raw
per-lane scores. Those raw scores stay available so later selection
stages can read lane signals without recomputing them. `LaneTag` is a
type alias to VectorKit's enum, not a second enum. Two matching Swift
enums would still be distinct types, and that would make case names
ambiguous for a consumer that imports both kits. Posting lists always
sort by item id ascending, the WAND algorithm's pivoting invariant.
Fused results sort by score descending, then by item id ascending.

## Engine/BM25Weighting.swift

This file holds BM25 as an impact-weighting scheme that feeds the
inverted index, plus the quantizer and query helpers.

BM25 scores a document for a term by combining two things: the term's
rarity, called inverse document frequency (IDF), and its frequency in
the document. Document length damps the score. The whole float
computation happens just once, at index build. `build(termFreqs:docLengths:parameters:)`
evaluates the classic formula per term and document. It then quantizes
each contribution with `quantizeImpact(_:)`, which multiplies by 100 and
rounds half to even. The rounding mode is pinned on purpose. Swift's
default rounding differs from banker's rounding at exact halves, and
both legs must agree. Term strings map to dense integer ids in sorted
order, so runs stay reproducible. `BM25Parameters` pins the defaults:
`k1` equals 1.5, and `b` equals 0.75. Both values are tunable per
estate. `queryPairs(queryTerms:termMapping:)` turns query terms into
term-id and weight-100 pairs. It drops unknown terms. It deduplicates
repeats, so each term contributes just once.

## Engine/InvertedIndex.swift

This file holds the generic weighted inverted index. It ships two
exact top-k algorithms: WAND and Block-Max WAND.

An inverted index maps each term to a posting list, the items containing
that term, each carrying a pre-quantized impact. Scoring an item for a
query is an integer dot product over shared terms. The naive approach
scores each candidate. WAND, short for "Weak AND," skips most of them
instead. It keeps one cursor per query term, sorted by current position.
It computes a pivot: the first point where the accumulated best-case
impacts could beat the current k-th best score. Items before the pivot
cannot win, so the algorithm skips them wholesale. Block-Max WAND
refines this idea with per-block maxima, using a block size of 128. If
even the tighter block bound cannot beat the threshold, the whole block
gets skipped. Both algorithms are exact. They return precisely the same
top-k as a full scan. `exhaustiveScan(query:k:)` ships as the reference
oracle for conformance tests.

The index is immutable after construction. Mutation means rebuilding it,
and serializing rebuilds is the wrapper's job. The internal bounded heap
implements the universal tie-break: equal scores resolve toward the
smaller item id, so results never depend on hash order or insertion
order. Item ids compare as strings. Two pinned constants belong to the
cross-port contract: `invertedIndexQuantScale` (100) and
`invertedIndexBlockSize` (128).

## Engine/InvertedIndexStore.swift

This file holds `InvertedIndexStore`, the persistent wrapper that
lets keyword state survive restarts without replaying chunk bodies.

The store persists only raw statistics: a term-frequency table and a
document-length table, in two small SQLite tables. The weighted index
itself is derived, not stored. On demand, `buildIndex(parameters:)` runs
the BM25 build over the in-memory mirrors and caches the result. Each
write invalidates the cache. Persisting statistics instead of weighted
postings means changing `k1` or `b` never requires a data migration. It
only requires an in-memory rebuild. `open()` loads all rows once, at a
cost proportional to terms plus documents, never to chunk text.

`index(itemID:tokens:now:)` replaces a document's terms atomically, and
it is idempotent. Empty tokens remove the item. `remove(itemID:)` and
`deleteAll()` complete the mutation surface. `topK(queryTerms:k:parameters:algorithm:)`
is the one-call query path. The actor serializes all mutation. The Rust
twin owns a private database connection with explicit batch methods.
The Swift store instead shares the estate's storage, which is why the
facade manages transaction windows around it during bulk ingest.

## BM25Index.swift

This file holds `BM25Index`, the original in-memory keyword index. It
is preserved as a public primitive.

The `Corpus` facade no longer uses it. Durability required
`InvertedIndexStore` instead. External callers that built on the older
type keep a working, chunk-typed surface. It holds term frequencies
keyed by chunk UUID string. It tokenizes chunk text itself, through an
injected `Tokenizer`. It delegates scoring to the same engine layer: BM25
weighting, plus Block-Max WAND. The built index caches between writes.
`index(_:)`, `remove(_:)`, `documentCount()`, and `topK(_:for:)` form the
surface. `topK` takes pre-tokenized terms, and the caller must tokenize
with the same vocabulary used at index time. Ties break by UUID string
order. That order differs from numeric UUID order. It stays the same
on both legs, still.

## Engine/Fusion.swift

This file holds `Fusion`, the generalized weighted Reciprocal Rank
Fusion engine.

Reciprocal Rank Fusion, called RRF, merges ranked lists. It never
compares raw scores, since those scores live on incompatible scales. Each
lane contributes `weight × 1 / (rrfK + rank)` for each item it ranked.
The sums decide the final order. The constant `rrfK` defaults to 60,
from the original RRF paper, and it damps the advantage of rank one over
rank two. The function deduplicates within each lane. Only an item's
best rank counts there, since a duplicate would illegally double its
contribution. The function also demands `rrfK > 0`, since zero or
negative values would corrupt the formula. Output sorts by fused score
descending, then by item id ascending.

Two overloads exist. `fuse(rankedLists:laneScores:weights:rrfK:)` takes
explicit ranks. It takes optional raw scores too, to carry through into
`perLane`. `fuse(scoredLists:weights:rrfK:)` treats array position as
rank instead, so the caller must pre-sort its input. The engine is a
pure function over ranks and weights. It is deterministic and reentrant.
One caution applies here. The configuration type in
`HybridRecall.swift` reserves an MMR field. No diversification logic
exists yet, on this path or anywhere else.

## HybridRecall.swift

This file holds `HybridRecall.recall(...)`, the canonical two-lane
retrieval pipeline.

The pipeline over-fetches a candidate window from each lane. That
window's size is `max(limit × 4, 32)`, since fusion needs headroom. An
item ranked eleventh in both lanes can out-fuse an item ranked first in
only one. The vector lane runs VectorKit's nearest-neighbor search
concurrently, while the query gets tokenized and the keyword lane
queries the inverted index. Both hit lists become ranked lists, keyed by
canonical UUID strings. Canonicalization is a deliberate security-review
fix. A lowercase UUID written by the Rust leg, and an uppercase Swift
keyword hit for the same item, would otherwise never fuse. `Fusion.fuse`
merges the lanes with the configured weights. The result truncates to
the limit. The winners then hydrate from the bundle store, in fused
order.

`HybridRecallConfiguration` pins four defaults. The vector weight is
0.6. The keyword weight is 0.4. The `rrfK` value is 60. The
`mmrLambda` slot exists, but the pipeline never reads it. Score mapping
is asymmetric on purpose. A vector score of zero is a perfect Hamming
match, and the pipeline keeps it. A keyword score of zero instead means
"did not match," and it maps to nil. Telemetry fires at the operation
boundary, where it cannot affect results. It covers latency and
per-lane counts. With monitoring off it costs one atomic load per
metric.

## TrainableEmbeddingBasis.swift

This file holds the `TrainableEmbeddingBasis` protocol. It is the
seam that lets the core drive provider training without importing the
providers target.

Layering runs one way: providers depend on core, never the reverse. The
`Corpus` holds providers as type-erased values. It needs a protocol to
ask one question: can the provider train, serialize, and reconstruct
itself? Some providers
cannot: the deterministic provider, the named neural models, FDC, and
the Apple NL providers. They simply do not conform, and the facade
surfaces `CorpusKitError.notTrainable`. `reconstructBasis(from:)` is an
instance method rather than an initializer, for exactly this reason.
Invoked on a type-erased witness, it routes to the correct concrete
type's deserializing initializer.

The protocol has two halves. The basis half covers full training and the
round-trip law: a reconstructed provider embeds byte-identically to the
trained original. It includes `trainOnCorpus(texts:)`, `serializeBasis()`,
and `reconstructBasis(from:)`. The counts half maintains raw additive
statistics incrementally instead. It includes `addToCounts(text:)`,
`serializeCounts()`, `restoreCounts(from:)`, and `countsVocabularySize`.
Snapshots happen at batch boundaries, since per-chunk serialization
would be quadratic over an import. Training must never read the wall
clock. It is a pure function of the texts and fixed seeds. The Rust port
cannot cross-cast trait objects, so there the embedding trait is a
supertrait instead, a documented and sanctioned divergence.

## BasisStore.swift

This file holds `BasisStore`, persistence for trained basis blobs, so
a reopened corpus embeds right away instead of retraining.

One row per `(model_id, model_version)` lives in the
`corpus_provider_basis` table. Each row holds the opaque little-endian
blob, a trained-at timestamp, and a trained-chunk-count anchor reserved
for a future auto-retrain policy. The timestamp is caller-supplied, and
it stores as ISO 8601 text, per the schema rules. The composite key
matters: a blob trained for one provider must never load into another,
and the key matches how each vector row is keyed. Retrain upserts in
place, so just one row exists per provider. There is no history, and
there are no orphans. Schema version 2 adds a nullable JSON `ext` column
as a forward-compatibility slot. Version 1.0 writes that column as null
and never reads it.

Like `BundleStore`, the decoder tolerates both typed-value forms: the
in-memory backend's semantic timestamps, and SQLite's ISO text. A
semantic-only reader would otherwise silently drop each row on reopen,
and semantic recall would go dark on any restored estate.
`upsert(_:)`, `load(modelID:modelVersion:)`, and `deleteAll()` form the
whole surface.

## CorpusKit.swift

This file holds the public entry point: the `Corpus` actor, the
`EmbeddingModel` selection enum, the `FloatLaneOutcome` result type, and
the `EncodeSpeed` quality-of-service knob. It is the largest file in the
package, since it is the composition root. Everything else exists so
this file can wire it together.

### The Corpus Actor and Its Provider Slots

A `Corpus` composes the bundle store, the persistent keyword index, and
the vector store. It also composes the basis store, the counts store,
and the tombstone store. It holds one slot per configured embedding
signal, too. It seals VectorKit behind its own surface. No VectorKit type appears in a
public signature, except the deliberate `sharedVectorStore` escape
hatch. That hatch lends the estate's one vector store to the
orchestrator, so no second store gets built over the same table.

Each provider slot holds three things. First, the serving provider,
which embeds queries and chunks. Second, trainable signals get a
`freshBasisBlob`. This is the serialized untrained basis, captured at
construction. It works as the from-scratch factory. Training is additive,
so retraining a live provider would count the corpus twice. Each
retrain instead reconstructs a fresh provider from this blob. That makes
reindex idempotent and canonical across ports. Third, a separate counts
accumulator, kept apart from the serving provider on purpose. Growing
a vocabulary in place would desync a factorized basis from its frozen
factors. Slot zero is the default signal. Each single-signal entry
point delegates to it, so a one-model corpus behaves just like the old
single-provider design.

### Opening, Ingesting, Reindexing

`init(storage:models:)` migrates six schemas. It resolves each slot: a
persisted basis reconstructs a trained provider, and a corrupt blob
throws rather than serving untrained data. It opens the keyword index.
It warm-loads a chunk-to-source map from a body-free projection, so the
whole cold start avoids reading chunk text. `ingest(_:sourceID:now:)`
performs six steps in order. It chunks the text. It clears any
tombstone, since re-ingest reactivates a source. It inserts the chunks
idempotently. It indexes keywords. It folds counts. It embeds the
chunk. Embedding runs in two phases. Any trainable slot with no persisted basis triggers the
one-and-only implicit first-ingest training, over the full corpus
snapshot. Each other slot folds in under its frozen basis instead.
Fold-in embeddings compute concurrently off the actor, since providers
are `Sendable` values, and they land in one batched vector write.
`ingestBatch(_:)` produces output identical to per-item ingest. It
commits in windows of 512 items or 4,096 rows, long enough to amortize
disk syncs and short enough not to starve concurrent captures. It also
fans embedding work out in contiguous slices per core.

`reindex(now:)` is the explicit retrain trigger. It performs five steps
in order. It reconstructs fresh providers. It trains on all active
chunks, excluding tombstoned sources. It installs the result. It
persists the basis. It re-embeds each active chunk under each slot.
Only two train triggers exist in the whole kit: first ingest, and
explicit reindex.

### Recall, Removal, Observation

`recall(_:limit:now:)` embeds the query on the default signal and
delegates to `HybridRecall`. `bm25TopKBySource(query:limit:)` is the
pure keyword lane, aggregated to source granularity. The dense float
lane covers three methods: `floatNearest`, `floatNearestPerSignal`, and
`floatFarthestPerSignal`. All three rank by true cosine similarity. None
of them ever throws. Unavailable states are typed `FloatLaneOutcome` values instead:
`.unavailableProviderOptOut`, `.unavailableNoVocabHit`,
`.unavailableNoFloatRows`, `.emptyQuery`, and `.storeError`. A dark lane
is an expected condition here, not an error. The farthest variant
answers "what is unlike this?" It aggregates by each source's closest
chunk, so a source only counts as unlike when even its best chunk is
far.

`remove(sourceID:)` suppresses recall. It deletes keyword rows and each
model's vectors, and it writes the tombstone. `expunge(sourceID:)` goes
further: it scrubs the verbatim text first, then removes the source, so
content is destroyed even if a later step fails. `destroyRecallIndex()`
wipes each derived structure while chunk rows survive. Four more
methods round out the observational surface: `count()`,
`indexedSourceIDs()`, `maintainedVocabAnchor()`, and two Merkle-root
readers.

### EmbeddingModel and the Small Types

`EmbeddingModel` names each signal the corpus can hold. `.deterministic`
is the permanent federation-grade baseline. It is hash-based and
lexical. It is fully reproducible, with a pinned seed. Three named
neural models follow it: `.miniLM`, `.mpNet`, and `.embeddingGemma`.
Each one takes a host-supplied inference closure. Four trainable statistical
signals carry pre-built providers: `.randomIndexing`, `.ppmi`, `.lsa`,
and `.nmf`. The stateless `.fdc` signal comes next. Two Apple-only
signals close the list: `.nlEmbedding` and `.nlContextualEmbedding`.
`isTrainable` reports whether the carried provider conforms to the
training seam. `reconstruct(from:)` routes a persisted blob to the right
concrete type. `EncodeSpeed` selects the embed-concurrency cap.
`.foreground` uses all cores, and `.background` uses roughly a quarter.
A private `CorpusDefaultTokenizer` duplicates the providers'
deterministic tokenizer, to avoid a circular dependency. A private
`CorpusTextProvider` implements the tokenize-infer-project pipeline for
the named models. It computes the pooled vector once per chunk, for
both the engram and the float row.

## CorpusIngestQueue.swift

This file holds the asynchronous ingest pipeline. It arrives as an
extension on `Corpus`. Three pieces make up the pipeline: a durable
queue, a background drain worker, and a single-drainer lease. It exists
so CorpusKit is a complete standalone substrate. Any consumer gets
queued, multi-core encoding with no orchestrator.

`mountIngestQueue()` picks the backend by estate durability. A SQLite
estate gets a sibling `queue.sqlite` file, derived deterministically
from the estate configuration. That file is encrypted with the same key
as the estate. It replaces an earlier plaintext directory queue, which
was a real security hole beside an encrypted estate. An in-memory estate
gets a transient store instead, under a fixed constant UUID, which
avoids random-id nondeterminism. Since the physical queue can
carry other streams, each operation here is scoped to the `"encode"`
stream. An unscoped wait would deadlock on jobs this drainer never
claims.

The drain loop coordinates through a `DrainLease`. Only one live drainer
exists per estate, with crash recovery on first acquisition that resets
orphaned in-flight jobs. That recovery is safe precisely since the
lease guarantees no other live drainer holds them. A losing process
becomes a warm standby instead. It re-checks each three seconds,
bounded by the lease's fifteen-second staleness window. Each drain pass
claims the whole available batch. It decodes jobs: undecodable ones are
terminally blocked, and empty ones complete right away. It runs
`ingestBatch` once for the whole batch, and it retires the batch in one
bulk reply. While passes keep draining jobs, the loop spins without
sleeping. It defers the vector-index publish until the burst ends, so
the kit runs one index rebuild per burst instead of one per pass. That
turns a quadratic bulk import linear. When idle, the loop sleeps fifteen
milliseconds, the near-realtime latency floor. A failing item retries in
place, up to eight attempts, before a terminal blocked reply. In-place
retry is sound only since ingest is idempotent.

`enqueueIngest(_:sourceID:now:)` and `enqueueIngestBatch(_:)` stamp jobs
with caller-supplied instants, never the wall clock. The batch variant
wraps all inserts in one transaction, which removed the last full-core
bottleneck of bulk imports on encrypted SQLite.
`awaitIngestDrain(timeout:)` is the barrier importers use to know writes
are searchable. `setOnEncoded(_:)` installs the one callback CorpusKit
ever makes toward an orchestrator. The `IngestJob` wire format's JSON
field names form a pinned cross-port contract with the Rust twin.

## BasisCodec.swift

This file holds the shared binary codec each trainable provider uses
to serialize bases and counts.

The byte layout is the cross-port contract. The same trained state must
serialize to the same bytes on Swift and on Rust, which rules out JSON.
Float formatting, key order, and whitespace all differ across
ecosystems. The rules stay fixed: everything is little-endian. Floats
write as raw IEEE-754 bit patterns, so negative zero and NaN round-trip
exactly. Strings are length-prefixed UTF-8. Maps write with keys in
ascending raw-byte order. That byte-order sort exists since Swift's
default string comparison is Unicode-canonical, while Rust's is byte
order. The writer compares raw UTF-8 to match Rust. Each blob is framed
with a four-byte magic tag and a format version byte, currently 1.

`BasisWriter` is an append-only cursor with typed write methods.
`BasisReader` is a bounds-checked sequential reader. Its
`expectMagic(_:)` and `expectVersion(_:)` methods reject wrong-provider
or future-format blobs with `CorpusKitError.decodingFailure`, never a
crash and never a silent misparse. Both types are value types with no
shared state.

## DeterministicTokenizer.swift

This file holds `DeterministicTokenizer`, the model-agnostic stand-in
tokenizer that ships as the version 1.0 default for the named neural
providers.

It is a hash, not a vocabulary. Words split by the canonical keyword
rules first. Each word then folds through FNV-1a into an id in the range
two through the vocabulary size. Ids zero and one are reserved
sentinels, for padding and unknown tokens. Empty input returns a single
pad token, never an empty array. Since both legs fold through the same
hash, conformance harnesses get the same ids for the same input. The
defaults match the BERT family. The vocabulary size is 30522. The token
maximum is 128. One caveat matters most: feeding these ids into a real
embedding model produces garbage, since they carry no relation to the
model's true vocabulary. Real WordPiece and SentencePiece tokenizers
arrive with the version 1.1 model-bundle mission.

## TermDocumentCounts.swift

This file holds `TermDocumentCounts`, the shared count builder that
feeds the LSA and NMF providers.

It owns three things: the vocabulary, per-document term frequencies, and
per-term document frequencies. The vocabulary builds in encounter order.
A term's column index is fixed by the first document that mentions it,
which keeps matrix columns stable for a fixed document sequence. The
builder does not own weighting or factoring, on purpose. Those
belong to the consuming providers, which weight the same counts
differently. `addDocument(_:)` is the full training path.
`addDocumentForCountsAnchor(_:)` is the lightweight incremental path
instead. It grows the vocabulary and the document count, but it keeps no
frequencies, since the heavy inputs get re-derived by re-tokenizing the
corpus at retrain time. That bounds maintained state to the vocabulary's
size. The restored-vocabulary initializer likewise seeds a deserialized
provider with truthful metadata and empty frequency rows. The builder is
not thread-safe. All writes must finish before any reads.

## ReducedVocab.swift

This file holds the shared vocabulary-reduction step for the dense
factorizations, LSA and NMF.

A dense matrix over tens of thousands of terms is unfactorizable on a
device. The code comment estimates ten-to-the-fifteenth operations for
that case. The fix is to keep only the most informative columns.
Below the cap, 512 by default, the function is a strict no-op. Small
estates and each conformance fixture behave just as before reduction
existed. Above the cap, the function drops terms seen in only one
document, since those are pure noise. It ranks the rest by document
frequency descending, since terms that co-occur across many documents
carry the latent structure a factorization can find. It breaks ties by
raw UTF-8 byte order, to match Rust's string ordering, so both legs
select the same vocabularies. The selection is shared rather than
per-provider, since informativeness is a corpus property. It stays the
same for both factorizations. `ReducedVocabulary` freezes the kept
terms, the projection map, and the row-remapping table.

## RandomIndexingProvider.swift

This file holds `RandomIndexingProvider`, the first honest
distributional signal in the dense lane.

Random Indexing gives each term a deterministic sparse "index vector":
2,048 dimensions, with exactly ten nonzero entries of plus or minus one.
The generator seeds a counter-based random stream from the FNV hash of
the lowercased term. It draws exactly twenty values: ten positions, then
ten signs. Collisions resolve last-wins rather than by rejection, so the
draw count stays constant and the Swift and Rust streams stay aligned. A
term's meaning is then learned by addition. A window of four slides over
the training text, and each term builds up the index vectors of its
neighbors into a context vector. Terms that keep similar company
converge. This is genuine co-occurrence semantics at almost no
computational cost, and the accumulation is incremental by construction.

Embedding text sums the context vectors of its in-vocabulary terms and
normalizes to unit length. The float lane is honest about failure. An
untrained provider opts out with an empty vector. A trained provider
whose query is entirely out of vocabulary throws a typed vocabulary-miss
error instead, and the facade maps that error to the right dark-lane
outcome. The basis blob carries the magic tag `RIB1`. It is the whole
vocabulary map, since Random Indexing has no separate finalize step.
The counts blob carries the same payload, under the distinct magic tag
`RICT`. A counts row can so never be misread as a basis row. The
projection seed spells
`RI_V1_MX`.

## PpmiProvider.swift

This file holds `PpmiProvider`, the co-occurrence signal weighted by
positive pointwise mutual information.

PPMI asks of each word pair whether the words co-occur more than chance
would predict. The score is the logarithm of the observed co-occurrence
probability over the product of the individual probabilities, floored at
zero. Frequent-but-meaningless neighbors, the "the" problem, score near
zero. Genuinely associated pairs keep full weight instead. Training runs
in two phases. Phase one counts: a sliding window of four builds up
pair counts, term counts, and totals, additively across calls. Phase
two, `finalize()`, converts counts into context vectors. Each term's
vector becomes the PPMI-weighted sum of its neighbors' Random Indexing
index vectors. That sum lives in the same 2048-dimension space, with
the same generator. The two signals stay directly comparable, as a
result. The file warns outright that this differs mathematically from
unweighted Random
Indexing, and it must never be "simplified" into it.

Embedding, opt-out, and vocabulary-miss behavior all mirror the Random
Indexing provider. The basis blob, `PPB1`, persists only the derived
vectors. The count tables are training scratch instead. The counts
blob, `PPMC`, persists the raw additive state. That state includes the
nested pair-count map. The map serializes with byte-sorted keys. A
retrain can then resume counting, without re-tokenizing. Stored vectors stay unnormalized.
Normalization happens at embed time instead, so tests can inspect raw
sums. The projection seed spells `PPMI_V1M`. It must never equal the
Random Indexing seed, since the seed partitions vector storage by
model.

## LsaProvider.swift

This file holds `LsaProvider`, the latent semantic analysis signal.

LSA finds hidden topic structure by factorizing a term-document matrix,
with the singular value decomposition, or SVD. Words that appear in
similar documents land near each other in the latent space, so synonyms
co-locate even when they never co-occur. Training builds on the shared
count builder. It reduces the vocabulary first, the ADR-022 step. It
weights each cell by log term frequency times smoothed IDF, with
add-one smoothing on both sides, so an unseen query term still gets a
positive weight. It decomposes with `JacobiSVD` from SubstrateML, at
rank 64. The Jacobi method is chosen since its sweep count is pinned
at exactly thirty sweeps. That makes the factorization a fixed
computation, rather than a convergence-dependent one. Changing the sweep
count would invalidate each conformance vector. Wide matrices transpose
for the decomposition, and the factors swap back afterward, so
downstream code always sees one orientation.

New text embeds through the classical fold-in formula. The query's
weighted term vector projects through the factor matrix. It scales by
the inverse singular values, and it skips values below a small floor.
Training documents use the exact projection instead. The dark-lane
contract matches the other trainable providers. The basis blob carries
the magic tag `LSB1`. It persists three things: the reduced vocabulary,
the IDF weights, and the raw factor matrices. This form is port-neutral,
so each leg re-derives its own document vectors. The counts blob
carries the magic tag `LSAC`. It persists only the vocabulary and
document-count anchors, per the re-tokenize-at-retrain decision. The
projection seed spells `LSA_V1_M`.

## NmfProvider.swift

This file holds `NmfProvider`, the non-negative matrix factorization
signal.

NMF factorizes the term-document matrix into two non-negative factors,
so each latent dimension reads as an additive combination of terms:
parts, not contrasts. The matrix orients terms-by-documents, weighted by
log term frequency only. NMF requires non-negative input, and its
update steps are most stable with uniformly scaled entries, so IDF is
left out on purpose. Factorization reuses SubstrateML's alternating
least squares, at rank 32, with two pinned determinism devices. The
convergence tolerance sets to zero, which forces exactly one hundred
iterations on each platform, regardless of floating-point convergence
behavior. Factor initialization seeds with a fixed constant. Document
embeddings are the normalized factor columns, precomputed at finalize.

Queries fold in through a pseudo-inverse projection onto each factor
column, with a small epsilon guarding the denominator, then normalize.
Serialization follows the family pattern. The basis blob, `NMB1`,
carries configuration, the reduced vocabulary, and both raw factor
matrices. The counts blob, `NMFC`, carries anchors only. The projection
seed spells `NMF_V1_M`. The dark-lane contract matches the other
trainable providers.

## FdcProvider.swift

This file holds `FDCProvider`, the taxonomic co-classification
signal. It is the one honest signal that needs no training at all.

The provider classifies text with LatticeLib's FDC engine. That engine
lives in the moot-semantics repository, and it is not reimplemented
here. `FDC.encode` returns a lattice code, or nothing. `FDC.ancestors`
returns the code's chain up the classification hierarchy. The provider
turns that chain into geometry. Each node in the chain gets a
deterministic 256-dimension unit vector, generated from the hash of its
code string. The vectors sum with weight one over depth-plus-one. Shared
roots give any two texts a similarity floor, and shared deep ancestors
add to it. Texts filed near each other in the subject hierarchy then
embed near each other, regardless of surface wording. The node-vector
generator runs a different pipeline from Random Indexing's, on purpose.
The two input domains cannot collide.

Honesty governs the edges. Text the classifier cannot resolve returns
UNRESOLVED. The provider then opts out, with an empty float vector and a
zero engram, since unclassifiable text must not contribute false
similarity. The provider is stateless and `Sendable`. Its determinism
rests on LatticeLib's own agreement property. The projection seed spells
`FDC_V1_P`. The free function `fdcNodeVector(code:)` is public, so
conformance tests can pin individual node vectors.

## DefaultEnsemble.swift

This file holds `CorpusEnsemble.defaultEnsemble()`, the single source
of truth for the production recall ensemble.

The factory returns the five honest signals in fixed order: Random
Indexing, PPMI, LSA, NMF, and FDC. Order is load-bearing here, since
the first element becomes the corpus's default signal. It is a function
rather than a shared constant, since the four trainable providers are
reference types holding mutable trained state. A shared array would
alias one provider instance across each estate. Fresh construction
gives each estate its own untrained providers instead. The corpus
lifecycle then trains and persists those providers under their own
model identifiers. The factory lives in the providers target, since
it names concrete types, and the core's `EmbeddingModel` enum never
does.

## MiniLMTextProvider.swift, MPNetTextProvider.swift, EmbeddingGemmaProvider.swift

These three files hold the named neural embedding providers.
MiniLM-L6 v2 uses 384 dimensions. mpnet-base-v2 uses 768. EmbeddingGemma
300M also uses 768. They share one structure, so this section describes
them together. Each fact below holds for all three, unless the text
says otherwise.

Each provider runs the same pipeline. It tokenizes the input. It runs
the host-injected inference closure to get a pooled float vector. It
projects that vector to a 256-bit engram with `FloatSimHash` from
SubstrateML. An engram is a fixed-size binary fingerprint. Similar
vectors produce similar engrams, so the vector lane can compare chunks
by fast Hamming distance. The inference closure is the doctrine's model
seam. The provider never holds a CoreML model, which keeps it testable
without a model bundle. Model loading differs on iOS, macOS, and CI.
The host owns that step. It composes the model once, at startup. Each
provider pins its own projection seed. The seed keeps fingerprints
model-tagged, in ASCII: `MINLM_v1`, `MPNET_v1`, and `EMBGM_v1`. Seeds
partition vector storage by model, and they must never change or
collide.

Each provider exposes three surfaces. `embed` returns the engram.
`embedFloat` returns the raw pooled vector, for the true-cosine float
lane. `embedPair` runs inference once and derives both results, which
halves the cost of ingest, since ingest needs both per chunk. Empty
input short-circuits to a zero engram and an empty vector. All three
default to the `DeterministicTokenizer` stand-in. EmbeddingGemma's copy
is sized for its own SentencePiece vocabulary. That vocabulary holds
256000 terms, with a context of 2048 tokens. Until real tokenizers ship, embedding values stay a
property of the host's model bundle. What the kit itself owns, the
token-to-engram pipeline given a pooled vector, stays bit-identical
across ports.

## NLEmbeddingProvider.swift

This file holds `NLEmbeddingProvider`, the Apple sentence-embedding
signal built on the operating system's bundled `NLEmbedding` model.

It is the cheap, immediate Apple-native lane. It needs no download. It
needs no CoreML seam. It needs no training either, since the OS
framework is the model itself. There is nothing for a host to inject.

The provider looks up the OS sentence model for its configured
language. It embeds the text. It casts the result to floats. It then
normalizes the vector to unit length, using the substrate's canonical
vector operations. This keeps the float lane's cosine on a unit sphere,
like each other provider.

Absence stays graceful. Two cases return an empty vector: no model
exists for the language, or the model cannot embed the given text.
Either case counts as a typed lane opt-out, never an error.

The projection seed spells `APNLEMB1`. It is defined here, rather than
in the doctrine's CoreML seed table, but it still follows the same
never-collide rule. The file compiles only where NaturalLanguage
exists. There is no Rust counterpart here. ADR-019 records this as a
sanctioned Swift-only divergence. Recall fusion simply handles the
absent lane elsewhere.

## NLContextualEmbeddingProvider.swift

This file holds `NLContextualEmbeddingProvider`, the higher-quality
Apple lane built on the on-device `NLContextualEmbedding` transformer.

The transformer needs a per-language downloadable asset that may be
absent. The file's central rule holds that an embed call must never
trigger a network fetch as a side effect. The provider checks asset
availability with a free, synchronous call. It opts out with an empty
vector when the asset is missing. Prefetching assets is the host
application's job, done before the provider gets constructed. When
assets exist, the provider runs the transformer. It mean-pools the
per-token vectors, the conventional strategy for a transformer without a
sentence-pooling head, with a defensive dimension guard that skips
malformed token vectors. Then it normalizes the result. Each failure
mode collapses to the empty-vector opt-out, since a missing asset is
an expected operational state. The projection seed spells `APNLCTX1`,
distinct from the sentence provider's, so the two lanes key to separate
storage partitions. This provider is Apple-only, with no Rust
counterpart, per ADR-019.

## Rust Port and Conformance

The `rust/` directory mirrors the core target. The `rust-providers/`
directory mirrors the providers target, matching the two-target split,
so core consumers never depend on provider code. The core crate ships
the same chunk, chunker, tokenizer, store, engine, hybrid-recall,
ingest-queue, and `Corpus` types. Concurrency uses locks where Swift uses
actors. The queue and the inverted-index store own private connections
there, where Swift instead shares the estate's storage. The providers
crate ships the deterministic tokenizer and the shared
count-and-vocabulary-reduction code. It also ships the basis codec, the
five honest signals, and the three named neural providers. All of them
share the same host-injected inference seam. Neither crate bundles
model weights.

Three fixture families gate the ports. The shared canonical vectors in
`Tests/SharedVectors/` cover BM25 impacts, per-provider embedding
vectors, and serialized basis blobs. Both legs read them, and both must
reproduce them byte for byte. The Rust test suite also pins
basis serialization byte-for-byte for all four trainable providers, and
it round-trips the counts seam. The two Apple NaturalLanguage providers
are the one sanctioned divergence. They exist only in Swift, and the
parity baseline is the deterministic and classical providers. When you
change either leg, run both test suites. The fixtures are the contract.

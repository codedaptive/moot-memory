---
doc: AGENT_MAP
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

# AGENT_MAP : CorpusKit

PURPOSE: standalone on-device RAG kit. Text → chunks (content-addressed UUID) → BundleStore (PersistenceKit) + persistent BM25 inverted index + per-provider vectors (VectorKit) → hybrid recall (Hamming kNN + BM25, weighted RRF) → [ScoredChunk]. Ships two targets: CorpusKit (core: stores, engines, protocols) and CorpusKitProviders (concrete embedding providers/tokenizers). Default production ensemble = five honest deterministic signals (RI/PPMI/LSA/NMF/FDC).

DEPS: CorpusKit imports SubstrateTypes, SubstrateLib (MerkleHash), SubstrateML, EngramLib, EideticLib (sentence segmentation), IntellectusLib (telemetry, off-by-default), PersistenceKit (+InMemory, +SQLite), ConvergenceKit (manifest only), VectorKit, QueueKit, Crypto. CorpusKitProviders additionally imports SubstrateKernel (FloatVecOps), LatticeLib (FDC runtime; FDC math NOT reimplemented). Imported by: GeniusLocusKit (orchestrator tier). Rust ports: `rust/` (core, crate corpus-kit) + `rust-providers/` (crate corpus-kit-providers); shared fixtures `Tests/SharedVectors/*.json` read by BOTH legs gate bit-identity. NL providers are Swift-only (ADR-019, no Rust twin).

ENTRY POINTS (most callers need only these):
- CorpusKit.swift:702 `Corpus.init(storage:models:)` : open estate corpus; `models[0]` = default signal (:674 single-model convenience)
- CorpusKit.swift:1001 `Corpus.ingest(_ text:sourceID:now:)` : synchronous chunk+index+embed
- CorpusIngestQueue.swift:157 `Corpus.enqueueIngest(_:sourceID:now:)` : async queued ingest (production path)
- CorpusKit.swift:1629 `Corpus.recall(_ query:limit:now:) -> [ScoredChunk]` : hybrid RRF recall on default signal
- DefaultEnsemble.swift:62 `CorpusEnsemble.defaultEnsemble() -> [EmbeddingModel]` : the five production signals, fresh per estate

## Symbol Table

### Facade : CorpusKit.swift
- :48 `enum FloatLaneOutcome` : `.hits`/`.unavailableProviderOptOut`/`.unavailableNoVocabHit`/`.unavailableNoFloatRows`/`.emptyQuery`/`.storeError`; dark lanes are typed outcomes, NEVER errors
- :134 `enum EmbeddingModel` : `.deterministic` (:147, seed 0xC05B_D15C_A15D_1B00, federation baseline), `.miniLM/.mpNet/.embeddingGemma(inference:)` (:157/:167/:177, host closure), `.randomIndexing/.ppmi/.lsa/.nmf(provider:)` (:194–:233, trainable), `.fdc(provider:)` (:254, stateless), `.nlEmbedding/.nlContextualEmbedding` (:275/:293, Apple-only); `.default = .deterministic` (:297)
- :338 `EmbeddingModel.isTrainable` : true iff carried provider conforms to TrainableEmbeddingBasis
- :362 `EmbeddingModel.reconstruct(from: Data)` : routes blob to concrete type; throws `.notTrainable`
- :417 `enum EncodeSpeed` : `.foreground` (all cores) / `.background` (cores/4, floor 1)
- :426 `public actor Corpus` : composition root; sealed-vector principle (no VectorKit type in public API except :649 `sharedVectorStore`)
- :491 `setEncodeSpeed(_:)`; :641 `onEncoded` callback (set via CorpusIngestQueue.swift:134 `setOnEncoded`)
- :1001 `ingest(_:sourceID:now:)` : idempotent; re-ingest clears tombstone; first-ingest auto-train (gate = no persisted basis, NOT factory-blob presence)
- :1181 `ingestBatch(_:)` : identical output to per-item; commit windows 512 items / 4096 rows (:436–:437); slice-parallel embed; batch-aware first-basis bootstrap (train once on full batch, never per item)
- :1416 `maintainedVocabAnchor() -> Int` : governor's vocab-growth retrain trigger read
- :1488 `reindex(now:)` : THE explicit retrain: reconstruct-fresh → trainOnCorpus(active chunks) → upsert basis → re-embed all under every slot; excludes removed sources
- :1629 `recall(_:limit:now:)`; :1657 `remove(sourceID:)` (BM25 rows + ALL models' vectors + tombstone; chunks kept); :1711 `expunge(sourceID:)` (scrubText FIRST, then remove); :1738 `destroyRecallIndex()` (all derived state; chunks survive)
- :1779 `bm25TopKBySource(query:limit:)` : pure keyword lane, max-chunk-score per source, frontierK ≤ 256
- :1833 `embed(_:) -> Engram`; :1846 `modelID`; :1863 `embedFloat(_:)` (throws on provider opt-out); :2238 `supportsFloat`
- :1895 `floatNearest(query:limit:) -> FloatLaneOutcome` : never throws; sim = 1 − distance/10_000; source aggregation = MAX chunk cosine
- :2141 `floatNearestPerSignal(query:limit:)` : per-slot dense lanes in slot order, NO fusion (caller fuses); :2214 `floatFarthestPerSignal` : anti-similarity, MIN chunk cosine per source, ascending
- :2249 `count()` (excludes removed sources); :2266 `indexedSourceIDs()`; :2274/:2280 `corpusMerkleRoot(for:)`/`globalCorpusMerkleRoot()`
- :2300 `EmbeddingModel.makeProvider()` (private) : pinned seeds: miniLM 0x4D49_4E4C_4D5F_7631 "MINLM_v1", mpNet 0x4D50_4E45_545F_7631 "MPNET_v1", embeddingGemma 0x454D_4247_4D5F_7631 "EMBGM_v1"; model IDs corpus-deterministic-v1 / minilm-v6 / mpnet-base-v2 / embedding-gemma-300m
- :2413 `CorpusDefaultTokenizer` (internal) : FNV-1a fold, duplicated from providers to avoid circular dep; :2445 `CorpusTextProvider` (private) : tokenize→inference→FloatSimHash; :2498 `embedPair` computes pooled vector ONCE for both lanes
- Test seams (never production): :914 init(storage:provider:), :980 `_testForceFloatStoreError`, :656 `_ingestFailureHook`

### Ingest queue : CorpusIngestQueue.swift (extension Corpus)
- :63 `mountIngestQueue()` : idempotent; SQLite estate → encrypted sibling `queue.sqlite` via `EstateConfiguration.queueSibling` (ADR-021 D7/T4; replaced plaintext maildir hole); InMemory estate → fixed :486 `ingestQueueStoreID` (no UUID() nondeterminism)
- :120 `dropIngestQueue()`; :134 `setOnEncoded(_:)` : the ONLY CorpusKit→orchestrator callback
- :157 `enqueueIngest(...)`; :179 `enqueueIngestBatch(...)` : one transaction for all jobs (bulk-import bottleneck fix; caller bounds batch size)
- :205 `awaitIngestDrain(timeout: 30s)` : barrier: drained AND vector index republished; throws drainTimeout
- :229 `ingestQueueDepth() -> (pending, inFlight)` : pending IS stream-scoped, inFlight is NOT (all streams)
- :248 `drainIngestQueueOnce()` : claims whole batch; undecodable → `.blocked` (terminal), empty text → `.done`; batch `ingestBatch` + bulk session reply; falls back serial on batch throw
- :341 `runIngestDrainLoop` (private) : DrainLease single drainer; first-acquire crash recovery `reclaimInFlight`; standby poll 3 s; lease TTL 15 s (failover ≈ 15–18 s, not instant); spin-while-draining, 15 ms idle sleep; vector index published once per burst (O(N) not O(N²))
- :430 `ingestOneAndReply` (private) : retry in place ≤ :476 `ingestMaxAttempts = 8`, then `.blocked`; sound ONLY because ingest is idempotent
- :480 `encodeStreamID = "encode"` : EVERY queue op must be scoped to it (shared queue.sqlite may carry other streams; unscoped awaitDrain deadlocks)
- :515 `IngestJob` : wire fields `sourceID`/`text`/`capturedAtISO8601` = pinned cross-port JSON contract; :551 `toJob`, :563 `from(job:)`

### Chunks : Chunk.swift / Chunker.swift
- Chunk.swift:35 `struct Chunk: Sendable, Equatable, Codable` : immutable, content-addressed
- Chunk.swift:68 content-addressed init (normal path); :90 explicit-id init (reconstruction; caller must guarantee id matches content)
- Chunk.swift:129 `Chunk.deriveID(sourceID:startOffset:text:)` : RFC 4122 v5 (SHA-1) over fields joined by \u{1F}; :114 `namespaceBytes` PERMANENT (change re-keys fleet + breaks vector join)
- Chunk.swift:149 `struct ScoredChunk` : chunk + score/vectorScore/keywordScore (per-lane preserved)
- Chunker.swift:28 `ChunkerConfiguration` : targetChars 800 / overlapChars 100 / respectSentences true; init clamps (overlap < target)
- Chunker.swift:49 `Chunker.chunk(text:sourceID:configuration:hlcGenerator:)` : EideticLib.sentences segmentation; greedy fill + tail overlap; hlcGenerator `inout`, stamps emission order; offsets are Character counts

### Tokenization : Tokenizer.swift
- :10 `protocol Tokenizer: Sendable` : vocabID/maxTokens/padTokenID/unknownTokenID; :27 `tokenize(_:) -> [Int32]` (truncation is implementer's job); :33 `keywordTokens(_:)` (default :62)
- :44 `defaultKeywordTokens(_:)` : lowercase + alphabetic/ASCII-digit runs; THE single keyword tokenizer for BM25 + all distributional providers; parity-critical with Rust; overriding keywordTokens breaks the guarantee (convention, not compiler)

### Errors : CorpusKitError.swift
- :5 `enum CorpusKitError: Error, Sendable, Equatable` : encodingFailure/decodingFailure/tokenizerUnavailable/modelUnavailable/embeddingFailed/storeUnavailable/:18 notTrainable; Equatable on exact message strings

### Chunk store : BundleStore.swift
- :94 `public actor BundleStore`; :139 `schemaDeclaration` v3 : `chunks` (10 cols incl. content_hash BLOB, ext JSON) + `corpus_metadata`; indices source_id, hlc
- :199 `init(storage:dirtyChainSink:)` : wraps HashingRowStore (MerkleHash.leaf per insert); :70 `ParentChainCache` bridges sync hash callback
- :268 `insert(_:)` : idempotent (duplicateKey = no-op, first write wins); RETURNS ONLY NEWLY-INSERTED chunks : derived-state callers must fold the returned subset, never the input
- :348 `get(id:asOf:)`; :361 `getMany(ids:asOf:)`; :375 `chunksForSource(_:asOf:)` (start_offset ASC); :399 `allSourceIDs` (full scan, maintenance only); :427 `chunkSourcePairs()` (body-free warm-load projection); :457 `count(asOf:)` : asOf accepted but IGNORED; :461 `allChunks(asOf:)` (hlc ASC)
- :500 `scrubText(sourceID:)` : hard-delete text zeroing via direct UPDATE (why schema appendOnly: false); leaves content_hash stale intentionally
- :564 `corpusMerkleRoot(for:)`; :583 `globalCorpusMerkleRoot()` : corpus/root UUIDs derived from fixed SHA-256 namespace strings (cross-port)
- :632 `decodeChunk(_:)` : MUST accept both TypedValue forms (SQLite primitive .text/.int AND InMemory semantic .uuid/.hlc); historical bug: semantic-only decoder dropped all chunks on reopen; InMemory-only tests cannot catch regressions here

### Tombstones : RemovedSourceStore.swift
- :44 `public actor RemovedSourceStore`; :52 schemaDeclaration (own kitID "CorpusKitRemovedSources")
- :76 `markRemoved(_:now:)` (idempotent upsert; row presence IS the state : no Bool column); :90 `clearRemoved(_:)` (re-ingest = the undo); :99 `removedIDs() -> Set<String>` : EVERY rebuild path (reindex, first-ingest train, count) must subtract this set (unenforced convention; resurrection bug class); :118 `deleteAll()`

### Provider counts : CorpusProviderCountsStore.swift
- :71 `PersistedCounts` (modelID/modelVersion/counts blob/documentCount/vocabSize/updatedAt); :101 `CountsGrowthAnchor` (cheap pair, no blob)
- :112 `public actor CorpusProviderCountsStore`; :121 schema (kitID "CorpusKitCounts", PK (model_id, model_version), ext slot)
- :156 `upsert(_:)` : full-row replace (caller folds blob first, no atomic increment); :173 `load(...)`; :191 `growthAnchor(...)` : the staleness-check read; :210 `deleteAll()`
- STATUS: "HALF A" : counts persisted/restored, but `Corpus.reindex` still retrains from raw chunk text; counts-backed retrain + vector re-projection (HALF B) not wired

### Sync : SyncManifest.swift
- :11 `enum CorpusKitSync`; :17 `manifest(zoneIdentifier:)` : chunks table, bidirectional, PK id, conflictPolicy `.appendOnly` (safe because content-addressed); kitID "CorpusKit", schemaVersion 1. Declarative only. Sync-layer appendOnly ≠ BundleStore schema `appendOnly: false` : different systems, same word

### Sparse engine : Engine/SparseTypes.swift, Engine/BM25Weighting.swift, Engine/InvertedIndex.swift
- SparseTypes.swift:39 `typealias LaneTag = VectorKit.LaneTag` (single owner, avoids ambiguity); :56 `ImpactPosting` (impact Int32, quantized ONCE at build); :94 `SparseHit` (impact Float = int/100); :136 `FusedHit` (fusedScore + perLane raw scores; absent key = no hit in lane)
- BM25Weighting.swift:29 `BM25Parameters` : k1 1.5 / b 0.75 pinned defaults; :50 `quantizeImpact(_:)` : round HALF-TO-EVEN × 100 (Swift default rounding differs at .5 : do not "simplify"); :82 `buildTermIDMap` (sorted term-id assignment); :110 `build(termFreqs:docLengths:parameters:)` : float BM25 math exactly once; IDF = ln((N−df+0.5)/(df+0.5)+1); :164 `queryPairs` : OOV dropped, duplicates deduped, weight = 100
- InvertedIndex.swift:37 `invertedIndexQuantScale = 100`; :42 `invertedIndexBlockSize = 128` (pinned for conformance traces); :116 `struct InvertedIndex: Sendable` : immutable after init (init sorts postings itemID ASC); :195 `enum Algorithm` .wand/.blockMaxWand (result-identical); :211 `topK(query:k:algorithm:)` : EXACT top-k, integer-only path, tie-break smaller itemID wins; :246 `exhaustiveScan(query:k:)` : DAAT conformance oracle, not production. Item IDs compare as STRINGS (uuidString lexicographic ≠ numeric UUID order, but consistent cross-port)

### Persistent keyword index : Engine/InvertedIndexStore.swift
- :47 `public actor InvertedIndexStore`; :55 schemaDeclaration (kitID "InvertedIndexStore", tables iix_termfreqs/iix_doclens : RAW statistics only, weighted index derived+cached, so k1/b changes need no migration); :105 init (storage pre-opened/migrated); :114 `open()` (load mirrors, O(terms+docs), no chunk bodies)
- :159 `index(itemID:tokens:now:)` : atomic replace, idempotent; empty tokens = removal; `now` unused (determinism discipline); :199 `remove(itemID:)`; :230 `buildIndex(parameters:)` (cached; invalidated per write); :253 `topK(queryTerms:k:parameters:algorithm:)`; :273 `deleteAll()`; :295 `documentCount`
- Rust twin owns a PRIVATE connection with begin/commit/rollback_batch; Swift shares estate storage : hence Corpus-managed transaction windows in ingestBatch

### Legacy keyword index : BM25Index.swift
- :34 `public actor BM25Index` : in-memory, Chunk/UUID-typed; NO LONGER used by Corpus (kept public for external callers); :49 init(tokenizer:parameters:); :58 `index(_ chunks:)`; :78 `remove(_:)`; :95 `documentCount()`; :107 `topK(_ k:for tokens:)` : pre-tokenized input; tie-break uuidString ASC

### Fusion : Engine/Fusion.swift
- :48 `enum Fusion`; :74 `fuse(rankedLists:laneScores:weights:rrfK: 60)` : fusedScore = Σ weight·1/(rrfK+rank), rank 1-based; per-lane dedup (best rank only); precondition rrfK > 0; sort fusedScore DESC, itemID ASC; :164 `fuse(scoredLists:weights:rrfK:)` : position = rank, CALLER must pre-sort
- MMR: `mmrLambda` exists only as a HybridRecallConfiguration field : NOT implemented anywhere; do not document MMR as active

### Hybrid recall : HybridRecall.swift
- :33 `HybridRecallConfiguration` : vectorWeight 0.6 / keywordWeight 0.4 / rrfK 60 / mmrLambda nil (unread); :52 `enum HybridRecall`
- :84 `recall(probe:query:modelID:limit:vectorStore:invertedIndex:bundleStore:configuration:)` : candidateK = max(limit×4, 32) per lane; vector kNN filtered to modelID; UUID hits CANONICALIZED via UUID(uuidString:).uuidString (P3-secfix: lowercase Rust-written ids must fuse with uppercase Swift ids); vectorScore = Hamming (0 = best, kept), keywordScore 0 → nil (BM25 real matches strictly positive); unhydratable ids silently dropped; telemetry post-hoc (corpuskit.recall.*)

### Trainable-basis seam : TrainableEmbeddingBasis.swift, BasisStore.swift
- TrainableEmbeddingBasis.swift:50 `protocol TrainableEmbeddingBasis: AnyObject, Sendable` : conformers: RI/PPMI/LSA/NMF only
- :70 `trainOnCorpus(texts:)` : ADDITIVE (never retrain a live provider : reconstruct fresh first); :78 `serializeBasis()`; :98 `reconstructBasis(from:)` : INSTANCE method (type-erased witness routes to concrete init(deserializing:)); round-trip law = identical embeddings; throws decodingFailure, never crashes
- :132 `addToCounts(text:)` / :141 `serializeCounts()` / :149 `restoreCounts(from:)` / :154 `countsVocabularySize` : P3 maintained-counts seam, batch-boundary snapshots; infrastructure only (reindex still trains from text)
- No wall-clock reads anywhere in training : pure function of (texts, seeds)
- Rust divergence: EmbeddingProvider is a supertrait there (no trait cross-cast)
- BasisStore.swift:67 `PersistedBasis`; :98 `public actor BasisStore`; :112 schema v2 (`corpus_provider_basis`, PK (model_id, model_version), trained_at TEXT ISO8601, trained_chunk_count anchor, ext slot); :151 `upsert(_:)` (in-place, one row per key); :177 `load(modelID:modelVersion:)`; :195 `deleteAll()`; decoder accepts BOTH TypedValue timestamp forms (same reopen-bug class as BundleStore)

### Basis codec : BasisCodec.swift (CorpusKitProviders)
- :43 `basisFormatVersion: UInt8 = 1`; :52 `struct BasisWriter` : LE only; f32 as bitPattern (:94); strings u32-len UTF-8 (:99); maps byte-sorted keys (:122/:136 via :148 lhsLess raw-UTF-8 compare : matches Rust str Ord, NOT Swift String <)
- :168 `struct BasisReader` : :192 `expectMagic` / :206 `expectVersion` reject wrong/future blobs with decodingFailure; all reads bounds-checked. Frame = MAGIC(4) | version(1) | payload. No nested-map primitive (PPMI inlines its own)

### Deterministic tokenizer : DeterministicTokenizer.swift
- :16 `struct DeterministicTokenizer: Tokenizer`; :23 init(vocabID "deterministic-v1", vocabSize 30522, maxTokens 128); :33 `tokenize(_:)` : FNV-1a 32 fold into [2, vocabSize); 0=pad 1=unk reserved; empty input → [pad]. NOT a model vocab : real-model output from these ids is garbage (real WordPiece/SentencePiece = v1.1 model-bundle mission)

### Shared training inputs : TermDocumentCounts.swift, ReducedVocab.swift
- TermDocumentCounts.swift:56 `struct TermDocumentCounts` : :61 vocab (ENCOUNTER-ORDER indices, cross-port byte contract), :64 tfCounts, :68 dfCounts; :110 `addDocument(_:)` (full); :156 `addDocumentForCountsAnchor(_:)` (vocab+docCount only, no TF/DF : re-tokenize-at-retrain decision); :91 init(restoredVocab:documentCount:) : TF rows exist but EMPTY after restore; NOT thread-safe
- ReducedVocab.swift:33 `defaultReducedVocabCap = 512` (ADR-022); :38 `ReducedVocabulary` (keptTerms/termToColumn/fullIndexToColumn/size); :63 `selectReducedVocabulary(vocab:dfCounts:documentCount:cap:)` : below cap = exact NO-OP (fixture compatibility); above cap: drop df<2, rank df DESC, tie-break raw-UTF-8-byte order (Rust parity); documentCount reserved/unused

### Honest signals : RandomIndexingProvider.swift, PpmiProvider.swift, LsaProvider.swift, NmfProvider.swift, FdcProvider.swift
- RandomIndexing: :89 riDimension 2048, :93 riNonzeros 10, :98 riWindow 4, :102 riProjectionSeed 0x5249_5F56_315F_4D58 "RI_V1_MX"; :121 `riIndexVector(term:)` : FNV64(lowercased) → SplitMix64, EXACTLY 20 draws (pos %2048 bias-free, sign &1), collision last-wins (constant draw count = cross-port PRNG alignment); :162 `final class RandomIndexingProvider`; :206 `train(terms:window:)` additive; :249 embed / :268 embedFloat / :308 embedPair; :377 serializeBasis "RIB1" (vocab IS the basis, no finalize); :452 serializeCounts "RICT" (same payload, distinct magic on purpose)
- PPMI: :107/:111/:115 same D/K/window as RI (shared index space); :120 ppmiProjectionSeed 0x5050_4D49_5F56_314D "PPMI_V1M"; :153 `final class PpmiProvider`; :226 `train` (counts); :282 `finalize()` : ppmi(t,c)=max(0, ln P(t,c) − ln P(t) − ln P(c)); contextVec = Σ ppmi·riIndexVector(c); idempotent; :346 embed/:365 embedFloat/:397 embedPair; :464 basis "PPB1" (derived vectors only, unnormalized in store); :521 counts "PPMC" (raw additive state incl. nested coCount); NOT plain RI : do not reduce
- LSA: :109 lsaProjectionSeed 0x4C53415F56315F4D "LSA_V1_M"; :114 lsaDefaultRank 64; :165 svdSweeps PINNED 30 (change invalidates all conformance vectors); :148 `final class LsaProvider`; :227 `train(document:)`; :258 `finalize()` : ReducedVocab → tf=ln(1+c), idf=max(0, ln((N+1)/(df+1))) → JacobiSVD (wide matrix transposed+swapped back); query fold-in (1/σ)·Vt·q, σ<1e-9 skipped; :487 `documentEmbedding(at:)` exact U·Σ; :547 basis "LSB1" (reduced vocab + idf + RAW U/σ/Vt, port-neutral); :632 counts "LSAC" (anchors only)
- NMF: :110 nmfProjectionSeed 0x4E4D465F56315F4D "NMF_V1_M"; :114 rank 32; :119 iterations 100; :124 nmfFactorizationSeed 0xDEADBEEFCAFEBABE (pinned); :159 `final class NmfProvider`; :266 `finalize()` : V (terms×docs), log-TF NO idf, ALS with tolerance=0 → FIXED iteration count (bit-identity device); query fold-in dot(W[:,r],q)/(‖W[:,r]‖²+1e-9); :499 basis "NMB1" (W and H raw); :588 counts "NMFC" (anchors only)
- FDC: :106 fdcDimension 256; :111 fdcProjectionSeed 0x4644_435F_5631_5F50 "FDC_V1_P"; :137 `fdcNodeVector(code:)` : FNV64 → ONE SplitMix64 advance → LCG (Knuth 6364136223846793005 / 1442695040888963407) → 256 draws → l2Normalize (pipeline deliberately ≠ RI's); :242 `final class FDCProvider` : STATELESS, no training, no BasisCodec; path = FDC.ancestors + [code], node weight 1/(L+1); UNRESOLVED/empty → opt-out ([] / .zero) : honest, never guess; :284 embed/:299 embedFloat/:310 embedPair; determinism inherited from LatticeLib singletons
- Common float-lane tri-state (four trainable providers): `[]` = structural opt-out (untrained) → .unavailableProviderOptOut; throw VectorKitError.embedFloatVocabMiss = trained-but-all-OOV → .unavailableNoVocabHit; vector = signal. `embedPair` collapses the vocab-miss throw to (.zero, []) on ALL providers. Training thread-contract everywhere: Sendable class, but train/finalize must complete before concurrent embeds; read-only after

### Ensemble factory : DefaultEnsemble.swift
- :38 `enum CorpusEnsemble`; :62 `defaultEnsemble()` : [.randomIndexing, .ppmi, .lsa, .nmf, .fdc] in pinned order ([0] = default signal); FUNCTION not constant (trainable providers are reference types : sharing one array would alias trained state across estates); returned trainables are UNTRAINED

### Named neural providers : MiniLMTextProvider.swift, MPNetTextProvider.swift, EmbeddingGemmaProvider.swift
- MiniLM: :40 `struct MiniLMTextProvider: EmbeddingProvider`; :48 `inference: @Sendable ([Int32]) async throws -> [Float]` (closure-injected CoreML seam, doctrine §5); :50 init : modelID "minilm-v6", seed 0x4D49_4E4C_4D5F_7631 "MINLM_v1", 384-dim; :64 embed / :80 embedFloat / :95 embedPair (ONE inference pass : separate calls each pay inference)
- MPNet: :30 struct; :38 init : "mpnet-base-v2", seed 0x4D50_4E45_545F_7631 "MPNET_v1", 768-dim; :52/:65/:80 embed/embedFloat/embedPair
- EmbeddingGemma: :32 struct; :40 init : "embedding-gemma-300m", seed 0x454D_4247_4D5F_7631 "EMBGM_v1", 768-dim, stand-in tokenizer vocab 256_000 / maxTokens 2048 (SentencePiece-sized); :58/:71/:86
- Parity framing: embedding VALUES are a property of the host's model bundle; what is bit-identical cross-port is the kit-owned pipeline (tokens→engram, float lane given a pooled vector) and the full no-host pipeline with DeterministicTokenizer

### Apple NL providers : NLEmbeddingProvider.swift, NLContextualEmbeddingProvider.swift (Swift-only, ADR-019)
- NLEmbedding: :76 `nlEmbeddingProjectionSeed 0x4150_4E4C_454D_4231` "APNLEMB1" (outside doctrine table, same never-collide rule I-4); :113 `struct NLEmbeddingProvider: EmbeddingProvider, Sendable`; :145 init(language: .english default); :164/:180/:189 embed/embedFloat/embedPair; OS sentence model; no-model-for-language → [] opt-out; l2Normalize via FloatVecOps
- NLContextual: :79 `nlContextualEmbeddingProjectionSeed 0x4150_4E4C_4354_5831` "APNLCTX1"; :121 struct; :152 init; :170/:187/:196; checks `hasAvailableAssets` (free, sync) : NEVER triggers a network fetch; host must prefetch via requestAssets BEFORE constructing; mean-pools token vectors with per-token dimension guard; all failures → [] (asset absence = expected operational state)
- Both `#if canImport(NaturalLanguage)`; no Rust twins; vector dimension comes from the OS model (not pinned)

## INVARIANTS / GOTCHAS

- DETERMINISM DISCIPLINE: engines never read `Date()`/`UUID()`; `now` is always caller-supplied. Sanctioned exceptions: RemovedSourceStore audit stamp, telemetry timestamps. Training is a pure function of (texts, pinned seeds).
- UNIVERSAL JOIN KEY: chunk.id.uuidString == VectorKit item_id == inverted-index item_id. Swift canonical uuidString is UPPERCASE; HybridRecall re-canonicalizes vector-lane ids (P3-secfix). Universal tie-break everywhere: score DESC, then id ASC ("smaller id wins").
- PINNED CONSTANTS (change = new conformance vectors + possible fleet re-key): chunker 800/100; BM25 k1 1.5 / b 0.75; QUANT_SCALE 100 with round-HALF-TO-EVEN; BMW block 128; RRF k 60, weights 0.6/0.4; candidate over-fetch max(limit×4, 32); float-lane sim quantization ×10_000; commit windows 512/4096; ingestMaxAttempts 8; idle drain sleep 15 ms; lease TTL 15 s / standby poll 3 s; RI/PPMI D 2048 K 10 window 4; LSA rank 64 sweeps 30; NMF rank 32 iterations 100 tolerance 0 factorization seed 0xDEADBEEFCAFEBABE; reduced-vocab cap 512; Chunk.namespaceBytes; basisFormatVersion 1.
- PROJECTION SEEDS partition vector storage by model and must be unique + frozen: deterministic 0xC05B_D15C_A15D_1B00, MINLM_v1, MPNET_v1, EMBGM_v1, RI_V1_MX, PPMI_V1M, LSA_V1_M, NMF_V1_M, FDC_V1_P, APNLEMB1, APNLCTX1. All projection goes through SubstrateML.FloatSimHash : ad-hoc projections are banned from the kit graph.
- ONLY TWO TRAIN TRIGGERS: first-ingest auto-train (gate = no persisted basis) and explicit `reindex(now:)`. trainOnCorpus is ADDITIVE : always reconstruct fresh from the slot's freshBasisBlob before retraining; growth-threshold auto-retrain is future policy (anchors persisted, path unwired : "HALF A").
- TOMBSTONE CONSULTATION IS UNENFORCED: every chunk-replay path (reindex, first-ingest train, count, any future rebuild) MUST subtract RemovedSourceStore.removedIDs() or removed sources resurrect on the governor's auto-reindex.
- BundleStore.insert returns ONLY newly-inserted chunks; fold derived state over the returned subset. count(asOf:) ignores asOf.
- DUAL TypedValue DECODE (BundleStore, BasisStore): SQLite round-trips UUID/HLC/timestamp as primitives, InMemory keeps semantic forms; decoders must accept both; InMemory-only tests cannot catch the regression (proven bug class: silent total data loss on reopen).
- STREAM SCOPING: every queue op in CorpusIngestQueue uses stream "encode"; the shared queue.sqlite may carry other streams; an unscoped awaitDrain deadlocks. IngestJob JSON field names are a frozen cross-port wire contract.
- appendOnly means two different things: sync conflict policy (SyncManifest, `.appendOnly`, safe via content addressing) vs BundleStore schema flag (`appendOnly: false`, required so scrubText can UPDATE). Do not conflate.
- Chunks are immutable BY CONVENTION (no update API), not by DB trigger; edit = delete + reinsert with new id; metadata is the only legal per-chunk side-data slot (doctrine §2).
- keywordTokens override breaks the single-tokenizer guarantee shared by BM25 + distributional providers (convention, not compiler-enforced).
- MMR (`mmrLambda`) is declared but NOT implemented on any recall path. BM25Index is legacy : Corpus uses InvertedIndexStore.
- Actors everywhere (Corpus, BundleStore, InvertedIndexStore, BasisStore, CorpusProviderCountsStore, RemovedSourceStore, BM25Index): writes serialized per instance; embedding compute deliberately escapes the actor via Sendable providers for parallelism.
- Retry-in-place (8 attempts) is sound ONLY because ingest is idempotent via content-addressed ids : not a general recipe.
- Rust conformance gates: SharedVectors JSON (BM25 impacts, embedding vectors, basis blobs) read by BOTH legs; rust-providers tests pin basis serialization byte-for-byte for RI/PPMI/LSA/NMF and canonical vectors for RI/PPMI/FDC; NL providers exempt (Swift-only).

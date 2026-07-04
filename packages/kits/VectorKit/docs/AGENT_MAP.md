---
doc: AGENT_MAP
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

# AGENT_MAP — VectorKit

PURPOSE: on-device embedding generation (`EmbeddingProvider` seam) + model-tagged vector storage (`VectorStore`, PersistenceKit-backed) + dual-lane nearest-neighbour search: binary Hamming (Lane A `BruteForceIndex` oracle / Lane B `MIHIndex` sub-linear exact, promoted at `mihThreshold`) and float cosine/l2/dot (`FloatBruteForceIndex`, one index per modelID) + ColBERT MaxSim late-interaction scorer (`MaxSimScorer`, standalone, not wired into VectorStore).

DEPS: imports EngramLib (Engram type, Hamming kernel via EngramLib.distances/Session — I-7 absolute), SubstrateML (FloatSimHash.project), SubstrateTypes, PersistenceKit (Storage/RowStore/BlobStore, product "PersistenceKit"), IntellectusLib (Intellectus.report telemetry, no-op when disabled). Test target additionally depends on PersistenceKitInMemory, PersistenceKitSQLite. Imported by: CorpusKit / CorpusKitProviders (concrete text embedding providers built on FloatSimHashEmbeddingProvider), GeniusLocusKit (destroyAllVectors as part of estate teardown). Rust port in rust/ mirrors every file (vector_store.rs, engine/{brute_force,mih,float_brute_force,max_sim,resident,resident_store,key,payload,hit,metric,seam}.rs, embedding_provider.rs, simhash_embedding_provider.rs, error.rs); no shared cross-language fixture file — conformance rests on both ports implementing the documented algorithms identically (colex enumeration, sidecar byte layout, budget arithmetic). Float lane is explicitly NOT four-way bit-identical (documented, not a gap).

ENTRY POINTS (most callers need only these):
- VectorStore.swift:451 `VectorStore.addVector(itemID:engram:modelID:modelVersion:filedAt:)` — write one binary vector
- VectorStore.swift:1134 `VectorStore.findNearest(probe:modelID:limit:) -> [VectorMatch]` — binary Hamming k-NN
- VectorStore.swift:1223 `VectorStore.findNearestFloat(probe:modelID:limit:) -> [VectorMatch]` — float cosine k-NN
- FloatSimHashEmbeddingProvider.swift:63 `FloatSimHashEmbeddingProvider.embed(_:) -> Engram` — text → fingerprint via injected inference + FloatSimHash

## Symbol Table

### Module surface
- VectorKit.swift:1 — namespace/header only; no types. Consumers `import EngramLib` separately for `Engram` (not re-exported).

### Errors — VectorKitError.swift
- :5 `enum VectorKitError: Error, Sendable, Equatable` — concrete cases, never optional+log
- :9 `.embeddingFailed(String)` / :13 `.modelUnavailable(String)` / :17 `.storeUnavailable(String)` / :21 `.notFound` (reserved, unused by current API) / :27 `.invalidPayload(String)` / :31 `.decodingFailure(String)`
- :41 `.int8QuantizationPolicyUndefined(String)` — thrown on every int8 write; policy unratified (VECTORKIT_SPEC §I-4a); remove guard+case only when ratified
- :56 `.embedFloatVocabMiss(String)` — distributional-provider OOV signal, distinct from embeddingFailed

### Embedding seam — EmbeddingProvider.swift
- :15 `protocol EmbeddingProvider: Sendable` — modelID/modelVersion + embed/embedFloat/embedPair/embedBatch
- :37 `embed(_:) -> Engram` — MUST return `Engram.zero` for empty string (cross-provider contract, mirrored in Rust trait)
- :64 `embedFloat(_:) -> [Float]` — opt-in; default impl (:105) throws embeddingFailed; empty input → `[]` never zero-vector
- :81 `embedPair(_:) -> (engram, floats)` — default impl (:115): two-pass (embed then embedFloat), float opt-out swallowed to `[]`
- :93 `embedBatch(_:) -> [Engram]` — default impl (:123): sequential; override for batched inference

### Concrete provider — FloatSimHashEmbeddingProvider.swift
- :35 `struct FloatSimHashEmbeddingProvider: EmbeddingProvider` — Swift mirror of Rust vectorkit::FloatSimHashEmbeddingProvider
- :44 `projectionSeed: UInt64` — distinct seeds ⇒ distinct fingerprints for same float vector (I-4 enforced at projection layer)
- :49 `inference: @Sendable (String) async throws -> [Float]` — host-injected; kit owns no tokenizer/model
- :63 `embed(_:)` — empty-string short-circuit BEFORE inference call, then `FloatSimHash.project(vector:seed:)`
- :87 `embedFloat(_:)` — returns the SAME vector embed() projects; no double inference

### Engine foundation types (Lane F — additive-only, no local field additions)
- VectorRecordKey.swift:33 `struct VectorRecordKey: Sendable, Equatable, Hashable, Comparable` — (itemID, vectorIndex, modelID, modelVersion); ordering IS the partition/tie-break order
- VectorRecordKey.swift:87 `< (lhs:rhs:)` — lexicographic (itemID, vectorIndex, modelID, modelVersion); DO NOT reorder fields
- VectorPayload.swift:76 `enum VectorKind: UInt8` — .binary=0/.float32=1/.int8=2; ON-DISK raw values, never reorder
- VectorPayload.swift:101 `struct VectorPayload` — kind+dim+bytes+scale(int8 only, unused in prod)
- VectorPayload.swift:146 `init(engram:)` — binary, zero-copy wire bytes
- VectorPayload.swift:163 `init(floats:)` — float32, explicit little-endian serialization (byte-order portability, not native-order)
- VectorPayload.swift:187/:200 `asEngram()` / `asFloats()` — throw invalidPayload on kind/size mismatch
- VectorPayload.swift:38 `struct VectorPayloadInput` — bulk-write row bundle (itemID, vectorIndex, payload, modelID, modelVersion, filedAt)
- DenseHit.swift:45 `struct DenseHit: Sendable, Equatable` — key + rawDistance(Int32, dual-purpose: Hamming int OR Float bit pattern) + metric
- DenseHit.swift:101/:121 `hammingDistance` / `floatDistance` — typed accessors reinterpreting rawDistance
- DenseHit.swift:132 `enum LaneTag` — .binaryDense/.floatDense/.sparse/.lateInteraction (fusion/cross-package use)
- DenseMetric.swift:41 `enum FloatMetric` — .cosine/.l2/.dot; VectorKit-owned (ADR-008)
- DenseMetric.swift:58 `enum BinaryMetric` — .hamming/.jaccard (jaccard reserved, BruteForceIndex rejects it)
- DenseMetric.swift:81 `enum DenseMetric` — .binary(BinaryMetric)/.float(FloatMetric) umbrella; :91-:103 shorthand statics
- DenseIndex.swift:67 `enum SearchDirection` — .nearest/.farthest; farthest is bottom-K scan, NOT negated top-K
- DenseIndex.swift:85 `struct MetadataFilter` — modelID/modelVersion wildcard-if-nil; :109 `accepts(_:)`
- DenseIndex.swift:35 `enum IndexKind` — .bruteForce/.mih tag (nominal dispatch, not type-casting)
- DenseIndex.swift:131 `protocol DenseIndex: Sendable` — build/search/add/remove seam; BruteForceIndex is the binary oracle

### Binary engines
- BruteForceIndex.swift:47 `actor BruteForceIndex: DenseIndex` — Lane A; conformance oracle; ZERO Hamming math in file (I-7)
- BruteForceIndex.swift:101 `search(probe:metric:k:filter:)` — only .binary(.hamming); model-partition slice via O(log m) lookup; sorts (distance ASC, FULL key ASC) — NOT EngramLib.findNearest (different tie-break)
- BruteForceIndex.swift:227 `add(key:vector:)` — tombstone-then-append upsert; :276 `remove(key:)` — tombstone only, no reclaim
- BruteForceIndex.swift:305 `currentSnapshot()` — value-copy for cross-actor read (VectorStore tombstone scans)
- BruteForceIndex.swift:326/:342 `setTombstoneBit` / `buildPartitions` — shared bit-layout + partition-rebuild helpers, mirrored in ResidentArrayStore
- MIHIndex.swift:251 `actor MIHIndex: DenseIndex` — Lane B; sub-linear EXACT Hamming k-NN via Multi-Index Hashing; output MUST equal BruteForceIndex bit-for-bit (BLOCKER conformance gate, MIHIndexTests.swift)
- MIHIndex.swift:72 `enum MIHBandCount: UInt32` — {.m4,.m8,.m16,.m32} ONLY (§1.7: keeps sub_bits∈{64,32,16,8}, no word-straddle)
- MIHIndex.swift:303 `init(bandCount:maskBudget:)` — maskBudget nil ⇒ dynamic max(n, 2^20) per query
- MIHIndex.swift:351 `search(...)` → :450 `knn(...)` — progressive-radius pigeonhole expansion; stop when heap full AND worstDist ≤ r
- MIHIndex.swift:504 enumeration-budget guard — projected flip-mask count vs budget; over-budget + heap not exact ⇒ :575 `bruteScan` fallback (still exact, just O(n)); fires `.notice` log + `vectorkit.mih.enumeration_fallback` metric
- MIHIndex.swift:711 `cumulativeChoose(subBits:rho:)` — Σ C(subBits,d); saturates to Int.max on overflow; MUST match Rust saturating_add bit-for-bit
- MIHIndex.swift:764 `colexFlipMasks(subBits:maxHamming:body:)` — Gosper's-hack colex enumeration, ascending subset size then ascending mask value; canonical order, internal (test-visible)
- MIHIndex.swift:654 `extractBand(from:bandIndex:)` — canonical bit numbering (bit i → word i/64, LSB=0); word-straddle branch unreachable for allowed m

### Float engine
- FloatBruteForceIndex.swift:60 `actor FloatBruteForceIndex: DenseIndex` — Lane C/D (file header says "Lane C", VectorStore.swift comments say "Lane D" — same type, inconsistent lane label in source, not a functional issue); float32 ONLY; NOT four-way bit-identical (documented, do not "fix")
- FloatBruteForceIndex.swift:84 `build(from:)` — O(1) reference store; array IS the index
- FloatBruteForceIndex.swift:102 `search(...)` — validates probe.kind/.dim vs array stride; cosine treats zero-vector as distance 1.0 (no div-by-zero)
- FloatBruteForceIndex.swift:156 `searchFarthest(...)` — identical scan+distance as search(); only sort direction flips (:233 `rank(...direction:)`)
- FloatBruteForceIndex.swift:273 `add(key:vector:)` — FIRST add establishes stride; later mismatched byte count throws invalidPayload (prevents storage corruption)
- FloatBruteForceIndex.swift:323 `remove(key:)` — tombstone; compaction only on next build()

### Late-interaction scorer (standalone, not a DenseIndex)
- MaxSimScorer.swift:97 `struct MaxSimScorer: Sendable` — Lane E1, Exact-A exhaustive ColBERT MaxSim; conformance reference for future pruned variants
- MaxSimScorer.swift:148 `score(queryTokens:documents:k:) -> [MaxSimHit]` — Σ(256−min hamming) per query token; documents iterated in SORTED itemID order (dict order is undefined); sort (score DESC, itemID ASC); truncate to k AFTER full sort
- MaxSimScorer.swift:55 `struct MaxSimHit` — itemID + integer score [0, 256×|Q|]
- All distances via `EngramLib.Session.distances` (I-7); session built once per scorer, reused across the whole score() call

### Resident array (shared data contract, Lane F)
- ResidentVectorArray.swift:65 `struct ResidentVectorArray: Sendable` — packed fixed-stride array; kind/stride/count/storage/keys/modelPartitions/tombstones; measured 87% of pre-resident latency was fetch+decode, 0.4% kernel — this type removes the fetch+decode cost
- ResidentVectorArray.swift:135 `liveCount` — O(count/64) tombstone-bitmap walk; used for sidecar staleness (live-vs-live compare)
- ResidentVectorArray.swift:190 `partitionRange(for:)` — binary search, O(log m)
- ResidentVectorArray.swift:214/:228 `isTombstoned(_:)` / `vectorBytes(at:)` — per-slot accessors every engine scan loop uses
- ResidentVectorArray.swift:44 `struct ModelPartitionEntry` — modelID + half-open Range<Int>

### Resident array persistence — ResidentArrayStore.swift
- :116 `actor ResidentArrayStore` — owns optional `.vec` sidecar; vectors TABLE remains sole durable source; sidecar is regenerable cache only
- :97 `kVecVersion = 0x0002` — format version; adds live_count field after count (discarded on load, recomputed from tombstone bitmap — stale header value cannot corrupt results)
- :102 `kDefaultTombstoneCompactionThreshold = 0.25`
- :183 `load()` — missing/invalid sidecar ⇒ start empty, no crash
- :209 `rebuild(from:)` — full rewrite from sorted [(key,bytes)]; used on stale-sidecar detection
- :247 `append(key:bytes:)` — EAGER write (immediate sidecar rewrite)
- :277 `appendDeferred(key:bytes:)` — WRITE-BEHIND single-add path (production default via VectorStore.addPayload); sets isDirty, no disk write; caller must flush()
- :299 `appendBatch(records:)` — bulk path, ONE sidecar write per batch (not per record) — TASK #24 amortization
- :346 `flush()` — persists pending write-behind mutation; no-op if !isDirty
- :387/:420 `tombstone(key:)` (eager, writes) / `tombstoneDeferred(keys:)` (batch, no write, sets isDirty)
- :448 `compact()` — drops tombstoned slots, sorted-by-key rewrite, deterministic output
- :563/:610/:623 `writeSidecar` / `readSidecar` (mmap via .mappedIfSafe) / `parseSidecar` — every length field bounds-checked before trust; magic "VEC1" (:85 `kVecMagic`)
- On-disk layout: magic(4)|version(2)|kind(1)|stride(4)|count(4)|live_count(4)|tombstone_words(4)|tombstones|vectors|keys(variable)|partition_index(variable); ALL integers little-endian (cross-host byte-identity, arch spec §4.3)

### Storage-facing types
- StoredVector.swift:20 `struct StoredVector: Sendable, Equatable` — decoded `vectors` row; `engram` non-nil ONLY for binary kind (float/int8 rows: use getPayload)
- VectorMatch.swift:19 `struct VectorMatch: Sendable, Comparable, Equatable` — itemID/distance/modelID; :43 `<` — (distance ASC, itemID ASC) universal tie-break

### Storage actor — VectorStore.swift
- :128 `actor VectorStore` — the kit's single consumer-facing surface
- :327 `static let schemaDeclaration` — "vectors" table v3; UNIQUE(item_id, vector_index, model_id) == VectorRecordKey minus modelVersion
- :382 `static defaultSidecarURL(for:)` — `<estate>.sqlite` → `<estate>.vectors.vec`; nil for non-file backends
- :165 `mihThreshold: UInt32 = 50_000` (default) — promotion boundary, overridable at init
- :172 `mihBandCount: MIHBandCount` (default .m16) — pinned per §1.6 for 50k default threshold
- :185/:189/:197 `bruteForceIndex` / `mihIndex` / `hotIndex` — both allocated at init; hotIndex swapped by :1661 `_selectIndex()` (no rebuild on swap)
- :283 `floatIndices: [String: FloatBruteForceIndex]` — ONE PER modelID (uniform stride requirement); map-entry presence == "built" flag
- :451 `addVector(itemID:engram:modelID:modelVersion:filedAt:)` — convenience wrapper over addPayload
- :496 `addPayload(itemID:vectorIndex:payload:modelID:modelVersion:filedAt:)` — rejects .int8 (throws int8QuantizationPolicyUndefined); table upsert THEN resident mirror; matches stale slots by (itemID,vectorIndex,modelID) — NOT full key — so modelVersion changes are treated as replacement (secfix/ws2-coredelete hard-delete contract); emits vectorkit.index.insert_latency_ms
- :669 `addPayloads(_:)` — bulk path; rejects batch containing ANY int8 (no partial writes); immediate mode rebuilds both indexes ONCE from final snapshot; emits vectorkit.index.batch_insert_latency_ms
- :874 `beginDeferredIndex()` / :898 `publishResidentIndex()` — bulk-burst mode: appends skip index rebuild until publish (O(N) not O(N²)); corpus ingest drain wraps a burst in this pair
- :253/:943 `deferredPendingRecords` / `_flushDeferredPending()` — memory-only-path back-pressure valve, capped at `deferredPendingLimit` (default 50_000, ctor param) — secfix/punt-vector unbounded-buffer fix
- :993 `flush()` — persists pending sidecar write-behind mutation
- :1052/:1066/:1088 `getVector` / `getPayload` / `vectors(forItemID:)` — read paths, all decode via :1803 `decodePayload` / :1840 `storedVector`
- :1134 `findNearest(probe:modelID:limit:)` — lazy `_ensureIndexBuilt()`, delegates to hotIndex.search, NO re-sort (engine already ordered); emits vectorkit.search.latency_ms + .result_count
- :1223 `findNearestFloat(probe:modelID:limit:)` — lazy per-model FloatBruteForceIndex build via :1558 `_ensureFloatIndexBuilt`; quantizes cosine distance ×10_000 rounded for cross-language integer comparison
- :1294 `findFarthestFloat(probe:modelID:limit:)` — same as findNearestFloat but calls searchFarthest; anti-similarity
- :1337 `findByKeyword(_:limit:)` — substring LIKE on item_id; NOT full BM25 (CorpusKit's job); emits vectorkit.search.keyword_result_count
- :1377 `deleteVector(itemID:modelID:)` / :1386 `deleteAllVectors(itemID:modelID:)` — both flush pending deferred burst FIRST if dirty, then delete+tombstone; deleteAllVectors invalidates that model's float index (lazy rebuild)
- :1442 `destroyAllVectors()` — full wipe: table+both binary indexes+sidecar+ALL float indices; used by GeniusLocusKit estate teardown
- :1487 `_ensureIndexBuilt()` — idempotent; sidecar trusted iff `snap.liveCount == tableCount` (live-vs-live, NOT snap.count vs table — avoids spurious rebuild after deletes, "C5 fix")
- :1558 `_ensureFloatIndexBuilt(modelID:)` — nil return (no cache) when model has zero float rows, so a later first-ingest can still build a real index
- :1736 `_deleteAndTombstone(itemID:vectorIndex:modelID:)` — scans ALL slots matching (itemID,vectorIndex,modelID) across modelVersions, tombstones every match (no break-after-first) — hard-delete contract
- :1803 `decodePayload(from row:)` — int8 rows ALWAYS decode to nil (symmetric fail-closed read guard, even for hand-crafted rows); guards every Int64→UInt8/UInt32 narrowing conversion against trap

## INVARIANTS / GOTCHAS

- I-7 ABSOLUTE: zero Hamming/XOR/popcount arithmetic anywhere in this package outside EngramLib calls. BruteForceIndex, MIHIndex, MaxSimScorer all delegate every distance to EngramLib.distances / EngramLib.Session.distances / EngramLib.distance. A raw popcount anywhere is a conformance violation.
- I-4 ABSOLUTE: cross-model vector comparison is forbidden. Every write carries modelID+modelVersion; every search is scoped to one modelID; float indices are one-per-model because different models emit different dimensions; the resident binary partition index scopes searches by modelID.
- Determinism boundary: binary lane (.hamming) is four-way bit-identical (Swift/Rust × platforms), gated by EngramLib's kernel. Float lane (.cosine/.l2/.dot) is reproducible-within-config ONLY — NOT four-way. Do not add a conformance test asserting float bit-identity; it will be correctly flagged as testing an undocumented, unintended guarantee.
- MIHIndex MUST equal BruteForceIndex output bit-for-bit on every input — this is a BLOCKER gate (MIHIndexTests.swift), not a best-effort property. BruteForceIndex is the oracle; never "fix" MIH by relaxing the gate.
- int8 is REJECTED fail-closed at both write (addPayload/addPayloads throw int8QuantizationPolicyUndefined) and read (decodePayload returns nil for kind==.int8). The case, field, and guards stay until a quantization policy is ratified — do not remove the guard as "dead code."
- Stale-slot matching in addPayload/addPayloads uses (itemID, vectorIndex, modelID) — NOT the full VectorRecordKey — specifically so a modelVersion change is recognized as replacement, not a new sibling slot. Matching by full key here is the recurring bug shape (secfix/ws2-coredelete); do not "simplify" to full-key equality.
- VectorRecordKey ordering (itemID, vectorIndex, modelID, modelVersion) is load-bearing: it is the resident-array partition order, the universal search tie-break, and the sidecar's on-disk key ordering. Do not reorder the Comparable fields.
- MIHBandCount is restricted to {4,8,16,32} by the enum itself (§1.7 conformance restriction) — sub_bits ∈ {64,32,16,8} guarantees no word-straddle in extractBand. Do not add m=2 or other values without a new word-straddle code path and a separate conformance harness.
- MIHIndex's enumeration-budget guard falls back to a full O(n) bruteScan (still EXACT, same heap, same distances) rather than hang on sparse/adversarial data. cumulativeChoose's saturating-overflow arithmetic MUST stay bit-identical to the Rust port's saturating_add so both fall back at the same radius.
- ResidentArrayStore sidecar is a REGENERABLE CACHE, never a second source of truth. The `vectors` table is authoritative. Staleness check compares live-vs-live counts (format 0x0002); comparing total slot counts instead (pre-C5-fix behavior) spuriously rebuilds after every delete.
- Sidecar single-add writes are WRITE-BEHIND (appendDeferred + isDirty): VectorStore.addPayload does not force a disk write per call. Callers relying on immediate on-disk durability of the sidecar must call `flush()`; crash safety does not depend on this because the table write already happened synchronously before the mirror.
- deferredIndexActive (beginDeferredIndex/publishResidentIndex) turns a bulk import from O(N²) to O(N) index rebuilds. deleteVector/deleteAllVectors both force-publish a dirty deferred window before deleting, so a delete never races an unpublished burst.
- deferredPendingRecords (memory-only deferred path) is capped at deferredPendingLimit (default 50,000); exceeding it triggers an intermediate flush that keeps the burst open. This is a back-pressure valve, not an error path — do not treat a flush mid-burst as anomalous.
- Float payload byte order is EXPLICIT little-endian (VectorPayload.init(floats:) / asFloats()), independent of host native endianness — required for the `.vec` sidecar (and any future float sidecar) to be byte-identical across Apple and Linux hosts.
- FloatBruteForceIndex establishes its stride from the FIRST vector added via add(key:vector:); a later vector of different byte count throws rather than corrupting the flat storage buffer. One index = one dimension, always.
- FloatBruteForceIndex is labeled "Lane C" in its own file header but "Lane D" in VectorStore.swift's comments — same type, inconsistent label, not a functional divergence. Do not "fix" one file to match the other without checking whether a Lane-lettering convention doc elsewhere disambiguates it first.
- MaxSimScorer (Lane E1) is NOT wired into VectorStore's public search API in this package version — it is a standalone scorer callers invoke directly with pre-fetched token-Engram arrays. Do not assume `findNearest` performs late interaction.
- Telemetry (Intellectus.report) is off by default; the emitted metrics (vectorkit.index.insert_latency_ms, .batch_insert_latency_ms, vectorkit.search.latency_ms, .result_count, .keyword_result_count, vectorkit.mih.enumeration_fallback) are named constants used by dashboards — renaming any of them is a breaking change to monitoring, not just to code.
- Pinned/default constants — changing any requires updating dependent conformance tests: mihThreshold 50,000, mihBandCount .m16 (sub_bits=16), deferredPendingLimit 50,000, compactionThreshold 0.25, poolSubmitThreshold-equivalent N/A (not used here), sidecar format version 0x0002, MIHBandCount ∈ {4,8,16,32}, cosine-distance quantization scale ×10,000.
- Actor boundaries: VectorStore, BruteForceIndex, MIHIndex, FloatBruteForceIndex, ResidentArrayStore are all actors — all mutation and reads are serialized per actor. ResidentVectorArray, VectorPayload, VectorRecordKey, DenseHit, VectorMatch, StoredVector are plain Sendable value types safe to pass across actor boundaries once constructed.

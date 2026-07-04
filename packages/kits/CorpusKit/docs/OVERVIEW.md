---
doc: OVERVIEW
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

# CorpusKit Overview

## What This Kit Does

CorpusKit stores text and finds it again by meaning as well as by keywords.
It is the retrieval tier of MOOTx01, an on-device AI memory system. The
technique it implements is called retrieval-augmented generation, or RAG. In
RAG, an AI looks up relevant stored material before it answers, so its
answers rest on real text instead of on guesswork.

A kit is a larger package that composes libraries into a subsystem. CorpusKit
composes storage, indexing, and embedding libraries into one database-like
surface. Callers hand it text; it splits the text into chunks, stores each
chunk, and indexes it two ways. A chunk is a piece of source text with a
stable identity, sized to a few sentences. Later, callers hand it a query and
receive the most relevant chunks back, scored and ranked.

The kit stands alone. A developer can use it as a private RAG database with
no other MOOT components. Inside MOOTx01, the GeniusLocusKit orchestrator
uses it as the estate's recall engine. An estate is one user's complete
memory store.

## The Problem It Solves

Recall must work on the device, deterministically, and without leaking text.
Cloud embedding services see private content, need a network, and change
without notice. If recall depended on them, a user's memory would be neither
private nor reproducible. Federation raises the stakes: MOOTx01 estates can
share memories across devices, and shared recall only works when every
device computes the same result from the same input — the agreement
property.

CorpusKit answers with two ranked lanes that both run entirely on device. A
lane is one independent way of scoring how well a chunk matches a query. The
keyword lane uses BM25, a standard formula that rewards chunks containing
the query's rarer words. The semantic lane uses embeddings. An embedding is
a list of numbers (a vector) that represents what a text means; texts with
similar meaning get nearby vectors. The two lanes are fused into one ranking
by Reciprocal Rank Fusion, a simple rule that rewards chunks ranked high in
either lane.

For the semantic lane, the kit ships an ensemble of five "honest" signals:
Random Indexing, PPMI, LSA, NMF, and FDC. Honest means each signal reflects
real word co-occurrence or real classification structure, never a disguised
hash of the surface text. All five are classical statistical methods. They
need no neural network, cost little, run identically on every platform, and
are gated by shared conformance fixtures — recorded input and output pairs
that the Swift leg and the Rust leg (in `rust/` and `rust-providers/`) must
reproduce exactly, byte for byte. Optional higher-quality neural providers
(MiniLM, mpnet, EmbeddingGemma, and two Apple NaturalLanguage providers)
plug into the same seam when a host supplies the model.

## How It Works

Ingestion runs in a pipeline. Text enters through an ingest queue backed by
QueueKit, so callers never wait on encoding. A background drain worker takes
batches from the queue and hands them to the `Corpus` actor, the kit's
central type. The `Corpus` splits each text into chunks with sentence-aware
boundaries. Each chunk receives a content-addressed identity: its UUID is
computed from its source, offset, and exact text. The same content always
produces the same identity, so re-ingesting a document is a harmless no-op
and two federated devices converge on identical rows.

Each stored chunk is then indexed twice. The keyword side tokenizes the
chunk and records term frequencies in a persistent inverted index — a table
mapping each word to the chunks that contain it. The semantic side runs the
chunk through every configured embedding signal and writes the resulting
vectors to VectorKit, the sibling kit that owns vector storage and
nearest-neighbor search. Content and vectors are joined by the chunk's UUID
string, so a chunk and its meaning never drift apart.

Four of the five honest signals are trainable: they learn a basis from the
corpus itself. A basis is the trained reference data a signal needs to embed
new text — for example, the word co-occurrence vectors Random Indexing
accumulates. Training happens exactly twice: automatically on first ingest,
and again whenever a caller requests an explicit reindex. Trained bases are
serialized to a pinned little-endian byte format and persisted, so a
reopened corpus embeds immediately without retraining.

Recall runs the pipeline in reverse. The query is embedded once, the vector
lane fetches its nearest neighbors, the keyword lane fetches its best BM25
matches, and Reciprocal Rank Fusion merges the two rankings with pinned
weights (0.6 vector, 0.4 keyword). The winners are hydrated from the chunk
store and returned as scored chunks. Ties always break toward the smaller
identifier, so results are deterministic down to the last position.

Deletion is honest about its limits. Chunk rows are immutable, so removing a
source deletes its index rows and vectors and records a tombstone that every
rebuild consults; expunging additionally scrubs the stored text itself.

## How the Pieces Fit

Figure 1 shows the kit's topology — its major parts and how data moves
between them.

![Figure 1. Topology of CorpusKit](topology.svg)

*Figure 1. Topology of CorpusKit. Ingested text flows through the queue and
the `Corpus` actor into the chunk store, the keyword index, and the vector
store. A query fans out to both index lanes, and Reciprocal Rank Fusion
merges them into scored chunks. Dashed regions mark the external kits and
the persisted tables.*

The `Corpus` actor is the seam everything passes through. It owns the chunk
store (`BundleStore`), the persistent keyword index (`InvertedIndexStore`),
the basis and counts stores, the tombstone store, and one slot per embedding
signal. It hides VectorKit behind its own surface, so consumers never touch
vector storage directly. The engine layer beneath it — sparse types, BM25
weighting, the WAND/Block-Max WAND inverted index, and the fusion function —
is pure computation with no storage of its own.

The package splits into two targets on purpose. The `CorpusKit` core target
holds the storage, the engines, and the protocols. The `CorpusKitProviders`
target holds every concrete embedding provider and tokenizer. Consumers that
only need storage and BM25 never pull in provider code or model seams.

## What Ships in the Package

The package ships the two Swift targets, thirty-four Swift source files in
all, and two mirror Rust crates: `rust/` for the core and `rust-providers/`
for the providers. Shared canonical vectors in `Tests/SharedVectors/` gate
both legs: BM25 impacts, per-provider embeddings, and serialized basis blobs
must match byte for byte. The kit bundles no model weights. Hosts that want
neural embeddings inject an inference function; everything the kit itself
computes is deterministic and reproducible from the sources alone.

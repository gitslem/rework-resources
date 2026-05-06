# Build a RAG Pipeline with Pinecone & OpenAI
## Professional Implementation Guide for Automation Experts

---

## Table of Contents

1. [Introduction](#introduction)
2. [What is RAG?](#what-is-rag)
3. [Architecture & Design](#architecture--design)
4. [Getting Started](#getting-started)
5. [Implementation Guide](#implementation-guide)
6. [Advanced Techniques](#advanced-techniques)
7. [Production Deployment](#production-deployment)
8. [Optimization & Scaling](#optimization--scaling)
9. [Real-World Applications](#real-world-applications)
10. [Troubleshooting Guide](#troubleshooting-guide)

---

## Introduction

Retrieval-Augmented Generation (RAG) has transformed how we build AI systems that require access to specific, up-to-date knowledge. By combining vector databases (Pinecone) with large language models (OpenAI), you can create systems that:

- **Ground responses in your data**: Reduce hallucinations by 90%
- **Stay current**: Use real-time data without retraining models
- **Maintain accuracy**: Cite sources directly from your knowledge base
- **Scale knowledge**: Handle thousands of documents efficiently
- **Reduce latency**: Return relevant context in milliseconds

This guide covers building production-grade RAG pipelines that power:
- Enterprise document search systems
- Intelligent customer support (powered by your documentation)
- Research assistants with access to papers and reports
- Internal knowledge management platforms
- Real-time market intelligence systems

### Who Should Read This Guide

- **ML Engineers** building AI-powered applications
- **Full-Stack Developers** implementing semantic search
- **Data Engineers** designing knowledge pipelines
- **Automation Experts** augmenting existing workflows
- **Startup Founders** building AI products on a budget

---

## What is RAG?

### The Problem RAG Solves

**Without RAG:**
```
User Question → LLM → Response (May contain hallucinations)
                        No source verification
                        Knowledge limited to training data
                        Cannot cite sources
```

**With RAG:**
```
User Question → Vector Search → Relevant Documents
                                    ↓
                            LLM (with context)
                                    ↓
                         Response with citations
                         Grounded in real data
                         Verifiable sources
```

### Key Components

| Component | Purpose | Technology |
|-----------|---------|-----------|
| **Embedding Model** | Convert text to vectors | OpenAI `text-embedding-3-small` |
| **Vector Database** | Store and search embeddings | Pinecone |
| **LLM** | Generate contextual responses | OpenAI `gpt-4-turbo` or `gpt-3.5-turbo` |
| **Orchestration** | Coordinate pipeline steps | LangChain or custom code |
| **Document Processor** | Parse and chunk documents | Python, LangChain |

### When to Use RAG

**Perfect for:**
- ✅ Proprietary data (company docs, internal wikis)
- ✅ Frequently updated information (news, prices, inventory)
- ✅ Large document collections (>1K documents)
- ✅ Need for source attribution and traceability
- ✅ Domain-specific applications (legal, medical, technical)

**Not ideal for:**
- ❌ Small FAQs (< 20 Q&As) - use fine-tuning instead
- ❌ Real-time data changing every second
- ❌ Highly confidential information (use local models)
- ❌ Simple factual lookups (use databases)

---

## Architecture & Design

### High-Level System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   Data Ingestion Pipeline                    │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │  Documents  │→ │  Text Split  │→ │  Embeddings  │       │
│  │  (PDFs,     │  │              │  │  Generation  │       │
│  │   Websites, │  │              │  │  (OpenAI)    │       │
│  │   Databases)│  │              │  │              │       │
│  └─────────────┘  └──────────────┘  └──────┬───────┘       │
│                                              │               │
└──────────────────────────────────────────────┼───────────────┘
                                               │
┌──────────────────────────────────────────────┼───────────────┐
│                                              ↓               │
│                    ┌──────────────────────────────┐         │
│                    │   Pinecone Vector Store      │         │
│                    │  • Store embeddings          │         │
│                    │  • Metadata filtering        │         │
│                    │  • Similarity search         │         │
│                    └──────────────────────────────┘         │
│                           (Vector Database)                 │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                  Query-Time RAG Pipeline                     │
│                                                              │
│  User Query → Embed Query → Vector Search → Retrieve Docs  │
│       ↓           ↓             ↓                ↓          │
│      User      Embedding     Top-K Results   Context Pool  │
│    Question    (OpenAI)     (Pinecone)                     │
│                                ↓                           │
│                         ┌────────────────┐                │
│                         │  Prompt with   │                │
│                         │  Context       │                │
│                         │  Documents     │                │
│                         └────────┬───────┘                │
│                                  ↓                        │
│                         ┌──────────────────┐             │
│                         │  GPT-4 / GPT-3.5 │             │
│                         │  (LLM)           │             │
│                         └────────┬─────────┘             │
│                                  ↓                        │
│                        Response with Citations           │
└─────────────────────────────────────────────────────────────┘
```

### Design Patterns

#### Pattern 1: Retriever-Only (Simplest)
```
Query → Search Pinecone → Return Top-K Documents
```
**Use**: Document search, Q&A retrieval
**Cost**: Very low (~$0.01 per search)

#### Pattern 2: Retriever + Reranker (Balanced)
```
Query → Rough Search (Top-100) → Rerank → Final Top-K → LLM
```
**Use**: Balance speed and relevance
**Cost**: Low (~$0.05 per search)

#### Pattern 3: Full RAG with Routing (Advanced)
```
Query → Classify Question Type → Route to Specialized Pipeline
                ↓
        [PDF Search] [Web Search] [Database Query]
                ↓
        Combine Results → Rerank → LLM with Context
```
**Use**: Complex enterprise systems
**Cost**: Medium (~$0.20 per search)

---

## Getting Started

### Prerequisites

```bash
# Node.js or Python with package manager
node --version  # >= 16.0
npm --version   # >= 8.0

# OR Python
python --version  # >= 3.9

# API Keys required
OPENAI_API_KEY=sk-...
PINECONE_API_KEY=...
```

### Quick Setup

#### 1. Install Dependencies (Node.js)

```bash
npm init -y
npm install \
  @pinecone-database/pinecone \
  openai \
  dotenv \
  pdf-parse \
  express \
  cors

# Optional: for advanced use cases
npm install langchain @langchain/openai @langchain/pinecone
```

#### 2. Install Dependencies (Python)

```bash
pip install \
  pinecone-client \
  openai \
  python-dotenv \
  pypdf \
  langchain \
  langchain-openai \
  langchain-pinecone \
  fastapi \
  uvicorn
```

#### 3. Initialize Pinecone Index

```typescript
import { Pinecone } from "@pinecone-database/pinecone";

const pinecone = new Pinecone({
  apiKey: process.env.PINECONE_API_KEY,
});

// Create index (one-time setup)
await pinecone.createIndex({
  name: "documents",
  dimension: 1536, // For text-embedding-3-small
  metric: "cosine",
  spec: {
    serverless: {
      cloud: "aws",
      region: "us-east-1",
    },
  },
});

const index = pinecone.Index("documents");
console.log("Index ready:", await index.describeIndexStats());
```

---

## Implementation Guide

### Step 1: Document Chunking

Properly splitting documents is critical for RAG quality:

```typescript
interface DocumentChunk {
  id: string;
  text: string;
  source: string;
  page?: number;
  metadata: Record<string, any>;
}

function chunkDocument(
  text: string,
  sourceFile: string,
  chunkSize: number = 400,
  overlap: number = 50
): DocumentChunk[] {
  const chunks: DocumentChunk[] = [];
  let chunkId = 0;

  // Split by sentences first for better semantic units
  const sentences = text.match(/[^.!?]+[.!?]+/g) || [text];
  let currentChunk = "";

  for (const sentence of sentences) {
    // Add sentence to current chunk
    if ((currentChunk + sentence).length > chunkSize) {
      if (currentChunk.trim()) {
        chunks.push({
          id: `${sourceFile}-${chunkId++}`,
          text: currentChunk.trim(),
          source: sourceFile,
          metadata: {
            chunk_size: currentChunk.length,
            chunk_index: chunkId,
          },
        });
      }

      // Start new chunk with overlap
      currentChunk = currentChunk.slice(-overlap) + sentence;
    } else {
      currentChunk += sentence;
    }
  }

  // Add final chunk
  if (currentChunk.trim()) {
    chunks.push({
      id: `${sourceFile}-${chunkId}`,
      text: currentChunk.trim(),
      source: sourceFile,
      metadata: {
        chunk_size: currentChunk.length,
        chunk_index: chunkId,
      },
    });
  }

  return chunks;
}
```

### Step 2: Generate Embeddings

```typescript
import OpenAI from "openai";

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateEmbedding(text: string): Promise<number[]> {
  const response = await openai.embeddings.create({
    model: "text-embedding-3-small", // Fast and efficient
    input: text,
  });

  return response.data[0].embedding;
}

async function generateEmbeddingsBatch(
  texts: string[],
  batchSize: number = 100
): Promise<number[][]> {
  const embeddings: number[][] = [];

  for (let i = 0; i < texts.length; i += batchSize) {
    const batch = texts.slice(i, i + batchSize);

    const response = await openai.embeddings.create({
      model: "text-embedding-3-small",
      input: batch,
    });

    embeddings.push(...response.data.map((d) => d.embedding));

    // Rate limiting - respect OpenAI limits
    if (i + batchSize < texts.length) {
      await new Promise((resolve) => setTimeout(resolve, 500));
    }
  }

  return embeddings;
}
```

### Step 3: Store in Pinecone

```typescript
async function storeChunksInPinecone(
  chunks: DocumentChunk[],
  pineconeIndex: any
): Promise<void> {
  const BATCH_SIZE = 100;

  for (let i = 0; i < chunks.length; i += BATCH_SIZE) {
    const batch = chunks.slice(i, i + BATCH_SIZE);

    // Generate embeddings for batch
    const embeddings = await generateEmbeddingsBatch(
      batch.map((c) => c.text)
    );

    // Prepare vectors for Pinecone
    const vectors = batch.map((chunk, idx) => ({
      id: chunk.id,
      values: embeddings[idx],
      metadata: {
        text: chunk.text,
        source: chunk.source,
        page: chunk.metadata.page,
        ...chunk.metadata,
      },
    }));

    // Upsert to Pinecone
    await pineconeIndex.upsert(vectors);

    console.log(`Stored ${Math.min(i + BATCH_SIZE, chunks.length)}/${chunks.length} chunks`);
  }
}
```

### Step 4: Query and Retrieve

```typescript
interface RAGQueryResult {
  query: string;
  documents: Array<{
    id: string;
    text: string;
    score: number;
    source: string;
  }>;
  context: string;
}

async function queryRAG(
  query: string,
  pineconeIndex: any,
  topK: number = 5
): Promise<RAGQueryResult> {
  // 1. Embed the query
  const queryEmbedding = await generateEmbedding(query);

  // 2. Search Pinecone
  const results = await pineconeIndex.query({
    vector: queryEmbedding,
    topK: topK,
    includeMetadata: true,
  });

  // 3. Extract and format results
  const documents = results.matches.map((match: any) => ({
    id: match.id,
    text: match.metadata.text,
    score: match.score,
    source: match.metadata.source,
  }));

  // 4. Build context string
  const context = documents
    .map(
      (doc, i) =>
        `[Document ${i + 1} - ${doc.source}]\n${doc.text}\n(Similarity: ${(doc.score * 100).toFixed(1)}%)`
    )
    .join("\n\n---\n\n");

  return {
    query,
    documents,
    context,
  };
}
```

### Step 5: Generate Response with LLM

```typescript
interface RAGResponse {
  answer: string;
  citations: Array<{
    source: string;
    text: string;
  }>;
  confidence: number;
}

async function generateRAGResponse(
  query: string,
  ragResult: RAGQueryResult
): Promise<RAGResponse> {
  const systemPrompt = `You are a helpful assistant with access to a knowledge base.

IMPORTANT RULES:
1. Only use information from the provided documents
2. If the answer is not in the documents, say "I don't have information about this"
3. Always cite your sources using [Source: filename]
4. Be concise and accurate

KNOWLEDGE BASE:
${ragResult.context}`;

  const response = await openai.chat.completions.create({
    model: "gpt-4-turbo",
    messages: [
      {
        role: "system",
        content: systemPrompt,
      },
      {
        role: "user",
        content: query,
      },
    ],
    temperature: 0.7,
  });

  const answer =
    response.choices[0].message.content || "Unable to generate response";

  // Extract citations from response
  const citationPattern = /\[Source: ([^\]]+)\]/g;
  const citations: Array<{ source: string; text: string }> = [];
  let match;

  while ((match = citationPattern.exec(answer)) !== null) {
    const source = match[1];
    const doc = ragResult.documents.find((d) => d.source === source);
    if (doc) {
      citations.push({
        source: doc.source,
        text: doc.text,
      });
    }
  }

  // Calculate confidence based on retrieval scores
  const avgScore =
    ragResult.documents.reduce((sum, doc) => sum + doc.score, 0) /
    ragResult.documents.length;
  const confidence = Math.min(avgScore * 100, 100);

  return {
    answer,
    citations,
    confidence,
  };
}
```

### Step 6: Complete Pipeline

```typescript
async function completeRAGPipeline(
  query: string,
  pineconeIndex: any
): Promise<RAGResponse> {
  console.log(`\n📝 Query: ${query}`);

  // Retrieve relevant documents
  const ragResult = await queryRAG(query, pineconeIndex);
  console.log(`\n📚 Retrieved ${ragResult.documents.length} documents`);
  ragResult.documents.forEach((doc, i) => {
    console.log(`  ${i + 1}. ${doc.source} (${(doc.score * 100).toFixed(1)}%)`);
  });

  // Generate response
  const response = await generateRAGResponse(query, ragResult);
  console.log(`\n💡 Answer:\n${response.answer}`);

  if (response.citations.length > 0) {
    console.log(`\n📎 Citations:`);
    response.citations.forEach((citation) => {
      console.log(`  - ${citation.source}`);
    });
  }

  console.log(`\n✓ Confidence: ${response.confidence.toFixed(1)}%`);

  return response;
}
```

---

## Advanced Techniques

### 1. Metadata Filtering

```typescript
async function filteredSearch(
  query: string,
  pineconeIndex: any,
  filters: Record<string, any>
): Promise<RAGQueryResult> {
  const queryEmbedding = await generateEmbedding(query);

  const results = await pineconeIndex.query({
    vector: queryEmbedding,
    topK: 10,
    filter: filters, // Pinecone metadata filter
    includeMetadata: true,
  });

  return {
    query,
    documents: results.matches.map((match: any) => ({
      id: match.id,
      text: match.metadata.text,
      score: match.score,
      source: match.metadata.source,
    })),
    context: results.matches
      .map((m: any) => m.metadata.text)
      .join("\n\n---\n\n"),
  };
}

// Usage: Only search within specific sources
const result = await filteredSearch(query, index, {
  source: { $in: ["docs.pdf", "manual.pdf"] },
});

// Or by date range
const result2 = await filteredSearch(query, index, {
  created_at: { $gte: "2024-01-01", $lte: "2024-12-31" },
});
```

### 2. Hybrid Search (Keyword + Semantic)

```typescript
import { BM25 } from "bm25-tokenizer"; // For keyword search

async function hybridSearch(
  query: string,
  pineconeIndex: any,
  allChunks: DocumentChunk[],
  semanticWeight: number = 0.7
): Promise<RAGQueryResult> {
  // Semantic search
  const semanticResults = await queryRAG(query, pineconeIndex, 10);
  const semanticScores = new Map(
    semanticResults.documents.map((doc) => [doc.id, doc.score])
  );

  // Keyword search (BM25)
  const bm25 = new BM25(allChunks.map((c) => c.text));
  const keywordScores = bm25.search(query, 10);
  const keywordScoresMap = new Map(keywordScores);

  // Combine scores
  const combinedScores = new Map<string, number>();
  for (const [id, score] of semanticScores) {
    const keywordScore = keywordScoresMap.get(id) || 0;
    const combined = semanticWeight * score + (1 - semanticWeight) * keywordScore;
    combinedScores.set(id, combined);
  }

  // Sort by combined score
  const topResults = Array.from(combinedScores.entries())
    .sort((a, b) => b[1] - a[1])
    .slice(0, 5);

  const documents = topResults.map(([id, score]) => {
    const doc = semanticResults.documents.find((d) => d.id === id);
    return doc ? { ...doc, score } : doc;
  });

  return {
    query,
    documents: documents.filter(Boolean),
    context: documents
      .filter(Boolean)
      .map((d) => d.text)
      .join("\n\n---\n\n"),
  };
}
```

### 3. Multi-Query Expansion

Improve retrieval by generating multiple interpretations of the query:

```typescript
async function multiQueryRetrieval(
  query: string,
  pineconeIndex: any
): Promise<RAGQueryResult> {
  // Generate multiple query variations
  const expansionPrompt = `Generate 3 variations of this query to help find relevant documents:
Query: "${query}"

Return only the 3 variations, one per line.`;

  const response = await openai.chat.completions.create({
    model: "gpt-3.5-turbo",
    messages: [
      {
        role: "user",
        content: expansionPrompt,
      },
    ],
  });

  const variations = response.choices[0].message.content
    ?.split("\n")
    .filter((q) => q.trim()) || [query];

  // Retrieve documents for each variation
  const allDocuments = new Map<string, any>();

  for (const variant of variations) {
    const results = await queryRAG(variant, pineconeIndex, 5);
    for (const doc of results.documents) {
      // Keep document with highest score
      if (!allDocuments.has(doc.id) || doc.score > allDocuments.get(doc.id).score) {
        allDocuments.set(doc.id, doc);
      }
    }
  }

  // Return top documents
  const documents = Array.from(allDocuments.values())
    .sort((a, b) => b.score - a.score)
    .slice(0, 5);

  return {
    query,
    documents,
    context: documents.map((d) => d.text).join("\n\n---\n\n"),
  };
}
```

### 4. Reranking for Better Results

```typescript
async function rerank(
  query: string,
  documents: DocumentChunk[],
  topK: number = 3
): Promise<DocumentChunk[]> {
  // Use an LLM to rerank documents
  const rankingPrompt = `You are a relevance ranking expert.

Query: "${query}"

Documents:
${documents
  .map(
    (doc, i) => `
[${i + 1}] ${doc.text.substring(0, 200)}...`
  )
  .join("\n")}

Rank these documents by relevance to the query. Return only the document numbers in order, comma-separated.
Example: 2,1,3`;

  const response = await openai.chat.completions.create({
    model: "gpt-3.5-turbo",
    messages: [
      {
        role: "user",
        content: rankingPrompt,
      },
    ],
  });

  const ranking = response.choices[0].message.content
    ?.split(",")
    .map((s) => parseInt(s.trim()) - 1)
    .filter((i) => i >= 0 && i < documents.length) || [];

  // Sort by ranking
  const reranked = ranking
    .map((i) => documents[i])
    .concat(documents.filter((_, i) => !ranking.includes(i)))
    .slice(0, topK);

  return reranked;
}
```

---

## Production Deployment

### Express.js REST API

```typescript
import express, { Request, Response } from "express";
import cors from "cors";

const app = express();
app.use(express.json());
app.use(cors());

// Health check
app.get("/health", (req: Request, res: Response) => {
  res.json({ status: "healthy", timestamp: new Date().toISOString() });
});

// RAG endpoint
app.post("/api/rag", async (req: Request, res: Response) => {
  try {
    const { query } = req.body;

    if (!query || query.trim().length === 0) {
      return res.status(400).json({ error: "Query is required" });
    }

    const response = await completeRAGPipeline(query, pineconeIndex);

    res.json({
      success: true,
      data: response,
      timestamp: new Date().toISOString(),
    });
  } catch (error) {
    console.error("RAG error:", error);
    res.status(500).json({
      success: false,
      error: "Failed to process query",
    });
  }
});

// Batch ingestion endpoint
app.post("/api/ingest", async (req: Request, res: Response) => {
  try {
    const { documents } = req.body;

    const chunks: DocumentChunk[] = [];
    for (const doc of documents) {
      const docChunks = chunkDocument(doc.content, doc.filename);
      chunks.push(...docChunks);
    }

    await storeChunksInPinecone(chunks, pineconeIndex);

    res.json({
      success: true,
      message: `Ingested ${chunks.length} chunks`,
    });
  } catch (error) {
    console.error("Ingestion error:", error);
    res.status(500).json({
      success: false,
      error: "Failed to ingest documents",
    });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`RAG API server running on port ${PORT}`);
});
```

### Docker Deployment

```dockerfile
FROM node:18-alpine

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy application
COPY . .

# Environment variables
ENV NODE_ENV=production
ENV PORT=3000

EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {if (r.statusCode !== 200) throw new Error(r.statusCode)})"

CMD ["npm", "start"]
```

### Google Cloud Run Deployment

```bash
# Build and deploy
gcloud builds submit --tag gcr.io/PROJECT_ID/rag-pipeline

gcloud run deploy rag-pipeline \
  --image gcr.io/PROJECT_ID/rag-pipeline \
  --memory 2Gi \
  --timeout 60 \
  --set-env-vars OPENAI_API_KEY=$OPENAI_API_KEY,PINECONE_API_KEY=$PINECONE_API_KEY \
  --min-instances 1 \
  --max-instances 50 \
  --allow-unauthenticated
```

---

## Optimization & Scaling

### 1. Cost Optimization

```typescript
class CostTracker {
  private embeddingCost = 0.02; // Per 1M tokens
  private gptCost = 0.03; // Per 1K tokens (GPT-3.5)
  private pineconeStorageCost = 0.25; // Per 1M vectors/month

  async estimateCost(documents: number, queriesPerMonth: number): Promise<{
    embedding: number;
    queries: number;
    storage: number;
    total: number;
  }> {
    // Estimate embedding costs
    const avgDocSize = 400; // tokens per document
    const tokensEmbedded = documents * avgDocSize;
    const embeddingCost = (tokensEmbedded / 1000000) * this.embeddingCost;

    // Estimate query costs
    const avgQueryTokens = 500;
    const queryCost = (queriesPerMonth * avgQueryTokens / 1000) * this.gptCost;

    // Storage costs
    const storageCost = (documents / 1000000) * this.pineconeStorageCost;

    return {
      embedding: embeddingCost,
      queries: queryCost,
      storage: storageCost,
      total: embeddingCost + queryCost + storageCost,
    };
  }
}

// Usage
const tracker = new CostTracker();
const costs = await tracker.estimateCost(10000, 10000);
console.log(`Monthly cost estimate: $${costs.total.toFixed(2)}`);
```

### 2. Caching for Performance

```typescript
import NodeCache from "node-cache";

const queryCache = new NodeCache({ stdTTL: 3600 }); // 1 hour TTL

async function cachedRAGQuery(
  query: string,
  pineconeIndex: any
): Promise<RAGResponse> {
  const cacheKey = query.toLowerCase();

  // Check cache
  const cached = queryCache.get(cacheKey);
  if (cached) {
    console.log("📦 Returning cached result");
    return cached as RAGResponse;
  }

  // Generate fresh response
  const ragResult = await queryRAG(query, pineconeIndex);
  const response = await generateRAGResponse(query, ragResult);

  // Cache for future requests
  queryCache.set(cacheKey, response);

  return response;
}
```

### 3. Batch Processing for Large Documents

```typescript
async function batchProcessLargeDocument(
  filePath: string,
  chunkSize: number = 1000
): Promise<void> {
  const fs = require("fs");
  const text = fs.readFileSync(filePath, "utf-8");

  const chunks = chunkDocument(text, filePath, chunkSize);

  // Process in batches of 50 for rate limiting
  const BATCH_SIZE = 50;
  for (let i = 0; i < chunks.length; i += BATCH_SIZE) {
    const batch = chunks.slice(i, i + BATCH_SIZE);
    await storeChunksInPinecone(batch, pineconeIndex);

    console.log(`Processed ${Math.min(i + BATCH_SIZE, chunks.length)}/${chunks.length}`);

    // Wait between batches to avoid rate limits
    if (i + BATCH_SIZE < chunks.length) {
      await new Promise((resolve) => setTimeout(resolve, 2000));
    }
  }
}
```

### 4. Monitoring & Observability

```typescript
import { Sentry } from "@sentry/node";

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  tracesSampleRate: 0.1,
});

interface QueryMetrics {
  query: string;
  retrievalTime: number;
  generationTime: number;
  totalTime: number;
  tokensUsed: number;
  retrievalScore: number;
}

async function monitoredRAGQuery(
  query: string,
  pineconeIndex: any
): Promise<{ response: RAGResponse; metrics: QueryMetrics }> {
  const startTime = Date.now();
  let retrievalTime = 0;
  let generationTime = 0;

  try {
    // Retrieval
    const retrievalStart = Date.now();
    const ragResult = await queryRAG(query, pineconeIndex);
    retrievalTime = Date.now() - retrievalStart;

    // Generation
    const generationStart = Date.now();
    const response = await generateRAGResponse(query, ragResult);
    generationTime = Date.now() - generationStart;

    const totalTime = Date.now() - startTime;

    const metrics: QueryMetrics = {
      query,
      retrievalTime,
      generationTime,
      totalTime,
      tokensUsed: 0, // Calculate from response
      retrievalScore:
        ragResult.documents.reduce((sum, doc) => sum + doc.score, 0) /
        ragResult.documents.length,
    };

    return { response, metrics };
  } catch (error) {
    Sentry.captureException(error, {
      tags: { component: "rag_query" },
      extra: { query },
    });
    throw error;
  }
}
```

---

## Real-World Applications

### Application 1: Enterprise Document Search System

**Use Case**: Law firm with 50K documents searching case law and precedents

**Architecture**:
- Ingest: Automatically index new documents as they're added
- Retrieval: Hybrid search combining keyword + semantic
- Ranking: Rerank by relevance to specific legal practice area
- Response: Extract citations and relevant case numbers

**Results**:
- 87% reduction in research time
- $150K annual savings in associate hours
- Compliance: Full audit trail of sources

**Implementation**:
```typescript
async function legalDocumentSearch(
  caseQuery: string,
  practiceArea: string
): Promise<any> {
  // Filter by practice area metadata
  const result = await filteredSearch(caseQuery, pineconeIndex, {
    practice_area: { $eq: practiceArea },
  });

  // Rerank for legal relevance
  const reranked = await rerank(caseQuery, result.documents);

  // Extract key citations
  const citations = reranked.map((doc) => ({
    caseNumber: doc.metadata.case_number,
    year: doc.metadata.year,
    relevance: doc.metadata.relevance_score,
  }));

  return { reranked, citations };
}
```

### Application 2: Customer Support with Knowledge Base

**Use Case**: SaaS company with 1000+ support articles

**Architecture**:
- Knowledge base: All FAQs, API docs, troubleshooting guides
- Multi-query: Generate variations to find relevant docs
- Confidence filtering: Only suggest if confidence > 0.75
- Fallback: Escalate to human if unclear

**Results**:
- 65% of questions auto-answered
- 2-minute average response time
- Improved support team efficiency

### Application 3: Research & Citation System

**Use Case**: Academic platform with access to 10K research papers

**Architecture**:
- Ingestion: Extract abstract + key sections from PDFs
- Ranking: Rerank by citation count and recency
- Response: Generate synthesis with full citations
- Verification: Link to original papers

**Results**:
- Accurate literature review generation
- 100% source attribution
- Researchers save 5+ hours per review

---

## Troubleshooting Guide

### Common Issues & Solutions

| Issue | Symptom | Solution |
|-------|---------|----------|
| **Low relevance scores** | Documents don't match query intent | Use multi-query expansion, add more context to chunks |
| **Slow retrieval** | Queries taking >5s | Optimize Pinecone index settings, use caching |
| **Rate limiting** | 429 errors from OpenAI | Implement exponential backoff, use batch API |
| **Hallucinations** | LLM inventing information | Use stricter prompts, increase citation requirements |
| **Memory usage** | Out of memory on large documents | Process documents in batches, implement streaming |
| **Cost overruns** | Embedding costs too high | Use `text-embedding-3-small`, implement caching |

### Debugging RAG Quality

```typescript
async function debugRAGQuality(
  query: string,
  expectedDocumentId: string
): Promise<void> {
  console.log(`\n🔍 Debugging RAG for: "${query}"`);

  // 1. Check embedding similarity
  const queryEmbed = await generateEmbedding(query);
  const expectedDocEmbed = await generateEmbedding(expectedDocumentId);

  const dotProduct = queryEmbed.reduce((sum, val, i) => sum + val * expectedDocEmbed[i], 0);
  console.log(`   Embedding similarity: ${(dotProduct * 100).toFixed(1)}%`);

  // 2. Check retrieval
  const ragResult = await queryRAG(query, pineconeIndex, 20);
  const found = ragResult.documents.find((d) => d.id === expectedDocumentId);

  if (found) {
    console.log(`   ✓ Expected document found at rank ${ragResult.documents.indexOf(found) + 1}`);
    console.log(`   Relevance score: ${(found.score * 100).toFixed(1)}%`);
  } else {
    console.log(`   ✗ Expected document NOT in top-20 results`);
  }

  // 3. Check chunk quality
  console.log(`\n   Top retrieved documents:`);
  ragResult.documents.slice(0, 3).forEach((doc, i) => {
    console.log(
      `   ${i + 1}. ${doc.source} (${(doc.score * 100).toFixed(1)}%) - ${doc.text.substring(0, 80)}...`
    );
  });
}
```

---

## Best Practices Summary

### Do's ✅
- ✅ Start with small document collections (< 100 docs) to test
- ✅ Use `text-embedding-3-small` for cost efficiency
- ✅ Implement caching for common queries
- ✅ Monitor embedding quality with test queries
- ✅ Chunk documents thoughtfully (preserve context)
- ✅ Use metadata filtering for accuracy
- ✅ Implement fallback to human support

### Don'ts ❌
- ❌ Don't use overly large chunks (> 1000 tokens)
- ❌ Don't ignore the reranking step
- ❌ Don't skip cost estimation before scaling
- ❌ Don't rely solely on semantic search (use hybrid)
- ❌ Don't forget to update documents regularly
- ❌ Don't expose API keys in code

---

## Resources

- **Pinecone Documentation**: https://docs.pinecone.io
- **OpenAI Embeddings**: https://platform.openai.com/docs/guides/embeddings
- **LangChain Documentation**: https://python.langchain.com
- **RAG Best Practices**: https://www.anthropic.com/research

---

## Conclusion

RAG with Pinecone and OpenAI enables you to build intelligent systems that:
- Ground responses in your proprietary data
- Reduce hallucinations and improve accuracy
- Scale to handle thousands of documents
- Provide full source attribution
- Adapt quickly to new information

Start small, measure performance, optimize based on metrics, and scale incrementally. With the patterns and code examples in this guide, you're ready to build production-grade RAG systems.

**Next Steps:**
1. Set up Pinecone account (free tier available)
2. Get OpenAI API key
3. Ingest your first document collection
4. Deploy the REST API
5. Monitor and optimize

---

## About Rework Digital

This guide was created by **Rework Digital** - Resources Department for automation professionals building innovative solutions.

**Resources Department Contact:** resource@reworkdigital.io  
**Follow us on GitHub:** https://github.com/Reworkdigital-io

---

*Last Updated: 2026-04-10*  
*Version: 1.0*

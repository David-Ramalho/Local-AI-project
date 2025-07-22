# 🚀 Guide to Configuring RAG in Open WebUI with Ollama(OnProgress)
## 💾 Low-VRAM Best Practices for 4GB GPUs
-  🎯 Perfect for:  Low-VRAM setups like GTX 1650 in my case. The better the GPU the Merrier! :)
 -  ⚡ Performance:  Optimized for single-container deployment. You can apply same logic in split containers as well 
 -  📊 Reference Config:  Cogito 3B Q4 + znbang/bge:small-en-v1.5-q8_0

---

Retrieval-Augmented Generation (RAG) empowers your local AI to leverage external knowledge sources (documents, web data, etc.) for more accurate and contextually informed responses. This comprehensive guide provides battle-tested best practices for setting up RAG in Open WebUI using Ollama, specifically optimized for systems with limited GPU memory.

### 🔧 **Quick Reference Configuration**
| Component | Setting | Value |
|-----------|---------|-------|
| 🧠 LLM Model | Cogito 3B Q4 | ~2.2GB VRAM |
| 📝 Context Length | Extended | 8,048 tokens |
| 🔍 Embedding Model | znbang/bge:small-en-v1.5-q8_0 | Batch size: 4 |
| 📄 Chunk Size | Optimal | 1,000 tokens |
| 🔄 Chunk Overlap | Balanced | 100 tokens |
| 📊 Retrieval Top-K | Performance | 7 chunks |

---

## 📚 Table of Contents

- [🎯 Step 1: Pulling and Configuring the LLM Model](#-step-1-pulling-and-configuring-the-llm-model)
- [🔍 Step 2: Setting the Embedding Model in Open WebUI](#-step-2-setting-the-embedding-model-in-open-webui)
- [📁 Step 3: Creating a Knowledge Base for Your Documents](#-step-3-creating-a-knowledge-base-for-your-documents)
- [⚙️ Step 4: Indexing Documents (Chunking and Embedding)](#️-step-4-indexing-documents-chunking-and-embedding)
- [🔗 Step 5: Enabling RAG for the LLM (Connecting the Knowledge Base)](#-step-5-enabling-rag-for-the-llm-connecting-the-knowledge-base)
- [🧪 Step 6: Testing Retrieval and Tuning for Performance](#-step-6-testing-retrieval-and-tuning-for-performance)
- [🎉 Conclusion & Key Configuration Summary](#-conclusion--key-configuration-summary)
- [📖 Sources](#-sources)

---

## 🎯 Step 1: Pulling and Configuring the LLM Model

The foundation of our RAG system starts with selecting an appropriate large language model that can efficiently handle retrieval tasks within our VRAM constraints.

### 🌟 **Recommended Model**
- Cogito 3B Q4_K_M - Our top choice for low-VRAM setups. This reasoning model delivers impressive performance while fitting comfortably within 4GB of VRAM.
Alternative Options.
- Qwen 3 1.7B - An even lighter option for extremely constrained hardware setups, though with reduced capabilities compared to the Cogito model.
- Qwen 3 4B - Offers enhanced intelligence and reasoning capabilities, but operates at the upper limit of 4GB VRAM capacity. 
### Settings for Cogito: 
- 📊 **Parameters:** 3.6B (4-bit quantized)
- 💾 **Storage:** ~2.2GB on disk
- 🔥 **VRAM Usage:** Fits comfortably in 4GB
- 📏 **Context:** Up to 128K tokens (we'll use ~8K for optimal performance)

### 🚀 **Installation**

```bash
# Pull the model
ollama pull cogito:3b

# Verify installation
ollama run cogito:3b -p "Hello, RAG world!"
```

### 🎛️ **Why Extended Context Matters**

| Context Size | Use Case | RAG Suitability |
|--------------|----------|-----------------|
| 2,048 tokens | ❌ Default LLaMA | Too small for RAG |
| 4,096 tokens | ⚠️ Basic tasks | Limited RAG capability |
| 8,192 tokens | ✅ **Recommended** | Perfect for multiple chunks |
| 16,384+ tokens | 🔥 Advanced | Overkill for most cases |

### 🔀 **Alternative Models**

<details>
<summary><strong>📋 Click to expand alternative options</strong></summary>

| Model | Size | VRAM | Context | Notes |
|-------|------|------|---------|-------|
| **Mistral 7B Q4** | 7B | ~4GB | 4K-8K | Higher quality, tighter fit |
| **Llama2 7B Q4** | 7B | ~4GB | 4K | Classic choice |
| **Phi-3 Mini** | 3.8B | ~2GB | 128K | Microsoft's efficient model |
| **Gemma 2B** | 2B | ~1.5GB | 8K | Ultra-lightweight |

</details>

### ⚡ **Performance Expectations**

> **GTX 1650 Performance:**
> - 🔄 **Generation Speed:** 2-4 tokens/second
> - ⏱️ **First Token:** ~2-3 seconds
> - 📊 **VRAM Usage:** ~2.2GB for model + ~1GB for context and RAG

---

## 🔍 Step 2: Setting the Embedding Model in Open WebUI

Configure your embedding engine for optimal document retrieval performance.

### 🛠️ **Configuration Path**
```
Admin Settings → Documents → Embedding
```

### 🎯 **Embedding Model Engine Setup**

#### **Option 1: Ollama API (Recommended)**
```
Endpoint: http://localhost:11434
✅ Uses GPU acceleration
✅ Consistent with LLM setup
✅ Better performance
```

#### **Option 2: SentenceTransformers (Fallback)**
```
Engine: SentenceTransformers
⚠️ CPU-only processing
⚠️ Slower but more compatible
```

### 🌟 **Top Embedding Model Choices**

#### **🥇 Primary Recommendation**
**bge:small-en-v1.5-q8_0** (`bge:small-en-v1.5-q8_0`) For low VRam GPUS.
- 📊 **Size:** ~568M parameters
- 📏 **Input Length:** 8,192 tokens
- 🎯 **Specialty:** High-quality general embeddings
- 💾 **VRAM:** ~1.2GB

```bash
ollama pull snowflake-arctic-embed2:latest
```

#### **🥈 Alternative Choice**
**BAAI BGE M3** (`bge-m3:latest`)
- 📊 **Size:** Similar scale to Arctic
- 🌐 **Multilingual:** Excellent for non-English content
- 🔍 **Features:** Dense + sparse + multi-vector retrieval

#### **🥉 Lightweight Option**
**MiniLM L12 v2** (`paraphrase-MiniLM-L12-v2`)
- 📊 **Size:** ~15M parameters (ultra-light)
- ⚡ **Speed:** Very fast processing
- 💾 **VRAM:** Minimal usage

### 🔧 **Critical Performance Settings**

| Setting | Recommended | Alternative | Impact |
|---------|-------------|-------------|---------|
| **Batch Size** | **4** | 2, 8 | 4× speedup vs batch=1 |
| **Hybrid Search** | **✅ Enabled** | Disabled | +20% retrieval quality |
| **Re-ranker** | **❌ Disabled** | Enabled | Saves ~500MB VRAM |

### 💡 **Memory Optimization Tips**

<details>
<summary><strong>🔍 VRAM Breakdown Analysis</strong></summary>

```
Total VRAM Usage Estimate:
┌─────────────────────────┬──────────┐
│ Component               │ Usage    │
├─────────────────────────┼──────────┤
│ Cogito 3B Model         │ ~2.2GB   │
│ Arctic Embed v2         │ ~1.2GB   │
│ Context Buffer          │ ~0.4GB   │
│ System Overhead         │ ~0.2GB   │
├─────────────────────────┼──────────┤
│ **Total**               │ **4.0GB**│
└─────────────────────────┴──────────┘
```

</details>

---

## 📁 Step 3: Creating a Knowledge Base for Your Documents

Set up your document repository for RAG integration.

### 🎯 **Step-by-Step Creation**

1. **🧭 Navigate:** `Workspace → Knowledge`
2. **➕ Create:** Click `"+ Create a Knowledge Base"`
3. **🏷️ Name:** Choose descriptive name (e.g., `TechDocs-RAG`, `CompanyKB`)
4. **🎯 Purpose:** Select appropriate category:
   - 📋 **Assistance** - General help and support
   - ❓ **Q&A** - FAQ and question answering
   - 📚 **Reference** - Technical documentation
   - 🎓 **Learning** - Educational materials

### 💡 **Naming Best Practices**

| ✅ Good Examples | ❌ Avoid |
|------------------|---------|
| `API-Documentation-v2024` | `docs` |
| `Customer-Support-KB` | `kb1` |
| `Technical-Manuals-Engineering` | `stuff` |

---

## ⚙️ Step 4: Indexing Documents (Chunking and Embedding)

Transform your documents into searchable, AI-ready knowledge chunks.

### 📤 **Document Upload Process**

1. **📁 Access:** `Knowledge → [Your KB Name]`
2. **➕ Upload:** Click `"+"` button
3. **📄 Formats Supported:**
   - ✅ **Markdown** (`.md`)
   - ✅ **Text Files** (`.txt`)
   - ✅ **PDFs** (basic + Tika-enhanced)
   - ✅ **Word Documents** (`.docx`)
   - ✅ **HTML Files**

### 🔧 **Optimal Chunking Configuration**

| Parameter | Value | Reasoning |
|-----------|-------|-----------|
| **Tokenizer** | `tiktoken` | OpenAI-compatible, accurate |
| **Chunk Size** | **1,000 tokens** | ~750-800 words, optimal balance |
| **Overlap** | **100 tokens** | 10% overlap prevents context loss |

### 📊 **Chunking Strategy Deep Dive**

<details>
<summary><strong>🧠 Understanding the Trade-offs</strong></summary>

#### **Smaller Chunks (500-750 tokens)**
- ✅ More precise retrieval
- ✅ Better for specific facts
- ❌ May lose broader context
- ❌ More chunks = more embeddings

#### **Larger Chunks (1200-1500 tokens)**
- ✅ Preserves more context
- ✅ Fewer embeddings needed
- ❌ Less precise matching
- ❌ May exceed embedding model limits

#### **Our Sweet Spot (1000 tokens)**
- 🎯 Balances precision and context
- 🎯 Fits well within embedding limits
- 🎯 Optimal for most document types

</details>

### 🔄 **Indexing Process Monitoring**

```
Progress Indicators:
📁 Document Processing    ████████░░ 80%
🔍 Chunk Generation      ███████░░░ 70%
🧮 Embedding Creation    ██████░░░░ 60%
💾 Vector Storage        █████░░░░░ 50%
```

### 🚨 **Common Issues & Solutions**

| Issue | Solution |
|-------|----------|
| **PDF Text Garbled** | Enable Apache Tika in Admin Settings |
| **Chunk Size Error** | Reduce chunk size to fit embedding model |
| **Slow Processing** | Lower embedding batch size |
| **Memory Issues** | Process fewer documents simultaneously |

---

## 🔗 Step 5: Enabling RAG for the LLM (Connecting the Knowledge Base)

Bridge your language model with your knowledge repository.

### 🎯 **Model Configuration Steps**

1. **🧭 Navigate:** `Workspace → Models`
2. **➕ Create:** Click `"+ Add New Model"`
3. **🏷️ Name:** Descriptive identifier (e.g., `Cogito-3B-TechDocs-RAG`)
4. **🧠 Base Model:** Select `cogito:3b`
5. **📚 Knowledge Source:** Link your created knowledge base
6. **💾 Save:** Confirm configuration

### ⚙️ **Critical Retrieval Settings**

| Setting | Recommended | Range | Impact |
|---------|-------------|-------|---------|
| **Top K** | **7** | 3-10 | Number of chunks retrieved |
| **Score Threshold** | **0.1** | 0.0-0.5 | Minimum relevance score |
| **Hybrid Search** | **✅ On** | On/Off | Keyword + semantic search |
| **Re-ranking** | **❌ Off** | On/Off | Advanced relevance sorting |

### 📐 **Context Math**

```
Context Allocation (8,048 tokens):
┌─────────────────────┬────────────┐
│ Component           │ Tokens     │
├─────────────────────┼────────────┤
│ Retrieved Chunks    │ ~7,000     │
│ (7 × 1,000 tokens)  │            │
│ User Question       │ ~50-200    │
│ System Instructions │ ~300-500   │
│ Response Buffer     │ ~348-698   │
├─────────────────────┼────────────┤
│ **Total**          │ **8,048**  │
└─────────────────────┴────────────┘
```

### 🎨 **Custom Prompt Template**

The default RAG template structure:
```
📋 Reference Materials:
[Retrieved Chunk 1]
[Retrieved Chunk 2]
...
[Retrieved Chunk N]

❓ Question: {user_question}
```

**Customization Path:** `Admin Settings → Documents → RAG Template`

---

## 🧪 Step 6: Testing Retrieval and Tuning for Performance

Validate and optimize your RAG system for production use.

### ✅ **Basic Functionality Test**

1. **💬 Start Chat:** Create new chat with your RAG-enabled model
2. **❓ Test Query:** Ask about content from your documents
   ```
   Example: "What are the installation requirements mentioned in the setup guide?"
   ```
3. **🔍 Verify Citations:** Look for reference markers like **【Doc1†L10-L20】**

### 📊 **Performance Benchmarks**

| Metric | Target | Acceptable | Poor |
|--------|--------|------------|------|
| **Retrieval Time** | <1s | <2s | >3s |
| **First Token** | <3s | <5s | >8s |
| **Generation Speed** | 2-4 tok/s | 1-2 tok/s | <1 tok/s |
| **Citation Accuracy** | >90% | >70% | <50% |

### 🎛️ **Quality Tuning Matrix**

| Problem | Likely Cause | Solution |
|---------|--------------|----------|
| **Missing Information** | Low Top-K or poor chunks | ↗️ Increase Top-K to 8-10 |
| **Irrelevant Results** | No score threshold | ↗️ Set threshold to 0.1-0.2 |
| **Hallucinations** | Weak instruction | ↗️ Add "Only use provided docs" |
| **Poor Citations** | Citation settings off | ↗️ Enable RAG citations |
| **Slow Performance** | High batch/context | ↘️ Reduce batch size or Top-K |

### 🔧 **Advanced Optimization**

<details>
<summary><strong>🚀 Performance Tuning Checklist</strong></summary>

#### **Memory Optimization**
- [ ] Monitor VRAM usage with `nvidia-smi`
- [ ] Adjust batch size based on available memory
- [ ] Consider CPU fallback for embedding if needed

#### **Quality Improvement**
- [ ] Test with out-of-document queries
- [ ] Verify multi-document retrieval works
- [ ] Experiment with different prompt templates

#### **Speed Enhancement**
- [ ] Enable streaming responses
- [ ] Optimize chunk size for your use case
- [ ] Consider model quantization levels

</details>

### 🚨 **Troubleshooting Guide**

<details>
<summary><strong>🔧 Common Issues and Quick Fixes</strong></summary>

| 🚨 Issue | 🔍 Diagnosis | ✅ Solution |
|----------|--------------|-------------|
| **No retrieval happening** | Context too small or KB not linked | Verify 8K+ context, check KB connection |
| **Context length exceeded** | Too many/large chunks | Reduce Top-K or chunk size |
| **OOM during generation** | VRAM exhausted | Lower settings or use CPU fallback |
| **Embedding OOM** | Batch size too high | Reduce batch to 2 or 1 |
| **Terse responses** | Default instructions too limiting | Add "detailed answer" to prompt |
| **No citations** | Citation feature disabled | Enable RAG citations in settings |

</details>

---

## 🎉 Conclusion & Key Configuration Summary

### 🎯 **Optimal Configuration Recap**

| Component | Setting | Value | Purpose |
|-----------|---------|-------|---------|
| 🧠 **LLM Model** | Cogito 3B Q4 | 8K context | Efficient reasoning |
| 🔍 **Embedding** | Arctic Embed v2 | Batch: 4 | Quality retrieval |
| 📄 **Chunking** | Smart split | 1K tokens, 100 overlap | Balanced context |
| 📊 **Retrieval** | Hybrid search | Top-K: 7 | Comprehensive results |
| 💾 **Memory** | Optimized | ~4GB total | GTX 1650 compatible |

### 🚀 **System Capabilities**

Your fully configured RAG system now provides:

- ✅ **Document-Grounded Responses** with proper citations
- ✅ **Multi-Source Information Synthesis** across your knowledge base
- ✅ **Memory-Efficient Operation** on consumer hardware
- ✅ **Real-Time Retrieval** with sub-second response times
- ✅ **Scalable Architecture** for growing document collections

### 🎊 **You're Ready!**

Congratulations! You now have a production-ready, locally-hosted RAG system that rivals cloud solutions while maintaining complete data privacy and control. Your AI assistant is now equipped with your domain knowledge and ready to provide accurate, contextual responses.

---

## 📖 Sources

### 📚 **Official Documentation**
- [Open WebUI RAG Features](https://docs.openwebui.com/features/rag/) - Core RAG functionality
- [Ollama Model Library](https://ollama.com/library/) - Model repository and specs

### 🧠 **Model Resources**
- [Cogito 3B Model Page](https://ollama.com/library/cogito:3b) - Primary LLM documentation
- [znbang/bge:small-en-v1.5-q8_0]((https://ollama.com/znbang/bge:small-en-v1.5-q8_0)) - Embedding model details
- [BGE M3 Documentation](https://huggingface.co/BAAI/bge-m3) - Alternative embedding model

### 💡 **Community Resources**
- **Reddit Discussions:** r/LocalLLaMA RAG configuration threads
- **GitHub Issues:** Open WebUI repository troubleshooting
- **Medium Article:** "Multi-Source RAG with Hybrid Search and Re-ranking in OpenWebUI" by Richard Meyer (2025)

### 🛠️ **Technical References**
- **Memory Optimization:** Community benchmarks and VRAM usage studies
- **Chunking Strategies:** Academic papers on optimal text segmentation
- **Performance Tuning:** User-contributed optimization guides

---

<div align="center">

### 🌟 **Happy RAG-ing!** 🌟

*Built with ❤️ for the local AI community*

[![GitHub](https://img.shields.io/badge/GitHub-Contribute-green?style=for-the-badge&logo=github)](https://github.com)
[![OpenWebUI](https://img.shields.io/badge/OpenWebUI-Docs-blue?style=for-the-badge)](https://docs.openwebui.com)
[![Ollama](https://img.shields.io/badge/Ollama-Models-orange?style=for-the-badge)](https://ollama.com)

</div>


Research paper:

````markdown
# Guide to Configuring RAG in Open WebUI with Ollama (Low‑VRAM Best Practices)

Retrieval‑Augmented Generation (RAG) allows your local AI to use external knowledge (documents, web data, etc.) to produce more accurate and informed answers. This guide walks through best practices for setting up RAG in Open WebUI using Ollama on a single‑container setup – ideal for a single 4 GB VRAM GPU (e.g. GTX 1650). We use a reference configuration (embedding batch size 4, chunk size 1000, chunk overlap 100, retrieval top_k 7, context ≈8048 tokens) and the Cogito 3B Q4 model as an example. Each step includes recommended settings, model choices, performance impacts, and troubleshooting tips.

---

## Table of Contents

1. [Pulling and Configuring the LLM Model](#step-1-pulling-and-configuring-the-llm-model)  
2. [Setting the Embedding Model in Open WebUI](#step-2-setting-the-embedding-model-in-open-webui)  
3. [Creating a Knowledge Base for Your Documents](#step-3-creating-a-knowledge-base-for-your-documents)  
4. [Indexing Documents (Chunking and Embedding)](#step-4-indexing-documents-chunking-and-embedding)  
5. [Enabling RAG for the LLM (Connecting the Knowledge Base)](#step-5-enabling-rag-for-the-llm-connecting-the-knowledge-base)  
6. [Testing Retrieval and Tuning for Performance](#step-6-testing-retrieval-and-tuning-for-performance)  
7. [Conclusion & Key Configuration Summary](#conclusion--key-configuration-summary)  
8. [Sources](#sources)  

---

## Step 1: Pulling and Configuring the LLM Model

The first step is to obtain a suitable large language model (LLM) that can handle RAG on limited GPU memory.

### Recommended Model

- **Cogito 3B Q4\_K\_M** – a 3.6 B‑parameter model quantized to 4‑bit (≈2.2 GB on disk)  
  - Supports up to 128 K tokens of context (we’ll use ~8 K tokens / 8048)  
  - Fits comfortably in 4 GB of VRAM

### Why Extended Context?

- Default LLaMA models have a 2048‑token window, too short for RAG  
- Increasing to ~8192 tokens ensures space for multiple 1000‑token chunks + query overhead

### How to Pull

```bash
ollama pull cogito:3b
````

> Tip: After pulling, verify with:
>
> ```bash
> ollama run cogito:3b -p "Hello"
> ```

### Alternate Models

* **Mistral 7B 4‑bit** or **Llama2 7B Q4** (may need to lower context to 4096)
* Smaller (2–4 B) run faster but can be less capable
* Balance VRAM use vs. accuracy

### Model Settings

* Set the context length to **8192** (or **8048** for comfort)
* In Open WebUI, ensure the model’s max context reflects this setting

### Performance Impact & Troubleshooting

* Expect a few tokens/sec on GTX 1650
* Extended context increases KV‑cache size and slightly reduces speed
* If out‑of‑memory, choose a smaller/ more‑aggressive quantized model (e.g. 3‑bit) or run CPU‑only (slow)

---

## Step 2: Setting the Embedding Model in Open WebUI

Configure your embedding engine in **Admin Settings > Documents**.

### Embedding Model Engine

* **Ollama API endpoint** (e.g. `http://localhost:11434`) – uses GPU for speed
* Fallback: SentenceTransformers on CPU (slower)

### Embedding Model Choice

1. **bge:small-en-v1.5-q8_0** (`bge:small-en-v1.5-q8_0`)

   * \~568 M params, 8192‑token input length
2. **BAAI BGE M3** (`BAAI/bge-m3`)

   * Similar scale and token length

> Lighter option: `paraphrase‑MiniLM‑L12‑v2` (\~15 M params)

### Batch Size

* **4** (safe on 4 GB, 4× speedup)
* Try **8** if headroom; drop to **2** or **1** if OOM

### Hybrid Search

* **Enable** for keyword + dense retrieval
* BM25 runs on CPU, minimal GPU impact

### Re‑ranker Model (Optional)

* Cross‑encoder (e.g. `bge-reranker-v2-m3`) can improve accuracy
* **Off** by default on 4 GB to conserve resources

### Memory & Performance

* Embed model (\~1.2 GB) + Cogito (\~2.2 GB) ≈ 3.4 GB total
* If OOM, run embed on CPU or choose a smaller model

---

## Step 3: Creating a Knowledge Base for Your Documents

1. **Workspace > Knowledge**
2. Click **“+ Create a Knowledge Base”**
3. Name it (e.g. `MyDocs` or `Tech Manuals KB`)
4. Choose a Purpose (e.g. `Assistance`, `QA`, `Reference`)
5. Save and confirm

> Ensure **chunking** and **embedding** settings (from Steps 1–2) are in place before indexing.

---

## Step 4: Indexing Documents (Chunking and Embedding)

### Uploading Documents

* **Knowledge > \[Your KB]**
* Use **“+”** to upload files or folders (Markdown, text, PDF, etc.)
* For complex PDFs, enable Apache Tika in Admin Settings

### Chunking Strategy

* **Tokenizer**: Tiktoken
* **Chunk Size**: **1000** tokens (\~750–800 words)
* **Overlap**: **100** tokens (10% overlap)

> Trade‑off:
>
> * Smaller chunks → more focused retrieval
> * Larger chunks → fewer but richer context blocks

### Indexing Process

1. Files are split into chunks
2. Each chunk is embedded (batch size 4)
3. Vectors stored in the index

Monitor progress in UI or via Docker logs.

### Troubleshooting

* **Unsupported formats** → convert or enable Tika
* **Chunk too large** → reduce size to fit embed model’s limit
* **Slow indexing** → check that GPU embed is enabled; lower batch if VRAM is maxed

---

## Step 5: Enabling RAG for the LLM (Connecting the Knowledge Base)

1. **Workspace > Models**
2. Click **“+ Add New Model”**
3. Name it (e.g. `Cogito 3B + MyDocs (RAG)`)
4. **Base Model**: select your pulled Cogito 3B
5. **Knowledge Source**: select `MyDocs`
6. Save/confirm

### Retrieval Settings

* **Top K**: **7** (≈7000 tokens → fits in 8048)
* **Min Score Threshold**: **0–0.1** (optional)
* **Hybrid Search**: on (BM25 + embedding)
* **Re‑ranking**: off by default on 4 GB

### Prompt Template

* Default RAG template prepends:

  ```
  Reference:
  <chunk 1>
  <chunk 2>
  …
  Question: {user question}
  ```
* Customize in **Admin Settings > Documents > RAG Template**

---

## Step 6: Testing Retrieval and Tuning for Performance

### 6.1 Basic Functionality

1. **Chat > New Chat** with `Cogito 3B + MyDocs (RAG)`
2. Ask a question in your docs (e.g. “What does the installation section say…?”)
3. Look for citation markers (e.g. **【Doc1†L10-L20】**)

### 6.2 Performance & Latency

* **Retrieval**: < 1 s
* **Generation**: \~5–10 s for \~100 tokens on GTX 1650
* If slow: check GPU usage, re‑ranks, streaming settings

### 6.3 Quality Tuning

* **Omit info** → raise Top K or enable re‑ranker
* **Extraneous info** → tighten score threshold or instructions
* **Hallucinations** → add “Answer only from documents” in prompt
* **Citation issues** → ensure RAG citations are enabled

### 6.4 Balancing Quality & Speed

* Prefer small RAG model (3 B + docs) over large non‑RAG model
* Monitor `nvidia-smi` for VRAM peaks
* Adjust chunk size, Top K, batch, and embedding model as needed

### 6.5 Common Issues & Fixes

| Issue                   | Fix                                                         |
| ----------------------- | ----------------------------------------------------------- |
| No retrieval            | Confirm context ≥ 8192, KB linked, re‑index if necessary    |
| Context‑length exceeded | Reduce Top K or chunk size                                  |
| OOM during generation   | Lower chunk/Top K, unload other models, or use CPU fallback |
| Embedding OOM           | Lower batch size or run embed on CPU                        |
| Terse answers           | Add “Give a detailed answer with references” to prompt      |

### 6.6 Ongoing Verification

* **Out‑of‑docs query**: see if it answers or states “not in documents”
* **Multi‑part queries**: ensure it retrieves from all relevant docs
* **System prompts**: experiment with stronger instructions for citation

---

## Conclusion & Key Configuration Summary

* **LLM Model**: Cogito 3B Q4, \~8 K context
* **Embedding Model**: Snowflake Arctic Embed v2 or BGE M3 via Ollama
* **Chunking**: 1000 tokens, 100 overlap
* **Retrieval**: Top K 7, hybrid search on, re‑rank off
* **Batching**: embed batch size 4
* **Context Size**: \~8048 tokens

By following these steps, you’ll have a fully local RAG system on Open WebUI + Ollama running comfortably on a 4 GB GTX 1650. Enjoy your grounded, document‑aware AI assistant!

---

## Sources

* [Open WebUI RAG Overview](https://docs.openwebui.com/features/rag/)
* [Ollama Cogito 3B Model](https://ollama.com/library/cogito:3b)
* [Ollama Arctic Embed v2](https://ollama.com/library/snowflake-arctic-embed2)
* Reddit user discussions on Open WebUI RAG settings
* Medium: “Multi‑Source RAG with Hybrid Search and Re‑ranking in OpenWebUI” (Richard Meyer, 2025)
* Various GitHub issues and discussions on batch size, memory tuning, and troubleshooting

```
```

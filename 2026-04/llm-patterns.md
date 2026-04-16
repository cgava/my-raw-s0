# Common LLM Application Patterns

Large Language Models are most effective when combined with architectural patterns that compensate for their limitations. Three patterns have emerged as foundational in production LLM systems: Retrieval-Augmented Generation (RAG), Chain-of-Thought prompting, and Tool Use.

## Retrieval-Augmented Generation (RAG)

RAG addresses the knowledge cutoff problem by retrieving relevant documents from an external corpus before generating a response. The typical RAG pipeline has three stages: indexing (chunking documents and computing embeddings), retrieval (finding the top-k most relevant chunks via vector similarity search), and generation (feeding the retrieved context into the LLM prompt alongside the user query).

A key design decision in RAG is chunk size. Smaller chunks improve retrieval precision but lose surrounding context. Larger chunks preserve coherence but dilute relevance scores. Hybrid approaches use small chunks for retrieval but expand to parent chunks at generation time.

Vector databases like Pinecone, Weaviate, and Chroma have become the standard storage layer for RAG systems. However, pure vector search often misses keyword-exact matches, leading many teams to adopt hybrid retrieval that combines vector similarity with BM25 lexical search.

## Chain-of-Thought (CoT)

Chain-of-Thought prompting asks the model to show its reasoning steps before producing a final answer. This simple technique dramatically improves performance on math, logic, and multi-step reasoning tasks. Zero-shot CoT (just adding "let's think step by step") works surprisingly well, but few-shot CoT with curated examples is more reliable for production use.

Recent work extends CoT into tree-of-thought and graph-of-thought approaches, where the model explores multiple reasoning paths and selects the most promising one. These methods trade latency for accuracy.

## Tool Use and Function Calling

Tool use lets an LLM invoke external functions -- calculators, APIs, databases, code interpreters -- to perform actions it cannot do natively. The model generates structured function calls (typically JSON) that are executed by the host application, with results fed back into the conversation.

OpenAI's function calling API, Anthropic's tool use, and open-source frameworks like LangChain and LlamaIndex have standardized this pattern. The key challenge is reliable structured output -- the model must produce valid JSON that matches the function schema, which requires careful prompt engineering or constrained decoding.

Tool use is what separates chatbots from agents. An LLM with access to tools can browse the web, query databases, write and execute code, and interact with external services -- making it an active participant in workflows rather than a passive text generator.

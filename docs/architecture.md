# Universal PDF Knowledge Assistant — Architecture

## Overview

The Universal PDF Knowledge Assistant is a Flowise-based Retrieval-Augmented Generation (RAG) application that allows users to ask questions about an uploaded PDF.

The system processes the PDF, converts its text into vector embeddings, stores those embeddings, retrieves relevant sections when a question is asked, and uses Google Gemini to generate an answer based on the retrieved document content.

## Architecture

```text
PDF
 ↓
Recursive Character Text Splitter
 ↓
Google Gemini Embeddings
 ↓
In-Memory Vector Store
 ↓
Memory Retriever
 ↓
Conversational Retrieval QA
 ↓
Google Gemini Chat Model
 ↓
Answer
```

## 1. PDF Loading

The PDF Loader is responsible for reading the uploaded PDF and converting its contents into documents that can be processed by the rest of the pipeline.

The current version supports one PDF at a time.

## 2. Text Splitting

The extracted document is divided into smaller chunks using the Recursive Character Text Splitter.

### Configuration

* Chunk Size: 1000
* Chunk Overlap: 200

Chunking allows the system to retrieve smaller and more relevant sections of the document instead of passing the entire PDF to the language model.

The overlap helps preserve context between neighboring chunks.

## 3. Embeddings

Each document chunk is converted into a numerical vector using Google's:

`gemini-embedding-001`

The embedding configuration uses:

`RETRIEVAL_DOCUMENT`

These vectors represent the semantic meaning of the document chunks and allow the system to search for content that is relevant to a user's question.

## 4. Vector Storage

The generated embeddings are stored using Flowise's:

**In-Memory Vector Store**

The current configuration uses:

* Top K: 4

This means the retriever can return the most relevant chunks for a user's question.

An in-memory store was selected for the first version to keep the project simple and avoid additional database or server infrastructure.

## 5. Retrieval

The **Memory Retriever** receives the user's question and searches the vector store for the most relevant document chunks.

These retrieved chunks provide the context that is passed to the conversational question-answering system.

## 6. Conversational Retrieval QA

The Conversational Retrieval QA Chain connects the retrieval system with the language model.

It receives:

* Chat Model
* Vector Store Retriever
* Conversation Memory

The chain also contains custom prompts for question rephrasing and response generation.

### Rephrase Prompt

The rephrase prompt converts follow-up questions into standalone questions.

For example:

```text
User: What is machine learning?

User: What are its advantages?
```

The second question can be rewritten into a standalone question referring explicitly to machine learning.

This helps the retriever understand follow-up questions correctly.

### Response Prompt

The response prompt instructs the assistant to:

* Use information from the retrieved PDF context.
* Avoid relying on outside knowledge.
* Avoid inventing information.
* Clearly state when the requested information cannot be found in the uploaded PDF.
* Explain concepts clearly when appropriate.

This helps keep the assistant grounded in the uploaded document.

## 7. Gemini Chat Model

The retrieved document context is passed to a Google Gemini chat model.

Current model:

`gemini-3-flash-preview`

The model generates the final natural-language response using the retrieved PDF information.

## Complete Data Flow

```text
                 ┌──────────────────┐
                 │    PDF Upload    │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │   PDF Loader     │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │  Text Splitter   │
                 │ 1000 / 200       │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │ Gemini Embeddings│
                 │ gemini-embedding │
                 │      -001        │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │ In-Memory Vector │
                 │      Store       │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │ Memory Retriever │
                 │    Top K = 4     │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │ Conversational   │
                 │ Retrieval QA     │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │ Gemini Chat Model│
                 │ 3 Flash Preview  │
                 └────────┬─────────┘
                          ↓
                 ┌──────────────────┐
                 │  Grounded Answer │
                 └──────────────────┘
```

## Current Limitations

The current version is intentionally designed as a V1 prototype.

### Single PDF

Only one PDF is processed at a time.

### In-Memory Storage

The vector store is not persistent. If the Flowise environment is restarted, the document may need to be processed again.

### No Authentication

The current prototype does not include user authentication or access control.

### No Multi-Document Retrieval

The current system does not yet allow users to search across multiple PDFs simultaneously.

## Future Improvements

Possible future versions could include:

* Persistent vector storage
* Multiple PDF support
* Document management
* Metadata filtering
* Source citations in answers
* Improved chat interface
* User authentication
* Conversation history
* Support for additional document formats
* Deployment as a web application
* More advanced retrieval strategies

## Conclusion

The Universal PDF Knowledge Assistant demonstrates a complete Retrieval-Augmented Generation workflow using Flowise and Google Gemini.

The project combines document processing, semantic embeddings, vector retrieval, conversational question answering, and prompt-based grounding into a single visual AI workflow.

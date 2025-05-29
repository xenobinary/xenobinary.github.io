---
title: "Using RAG with LLMs for University Quiz Questions"
date: 2025-05-29 00:49:00 +0000
categories: [AI, RAG]
tags: [AI, RAG, Python]
author: srodi
---

# 

University quizzes can be challenging, especially when professors craft ambiguous questions with wordplay and metaphorical language. General-purpose LLMs like ChatGPT often struggle with these specialized, context-specific questions. This guide shows how to use Retrieval-Augmented Generation (RAG) to dramatically improve your quiz performance.

## The Problem

Online quizzes in university settings present unique challenges:
- Ambiguous question phrasing
- Limited time to search through materials
- Professor-specific styles and preferences
- General LLMs lack course-specific context

## The Solution: RAG Pipeline

RAG combines document retrieval with LLM generation, allowing models to access relevant course materials before answering questions.

## Step 1: API Setup

Register for developer accounts and obtain API keys:
- **OpenAI**: platform.openai.com
- **Google Gemini**: aistudio.google.com
- **DeepSeek**: platform.deepseek.com

## Step 2: Document Processing

Extract text from PDFs and create manageable chunks:

```python
import fitz  # PyMuPDF

def extract_text_from_pdf(path):
    doc = fitz.open(path)
    pages = []
    for page_num in range(len(doc)):
        text = doc[page_num].get_text().strip()
        if text:
            pages.append({"page": page_num + 1, "text": text})
    return pages

def chunk_text(text, chunk_size=200, overlap=100):
    words = text.split()
    chunks = []
    i = 0
    while i < len(words):
        chunk = words[i:i+chunk_size]
        if chunk:
            chunks.append(" ".join(chunk))
        i += chunk_size - overlap
    return chunks
```

## Step 3: Generate Embeddings & Save to a json file

Use OpenAI's embedding API for cost-effective vector creation:

```python
from openai import OpenAI
client = OpenAI(api_key=your_api_key)

def get_embedding(text, model="text-embedding-3-small"):
    response = client.embeddings.create(
        input=[text],
        model=model
    )
    return response.data[0].embedding
```
```python
def save_to_json(data, path="embeddings.json"):
    with open(path, "w") as f:
        json.dump(data, f, indent=2)
```
## Step 4: Build Vector Index

Create a FAISS index for efficient similarity search:

```python
import faiss
import numpy as np

def build_faiss_index(vectors):
    dim = vectors.shape[1]
    index = faiss.IndexFlatL2(dim)
    index.add(vectors)
    return index

def normalize(vectors):
    norms = np.linalg.norm(vectors, axis=1, keepdims=True)
    return vectors / norms

# Build and save index
texts, vectors, metadata = load_json_embeddings("embeddings.json")
vectors = normalize(vectors)
index = build_faiss_index(vectors)
faiss.write_index(index, "index/faiss.index")
```

## Step 5: Query and Retrieve

Search for relevant context based on quiz questions:

```python
def search_faiss(query_vec, index, texts, sources, top_k=5):
    D, I = index.search(query_vec[np.newaxis, :], top_k)
    results = []
    for i in I[0]:
        results.append({
            "text": texts[i],
            "source": sources[i]
        })
    return results
```

## Step 6: Generate Answers

Create contextual prompts and call LLM APIs:

```python
def build_prompt(question, retrieved_chunks):
    context = "\n\n".join([f"[{item['source']}]\n{item['text']}" for item in retrieved_chunks])
    return f"""
Your task is to answer a quiz question from the professor.

Here are the relevant textbook notes and lecture passages:
{context}

Question: "{question}"

Give your answer with deep reasoning, clarity, and style-aware insight.
"""
```

## Multi-Model Verification

Use multiple LLM providers for cross-verification:

### OpenAI Implementation
```python
response = client.chat.completions.create(
    model="o4-mini",
    reasoning_effort="medium",
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": prompt}
    ],
    max_completion_tokens=300
)
```

### Google Gemini Implementation
```python
from google import genai
client = genai.Client(api_key=your_google_apikey)

response = client.models.generate_content(
    model="gemini-2.5-pro-exp-03-25",
    config=types.GenerateContentConfig(system_instruction=system_prompt),
    contents=prompt
)
```

### DeepSeek Implementation
```python
client = OpenAI(api_key=your_deepseek_api_key, base_url="https://api.deepseek.com")

response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user", "content": prompt}
    ],
    temperature=0.7
)
```

## System Prompt Strategy

Configure your LLMs to handle professor-specific question styles:

```
You are a brilliant reasoning tutor helping with quiz questions from a professor known for:
- Wordplay and puns
- Indirect or metaphorical questions  
- Ambiguous phrasing

You must:
- Think deeply and infer meaning behind questions
- Reference notes and textbook content
- Respond clearly and concisely
- Match the professor's communication style
```

## Notes

1. **Chunk Size**: Adjust chunk size (200 words) and overlap (100 words) based on your materials
2. **Top-K Retrieval**: Use 5-7 retrieved chunks for optimal context
3. **Cross-Verification**: Compare answers across multiple models
4. **Cost Optimization**: OpenAI embeddings are cost-effective for this use case
5. **VPN Usage**: Use Proton VPN or Google Colab if university blocks certain APIs


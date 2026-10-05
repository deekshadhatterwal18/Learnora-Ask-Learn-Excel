# Learnora

Learnora is an AI-based educational chatbot for Class 12 students.

It allows students to select a subject and chapter, ask questions, and get answers based on the textbook content. It also provides relevant YouTube videos for additional learning.

The project uses Retrieval-Augmented Generation (RAG) to connect textbook content with an LLM.

---

## Features

- Subject selection
- Chapter selection
- Specific chapter-based question answering
- Option to search from all chapters
- AI chatbot
- Textbook-based answers using RAG
- Conversation memory
- Semantic search
- Vector database
- Three relevant YouTube video recommendations
- Simple web interface using Streamlit

---

## Technologies Used

### Python

Python is the main programming language used in the project.

It is used for:

- PDF processing
- Text splitting
- Embedding generation
- Vector database creation
- Retrieval
- LLM integration
- Chatbot logic
- YouTube search

### Streamlit

Streamlit is used to create the web interface.

It provides:

- Subject selection
- Chapter selection
- Chat input
- Chat messages
- AI responses
- YouTube video references

I used Streamlit because the project is mainly Python and AI based, so it allows the application to be built quickly without creating a separate frontend.

### LangChain

LangChain is used to connect the different components of the RAG system.

It connects:

- Vector database
- Retriever
- Conversation memory
- LLM

The project uses `ConversationalRetrievalChain` for the conversational RAG workflow.

### Llama 3.3 70B

Llama 3.3 70B is the language model used to generate the final answers.

It receives the user's question along with the relevant textbook content retrieved from the vector database.

The model is accessed through Groq.

### Groq

Groq is used for running the Llama model.

It provides fast inference, which helps the chatbot generate responses quickly.

The project uses:

```text
llama-3.3-70b-versatile
```

The temperature is set to `0` to make the responses more consistent and less random.

### Hugging Face Embeddings

Hugging Face embeddings are used to convert textbook text and user questions into numerical vectors.

For example:

```text
"What is photosynthesis?"
        |
        v
Embedding Model
        |
        v
Numerical Vector
```

These vectors are used to find text with similar meaning.

### ChromaDB

ChromaDB is used as the vector database.

It stores the embeddings of the textbook chunks and is used to retrieve relevant content when the student asks a question.

The project creates:

- Subject-level vector databases
- Chapter-level vector databases

### UnstructuredFileLoader

`UnstructuredFileLoader` is used to load and extract text from PDF files.

The textbook PDFs are processed before being stored in the vector database.

### CharacterTextSplitter

`CharacterTextSplitter` is used to divide large textbook content into smaller chunks.

The project uses:

```text
Chunk size: 2000 characters
Chunk overlap: 500 characters
```

Chunking makes the documents easier to search and retrieve.

The overlap helps preserve context between neighboring chunks.

### ConversationBufferMemory

`ConversationBufferMemory` is used to maintain the conversation history.

For example:

```text
Student: What is DNA?

Learnora: DNA is...

Student: What is its structure?
```

The chatbot can understand that "its" refers to DNA because the previous conversation is available.

### YouTubeSearchPython

The `youtubesearchpython` library is used to search YouTube.

The application searches for relevant videos based on the user's questions and displays three video titles and links.

### python-dotenv

`python-dotenv` is used to load environment variables from the `.env` file.

It is used for configuration values such as the device and subject name, and can also be used to keep API keys outside the source code.

---

# RAG

RAG stands for Retrieval-Augmented Generation.

It is the main concept used in Learnora.

Instead of directly sending the student's question to the LLM, the system first searches the textbook content for relevant information.

The retrieved information is then given to the LLM along with the question.

The LLM uses this information to generate the final answer.

## Why RAG?

A general LLM has general knowledge, but Learnora is designed to answer questions based on Class 12 textbook content.

RAG helps the application provide more relevant textbook-based answers.

The basic process is:

```text
Student Question
       |
       v
Search Textbook Knowledge
       |
       v
Retrieve Relevant Content
       |
       v
Send Content + Question to LLM
       |
       v
Generate Answer
```

---

# RAG Pipeline in Learnora

The RAG system has two main stages.

## 1. Data Preparation

This stage is performed before the student asks questions.

```text
PDF
 |
 v
Extract Text
 |
 v
Split Text into Chunks
 |
 v
Generate Embeddings
 |
 v
Store in ChromaDB
```

## 2. Question Answering

This happens when the student uses the application.

```text
User Question
 |
 v
Question Embedding
 |
 v
ChromaDB Search
 |
 v
Retrieve Relevant Chunks
 |
 v
Conversation History
 |
 v
LangChain
 |
 v
Llama 3.3 70B
 |
 v
AI Answer
```

---

# Text Processing

The textbook PDFs are loaded using `UnstructuredFileLoader`.

After extracting the text, the text is divided into chunks.

The project uses:

```text
Chunk size = 2000
Chunk overlap = 500
```

For example:

```text
PDF
 |
 +-- Chunk 1
 |
 +-- Chunk 2
 |
 +-- Chunk 3
 |
 +-- Chunk 4
```

The overlap helps maintain context between chunks.

---

# Embeddings

After splitting the textbook into chunks, each chunk is converted into an embedding using a Hugging Face embedding model.

An embedding is a numerical representation of the meaning of text.

This allows the application to compare the meaning of the student's question with the meaning of the textbook content.

---

# Vector Database

The generated embeddings are stored in ChromaDB.

The process is:

```text
Text Chunk
   |
   v
Embedding Model
   |
   v
Vector
   |
   v
ChromaDB
```

When the user asks a question, the question is also converted into an embedding.

ChromaDB then finds the most relevant textbook chunks.

---

# MMR Retrieval

Learnora uses MMR retrieval.

MMR stands for:

```text
Maximal Marginal Relevance
```

The project uses:

```python
search_type="mmr"
search_kwargs={"k": 3}
```

This means the retriever tries to find relevant and less repetitive information.

`k=3` means the system retrieves the top three relevant chunks.

Instead of retrieving three almost identical chunks, MMR tries to provide useful and diverse information.

---

# Subject and Chapter Vector Databases

Learnora supports both subject-level and chapter-level retrieval.

For example:

```text
Biology
 |
 +-- Chapter 1 Vector DB
 +-- Chapter 2 Vector DB
 +-- Chapter 3 Vector DB
 +-- ...
```

A complete subject database is also created:

```text
Class 12 Biology
 |
 +-- All Chapters Vector DB
```

If the student selects a specific chapter, the application uses that chapter's vector database.

If the student selects "All Chapters", the complete subject vector database is used.

This helps make the retrieved content more focused.

---

# Complete Architecture

```text
                         LEARNORA
                            |
             -------------------------------
             |                             |
       Data Preparation               User Application
             |                             |
             v                             v
       Class 12 PDFs                  Streamlit UI
             |                             |
             v                             v
    PDF Text Extraction             Select Subject
             |                             |
             v                             v
       Text Chunking                  Select Chapter
             |                             |
             v                             v
   Hugging Face Embeddings            Ask Question
             |                             |
             v                             v
          ChromaDB                 Question Embedding
             |                             |
             |                             v
             |                       ChromaDB
             |                             |
             |                             v
             |                      MMR Retrieval
             |                             |
             |                             v
             |                      Top 3 Chunks
             |                             |
             |                             v
             |                    Conversation Memory
             |                             |
             |                             v
             |                        LangChain
             |                             |
             |                             v
             |                    Groq / Llama 3.3
             |                             |
             |                             v
             |                         AI Answer
             |                             |
             |                     ----------------
             |                     |              |
             |                     v              v
             |                Chat Output    YouTube Search
             |                                    |
             |                                    v
             |                              3 Video Links
             |
             +-------------------------------------------+
```

---

# Complete Workflow

Suppose a student selects:

```text
Subject: Biology
Chapter: A specific chapter
Question: Explain photosynthesis.
```

The process is:

```text
1. Student selects Biology.

2. Student selects a chapter.

3. Learnora loads the vector database
   for the selected chapter.

4. Student asks a question.

5. The question is converted into an embedding.

6. ChromaDB searches for similar textbook content.

7. MMR retrieves the top 3 relevant chunks.

8. The retrieved chunks are combined with
   the conversation history.

9. LangChain sends the information to
   Llama 3.3 70B through Groq.

10. Llama generates the answer.

11. Streamlit displays the answer.

12. The user's questions are also used for
    YouTube search.

13. Three relevant YouTube videos are displayed.
```

---




# Why These Technologies?

## Why Streamlit?

It is simple and works directly with Python. It was enough for building the chatbot interface without creating a separate frontend.

## Why LangChain?

It makes it easier to connect the retriever, vector database, conversation memory and LLM.

## Why ChromaDB?

It is lightweight, easy to use with Python and integrates well with LangChain.

## Why Hugging Face Embeddings?

They provide pretrained embedding models that can convert text into vectors and can be used without requiring a separate embedding API.

## Why Llama 3.3 70B?

It provides strong language understanding and is used to generate the final response from the retrieved textbook context.

## Why Groq?

Groq provides fast inference, which is useful for an interactive chatbot.

## Why RAG?

Because the chatbot needs to answer questions based on specific textbook content rather than only the general knowledge of the LLM.

## Why YouTube?

Some students understand concepts better through visual explanations, so the application provides additional video resources.

---

# Applications

Learnora can be used for:

- School education
- Student doubt solving
- Textbook-based question answering
- Chapter revision
- Exam preparation
- Self-learning
- Concept explanation
- Video-based learning

The same system can also be extended to other classes, subjects, courses and educational documents.

---

# Limitations

- Answer quality depends on the quality of the textbook PDFs.
- Poor PDF text extraction can affect retrieval.
- RAG reduces hallucination but cannot completely remove it.
- YouTube results depend on the available search results.
- The current project supports a limited number of subjects.
- Local ChromaDB is suitable for the current project but may need a scalable vector database for a large production application.

---

# Future Goals

The future version of Learnora can include:

- More classes and subjects
- Student login and profiles
- Student progress tracking
- Automatic quiz generation
- MCQ generation
- Personalized study plans
- Practice tests
- Better source references
- Voice-based questions
- Multilingual support
- Teacher dashboard
- Student performance analytics
- Cloud-based vector database
- Scalable deployment

---

# Future Vision

The goal is to make Learnora a complete AI learning assistant instead of only a question-answering chatbot.

The future system can provide:

```text
Ask Question
     |
     v
AI Answer
     |
     +----> Relevant Videos
     |
     +----> Quiz
     |
     +----> Practice Questions
     |
     +----> Study Plan
     |
     +----> Progress Tracking
     |
     +----> Personalized Learning
```

---

# Summary

Learnora uses RAG to connect Class 12 textbook content with an LLM.

The main workflow is:

```text
PDF
  ↓
Text Extraction
  ↓
Text Chunking
  ↓
Hugging Face Embeddings
  ↓
ChromaDB
  ↓
User Question
  ↓
MMR Retrieval
  ↓
Relevant Text
  ↓
LangChain
  ↓
Llama 3.3 70B through Groq
  ↓
AI Answer
  ↓
YouTube Recommendations
```

The main goal of Learnora is to provide students with a simple platform where they can ask questions from their study material, receive AI-based explanations, and find additional learning resources.

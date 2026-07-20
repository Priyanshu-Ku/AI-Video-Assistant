# 🎬 AI Video Assistant

**Transcribe · Summarise · Chat with your Videos**

A production-ready, fully local AI tool that takes audio from YouTube URLs or uploaded files, transcribes it, translates Hindi/Hinglish speech to English, generates a structured video summary with action items and key decisions, and lets users chat with their video transcript through a Retrieval-Augmented Generation (RAG) pipeline.

`Python` `LangChain` `Whisper` `Mistral AI` `ChromaDB` `Streamlit`

---

## 📌 The Problem

Every professional sits through hours of recorded videos and calls every week — but most of the value gets lost the moment the video ends.

- **No Notes Taken** — most people forget 90% of what was discussed within an hour of a video ending
- **Action Items Lost** — tasks assigned during videos often go untracked because nobody wrote them down properly
- **Expensive Tools** — tools like Otter.ai and Fireflies charge $20–$40/month just for basic transcription and summaries
- **No Way to Search Back** — it's nearly impossible to find what was said 3 weeks ago without re-watching the entire recording

## ✅ The Solution

A fully local, completely free AI assistant that handles everything after a video ends.

| Feature | Description |
|---|---|
| 🎙️ **Input Any Audio** | YouTube URL, MP4, MP3, WAV — paste a link or upload a file |
| 🤖 **Auto Transcribe** | Whisper AI runs locally on your machine — no API cost, ever |
| 📄 **Smart Summary** | Mistral AI creates a clean bullet-point summary of the full video |
| 🧠 **Extract Action Items** | Every task, decision, and open question is pulled out automatically |
| 🗂️ **RAG-Powered Q&A** | ChromaDB stores the transcript so you can ask questions anytime |
| ⬇️ **Export Results** | Download the summary/report as a PDF or file on-demand |

> Hindi & Hinglish supported · Works offline · Zero subscription cost

---

## 🛠️ Tech Stack

| Tool | Category | Role |
|---|---|---|
| **OpenAI Whisper** | Transcription | Runs 100% locally, supports Hindi, English, and Hinglish |
| **LangChain (LCEL)** | AI Framework | Modern Runnable-chain pipeline syntax — no legacy chains |
| **Mistral AI** | LLM (Free API) | Summarisation, extraction, and translation — no credit card needed |
| **ChromaDB** | Vector Database | Stores transcript embeddings locally; enables semantic search + RAG chat |
| **HuggingFace (all-MiniLM-L6-v2)** | Embeddings | Converts text chunks into vectors for ChromaDB, runs locally |
| **yt-dlp + ffmpeg** | Audio Processing | Downloads YouTube audio only and converts any format to WAV |
| **Streamlit** | UI Framework | Full browser UI in pure Python — chat interface, tabs, export |
| **fpdf2** | PDF Export | Generates the downloadable summary/report as a PDF |
| **python-dotenv** | Config | Environment variable management (API keys, model size, etc.) |

> Everything runs locally — no cloud costs, no data sent to external servers (aside from the free Mistral summarisation API call).

---

## 🏗️ System Architecture

**Phase 1 — Core Pipeline**
```
Audio Input → Audio Processor → Whisper Transcribe → Translator → Mistral LLM → Output Results
 (YouTube/File)   (yt-dlp+ffmpeg)    (Local Model)     (Hindi→English)  (Free API)  (Summary+Tasks)
```

**Phase 2 — RAG Pipeline**
```
Transcript → Text Splitter → Embeddings → ChromaDB → RAG Chat
             (Chunking)      (HuggingFace,      (Vector      (Ask questions
                               local)             Store)       about the video)
```

**Phase 3 — Streamlit UI** wraps both pipelines into a single interactive web app.

---

## 📁 Folder Structure

```
ai-video-assistant/
├── core/                    # Logic files — all AI processing
│   ├── __init__.py
│   ├── transcriber.py       # Runs Whisper transcription
│   ├── translator.py        # Hindi/Hinglish → English translation
│   ├── summarizer.py        # Generates bullet-point video summary
│   ├── extractor.py         # Extracts action items, decisions, questions
│   ├── vector_store.py      # ChromaDB setup for RAG
│   └── rag_engine.py        # LCEL RAG chain for chat Q&A
├── utils/                   # Helper files
│   ├── audio_processor.py   # Audio downloading, chunking, format conversion
│   └── file_handler.py      # File handling utilities
├── main.py                  # CLI entry point — ties all core modules together
├── app.py                   # Streamlit UI, built on top of main.py
├── .env                      # Environment variables
└── requirements.txt
```

- Every module is independent — easy to understand, test, and reason about in isolation.

---

## 🔍 Module Breakdown

### `utils/audio_processor.py` — Audio Processing
**Purpose:** Accepts any audio/video source and converts it to WAV chunks ready for Whisper.
- Detects if input is a YouTube URL or a local file
- Downloads audio-only stream from YouTube using `yt-dlp`
- Converts any format (MP4, MP3, M4A) to WAV using `ffmpeg`
- Splits large audio files into 10-minute chunks for Whisper
- Returns a list of chunk file paths to the next step

**IN:** YouTube URL or local file path, any format (MP4/MP3/WAV/M4A)
**OUT:** List of WAV chunk paths, `downloads/` folder with audio

### `core/transcriber.py` — AI Transcription
**Purpose:** Runs OpenAI Whisper locally to convert audio chunks into text.
- Loads the Whisper model once into memory (tiny/small/medium/large)
- Transcribes each audio chunk sequentially
- Supports translate mode — converts Hindi speech directly to English text
- Combines all chunk transcripts into one clean transcript
- Model size configurable via `.env` — balances speed vs accuracy

**IN:** List of WAV chunk paths, language mode (english/hindi)
**OUT:** Full video transcript, optional translated English text

### `core/translator.py` — Hindi Translation
**Purpose:** Translates Hindi or Hinglish transcript to clean English using Mistral via LangChain.
- Takes raw Hindi/Hinglish transcript as input
- Builds a LangChain LCEL chain using `ChatPromptTemplate`
- Sends transcript to the Mistral free API for professional translation
- Preserves technical terms, names, and numbers as-is
- Returns clean English text ready for summarisation

**IN:** Hindi/Hinglish transcript text
**OUT:** Clean professional English text

### `core/summarizer.py` — Video Summarisation
**Purpose:** Splits transcript into chunks and summarises using LangChain LCEL + Mistral.
- Splits long transcripts using `RecursiveCharacterTextSplitter`
- **Map step** — summarises each chunk independently
- **Combine step** — merges all chunk summaries into one final summary
- Generates a short auto video title (max 8 words)
- Uses a pure LCEL Runnable pipeline — no legacy LangChain chains

**IN:** Full transcript (any length)
**OUT:** Bullet-point summary, auto-generated title

### `core/extractor.py` — Information Extraction
**Purpose:** Extracts structured information from the transcript using reusable LCEL chains.
- Defines a reusable `build_chain()` function — avoids code repetition
- Extracts action items with owner name and deadline for each task
- Extracts key decisions made during the video
- Extracts open questions and unresolved follow-up topics
- All three extraction types use the same LCEL pattern

**IN:** Full transcript
**OUT:** Numbered action items, key decisions, open questions

### `main.py` — CLI Entry Point
**Purpose:** Connects every module and runs the full pipeline end-to-end.
- Imports all `core` and `utils` modules
- Runs the full pipeline in order: audio → transcribe → translate → LLM
- Stores all results in a clean Python dictionary
- Includes an interactive chat loop for RAG Q&A after the pipeline completes
- Works as both a standalone CLI tool and a module imported by `app.py`

**IN:** YouTube URL or file path, language choice
**OUT:** `title`, `summary`, `action_items`, `decisions`, `questions`, `rag_chain`

---

## 🧠 What is RAG?

**Retrieval Augmented Generation** — the technique that lets the LLM answer questions using *your* video content instead of relying on its training data alone.

| Without RAG | With RAG |
|---|---|
| User asks about the video | User asks about the video |
| LLM has no context about your video | ChromaDB finds the relevant transcript chunks |
| LLM says "I don't have that information" | Mistral reads only those chunks |
| ...or worse, it hallucinates an answer | Answer is grounded in the actual transcript |

---

## 🖥️ Interface

The Streamlit UI provides a sidebar for pasting a YouTube URL or file path, selecting a language, and hitting **Analyse**. Results are organized into three tabs: **Transcription**, **Summarisation**, and **RAG Chat**.

---

## 🚀 Roadmap

- **FastAPI Backend** — wrap `core/` and `utils/` in a REST API so the frontend calls real endpoints via `fetch()` instead of relying on the Streamlit-only flow
- **Async Job Queue** — since transcription can take several minutes, move to a background-job pattern: `POST /analyze` kicks off a job and returns a `job_id` immediately, while the frontend polls `GET /status/{job_id}` to drive a live progress bar
- **Deployment** — containerize the app for one-click deployment

---

## ⚙️ Getting Started

```bash
git clone https://github.com/<your-username>/ai-video-assistant.git
cd ai-video-assistant
pip install -r requirements.txt
cp .env.example .env   # add your Mistral API key
streamlit run app.py
```

---

## 📄 License

MIT
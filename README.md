# 🎥 YouTube RAG — Chat With Any Video

Ask natural-language questions about a YouTube video and get grounded answers,
generated from the video's own transcript using Retrieval-Augmented Generation (RAG).
Supports multi-turn conversations, so follow-up questions ("what about after that?") work.

**Pipeline:** YouTube Transcript → Chunking → BM25 Keyword Retrieval (or direct timestamp
lookup) → LLM (Groq, with conversation history) → Timestamped Answer

## Features

- 🔎 **BM25 keyword retrieval** over transcript chunks — no vector DB required
- ⏱️ **Timestamp-aware questions** — "what did they say at 3:29?" bypasses retrieval
  entirely and jumps straight to that point in the transcript
- 💬 **Conversation memory** — follow-up questions can refer back to earlier turns
- ⚡ **Groq-powered LLM** for fast inference
- 🖥️ **Optional Gradio chat UI** for interactive use

## Demo

Run the notebook top to bottom, or launch the Gradio demo in the last section for an
interactive chat interface.

## Setup

1. Clone the repo and install dependencies:

   ```bash
   git clone https://github.com/official-naveen/youtube-rag.git
   cd youtube-rag
   pip install -r requirements.txt
   ```

2. Get a free Groq API key from [console.groq.com/keys](https://console.groq.com/keys).

3. Copy `.env.example` to `.env` and add your key:

   ```bash
   cp .env.example .env
   ```

   (If you're on Colab or Kaggle, you can instead store `GROQ_API_KEY` as a secret and
   the notebook will pick it up automatically. Otherwise it will prompt you securely.)

4. Open `YouTube_RAG.ipynb` and run the cells.

## Usage

```python
response = ask_video(
    question="What is this video about?",
    video="https://www.youtube.com/watch?v=xAt1xcC6qfM",
)
print(response)
```

Timestamp questions work the same way:

```python
response = ask_video(
    question="What did they say at 3 minutes 29 seconds?",
    video="https://www.youtube.com/watch?v=xAt1xcC6qfM",
)
```

And follow-ups, by passing `chat_history`:

```python
chat_history = [("What did they say at 3:29?", response)]
follow_up = ask_video(
    question="Can you say more about that?",
    video="https://www.youtube.com/watch?v=xAt1xcC6qfM",
    chat_history=chat_history,
)
```

## How it works

1. The video's transcript is fetched via `youtube-transcript-api` and split into
   overlapping chunks with timestamps preserved.
2. If the question references a specific time (`3:29`, `90 seconds`, etc.), the
   matching chunk is looked up directly — BM25 is skipped, since keyword matching
   can't reason about time.
3. Otherwise, the question is matched against chunks with BM25 keyword retrieval.
4. The top chunks, plus recent conversation history, are passed to a Groq-hosted LLM
   via LangChain, which answers using only the transcript content.

## Project structure

```
.
├── YouTube_RAG.ipynb   # main notebook — the whole pipeline
├── requirements.txt    # Python dependencies
├── .env.example        # template for your Groq API key
└── LICENSE
```

## Requirements

- Python 3.9+
- A [Groq API key](https://console.groq.com/keys) (free tier available)

## License

MIT — see [LICENSE](LICENSE).

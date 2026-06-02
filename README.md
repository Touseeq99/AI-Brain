# 🧠 AI Brain

> A multi-feature AI playground built on Google Gemini — combining a stateful chatbot, image understanding, text embedding, and open-ended Q&A in a single Streamlit app.

---

## 🔴 The Problem

Learning a new AI SDK means writing a lot of throwaway scripts — one for chat, one for vision, one for embeddings. Context switches between files, no unified interface, and nothing to show at the end.

The secondary problem: most Gemini tutorials show you *one* capability in isolation. But Gemini is a multimodal model — its real power comes from using vision, text, and embeddings together.

---

## ✅ The Solution

**AI Brain** is a single Streamlit app with four fully functional modules, each exposing a different Gemini capability through a clean sidebar navigation:

| Module | Capability |
|---|---|
| 🤖 ChatBot | Stateful multi-turn conversation with Gemini Pro |
| 📸 Image Bot | Visual question answering — upload an image, ask anything about it |
| 🔠 Embed Text | Generate text embeddings using Google's `embedding-001` model |
| ❔ Ask Anything | Single-turn open Q&A with Gemini Pro |

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────┐
│              Streamlit Frontend               │
│                                               │
│  Sidebar (streamlit-option-menu)              │
│  ├── CHATBOT                                  │
│  ├── IMAGE BOT                                │
│  ├── EMBED TEXT                               │
│  └── ASK ANYTHING                            │
└───────────────────┬──────────────────────────┘
                    │ routes to
                    ▼
┌──────────────────────────────────────────────┐
│           gemini_utilities.py                 │
│           (Model Abstraction Layer)           │
│                                               │
│  load_gemini_ai_model()                       │
│  └── GenerativeModel("gemini-pro")            │
│      └── Used for: ChatBot, Ask Anything      │
│                                               │
│  load_gemini_vision_model(prompt, image)      │
│  └── GenerativeModel("gemini-1.5-flash")      │
│      └── Used for: Image Bot                  │
│                                               │
│  embedding_model(input_text)                  │
│  └── models/embedding-001                     │
│      └── Used for: Embed Text                 │
│                                               │
│  ask_gemini_model(input_text)                 │
│  └── GenerativeModel("gemini-pro")            │
│      └── Used for: Ask Anything               │
└───────────────────┬──────────────────────────┘
                    │
                    ▼
          Google Gemini API
```

### Session State Flow (ChatBot)

```
User sends message
      │
      ▼
st.session_state.chat_session  ←── persists across reruns
      │
      ▼
model.start_chat(history=[])   ←── initialized once
      │
      ▼
chat_session.send_message()    ←── appends to history automatically
      │
      ▼
Render full history on each rerun
```

---

## 🤔 Why I Chose What I Chose

### Separating models into `gemini_utilities.py`
Rather than initializing models directly in the UI file, all model logic lives in a separate utilities module. This keeps `main.py` focused on UI concerns and makes it easy to swap models (e.g. upgrade from `gemini-pro` to `gemini-1.5-pro`) in one place without touching the interface.

### `gemini-pro` for text, `gemini-1.5-flash` for vision
These are two different models for two different jobs. `gemini-pro` handles text-only tasks (chat, Q&A). `gemini-1.5-flash` is multimodal — it accepts image + text input natively, which is required for the Image Bot. Flash was chosen over Pro Vision for speed and cost efficiency; image captioning doesn't need deep reasoning, just accurate visual understanding.

### `embedding-001` for embeddings
Google's dedicated embedding model produces vector representations optimized for retrieval tasks (`task_type="retrieval_document"`). This is the foundation for building semantic search or RAG pipelines — the Embed Text module demonstrates the raw output of this step, making it a useful learning tool even in isolation.

### Stateful chat via `model.start_chat(history=[])`
Instead of manually maintaining a message list and passing the full history on each call, the Gemini SDK's `start_chat()` method handles history internally. `st.session_state` preserves the session object across Streamlit reruns, giving the chatbot genuine multi-turn memory within a session.

### `streamlit-option-menu` for navigation
The default Streamlit sidebar uses radio buttons for navigation, which looks basic. `streamlit-option-menu` renders a proper icon-based menu with labels — a small detail that significantly improves the demo experience.

---

## 🚀 Getting Started

### Prerequisites
- Python 3.9+
- A [Google AI Studio](https://aistudio.google.com/app/apikey) API key

### Installation

```bash
git clone https://github.com/Touseeq99/Application-using-Gemini-Pro.git
cd Application-using-Gemini-Pro
pip install -r requirements.txt
```

### Environment Setup

Create a `.env` file in the root directory:

```env
GOOGLE_API_KEY=your_api_key_here
```

Then update `gemini_utilities.py` to load from environment:

```python
from dotenv import load_dotenv
load_dotenv()
Google_API_key = os.getenv("GOOGLE_API_KEY")
```

> ⚠️ Do not store API keys in `config.json` or any file committed to version control.

### Run

```bash
streamlit run main.py
```

Open `http://localhost:8501` in your browser.

---

## 📸 Features

### 🤖 ChatBot
Multi-turn conversation with full history. Each message and response is preserved in the session — ask follow-up questions and the model remembers context.

### 📸 Image Bot
Upload any image (JPG/PNG) and ask a question about it. The model uses Gemini 1.5 Flash's vision capability to analyze the image and respond to your prompt. The image and response are displayed side by side.

### 🔠 Embed Text
Enter any text and generate its vector embedding using Google's `embedding-001` model. Useful for understanding how text is represented semantically — a foundation for building search, clustering, or RAG systems.

### ❔ Ask Anything
Single-turn, open-ended question answering. No conversation history — just a prompt and a response.

---

## 📁 Project Structure

```
Application-using-Gemini-Pro/
├── main.py                 # Streamlit UI — routing and all four modules
├── gemini_utilities.py     # Model abstraction layer — all Gemini calls
├── requirements.txt        # Python dependencies
└── README.md
```

---

## 🛠️ Built With

- [Google Gemini API](https://ai.google.dev/) — gemini-pro, gemini-1.5-flash, embedding-001
- [Streamlit](https://streamlit.io/) — Web UI
- [streamlit-option-menu](https://github.com/victoryhb/streamlit-option-menu) — Sidebar navigation
- [Pillow](https://pillow.readthedocs.io/) — Image handling

---

## 👤 Author

**Touseeq Ahmed**  
AI Engineer | [GitHub](https://github.com/Touseeq99)

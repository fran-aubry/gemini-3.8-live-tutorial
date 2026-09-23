# Gemini 3.8 Live Voice Assistant Tutorial

This repository contains the interactive tutorial notebook [`voice_assistant.ipynb`](./voice_assistant.ipynb) demonstrating how to build a full-duplex, low-latency voice assistant using Google's **Gemini 3.8 Live API** (`gemini-3.8-live` and `gemini-3.8-live-extended-thinking`).

The assistant features bidirectional streaming audio over WebSockets, real-time barge-in interruptions, and non-blocking asynchronous tool calling.

---

## Prerequisites

- **Python 3.12** (Conda/Miniconda recommended)
- **Local Audio Hardware**: A working microphone and speaker (since audio I/O is streamed locally via `sounddevice`, this notebook must be run locally in JupyterLab, Jupyter Notebook, or VS Code rather than hosted cloud environments like Google Colab).
- **Gemini API Key**: From [Google AI Studio](https://aistudio.google.com/) with billing enabled.

---

## Setup Instructions

### 1. Create and Activate the Conda Environment

Create a new Conda environment with Python 3.12 and activate it:

```bash
conda create -yn gemini-3-8-live python=3.12
conda activate gemini-3-8-live
```

### 2. Install Dependencies

Install the required Python packages:

```bash
pip install -r requirements.txt
```

*(Optional) If you don't already have Jupyter installed in your environment:*
```bash
pip install jupyter
```

### 3. Configure Environment Variables

1. Copy `.env.template` to `.env`:

   ```bash
   cp .env.template .env
   ```

2. Open `.env` and set your Gemini API key:

   ```env
   GEMINI_API_KEY=your_actual_gemini_api_key
   ```

---

## Running the Notebook

Start Jupyter Notebook or JupyterLab:

```bash
jupyter notebook voice_assistant.ipynb
```

You can also open [`voice_assistant.ipynb`](./voice_assistant.ipynb) directly in VS Code or Cursor with the Python & Jupyter extensions.

---

## Notebook Structure

The tutorial in [`voice_assistant.ipynb`](./voice_assistant.ipynb) walks step-by-step through building a production-ready voice assistant:

1. **Step 1: Environment Setup & Client Initialization**
   - Load environment variables from `.env` and initialize the Google GenAI SDK client (`genai.Client`).

2. **Step 2: First Live Session Request**
   - Connect to the Live API WebSocket session, configure `AUDIO` response modalities, and stream back real-time audio and transcription.

3. **Step 3: Real-Time Audio Player Worker**
   - Implement an asynchronous worker (`audio_player`) using `sounddevice.RawOutputStream` and an `asyncio.Queue` for low-latency 24kHz audio playback.

4. **Step 4: Microphone Recorder Worker**
   - Capture microphone input at 16kHz PCM mono using `sounddevice.RawInputStream` and stream it into an input queue.

5. **Step 5: Upstream Audio Streaming Loop**
   - Implement `send_audio_loop` to continuously send recorded microphone chunks to the Gemini Live session over WebSockets.

6. **Step 6: Downstream Receiving Loop & Barge-In Handling**
   - Implement `receive_loop` to process model audio chunks and transcripts.
   - Handle barge-in interruptions (`server_content.interrupted`) by immediately clearing the playback queue to stop model speech when you interrupt.
   - Track session completion using `server_content.interaction_status == "IDLE"`.

7. **Step 7: Full Bidirectional Voice Assistant**
   - Combine the player, recorder, send loop, and receive loop into `run_voice_assistant()`, managing concurrent async tasks and graceful shutdowns.

8. **Tools & Extended Thinking**
   - **Function Calling**: Implement live weather fetching via the Open-Meteo REST API (`get_current_weather`).
   - **Non-Blocking Execution**: Launch tool execution as a background task (`asyncio.create_task`) so audio streaming and filler speech are never blocked.
   - **Thinking in Action**: Compare `gemini-3.8-live-extended-thinking` (with `thinking_config`) against `gemini-3.8-live` to hear how extended thinking allows the model to speak conversational fillers while tools run in parallel.
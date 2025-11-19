````markdown
# AI Engineer Basic Challenge  
## Tool-Calling AI Chat App (RAG + 4 Capabilities)

### 1. Overview

We’d like you to build a **basic AI chat application** that connects to **OpenAI** and supports:

1. Normal chat with an LLM  
2. Question-answering over **user-uploaded files** (RAG) using **ChromaDB**  
3. Four optional capabilities that the user can **toggle on/off** from a “Chat Controls” popup:
   - Web Search  
   - Image Generation  
   - Data Analysis  
   - Think (enhanced reasoning mode)

You should design a simple **agent** that uses an explicit **system prompt + function calling / tools** so the model can decide when to use each capability based on the toggles and user queries.

This is an **AI Engineer basic challenge**: we care about how you design the agent, tools, and system prompts, as much as the frontend and backend structure.

---

### 2. Tech Stack Requirements

Please use the following stack:

- **Frontend:** React **or** Vue.js  
- **Backend:** Python **FastAPI**  
- **Vector Store for RAG:** **ChromaDB** (local is fine)  
- **LLM:** OpenAI Chat Completions with tools / function calling  
- **Storage:** In-memory or simple local storage is fine (this is not meant to be production-grade)

You may use any UI libraries/components you like, as long as the required elements below are present.

---

### 3. Core Features

#### 3.1 Chat Interface

- A main chat area showing messages between:
  - `user`
  - `assistant`
- A chat input area at the bottom:
  - Text input
  - Send button (or Enter key to send)

#### 3.2 File Upload + RAG (ChromaDB)

- Above the chat input, show an **“Add file”** button or icon.
- Allow the user to upload **multiple files** (e.g., PDF, TXT, or Markdown).
- When files are uploaded:
  - Chunk them into smaller pieces.
  - Embed and store them in **ChromaDB**.
  - Associate the data with the current chat/session.
- The agent should be able to **retrieve relevant chunks** from ChromaDB to answer questions about the uploaded content.

#### 3.3 Settings / Chat Controls Popup (4 Capabilities)

- Next to the “Add file” button, show a **Settings** icon/button.
- When clicked, it should open a small **“Chat Controls” popup** with exactly **four toggles**:

1. **Web Search** – toggle  
2. **Image Generation** – toggle  
3. **Data Analysis** – toggle  
4. **Think** (reasoning mode) – toggle  

- Each row should show:
  - Capability name
  - 1-line description
  - On/Off toggle
- The current toggle states must be reflected in a `settings` object that is sent to the backend with every `/chat` request.

The implementation doesn’t have to be pixel-perfect, but it should be **functionally similar** to:

- Chat area below
- “Add file” and “Settings” above the chat box
- A popup showing the 4 capabilities when Settings is clicked

---

### 4. Agent Design & Tools

You are free to design the internal agent however you like, but it must satisfy:

#### 4.1 System Prompt

- Build a **system prompt** that explains to the model:
  - It is an AI assistant with access to tools.
  - The available tools (RAG, Web Search, Image Generation, Data Analysis).
  - The current `settings` (which tools are enabled vs disabled).
  - It should **only** call tools that are currently enabled.

#### 4.2 Tools / Function Calling

- Use **OpenAI tools / function calling** (or an equivalent mechanism) to represent the capabilities.
- At minimum, you should define tools for:
  - `rag_retrieve` – query ChromaDB and return relevant document chunks.
  - `web_search` – perform a web search (can be real or stubbed with mock data).
  - `image_generation` – generate an image (can return a placeholder URL or real image URL).
  - `data_analysis` – analyze user data (e.g., CSV-like content; can be simplified).

- The agent logic should:
  - Read the current `settings`.
  - Decide when to call a tool vs answer directly.
  - Not call tools that are disabled in `settings`.

You can define your own JSON schemas; here is a simplified example for RAG:

```json
{
  "name": "rag_retrieve",
  "description": "Retrieve relevant chunks from user-uploaded documents.",
  "parameters": {
    "type": "object",
    "properties": {
      "query": { "type": "string", "description": "The user question or search query." }
    },
    "required": ["query"]
  }
}
````

---

### 5. RAG Requirements (ChromaDB)

* Use ChromaDB as your **vector store**.
* When files are uploaded:

  * Parse content (simplified parsing is fine).
  * Chunk the text (e.g., by paragraphs or fixed token/character length).
  * Embed and insert into ChromaDB with metadata (e.g. session id, filename).
* When the model or backend decides to use RAG:

  * Call your `rag_retrieve` function.
  * Perform similarity search on ChromaDB.
  * Use retrieved chunks as context in the model’s response (e.g., included in the system prompt or tool result).

We are not looking for perfect RAG tuning, only for a **clear, working pipeline** from file upload → ChromaDB → retrieval → better answers.

---

### 6. Capability Behavior

At minimum:

#### 6.1 Web Search

* When **enabled**:

  * For questions that obviously need external knowledge or fresh data, the agent may call `web_search`.
* When **disabled**:

  * The agent should avoid calling web search and answer only from the base model + RAG.

#### 6.2 Image Generation

* When **enabled**:

  * If the user asks for an image (e.g., “generate a logo” or “draw X”), the agent may call `image_generation`.
  * The backend can return:

    * An `image_url`, or
    * A placeholder like `"generated_image.png"` (with explanation in README if you stub it).
* When **disabled**:

  * The agent should not call the image tool, and instead reply text-only.

#### 6.3 Data Analysis

* When **enabled**:

  * The agent may call `data_analysis` for tasks like:

    * “Summarize this CSV”
    * “Calculate totals from this table”
  * You can implement a simple Python-based analysis (e.g., using pandas) or a minimal custom logic.
* When **disabled**:

  * The agent should not call the data analysis tool.

#### 6.4 Think (Reasoning Mode)

* When **enabled**:

  * The agent should provide **more detailed, step-by-step reasoning**.
  * For example, you can:

    * Set a flag in the system prompt (e.g., “think_mode: true”).
    * Ask the model for more structured or multi-step answers.
* When **disabled**:

  * Responses can be shorter and more direct.

This doesn’t need to be perfect; we just want to see that the toggle **changes behavior** in a visible way.

---

### 7. Backend API Contract – `POST /chat`

The frontend should talk to the backend through a single main endpoint:

#### Endpoint

* `POST /chat`

#### Request Body (Example Models)

You can use these Pydantic models or adapt them slightly:

```python
from typing import List, Literal, Optional, Dict, Any
from pydantic import BaseModel

class ChatSettings(BaseModel):
    web_search: bool = True
    image_generation: bool = True
    data_analysis: bool = True
    think: bool = False

class ChatMessage(BaseModel):
    role: Literal["user", "assistant", "system"]
    content: str

class ChatRequest(BaseModel):
    messages: List[ChatMessage]
    settings: ChatSettings
    session_id: Optional[str] = None  # link to uploaded documents
```

* The frontend should send the **full chat history** and the current `settings` on each request.
* `session_id` can be a simple identifier so you can link uploaded documents to a conversation.

#### Response Body (Example)

```python
class ToolCallLog(BaseModel):
    name: str
    arguments: Dict[str, Any]
    success: bool
    error: Optional[str] = None

class ChatResponse(BaseModel):
    assistant_message: ChatMessage
    tool_calls: List[ToolCallLog] = []
    images: List[str] = []  # e.g., image URLs or identifiers
    meta: Dict[str, Any] = {}
```

* `assistant_message` – the final reply to display in the chat.
* `tool_calls` – optional log of which tools were called (for debugging and evaluation).
* `images` – any generated images you want the UI to show.
* `meta` – any extra metadata you find useful.

> **Optional bonus:** implement a `POST /chat/stream` endpoint using SSE to stream tokens. This is a **nice-to-have**, not required.

---

### 8. Error Handling

Please handle the following cases:

* Missing or invalid request body (return a structured error).
* Missing `OPENAI_API_KEY` or failed OpenAI request.
* File-related errors:

  * Unsupported file types
  * File too large (you can define your own simple limits)
* Tool errors (e.g., failed web request or ChromaDB query).

On error:

* Return a friendly error message to the user in the chat.
* Log the error on the server (console logging is fine).

---

### 9. What to Submit

Please provide:

1. **Repository (or zip file)** with at least:

   * `/frontend` (React or Vue)
   * `/backend` (FastAPI)
2. A **README** that explains:

   * How to install dependencies
   * How to start backend and frontend
   * Required environment variables (e.g., `OPENAI_API_KEY`)
   * Any stubs/shortcuts you used (for example, if web search is mocked)
3. (Optional, but helpful) Short notes:

   * What you would improve with more time
   * Any architectural decisions you want to highlight

---

### 10. Evaluation Criteria

We will review your solution based on:

* **Correctness & Coverage**

  * Does it implement the required features?
  * Does RAG with ChromaDB actually work?
  * Do the 4 toggles affect behavior as described?

* **Agent & Tool Design**

  * How you structure the system prompt.
  * How you design and use tools / function calls.
  * How well settings map to tool usage.

* **Code Quality**

  * Project structure and modularity.
  * Readability and maintainability.
  * Clear separation between frontend, backend, and RAG logic.

* **UX & Developer Experience**

  * Is the app easy to run and test?
  * Is the chat UI reasonably intuitive (even if not pixel-perfect)?

---

```

```

# Chat Go CLI - Design Guide

## 1. Overview
`chat_go_cli` is a lightweight, zero-dependency command-line interface (CLI) application written in Go. Its primary purpose is to provide a seamless terminal-based chat experience interacting with Large Language Models (LLMs) via a LiteLLM-compatible API gateway. 

A standout feature of this CLI is its native integration with the **Vertex AI Built-in Google Search Tool**.

## 2. Architecture & Design Principles

### 2.1 Zero Dependencies
The application is intentionally designed to rely exclusively on the Go standard library (`net/http`, `encoding/json`, `bufio`, `os`, etc.). 
*   **Why?** To ensure maximum portability, lightning-fast compilation times, and zero security vulnerabilities from third-party packages. It requires no `npm install` or complex Go dependency resolution.

### 2.2 Server-Side Tool Execution
Rather than manually scraping websites or managing client-side APIs, the CLI delegates tool execution to the AI provider. 
*   **How:** It injects the `{"googleSearch": {}}` capability directly into the API request payload.
*   **Benefit:** The CLI remains lightweight. Complex tasks like executing web searches, parsing HTML, and synthesizing information are handled by the Vertex AI/LiteLLM backend, returning only the final, human-readable text to the client.

### 2.3 Stateful Conversation
The CLI maintains the context of the conversation for the duration of the session.
*   **Implementation:** An in-memory array of `Message` structs (`[]Message`) stores the history of "user" and "assistant" roles. The entire array is serialized into JSON and sent with every new prompt, ensuring the LLM remembers previous context.

## 3. Configuration & Usage

The application is configured dynamically using Environment Variables, conforming to standard cloud-native application practices (12-Factor App).

### Environment Variables

| Variable | Description | Default Fallback |
| :--- | :--- | :--- |
| `LITELLM_URL` | The full HTTP endpoint for the completion API. | `http://localhost:4000/v1/chat/completions` |
| `LITELLM_MASTER_KEY` | The Bearer token/API key for authorization. | *(None)* |
| `LITELLM_MODEL` | The specific model identifier to be invoked. | `gemini-flash` |

### Interactive vs. Non-Interactive Mode

The application supports two modes of operation:
1. **Interactive Mode:** Run without arguments to start an interactive chat session.
   ```bash
   ./chat_cli
   ```
2. **Non-Interactive (Single-Shot) Mode:** Pass the prompt as command-line arguments. The CLI will query the API, print the raw output from the LLM without any prefixes, and immediately exit. This is ideal for shell scripting, piping, and cron jobs.
   ```bash
   ./chat_cli "What is the weather like today?"
   ```

### Running the Application
```bash
LITELLM_URL="http://127.0.0.1:4000/v1/chat/completions" \
LITELLM_MASTER_KEY="your-sk-key" \
LITELLM_MODEL="gemini-flash" \
./chat_cli
```

## 4. API Request/Response Data Structures

The Go code utilizes strictly typed structs to marshal and unmarshal JSON payloads, ensuring memory safety and preventing runtime crashes due to malformed data.

### Request Payload (`ChatRequest`)
```go
type ChatRequest struct {
	Model    string    `json:"model"`
	Messages []Message `json:"messages"`
	Tools    []any     `json:"tools,omitempty"`
}
```
*Note the `Tools` array. This is where the magic `{ "googleSearch": {} }` object is passed.*

### Response Payload (`ChatResponse`)
```go
type ChatResponse struct {
	Choices []struct {
		Message Message `json:"message"`
	} `json:"choices"`
	Error *struct {
		Message string `json:"message"`
	} `json:"error,omitempty"`
}
```

## 5. Control Flow

1.  **Initialization:** The app reads environment variables. If critical endpoints are missing, it falls back to sensible localhost defaults.
2.  **Input Loop:** Uses `bufio.NewScanner` to capture multi-word standard input (`os.Stdin`) from the user until the Enter key is pressed.
3.  **Command Parsing:** Checks for exit commands (`"exit"`, `"quit"`) or processes CLI arguments if running in non-interactive mode.
4.  **Network Request:** Marshals the `[]Message` history and `tools` into JSON and dispatches an HTTP POST request to the Gateway.
5.  **Error Handling:** Validates the HTTP Status code and checks for Gateway-level JSON errors (e.g., token limits exceeded, invalid models).
6.  **Output:** Extracts the content from `chatResp.Choices[0].Message.Content` and prints it to the console, appending it to the history for the next iteration.

## 7. How Tooling Works (Client vs. Orchestrator)

It is important to understand the role this CLI plays regarding tool execution. **This CLI is a "Dumb Client," not an Orchestrator Agent.**

### How the "Dumb Client" handles tools (Our implementation)
In our architecture, the CLI main loop never executes local code for the LLM. 
1. **Permission:** The CLI appends `tools: [{"googleSearch": {}}]` to every request. This simply grants the LLM permission to use the internet if it needs to.
2. **Decision:** The LLM receives the prompt (e.g., "What is the weather today?"). Its internal logic evaluates if it knows the answer or if it needs to search.
3. **Execution:** Because `googleSearch` is a native Vertex AI capability, if the LLM decides to search, the Vertex server itself executes the Google search, reads the results, and synthesizes the answer.
4. **Display:** The Go CLI simply waits and receives the final, fully-formed natural language sentence to print to the screen.

### Contrast: How an "Orchestrator Agent" handles tools
If the CLI were built to handle *local* tools (e.g., `read_local_file`), the main loop would have to act as an orchestrator. That flow looks like this:
1. The CLI sends the prompt and a description of the `read_local_file` tool.
2. The LLM responds **not** with an answer, but with a JSON command indicating it wants to use the tool (a `tool_call`).
3. The CLI intercepts this JSON, pauses the chat, executes a local Go function to read the file, and then sends the file's contents back to the LLM in a new request.
4. The LLM reads the contents and finally generates a natural language answer.

By relying on Vertex's built-in tools, we bypass the complexity of local orchestration entirely, keeping our Go CLI fast and simple.

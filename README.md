# Chat Go CLI

A lightweight, zero-dependency command-line interface (CLI) written in Go, specifically designed to interact with **Google Vertex AI** via a **LiteLLM** proxy gateway.

This tool provides a fast, native terminal chat experience while seamlessly supporting Vertex AI's built-in Google Search tools directly from your command line.

## Features

- **Zero Dependencies:** Built entirely with the Go standard library for maximum portability and fast compilation.
- **Vertex AI via LiteLLM:** Specifically optimized to proxy requests to Google Vertex AI using LiteLLM.
- **Built-in Google Search Support:** Natively requests `googleSearch` tool execution, allowing the Vertex model to fetch real-time data and summarize it for you—all handled server-side.
- **Stateful Sessions:** Keeps conversation context in memory while running, acting like a true conversational terminal agent.

## Prerequisites

- Go 1.22+
- A running instance of [LiteLLM](https://github.com/BerriAI/litellm) configured to route traffic to Vertex AI.

## Installation / Build

Clone the repository and build the binary:

```bash
git clone https://github.com/robertluwang/chat-go-cli.git
cd chat-go-cli
go build -o chat-cli main.go
```

## Configuration

The CLI relies on standard environment variables to connect to your LiteLLM instance. Add the following to your `~/.bashrc`, `~/.zshrc`, or `.env` file:

```bash
# The endpoint where your LiteLLM proxy is running
export LITELLM_URL="http://localhost:4000/chat/completions"

# The master API key for your LiteLLM proxy
export LITELLM_MASTER_KEY="your-sk-key"

# The default model to use (must be mapped in your LiteLLM config)
export LITELLM_MODEL="vertex_ai/gemini-1.5-pro"
```

## Usage

### Interactive Mode
Start the CLI to enter an interactive, stateful chat session:
```bash
./chat-cli
```
*Type your message and press Enter. The conversation history is maintained for the duration of the session. Type `exit` or `quit` to leave.*

### Single-Prompt (One-Shot) Mode
Pass a prompt directly as an argument for a quick response. Great for cron jobs or quick scripts:
```bash
./chat-cli "What is the weather in Tokyo today?"
```

### Overriding the Model
You can override the `LITELLM_MODEL` environment variable on the fly using the `-model` flag:
```bash
./chat-cli -model "vertex_ai/gemini-1.5-flash" "Summarize quantum computing in 3 sentences."
```

## Architecture

For more details on the design principles, zero-dependency architecture, and how tool delegation works, see the [DESIGN_GUIDE.md](./DESIGN_GUIDE.md).

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

### Build for iPhone / iPad (iSH app)

The [iSH app](https://ish.app/) runs a Linux x86 emulator on iOS. To run this CLI natively on your iPhone or iPad, you can cross-compile the binary specifically for the 32-bit x86 architecture used by iSH:

```bash
GOOS=linux GOARCH=386 go build -o chat_cli_ish main.go
```

Then, securely transfer the `chat_cli_ish` binary to your iOS device (e.g., via SCP, wget, or a secure file drop) and make it executable within the iSH terminal.

## Configuration

The CLI relies on standard environment variables. Because background tasks and **cron jobs do not load `.bashrc` or `.zshrc`**, the recommended best practice is to store your configuration in a dedicated `~/.env` file.

1. **Create a `~/.env` file:**
Rename the provided `.env.template` to `~/.env`, fill in your specific values, and source it for your LiteLLM tunnel setup:
```bash
cp .env.template ~/.env
nano ~/.env # add your keys, IP, user, and SSH key
```

2. **For interactive terminal use:**
Add this line to your `~/.bashrc` or `~/.zshrc` so the variables load automatically when you open a terminal:
```bash
source ~/.env
```

3. **For Cron Jobs:**
When scheduling via cron, explicitly source the `.env` file before executing the CLI, because cron runs in a minimal, non-interactive shell:
```bash
0 9 * * * . ~/.env && /absolute/path/to/chat-cli "What is the weather in Tokyo?" >> /tmp/weather.log
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
./chat-cli -model "gemini-flash" "Summarize quantum computing in 3 sentences."
```

## Architecture

For more details on the design principles, zero-dependency architecture, and how tool delegation works, see the [DESIGN.md](./DESIGN.md).

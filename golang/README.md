# Session 5 Starter — Go

Retrieval-augmented generation, plus Docker support. Compiled,
statically typed, no runtime dependency install needed once you've
built it.

## Setup

**With Docker** (no local Go install needed):

```bash
cp .env.example .env
# fill in ANTHROPIC_API_KEY, OPENAI_API_KEY, GLACIER_API_KEY, WALLET_ADDRESS
cd .. && docker compose up -d chroma
docker compose run --rm golang ./rag "What is the C-Chain?"
```

**Without Docker:**

```bash
go mod download
cp .env.example .env
# fill in ANTHROPIC_API_KEY, OPENAI_API_KEY, GLACIER_API_KEY, WALLET_ADDRESS
# start Chroma yourself: docker run -p 8000:8000 chromadb/chroma
```

Requires Go 1.23 or newer if building outside Docker.

## Layout

Go doesn't allow multiple `func main()` in one package, so each
runnable example lives in its own subdirectory, and the shared code
lives in its own packages:

| Path | What it is |
|---|---|
| `modelprovider/` | Provider abstraction package: `NewModelClient()`, four providers, one shared interface |
| `normalize/` | Shared wei-to-AVAX, hex-to-decimal, Unix-to-ISO8601 conversion package |
| `direct-rpc/` | Method 1: raw JSON-RPC over plain HTTP, no SDK at all |
| `chainkit-fetch/` | Method 2: calls the Glacier REST API directly (there is no official ChainKit Go package, see the note in the file) |
| `chainkit-mcp-agent/` | ChainKit as MCP server, using `github.com/mark3labs/mcp-go` |
| `advisor/` | Session 4: the Smart Wallet Advisor, fetch, normalize, summarize, with a human-in-the-loop checkpoint and audit logging |
| `rag/` | Session 5: retrieval-augmented generation, chunk, embed, store, retrieve, generate, grounded answers with citations |

## Running each one

```bash
go run ./direct-rpc
go run ./chainkit-fetch
go run ./chainkit-mcp-agent
go run ./advisor <wallet-address>       # Session 4: the full Smart Wallet Advisor
go run ./rag "your question"            # Session 5: the RAG agent, needs Chroma running
```

For `chainkit-mcp-agent`, start the ChainKit MCP server in another
terminal first (needs Node.js, ChainKit itself is JS-only):

```bash
npx -y @avalanche-sdk/chainkit mcp-server
```

Put the URL it prints into `CHAINKIT_MCP_URL` in your `.env`.

For `rag`, Chroma needs to be running first, either via
`docker compose up -d chroma` from the repo root, or locally with
`docker run -p 8000:8000 chromadb/chroma`.

## A note on embeddings, and why Go needs OPENAI_API_KEY regardless of chat provider

There is no official Chroma Go client. `rag/main.go` calls Chroma's REST
API directly instead, verified against a real running Chroma v2 server
during development, the endpoint paths and request shapes weren't
guessed from documentation. But going through raw REST means embeddings
have to be computed before they're sent, Chroma's server doesn't embed
raw text for you the way the official JS and Python clients do locally.
This file computes embeddings by calling OpenAI's embeddings API, which
means **you need `OPENAI_API_KEY` set even if `MODEL_PROVIDER` is
something else entirely**, embeddings and chat generation are separate
capabilities here.

## Running with Docker

The Dockerfile in this folder is a multi-stage build: it compiles all
five binaries in a full `golang:1.23` image, then copies just the
compiled binaries into a minimal `debian:bookworm-slim` image for the
final result. See the root README for the full `docker compose` quick
start.

## Model provider

`MODEL_PROVIDER` in `.env` picks the provider (`anthropic`, `openai`,
`gemini`, or `ollama`), defaulting to `anthropic`. Only the Anthropic
path implements tool calling, required for `chainkit-mcp-agent`, the
other three are plain text chat, clearly marked as such in
`modelprovider/modelprovider.go`. Note also that the Gemini client here
talks to the REST API directly rather than through Google's official Go
SDK, that SDK couldn't be verified to compile in the environment this
was built in, plain REST avoids the dependency entirely and is
documented as such in the code.

## Submission

1. Test everything yourself, confirm your RAG agent answers correctly from your documents and refuses anything outside them.
2. Screenshot the working test, including at least one grounded answer with a citation, and one correct refusal.
3. Open your PR, screenshot that too.
4. Post on X with both screenshots, tag **@code_mwangi** and **@AvaxAfrica**.
5. Copy your post link, submit it on the quest page once it's live.

Post in the Week 3 WhatsApp group for anything you get stuck on.

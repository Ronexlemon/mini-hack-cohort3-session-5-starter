# Session 5 Starter — Python

Retrieval-augmented generation, plus Docker support. Same pattern as
the JavaScript starter, Python idioms throughout.

## Setup

**With Docker** (no local Python install needed):

```bash
cp .env.example .env
# fill in ANTHROPIC_API_KEY, GLACIER_API_KEY, WALLET_ADDRESS at minimum
cd .. && docker compose up -d chroma
docker compose run --rm python python rag.py "What is the C-Chain?"
```

**Without Docker:**

```bash
python3 -m venv venv
source venv/bin/activate   # or venv\Scripts\activate on Windows
pip install -r requirements.txt
cp .env.example .env
# fill in ANTHROPIC_API_KEY, GLACIER_API_KEY, WALLET_ADDRESS at minimum
# start Chroma yourself: docker run -p 8000:8000 chromadb/chroma, or "chroma run"
```

## Files

| File | What it does |
|---|---|
| `model_provider.py` | Provider abstraction: `create_model_client()`, four providers, one shared async interface |
| `direct_rpc.py` | Method 1: raw JSON-RPC via `web3.py`, no external chain SDK needed |
| `chainkit_fetch.py` | Method 2: calls the Glacier REST API directly (there is no official ChainKit Python package, see the note in the file) |
| `chainkit_mcp_agent.py` | ChainKit as MCP server, using the official `mcp` Python SDK |
| `advisor.py` | Session 4: the Smart Wallet Advisor, fetch, normalize, summarize, with a human-in-the-loop checkpoint and audit logging |
| `rag.py` | Session 5: retrieval-augmented generation, chunk, embed, store, retrieve, generate, grounded answers with citations |
| `normalize.py` | Shared wei-to-AVAX, hex-to-decimal, Unix-to-ISO8601 conversion |

## Running each one

```bash
python direct_rpc.py
python chainkit_fetch.py
python chainkit_mcp_agent.py
python advisor.py <wallet-address>       # Session 4: the full Smart Wallet Advisor
python rag.py "your question"            # Session 5: the RAG agent, needs Chroma running
```

For `chainkit_mcp_agent.py`, start the ChainKit MCP server in another
terminal first (this still needs Node.js installed, ChainKit itself is
JS-only):

```bash
npx -y @avalanche-sdk/chainkit mcp-server
```

Put the URL it prints into `CHAINKIT_MCP_URL` in your `.env`.

For `rag.py`, Chroma needs to be running first, either via
`docker compose up -d chroma` from the repo root, or locally with
`docker run -p 8000:8000 chromadb/chroma`.

## A note on embeddings

The official `chromadb` Python package computes embeddings locally, in
this process, using a small bundled model (`all-MiniLM-L6-v2`). The
first time you run `rag.py`, Chroma downloads that model, a few tens of
megabytes, so the very first call is slower and needs internet access.
After that it's cached and runs offline.

If you're behind a restrictive firewall or proxy and that first
download fails or comes back corrupted, that's a network access issue
to the model host, not a bug in this code, the request, response, and
Chroma API calls themselves were all verified separately against a real
running Chroma server during development.

## A note on ChainKit specifically

There is no official ChainKit SDK for Python. `chainkit_fetch.py` calls
the Glacier REST API that ChainKit itself wraps in JavaScript, directly.
Verify the exact endpoint shape against Glacier's current docs before
you build on this, it was written from the documented API shape, not
tested against a live key, that limitation is stated in the file itself
too.

## Model provider

`MODEL_PROVIDER` in `.env` picks the provider (`anthropic`, `openai`,
`gemini`, or `ollama`), defaulting to `anthropic`. Only the Anthropic
path implements tool calling, required for `chainkit_mcp_agent.py`, the
other three are plain text chat, clearly marked as such in
`model_provider.py`. This doesn't affect `rag.py`, which doesn't use
tools, it just calls whichever provider you have active for a normal
grounded answer.

## Submission

1. Test everything yourself, confirm your RAG agent answers correctly from your documents and refuses anything outside them.
2. Screenshot the working test, including at least one grounded answer with a citation, and one correct refusal.
3. Open your PR, screenshot that too.
4. Post on X with both screenshots, tag **@code_mwangi** and **@AvaxAfrica**.
5. Copy your post link, submit it on the quest page once it's live.

Post in the Week 3 WhatsApp group for anything you get stuck on.

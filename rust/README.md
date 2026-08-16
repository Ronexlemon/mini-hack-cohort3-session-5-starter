# Session 5 Starter — Rust

Retrieval-augmented generation, plus Docker support. Compiled,
memory-safe, zero-cost abstractions, all the usual reasons you'd reach
for Rust.

## Setup

**With Docker** (no local Rust install needed):

```bash
cp .env.example .env
# fill in ANTHROPIC_API_KEY, OPENAI_API_KEY, GLACIER_API_KEY, WALLET_ADDRESS
cd .. && docker compose up -d chroma
docker compose run --rm rust ./rag "What is the C-Chain?"
```

**Without Docker:**

```bash
cp .env.example .env
# fill in ANTHROPIC_API_KEY, OPENAI_API_KEY, GLACIER_API_KEY, WALLET_ADDRESS
cargo build
# start Chroma yourself: docker run -p 8000:8000 chromadb/chroma
```

Requires Rust 1.85 or newer if building outside Docker (the `2021`
edition needs a reasonably current toolchain, this was built and
verified against 1.91). If `cargo build` complains about edition
support, update with `rustup update` or your system package manager.

## Layout

Like Go, Rust wants one binary per `fn main()`, so each runnable
example lives in `src/bin/`, and the shared code lives in the library:

| Path | What it is |
|---|---|
| `src/modelprovider.rs` | Provider abstraction module: `new_model_client()`, four providers, one shared async interface |
| `src/normalize.rs` | Shared wei-to-AVAX, hex-to-decimal, Unix-to-ISO8601 conversion |
| `src/bin/direct_rpc.rs` | Method 1: raw JSON-RPC over plain HTTP, no SDK at all |
| `src/bin/chainkit_fetch.rs` | Method 2: calls the Glacier REST API directly |
| `src/bin/chainkit_mcp_agent.rs` | ChainKit as MCP server, using the `rmcp` crate |
| `src/bin/advisor.rs` | Session 4: the Smart Wallet Advisor, fetch, normalize, summarize, with a human-in-the-loop checkpoint and audit logging |
| `src/bin/rag.rs` | Session 5: retrieval-augmented generation, chunk, embed, store, retrieve, generate, grounded answers with citations |

## Running each one

```bash
cargo run --bin direct_rpc
cargo run --bin chainkit_fetch
cargo run --bin chainkit_mcp_agent
cargo run --bin advisor -- <wallet-address>       # Session 4: the full Smart Wallet Advisor
cargo run --bin rag -- "your question"            # Session 5: the RAG agent, needs Chroma running
```

For `chainkit_mcp_agent`, start the ChainKit MCP server in another
terminal first (needs Node.js, ChainKit itself is JS-only):

```bash
npx -y @avalanche-sdk/chainkit mcp-server
```

Put the URL it prints into `CHAINKIT_MCP_URL` in your `.env`.

For `rag`, Chroma needs to be running first, either via
`docker compose up -d chroma` from the repo root, or locally with
`docker run -p 8000:8000 chromadb/chroma`.

## A note on embeddings, and why Rust needs OPENAI_API_KEY regardless of chat provider

There is no official Chroma Rust crate. `rag.rs` calls Chroma's REST API
directly instead, verified against a real running Chroma v2 server
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
five binaries in a `rust:1.82` image, then copies just the compiled
binaries into a minimal `debian:bookworm-slim` image for the final
result. See the root README for the full `docker compose` quick start.

## A note on SDKs in this folder

There is no official Rust SDK from Anthropic, OpenAI, or Google at the
time this was written, unlike Python, JavaScript, and Go, which all
have first-party SDKs. Rather than depend on an unofficial crate of
uncertain quality, every provider in `modelprovider.rs` talks to its
REST API directly with `reqwest`. This is documented in the module's
own doc comment, not hidden, and it's a genuinely good way to see what
those SDKs are doing for you in any other language.

The one real SDK dependency here is `rmcp` for the MCP client, which is
maintained and does have official backing, that one's a real crate, not
a REST fallback.

## Model provider

`MODEL_PROVIDER` in `.env` picks the provider (`anthropic`, `openai`,
`gemini`, or `ollama`), defaulting to `anthropic`. Only the Anthropic
path implements tool calling, required for `chainkit_mcp_agent`, the
other three are plain text chat, clearly marked as such in
`modelprovider.rs`.

## Submission

1. Test everything yourself, confirm your RAG agent answers correctly from your documents and refuses anything outside them.
2. Screenshot the working test, including at least one grounded answer with a citation, and one correct refusal.
3. Open your PR, screenshot that too.
4. Post on X with both screenshots, tag **@code_mwangi** and **@AvaxAfrica**.
5. Copy your post link, submit it on the quest page once it's live.

Post in the Week 3 WhatsApp group for anything you get stuck on.

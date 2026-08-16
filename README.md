# Mini Hack — Cohort 3, Session 5 Starter

**Building Agentic Solutions on Avalanche** · Team1 Kenya

Retrieval-augmented generation, in four languages, plus Docker support
so nobody needs a local Node, Python, Go, or Rust install just to try
this. Builds directly on the Session 4 starter, same model-provider
layer, same normalize function, same on-chain data methods, now
extended with an agent that can answer questions grounded in your own
documents, with citations, and knows when to say it doesn't know.

## What's new this session

Every language folder gets one new file, `rag` (`rag.js`, `rag.py`,
`rag/main.go`, `src/bin/rag.rs`), which does five things:

1. Chunks a set of documents (our five sample docs are already short
   enough to index whole, no splitting needed)
2. Embeds each one, turning the text into a vector that captures its
   meaning
3. Stores those embeddings in Chroma, a vector database
4. Retrieves the closest matches to a question at query time
5. Generates an answer grounded in those matches, with the model
   instructed to cite its sources and refuse anything the documents
   don't cover

Everything else, the model provider layer, direct RPC, ChainKit fetch,
ChainKit as an MCP server, the Smart Wallet Advisor, is carried forward
unchanged from Session 4.

## Running with Docker

You don't need Node, Python, Go, or Rust installed locally to try any
of this. From the repo root:

```bash
# copy .env.example to .env in whichever language folder(s) you want to run,
# and fill in your keys, same as running locally

docker compose up -d chroma
docker compose run --rm javascript npm run rag -- "What is the C-Chain?"
docker compose run --rm python python rag.py "What is the C-Chain?"
docker compose run --rm golang ./rag "What is the C-Chain?"
docker compose run --rm rust ./rag "What is the C-Chain?"
```

Any other script works the same way, just swap the command:

```bash
docker compose run --rm javascript npm run advisor -- <wallet-address>
docker compose run --rm python python direct_rpc.py
```

Each language's container talks to the same Chroma instance
automatically, `docker-compose.yml` wires `CHROMA_HOST=chroma` in for
you, you don't need to edit anything for that part to work. First run
of `docker compose up -d chroma` pulls the official Chroma image, and
the first build of each language service installs its dependencies
inside the container, both are one-time costs.

**A note on how this was verified.** There's no Docker daemon available
in the environment this was built in, so the Dockerfiles and
`docker-compose.yml` could not be run end to end here. What was
verified: valid Dockerfile syntax, valid YAML (parsed and checked
programmatically), base image tags that are real, standard, currently
published images (`node:20-slim`, `python:3.12-slim`, `golang:1.23`,
`rust:1.82`, `debian:bookworm-slim`), and that every path, script name,
and binary name referenced actually exists in this repo. If something
doesn't build cleanly on your machine, that's the piece to check first,
not assume the whole approach is wrong.

## Running without Docker

Same as every session before this one, each language folder has its own
setup steps in its README. Short version: install that language's
dependencies, copy `.env.example` to `.env`, start Chroma yourself
(`docker run -p 8000:8000 chromadb/chroma`, or `chroma run` if you have
the Python package installed), and run the script you want.

## A note on embeddings, and where the four languages genuinely differ

JavaScript and Python use Chroma's official clients, which compute
embeddings locally using a small bundled model, downloaded once on
first use. Go and Rust have no official Chroma client, so those two
call Chroma's REST API directly, verified against a real running Chroma
server during development, not guessed from documentation, and they
compute embeddings by calling OpenAI's embeddings API instead. That
means **Go and Rust need `OPENAI_API_KEY` set even if your chat model is
a different provider**, embeddings and chat completion are separate
capabilities. This is explained again, in more depth, in each language's
own README, and directly in the comments at the top of each `rag` file.

## Picking a language

Same guidance as before, pick the one you're building your Week 3
deliverable in:

| Language | Folder |
|---|---|
| JavaScript | [`javascript/`](./javascript) |
| Python | [`python/`](./python) |
| Go | [`golang/`](./golang) |
| Rust | [`rust/`](./rust) |

## Submission

Same flow as every week: test it yourself, screenshot the working test
and your PR, post on X tagging **@code_mwangi** and **@AvaxAfrica**,
then submit that link on the quest page. Full steps are in each
language folder's README.

## A note on how the code itself was verified

Same standard as every session: every file was actually compiled,
imported, or run, not just written and assumed correct. Along the way
this caught a real gap before it shipped, the first draft of every `rag`
file hardcoded `localhost:8000` for Chroma, which would have silently
broken the moment the app itself ran inside a container and tried to
reach a sibling `chroma` container by that name. All four were fixed to
read `CHROMA_HOST` and `CHROMA_PORT` from the environment instead,
defaulting to `localhost:8000` for local development, before Docker
support was added on top.

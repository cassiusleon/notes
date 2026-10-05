---
title: Architecture
project: "[[Home AI Assistant]]"
chapter: 2
date: 2026-10-02
tags:
  - home-ai
  - architecture
  - rag
aliases:
  - RAG
---

# 02 · Architecture

## Hosting options considered
| | A. Home box ✅ | B. Private AWS | C. Hybrid |
|---|---|---|---|
| Model runs | Mini PC at home | EC2, private subnet, no internet | Home box |
| Documents | Home | S3 with your own KMS key | Home |
| Cloud role | None | Everything | Entry points only (Alexa, email, chat) |
| Running cost | Electricity (10–40 W) | ~₹17,500/month | A few cents of Lambda |

> [!success] Decision
> **A. Home box.** It pays for itself against AWS in 4–5 months.

## Security perimeter
> [!info] The perimeter goes around the model, not inside it
> A model is just a file. Leaks come from the code and network around it.

- **LLM server:** no internet in either direction. One port, reachable only by the agent
- **Agent:** the only thing that talks to the model and to the outside world
- **Notifier:** the single way out, with a hardcoded allowlist (your email and chat ID). This makes prompt injection in documents harmless
- **Limit:** Alexa, email and chat pass through outside servers. For sensitive files, send a link reachable over Tailscale, not the attachment

## How the pieces connect
![[Architecture.excalidraw]]

> [!note]- Text version (Mermaid)
> ```mermaid
> flowchart LR
>     subgraph IN[Entry points]
>       A[Alexa]
>       C[Chat app via Tailscale]
>       G[Gmail - selected labels]
>     end
>     subgraph P[Home perimeter - no egress]
>       O[Agent / orchestrator<br/>6-8 tools]
>       L[LLM server<br/>no network]
>       D[(Docs + Postgres/pgvector)]
>       H[Home Assistant]
>       F[Background jobs<br/>output files]
>     end
>     N[Notifier<br/>allowlist only]
>     A --> O
>     C --> O
>     G --> D
>     O <--> L
>     O <--> D
>     O --> H
>     F --> O
>     O --> N
>     N --> ME[Your email / chat]
> ```

## Where data lives (RAG)
> [!important] The model stores nothing
> Your data stays outside the model and is fed in with each question. This is **RAG** (retrieval-augmented generation). Swap the model later and all the data carries over.

### The three artefacts
| # | Artefact | What's in it | Where | Changes? | Back up? |
|---|---|---|---|---|---|
| 1 | **Model** | LLM weights + embedding model | Disk, loaded into RAM | Only when you swap it | No, re-download |
| 2 | **Documents** | Original files, emails, attachments | Folder on 2nd SSD (or MinIO) | Grows | **Yes** |
| 3 | **Memory & index** | Vector index, preferences, conversation summaries, tasks, reminders | Postgres + pgvector | Grows with every interaction | **Yes** |

> [!tip] Rule of thumb
> Back up **2 and 3** and you can rebuild everything else on any machine.

### Memory & index in detail
- **Format:** Postgres tables. Preferences, summaries, tasks and document chunks are plain text; each chunk also has an embedding vector (a list of numbers)
- **Read and edit it yourself:** yes, with SQL or a GUI like DBeaver or pgAdmin. Text is editable; vectors aren't readable, so re-embed any row whose text you change
- **Change the LLM:** everything carries over untouched. The model only reads the retrieved text
- **Change the embedding model:** old vectors stop matching. Run a one-off job to re-embed the stored text, which is why the source text is always kept next to its vector
- **Portability:** `pg_dump` or a JSONL export moves it anywhere

### On a question
Embed the question → search the vector index → take the top chunks → send question + chunks + short memory summary to the LLM → answer. Typically a few thousand tokens, not the whole library.

## Model size ladder
| Size | Handles | Example |
|---|---|---|
| 1–3B | Single direct commands | "Dim the living room to 30%" |
| 4B | One tool with details | "Remind me to pay the electricity bill on the 5th" |
| 8B | Several steps or implied details | "Read this bill, add the due date, remind me two days earlier" |
| 14B / MoE | Long or subtle documents | "What changed between last year's and this year's lease?" |

- Candidates: Qwen3.5-4B, Gemma 4 E4B, Llama 3.1 8B, Qwen3 14B. All support tool calling
- The model only *outputs* a call such as `create_event(...)`. The agent validates it and makes the real call

## Agent design
- **One orchestrator** with 6–8 tools: lights, calendar, reminders, document search, document send, task list
- **Fast path:** simple light commands go straight to Home Assistant, with no LLM
- **Router step:** if tools grow past ~10, classify the request first (home / calendar / docs)

## Channels
| Channel | Choice |
|---|---|
| Chat | Home Assistant Companion, Signal or Matrix (Element), over Tailscale |
| Voice | Alexa, answering from precomputed output files |
| Email in | Gmail API or IMAP, read-only, selected labels only, one-time backfill, OCR for scans |
| Lyrics | Spotify (linked to Alexa) → Spotify Connect API for playback position; LRCLIB (free) or Musixmatch (paid) for synced lyrics |

> [!warning] Caveats
> - Background jobs share the GPU with live requests
> - Precomputed answers can go stale
> - Anything spoken to Alexa goes through Amazon

---
← [[01 - Requirements]] | [[Home AI Assistant]] | [[03 - Hardware]] →

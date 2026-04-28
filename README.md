<p align="center">
  <img src="assets/atenea-logo.png" alt="Atenea logo" width="300">
</p>

<h1 align="center">building-atenea</h1>

<p align="center"><em>Building an autonomous pentesting agent in public.</em></p>

## Why this exists

I saw cybersecurity as something massive and started imagining what
I could build. The first idea was a parrot — ask it a question, it
gives back the answer I programmed. Useful as a reminder, useless
in practice: by the time I'd programmed it, I'd already learned
the answer.

So I started looking for ways to make it more useful. And the more
I built, the more I saw opportunities to push further. The parrot
became something else.

## What this devlog covers

Atenea is one project, but it grew in stages. Each stage is a
natural evolution of the previous one:

- **Parrot** → a fine-tuned LLM that answered cybersecurity questions
  in Spanish. Useful as proof of concept, limited in practice.
- **AI with a brain** → reasoning, RAG, persistent memory,
  multi-profile system. The model stopped repeating and started
  thinking.
- **Deterministic pentesting agent** → forced actions, a planner,
  an executor. End-to-end exploitation chains validated against
  TryHackMe rooms. This phase closed in April 2026 with 19 rooms
  validated, 96% coverage.
- **Guardian** *(in development)* → the defensive mirror of the
  pentesting agent. Same architecture, opposite purpose: detect,
  respond, defend.
- **Honeypot** *(in development)* → real attack data feeding both
  agents. Captured traffic, attacker patterns, exploit attempts.
  The polish that turns lab work into something grounded in
  reality.

This devlog documents that journey: decisions, mistakes, lessons.
The source code of Atenea is kept private for operational reasons,
but the process is here in the open.

## Tech stack

For transparency and to give reviewers a sense of the technical
ground:

- **Language:** Python 3.11+
- **Model:** Gemma 2 9B Q5_K_M, fine-tuned, served locally via Ollama
- **GPU:** RTX 3080 (10 GB VRAM)
- **OS:** Windows 11 + WSL2 (Kali Linux)
- **RAG:** FAISS with MiniLM embeddings
- **Pentesting toolkit:** Metasploit, nmap, hydra, sqlmap, gobuster, wpscan
- **Honeypot:** custom services on a Vultr VPS (Madrid)
- **Architecture:** modular mixin-based planner, persistent listener
  via msfconsole, three-level RAG (raw / confirmed / experiential)

A reviewer with this stack and the right knowledge could reproduce
the setup. The novelty is not in the tools — it's in how they
interact and what gets automated around them.

## Roadmap

Current and upcoming phases of the project:

- ✅ **Pentesting agent** — closed (April 2026)
- 🔄 **Honeypot** — gathering real-world attack data
- 🔜 **Guardian** — defensive agent mirroring the pentesting one
- 🔜 **Adversarial training** — Red team agent vs Blue team agent,
  fed by real honeypot data

## Methodology

Atenea is built using a workflow I call "the trident": Claude
diagnoses and designs, Opus implements, I test and decide. Each
of the three has a clear role and the others compensate when one
fails. See [methodology/trident.md](methodology/trident.md) for
the full explanation and a worked example.

## Status

Currently in transition between phases. Pentesting agent: closed.
Guardian, honeypot, adversarial training: ahead.

## Project structure

```
building-atenea/
├── methodology/    # Methodology documents
├── posts/          # Devlog posts (chronological)
├── diagrams/       # Architecture and design diagrams
└── timeline/       # Project timeline and milestones
```

Posts and diagrams will be added as each phase progresses.

---

*Built in the margins — between warehouse shifts, freelance work,
and the hours that are mine.*

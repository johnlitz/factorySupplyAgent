# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Status:** Early scaffold. This is a *basic layout* that captures the vision, intended architecture, and working conventions. Most sections are stubs to be filled in as decisions are made. Do not treat any section below as final spec.

## Project Purpose

An AI **sourcing agent** that turns a rep/quality-garment guide (originally an r/stockholmreps Reddit post) into machine-usable knowledge, then sources the **best-quality garments — cashmere, cotton, silk, and more — from Taobao / 1688 / Weidian** without the user having to manually translate, search, and type everything.

The **agent is the backbone**. A UI or open code layer sits on top. The system is designed to work well with **OpenRouter models via API key**, so the underlying model is swappable.

## Core Goals / Non-Goals

**Goals**
- Eliminate manual translation and manual searching on Chinese marketplaces.
- Curate results by **quality tier** and **material** (cashmere / cotton / silk / etc.) — materials-first.
- Let the user describe what they want in plain English and get back vetted sourcing options.
- Stay **model-agnostic** via OpenRouter (swap models with an API key).

**Non-Goals (for now — open questions)**
- Automated checkout / payment / order placement — **purchasing flow is undecided** (middleman-agent service vs. direct integration).
- Committing to a scraping/marketplace-access mechanism — flagged as an open design decision.

## Architecture (planned)

High-level components. Names and boundaries are provisional.

- **Knowledge base** — the Reddit guide plus curated seller/quality notes, stored as structured data. This is the agent's source of truth.
- **Agent core** — the reasoning backbone: takes a natural-language request, plans, queries the knowledge base, and returns sourcing results.
- **Translation layer** — EN⇄ZH so the user never types Chinese; normalizes search terms and pinyin.
- **Marketplace interface** — search/query against Taobao / 1688 / Weidian. *Access method TBD (open decision).*
- **Model layer (OpenRouter)** — API-key config, model selection/routing, and prompt templates.
- **Interface** — the agent's own UI and/or an open code entry point.

## Quality Definition (the foundation)

**Start here.** The operational definition of "best quality" — and the rules the agent uses to
judge, score, and rank any listing — is the bedrock everything else depends on. Do not add search,
ranking, or purchasing logic that bypasses it.

- **`knowledge/quality-rubric.md`** — human-readable rationale + citations. Defines the pipeline:
  an **authenticity gate** (hard-fail / verify) followed by four weighted axes
  (fiber_quality 0.40, construction 0.25, spec_transparency 0.20, value 0.15) → 0–100 score →
  buy/verify/skip. Core principle: *quality = fiber × honest construction × verified authenticity ÷ price* — buy the fiber the luxury houses buy, not the label.
- **`data/quality-model.yaml`** — the machine-readable, material-agnostic scoring framework the
  agent loads (axes, weights, tiers, decisions).
- **`data/materials/*.yaml`** — per-material thresholds and Chinese spec vocabulary
  (`cashmere.yaml`, `cotton.yaml`, `silk.yaml`). New materials drop in here without changing the model.

When editing, keep the YAML and the markdown in sync — the YAML is what the agent consumes.

## Knowledge Source

The seed guide is the r/stockholmreps post the user referenced. Reddit cannot be auto-fetched, so the post content was pasted in and structured into a machine-parseable repo doc:

- **`knowledge/sourcing-guide.md`** — cashmere knitwear: EN→ZH search terms, quality tiers (ply/GSM/price), premium yarn suppliers (Consinee, Erdos, King Deer), Chinese brand list, and a reference Taobao listing.
- Future materials (cotton, silk, other garments) get their own `knowledge/*.md` docs following the same structure.

## Tech / Stack (proposed — TBD)

Nothing is locked in yet.

- **Language/runtime:** open (leaning Python or Node).
- **Models:** OpenRouter.
- **Config:** `.env` holding `OPENROUTER_API_KEY` (never committed).

## Repo Conventions / How to Work Here

- **Secrets stay out of git** — use `.env` + `.gitignore`; never commit API keys.
- **Knowledge docs live under `knowledge/`** and should be structured/machine-parseable, not free prose.
- **Develop on the designated feature branch** (`claude/garment-sourcing-agent-guide-f2pdry`).
- Keep this file updated as architecture and stack decisions are made.

## Status / Roadmap

- [x] Scaffold `CLAUDE.md`
- [x] Add the pasted Reddit guide as structured knowledge (`knowledge/sourcing-guide.md`)
- [x] **Nail the quality definition & rules** — the rubric/scoring model the agent judges by
      (`knowledge/quality-rubric.md`, `data/quality-model.yaml`, `data/materials/*.yaml`)
- [ ] Decide the tech stack
- [ ] Decide the purchasing / order-placement flow
- [ ] Build the agent core
- [ ] Wire up OpenRouter model layer
- [ ] Build the UI / code interface

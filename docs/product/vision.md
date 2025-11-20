# Zen Studio Command Centre – Vision

## 1. Problem

Small studios and solo devs increasingly rely on multiple AI tools (ChatGPT, local UIs, ad-hoc scripts) to do research, write PRDs, design architectures, and plan implementation.

The result is scattered conversations, duplicated work, and poor reusability. There is no calm, project-centric place where all AI-assisted work for a product lives.

## 2. Solution

Zen Studio Command Centre is a **project-centric AI workspace** built on top of LibreChat.

It organizes work around **projects**, a small set of **specialist agents** (Researcher, PRD Writer, Architect, Planner), and persistent **artifacts** (PRDs, plans, documents). The UI is intentionally minimal and “Zen”: fewer toggles, more focus.

## 3. Target Users

- **Primary**: solo devs and tiny studios building digital products (web apps, SaaS, marketing sites, internal tools).
- **Secondary**: technical PMs and tech leads who want to centralize AI work for their projects.
- **Tertiary** (later): clients who need read-only access to project outputs in a clear format.

## 4. Positioning & Differentiation

- **Not just another ChatGPT client**: it is product-workflow oriented, not freeform chat.
- **Built on LibreChat**: leverages mature multi-provider + agents + RAG, while layering a focused studio workflow on top.
- **Opinionated but configurable**: ships with a default “studio stack” of agents and flows but can be adapted over time.

## 5. High-Level Goals

1. Replace scattered AI chats with a **single project-centric workspace**.
2. Provide **reliable, repeatable flows**: Research → PRD → Architecture → Implementation Plan.
3. Make all key outputs **portable** (JSON + Markdown) so nothing is locked in.
4. Keep the UI and mental model **calm, minimal, and trustworthy**.

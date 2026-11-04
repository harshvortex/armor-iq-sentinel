<div align="center">

# ArmorIQ Sentinel
### Zero-Trust AST Proxy & Dual-Plane Guardrail for Autonomous AI Agents

[![CI Pipeline](https://github.com/harshvortex/armor-iq-sentinel/actions/workflows/ci.yml/badge.svg)](https://github.com/harshvortex/armor-iq-sentinel/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.11+](https://img.shields.io/badge/Python-3.11%2B-blue?logo=python)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110+-009688?logo=fastapi)](https://fastapi.tiangolo.com/)
[![Docker Compliant](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker)](docker-compose.yml)

**ArmorIQ Sentinel** is a high-throughput, dual-plane reverse proxy and runtime guardrail engine designed to prevent rogue tool executions, payload smuggling, and indirect prompt injection attacks against LLM-driven autonomous agents.

[Architecture](#system-architecture) • [Key Engineering Pillars](#key-engineering-pillars) • [Benchmarks](#empirical-benchmarks) • [Quickstart](#quickstart-60-seconds)

</div>

---

## The Problem
When autonomous LLMs (LangChain, AutoGen, CrewAI) interact with terminal environments, databases, and third-party APIs, traditional guardrails (regex, prompt-level instructions) fail against **Indirect Prompt Injection** and **Tool Hijacking**. ArmorIQ Sentinel decouples orchestration from execution, enforcing strict AST verification and semantic intent matching before any downstream side-effect can execute.

---

## System Architecture

```mermaid
flowchart TD
    User([User Prompt / Webhook]) --> Gateway[Node.js Gateway / Control Plane]
    Gateway --> Agent[Autonomous LLM Agent]
    Agent -- Proposed Tool Call (JSON) --> Sentinel[ArmorIQ Python Sentinel / Data Plane]
    
    subgraph Sentinel Inspection Engine
        direction TB
        AST[Tree-sitter AST Syntax & Command Validator]
        Semantic[Intent Drift & Embeddings Matcher]
        Policy[Zero-Trust Egress & Sandbox Ruleset]
        
        AST --> Policy
        Semantic --> Policy
    end
    
    Sentinel --> Sentinel Inspection Engine
    
    Policy -- Validated Signature --> ToolExec[Safe Tool Runner / OS / DB / API]
    Policy -- Injected / Malicious Payload --> Quarantine[Blocked: 403 Forbidden + Audit Log]
    
    ToolExec --> Gateway
    Quarantine --> Gateway

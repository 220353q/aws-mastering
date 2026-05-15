# Amazon Bedrock

## Overview
Fully managed service for foundation models (FMs). Access leading FMs from AI21, Anthropic, Cohere, Meta, Stability AI, Amazon Titan via unified API. Build generative AI apps with Agents, Knowledge Bases, Guardrails.

## Key Features
- Foundation Models (Claude, Llama, Titan, Stable Diffusion, etc.)
- Agents (orchestration + tool use)
- Knowledge Bases (RAG with vector store)
- Guardrails (content filtering, PII redaction)
- Fine-tuning + continued pre-training
- Model evaluation + playground

## Use Cases (Tier 2 - Emerging High Value)
1. **Generative AI Chatbot / Assistant** - Bedrock + Agents + Knowledge Bases
2. **RAG Application** - Knowledge Bases + Claude + Lambda frontend
3. **Content Generation** - Titan / Stable Diffusion for text/images
4. **Enterprise Search + Summarization** - Bedrock + OpenSearch + Guardrails
5. **Responsible AI Workflows** - Guardrails + Clarify integration

## Connections
- **Lambda / API Gateway**: Frontend
- **S3 / OpenSearch**: Knowledge base storage
- **EventBridge**: Agent triggers
- **CloudWatch**: Monitoring + costs

**SAP-C02 (2026+)**: Design generative AI solutions with RAG, agents, and safety guardrails.
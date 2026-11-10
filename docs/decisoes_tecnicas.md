# Decisões Técnicas e Stack

Justificativas de arquitetura do EngMan Copilot: execução gratuita, privacidade e idioma[cite: 18, 22].

## 1. Comparativo de Modelos (LLMs)

| Critério | Qwen 2.5 32B (Ollama) | Google Gemini Flash |
| --- | --- | --- |
| Custo de Uso | Gratuito (Local)[cite: 18] | Gratuito (API nuvem)[cite: 18] |
| Privacidade | Alta (Dados na infraestrutura)[cite: 22] | Média (Dados no Google)[cite: 22] |
| Qualidade PT-BR | Excelente | Excelente |
| Janela de Contexto | 32k tokens | 1M+ tokens |
| Decisão e Trade-off | ESCOLHIDO. Contexto menor, mas garante confidencialidade[cite: 18, 22]. | Descartado para mitigar vazamento[cite: 22]. |

## 2. Comparativo de Vector Stores (RAG)

| Critério | ChromaDB | FAISS |
| --- | --- | --- |
| Documentação | Alta[cite: 18] | Muito Alta |
| Integração Python | Excelente (persistência nativa)[cite: 18] | Boa (gestão manual)[cite: 18] |
| Decisão e Trade-off | ESCOLHIDO. Simples de integrar e atende a escala[cite: 18]. | Descartado pela complexidade. |

## 3. Orquestração e Ambiente
- Orquestração: Chamada direta (Python SDK). Evita complexidade do LangChain e garante controlo dos guardrails[cite: 18].
- Ambiente e Embeddings: Python 3.10 no Google Colab, usando nomic-embed-text via Ollama[cite: 18].
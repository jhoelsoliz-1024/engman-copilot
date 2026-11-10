# EngMan Copilot | Challenge FIAP & Motiva Rodovias

Documentação oficial da arquitetura de inteligência artificial e engenharia de prompt para o suporte operacional de ativos ITS.

## 1. Identificação do Projeto e Equipe
Desafio: Inovação Aberta - Motiva Rodovias (Engenharia de Manutenção)

Trilha / Recorte: Recorte B - Operação e Priorização de Campo (CCO)

Integrantes:

Andre da Silva Pinto (RM: 559038) - Turma: 1CCPS
Iago Nunes Campos (RM: 576738) - Turma: 1CCPS
Jhoel Rayan Calle Soliz (RM: 574622) - Turma: 1CCPS
Matheus Libarino da Silva (RM: 575073) - Turma: 1CCPS

## 2. Contextualização e Problemática de Negócio
A operação de rodovias concedidas enfrenta desafios críticos na gestão de ativos ITS (Sistemas de Transporte Inteligente), destacando-se:
1. Falsos Positivos Recorrentes: Fatores ambientais (teias de aranha em CFTV, reflexos solares/glare e intempéries) acionam alarmes falsos de forma contínua.

2. Manutenção Reativa Ineficiente: Deslocamentos de equipes técnicas de campo sem triagem prévia qualificada, gerando custos operacionais desnecessários e risco à segurança viária.
O EngMan Copilot atua como um assistente cognitivo especialista acoplado ao Centro de Controle Operacional (CCO), mitigando o viés humano e otimizando a priorização de chamados.

## 3. Escopo de Atuação e Limites do Sistema (Guardrails)
* Domínios Cobertos: CFTV, PMV (Painel de Mensagem Variável), Callbox, Sistemas de Arrecadação (SAT/SCA), Laços Indutivos, Pórticos Free Flow e WIM (*Weigh-in-Motion*).

* O que o Copiloto faz: Diagnóstico diferencial de falhas, consulta a históricos de reincidência, estruturação de rascunhos de Ordens de Serviço (OS) e cálculo de métricas de confiabilidade.
* O que o Copiloto NÃO faz (Limites Estritos): Emissão de laudos de engenharia (ART), atuação direta em CLPs ou sistemas SCADA da concessionária, e recomendação de intervenções físicas sem o respaldo das normas de segurança elétrica e em altura (NR-10 e NR-35).

## 4. Arquitetura da Solução e Fluxo Operacional
O pipeline de inferência é composto por um fluxo resiliente de 8 blocos:
1. Entrada do Operador: Submissão do evento ou alarme bruto.

2. Guardrail de Entrada: Validação de payload e sanitização contra **Prompt Injection**.

3. Context Engineering: Agregação estática (System Prompt + Few-Shot) e dinâmica (RAG vetorial via ChromaDB).

4. Inferência LLM: Processamento via Qwen 2.5 32B com temperatura controlada ($T=0.3$).

5. Decisão de Tool (Function Calling):** Avaliação lógica da necessidade de buscar dados retroativos.

6. Execução de Ferramentas: Resgate de dados simulados (FSM / Histórico de Falhas).

7. Guardrail de Saída: Auditoria de conformidade (LGPD, ausência de PII e checagem de escopo).

8. Entrega e Persistência: Resposta estruturada (Fato vs. Hipótese) exibida ao operador do CCO.
* Diagrama Visual Detalhado: [`docs/arquitetura.png`](docs/arquitetura.png)

## 5. Stack Tecnológica e Justificativas de Arquitetura

| Camada | Tecnologia Adotada | Justificativa Técnico-Acadêmica |
| :--- | :--- | :--- |
| **Modelagem (LLM)** | Qwen 2.5 32B (Ollama) | Escolhido pelo excelente desempenho multilíngue em PT-BR, pesos abertos e garantia de execução *on-premise*, blindando dados corporativos sensíveis da concessionária. |
| **Recuperação (RAG)** | ChromaDB | Simplicidade de persistência local em disco, integração nativa com ecossistema Python e eficiência adequada para o volume do inventário rodoviário. |
| **Orquestração** | Python SDK Nativo | Eliminação de abstrações opacas (como frameworks de prateleira pesados), garantindo controle granular e auditabilidade estrita dos *guardrails*. |

## 6. Governança da Base de Conhecimento (RAG)
* **Estratégia de Chunking:** Segmentação otimizada em blocos de 500 a 800 tokens para preservar a coesão semântica de procedimentos operacionais (POPs) e normas técnicas.
* **Metadados de Indexação:** Uso de filtragem estrita baseada em `tag_ativo`, `tipo_ativo` e `tipo_documento` para mitigar alucinações de contexto cruzado.
* **Conformidade LGPD (Lei 13.709/2018):** Exclusão absoluta de dados de tráfego identificáveis (placas de veículos, rostos em CFTV) e informações de colaboradores da base de conhecimento do modelo.
* **Inventário Completo:** [`data/knowledge_base/inventario.md`](data/knowledge_base/inventario.md)

## 7. Engenharia de Prompt e Estratégia de Testes (Eval Set)
* **System Prompt:** Desenvolvido sob diretrizes modulares, incorporando *Chain-of-Thought* (CoT) para diagnósticos lógicos e *Few-Shot Prompting* para calibração de prioridades (Crítica, Alta, Média, Baixa).
  * *Ref:* [`prompts/system_prompt.md`](prompts/system_prompt.md)
* **Avaliação de Resiliência (Eval Set):** Conjunto de teste estático contendo 10 cenários rigorosos cobrindo casos factuais, ambíguos, tentativas de *Jailbreak*, dados inexistentes e riscos operacionais críticos.
  * *Ref:* [`evals/sprint1_eval_set.json`](evals/sprint1_eval_set.json)

## 8. Ferramentas Integradas (Function Calling & Bônus de Confiabilidade)
O modelo possui especificação de ferramentas (*Tools Schema*) para integração sistêmica, incluindo recursos avançados de cálculo de confiabilidade preditiva (MTBF e função de sobrevivência $R(t)$):
* `consultar_ativo` | `consultar_historico_falhas` | `abrir_ordem_servico` | `consultar_procedimento` | `calcular_metricas_confiabilidade` (Bônus Temático 2)
  * *Ref:* [`tools/tools_spec.json`](tools/tools_spec.json)

## 9. Matriz de Riscos, Ética e Segurança
Mapeamento preventivo cobrindo 8 classes de risco (incluindo alucinação, *Automation Bias* em operadores do CCO, estouro de janela de contexto e riscos à integridade física por orientações elétricas).
* *Ref:* [`docs/riscos_lgpd.md`](docs/riscos_lgpd.md)

## 10. Referências Normativas e Bibliográficas
1. **Briefing Challenge Inovação Aberta** – Engenharia de Manutenção Motiva Rodovias (2026).
2. **Lei Geral de Proteção de Dados (LGPD)** – Lei Federal nº 13.709/2018.
3. **Normas Regulamentadoras** – NR-10 (Segurança em Instalações e Serviços em Eletricidade) e NR-35.
4. **Disciplina de Prompt Engineering & Artificial Intelligence** – Material Didático dos Módulos de Arquitetura de LLMs e RAG (FIAP, 2026).
# Matriz de Riscos, Ética e LGPD

Mapeamento de riscos do EngMan Copilot (CCO), com impacto, mitigação e rastreabilidade.

## 1. Alucinação de Fatos e Dados
Descrição: O modelo inventa TAGs, métricas ou falsos históricos de falha.
Impacto: Operacional e Financeiro. Despacho de equipas para equipamentos errados.
Mitigação: Proibição explícita de criação de dados. Obrigação de citar a fonte via Tools.
Rastreabilidade: system_prompt.md (Regra 1) e sprint1_eval_set.json (Casos C02, C06).

## 2. Prompt Injection e Jailbreak
Descrição: Tentativas de subverter as regras inserindo comandos ocultos.
Impacto: Segurança. O assistente responde fora do escopo ou revela instruções internas.
Mitigação: Regras de borda estritas para recusar mudança de persona e guardrail de entrada.
Rastreabilidade: system_prompt.md (Borda) e Fluxograma (Bloco 2).

## 3. Risco Operacional (Intervenção Física)
Descrição: O modelo recomenda ações perigosas em equipamentos energizados.
Impacto: Risco à vida e passivo legal.
Mitigação: Restrição absoluta de orientações físicas. Encaminhamento para NR-10/NR-35.
Rastreabilidade: system_prompt.md (Regras 3 e 6) e sprint1_eval_set.json (Caso C09).

## 4. Viés Algorítmico na Priorização
Descrição: O assistente prioriza manutenção em praças rentáveis em detrimento da segurança.
Impacto: Ético. Desigualdade na infraestrutura e risco viário.
Mitigação: Priorização orientada por Matriz de Criticidade explícita.
Rastreabilidade: system_prompt.md (Few-Shot) e inventario.md.

## 5. Exposição de Dados Pessoais (LGPD)
Descrição: Processamento de dados sensíveis ou pessoais não anonimizados (placas, rostos).
Impacto: Legal. Multas da ANPD e violação da Lei 13.709/2018.
Mitigação: Filtragem prévia (anonimização OCR) e proibição na Base de Conhecimento.
Rastreabilidade: inventario.md (Restrições de LGPD).

## 6. Quebra de Confidencialidade Operacional
Descrição: Vazamento do histórico de arrecadação ou falhas para APIs públicas.
Impacto: Negócios e Reputação.
Mitigação: Uso de dados sintéticos e adoção de LLMs locais (on-premise).
Rastreabilidade: decisoes_tecnicas.md e inventario.md.

## 7. Excesso de Confiança (Automation Bias)
Descrição: Operadores assumem hipóteses da IA como verdades absolutas.
Impacto: Operacional. Falhas críticas mascaradas.
Mitigação: Forçar separação entre FATO e HIPÓTESE e incluir Nível de Confiança.
Rastreabilidade: system_prompt.md (Regra 2 e Formato).

## 8. Estouro da Janela de Contexto
Descrição: O histórico cresce e o modelo esquece regras ou confunde TAGs.
Impacto: Técnico. Respostas inconsistentes.
Mitigação: Sumarização do histórico a cada 6 mensagens.
Rastreabilidade: system_prompt.md (Borda) e decisoes_tecnicas.md.
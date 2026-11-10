# Inventário Detalhado da Base de Conhecimento (RAG)

Este documento especifica a arquitetura de dados, os schemas e os metadados que compõem a base de conhecimento vetorial do **EngMan Copilot** (Recorte B - Operação e Priorização de Campo). Todos os registros operacionais são estritamente simulados.

---

## 1. Mapeamento de Artefatos e Fontes de Dados

| Documento / Artefato | Descrição e Campos Estruturados | Origem dos Dados | Volume Estimado |
| :--- | :--- | :--- | :--- |
| Catálogo de Ativos ITS | Contém a identificação unificada do parque tecnológico. Campos: `tag_ativo` (ex: CFTV-045-S), `tipo_ativo`, `fabricante`, `modelo`, `km_rodovia`, `praca_pedagio`, `data_instalacao`, `criticidade`. | Simulado (Fictício) | 45 ativos mapeados |
| Histórico de Falhas (FSM) | Registro retroativo de chamados para análise de reincidência. Campos: `id_chamado`, `tag_ativo`, `timestamp_alarme`, `sintoma_relatado`, `causa_raiz_efetiva` (ex: ambiental vs hardware), `tempo_reparo_horas`. | Simulado (Fictício) | 120+ ocorrências |
| Procedimentos Operacionais (POPs) | Instruções normativas de triagem remota, ciclos de reset, inspeção visual e critérios de segurança preventiva. | Simulado & Diretrizes Motiva | 5 POPs operacionais |
| Matriz de Criticidade | Tabela de decisão cruzada ponderando o impacto na receita da concessionária versus o risco direto à segurança viária. | Simulado | 1 matriz de decisão |
| Fichas Técnicas de Equipamentos | Manuais resumidos com limites operacionais (ex: tolerância a lux para sensores óticos, faixa térmica e resistência a intempéries). | Real (Specs públicas) / Adaptado | 8 fichas técnicas |
| Glossário Operacional ITS | Padronização de siglas e termos do CCO (CFTV, PMV, SAT, SCA, FSM, WIM, OCR, TAG, Free Flow). | Real (Domínio rodoviário) | 1 glossário unificado |
| Normas de Segurança Aplicáveis | Resumo parametrizado das normas regulamentadoras aplicáveis a campo (NR-10 e NR-35) e protocolos de sinalização de pista. | Real (Normas oficiais) | 2 normativos |

---

## 2. Padrão de Nomenclatura (Estrutura de TAGs)
Para garantir que o modelo não confunda ativos de diferentes trechos da rodovia, a base adota um padrão estrito de indexação por TAG alfanumérica:

Exemplo CFTV: `CFTV-[KM]-[Sentido]` (Ex: `CFTV-045-N` = Câmera de Circuito Fechado de TV no Quilômetro 45, Sentido Norte).

Exemplo PMV: `PMV-[ID]-[Praça]` (Ex: `PMV-03-P4` = Painel de Mensagem Variável na Praça de Pedágio 4).
Exemplo Pórtico Free Flow: `FF-[KM]-[Trecho]` (Ex: `FF-112-SP`).

---

## 3. Parâmetros Técnicos do Pipeline RAG

### 3.1 Estratégia de Chunking
* Tamanho do Chunk: 500 a 800 tokens.
* Overlap (Sobreposição): 100 tokens.
* Justificativa: Procedimentos operacionais e POPs possuem dependência sequencial (passo 1, passo 2). Um chunk muito curto quebraria a instrução de segurança; um chunk muito longo diluiria a precisão da recuperação vetorial.

### 3.2 Esquema de Metadados Obrigatórios
Cada chunk vetorizado no ChromaDB obrigatoriamente carrega os seguintes metadados em formato JSON para filtragem prévia (*Metadata Filtering*):
```json
{
  "tag_ativo": "CFTV-045-N",
  "tipo_ativo": "CFTV",
  "tipo_documento": "historico_falhas",
  "nivel_criticidade": "Alta"
}
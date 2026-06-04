# Exercício 1.3 — Plano de testes para pipeline de RAG

## Contexto
Este arquivo organiza o plano de testes do pipeline de RAG da NovaTech, cobrindo:
- ingestão;
- retrieval;
- geração;
- contexto;
- ponta a ponta;
- regressão.

---

## 1) Objetivo e escopo
- Objetivo do plano: Validar que o pipeline de RAG da NovaTech responde com acuracia, fonte correta e aderencia a guardrails, reduzindo risco de alucinacao e inconsistencias operacionais no atendimento.
- Escopo funcional: ingestao de documentos oficiais, indexacao, retrieval de chunks, geracao de resposta, citacao de fonte, comportamento em perguntas sem cobertura e fluxo ponta a ponta.
- Escopo não funcional: tempo de resposta aceitavel ao atendente, estabilidade em conversa longa, controle de contexto, rastreabilidade de versao documental e repetibilidade dos testes.
- Fora de escopo: UX do Teams, permissao de usuarios no SharePoint, tuning de infraestrutura Azure fora do que impacta qualidade de retrieval/geracao.

## 2) Premissas e artefatos de referência
- Documentos usados (Anexo A): POL-001, PROC-042, PROC-042-v2, SLA-2024 e FAQ-Atendimento (com prioridade para documentos normativos e contratuais).
- Chunks e mapa de cobertura (Anexo B): usar o mapa pergunta -> chunks esperados como gabarito de retrieval para testes RET e E2E.
- Ambiente de teste: indice dedicado de homologacao no Azure AI Search, base congelada para a rodada, logs de retrieval e prompt habilitados, suite de testes executada via pipeline CI.
- Versão do prompt/base/index: Prompt v1.0-QA, Base documental snapshot 2026-06-04, Index rag-novatech-hml-v1.

---

## 3) Estratégia de testes por etapa do pipeline

### 3.1 Testes de ingestão
#### Objetivo
Verificar extração, conversão e indexação corretas dos documentos.

#### Casos de teste
| ID | Cenário | Entrada | Resultado esperado | Evidência | Tipo (Manual/Auto) |
|----|---------|---------|--------------------|-----------|--------------------|
| ING-01 | Documento novo indexado | Arquivo novo em fonte oficial | Documento aparece no indice com metadados corretos (origem, data, versao, hash) | Log de ingestao + consulta no indice + hash do arquivo | Auto |
| ING-02 | Documento atualizado substitui versão | Nova versao de PROC-042-v2 | Conteudo novo indexado e versao antiga marcada para desempate por vigencia | Comparativo de chunks antes/depois + metadado de vigencia | Auto |
| ING-03 | Falha de parsing | PDF com tabela complexa | Erro registrado, sem quebrar pipeline, item vai para fila de correcao | Log de erro + ticket automatico criado | Auto |
| ING-04 | Ingestao SharePoint | PDF/Word da biblioteca oficial | Extracao completa de texto e metadado de origem=SharePoint | Log do conector + amostra de chunks | Auto |
| ING-05 | Ingestao Confluence | Pagina wiki com tabela e links internos | Conversao correta de markdown/texto e metadado de origem=Confluence | Log do conector + diff de conversao | Auto |
| ING-06 | Ingestao pasta de rede | Planilha de referencia mensal | Valores tabulares convertidos sem perda de coluna critica | Log ETL + validacao de campos obrigatorios | Auto |

### 3.2 Testes de retrieval
#### Objetivo
Garantir recuperação dos chunks corretos para perguntas conhecidas.

#### Pares pergunta -> chunk esperado (min. 5)
| ID | Pergunta | Chunks esperados | Critério de aprovação | Tipo |
|----|----------|------------------|-----------------------|------|
| RET-01 | Qual o prazo de devolução? | POL-001-A, POL-001-B | Ambos os chunks no top-5; se faltar POL-001-B o teste falha | Auto |
| RET-02 | Posso devolver carga perigosa? | POL-001-B | POL-001-B no top-3 e sem priorizar resposta positiva | Auto |
| RET-03 | Qual o SLA do cliente Gold? | SLA-2024-B | SLA-2024-B no top-3 e sem conflito com tiers inexistentes | Auto |
| RET-04 | Qual o SLA do cliente Platinum? | SLA-2024-A | SLA-2024-A no top-3 contendo negacao de tier extra | Auto |
| RET-05 | Frete para 600kg para Manaus? | PROC-042v2-B, PROC-042v2-A | Chunks v2 no top-5 e chunk v1 nao pode rankear acima de v2 | Auto |
| RET-06 | Qual o multiplicador para o Sudeste? | PROC-042v2-B | Resgate do valor 1.1 via v2; sem mistura de 1.0 da v1 | Auto |
| RET-07 | Carga perigosa com frete expresso? | FAQ-32 (secundario) + ausencia de PROC/POL formal especifica | FAQ pode aparecer, mas resposta final deve sinalizar que e fonte informal e requer confirmacao oficial | Manual + Auto |

### 3.3 Testes de geração
#### Objetivo
Avaliar a qualidade da resposta quando os chunks corretos já foram recuperados.

#### Casos de teste
| ID | Cenário | Entrada (pergunta + chunks) | Resultado esperado | Risco principal | Tipo |
|----|---------|------------------------------|--------------------|-----------------|------|
| GEN-01 | Resposta factual com fonte | Pergunta simples + chunk unico | Resposta correta, objetiva e com citacao da fonte | Omissao de citacao | Auto |
| GEN-02 | Pergunta sem cobertura | Pergunta fora da base | Recusa apropriada, sem inventar valor/prazo, orientando proximo passo | Alucinacao | Auto |
| GEN-03 | Contradição entre versões | Chunks v1 e v2 juntos | Explicar regra de transicao e priorizar versao vigente pelo contexto temporal | Mistura de regras | Manual + Auto |
| GEN-04 | Fonte informal em tema critico | Pergunta critica com chunk principal vindo do FAQ | Resposta inclui alerta de baixa confiabilidade e direciona para validacao em fonte normativa | Confianca alta em fonte fraca | Manual + Auto |

### 3.4 Testes de contexto
#### Objetivo
Validar orçamento de contexto e efeitos de contexto em conversas reais.

#### Casos de teste
| ID | Cenário | Estratégia | Resultado esperado | Métrica | Tipo |
|----|---------|------------|--------------------|---------|------|
| CTX-01 | Context rot em conversa longa | Sessao com 8 turnos, retomando regra critica no fim | Ultimo turno mantem regra correta de excecao | Acuracia no ultimo turno >= 90% | Auto |
| CTX-02 | Lost in the middle | Mover instrucao critica para inicio/meio/fim | Queda de acuracia no meio <= 10 p.p. | Delta de acuracia por posicao | Auto |
| CTX-03 | Chunk errado | Injetar chunk irrelevante no top-k | Resposta ignora chunk irrelevante quando contraditorio | Taxa de erro por contaminacao <= 5% | Manual + Auto |
| CTX-04 | Context overflow | Forcar contexto acima da janela do modelo | Sistema resume/reduz contexto sem perder chunks criticos | Taxa de truncamento com erro <= 3% | Auto |

### 3.5 Testes de ponta a ponta
#### Objetivo
Validar o fluxo completo: pergunta -> retrieval -> geração -> resposta final.

#### Casos de teste
| ID | Pergunta | Resultado esperado | Critério de aceite | Tipo |
|----|----------|--------------------|--------------------|------|
| E2E-01 | Qual o prazo de devolução? | Regra correta + excecao de carga perigosa + fonte | Erro factual <= 2% e citacao correta >= 95% em 50 execucoes | Auto |
| E2E-02 | Posso devolver carga perigosa? | Negar processo padrao e orientar Gestao de Riscos | Taxa de inversao de regra <= 2% em 50 execucoes | Auto |
| E2E-03 | Qual o SLA do cliente Platinum? | Negar tier inexistente e orientar tiers validos | Taxa de alucinacao <= 2% em 50 execucoes | Auto |
| E2E-04 | Frete 600kg Manaus | Usar multiplicador v2 (Norte 1.8) | Uso de valor antigo <= 5% em 50 execucoes | Auto |
| E2E-05 | Frete 300kg Salvador | Informar ausencia de cobertura para <500kg | Inventar formula/valor <= 2% em 50 execucoes | Auto |
| E2E-06 | Carga perigosa com frete expresso? | Tratar FAQ como apoio, nao como regra normativa final | 100% das respostas devem sinalizar natureza informal da fonte e recomendar validacao formal | Manual + Auto |

### 3.6 Testes de regressão
#### Objetivo
Definir o conjunto mínimo de testes automáticos após mudanças.

#### Gatilhos e suíte
| Gatilho | Testes que devem rodar | Critério de bloqueio de release |
|---------|-------------------------|----------------------------------|
| Mudança de prompt | RET criticos + GEN completo + CTX-01/02 + E2E smoke | Qualquer regressao em regras normativas ou queda > 10% em acuracia |
| Atualização de documento | ING relacionado + RET do dominio alterado + E2E focal | Divergencia com regra vigente ou fonte incorreta |
| Mudança no índice/embedding | RET completo + CTX completo + E2E completo | Queda > 5 p.p. em retrieval top-5 ou aumento de alucinacao > 2 p.p. |

---

## 4) Organização para acompanhamento (formato Cowork)

### Backlog de testes
| ID | Categoria | Caso de teste | Prioridade | Responsável | Status | Evidência |
|----|-----------|---------------|------------|-------------|--------|-----------|
| ING-01 | Ingestao | Documento novo indexado | Alta | Eng. Dados | Concluido | Log de ingestao e consulta no indice |
| RET-05 | Retrieval | Frete 600kg Manaus com v2 priorizada | Alta | Eng. IA | Em andamento | Relatorio top-k |
| GEN-02 | Geracao | Pergunta sem cobertura sem alucinacao | Alta | QA IA | A fazer | Suite de prompts negativos |
| CTX-02 | Contexto | Lost in the middle | Media | QA IA | A fazer | Benchmark por posicao |
| E2E-03 | E2E | SLA de cliente Platinum | Alta | QA IA | Em andamento | Resultado de 50 execucoes |

### Critérios de prontidão
- [ ] Casos críticos implementados.
- [ ] Evidências anexadas.
- [ ] Regressão automatizada configurada.
- [x] Responsáveis definidos por categoria.

---

## 5) Métricas e saída esperada
- Precisão de retrieval (top-k): alvo >= 90% de presenca de chunk critico no top-5 para perguntas mapeadas.
- Taxa de alucinação em perguntas sem cobertura: alvo <= 3%.
- Taxa de respostas com fonte correta: alvo >= 95%.
- Taxa de falha em contexto longo: alvo <= 10% no ultimo turno.
- Critério final de go/no-go: Go somente se todos os bloqueantes estiverem zerados e todos os alvos minimos de metrica forem atingidos por 2 rodadas consecutivas.

---

## 6) Checklist de entrega
- [x] O plano cobre ingestão, retrieval, geração, contexto, e2e e regressão.
- [x] Há pelo menos 5 pares pergunta -> chunk esperado em retrieval.
- [x] O plano reconhece testes não determinísticos de IA.
- [x] O artefato está estruturado para rastreio (status e responsável).

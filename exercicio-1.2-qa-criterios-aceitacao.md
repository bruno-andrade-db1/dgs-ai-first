# Exercício 1.2 — Design de critérios de aceitação para respostas de IA

## Contexto
Este arquivo organiza o entregável do Exercício 1.2, cobrindo:
- avaliação manual inicial das 5 respostas;
- rubrica de avaliação (4 dimensões, escala 1-3);
- template reutilizável para QA;
- aplicação da rubrica com pontuação final.

---

## 1) Avaliação manual inicial (antes da rubrica)

### Resposta 1
- Pergunta: Qual o prazo de devolução?
- Classificação: Parcialmente correta
- Justificativa baseada no Anexo A: A resposta acerta o prazo geral de 7 dias úteis e menciona corretamente a exceção de cargas perigosas classes 1 a 6. Porém, a fonte citada está incompleta, porque o prazo geral está na seção 3.1 e a exceção na 3.2.
- Evidência (documento e seção/chunk): POL-001 seção 3.1 (prazo geral) + POL-001 seção 3.2 (exceções).

### Resposta 2
- Pergunta: Quanto custa frete para 600kg para Manaus?
- Classificação: Parcialmente correta
- Justificativa baseada no Anexo A: O multiplicador 1.8 para Norte está correto na v2. No entanto, a pergunta pede "quanto custa" e a resposta não calcula o valor final (faltam valor base e explicitação do fator de peso, mesmo que para 600kg seja 1.0).
- Evidência (documento e seção/chunk): PROC-042-v2 seção 2 (fórmula e fator de peso) + seção 2.1 (multiplicadores; Norte = 1.8).

### Resposta 3
- Pergunta: Qual o SLA do cliente Platinum?
- Classificação: Incorreta
- Justificativa baseada no Anexo A: O tier Platinum não existe na NovaTech. Portanto, os tempos informados (1h e 12h) são alucinação.
- Evidência (documento e seção/chunk): SLA-2024 seção 1 (somente Gold, Silver e Standard; nota explicita que nao existem outros tiers).

### Resposta 4
- Pergunta: Posso devolver carga perigosa?
- Classificação: Incorreta
- Justificativa baseada no Anexo A: A resposta inverte a regra. Cargas perigosas classes 1 a 6 nao sao elegiveis para devolucao pelo processo padrao.
- Evidência (documento e seção/chunk): POL-001 seção 3.2 (exceções ao prazo geral).

### Resposta 5
- Pergunta: Qual o multiplicador de frete para o Sudeste?
- Classificação: Correta
- Justificativa baseada no Anexo A: O multiplicador do Sudeste na versão revisada é 1.1, exatamente como informado na resposta.
- Evidência (documento e seção/chunk): PROC-042-v2 seção 2.1 (multiplicadores regionais atualizados).

---

## 2) Rubrica de avaliação (gerada com Claude)

### Dimensão 1: Precisão factual
- Nota 1: Conteudo factualmente incorreto, inventado, ou contraditorio em ponto critico.
- Nota 2: Conteudo majoritariamente correto, mas com omissao relevante, ambiguidade, ou pequeno erro nao critico.
- Nota 3: Conteudo correto, sem erros factuais relevantes para a pergunta.

### Dimensão 2: Citação de fonte
- Nota 1: Sem fonte, fonte inexistente, ou fonte citada contradiz a resposta.
- Nota 2: Fonte citada parcialmente correta (documento correto, secao imprecisa, ou faltando complementar).
- Nota 3: Fonte oficial correta, especifica e suficiente para sustentar a resposta.

### Dimensão 3: Aderência aos guardrails
- Nota 1: Viola guardrail critico (inventa valor/prazo, nao admite ausencia de informacao, idioma inadequado).
- Nota 2: Atende parcialmente aos guardrails, com falha leve de forma ou de completude.
- Nota 3: Atende totalmente aos guardrails definidos (fonte, nao inventar, explicitar ausencia quando aplicavel, portugues formal).

### Dimensão 4: Completude e utilidade
- Nota 1: Resposta inutil para decisao operacional (incompleta de forma critica).
- Nota 2: Resposta util, mas incompleta em ponto importante para execucao.
- Nota 3: Resposta completa, acionavel e adequada ao contexto da pergunta.

### Regra de pontuação total
- Soma máxima por resposta: 12 pontos.
- Faixas sugeridas (exemplo):
  - 10 a 12: Aprovada
  - 7 a 9: Revisar
  - 4 a 6: Reprovada
- Critério de aprovação/reprovação: Qualquer resposta com erro factual critico em regra normativa (nota 1 em Precisao factual) nao pode ser considerada Aprovada, mesmo com total >= 10.

---

## 3) Template reutilizável de avaliação (formato QA)

Use este template para qualquer lote futuro de respostas.

### Cabeçalho de execução
- Data: 04/06/2026
- Avaliador(a): Bruno Andrade
- Versão do prompt: Exercicio 1.2 v1
- Versão da base documental: Anexo A (POL-001 v3.1, PROC-042 v1.0, PROC-042-v2 v2.0, SLA-2024 v2024.1, FAQ nao controlado)
- Lote avaliado: 5 respostas simuladas do assistente

### Tabela de avaliação
| ID | Pergunta | Resumo da resposta | Precisão factual (1-3) | Fonte (1-3) | Guardrails (1-3) | Completude (1-3) | Total (4-12) | Resultado (Aprovada/Revisar/Reprovada) | Observações |
|----|----------|--------------------|-------------------------|-------------|------------------|------------------|--------------|------------------------------------------|-------------|
| 1  | Qual o prazo de devolucao? | 7 dias uteis com excecao para carga perigosa | 3 | 2 | 3 | 2 | 10 | Aprovada | Fonte parcial (3.1 + 3.2), resposta util mas nao completa em detalhamento de procedimento |
| 2  | Quanto custa frete para 600kg para Manaus? | Multiplicador Norte 1.8 sobre valor base | 2 | 2 | 3 | 2 | 9 | Revisar | Faltou calculo final do custo e explicitar fator de peso |
| 3  | Qual o SLA do cliente Platinum? | Informa SLA de tier inexistente | 1 | 1 | 1 | 1 | 4 | Reprovada | Alucinacao: Platinum nao existe |
| 4  | Posso devolver carga perigosa? | Diz que sim em ate 7 dias | 1 | 1 | 1 | 1 | 4 | Reprovada | Inversao de regra normativa da POL-001 |
| 5  | Qual o multiplicador de frete para o Sudeste? | Multiplicador 1.1 | 3 | 3 | 3 | 2 | 11 | Aprovada | Correta e acionavel; poderia citar transicao de versao em contexto ambíguo |

### Regras de decisão do lote
- Taxa minima de aprovacao: 80% de respostas Aprovadas e nenhuma falha bloqueante.
- Erros bloqueantes (exemplo): alucinacao de tier/SLA; inversao de regra normativa; inventar prazo ou valor sem respaldo.
- Acao recomendada para respostas reprovadas: ajustar prompt e retrieval, reexecutar lote completo e abrir acao corretiva para casos bloqueantes.

---

## 4) Aplicação da rubrica às 5 respostas simuladas

### Resultado por resposta
- Resposta 1:
  - Notas por dimensão: Precisao 3, Fonte 2, Guardrails 3, Completude 2
  - Total: 10
  - Resultado: Aprovada
  - Comentário: Correta no essencial; falta precisao de citacao (secao 3.1 tambem deveria ser referenciada).

- Resposta 2:
  - Notas por dimensão: Precisao 2, Fonte 2, Guardrails 3, Completude 2
  - Total: 9
  - Resultado: Revisar
  - Comentário: Acerta multiplicador da v2, mas nao responde plenamente o "quanto custa".

- Resposta 3:
  - Notas por dimensão: Precisao 1, Fonte 1, Guardrails 1, Completude 1
  - Total: 4
  - Resultado: Reprovada
  - Comentário: Alucinacao grave; deveria negar o tier Platinum com base na SLA-2024.

- Resposta 4:
  - Notas por dimensão: Precisao 1, Fonte 1, Guardrails 1, Completude 1
  - Total: 4
  - Resultado: Reprovada
  - Comentário: Contradiz regra explicita da POL-001; erro bloqueante.

- Resposta 5:
  - Notas por dimensão: Precisao 3, Fonte 3, Guardrails 3, Completude 2
  - Total: 11
  - Resultado: Aprovada
  - Comentário: Resposta correta e util para consulta direta do multiplicador.

### Síntese do lote
- Média geral: 7,6 / 12
- Principais falhas encontradas: alucinacao de tier inexistente; inversao de regra de devolucao de carga perigosa; resposta incompleta para pergunta de custo.
- Recomendações para melhoria do assistente: reforcar guardrail de nao-invencao para entidades nao existentes; priorizar chunks normativos de excecao (POL-001 3.2); incluir padrao de resposta para perguntas de custo exigindo formula completa e variaveis faltantes.

---

## 5) Checklist de entrega
- [x] Fiz a avaliação manual antes da rubrica.
- [x] A rubrica tem 4 dimensões com escala 1-3 bem definida.
- [x] O template é reutilizável para novos lotes.
- [x] Apliquei a rubrica nas 5 respostas.
- [x] As respostas sobre Platinum e devolução de carga perigosa foram tratadas corretamente.

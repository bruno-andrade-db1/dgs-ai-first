# Exercício 3.1 — Revisão crítica das respostas do assistente

## Tópico
Revisão Crítica de Outputs de IA

## Contexto
Antes do go-live, você executa uma bateria de avaliação sobre as respostas do assistente e produz um parecer.

## Ferramentas a utilizar
- Claude (chat)
- Claude Cowork

## Inputs fornecidos
- O cenário completo.
- A documentação da NovaTech (Anexo A) como fonte de verdade.
- A rubrica de avaliação criada no cenário 2 (4 dimensões: precisão factual, citação de fonte, aderência a guardrails, completude; escala 1-3 cada).
- 8 respostas do assistente em staging (simuladas):

| # | Pergunta | Resposta Resumida | Fonte | Esperado |
|---|----------|-------------------|-------|----------|
| 1 | "Prazo de devolução?" | 7 dias, exceto perigosas | POL-001 | Aprovada |
| 2 | "Devolução carga perigosa?" | Não é possível, escalar supervisor | POL-001 | Aprovada |
| 3 | "SLA Gold resolução?" | 24h | SLA-2024 | Aprovada |
| 4 | "SLA Platinum?" | Tier não encontrado, sugere verificar | — | Aprovada (reconheceu) |
| 5 | "Frete 600kg Manaus?" | Multiplicador 1.8 | PROC-042-v2 | Aprovada |
| 6 | "Frete 600kg sem destino?" | "O frete para o Sudeste é 1.1" (assumiu Sudeste sem ser informado) | PROC-042-v2 | Reprovada (assumiu dado) |
| 7 | "Receita de bolo?" | "Não tenho informações sobre receitas. Posso ajudar com logística." | — | Aprovada (escopo) |
| 8 | "What is the return policy?" | Responde em inglês | POL-001 | Reprovada (idioma) |

## Tarefa
1. Aplique a rubrica a cada uma das 8 respostas. Pontue as dimensões e calcule o score.
2. Faça isso PRIMEIRO por conta própria, DEPOIS use o Claude para uma segunda avaliação e compare.
3. Usando o Claude Cowork, gere um relatório de qualidade curto: score médio, respostas reprovadas com motivo, e parecer de go-live (pronto? com quais ressalvas?).

## Entregável
- Sua avaliação.
- Avaliação do Claude.
- Comparação entre as avaliações.
- Relatório do Cowork com parecer.

## Critérios de avaliação
- As respostas 6 e 8 são identificadas como reprovadas (assunção indevida e idioma errado).
- A rubrica é aplicada de forma consistente (mesma régua para todas).
- O parecer de go-live é fundamentado nos dados e pragmático.

---

## 1) Rubrica aplicada (base: Exercício 1.2)

### Dimensões (escala 1 a 3)
- Precisão factual
- Citação de fonte
- Aderência a guardrails
- Completude

### Regra de score
- Total por resposta: 4 a 12.
- Faixas:
	- 10 a 12: Aprovada
	- 7 a 9: Revisar
	- 4 a 6: Reprovada

### Regras bloqueantes
- Erro factual crítico em regra normativa: Reprovada, independentemente do total.
- Violação crítica de guardrail (ex.: assunção indevida de dado e idioma inadequado): Reprovada, independentemente do total.

---

## 2) Minha avaliação (primeira avaliação)

| # | Pergunta | Precisão | Fonte | Guardrails | Completude | Total | Resultado | Justificativa resumida |
|---|----------|----------|-------|------------|------------|-------|-----------|------------------------|
| 1 | Prazo de devolução? | 3 | 2 | 3 | 2 | 10 | Aprovada | Correta no essencial: 7 dias úteis e exceção para perigosas; fonte genérica sem seção. |
| 2 | Devolução carga perigosa? | 3 | 3 | 2 | 2 | 10 | Aprovada | Regra principal correta (não elegível no processo padrão), mas faltou orientar explicitamente Gestão de Riscos (ramal 4500). |
| 3 | SLA Gold resolução? | 3 | 3 | 3 | 2 | 11 | Aprovada | Valor correto (24h úteis); poderia explicitar que é chamado geral para ficar totalmente completo. |
| 4 | SLA Platinum? | 3 | 1 | 3 | 3 | 10 | Aprovada | Reconhece tier inexistente corretamente; faltou citação de fonte formal (SLA-2024 seção de tiers). |
| 5 | Frete 600kg Manaus? | 3 | 3 | 3 | 2 | 11 | Aprovada | Multiplicador 1.8 correto na v2; incompleta para "quanto custa" sem valor base/fórmula final. |
| 6 | Frete 600kg sem destino? | 1 | 2 | 1 | 1 | 5 | Reprovada | Assumiu Sudeste sem dado de entrada. Violação crítica de guardrail e resposta não confiável. |
| 7 | Receita de bolo? | 3 | 2 | 3 | 3 | 11 | Aprovada | Recusa adequada por escopo e redireciona para domínio correto (logística). |
| 8 | What is the return policy? | 2 | 3 | 1 | 1 | 7 | Reprovada | Conteúdo pode estar correto, mas falha crítica de guardrail por idioma de saída inadequado para o contexto definido. |

### Síntese da minha avaliação
- Score médio: 9,38/12.
- Aprovadas: 6/8 (75%).
- Reprovadas: 2/8 (25%) -> respostas 6 e 8.

---

## 3) Segunda avaliação (Claude)

| # | Precisão | Fonte | Guardrails | Completude | Total | Resultado Claude |
|---|----------|-------|------------|------------|-------|------------------|
| 1 | 3 | 2 | 3 | 2 | 10 | Aprovada |
| 2 | 3 | 2 | 2 | 2 | 9 | Revisar |
| 3 | 3 | 3 | 3 | 2 | 11 | Aprovada |
| 4 | 3 | 1 | 3 | 2 | 9 | Revisar |
| 5 | 3 | 3 | 3 | 2 | 11 | Aprovada |
| 6 | 1 | 1 | 1 | 1 | 4 | Reprovada |
| 7 | 3 | 2 | 3 | 2 | 10 | Aprovada |
| 8 | 2 | 2 | 1 | 1 | 6 | Reprovada |

### Síntese da avaliação do Claude
- Score médio: 8,75/12.
- Reprovadas: 6 e 8.

---

## 4) Comparação (minha avaliação x Claude)

- Convergência total nos casos críticos: ambos reprovam 6 (assunção indevida) e 8 (idioma inadequado).
- Diferenças de severidade em casos não críticos:
	- Resposta 2: eu aprovei (10), Claude marcou Revisar (9) por falta de orientação de escalonamento mais específica.
	- Resposta 4: eu aprovei (10), Claude marcou Revisar (9) por ausência de fonte explícita.
- Tendência geral: Claude foi mais conservador em fonte/completude; eu fui mais orientado a aderência funcional da resposta ao esperado.

---

## 5) Relatório curto de qualidade (estilo Claude Cowork)

### Score médio
- Referência principal (minha avaliação): 9,38/12.
- Contraprova (Claude): 8,75/12.

### Respostas reprovadas e motivo
- #6: Reprovada por assunção indevida de dado ausente (destino não informado) e inferência não autorizada.
- #8: Reprovada por violação de guardrail de idioma (resposta em inglês em contexto operacional esperado em português).

### Parecer de go-live
- Status: Pronto com ressalvas, não pronto para go-live irrestrito.
- Fundamentação:
	- Taxa de aprovação de 75% (abaixo da meta típica de 80% para operação sem supervisão).
	- Há 2 falhas bloqueantes de governança (assunção de dado e idioma).
	- Em domínio regulado (logística com regras normativas), falhas desse tipo exigem contenção antes de escalar uso.

### Ressalvas e ações pragmáticas pré-go-live
1. Bloquear inferência quando parâmetros obrigatórios estiverem ausentes (ex.: destino para cálculo de frete).
2. Forçar idioma de saída em português no prompt do sistema e no pós-processamento.
3. Exigir campo estruturado de fonte e evidência mínima para respostas normativas.
4. Reexecutar bateria com os mesmos 8 casos e só liberar quando 6 e 8 passarem sem regressão.

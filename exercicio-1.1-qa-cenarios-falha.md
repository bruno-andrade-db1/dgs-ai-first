# Exercício 1.1 — Identificação de cenários de falha de IA

## 1) Lista inicial (feita sem IA)

### Cenário 1
- Categoria: Alucinação
- Origem: Humano (sem IA)
- Pergunta de teste: Qual o SLA do cliente Platinum?
- Comportamento esperado: Informar que o tier Platinum não existe e citar os tiers válidos (Gold, Silver e Standard), com fonte.
- Comportamento indesejado: Inventar tempos de SLA para Platinum.
- Como verificar: Conferir se a resposta nega explicitamente a existência de Platinum e referencia o documento de classificação de clientes.

### Cenário 2
- Categoria: Alucinação
- Origem: Humano (sem IA)
- Pergunta de teste: Qual o multiplicador para frete de 300kg para Salvador?
- Comportamento esperado: Informar ausência de regra documentada para frete especial abaixo de 500kg e não inventar valor.
- Comportamento indesejado: Responder com multiplicador numérico como se fosse regra oficial.
- Como verificar: Detectar se a resposta traz número/fórmula para <500kg sem ressalva de ausência de cobertura.

### Cenário 3
- Categoria: Informação desatualizada/contraditória
- Origem: Humano (sem IA)
- Pergunta de teste: Qual o multiplicador de frete para o Norte em chamado aberto hoje (600kg)?
- Comportamento esperado: Usar a versão revisada PROC-042-v2 e retornar 1.8.
- Comportamento indesejado: Usar 1.6 (versão antiga) sem justificativa de transição.
- Como verificar: Comparar resposta com regra vigente para chamados novos e validar consistência da versão citada.

### Cenário 4
- Categoria: Falha de contexto (context rot)
- Origem: Humano (sem IA)
- Pergunta de teste: Em conversa de 5 turnos, após discussão inicial sobre devolução de carga perigosa, confirmar no final: Posso devolver carga perigosa pelo processo padrão?
- Comportamento esperado: Manter resposta consistente de que não é elegível no processo padrão.
- Comportamento indesejado: Esquecer o contexto anterior e afirmar que pode devolver em 7 dias.
- Como verificar: Rodar teste multi-turn fixo e validar a resposta do último turno contra a regra de exceção.

## 2) Cenários adicionais gerados com Claude

### Cenário 5
- Categoria: Falha de contexto (lost in the middle)
- Origem: Claude
- Pergunta de teste: Prompt extenso com a condição no meio do texto: chamado aberto após 01/12/2023; ao final perguntar qual multiplicador do Norte.
- Comportamento esperado: Capturar a condição temporal e aplicar v2 (1.8).
- Comportamento indesejado: Ignorar a informação do meio e responder com valor antigo.
- Como verificar: Executar o mesmo teste com a condição no início, meio e fim; comparar queda de acurácia no caso do meio.

### Cenário 6
- Categoria: Falha de contexto (chunk errado)
- Origem: Claude
- Pergunta de teste: Frete para 600kg para Manaus.
- Comportamento esperado: Priorizar chunks da PROC-042-v2 e responder com multiplicadores atuais.
- Comportamento indesejado: Resposta contaminada por chunk da PROC-042 v1, misturando valores.
- Como verificar: Inspecionar logs de retrieval top-k e validar presença obrigatória de chunks v2 no contexto final.

### Cenário 7
- Categoria: Recusa inadequada
- Origem: Claude
- Pergunta de teste: Qual o multiplicador regional para o Sudeste no frete especial acima de 500kg?
- Comportamento esperado: Responder 1.1 com fonte atualizada.
- Comportamento indesejado: Dizer que não encontrou a informação.
- Como verificar: Para perguntas com chunk obrigatório conhecido, marcar qualquer recusa como falha.

### Cenário 8
- Categoria: Falha de guardrail
- Origem: Claude
- Pergunta de teste: Posso devolver carga perigosa?
- Comportamento esperado: Resposta em português formal, com citação de fonte, sem inventar prazo/valor.
- Comportamento indesejado: Responder em inglês, sem fonte, ou adicionar informação não fundamentada.
- Como verificar: Validar idioma, formato de citação e aderência aos guardrails por checklist automatizado.

## 3) Lista final consolidada (mínimo 10)

## Alucinação (mínimo 3)

### Cenário 1 (consolidado do Cenário 1 da seção 1)
- Categoria: Alucinação
- Origem: Humano (sem IA)
- Pergunta de teste: Qual o SLA do cliente Platinum?
- Comportamento esperado: Negar a existência de Platinum e informar tiers válidos.
- Comportamento indesejado: Criar SLA fictício para Platinum.
- Como verificar: Regra de validação textual para bloquear qualquer SLA associado a Platinum.

### Cenário 2 (consolidado do Cenário 2 da seção 1)
- Categoria: Alucinação
- Origem: Humano (sem IA)
- Pergunta de teste: Qual o multiplicador para frete de 300kg para Salvador?
- Comportamento esperado: Informar ausência de cobertura documental para <500kg.
- Comportamento indesejado: Inventar multiplicador/fórmula.
- Como verificar: Teste sem cobertura: resposta não pode conter valor numérico normativo sem ressalva.

### Cenário 3 (novo)
- Categoria: Alucinação
- Origem: Consolidado (novo)
- Pergunta de teste: Posso devolver carga perigosa classe 3 em até 7 dias úteis?
- Comportamento esperado: Dizer que não é elegível no processo padrão e orientar para Gestão de Riscos.
- Comportamento indesejado: Afirmar que a devolução padrão é permitida.
- Como verificar: Classificador semântico para regra crítica de devolução de carga perigosa.

## Informação desatualizada ou contraditória (mínimo 2)

### Cenário 4 (consolidado do Cenário 3 da seção 1)
- Categoria: Informação desatualizada/contraditória
- Origem: Humano (sem IA)
- Pergunta de teste: Qual o multiplicador do Norte para chamado novo de frete especial?
- Comportamento esperado: Retornar 1.8 da PROC-042-v2.
- Comportamento indesejado: Retornar 1.6 da PROC-042 v1.
- Como verificar: Validação por data de abertura do chamado e comparação com matriz oficial vigente.

### Cenário 5 (novo)
- Categoria: Informação desatualizada/contraditória
- Origem: Consolidado (novo)
- Pergunta de teste: Passe a fórmula e os multiplicadores atuais do frete especial.
- Comportamento esperado: Resposta consistente em uma única versão (v2) ou explicação clara de transição.
- Comportamento indesejado: Misturar fator de peso da v2 com multiplicadores da v1.
- Como verificar: Extrair números da resposta e validar consistência contra uma única tabela de versão.

## Falha de contexto (mínimo 3)

### Cenário 6 (consolidado do Cenário 4 da seção 1)
- Categoria: Falha de contexto
- Subtipo: Context rot / Lost in the middle / Chunk errado / Overflow
- Origem: Humano (sem IA)
- Pergunta de teste: Conversa longa no Teams com 5 perguntas; na 5a, voltar a perguntar sobre devolução de carga perigosa.
- Comportamento esperado: Preservar regra de não elegibilidade no processo padrão.
- Comportamento indesejado: Repetir informação equivocada do histórico e ignorar chunks atuais.
- Como verificar: Teste multi-turn com avaliação apenas do último turno.

### Cenário 7 (consolidado do Cenário 5 da seção 2)
- Categoria: Falha de contexto
- Subtipo: Context rot / Lost in the middle / Chunk errado / Overflow
- Origem: Claude
- Pergunta de teste: Pergunta com muitos detalhes e condição-chave no meio: chamado aberto após 01/12/2023.
- Comportamento esperado: Usar a condição temporal corretamente e aplicar v2.
- Comportamento indesejado: Ignorar detalhe do meio (lost in the middle).
- Como verificar: Benchmark A/B mudando posição da condição no prompt e comparando acurácia.

### Cenário 8 (consolidado do Cenário 6 da seção 2)
- Categoria: Falha de contexto
- Subtipo: Context rot / Lost in the middle / Chunk errado / Overflow
- Origem: Claude
- Pergunta de teste: Prazo de devolução + carga perigosa + frete especial para 600kg no Norte.
- Comportamento esperado: Combinar corretamente múltiplos domínios sem truncar regras críticas.
- Comportamento indesejado: Resposta parcial por overflow ou contaminada por chunk irrelevante.
- Como verificar: Monitorar orçamento de tokens e checar se todos os chunks obrigatórios foram recuperados.

## Recusa inadequada (mínimo 1)

### Cenário 9 (consolidado do Cenário 7 da seção 2)
- Categoria: Recusa inadequada
- Origem: Claude
- Pergunta de teste: Qual o multiplicador regional para o Sudeste no frete especial acima de 500kg?
- Comportamento esperado: Responder com valor vigente e fonte.
- Comportamento indesejado: Recusar resposta afirmando que não há informação.
- Como verificar: Teste de cobertura com mapa pergunta -> chunk esperado.

## Falha de guardrail (mínimo 1)

### Cenário 10 (consolidado do Cenário 8 da seção 2)
- Categoria: Falha de guardrail
- Origem: Claude
- Pergunta de teste: Posso devolver carga perigosa?
- Comportamento esperado: Português formal, fonte citada e ausência de invenção.
- Comportamento indesejado: Sem fonte, em outro idioma, ou com afirmação não fundamentada.
- Como verificar: Regras automáticas para idioma e presença de citação + auditoria manual por amostragem.

---

## 4) Checklist rápido de validação do entregável

- [x] Criei pelo menos 4 cenários sem IA antes de usar Claude.
- [x] Claude adicionou pelo menos 4 cenários novos.
- [x] Lista final tem pelo menos 10 cenários.
- [x] Atendi os mínimos por categoria (3/2/3/1/1).
- [x] Cada cenário tem pergunta de teste, esperado, indesejado e verificação.
- [x] Pelo menos metade dos cenários tem proposta de automação.
- [x] Marquei claramente a origem de cada cenário (Humano ou Claude).

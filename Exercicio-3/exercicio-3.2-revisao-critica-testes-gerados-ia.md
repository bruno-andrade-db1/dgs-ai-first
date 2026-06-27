# Exercicio 3.2 — Revisao critica dos testes gerados por IA

## Topico
Revisao Critica de Outputs de IA

## Contexto
Alguns testes de integracao foram gerados por IA. O objetivo e verificar se eles realmente validam o comportamento esperado do dominio e se estao alinhados ao contexto tecnico do projeto.

## Ferramenta a utilizar
- Claude (chat)

## Inputs considerados
- Cenario 3 (fase de governanca e validacao).
- Requisito do cenario: framework oficial de testes e Vitest (nao jest).
- 3 testes simulados fornecidos no enunciado.

---

## 1) Minha revisao dos 3 testes

### Teste 1 — assertions vagas
Trecho avaliado:

```typescript
describe('query endpoint', () => {
  it('should return a response', async () => {
    const res = await request(app).post('/api/query').send({ question: 'prazo devolucao' });
    expect(res.status).toBe(200);
    expect(res.body).toBeDefined();
  });
});
```

O que testa:
- Verifica apenas que a rota responde com status 200.
- Verifica apenas que existe algum corpo na resposta.

O que falha em testar:
- Nao valida se a resposta esta correta para o dominio (prazo de 7 dias uteis).
- Nao valida citacao de fonte (ex.: POL-001).
- Nao valida campos obrigatorios (ex.: answer, source_document, confidence).

Risco se passar com codigo errado:
- Alto. O endpoint pode devolver texto incorreto/alucinado e ainda assim o teste passa.

Classificacao:
- Insuficiente.

---

### Teste 2 — edge case isolado e sem dominio
Trecho avaliado:

```typescript
describe('query endpoint edge cases', () => {
  it('should handle empty question', async () => {
    const res = await request(app).post('/api/query').send({ question: '' });
    expect(res.status).toBe(400);
  });
});
```

O que testa:
- Valida corretamente um caso basico de entrada invalida (question vazia).

O que falha em testar:
- Nao cobre casos reais de negocio/logistica.
- Nao valida recuperacao de informacao correta para perguntas validas.
- Nao testa comportamento de guardrail (idioma, nao inventar dados, ausencia de cobertura documental).

Risco se passar com codigo errado:
- Medio/alto. A API pode tratar input vazio e mesmo assim falhar no principal: responder corretamente perguntas do dominio.

Classificacao:
- Incompleto.

---

### Teste 3 — mock perigoso e framework errado
Trecho avaliado:

```typescript
describe('feedback endpoint', () => {
  it('should save feedback', async () => {
    const mockCreate = jest.fn().mockResolvedValue({ id: '123' });
    const res = await request(app).post('/api/feedback').send({
      queryId: 'q1', rating: 5, comment: 'great'
    });
    expect(res.status).toBe(200);
    expect(mockCreate).toHaveBeenCalled();
  });
});
```

O que testa:
- Tenta validar o fluxo de persistencia de feedback.

O que falha em testar:
- O mock nao esta claramente injetado no componente real; pode nao estar ligado ao caminho executado.
- Nao valida persistencia real nem contrato de dados.
- Nao valida cenarios invalidos (rating fora da faixa, queryId ausente, comentario muito longo).
- Inconsistencia de framework: usa jest.fn() em projeto que deve usar Vitest (vi.fn()).

Risco se passar com codigo errado:
- Alto. Pode haver falso positivo mesmo sem validacao de input ou sem gravacao correta.

Classificacao:
- Perigoso.

---

## 2) Segunda revisao (Claude) e comparacao

### Resumo da revisao do Claude (simulada)
- Teste 1: insuficiente por validar existencia da resposta, nao acuracia.
- Teste 2: util para validacao de contrato HTTP basico, mas fraco para validar regras de negocio.
- Teste 3: fragil por mock permissivo e erro de stack de teste (jest vs Vitest).

### Comparacao com minha revisao
- Concordancia total nos tres pontos centrais:
  1. Teste 1 nao prova corretude da resposta.
  2. Teste 2 cobre apenas validacao superficial de input.
  3. Teste 3 gera risco de falso positivo e esta inconsistente com o framework.
- Diferenca de enfase:
  - Minha revisao enfatiza mais risco de governanca (guardrails, fonte, alucinacao).
  - Claude enfatiza mais qualidade tecnica de isolamento e estrutura de teste.

---

## 3) Teste 1 reescrito (versao melhor)

Objetivo da reescrita:
- Verificar conteudo semantico da resposta (prazo correto).
- Verificar citacao de fonte.
- Manter compatibilidade com Vitest.

```typescript
import request from 'supertest';
import { describe, it, expect } from 'vitest';
import { app } from '../src/app';

describe('query endpoint', () => {
  test('deve responder prazo de devolucao correto com fonte', async () => {
    const res = await request(app)
      .post('/api/query')
      .send({ question: 'Prazo de devolucao?' });

    expect(res.status).toBe(200);

    // Contrato minimo esperado
    expect(res.body).toMatchObject({
      answer: expect.any(String),
      source_document: expect.any(String),
    });

    // Conteudo de dominio: POL-001 define 7 dias uteis
    expect(res.body.answer.toLowerCase()).toContain('7');
    expect(res.body.answer.toLowerCase()).toContain('dias uteis');

    // Fonte esperada
    expect(res.body.source_document).toBe('POL-001');
  });
});
```

Observacao:
- Se a API retornar fontes como array (ex.: sources), adaptar para validar inclusao de POL-001 nesse array.

---

## 4) Conclusao do exercicio

- O Teste 1 original e insuficiente.
- O Teste 2 original e incompleto para validar o dominio.
- O Teste 3 original e perigoso e inconsistente com o stack (usa jest em vez de Vitest).
- A versao reescrita do Teste 1 melhora a confiabilidade por validar conteudo e fonte, nao apenas existencia de resposta.

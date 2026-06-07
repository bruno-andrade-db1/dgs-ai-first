# Exercicio 2.1 - Contribuicao para o AGENTS.md: Testing Standards

## Objetivo
Produzir os 3 entregaveis do exercicio:
1. Secao "Testing Standards" para o AGENTS.md.
2. Reescrita de um teste ruim (before/after) com explicacoes objetivas.
3. Criterios de code review de QA para testes gerados por IA.

## Base de referencia (Anexos A e B)
- Fonte normativa: POL-001, PROC-042-v2 e SLA-2024.
- Fonte informal: FAQ-Atendimento (pode apoiar contexto, nao pode sobrepor regra formal).
- Chunks de teste devem usar IDs reais do Anexo B (ex.: POL-001-B, SLA-2024-B, PROC-042v2-B).

## Contexto tecnico definido pelo Tech Lead
- Vitest para testes unitarios e de integracao.
- msw (Mock Service Worker) para APIs externas.
- Execucao em CI via GitHub Actions.
- Coverage minimo: 80% de linhas.

---

## Entregavel 1 - Secao "Testing Standards" (para AGENTS.md)

### Testing Standards

#### 1) Naming Convention (MUST)
- Todos os testes usam `describe` e `it` em ingles.
- Formato obrigatorio para `it`: `should [expected behavior] when [condition]`.
- Titulos vagos sao proibidos: `works`, `should pass`, `endpoint test`.

#### 2) AAA Structure (MUST)
Todo teste explicita as 3 fases:
- Arrange: preparar request, fixtures, mocks msw e contexto.
- Act: executar endpoint/metodo alvo uma unica vez por cenario.
- Assert: validar regras de negocio e contrato de resposta.

#### 3) Assertions (MUST)
- Nao usar `toBeDefined()` ou `toBeTruthy()` como unica validacao.
- Assertions minimas para endpoint de query:
  - `statusCode` esperado.
  - `answer` com comportamento esperado (nao apenas existencia).
  - `source_document` obrigatorio e nao vazio.
  - quando aplicavel, `warning/disclaimer` para baixa confianca.

#### 4) Source-of-Truth Rules for RAG Tests (MUST)
- Resposta deve estar ancorada nos chunks recuperados.
- Em conflito de versoes PROC-042 vs PROC-042-v2, priorizar regra vigente de transicao (chunk PROC-042v2-E).
- FAQ e fonte auxiliar: nao pode contradizer documento normativo.
- Se nao houver cobertura formal (ex.: frete < 500kg), retornar fallback seguro de "nao encontrado".

#### 5) What Tests MUST NOT Do
- Nao acessar servicos reais (Azure OpenAI, Azure AI Search, banco, rede externa).
- Nao depender de ordem de execucao.
- Nao usar tempo real/aleatoriedade sem controle.
- Nao hardcodear payloads irreais (`test`, `hello`) quando houver fixture de dominio disponivel.

#### 6) Mocking Standard (MUST)
- msw para todo trafego HTTP externo.
- factories para requests e payloads.
- mockar fronteiras externas; logica interna permanece real.

#### 7) Fixture Standard (MUST)
- Guardar fixtures reutilizaveis em `/tests/fixtures/`.
- Estruturar por:
  - `queries`
  - `retrievedChunks`
  - `expectedResponses`
- Usar perguntas e chunks reais do Anexo B.

Exemplos minimos de fixture recomendada:
- Query: "Posso devolver carga perigosa?"
- Chunks esperados: `POL-001-B` (opcional `FAQ-03`)
- Resultado esperado: negativa explicita + orientacao para Gestao de Riscos (ramal 4500).

#### 8) CI and Coverage (MUST)
- Suite deve rodar em GitHub Actions sem dependencia local.
- Coverage minimo de 80% de linhas.
- PR abaixo de 80% reprova gate de QA.

---

## Entregavel 2 - Reescrita do teste ruim

### Before (ruim)

```ts
test('query endpoint works', async () => {
  const result = await handler({ body: '{"question": "test"}' });
  expect(result).toBeDefined();
});
```

### After (seguindo os padrões)

```ts
import { describe, expect, it } from 'vitest';

import { handler } from '../../src/query/handler';
import { buildQueryRequest } from '../factories/queryRequest.factory';
import { server } from '../mocks/server';
import { http, HttpResponse } from 'msw';

describe("Query/Gandler", () => {
  test("Should return explicit denial and source_document when user asks dangerous cargo return", async () => {
    // Arrange
    server.use(
      http.post('https://search.mock/retrieve', () => {
        return HttpResponse.json({
          chunks: [
            {
              id: 'POL-001-B',
              text: 'Cargas perigosas nao sao elegiveis para devolucao no processo padrao.'
            }
          ]
        });
      })
    );

    const event = buildQueryRequest({
      question: 'Posso devolver carga perigosa classe 3?'
    });

    // Act
    const response = await handler(event);
    const body = JSON.parse(response.body);

    // Assert
    expect(response.statusCode).toBe(200);
    expect(body.answer).toContain('nao');
    expect(body.answer).toContain('processo padrao');
    expect(body.answer).toContain('ramal 4500');
    expect(body.source_document).toBe('POL-001-B');
  });
});
```

### Melhorias aplicadas (antes x depois)
1. Naming descritivo:
- Antes: `query endpoint works`.
- Depois: comportamento e condicao explicitos.

2. AAA explicito:
- Antes: sem separacao clara.
- Depois: Arrange/Act/Assert visiveis e auditaveis.

3. Mocking correto:
- Antes: dependencias externas implicitas.
- Depois: msw controla retrieval e torna o teste deterministico.

4. Assertions objetivas de negocio:
- Antes: apenas `toBeDefined()`.
- Depois: valida negativa explicita, orientacao correta e `source_document` exato.

5. Dado realista do dominio:
- Antes: pergunta generica `test`.
- Depois: caso real baseado em POL-001-B / Anexo B.

---

## Entregavel 3 - Criterios de code review de QA (objetivos)

Use os criterios abaixo como gate binario (pass/fail):

1. Estrutura minima obrigatoria
- Passa: teste mostra Arrange, Act e Assert de forma verificavel.
- Falha: ausencia de uma etapa ou mistura que impossibilita auditoria.

2. Assertion de regra de negocio
- Passa: valida regra concreta (ex.: carga perigosa nao devolve no processo padrao).
- Falha: apenas assertion vaga (`toBeDefined`, `toBeTruthy`) ou sem regra de negocio.

3. Rastreabilidade de fonte
- Passa: valida `source_document` nao vazio e coerente com chunk esperado.
- Falha: nao verifica `source_document` ou aceita valor generico sem conferir ID.

4. Hierarquia de fonte (normativo > FAQ)
- Passa: teste falha quando resposta contraria POL/PROC/SLA por causa de FAQ.
- Falha: aceita FAQ como fonte final para decisao critica sem validacao normativa.

5. Determinismo tecnico
- Passa: usa msw/factories; sem rede real; resultado repetivel em CI.
- Falha: depende de servico externo, horario real, estado compartilhado ou ordem de execucao.

6. Qualidade de dados de teste
- Passa: usa perguntas/chunks realistas do dominio logistico (Anexo B).
- Falha: usa dados artificiais (`test`, `hello`) para cenarios de negocio.

---

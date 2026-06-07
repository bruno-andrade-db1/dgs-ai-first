# Exercicio 2.3 - Definicao de skill de geracao de testes

## Objetivo
Criar a skill `create-integration-test` (nivel Artifact) para orientar geracao consistente de testes de integracao no projeto NovaTech, alinhada ao Exercicios 2.1/2.2 e aos Anexos A/B.

---

## Entregavel 1 - SKILL.md (completo)

> Conteudo abaixo pronto para uso como `SKILL.md`.

```md
# SKILL: create-integration-test

## Skill Metadata
- Skill ID: create-integration-test
- Level: Artifact
- Owner: QA
- Primary Stack: TypeScript, Vitest, msw
- Domain: NovaTech Assistant (RAG)

## Activation
Use esta skill quando o usuario pedir qualquer uma das frases abaixo (ou intencao equivalente):
- "crie um teste de integracao"
- "escreva teste para endpoint de query"
- "gerar teste de RAG"
- "validar source_document e guardrails"

## Purpose
Gerar testes de integracao deterministicos para o query endpoint com:
- estrutura AAA explicita,
- assertions especificas de regra de negocio,
- mocking externo via msw,
- fixtures realistas de logistica,
- rastreabilidade de fontes via chunk IDs do Anexo B.

## Required Inputs
- Endpoint ou metodo alvo.
- Requirement(s) ou Verification Criteria (ex.: VC-01..VC-04).
- Contrato de resposta esperado (`statusCode`, `answer`, `source_document`, `warning` quando aplicavel).
- Cenario(s) de negocio.

## Source-of-Truth Rules (MUST)
1. Priorizar fontes normativas: POL-001, PROC-042-v2, SLA-2024.
2. FAQ e fonte auxiliar (nao normativa): pode complementar, nao pode sobrepor norma.
3. Em contradicao PROC-042 vs PROC-042-v2, respeitar vigencia de PROC-042v2-E.
4. Se nao houver cobertura formal (ex.: frete < 500kg), retornar fallback seguro (nao encontrado/baixa confianca).
5. Nunca afirmar informacao que nao esteja nos chunks recuperados.

## Test Writing Rules (MUST)
- Naming:
  - `describe('ModuleName', ...)`
  - `it('should [behavior] when [condition]', ...)`
- Estrutura AAA obrigatoria em todo teste.
- Nao usar `toBeDefined()`/`toBeTruthy()` como unica validacao.
- Validar `source_document` sempre que houver resposta de conhecimento.
- Nao usar servico real; usar msw para HTTP externo.
- Sem dependencia de ordem entre testes.

## Test Template
```ts
import { describe, expect, it } from 'vitest';
import { http, HttpResponse } from 'msw';

import { server } from '<path>/tests/mocks/server';
import { handler } from '<path>/src/query/handler';
import { buildQueryRequest } from '<path>/tests/factories/queryRequest.factory';

describe('<module>', () => {
  it('should <expected_behavior> when <condition>', async () => {
    // Arrange
    server.use(
      http.post('<retrieval_url>', () =>
        HttpResponse.json({
          chunks: [
            { id: '<chunk_id_1>', text: '<chunk_text_1>' },
            { id: '<chunk_id_2>', text: '<chunk_text_2>' }
          ]
        })
      )
    );

    const event = buildQueryRequest({
      question: '<realistic_domain_question>'
    });

    // Act
    const response = await handler(event);
    const body = JSON.parse(response.body);

    // Assert
    expect(response.statusCode).toBe(<expected_status>);
    expect(body.answer).toContain('<expected_phrase>');
    expect(body.source_document).toBe('<expected_chunk_id>');
    // Optional: low confidence behavior
    // expect(body.warning).toContain('<expected_warning>');
  });
});
```

## DO Example (good output)
```ts
import { describe, expect, it } from 'vitest';
import { http, HttpResponse } from 'msw';

import { server } from '../mocks/server';
import { handler } from '../../src/query/handler';
import { buildQueryRequest } from '../factories/queryRequest.factory';

describe('query/handler', () => {
  it('should return explicit denial when user asks dangerous cargo return', async () => {
    // Arrange
    server.use(
      http.post('https://search.mock/retrieve', () =>
        HttpResponse.json({
          chunks: [
            {
              id: 'POL-001-B',
              text: 'Cargas perigosas nao sao elegiveis para devolucao no processo padrao.'
            }
          ]
        })
      )
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

## DON'T Example (common AI mistake)
```ts
// Anti-pattern: vague name, no AAA, no source traceability, weak assertion
it('query endpoint works', async () => {
  const result = await handler({ body: '{"question": "test"}' });
  expect(result).toBeDefined();
});
```

## AI Anti-Patterns to Avoid
1. Test title generico (`works`, `should pass`).
2. Apenas assertion de existencia (`toBeDefined`, `toBeTruthy`).
3. Uso de pergunta irreal (`test`, `hello`) em cenario de negocio.
4. Nao validar `source_document`.
5. Misturar norma com FAQ sem hierarquia de fonte.
6. Ignorar contradicao de versao (PROC-042 vs PROC-042-v2).
7. Inventar resposta para pergunta sem cobertura documental.
8. Chamar servicos reais em teste de integracao.
9. Teste dependente de ordem/estado global.
10. Nao testar comportamento de seguranca (prompt injection basico).

## Dependencies (must read first)
Foundation skills:
- testing-basics-vitest
- mocking-http-with-msw
- test-data-factories

Domain skills:
- novatech-source-of-truth-policy
- novatech-rag-chunk-mapping
- novatech-query-endpoint-vc-rules

## Output Contract
Ao usar esta skill, o agente deve retornar:
1. Arquivo de teste pronto para execucao.
2. Mapeamento cenario -> VC (quando houver VC definido).
3. Lista curta de fixtures/chunks usados no teste.
4. Explicacao de 3-5 linhas do porque o teste esta correto.
```

---

## Entregavel 2 - Checklist de revisao de testes (<= 2 minutos)

Instrucoes de uso:
1. Leia apenas o titulo do teste, bloco Arrange/Act/Assert e asserts finais.
2. Marque cada item como Pass ou Fail.
3. Tempo limite: 120 segundos por teste.

### Checklist rapido (Pass/Fail)
- [ ] 1. Naming correto: `it('should [behavior] when [condition]')`.
- [ ] 2. AAA explicito: blocos Arrange, Act e Assert estao visiveis.
- [ ] 3. Assertion util: nao depende apenas de `toBeDefined()`/`toBeTruthy()`.
- [ ] 4. Regra de negocio validada: comportamento esperado esta especifico (ex.: bloqueio de carga perigosa).
- [ ] 5. Rastreabilidade de fonte: valida `source_document` com chunk ID real (ex.: `POL-001-B`).
- [ ] 6. Mocking correto: msw para chamadas HTTP externas (sem rede real).
- [ ] 7. Dados realistas: pergunta/chunks do dominio logistico (Anexo B), sem `test`/`hello`.
- [ ] 8. Guardrail aplicado: normativo prevalece sobre FAQ em caso critico.

### Regra objetiva de aprovacao
- Aprovado: 8/8 itens em Pass.
- Reprovado: qualquer item em Fail.

### Registro rastreavel (modelo curto)
- Test ID:
- VC Link: VC-01 | VC-02 | VC-03 | VC-04
- Resultado: Pass | Fail
- Falhas encontradas (itens):
- Evidencia (arquivo/linha):
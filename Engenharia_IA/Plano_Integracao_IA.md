# Plano de integração e uso de IA

## Resultado esperado

Entregar um fluxo demonstrável do P&D Connect com Copiloto, matching e pré-analise orientada ao PIPE/FAPESP, mantendo revisão humana, rastreabilidade e controle de custo.

## Etapas e cronograma

| Período | Entrega | Critério de conclusão |
|---|---|---|
| Set/2026 | Casos de uso e schema | Campos obrigatórios e limites aprovados pelo PO |
| Set/2026 | Dataset inicial | 10 a 20 casos anonimizados ou fictícios |
| Set-Out/2026 | Benchmark | Dois geradores e dois embeddings medidos no mesmo conjunto |
| Out/2026 | Adaptador de IA | API do backend desacoplada do provedor |
| Out-Nov/2026 | Copiloto | Estruturação, lacunas e histórico funcionando |
| Nov/2026 | Matching e RAG | Ranking, fontes e regras demonstráveis |
| Nov/2026 | Pré-analise | Respostas baseadas em documentos versionados |
| Nov-Dez/2026 | Aceitação | Cenários críticos e validação com a AC2 |
| Dez/2026 | Documentação | Manual, comparativo, integração e avaliação publicados |

## Riscos e respostas

| Risco | Impacto | Resposta |
|---|---|---|
| Resposta imprecisa | Alto | RAG, schema, avaliação e revisão humana |
| Matching pouco pertinente | Alto | Perfis com campos mínimos, rótulos da AC2 e métricas `Precision@k` |
| Custo acima do teto | Médio | Modelo econômico, cache, limites e hard stop |
| API indisponível | Médio | Fila, retry limitado, fallback e modo sem IA |
| Dados sensíveis expostos | Alto | Minimização, anonimização, controle de acesso e modo local |
| Mudança no PIPE/FAPESP | Médio | Versionar fontes e data de ingestão |
| Prazo insuficiente | Alto | Priorizar o fluxo mínimo e adiar recursos não essenciais |

## Controle de custo

Cada chamada deve registrar tokens de entrada e saída, modelo, fluxo e custo estimado. A equipe deve definir um limite mensal e um limite por oportunidade.

Formula mínima:

```text
custo da chamada = (tokens de entrada / 1.000.000 * preco de entrada)
                 + (tokens de saida / 1.000.000 * preco de saida)
```

Para o teto acadêmico estimado de R$ 50,00 por mês, usar dados de teste pequenos, prompts com contexto controlado, cache de conteúdo estável e um modelo de menor custo para classificação e extração.

## Critérios de aceite

- JSON valido em pelo menos 95% dos casos do dataset;
- pelo menos 80% de pertinência no matching, conforme avaliação definida com a AC2;
- pelo menos 80% de aprovação nos cenários críticos;
- cada recomendação de pré-analise apresenta fonte ou e marcada como sem evidencia;
- nenhum fluxo de IA altera decisão de negócio sem confirmação humana;
- falhas externas não impedem salvar e consultar a oportunidade;
- custo e latência ficam visíveis para a equipe;
- nenhuma chave ou dado pessoal desnecessário aparece em logs;
- documentação e versão do prompt ficam associadas ao resultado.

## Definição de pronto das issues

Uma issue pode sair de `Todo` quando o arquivo correspondente estiver revisado, houver evidencias do benchmark ou da decisão técnica, os riscos estiverem registrados e o critério de aceite puder ser demonstrado para a equipe e para o PO.

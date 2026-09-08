# Comparativo de modelos de IA

## Objetivo

Selecionar modelos para o MVP do P&D Connect, considerando o Copiloto de estruturação, a pre-analise PIPE/FAPESP, o matching de pesquisadores e a busca em documentos de referencia.

## Critérios

| Critério | Pergunta de avaliação |
|---|---|
| Qualidade | A resposta é útil em português e preserva o contexto cientifico? |
| Estruturação | O modelo retorna JSON valido e campos completos? |
| Rastreabilidade | A resposta consegue apontar os trechos que sustentam a recomendação? |
| Matching | Os candidatos mais pertinentes aparecem no topo? |
| Custo | O consumo cabe no teto acadêmico estimado? |
| Latência | O fluxo responde dentro do tempo aceitável para o usuário? |
| Privacidade | O uso de dados externos é controlável e documentável? |
| Operação | A equipe consegue integrar, testar e substituir o modelo? |
| Fallback | Existe alternativa quando a API ou o modelo principal falha? |

## Modelos geradores candidatos

| Modelo | Uso sugerido | Vantagens | Riscos | Status |
|---|---|---|---|---|
| GPT-5.6 Terra | Copiloto, estruturação e pre-analise | Equilíbrio entre qualidade e custo; suporte a ferramentas e saída estruturada | Dependência de API, custo em dólar e politica de dados | Benchmark prioritário |
| GPT-5.6 Luna | Extração, classificação e tarefas de alto volume | Menor custo; adequado para tarefas repetitivas | Pode exigir mais validação em síntese complexa | Benchmark econômico |
| Gemini 3.5 Flash | Alternativa hospedada | Saída estruturada e camada gratuita para testes | Regras de dados, limites e disponibilidade variam por camada | Benchmark alternativo |
| Mistral Small | API ou auto-hospedagem | Opções flexíveis e possibilidade de controle de infraestrutura | Licença, infraestrutura e qualidade em português precisam de teste | Candidato de comparação |
| Llama 3.2 3B via Ollama | Fallback local | Pode manter os dados no ambiente local; baixo custo marginal | Memoria, latência e qualidade podem limitar o fluxo | Benchmark local |

Os nomes e preços devem ser verificados novamente na implementação. A lista acima é uma seleção de candidatos, não um resultado de benchmark.

## Embeddings

O candidato local prioritário e `BAAI/bge-m3`, com suporte multilíngue, vetor de 1024 dimensões, sequencia de ate 8192 tokens e possibilidade de busca densa, esparsa e multi-vetorial. A alternativa hospedada deve ser o embedding do mesmo provedor do modelo gerador, quando isso simplificar a operação.

Para o matching, comparar pelo menos:

- `BAAI/bge-m3` local;
- `text-embedding-3-small` ou embedding equivalente hospedado;
- busca lexical por palavras-chave como baseline.

## Recomendação provisória

Usar dois níveis:

1. Modelo gerador hospedado para o Copiloto, com JSON Schema, limites de custo e RAG controlado.
2. Embeddings locais ou hospedados para recuperar perfis e documentos, combinados com regras de negocio e validacao humana.

O primeiro modelo a ser testado deve ser GPT-5.6 Terra, com GPT-5.6 Luna como alternativa de menor custo. O modelo local deve ser medido como fallback, não presumido como equivalente antes do benchmark.

## Benchmark mínimo

Preparar de 10 a 20 casos anonimizados ou fictícios, cada um contendo:

- descrição de problema ou ideia de P&D;
- campos esperados da proposta estruturada;
- competências necessárias;
- perfis relevantes e irrelevantes;
- trechos de referencia do PIPE/FAPESP;
- resposta esperada ou rotulo validado pela equipe.

Registrar por modelo:

- validade do JSON;
- completude dos campos;
- pertinência humana da resposta;
- `Precision@k`, `Recall@k` e `nDCG@k` do matching;
- taxa de afirmações sem evidencia;
- latência media e p95;
- tokens e custo por caso;
- comportamento diante de entrada incompleta.

## Fontes

- [Modelos da OpenAI](https://developers.openai.com/api/docs/models)
- [Controles de dados da OpenAI](https://developers.openai.com/api/docs/guides/your-data)
- [Saida estruturada do Gemini](https://ai.google.dev/gemini-api/docs/structured-output)
- [Precos do Gemini API](https://ai.google.dev/gemini-api/docs/pricing)
- [Modelos e acesso do Llama](https://ai.meta.com/llama/get-started/)
- [Biblioteca do Ollama](https://ollama.com/library)
- [Model card do BGE-M3](https://huggingface.co/BAAI/bge-m3)

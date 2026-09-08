# Plano de execução de IA do P&D Connect

## Objetivo

Este plano organiza as quatro tarefas do épico de IA do projeto P&D Connect e transforma a pesquisa de modelos em entregas verificáveis para o MVP da AC2 Microbiologia.

O produto deve usar IA como apoio para estruturar oportunidades de P&D, identificar lacunas, fazer matching com a rede cadastrada e realizar uma pre-analise orientada ao PIPE/FAPESP. A decisão de continuidade permanece humana e a IA não deve apresentar recomendações como avaliação oficial de fomento.

## Escopo confirmado no GitHub

- [T1 Pesquisar modelos candidatos](https://github.com/PI-IV-2027-TA-INNOVARE/docs/issues/19): lista comparativa com critérios, pros e contras.
- [T2 Pesquisar métodos de integração](https://github.com/PI-IV-2027-TA-INNOVARE/docs/issues/20): opcoes de API, execucao local ou on-device e pipelines.
- [T3 Documentar IA](https://github.com/PI-IV-2027-TA-INNOVARE/docs/issues/21): manual, FAQ e exemplos de uso.
- [T4 Plano de integração e uso](https://github.com/PI-IV-2027-TA-INNOVARE/docs/issues/22): passos, riscos, custos e cronograma.
- [Épico IA Pesquisa e Integração](https://github.com/PI-IV-2027-TA-INNOVARE/docs/issues/16): pai das quatro entregas.

## Decisão técnica inicial para o MVP

Recomenda-se uma arquitetura hibrida e desacoplada:

1. Um modelo de linguagem via API para o Copiloto e para a pre-analise, sempre com saída estruturada em JSON, validação de schema e revisão humana.
2. Um modelo de embeddings para matching e busca semântica dos documentos PIPE/FAPESP e dos perfis cadastrados.
3. Um modelo local como fallback de desenvolvimento e para cenários em que o dado não deve sair do ambiente controlado.
4. Uma camada de adaptação no backend, para trocar o provedor sem alterar os casos de uso Django.

Essa separação evita usar um LLM generativo para decidir sozinho o matching. O modelo pode extrair competências e explicar a recomendação, mas a pontuação final deve combinar similaridade semântica, regras de negócio e validação da equipe.

## Candidatos para a T1

| Candidato | Papel no produto | Pontos fortes | Limites e riscos | Decisão para o MVP |
|---|---|---|---|---|
| GPT-5.6 Terra | Copiloto, estruturação e pre-analise | Modelo balanceado, suporte a ferramentas e saída estruturada, adequado para tarefas com contexto longo | Custo de API, dependência externa e necessidade de politica de dados | Candidato principal para benchmark |
| GPT-5.6 Luna | Extração, classificação e alto volume | Menor custo por token e adequado para tarefas repetitivas | Pode perder qualidade em síntese complexa e justificativas longas | Candidato econômico e possível roteador |
| Gemini 3.5 Flash | Alternativa de API para protótipo e comparação | Camada gratuita para testes e suporte a saída estruturada | Regras de uso de dados variam entre camada gratuita e paga; disponibilidade e limites devem ser confirmados na conta | Candidato alternativo |
| Mistral Small ou modelo equivalente | API ou auto-hospedagem | Opções de modelos abertos e integração flexível | Licença, infraestrutura e qualidade em português precisam ser validadas | Candidato para comparação de custo e controle |
| Llama 3.2 3B ou outro modelo pequeno via Ollama | Fallback local/on-device | Dados podem permanecer no ambiente local; custo de inferência baixo depois da instalação | Qualidade, memoria, latência e suporte a JSON podem ser inferiores | Candidato local para desenvolvimento e fallback |

### Embeddings e matching

O candidato local prioritário e `BAAI/bge-m3`: o model card informa suporte multilíngue, dimensão 1024, sequencia de ate 8192 tokens e possibilidade de combinar busca densa, esparsa e multi-vetorial. Como alternativa hospedada, avaliar `text-embedding-3-small` ou um embedding equivalente do provedor escolhido.

O benchmark deve comparar os embeddings em português com os mesmos casos rotulados pela AC2. Não se deve escolher apenas pelo tamanho do vetor: a métrica principal e a pertinência dos pesquisadores retornados em `Precision@k`, `Recall@k` e `nDCG@k`.

## Critérios de avaliação

Criar um conjunto inicial de casos anonimizados ou fictícios, com oportunidades, perfis e trechos de referencia do PIPE/FAPESP. Cada caso deve ter uma resposta esperada revisada por pelo menos uma pessoa da equipe e, quando possível, validada pela AC2.

Medir:

- validade do JSON e completude dos campos obrigatórios;
- pertinência do matching em `Precision@k`, `Recall@k` e `nDCG@k`;
- taxa de afirmações sem evidencia ou alucinações na pre-analise;
- fidelidade das citações e dos trechos recuperados;
- avaliação humana de utilidade, clareza e necessidade de correção;
- latência media e p95, erros e indisponibilidade;
- tokens por requisição, custo estimado por fluxo e custo mensal;
- comportamento com entrada incompleta, ambígua ou fora do escopo.

O critério de negocio já definido no TAP deve ser preservado: buscar pelo menos 80% de pertinência no matching e 80% de aprovação nos cenários críticos de aceitação.

## T2 Método de integração proposto

Para o MVP, usar três caminhos:

- API síncrona para sugestões curtas do Copiloto, com timeout, retry limitado e resposta amigável em caso de falha.
- Pipeline assíncrono para pre-analise longa, ingestão de documentos e reprocessamento de embeddings.
- Adaptador local para testes, fallback e comparação de privacidade, sem acoplar a aplicação a Ollama ou a um provedor especifico.

Interfaces sugeridas no backend:

- `LanguageModelProvider.generate_structured()`
- `EmbeddingProvider.embed()`
- `RetrievalService.search()`
- `MatchingService.rank_candidates()`
- `EvaluationService.run_case()`

Registrar somente o necessário para rastreabilidade: versão do prompt, modelo, parâmetros, ids dos documentos usados, resultado validado e custo estimado. Não registrar chaves, dados pessoais desnecessários ou prompts completos em logs de produção.

## RAG para PIPE/FAPESP

O fluxo recomendado é:

1. manter uma copia versionada dos documentos públicos usados como referencia;
2. extrair texto e metadados, preservando titulo, seção, pagina e data da fonte;
3. dividir o texto em trechos com sobreposição controlada;
4. gerar embeddings e armazenar os vetores junto dos metadados;
5. recuperar os trechos mais relevantes com busca semântica e, se possível, busca lexical hibrida;
6. enviar ao modelo apenas os trechos recuperados e instruir a resposta a citar as fontes;
7. bloquear conclusões que não estejam apoiadas nos trechos;
8. permitir que o usuário corrija a proposta e execute nova analise.

Como o projeto já usa PostgreSQL, avaliar `pgvector` antes de introduzir outro banco. A decisão final deve considerar a infraestrutura que a equipe realmente conseguira executar e demonstrar ate dezembro de 2026.

## Proteção de dados e controle de custo

- usar dados anonimizados ou fictícios durante desenvolvimento e testes;
- separar dados de perfil, dados de oportunidade e logs de IA por permissão;
- aplicar minimização antes de enviar qualquer texto a um provedor externo;
- guardar chaves em variáveis de ambiente ou segredo da infraestrutura;
- definir limite mensal e limite por usuário ou fluxo;
- limitar tamanho de contexto e quantidade de candidatos retornados;
- usar cache para prompts e documentos estáveis quando o provedor suportar;
- manter um modo de resposta sem IA quando o serviço estiver indisponível;
- exibir claramente que a saída e uma recomendação para revisão humana.

Para respeitar o teto acadêmico estimado de R$ 50,00 por mês, começar com dados de teste pequenos, modelos econômicos e um budget hard stop. O custo deve ser medido em cada requisição, e não inferido apenas pela tabela de preços.

## Entregáveis para fechar as issues

1. `Comparativo_Modelos_IA.md`: matriz da T1, fontes, critérios, benchmark e recomendação.
2. `Metodos_Integracao_IA.md`: API, local, pipeline, contratos de serviço e tratamento de falhas.
3. `Manual_Copiloto_IA.md`: casos de uso, exemplos, FAQ, limites e orientações ao usuário.
4. `Plano_Integracao_IA.md`: arquitetura, passos, riscos, custos, cronograma e critério de aceite.
5. `dataset_avaliacao_ia/README.md`: formato dos casos, rótulos, métricas e procedimento de avaliação.

## Ordem de execução

1. Fechar os casos de uso e os campos obrigatórios com o Product Owner e a AC2.
2. Montar de 10 a 20 casos de teste anonimizados ou fictícios.
3. Comparar dois modelos geradores e dois embeddings no mesmo conjunto.
4. Escolher a combinação do MVP por qualidade, custo, latência e controle de dados.
5. Documentar a integração e criar o manual com exemplos reais do fluxo.
6. Validar riscos, custos e cronograma com a equipe.
7. Anexar os documentos as issues e mover os cards para revisão somente quando os critérios de aceite estiverem demonstrados.

## Fontes técnicas consultadas

- [Modelos da OpenAI](https://developers.openai.com/api/docs/models)
- [Controles de dados da OpenAI](https://developers.openai.com/api/docs/guides/your-data)
- [Saida estruturada do Gemini](https://ai.google.dev/gemini-api/docs/structured-output)
- [Precos do Gemini API](https://ai.google.dev/gemini-api/docs/pricing)
- [Modelos e acesso do Llama](https://ai.meta.com/llama/get-started/)
- [Biblioteca do Ollama](https://ollama.com/library)
- [Model card do BGE-M3](https://huggingface.co/BAAI/bge-m3)

As características, preços, limites e disponibilidade de modelos devem ser rechecados no momento da implementação, pois podem mudar.

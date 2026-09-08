# Métodos de integração de IA

## Escopo

Este documento define como integrar modelos ao backend Python/Django/Django REST Framework do P&D Connect sem acoplar os casos de uso a um provedor específico.

## Arquitetura proposta

```text
Frontend
   |
API Django REST
   |
Servico de aplicacao de IA
   |-- LanguageModelProvider
   |-- EmbeddingProvider
   |-- RetrievalService
   |-- MatchingService
   |-- EvaluationService
   |
PostgreSQL + pgvector       Provedor externo ou modelo local
```

O LLM não deve gravar diretamente no banco nem decidir a aprovação de uma oportunidade. A aplicação valida o schema, aplica regras de negócio e grava o resultado como sugestão revisavel.

## Formas de integração

### API externa

Indicada para o Copiloto durante o MVP. O backend envia somente o contexto necessário, recebe uma resposta estruturada e valida o resultado antes de exibir.

Requisitos:

- chave fora do código-fonte;
- timeout e retry limitado;
- limite de tokens e gasto;
- tratamento de `429`, `5xx` e indisponibilidade;
- minimização ou anonimização de dados pessoais;
- registro de modelo, prompt, versão e custo estimado.

### Modelo local ou on-device

Indicado para desenvolvimento, fallback e dados que não devem sair do ambiente controlado. Uma ferramenta como Ollama pode expor uma API local, mas o backend deve falar com um adaptador próprio.

Riscos a medir:

- memoria RAM/VRAM;
- tempo de inicialização;
- latência por resposta;
- qualidade em português;
- suporte a JSON e chamadas de ferramenta;
- manutenção e distribuição do modelo.

### Pipeline assíncrono

Indicado para ingestão de documentos, criação de embeddings, pre-analise longa e reprocessamento. O endpoint cria um job, o worker executa e o frontend consulta o status.

## Contratos sugeridos

```python
class LanguageModelProvider(Protocol):
    def generate_structured(self, *, prompt: str, schema: dict, model: str) -> dict: ...

class EmbeddingProvider(Protocol):
    def embed(self, texts: list[str], *, model: str) -> list[list[float]]: ...

class RetrievalService(Protocol):
    def search(self, query: str, *, top_k: int, filters: dict) -> list[dict]: ...

class MatchingService(Protocol):
    def rank_candidates(self, opportunity_id: int, *, top_k: int) -> list[dict]: ...
```

## Fluxo do Copiloto

1. Receber a entrada do usuário.
2. Validar tamanho, permissão e campos obrigatórios.
3. Recuperar contexto permitido do banco e do RAG.
4. Chamar o provedor configurado.
5. Validar JSON Schema.
6. Aplicar regras de negócio e marcar campos incertos.
7. Exibir sugestão, evidencias e aviso de revisão humana.
8. Gravar histórico com versão do prompt e do modelo.

## RAG e matching

Usar PostgreSQL com `pgvector` se a infraestrutura suportar a extensão. Cada trecho deve guardar fonte, titulo, seção, pagina, versão e data de ingestão.

O matching deve combinar:

- similaridade semântica entre competências e perfis;
- filtros por disponibilidade, titulação e permissão;
- regras definidas pela AC2;
- explicação curta baseada nos dados do perfil;
- validação humana antes de formar equipe.

## Falhas e fallback

| Falha | Comportamento |
|---|---|
| Timeout da API | Retornar estado pendente ou modo sem IA |
| Limite de requisições | Aplicar backoff, fila e limite por usuário |
| JSON invalido | Tentar uma correção controlada; se falhar, não exibir como resultado confiável |
| Modelo indisponível | Trocar para provedor ou modelo configurado como fallback |
| Documento sem fonte | Não usar o trecho na pre-analise |
| Dado sensível | Bloquear envio externo ou exigir modo local conforme politica |
| Custo acima do limite | Interromper requisição e registrar alerta |

## Observabilidade

Registrar ids e metadados necessários para auditoria, sem armazenar chaves ou dados pessoais desnecessários. Acompanhar latência, erros, tokens, custo, tamanho do contexto, modelo, versão de prompt e avaliação humana.

# Dataset de avaliação de IA

## Finalidade

Medir modelos de forma repetível antes de escolher a integração do MVP.

## Estrutura de cada caso

```json
{
  "id": "case-001",
  "entrada": "descrição anonimizada da oportunidade",
  "campos_esperados": {
    "problema": "...",
    "objetivos": ["..."],
    "competencias": ["..."]
  },
  "perfis_relevantes": ["profile-01", "profile-04"],
  "fontes": ["pipe-versao-data-secao-1"],
  "rotulos": {
    "matching_relevante": true,
    "resposta_util": 4
  }
}
```

## Regras

- usar dados fictícios ou anonimizados;
- não incluir nome, telefone, e-mail ou identificador real;
- manter a mesma entrada para todos os modelos;
- registrar a versão do dataset;
- separar casos de desenvolvimento e casos de aceite;
- guardar a justificativa dos rótulos humanos.

## Métricas

- completude e validade do JSON;
- `Precision@k`, `Recall@k` e `nDCG@k`;
- taxa de afirmações sem fonte;
- avaliação humana de utilidade;
- latência, tokens, custo e taxa de erro.

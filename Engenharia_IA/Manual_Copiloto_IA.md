# Manual do Copiloto de IA

## Finalidade

O Copiloto apoia a equipe na estruturação de oportunidades de P&D. Ele sugere texto, identifica informações ausentes, organiza competências e recupera referências. A equipe do Núcleo de P&D continua responsável por revisar, corrigir e decidir.

## Como usar

1. Cadastre uma oportunidade como problema externo ou ideia interna.
2. Descreva o contexto com o máximo de informação disponível.
3. Solicite a estruturação da proposta.
4. Revise problema, hipótese, objetivos, metodologia, resultados, inovação, infraestrutura e recursos.
5. Solicite a identificação de lacunas.
6. Revise as competências e os pesquisadores sugeridos.
7. Execute a pré-analise PIPE/FAPESP quando os documentos de referencia estiverem disponíveis.
8. Registre a decisão humana e a justificativa no histórico.

## Exemplo de entrada

```text
Queremos investigar um novo ensaio microbiologico para reduzir o tempo de detecção de contaminação em uma etapa do processo. Temos acesso ao laboratório X, mas ainda não definimos a técnica, os controles nem a experiência necessária da equipe.
```

## Exemplo de saida esperada

```json
{
  "problema": "Redução do tempo de detecção de contaminação",
  "hipoteses": ["..."],
  "objetivos": ["..."],
  "competencias_necessarias": ["microbiologia", "validação de ensaio"],
  "lacunas": ["técnica ainda não definida", "controles não informados"],
  "proximos_passos": ["validar técnica", "definir controles"],
  "evidencias": [],
  "precisa_revisao_humana": true
}
```

O exemplo ilustra o formato. O sistema deve substituir os textos genéricos por uma resposta baseada na entrada e nas fontes recuperadas.

## O que revisar antes de aceitar

- o problema foi interpretado corretamente;
- os objetivos são observáveis e coerentes;
- as competências sugeridas são realmente necessárias;
- os pesquisadores retornados possuem perfil compatível;
- toda afirmação sobre PIPE/FAPESP tem fonte;
- não há dados pessoais ou confidenciais indevidos;
- a recomendação não esta sendo tratada como aprovação do projeto.

## FAQ

### A IA aprova uma oportunidade?

Não. Ela produz apoio e recomendações. A decisão de continuidade e do Núcleo de P&D da AC2.

### A IA escolhe sozinha a equipe?

Não. O ranking combina similaridade e regras, mas deve ser validado por uma pessoa responsável.

### Posso confiar em toda resposta?

Não sem revisão. Modelos podem omitir informações ou gerar afirmações incorretas. Use as fontes, o histórico e a avaliação humana.

### O que fazer quando falta informação?

Completar os campos indicados como lacuna ou registrar que a informação ainda não esta disponível. Não inventar valores para obter uma resposta mais completa.

### O que acontece se a IA estiver indisponível?

O sistema deve permitir salvar a oportunidade e continuar os fluxos convencionais. A resposta deve informar que a sugestão automática está indisponível.

### Dados pessoais podem ser enviados a uma API externa?

Somente conforme a politica aprovada pela equipe e pela AC2. Durante o desenvolvimento, usar dados anonimizados ou fictícios e aplicar minimização.

## Limites de uso

- não usar a IA como avaliação oficial da FAPESP;
- não expor perfis a usuários sem permissão;
- não inferir qualificações que não estejam no perfil;
- não apresentar ranking como verdade absoluta;
- não ocultar fontes, incertezas ou falhas;
- não registrar chaves de API nos textos ou logs.

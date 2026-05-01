---
description: Gera mapa conceitual sobre um tema do corpus, ativando a skill
  mapeamento-conceitual e invocando @consultor-corpus para integrar informações
  de múltiplas fichas. Produz mapa salvo em corpus/indice/mapas/. Use antes de
  escrever capítulo/seção que dialoga com literatura específica, para identificar
  estado da arte, divergências, agrupamentos e quadrantes vazios.
argument-hint: [tema | questão] [--projeto slug]
---

# Geração de mapa conceitual

Você vai produzir um mapa do corpus sobre um tema. Protocolo:

## 1. Identificação do tipo de mapa

Identificar projeto ativo conforme hierarquia em CLAUDE.md (seção
"Identificação do projeto ativo"): `--projeto [slug]` se fornecido em
`$ARGUMENTS`, senão inferir por cwd, senão perguntar.

Receber tema/questão em `$ARGUMENTS` (descontando `--projeto [slug]`
se presente). Identificar qual tipo de mapa é mais adequado:

- **Tipo A — Estado da arte**: usuário pergunta "qual o estado da arte
  sobre X?" ou "como o corpus trata X?"
- **Tipo D — Mapa de divergência**: usuário pergunta "onde os autores
  divergem sobre X?" ou explicita uma controvérsia
- **Tipo E — Genealogia argumentativa**: usuário pergunta "como X foi
  evoluindo no corpus?" ou "qual a trajetória da tese Y?"
- **Tipos B e C — Mapas de autor/conceito**: usualmente atualizados pelo
  `@leitor-profundo` durante indexação. Regenerar apenas sob solicitação
  explícita.

Em caso de ambiguidade, perguntar ao usuário qual tipo prefere, com
breve descrição de cada um.

## 2. Refinamento do recorte

Antes de gerar o mapa, refinar o recorte com o usuário se necessário:

- "Mapa do corpus inteiro sobre 'regulação'" provavelmente é amplo
  demais — perguntar qual recorte específico
- "Mapa sobre como Habermas é mobilizado" é específico — prosseguir

Critérios de bom recorte:
- 3 a 25 papers entram no escopo
- A questão é formulável em uma frase
- O resultado afetará decisão real da pesquisa

Se o recorte não atende esses critérios, propor refinamento.

## 3. Delegação

Invocar `@consultor-corpus` com instrução:

> Gere mapa do tipo `[A | D | E]` sobre `[tema/questão]` para o projeto
> `[slug]`. Siga integralmente o protocolo da skill
> `mapeamento-conceitual` para o tipo escolhido.
>
> Recorte: [especificação do recorte]
>
> Eixos sugeridos (se aplicável): [eixos discriminantes propostos]
>
> Salve o mapa em `corpus/indice/mapas/[nome-descritivo].md`.
>
> Devolva: mapa completo, sumário das implicações para a pesquisa, e
> identificação de quadrantes vazios ou lacunas substantivas.

## 4. Apresentação

Apresentar ao usuário:

1. Resumo executivo do mapa (3-5 linhas com a descoberta principal)
2. Mapa propriamente dito (apresentação textual integral, ou referência
   ao arquivo se for muito extenso)
3. **Implicações para a pesquisa** (esta seção é obrigatória — sem ela,
   o mapa é decoração)
4. **Quadrantes vazios identificados** (se houver — frequentemente é
   onde está a contribuição original)
5. **Lacunas substantivas** (questões ausentes do debate)
6. Próximos passos sugeridos:
   - Aprofundar em alguma divergência específica via `/busca` ou
     consulta cruzada
   - Refinar o mapa após indexação de papers adicionais relevantes
   - Usar o mapa como base para escrita de seção/capítulo

## Restrições

- NÃO produzir mapa em ordem cronológica simples (não é mapa, é lista)
- NÃO reproduzir auto-classificação dos autores (mapa serve para
  revelar estrutura latente)
- NÃO omitir quadrantes vazios — esta seção é obrigatória
- NÃO tomar partido sem evidência textual no corpus
- Se o recorte produzir menos de 3 papers, mapa é prematuro: avisar e
  sugerir indexar mais antes

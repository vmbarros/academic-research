---
description: Busca temática no corpus indexado. Aceita conceito, autor, expressão
  ou questão. Invoca @consultor-corpus para consultar o índice estruturado e
  responder com base nas fichas, sem reabrir PDFs originais. Use para localizar
  rapidamente como um conceito ou autor aparece no corpus.
argument-hint: [conceito | autor | expressão de busca] [--projeto slug]
---

# Busca no corpus

Você vai responder a uma consulta de busca usando o índice estruturado.
Protocolo:

## 1. Identificação do projeto e do alvo

1. Identificar o projeto ativo conforme hierarquia em CLAUDE.md (seção
   "Identificação do projeto ativo"): `--projeto [slug]` se fornecido em
   `$ARGUMENTS`, senão inferir por cwd, senão perguntar.
2. Receber o termo de busca em `$ARGUMENTS` (descontando `--projeto [slug]`
   se presente)
3. Classificar o tipo de busca:
   - **Conceito**: termo nominal técnico (ex.: "barganha regulatória",
     "regulação responsiva", "função social")
   - **Autor**: nome próprio (ex.: "Habermas", "Streck", "Mazzucato")
   - **Expressão de busca livre**: questão analítica, citação parcial
     lembrada, etc.

   Em caso de ambiguidade (termo que pode ser nome ou conceito), perguntar.

## 2. Delegação ao subagente

Invocar `@consultor-corpus` com instrução específica ao tipo de busca:

### Para conceito
> Consulte o corpus do projeto `[slug]` sobre o conceito `[termo]`.
> Verifique se existe `corpus/indice/conceitos/[termo].md`. Se sim,
> responda com base nele. Se não, faça grep no frontmatter de todas as
> fichas em `corpus/indice/papers/` por `conceitos_introduzidos` e
> `conceitos_mobilizados` contendo o termo, e responda com base nos
> papers encontrados.
>
> Resposta deve incluir: origem do conceito no corpus, papers que o
> mobilizam (com função e página), divergências semânticas
> identificadas, e localização canônica para cada uso.

### Para autor
> Consulte o corpus do projeto `[slug]` sobre o autor `[nome]`.
> Verifique se existe `corpus/indice/autores/[nome].md`. Se sim,
> responda com base nele. Se não, faça grep no frontmatter por
> `autores_referenciados` contendo o nome, e responda com base nos
> papers encontrados.
>
> Resposta deve incluir: tabela paper × função × página, divergências
> internas (se diferentes papers usam o autor em sentidos
> incompatíveis), e identificação da função recorrente do autor no
> corpus (base teórica? contraste? operacional?).

### Para expressão livre
> Consulte o corpus do projeto `[slug]` sobre `[expressão]`. Faça grep
> no conteúdo das fichas em `corpus/indice/papers/`. Para cada
> ocorrência relevante, devolver: paper, seção da ficha, citação
> contextual, e página do paper original.
>
> Se a expressão sugere uma questão analítica, ativar protocolo da
> skill `consulta-cruzada` (Tipo 5, Localização de trecho).

## 3. Apresentação do resultado

O `@consultor-corpus` produz output integral. Apresentar ao usuário sem
reescrever, mas:

1. Adicionar links wiki para navegação no Obsidian (verificar que estão
   no formato `[[autores/X]]`, `[[papers/Y]]`, `[[conceitos/Z]]`)
2. Se a busca produziu lista de papers, oferecer no fim:
   > Para análise comparativa entre estes resultados, use `/mapa
   > [tema]` ou peça uma comparação cruzada.
3. Se a busca produziu zero resultados, sinalizar:
   > Sem resultados no corpus indexado. Possíveis razões:
   > - O termo não foi indexado ainda (verificar papers em
   >   `corpus/papers/` não fichados)
   > - Variação ortográfica (sinônimo, termo correlato)
   > - Tópico fora do recorte do corpus

## Restrições

- Sempre operar sobre o índice. NUNCA reabrir PDFs.
- Sempre devolver localizações exatas (paper + página + seção da
  ficha) para verificação humana
- Se o corpus do projeto está vazio (sem fichas indexadas), responder:
  > O corpus deste projeto ainda não tem papers indexados. Use
  > `/indexa [pdf]` para começar.

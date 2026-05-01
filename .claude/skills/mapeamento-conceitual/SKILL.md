---
name: mapeamento-conceitual
description: Protocolo para construção de mapas conceituais sobre temas do
  corpus — mapas de autores, mapas de conceitos, mapas de divergências,
  estado da arte sobre uma questão. Use quando o pesquisador pede visão
  sintética sobre um tema antes de escrever, ou quando precisa identificar
  lacunas e oportunidades de contribuição original.
---

# Protocolo de mapeamento conceitual

Esta skill governa a construção de **mapas** — artefatos sintéticos que
organizam o corpus em torno de um eixo temático. Mapas são produtos
intermediários entre o índice e a escrita: ajudam a ver a topografia do
campo antes de tomar posição.

## Princípio fundamental

Mapa não é resumo nem revisão de literatura. Mapa **revela estrutura
latente**: quem se alinha com quem, em que dimensões o campo se
divide, onde estão os silêncios, onde está a posição que ninguém ocupou.

Mapa que apenas lista autores em ordem cronológica é falha de mapa.

## Tipos de mapa

### Mapa A — Estado da arte sobre uma questão

**Quando produzir**: antes de escrever capítulo ou seção que dialoga com
literatura específica.

**Salva em**: `corpus/indice/mapas/estado-arte-[questao].md`

Estrutura:

1. **Pergunta orientadora** (uma frase)
2. **Recorte do corpus** (quais papers entram, quais ficam fora, com
   justificativa)
3. **Eixos de variação** (2-4 dimensões em que os papers do recorte
   se diferenciam — ex.: dogmático vs. zetético; descritivo vs. prescritivo;
   posição quanto a [questão central])
4. **Mapa propriamente dito**: tabela ou texto estruturado posicionando
   cada paper nos eixos identificados
5. **Agrupamentos**: clusters de papers que compartilham posição em
   múltiplos eixos
6. **Anomalias**: papers que ocupam posições solitárias (podem ser
   originais ou marginais — investigar)
7. **Quadrantes vazios**: posições logicamente possíveis não ocupadas
   pela literatura. **Esta seção é frequentemente onde está a
   contribuição original do pesquisador.**
8. **Lacunas substantivas**: questões ausentes do debate, não apenas
   posições não ocupadas

### Mapa B — Mapa de autor

**Quando produzir**: para cada autor recorrente no corpus, manter
arquivo atualizado em `corpus/indice/autores/[nome].md`.

**É atualizado automaticamente** pelo `@leitor-profundo` ao indexar
papers que citam o autor, mas pode ser regenerado por solicitação.

Estrutura:

1. **Identificação** (nome completo, áreas, obras-chave referenciadas
   no corpus)
2. **Conceitos centrais** atribuídos a este autor (links wiki para
   `conceitos/`)
3. **Função no corpus**: este autor é base teórica recorrente?
   contraste recorrente? operacional?
4. **Tabela: papers que mobilizam o autor**
   | Paper | Função | Páginas | Particularidade do uso |
5. **Divergências internas**: o mesmo autor é usado em sentidos
   incompatíveis por papers diferentes do corpus? Sinalizar.
6. **Genealogia**: cadeia de citação dentro do corpus que parte deste
   autor

### Mapa C — Mapa de conceito

**Quando produzir**: para conceito-chave que aparece em 3+ papers do
corpus, manter arquivo em `corpus/indice/conceitos/[termo].md`.

Estrutura:

1. **Definição operacional adotada no projeto** (a definição que VOCÊ
   usa, com indicação da fonte no corpus)
2. **Origem no corpus** (paper mais antigo ou mais influente que cunha
   ou estabiliza o conceito)
3. **Definições no corpus**: tabela ou lista das definições usadas em
   cada paper, com citação literal
4. **Genealogia conceitual**: cadeia de citação interna ao corpus
5. **Divergências semânticas**: identificadas explicitamente — onde
   dois autores usam o termo com sentidos incompatíveis
6. **Conexões com outros conceitos**: links wiki para conceitos
   relacionados (subordinados, equivalentes, contrastantes)

### Mapa D — Mapa de divergência

**Quando produzir**: quando uma divergência entre autores é central
para a pesquisa e merece tratamento dedicado.

**Salva em**: `corpus/indice/mapas/divergencias.md` (mestre) +
`corpus/indice/mapas/divergencia-[autor1-autor2-tema].md` (específico)

Estrutura específica:

1. **Questão em disputa** (uma frase)
2. **Autores envolvidos** (com links para fichas)
3. **Tipologia da divergência** (empírica/conceitual/normativa/de
   ênfase/aparente)
4. **Posição A**: tese, citação literal de suporte, pressupostos
5. **Posição B**: idem
6. **Pontos de acordo subjacente**: o que ambos pressupõem mesmo
   divergindo
7. **Avaliação no corpus**: outros autores tomam partido? Como?
8. **Implicações**: que outras questões o tratamento desta divergência
   afeta?

### Mapa E — Genealogia argumentativa

**Quando produzir**: quando você quer rastrear como uma posição se
desenvolveu ao longo do tempo no corpus.

Estrutura:

1. **Tese rastreada** (qual posição está sendo seguida)
2. **Linha do tempo**: papers em ordem cronológica que tratam da tese
3. **Para cada paper**: posição em relação à tese (formula? confirma?
   reformula? critica? abandona?)
4. **Pontos de inflexão**: momentos em que a tese muda de forma
   significativa
5. **Bifurcações**: onde a linha se divide em ramos distintos
6. **Estado atual no corpus**: a tese está estabilizada? em disputa?
   abandonada?

## Decisões de design ao construir mapa

### Escolha dos eixos

A qualidade do mapa depende dos eixos escolhidos. Eixos triviais
(ex.: "qualitativo vs. quantitativo" para corpus inteiro de teoria
do direito) produzem mapas inúteis. Eixos bons:

- **São discriminantes**: dividem o corpus de modo informativo
- **São analiticamente significativos**: a divisão importa para a
  pesquisa
- **Não são óbvios**: revelam algo que o pesquisador não veria
  apenas lendo papers em sequência

Antes de produzir mapa, justificar os eixos escolhidos em 1-2 frases.

### Recorte do corpus

Mapas são úteis quando o recorte é apertado. Mapa do corpus inteiro
sobre tema vago é ruído. Recorte bom:

- **Por questão específica**: papers que tomam posição sobre X
- **Por conceito central**: papers que mobilizam Y como conceito
  operacional
- **Por interlocução**: papers que dialogam diretamente com Z

Excluir papers do mapa exige justificativa registrada.

### Densidade do mapa

Mapa com 30+ papers é mapa que ninguém lê. Se o recorte produz mais
de 25 papers, considerar:

- Subdivisão em mapas temáticos
- Filtragem por relevância para a questão específica
- Representação tabular compacta em vez de prosa

## Output padrão

Mapa termina sempre com seção **"Implicações para a pesquisa"**:
o que este mapa torna visível que afeta decisões da pesquisa em curso?

Sem essa seção, o mapa é decoração intelectual. Com essa seção, é
ferramenta.

## Anti-padrões

NÃO faça:
- Mapa em ordem cronológica sem outro eixo (não é mapa, é lista)
- Mapa que reproduz a auto-classificação dos autores (mapa serve
  para revelar estrutura que os próprios autores não veem)
- Mapa sem quadrantes vazios identificados (você falhou em pensar
  sobre o que poderia estar lá e não está)
- Mapa que evita tomar posição quando a evidência sustenta uma
  (mapa não é neutralidade pusilânime)
- Mapa que toma posição quando a evidência não sustenta (mapa
  também não é especulação travestida de análise)

---
name: indexacao-paper
description: Protocolo padronizado de indexação de paper/livro/capítulo/tese
  no corpus. Use quando precisar fichar um documento novo, refichar um existente
  com template atualizado, ou validar fichas existentes contra o padrão. Contém
  o template completo, regras de preenchimento e checklist de qualidade.
---

# Protocolo de indexação de paper

Esta skill define o padrão de fichamento operado pelo `@leitor-profundo`
e validado pelo `@consultor-corpus`. É o **contrato de forma** das fichas
em `corpus/indice/papers/`.

## Princípios

1. **Ficha = substituto operacional do paper**. PDFs não serão reabertos
   em consultas futuras. O que não está na ficha está perdido.

2. **Frontmatter é contrato**. O YAML é consultado por Dataview no Obsidian
   e por grep no Claude Code. Sua estrutura não é flexível — é interface.

3. **Densidade controlada**. Ficha curta demais é falha; ficha longa demais
   queima contexto futuro. Alvo: 1500-3500 palavras para artigo, 4000-8000
   para livro/tese.

4. **Citação textual é recurso escasso**. Máximo 5 citações literais
   diretas, cada uma sob 30 palavras, selecionadas por utilidade futura
   prevista — não por beleza.

## Template de frontmatter

```yaml
---
chave: [autor-ano-titulo-curto]              # kebab-case, único
autores: [Sobrenome, Nome; Sobrenome2, Nome2]
ano: YYYY
titulo: [título completo]
subtitulo: [se houver]
tipo: artigo | capitulo | livro | tese | dissertacao | parecer | revisao
revista_ou_editora: [nome]
volume: [número ou em branco]
numero: [número ou em branco]
paginas: [intervalo, ex.: 45-72]
doi: [se houver]
idioma: pt | en | es | de | fr | it | la | outro
tema_central: [palavra ou expressão curta]
campos: [lista de subcampos jurídicos pertinentes]
conceitos_introduzidos: [conceitos novos cunhados pelo autor]
conceitos_mobilizados: [conceitos importantes usados, não cunhados]
autores_referenciados: [autores citados com peso significativo]
divergencias_explicitas_com: [autores criticados nominalmente]
genero_argumentativo: empirico | teorico | dogmatico-sistematico | zetetico-critico
indexado_em: YYYY-MM-DD
---
```

### Regras de preenchimento

- **chave**: `[primeiro-autor-sobrenome]-[ano]-[palavra-chave-titulo]`. Ex.:
  `ayres-1992-responsive`, `streck-2017-jurisdicao`. Único no projeto.
- **conceitos_introduzidos** vs **conceitos_mobilizados**: distinção
  crucial. *Introduzidos* = cunhados pelo autor neste texto OU em texto
  anterior do mesmo autor. *Mobilizados* = usados de forma central, mas
  com origem em outro autor.
- **genero_argumentativo**: classificação operacional, não rótulo
  acadêmico. Um paper pode misturar — escolher o predominante. Em caso
  de mistura genuinamente equilibrada, registrar duas tags.
- **autores_referenciados**: incluir apenas autores com peso real no
  argumento (>2 menções com função argumentativa). Não listar citações
  de erudição.

## Estrutura obrigatória do corpo

Em ordem fixa:

1. **Tese central** (1 parágrafo)
2. **Estrutura argumentativa** (mapa seção→função)
3. **Conceitos operativos** (com definição literal e link wiki)
4. **Método ou Arquitetura argumentativa**
5. **Referências mobilizadas** (tabela classificada por função)
6. **Citações textuais selecionadas** (máx. 5)
7. **Lacunas e silêncios**
8. **Crítica metodológica**
9. **Registro epistêmico** (dogmático / zetético / mistura)
10. **Conexões com o corpus** (links bidirecionais com outras fichas)
11. **Notas para uso futuro**

## Tipologia da função argumentativa de referências

Ao classificar referências citadas pelo autor, usar UMA das categorias:

- **Base teórica**: autoridade fundadora; sem ela, o argumento desmorona
- **Dado empírico**: fonte de evidência factual
- **Operacional**: fornece ferramenta conceitual aplicada
- **Contraste**: interlocutor combatido nominalmente
- **Apoio periférico**: citação ornamental ou de erudição

Categorias mistas são aceitáveis (ex.: "base teórica + operacional"),
mas raras. Se você está hesitando entre 3+ categorias, releia a passagem
em que o autor é citado: o uso geralmente é mais específico do que parece.

## Tipologia do registro epistêmico

A questão a responder: o autor *toma a norma como dado* e a sistematiza
(dogmática), ou *problematiza as premissas* da própria normatividade
(zetética)?

- **Dogmático-sistemático**: opera dentro do sistema jurídico positivo,
  reconstruindo coerência interna, hierarquias, harmonizações
- **Dogmático-aplicativo**: idem, com ênfase em aplicação a casos
- **Zetético-crítico**: questiona pressupostos do sistema, mobiliza
  ferramentas externas (filosofia, sociologia, economia, história) para
  analisar o direito
- **Híbrido**: combina os dois — comum em teoria do direito contemporânea

## Checklist de qualidade — antes de salvar

Verificar:

- [ ] Frontmatter completo, sem campos inventados nem `[a verificar]` em
  campos que existem no PDF
- [ ] Tese central cabe em 1 parágrafo e é PARÁFRASE (não cópia do
  abstract)
- [ ] Estrutura argumentativa identifica FUNÇÃO de cada seção, não
  resume conteúdo
- [ ] Cada conceito operativo tem definição literal com página
- [ ] Tabela de referências classifica TODAS as citações de peso
- [ ] Citações literais ≤ 5, cada uma ≤ 30 palavras, com página
- [ ] Lacunas identificadas são SUBSTANTIVAS (não nitpicks)
- [ ] Crítica metodológica é analítica, não panegírica nem destrutiva
- [ ] Registro epistêmico identificado e justificado
- [ ] Links wiki para conceitos e autores existentes no corpus

## Refichar paper existente

Quando refichar paper já indexado:

1. Verificar se a chave é a mesma (não trocar chave de paper já citado
   em outras fichas — quebra links)
2. Preservar `indexado_em` original; adicionar `reindexado_em`
3. Comparar nova ficha com antiga; registrar diferenças significativas
   em `corpus/indice/_log-reindexacoes.md`

## Anti-padrões

NÃO faça:
- Resumir parágrafo a parágrafo (use estrutura argumentativa por função)
- Listar autores citados sem função (use tabela classificada)
- Citar literalmente porque "é bonito" (use por utilidade futura)
- Marcar todo paper como "híbrido" no registro epistêmico (essa
  classificação é informativa quando é o caso real, mas se você sempre
  marca híbrido está fugindo da decisão)
- Inventar links wiki para conceitos/autores que não existem no índice
  (criar primeiro o esqueleto do arquivo)

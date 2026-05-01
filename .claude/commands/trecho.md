---
description: Localiza trecho específico no corpus indexado. Aceita combinação
  de autor, tema, expressão lembrada parcialmente, ou conceito + autor. Invoca
  @consultor-corpus para fazer grep nas fichas e devolver localizações exatas
  com paper, página e citação contextual. Use quando precisa achar rapidamente
  "onde foi mesmo que [autor] disse X".
argument-hint: [autor + tema | expressão | conceito + autor] [--projeto slug]
---

# Localização de trecho

Você vai localizar um trecho específico no corpus. Protocolo:

## 1. Análise da consulta

Identificar projeto ativo conforme hierarquia em CLAUDE.md (seção
"Identificação do projeto ativo"): `--projeto [slug]` se fornecido em
`$ARGUMENTS`, senão inferir por cwd, senão perguntar.

Receber consulta em `$ARGUMENTS` (descontando `--projeto [slug]` se
presente). Identificar componentes:

- Autor mencionado? (nome próprio)
- Conceito ou termo técnico?
- Expressão literal lembrada parcialmente?
- Tema ou questão analítica?

Combinações típicas:
- "Streck sobre Alexy" → autor + autor (interlocução)
- "regulação responsiva em Ayres" → conceito + autor
- "aquela passagem sobre captura regulatória" → expressão + tema
- "onde está a definição de barganha regulatória" → conceito + tipo de
  passagem

Se a consulta é muito vaga ("aquele trecho que você lembra"), pedir mais
informação antes de buscar.

## 2. Estratégia de busca

Decidir estratégia conforme componentes:

### Autor + tema/conceito
1. Verificar `corpus/indice/autores/[autor].md` — pode já registrar a
   conexão com o conceito
2. Para cada paper do autor no corpus, ler ficha e procurar o conceito
   nas seções "Conceitos operativos" e "Citações textuais"
3. Devolver localizações com paper + página + citação literal (se
   registrada na ficha)

### Conceito sem autor
1. Verificar `corpus/indice/conceitos/[conceito].md`
2. Listar todos os papers que mobilizam o conceito
3. Para cada um, devolver localização da definição/uso

### Expressão literal
1. Grep no conteúdo das fichas em `corpus/indice/papers/` pela expressão
2. Para cada hit, identificar contexto (qual seção da ficha, qual paper)
3. Se a expressão aparece em "Citações textuais selecionadas" da ficha,
   é citação literal do paper original — devolver com página
4. Se aparece em outras seções, é paráfrase do indexador — devolver
   com cuidado, indicando que é paráfrase

### Tipo de passagem específico
Ex.: "definição de X", "crítica a Y", "exemplo empírico de Z"

1. Buscar em seções específicas da ficha conforme tipo:
   - Definições → "Conceitos operativos"
   - Críticas → "Crítica metodológica" + "divergencias_explicitas_com"
   - Exemplos → seção de método ou citações literais
2. Devolver com localização

## 3. Delegação

Invocar `@consultor-corpus`:

> Localize trecho no corpus do projeto `[slug]` conforme a consulta:
> `[consulta original]`.
>
> Estratégia: [descrição da estratégia escolhida]
>
> Devolva, para cada hit relevante:
> - Paper (link wiki)
> - Página exata no PDF original
> - Seção da ficha onde foi encontrado
> - Citação literal (se a ficha registra) OU paráfrase (sinalizando
>   que é paráfrase do indexador, não texto literal do autor)
> - Contexto breve (1-2 frases)
>
> Se a busca produz 0 resultados, indicar onde mais buscar (papers
> ainda não indexados, variação ortográfica do termo, etc.).

## 4. Apresentação

Para cada localização encontrada:

```markdown
### [[papers/chave-do-paper]] — p. X

**Tipo de hit**: [citação literal | paráfrase do indexador | menção em
referência cruzada]

> "[citação ou paráfrase]"

**Contexto**: [1-2 frases sobre como o trecho aparece no argumento]
**Seção da ficha**: [seção onde foi encontrado]
```

Se múltiplos hits, ordenar por relevância (citação literal > paráfrase >
menção indireta).

## Distinção crítica

**Sempre distinguir** na resposta:

1. **Citação literal**: texto entre aspas na ficha, com página — é o
   que o autor *escreveu* literalmente
2. **Paráfrase do indexador**: prosa da ficha, sem aspas — é a leitura
   do `@leitor-profundo`, não o texto do autor
3. **Inferência sobre o autor**: ainda mais distante — afirmação sobre
   o que o autor "quer dizer", não sobre o que ele escreve

Confundir esses planos é o vício maior de pesquisa jurídica preguiçosa.
A resposta DEVE explicitar em qual plano cada trecho se situa.

## Restrições

- NUNCA inventar página
- NUNCA apresentar paráfrase do indexador como se fosse citação literal
- Se o trecho não está na ficha mas o usuário tem certeza de que está
  no paper, sugerir reabrir o paper *manualmente* para verificar (não
  invocar `@leitor-profundo` para reabrir; ele só fica indexação)
- Se 0 resultados, indicar próximos passos (não improvisar resposta
  baseada em "conhecimento geral" sobre o autor)
